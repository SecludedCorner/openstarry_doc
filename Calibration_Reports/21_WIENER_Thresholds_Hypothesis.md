# WIENER L2 + L3 Thresholds — HYPOTHESIS status & re-calibration schedule

> ⚠️ **[漂移更正 — v0.59.6 / 模組已截肢]** 本文 §1/§3/§5/§7 描述的集中化模組 `apps/runner/src/wiener/thresholds.ts`（含 `L2_THRESHOLD`/`L3_THRESHOLD` 常數與其測試）**在當前代碼樹中不存在**。Plan49 的「集中化」交付已於後續被**截肢（AMPUTATED）**：
> - 該目錄不存在：`apps/runner/src/wiener/`（驗：`ls apps/runner/src/` 無 `wiener/`；`Glob **/thresholds.ts` → 0）。
> - `L2_THRESHOLD`/`L3_THRESHOLD` 不在任何 source；僅見於歷史 CHANGELOG 條目（`CHANGELOG.md:1604`，描述當初 v0.49.0-alpha 交付物）。
> - 截肢理由（權威）：`openstarry/CHANGELOG.md:306-308`「**Plan49 wiener thresholds (79 LOC): "centralization" was architecturally impossible** (its consumer spc-monitor is a separate package that cannot import runner internals); event-contract ownership moved to its emitter.」
>
> 換言之：跨 package 集中化在架構上不可能（消費者 `spc-monitor` 是獨立 package，不得 import runner 內部），事件契約（`wiener_threshold_hit` 等）的擁有權改由其 emitter 持有。**HYPOTHESIS 免責聲明（§2）談的是「閾值『數值』未經再校準」，與「模組是否存在」無關——本文原先把『模組已建、測試已過』與『數值仍是 HYPOTHESIS』混為一談，前者已被代碼推翻，故下方逐項更正。** 以下原文保留作為歷史交付紀錄。

**Status**: Plan49 C49-M5a / C49-M5c / C49-M5g — preparation delivery. **(模組已於後續截肢，見上方更正牌)**
**Effective from**: v0.49.0-alpha (2026-04-24)。
**Module**: ~~`apps/runner/src/wiener/thresholds.ts`~~ — **AMPUTATED（不存在於當前代碼樹；見上方更正牌）**。

## 1. Scope

This document records the HYPOTHESIS status of the WIENER L2 + L3 safety-framework thresholds carried from Plan44/45 and the conditions under which VALUE changes become admissible. Plan49 is **preparation only** — it centralises the constants in an out-of-Core module but does **not** re-tune any value (C49-M5e binding).

## 2. σ transparency disclaimer (C49-M5g — MUST, unconditional)

> σ in the current deployment is a composition index over a deterministic event-count vector (cycle 03-13 §四 finding); it is **NOT** a variance estimator over a stochastic signal. Plan44/45 L2+L3 thresholds inherit HYPOTHESIS status until **(a) Rule #72 N≥10 is met AND (b) Plan50 σ_regime annotation is live + telemetry-validated**. The FR-2-pooled-mode activation rationale (formerly cited in prior spec drafts) has been **withdrawn**; the re-calibration trigger is **Rule #72 N≥10**, cited directly.

This disclaimer is binding on any delivery_report, research artefact, or downstream consumer that references `L2_THRESHOLD` or `L3_THRESHOLD`.

## 3. Current state

- `L2_THRESHOLD` and `L3_THRESHOLD` centralised in `apps/runner/src/wiener/thresholds.ts`.
- Values carried forward from Plan44/45 baseline; **no value changes in Plan49**.
- Observation count N = 4/10 (cycles 03-09 through 03-13 contribute data points; projected N≥10 around cycle 03-19 / R20).
- Telemetry event name `wiener_threshold_hit` exported for Plan48 structured-log + audit-sink integration (C49-M5b SHOULD; producer plumbing deferred in the initial v0.49.0-alpha delivery — consumer name is available for early integration).

## 4. Re-calibration schedule

Threshold value tuning becomes admissible only when both conditions hold:

