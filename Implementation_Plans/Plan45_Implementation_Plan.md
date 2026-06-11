<!-- Status: COMPLETE (delivered v0.45.0-alpha 2026-04-15; W3 deferred to Plan46 per Master-ratified LOC gate; header corrected 2026-06-11 repair audit) -->
<!-- Layer: 1-Engineering -->
<!-- Applies to: v0.45.0-alpha -->
<!-- Source: Architecture_Spec published 2026-04-15, cycle 20260415_cycle03-9 -->

# Plan45: WIENER L2+L3 Safety Framework

**Version**: v0.44.0-alpha -> v0.45.0-alpha
**Cycle**: 20260415_cycle03-9
**Master Ratified**: 2026-04-15
**Architecture Spec**: `share/test/reports/arch_reviews/20260415_cycle03-9/Plan45_Architecture_Spec.md`

---

## Overview

Plan45 completes the WIENER safety feedback loop (Tenet #8) by adding L2 Escalation Monitor and L3 Emergency Safety Gate above the existing L1 ShewhartChart (Plan44). Additional scope: Perturbation Diagnostic (W2), Context-Dependent Deltas (W2), Tool-Level Capability Filtering (W3 stretch), SEC-002 HMAC key clearing, SEC-003 ShewhartChart.deserialize validation, and CC-1 plugin dependency CI check.

All logic is plugin-scoped (Tenet #2). Zero Core modifications (Tenet #7). Master-ratified CC-2 Option B: L3 self-implements serialize/fromSnapshot; SDK framework hooks deferred to Plan46.

**DEV-1b Status**: CONDITIONAL CLOSE (2026-04-15). CC-1 and CC-2 completion required or DEV-1b reverts to OPEN.

---

## Wave Structure (Master Ratified 2026-04-15)

| Wave | Content | Target LOC | Priority |
|------|---------|:----------:|:--------:|
| W0 | SEC-002 + SEC-003 + CC-1 CI check + config propagation integration test | ~70-100 | MUST |
| W1 | WIENER L2 Escalation Monitor + L3 Safety Gate + CC-2 Option B (L3 self-serialize) | ~305 | MUST |
| W2 | Perturbation Diagnostic + Context-Dependent Deltas | ~175 | SHOULD |
| W3 | Tool-Level Capability Filtering | TBD | COULD (deferable to Plan46 if W0+W1+W2 > 1000 LOC) |

**Total target**: 650-1000 production LOC + ~350 test LOC.

---

## CONDITIONAL CLOSE Requirements (DEV-1b)

### CC-1 (W0, MUST)
- `apps/runner/scripts/verify-plugin-deps.mjs`: build-time script validates all `@openstarry-plugin/*` workspace packages are in `apps/runner/package.json` dependencies.
- Fails CI if any plugin missing.
- Acceptance: AC-W0-CC1.

### CC-2 (W1, MUST)
- L3 SafetyGate implements `serialize()/fromSnapshot()` (Plan44 Fix 12c pattern).
- Framework `PluginHooks.onCheckpoint/onRestore` deferred to Plan46.
- delivery_report §3.1 records K-3 deferral decision.
- Acceptance: AC-W1-CC2.

---

## Frozen Interfaces

See full definitions in Architecture Spec §1. Key types:

- `EscalationLevel` — `'normal' | 'watch' | 'warning' | 'critical'`
- `CategoryEscalation` — per-category snapshot (includes `monitoringOnly`)
- `EscalationEvent` — payload for `audit:spc_escalation`
- `EscalationConfig` — `{ windowMs?, thresholds?: { watch?, warning?, critical? } }`
- `SafetyGateConfig` — `{ enabled?, criticalCategoryThreshold?, cooldownShadowDecisions?, cooldownMs?, forceGear? }`
- `SafetyGateEvent` — payload for `audit:spc_safety_gate`
- `SafetyGateSnapshot` — `{ schemaVersion: 1, lastTriggerMs, shadowDecisionsSinceTrigger }` (CC-2)
- `SpcMonitorConfig` — expanded with `escalation?` and `safetyGate?`
- `PerturbationResult`, `PerturbationDiagnostic`, `PerturbationConfig`
- `ContextDeltaConfig` — `{ enabled?, categoryFactors?, defaultFactor? }`
- `GearArbiterDynamicConfig` — expanded with `perturbation?` and `contextDelta?`
- `PluginCapabilities.allowedTools?: string[]` — SDK extension (W3)
- `DynamicArbiter.forceNextGear(gear: number): void` — new method (W1)

### L3 Cooldown Resolution (§2)

O5 spec had `cooldownMs: 60000` (time-based). R3 D9-Q27 decided `cooldown = 50 shadow decisions`. Architect resolution: **dual-guard (AND logic)**:

```
Re-trigger allowed ONLY WHEN:
  shadowDecisionsSinceTrigger >= cooldownShadowDecisions (default: 50)
  AND
  msSinceTrigger >= cooldownMs (default: 60000)
```

Primary guard: shadow-decision count (D9-Q27 authoritative). Secondary: wall-clock (defense-in-depth). Both default values are HYPOTHESIS (Rule #59), configurable.

---

## Key Design Decisions

| Decision | Value | Source |
|---|---|---|
| L3 opt-in | `safetyGate.enabled = false` | D9-Q28 |
| L3 conservative-only | forceGear only DOWN | D9-Q29 |
| L3 cooldown | 50 shadow decisions (primary) + 60s (secondary, AND logic) | D9-Q27 + architect resolution |
| L3 cooldown unit | Shadow decisions (event count), not pure wall-clock | §2 resolution |
| L3 state persistence | Self-serialize (Option B) | CC-2 Master ratified |
| L3 trigger threshold | criticalCategories >= 2 (HYPOTHESIS) | doc #72 §4 |
| L2 window | 5 min (HYPOTHESIS) | doc #72 §3 |
| L2 thresholds | watch=2, warning=4, critical=7 (HYPOTHESIS) | doc #72 §3 |
| Destructive in L3 | monitoringOnly=true, never triggers L3 | Rule #55, Rule #57 |
| forceNextGear() | One-shot (consumed on next evaluate()) | Plan27b pattern |
| Tool filtering location | apps/runner/ (NOT Core) | Tenet #7, O5 §7.4 Option A |
| ENG-FAB version | v1.4 (F-7 added) | D9-Q33 |

---

## Hard Constraints

| ID | Constraint |
|---|---|
| C45-1 | Zero `packages/core/src/**` changes (Tenet #7) |
| C45-2 | All L2/L3 in spc-monitor + gear-arbiter-dynamic plugins |
| C45-3 | Destructive delta <= 0 at all layers (Rule #55) |
| C45-4 | L2 monitoring-only for destructive/state_modifying (Rule #57) |
| C45-5 | L3 forceGear <= currentGear; if forceGear > currentGear: no-op |
| C45-6 | safetyGate.enabled = false default |
| C45-7 | BIBO stable: L3 output not fed back into L1/L2 |
| C45-8 | All thresholds labeled HYPOTHESIS (Rule #59) |
| C45-9 | SEC-002: key zeroing on dispose |
| C45-10 | SEC-003: ShewhartChart.deserialize validates input |
| C45-11 | ENG-FAB v1.4 full checklist pass |
| C45-12 | Rule #63 L4: factory-level behavioral test for every new config param |
| C45-13 | CC-1 MUST (DEV-1b condition) |
| C45-14 | CC-2 MUST (DEV-1b condition) |
| C45-15 | SafetyGateSnapshot.schemaVersion = 1 |
| C45-16 | L3 cooldown = dual-guard AND logic (count + time) |
| C45-17 | computePerturbationDiagnostic is pure function |
| C45-18 | PluginCapabilities.allowedTools SDK type added even if W3 runtime defers |
| C45-19 | ENG-FAB upgraded to v1.4 (F-7) |
| C45-20 | Rule #68: two-path config verification for all new plugin configs |

---

## New Files

| File | Plugin | Purpose |
|---|---|---|
| `spc-monitor/src/escalation-types.ts` | spc-monitor | L2+L3 type definitions |
| `spc-monitor/src/escalation-monitor.ts` | spc-monitor | L2 EscalationMonitor class |
| `spc-monitor/src/safety-gate.ts` | spc-monitor | L3 SafetyGate class |
| `gear-arbiter-dynamic/src/perturbation-diagnostic.ts` | gear-arbiter-dynamic | Perturbation pure function + types |
| `apps/runner/scripts/verify-plugin-deps.mjs` | runner | CC-1 CI check |

## Modified Files

| File | Plugin | Changes |
|---|---|---|
| `spc-monitor/src/index.ts` | spc-monitor | L2+L3 wiring, SpcMonitorConfig expansion |
| `spc-monitor/src/shewhart-chart.ts` | spc-monitor | SEC-003 validated deserialize |
| `gear-arbiter-dynamic/src/dynamic-arbiter.ts` | gear-arbiter-dynamic | forceNextGear() method |
| `gear-arbiter-dynamic/src/index.ts` | gear-arbiter-dynamic | L3 response listener, config expansion |
| `gear-arbiter-dynamic/src/calibration-bridge.ts` | gear-arbiter-dynamic | Perturbation hook, context-delta |
| `gear-arbiter-dynamic/src/m4a-types.ts` | gear-arbiter-dynamic | ContextDeltaConfig, ShadowConfig augment |
| `distributed-alaya/src/seed-signature.ts` | distributed-alaya | SEC-002 clear() methods |
| `distributed-alaya/src/index.ts` | distributed-alaya | SEC-002 dispose wiring |
| `packages/sdk/src/types/plugin.ts` | SDK | allowedTools in PluginCapabilities |

## Files NOT Modified

```
packages/core/src/**/*  — ZERO changes (C45-1, Tenet #7)
```

---

## ENG-FAB v1.4 Checklist (Additions)

- **F-7** (NEW): Integration test exists for plugin config propagation through runner's plugin-resolver (Rule #68, D9-Q33). Satisfied by `apps/runner/__tests__/integration/plugin-config-propagation.test.ts`.

---

## Wave Dependency Graph

```
W0 (no deps) -> W1 (needs W0 SEC-003 + CC-1) -> W2 (needs W1 CalibrationBridge) -> [end]
                                                   W3 can parallel W2 (no code deps)
```

W3 LOC gate: if W0+W1+W2 > 1000 prod LOC, defer W3 to Plan46 (D9-Q26). Signal before starting W3.
`PluginCapabilities.allowedTools` SDK type is added regardless of W3 deferral (only ~8 LOC).

---

## Tenet Compliance

| Tenet | Status | How Plan45 Achieves |
|-------|:------:|---|
| #2 (Plugin arch) | COMPLIANT | All L2/L3 in spc-monitor plugin |
| #7 (Core purity) | COMPLIANT | Zero packages/core/src/ changes |
| #8 (Control theory) | CLOSES LOOP | L1->L2->L3->forceNextGear feedback complete |

Verification: `grep -r "spc_escalation\|spc_safety_gate\|forceNextGear" packages/core/src/` must return 0 hits post-delivery.

---

## References

- Architecture Spec (frozen interfaces): `share/test/reports/arch_reviews/20260415_cycle03-9/Plan45_Architecture_Spec.md`
- Research spec: `share/research_team_suggestion/cycle03-9/deliver/O5_plan45_engineering_spec.md`
- R3 decisions: `share/research_team_suggestion/cycle03-9/deliver/R3_decision_log.md`
- WIENER framework doc: `share/openstarry_doc/Architecture_Documentation/72_WIENER_L2_L3_Safety_Framework.md`
- Dev TODO: `share/research_team_suggestion/cycle03-9/deliver/todo_README.md`

---

*Plan45 Implementation Plan*
*Cycle 20260415_cycle03-9, Phase 1 complete*
*Author: architect*
*2026-04-15*
