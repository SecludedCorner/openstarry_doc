# WIENER L2 + L3 Thresholds — HYPOTHESIS status & re-calibration schedule

**Status**: Plan49 C49-M5a / C49-M5c / C49-M5g — preparation delivery.
**Effective from**: v0.49.0-alpha (2026-04-24).
**Module**: `apps/runner/src/wiener/thresholds.ts`.

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

| # | Sub-check | Evidence |
|---|-----------|----------|
| i | Code file exists | `apps/runner/src/wiener/thresholds.ts` |
| ii | Test file exists | `apps/runner/__tests__/wiener/thresholds.test.ts` |
| iii | Doc exists | This file + `docs/TW/wiener-thresholds.md` |
| iv | CHANGELOG references it | `CHANGELOG.md` v0.49.0-alpha entry under Plan49 C49-M5 |
| v | Cross-refs bidirectional | `thresholds.ts` JSDoc references this doc; this doc cross-refs `thresholds.ts`, Plan50 spec, Rule #72 |

## 8. Dissent preserved (MR-11, D-13 3/20)

TANENBAUM, SUSSMAN, RUSSELL voted SHOULD-conditional on C49-M5g (less aggressive v1.7→v1.8 transition). The majority (20) voted MUST-unconditional on the basis that the §四 σ-deterministic finding makes SHOULD-conditional rationale obsolete. Dissent is preserved here as an informational note per MR-11; enforcement remains MUST-unconditional.

## 9. References

- Plan49 engineering spec (full): `share/research_team_suggestion/cycle03-13/deliver/O2_plan49_engineering_spec.md` §2.5
- Plan49 dev spec (concise): `share/research_team_suggestion/cycle03-13/todo/Plan49_dev_spec.md` §1.2 C49-M5
- Rule #72 + §四 finding: `openstarry_doc/Reference/09_Rule_77_SPC_Pooled_Mode_Sigma_Regime.md`
- Plan50 σ_regime: `openstarry_doc/Technical_Specifications/Plan50_Sigma_Regime_Binding.md`
- L3 escalation consumer: `openstarry_plugin/spc-monitor/src/escalation-monitor.ts`
