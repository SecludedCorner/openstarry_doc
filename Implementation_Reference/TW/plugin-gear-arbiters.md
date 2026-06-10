# Plugin Gear Arbiters — 身分、優先序、與 null-output 契約

**狀態**：Plan49 C49-M7e（O4 doc artefact；D-11 UNANIMOUS）。
**生效版本**：v0.49.0-alpha（2026-04-24）。
**對應 canonical 文件**：`share/openstarry_doc/Architecture_Documentation/42_IGearArbiter_Interface_Spec.md`（Plan42 介面凍結）。

## 1. 目的

釐清已註冊 gear-arbiter plugin（特別是 `@openstarry-plugin/gear-arbiter-dynamic`）的設計意圖行為，避免將其刻意的 null-output 誤讀為 plugin silent failure。本文與 Plan49 C49-M7a（透過 `audit:completed` 傳遞 `riskCategory`）以及 Plan50 rename 範疇（D-01 20/3）互為配套。

## 2. 身分 — 各 arbiter 是什麼

| Plugin | 角色 | `riskCategory` 輸出 |
|--------|------|---------------------|
| `@openstarry-plugin/gear-arbiter-static` | 靜態規則型 routing | 依匹配規則填入 |
| `@openstarry-plugin/gear-arbiter-dynamic` | **WIENER 控制理論自適應 routing**（Plan41 CV-5；"dynamic" 指 runtime-dynamic 的 delta-based 數學，**不是** LLM-dynamic） | **設計上為 `undefined`** — 見 §3 |

## 3. `gear-arbiter-dynamic` null-output 契約

`gear-arbiter-dynamic` 是 **純 WIENER 數學 arbiter**：其輸出是 StateTracker 觀察歷史與 calibration bridge delta 的函數，內部無任何類別分類邏輯。每個它產生的 `GearEvaluation` 都會帶 `riskCategory === undefined`。

這是**刻意的 null output**。它**不是**：

- silent plugin failure。
- 待修的 bug。
- plugin 未安裝或設定錯誤的徵兆。
- 違反任何 `audit:completed` schema 契約的事件（該欄位本就 optional — `riskCategory?: RiskCategory` on `GearEvaluation`，`riskCategory?: string` on `ConfidenceAuditEntry`）。

下游 consumer **必須**將 `undefined` 視為一等值，意義等同於 "沒有類別宣告"，並 fall back 到：

- 鏈上同層的 sibling arbiter（例如 `gear-arbiter-static`）的規則型 routing。
- pipeline 後段被填入的 `gearEvaluation.routeResult` 之 `riskCategory`。
- 直接省略需要 risk-category 的 audit 欄位（保留為 `null`）。

## 4. 優先序與鏈式組合

依 `42_IGearArbiter_Interface_Spec.md`，arbiter 依宣告的優先序評估；第一個 confidence 通過 risk-weighted threshold 的 arbiter 勝出。`gear-arbiter-dynamic` 通常與 `gear-arbiter-static` 一同安裝，並放在較低優先序，使靜態規則先短路（short-circuit），dynamic 才接手 — 但這只是部署層級的可調設定，並非硬性約束。

## 5. Plan49 — Plan50 排序（D-01 20/3）

Plan49 保留 `gear-arbiter-dynamic` 名稱。改名 `→ gear-arbiter-wiener` **延至 Plan50**，依 D-01 20/3，3 票異議（WIENER、LEIBNIZ、TANENBAUM）偏好在 Plan49 內改名，依 MR-11 保留異議紀錄。

Plan50 將交付：(a) atomic rename、(b) dual-name 支援過渡窗口、(c) migration guide、(d) deprecation-warning 排程。Plan50 之前不會有任何 operational 變更。

## 6. `audit:completed` consumer 向前相容性（C49-M7f）

Plan49 C49-M7a 證明 `audit:completed` event payload 的 `riskCategory` 欄位本就 optional 且 pre-existing（Plan32 Wave 5 P0）。TURING 風格的 consumer audit 確認所有 in-tree consumer 皆 ignore-unknown 或將缺失的 `riskCategory` 視為 null，符合 MR-9 forward-compat 保證。

已知 consumer（截至 v0.49.0-alpha）：

- `packages/core/src/observability/audit-trail-writer.ts`（將 optional 欄位寫入 JSONL；`undefined` 視為缺失）。
- `packages/core/src/mano/mano-aggregator.ts`（threshold 計算；`riskCategory` 缺失 → 採 default base threshold）。
- 下游 plugin（WebSocket transport、structured-log forwarder）：以整包 envelope 接收，不對 `riskCategory` 是否存在做 gating。

## 7. 參考

- Plan49 engineering spec §2.7（C49-M7）：`share/research_team_suggestion/cycle03-13/deliver/O2_plan49_engineering_spec.md`
- Plan49 dev spec §1.2 C49-M7：`share/research_team_suggestion/cycle03-13/todo/Plan49_dev_spec.md`
- D-11 UNANIMOUS（O3+O4 composite）：`share/research_team_suggestion/cycle03-13/deliver/R3_decision_log.md §4.5`
- D-01 20/3（Plan50 rename 延後）：同上檔案；異議經 MR-11 保留
- IGearArbiter 介面：`share/openstarry_doc/Architecture_Documentation/42_IGearArbiter_Interface_Spec.md`
- gear-arbiter-dynamic 原始碼：`openstarry_plugin/gear-arbiter-dynamic/src/index.ts`
- audit-completed 型別：`packages/sdk/src/types/confidence-audit-log.ts`
