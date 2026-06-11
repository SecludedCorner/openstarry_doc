# WIENER L2 + L3 閾值 — HYPOTHESIS 狀態 與 重新校準排程

**狀態**：Plan49 C49-M5a / C49-M5c / C49-M5g — 準備性交付。
**生效版本**：v0.49.0-alpha（2026-04-24）。
**模組**：`apps/runner/src/wiener/thresholds.ts`。

## 1. 範圍

本檔記錄 Plan44/45 繼承而來的 WIENER L2 + L3 安全框架閾值的 HYPOTHESIS 狀態，以及閾值 VALUE 變更何時才可接受。Plan49 屬於**準備性** — 把常數集中到 Core 以外的模組，**不**重新調整任何數值（C49-M5e 拘束）。

## 2. σ 透明性聲明（C49-M5g — MUST，無條件）

> 當前部署中的 σ 是確定性 event-count 向量上的組合指標（cycle 03-13 §四 finding）；**不是**隨機訊號上的變異估計量。Plan44/45 L2+L3 閾值繼承 HYPOTHESIS 狀態，直到**(a) Rule #72 N≥10 達到 AND (b) Plan50 σ_regime annotation 上線並經 telemetry 驗證**。FR-2-pooled-mode 啟動理由（早期 spec 草稿曾引用）已**撤回**；重新校準觸發條件是 **Rule #72 N≥10**，直接引用。

此聲明拘束任何引用 `L2_THRESHOLD` 或 `L3_THRESHOLD` 的 delivery_report、研究文件或下游 consumer。

## 3. 當前狀態

- `L2_THRESHOLD` 與 `L3_THRESHOLD` 集中在 `apps/runner/src/wiener/thresholds.ts`。
- 數值繼承自 Plan44/45 baseline；**Plan49 無數值變更**。
- 觀察計數 N = 4/10（cycles 03-09 至 03-13 的 data point；預計 N≥10 在 cycle 03-19 / R20 左右）。
- Telemetry event 名稱 `wiener_threshold_hit` 已 export，供 Plan48 structured-log + audit-sink 整合（C49-M5b SHOULD；v0.49.0-alpha 初始交付中 producer plumbing 延後 — consumer 名稱可用於先行整合）。

## 4. 重新校準排程

閾值 VALUE tuning 僅在同時滿足下列兩個條件時才被接受：

| 條件 | 來源 | 狀態（2026-04-24） |
|------|------|-------------------|
| Rule #72 N≥10 觀察門檻 | Rule #72 §72.1 | N=4/10（預計 ~cycle 03-19 開啟） |
| Plan50 σ_regime annotation 上線並經 telemetry 驗證 | Plan50 spec `Technical_Specifications/Plan50_Sigma_Regime_Binding.md` | 等 Plan50 交付 |

兩個 gate 獨立開啟；較晚開啟者為拘束日期。

## 5. MR-6 Core-zero 姿態

Plan49 將 WIENER literals 抽離到 `apps/runner/src/wiener/thresholds.ts` 符合 MR-6（Plan49 C49-M5f）：

- Plan49 kickoff 時 `packages/core/` 內 L2/L3 literal 參照數：0（`grep -rn "L2_THRESHOLD\|L3_THRESHOLD" packages/core/src/`）。
- 本模組不引入新 Core import edge；Core 不從 `apps/runner/` 引入（架構規則）。
- 因此 Plan49 Gate 2（Core-import-surface delta）判定：**ALLOWED — apps/runner 內部 refactor**。

## 6. Plan50 前向參考

Plan50 引入 `σ_regime` 判別式 annotation：

```
σ_regime ∈ { deterministic_composition_index, stochastic_variance_estimator, mixed }
```

Plan49 以**唯讀 metadata** 方式消費此 annotation（僅透過 doc + 註解）；Plan50 交付 annotation code。見 `Technical_Specifications/Plan50_Sigma_Regime_Binding.md`。

## 7. Rule #74 L1' 五項子檢

| # | 子檢 | 證據 |
|---|------|------|
| i | Code 檔案存在 | `apps/runner/src/wiener/thresholds.ts` |
| ii | Test 檔案存在 | `apps/runner/__tests__/wiener/thresholds.test.ts` |
| iii | Doc 存在 | 本檔 + `docs/EN/wiener-thresholds.md` |
| iv | CHANGELOG 引用 | `CHANGELOG.md` v0.49.0-alpha Plan49 C49-M5 條目 |
| v | Cross-ref 雙向 | `thresholds.ts` JSDoc 引用本檔；本檔交叉引用 `thresholds.ts`、Plan50 spec、Rule #72 |

## 8. 保留異議（MR-11，D-13 3/20）

TANENBAUM、SUSSMAN、RUSSELL 對 C49-M5g 投 SHOULD-conditional（較不激進的 v1.7→v1.8 過渡）。多數（20）投 MUST-unconditional，理由是 §四 σ-deterministic finding 使 SHOULD-conditional 理由失效。異議依 MR-11 於此保留為資訊性註記；enforcement 仍為 MUST-unconditional。

## 9. 參考

- Plan49 engineering spec（完整）：`share/research_team_suggestion/cycle03-13/deliver/O2_plan49_engineering_spec.md` §2.5
- Plan49 dev spec（精簡）：`share/research_team_suggestion/cycle03-13/todo/Plan49_dev_spec.md` §1.2 C49-M5
- Rule #72 + §四 finding：`openstarry_doc/Reference/09_Rule_77_SPC_Pooled_Mode_Sigma_Regime.md`
- Plan50 σ_regime：`openstarry_doc/Technical_Specifications/Plan50_Sigma_Regime_Binding.md`
- L3 escalation consumer：`openstarry_plugin/spc-monitor/src/escalation-monitor.ts`
