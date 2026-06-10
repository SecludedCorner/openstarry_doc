# Schema-Drift 政策

**狀態**：Plan49 Follow-on A（C49-M3a / M3c / M3e / M3f / M3g）— 中央模組 + call-site migration。
**生效版本**：v0.49.0-alpha（2026-04-24）。
**模組**：`apps/runner/src/schema-drift-policy/index.ts`。

## 1. 目的

單一 process-wide 政策模組，用於處理 Zod `safeParse` 邊界上 inbound data 的未知或無效欄位。把之前各 call-site 臨時判斷的行為集中：靜默接受、大聲拋出，或發出結構化日誌稽核事件。

## 2. 模式（`SchemaDriftMode`）

| 模式 | 成功路徑 | 失敗路徑 | 使用場景 |
|------|---------|---------|---------|
| `tolerant`（預設） | `{ ok: true, data }` | `{ ok: false, error, issues }` — 不拋；caller 自行 log 或 fallback | 向後相容 Plan49 之前的行為。 |
| `strict` | `{ ok: true, data }` | `throw SchemaDriftError` | CI / 整合測試期間快速失敗，暴露未預期的 drift。 |
| `audited` | `{ ok: true, data }` | `{ ok: false, error, issues }` + 透過已 wire 的 sink emit `schema_drift_audit` 事件 | 合規敏感部署，需要防竄改的 drift 記錄，但不硬失敗。 |

## 3. 解析順序（C49-M3g process-global）

`resolveSchemaDriftMode()` 在第一次呼叫時讀取一次 `SCHEMA_DRIFT_MODE` 環境變數並快取結果於整個 process 生命週期。Production code 中 call-site **不得**傳入 `overrideMode` — 該參數僅供 unit test 使用。此設計滿足 C49-M3g（single boot-resolved mode；integration test 確認一致性）。

```
process.env.SCHEMA_DRIFT_MODE ∈ { "tolerant", "strict", "audited" }  — 預設 "tolerant"
未知值 → 靜默回退 "tolerant"
```

## 4. Call-site migration 狀態（v0.49.0-alpha）

Plan49 規格列出期望中的 8 個 call-site（event-bus ×3、checkpoint-store ×2、IAgentConfig ×1、WebSocket ×1、hook-registry ×1）。對 v0.48.0-alpha baseline 的 audit 顯示 **safeParse 邊界僅存在於今日已明確呼叫 Zod schema 的地方**。現況：

| Call-site | 狀態 | 模組 |
|-----------|------|------|
| IAgentConfig | ✅ 已遷移 | `apps/runner/src/utils/config-validator.ts` |
| ProjectPermissions | ✅ 已遷移 | `apps/runner/src/utils/permission-validator.ts` `validatePermissionsFile` |
| ProjectPlugins | ✅ 已遷移 | `apps/runner/src/utils/permission-validator.ts` `validatePluginsFile` |
| ProjectConfig（strict + unknown-keys-tolerate 混合）| ◼ 保持原狀 | `validateConfigFile` — 有刻意的 site-specific 語義（unknown key 發警告、用 neutral 欄位重試），與統一政策不相容。文檔中記為 intentional divergence。 |
| `packages/shared/src/utils/validation.ts` 通用 helper | ◼ 保持原狀 | 架構考量：`packages/shared` 比 `apps/runner` 更基礎；從 runner 把政策往 shared 引入會反轉依賴圖。通用 helper 保持不遷；consumer 於呼叫層自行選用政策。 |
| event-bus ×3、checkpoint-store ×2、WebSocket ×1、hook-registry ×1（spec 列舉）| ❌ 不存在 | 這些模組目前沒有 Zod `safeParse` 呼叫；spec 列舉是 aspirational。需先在這些點引入 Zod 邊界，才能遷移（非 Plan49 scope 的獨立重構）。 |

**實際已遷移 call-site**：apps/runner 內 3 個。

## 5. API

```typescript
import {
  applySchemaDriftPolicy,
  resolveSchemaDriftMode,
  setSchemaDriftAuditSink,
  SchemaDriftError,
  type SchemaDriftResult,
  type SchemaDriftMode,
  type SchemaDriftIssue,
  type SchemaDriftAuditEvent,
  type SchemaDriftAuditSink,
} from "./schema-drift-policy/index.js";

// 簡單用法 — 讓政策依 global mode 決定。
const result = applySchemaDriftPolicy(MySchema, rawInput, "my-context");
if (result.ok) {
  use(result.data);
} else {
  log("schema drift", result.error, result.issues);
}

// 啟動時：選擇性地把 audited-mode sink 接到 log infrastructure。
setSchemaDriftAuditSink((event) => structuredLogger.emit(event));
```

## 6. 錯誤型別

`SchemaDriftError extends Error` 僅在 `strict` 模式拋出。帶 `context` 與 `zodIssues` 欄位供 operator 診斷。原本預期「任何 Zod 失敗就 `throw new ConfigError(...)`」的 caller 仍繼續運作，因為已遷移的 site 把 `applySchemaDriftPolicy` 的非-ok 回傳包成 `ConfigError`（strict 模式拋出的 `SchemaDriftError` 則繞過 wrap 向上傳播，保留「invalid → throw」契約）。

## 7. MR-6 姿態（C49-M3f）

模組位於 `apps/runner/src/schema-drift-policy/`。新增 0 個 Core policy 常數；新增 0 條 Core import edge。交付時驗證：

```bash
grep -rn "schema-drift-policy" packages/core/
# → 0 matches
```

## 8. Rule #74 L1' 五項子檢

| # | 子檢 | 證據 |
|---|------|------|
| i | Code 檔案存在 | `apps/runner/src/schema-drift-policy/index.ts` |
| ii | Test 檔案存在 | `apps/runner/__tests__/schema-drift-policy/index.test.ts`（11 tests PASS） |
| iii | Doc 存在 | 本檔 + `docs/EN/schema-drift-policy.md` |
| iv | CHANGELOG | v0.49.0-alpha Plan49 Follow-on A 條目 |
| v | Cross-ref 雙向 | 模組 JSDoc 引用本檔；本檔交叉引用已遷移 call-sites + `docs/EN/plugin-install-reliability.md` + `wiener-thresholds.md` |

## 9. 參考

- Plan49 spec §2.3（C49-M3）：`share/research_team_suggestion/cycle03-13/deliver/O2_plan49_engineering_spec.md`
- Plan49 dev spec §1.2 C49-M3：`share/research_team_suggestion/cycle03-13/todo/Plan49_dev_spec.md`
- D-12 UNANIMOUS（SHOULD per-site + process-global C49-M3g）：`share/research_team_suggestion/cycle03-13/deliver/R3_decision_log.md §4.5`
- MR-6 conditional gates：`share/research_team_suggestion/cycle03-13/openstarry_doc/Technical_Specifications/Plan49_MR6_Conditional_Gates.md`
