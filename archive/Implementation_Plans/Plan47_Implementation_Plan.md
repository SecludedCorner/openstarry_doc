# Plan47 — K-3 Wire-In 5 MUST + HMAC Sync + MR-6 Core 零 Policy

**版本**: v0.47.0-alpha
**狀態**: DELIVERED (2026-04-19)
**上游授權**:
- `Master_Ratification_Plan47_K3_5MUST_APPROVED.md` (2026-04-18, binding constraint)
- `Master_Ratification_Rule_74.md` (Rule #74 L1' code+doc sync)
- `Master_Ratification_ENG_FAB_v1_6.md` (F-8 revised + F-9 new, 42 items)
- Cycle 03-11 R3 24/24 UNANIMOUS (D2)
**前置**: Plan46 (K-3 SDK framework hook + CheckpointManager + capture wrapper)

---

## 1. Scope 摘要

Plan47 收口 Plan46 留下的 K-3 wire-in gap：`capturePluginHooks` 條件 `hooks.onCheckpoint || hooks.onRestore` 在 spc-monitor / gear-arbiter-dynamic 兩家 stateful plugin 的 factory 層不為 true（KERNEL §4.2 grep 證據），導致 `hookMap.size === 0`、runtime 無法 exercise checkpoint/restore。本 Plan 在 factory 層接上 K-3 bridge，並同步加入 HMAC 簽章 + nonce 防重放（硬性限制：不可分階段）。

硬性限制（Master 2026-04-18 批示）：
1. **單機 only** — 跨機版本 gated by §四 pushInput source authentication 未來仲裁
2. **HMAC sign + nonce 必須同 Plan 一次加入** — 不可分階段交付

Scope edge：
- `§四 pushInput source authentication R3/R4 spec` 明確 NOT in scope（Master 03-12+ 才裁決候選 A/B/B'/C）
- V11 範圍維持 [0.7, 1.5]（MR_REQ_5 路徑 C）
- S-3 降級排 03-14+ 正式提案

---

## 2. W0 — Factory Bridge + Composite Schema + 硬性限制

### 2.1 C47-K3-M1 — spc-monitor factory bridge

**檔案**: `openstarry_plugin/spc-monitor/src/index.ts`
**變更**: factory return 加 `onCheckpoint` / `onRestore`，透過新增的 `composite-snapshot` 模組 delegate 至 SafetyGate + ShewhartChart + EscalationMonitor 三個 stateful component
**新增**: `openstarry_plugin/spc-monitor/src/composite-snapshot.ts` — schema + 雙向 capture/apply helper
**依據**: KERNEL §10.1, §4.2; R3 §D2.5 UNANIMOUS

### 2.2 C47-K3-M2 — gear-arbiter-dynamic factory bridge

**檔案**: `openstarry_plugin/gear-arbiter-dynamic/src/index.ts`
**變更**: factory return 加 `onCheckpoint` / `onRestore`，以 manifest name (`@openstarry-plugin/gear-arbiter-dynamic`) 作為 PluginSnapshot.pluginName，內部包裹 StateTracker Plan46 hooks
**重要**: 採 KERNEL §2.2 推薦的 mutate-fields pattern（`StateTracker.onRestore` 內部已實作），保留 CalibrationBridge 的 tracker reference 有效
**依據**: KERNEL §10.2, §2.2

### 2.3 C47-K3-M3 — daemon resume / runner command consumer

**檔案**:
- `apps/runner/src/commands/start.ts` — 移除 L136 `void _checkpointMgr;`，接入 daemon lifecycle
- `apps/runner/src/commands/checkpoint.ts` (NEW) — `runner checkpoint verify|inspect` 離線 subcommand
- `apps/runner/src/utils/snapshot-store.ts` (NEW) — file-system backend + 簽章封裝
- `apps/runner/src/utils/snapshot-hmac.ts` (NEW) — HMAC-SHA256 sign/verify + nonce + replay 防禦

**方案**: KERNEL §5.3 推薦的 **Option B + C 組合**
- **Option B**: daemon startup 若 `--checkpoint-path` 指向既存檔案 → verify + restore；shutdown (SIGINT/SIGTERM/__QUIT__) → checkpoint 寫回
- **Option C**: `openstarry checkpoint verify` / `inspect` 離線 CLI — 無需 live daemon 即可驗章與列清單
- **Backend**: file-system（first iteration；Plan48+ 若需要 plugin 化可擴展）

**MR-6 遵循**: HMAC key / checkpoint path 只接受 CLI flag (`--checkpoint-path`, `--checkpoint-hmac-key`) 或 env (`OPENSTARRY_CHECKPOINT_PATH`, `OPENSTARRY_CHECKPOINT_HMAC_KEY`)，Core 完全無感。

### 2.4 C47-K3-M4 — composite snapshot schema (spc-monitor)

**檔案**: `openstarry_plugin/spc-monitor/src/composite-snapshot.ts` (NEW) + `escalation-monitor.ts` (擴充)
**Schema**:
```
PluginSnapshot {
  pluginName: '@openstarry-plugin/spc-monitor',
  schemaVersion: 1,                           // FROZEN per Rule #45 K-3 reauth
  state: {
    safetyGate?:        { schemaVersion: 1, lastTriggerMs, shadowDecisionsSinceTrigger },
    shewhartChart?:     { schemaVersion: 1, windows: <JSON string> },
    escalationMonitor?: { schemaVersion: 1, states: [[category, { anomalyTimestamps, level, monitoringOnly }]] },
  },
  timestamp
}
```
**新增**: `EscalationMonitor.serialize()` / `fromSnapshot()` / `applySnapshot()` — 補 Plan46 Q-K3-OQ-2 open question（EscalationMonitor 有 internal state，需 K-3 hook）
**依據**: KERNEL §10.1, §Q-K3-OQ-2

### 2.5 C47-K3-M5 — dual-path resolution

**檔案**: `openstarry_plugin/spc-monitor/src/index.ts`
**處置**: `config.safetyGate.snapshot` cold-start path 加 `console.warn` DEPRECATED 訊息；框架 K-3 onRestore 為 authoritative path（"K-3 wins"）；cold-start path 排 Plan48 移除（per KERNEL Q-K3-OQ-4）
**依據**: KERNEL §F-K3-3, §8.3, §Q-K3-OQ-4

### 2.6 HMAC sign + nonce（Master 硬性限制 2）

**檔案**: `apps/runner/src/utils/snapshot-hmac.ts` (NEW)
**演算法**: HMAC-SHA256（Node `node:crypto`）
**簽章材料**: `nonce || ":" || signedAt || ":" || payload`
**Nonce**: 16 bytes `randomBytes`；`NonceRegistry` process-local Set 防 replay
**Key**: 最短 32 bytes，經 CLI/env 注入；不寫入 Core（MR-6）
**覆蓋**: snapshot read/write 雙向路徑
**依據**: GUARDIAN §11 結尾(3), F-L3-SEC-2/3; R3 §D2.5 BABBAGE 翻轉（分階段反而更脆弱）

---

## 3. W1 — Consumer Wire-In

### 3.1 runner start.ts 整合（Path B）

- `start.ts` lifecycle hook：
  - Startup：解析 `--checkpoint-path` / `--checkpoint-hmac-key` / env；若 checkpoint file 存在 → `readSnapshotStore` → `checkpointMgr.restore(snapshots)`；失敗則 log + fall back to fresh state
  - Shutdown：`checkpointMgr.checkpoint()` → `writeSnapshotStore`；shutdown 前才寫，確保 plugin state 尚未被 `dispose()` 清掉

### 3.2 `runner checkpoint` CLI（Path C）

- `checkpoint verify <path> [--hmac-key <hex>]` — 全程 verify（HMAC + nonce + envelope shape）
- `checkpoint inspect <path>` — envelope metadata + plugin name 清單（不驗章，純離線觀察）

### 3.3 dead code 移除（KERNEL F-K3-4）

`apps/runner/src/commands/start.ts:136` `void _checkpointMgr;` 已移除，`checkpointMgr` 改為真實 reference holder 接入 lifecycle。

---

## 4. 非範疇（明確排除）

- **§四 pushInput source authentication R3/R4 spec** — Master 03-12+ 才裁決；Plan47 只做 K-3 wire-in，不含 source auth 設計
- **V11 窗格調整** — 維持 [0.7, 1.5]（MR_REQ_5 路徑 C）
- **S-3 降級** — DEFER 03-14
- **跨機 / 分散式 checkpoint** — 單機 only（Master 硬性限制 1）
- **HMAC key rotation 設計** — 留給 Plan48+（本 Plan 只負責 sign+verify 路徑，key rotation 是 operational policy）

---

## 5. 驗收條件（Acceptance Criteria）

| # | Criterion | 驗證方式 |
|---|-----------|----------|
| AC47-1 | spc-monitor factory return 含 `onCheckpoint` + `onRestore` | `spc-monitor/src/__tests__/factory-k3-bridge.test.ts` |
| AC47-2 | gear-arbiter-dynamic factory return 含 `onCheckpoint` + `onRestore` | `gear-arbiter-dynamic/src/__tests__/factory-k3-bridge.test.ts` |
| AC47-3 | `capturePluginHooks` 對兩家 plugin 皆將 hooks 寫入 hookMap | Plan46 既有 integration test + 新增 factory-bridge test 回歸 |
| AC47-4 | 複合 snapshot round-trip 保留 SafetyGate + ShewhartChart + EscalationMonitor 狀態 | `composite-snapshot.test.ts` |
| AC47-5 | dual-path: `config.safetyGate.snapshot` emit deprecation warn | `factory-k3-bridge.test.ts` dual-path 段 |
| AC47-6 | HMAC 簽章 round-trip；tampered payload / wrong key / tampered signedAt 皆 fail-closed | `snapshot-hmac.test.ts` |
| AC47-7 | snapshot store round-trip（file write → read）保留全部 PluginSnapshot | `snapshot-store.test.ts` |
| AC47-8 | replay 偵測：相同 nonce 第二次讀取回傳 `{ok:false, reason:'replay...'}` | `snapshot-store.test.ts` |
| AC47-9 | `start.ts:136 void _checkpointMgr;` 已移除；`checkpointMgr` 接入 shutdown lifecycle | grep + `start.ts` code review |
| AC47-10 | `runner checkpoint verify` / `inspect` 可離線執行並回傳正確 exit code | CLI manual + checkpoint 命令 unit test |
| AC47-11 | MR-6 Core 零 policy 常量 — Plan47 零 Core 修改 | purity check 無新違規；Plan47 diff grep 證據 |
| AC47-12 | Rule #74 L1' code+doc byte-identical | `diff -rq` release/openstarry_doc vs share/openstarry_doc 為空 |
| AC47-13 | 既有 2580 tests 無 regression | `pnpm test` 2613 passed (+33 new) / 3 skipped |
| AC47-14 | Build clean | `pnpm build` (monorepo) + 28 plugin 無 error |

---

## 6. ENG-FAB v1.6 自評（42 items）

見 `share/engineering_delivery/cycle03-11_plan47/delivery_report.md §ENG-FAB 42 items` 的逐項 PASS/FAIL 表。

---

## 7. 實作摘要 — 檔案清單

### 新增
- `openstarry_plugin/spc-monitor/src/composite-snapshot.ts` (~160 LOC)
- `openstarry_plugin/spc-monitor/src/__tests__/composite-snapshot.test.ts` (~130 LOC)
- `openstarry_plugin/spc-monitor/src/__tests__/factory-k3-bridge.test.ts` (~90 LOC)
- `openstarry_plugin/gear-arbiter-dynamic/src/__tests__/factory-k3-bridge.test.ts` (~60 LOC)
- `apps/runner/src/utils/snapshot-hmac.ts` (~145 LOC)
- `apps/runner/src/utils/snapshot-store.ts` (~150 LOC)
- `apps/runner/src/commands/checkpoint.ts` (~105 LOC)
- `apps/runner/__tests__/utils/snapshot-hmac.test.ts` (~60 LOC)
- `apps/runner/__tests__/utils/snapshot-store.test.ts` (~100 LOC)

### 修改
- `openstarry_plugin/spc-monitor/src/index.ts` — factory + deprecation warn
- `openstarry_plugin/spc-monitor/src/escalation-monitor.ts` — serialize/fromSnapshot/applySnapshot
- `openstarry_plugin/gear-arbiter-dynamic/src/index.ts` — factory K-3 bridge
- `apps/runner/src/commands/start.ts` — daemon lifecycle wire-in + dead code 移除
- `apps/runner/src/bin.ts` — 註冊 CheckpointCommand
- 根 `package.json` — 版本 0.46.0-alpha → 0.47.0-alpha

---

## 8. Post-Delivery 章節

Plan47 交付後任何 code 修改必須記錄於 `share/engineering_delivery/cycle03-11_plan47/delivery_report.md` post-delivery 章節，並按 Rule #62 分類（A+: 方向性錯誤, A: 重大遺漏, B: 小修正, C: 文件同步 etc.）+ 根因。

交付 baseline (2026-04-19)：尚無 post-delivery 修改（N/A）。

---

## 9. Source of Truth 引用

- `Master_Ratification_Plan47_K3_5MUST_APPROVED.md`
- `R1_independent/KERNEL_plan46_k3_hook.md §10, §5.3, §F-K3-*, §Q-K3-OQ-2/-4`
- `R1_independent/GUARDIAN_plan46_L3_security.md §11, §F-L3-SEC-2/-3`
- `R3_debates/R3_decision_log.md §D2 (24/24 UNANIMOUS)`
- `deliver/O1_plan46_super_verification.md §6 + §8 D2`
- `deliver/todo_README.md` (Cycle 03-11)

*Authored by Dev Team (Cycle 03-11 Dev, 2026-04-19)*