| Condition | Source | Status (2026-04-24) |
|-----------|--------|---------------------|
| Rule #72 N≥10 observation gate | Rule #72 §72.1 | N=4/10 (open at ~cycle 03-19) |
| Plan50 σ_regime annotation live + telemetry-validated | Plan50 spec `Technical_Specifications/Plan50_Sigma_Regime_Binding.md` | Pending Plan50 delivery |

Both gates open independently. Whichever opens later is the binding date.

## 5. MR-6 Core-zero posture

Plan49 extraction of WIENER literals into `apps/runner/src/wiener/thresholds.ts` is MR-6 compliant (Plan49 C49-M5f):

- Current location of L2/L3 literal references in `packages/core/` at Plan49 kickoff: 0 (audit via `grep -rn "L2_THRESHOLD\|L3_THRESHOLD" packages/core/src/`).
- This module introduces no new Core import edge; Core does not import from `apps/runner/` (architectural rule).
- Therefore Plan49 Gate 2 (Core-import-surface delta) verdict: **ALLOWED — refactor-internal to apps/runner**.

## 6. Plan50 forward-reference

Plan50 introduces the `σ_regime` discriminant annotation:

```
σ_regime ∈ { deterministic_composition_index, stochastic_variance_estimator, mixed }
```

Plan49 consumes this annotation as **read-only metadata** (via doc + comments only); Plan50 delivers the annotation code. See `Technical_Specifications/Plan50_Sigma_Regime_Binding.md`.

## 7. Rule #74 L1' sub-checks

> ⚠️ **[漂移更正 — v0.59.6]** 下表原為 v0.49.0-alpha 交付時的 PASS 斷言。模組已截肢後，i 與 ii 不再成立——更正為 **AMPUTATED**，並標明驗證命令。原斷言以刪除線保留。

| # | Sub-check | 原始斷言（v0.49.0-alpha） | 現況（v0.59.6 對代碼驗證） |
|---|-----------|---------------------------|----------------------------|
| i | Code file exists | ~~`apps/runner/src/wiener/thresholds.ts`~~ | **AMPUTATED** — 目錄不存在（`ls apps/runner/src/` 無 `wiener/`；`Glob **/thresholds.ts` → 0）。`CHANGELOG.md:306-308` 記為架構不可能而截肢。 |
| ii | Test file exists | ~~`apps/runner/__tests__/wiener/thresholds.test.ts`~~ | **AMPUTATED** — 測試檔隨模組一併不存在（同上驗證；`L2_THRESHOLD`/`L3_THRESHOLD` 不在任何 source）。 |
| iii | Doc exists | This file + `docs/TW/wiener-thresholds.md` | 本文仍在（已掛更正牌）。 |
| iv | CHANGELOG references it | `CHANGELOG.md` v0.49.0-alpha entry under Plan49 C49-M5 | 歷史交付條目仍在；另見 `CHANGELOG.md:306-308` 截肢紀錄。 |
| v | Cross-refs bidirectional | ~~`thresholds.ts` JSDoc references this doc~~；this doc cross-refs Plan50 spec, Rule #72 | `thresholds.ts` 已不存在，JSDoc 反向交叉引用作廢；本文對外交叉引用仍保留作歷史。 |

## 8. Dissent preserved (MR-11, D-13 3/20)

TANENBAUM, SUSSMAN, RUSSELL voted SHOULD-conditional on C49-M5g (less aggressive v1.7→v1.8 transition). The majority (20) voted MUST-unconditional on the basis that the §四 σ-deterministic finding makes SHOULD-conditional rationale obsolete. Dissent is preserved here as an informational note per MR-11; enforcement remains MUST-unconditional.

## 9. References

- Plan49 engineering spec (full): `share/research_team_suggestion/cycle03-13/deliver/O2_plan49_engineering_spec.md` §2.5
- Plan49 dev spec (concise): `share/research_team_suggestion/cycle03-13/todo/Plan49_dev_spec.md` §1.2 C49-M5
- Rule #72 + §四 finding: `openstarry_doc/Reference/09_Rule_77_SPC_Pooled_Mode_Sigma_Regime.md`
- Plan50 σ_regime: `openstarry_doc/Technical_Specifications/Plan50_Sigma_Regime_Binding.md`
- L3 escalation consumer: `openstarry_plugin/spc-monitor/src/escalation-monitor.ts`
