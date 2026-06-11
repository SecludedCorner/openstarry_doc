<!-- Status: COMPLETE -->
<!-- Layer: 1-Engineering -->
<!-- Applies to: v0.44.0-alpha -->

# Plan44: Phase 3 Shadow Decision + SPC Monitor

`[Cycle 03-8 設計 → 實作完成]`

## Overview

| Field | Value |
|-------|-------|
| Version | v0.43.0-alpha → v0.44.0-alpha |
| Type | Feature Plan (first under S-1 scope expansion) |
| Est. LOC | ~600-900 production |
| Actual LOC | ~455 production + ~280 test |
| Waves | 4 (all complete) |
| Risk | MEDIUM-HIGH |
| D2 Predicate | v2 (effective this Plan) |
| Rule #47 | Even → full traceability |
| Compliance | 10/0/0 (8th consecutive, v0.37→v0.44) |
| Tests | 225 files, 2422 passed, 3 skipped, 0 failed |

## Waves — Delivery Summary

| Wave | Content | Est. LOC | Actual LOC | Status |
|------|---------|:--------:|:----------:|:------:|
| W0 | D2 Predicate v2 + housekeeping | ~20-30 | ~5 | ✅ |
| W1 | Phase 3 shadow + M4a framework | ~200-300 | ~215 | ✅ |
| W2 | SPC Monitor Plugin (D-30-6) | ~200-300 | ~235 | ✅ |
| W3 | NC3 integration tests + latency | ~80-150 | ~280 (test) | ✅ |

## Wave 0: Housekeeping

- **W0-2**: agent_test package.json version fixed (0.42.0-alpha → 0.44.0-alpha)
- **W0-3**: Plugin count = 38 directories (37 plugins + spc-monitor new)

## Wave 1: Phase 3 Shadow Decision + M4a Framework

### New Files
- `shadow-decision.ts` (~45 LOC): `computeShadowDecision()` pure function. WIENER constants duplicated for isolation (C44-1). Takes immutable inputs: `(deltas, totalObs, gear, dwell) → { shadowGear, abstains }`. No `this`, no mutation (AC-W1-1).
- `m4a-types.ts` (~60 LOC): `TrackerSnapshot`, `ShadowDecisionRecord`, `M4aCategoryRecord`, `M4aReport`, `ShadowConfig` interfaces. All fields `readonly`.
- `m4a-aggregator.ts` (~85 LOC): `M4aAggregator` class. Append-only storage. Per-category agreement rate + mean deviation (Rule #59 dual-track). HYPOTHESIS thresholds as strings (AC-W1-5). `isMonitoringOnly()` for destructive/state_modifying (Rule #57, AC-W1-4).

### Modified Files
- `dynamic-arbiter.ts`: Added `getState()` method returning `{ gear, dwell }` (readonly accessor, +5 LOC). No changes to `evaluate()` (C44-2).
- `calibration-bridge.ts`: Added `fireShadow()` method (~30 LOC). Fires AFTER delta recording (AC-W1-8 temporal isolation). Accepts `ShadowConfig` as 4th constructor parameter.
- `index.ts`: Factory wiring — reads `config?.phase3?.enabled`, creates `M4aAggregator`, builds `ShadowConfig`, emits `audit:shadow_decision` events (AC-W1-10). Default: `{ enabled: false }` (AC-W1-6).

### Key Design Decisions
- **Option B (Pure Function)**: Selected per WIENER (#12) Phase 3 design. No dual-mode, no ManoAggregator reference.
- **Temporal isolation**: Shadow fires in CalibrationBridge event handler AFTER routing decision.
- **State accessor pattern**: `DynamicArbiter.getState()` provides readonly view; CalibrationBridge accesses via `ShadowConfig.getArbiterState()` callback.

## Wave 2: SPC Monitor Plugin

### New Plugin: `@openstarry-plugin/spc-monitor`
- Independent plugin (Tenet #2, AC-W2-1). Zero Core modifications (Tenet #7, C44-8).
- Skandha: vijnana (識蘊 — monitoring/observation).
- Subscribes to `audit:shadow_decision` events from gear-arbiter-dynamic.
- Emits `audit:spc_anomaly` events (informational ONLY, C44-5).
- Configurable: `{ enabled?: boolean; windowSize?: number }` (default: true, 50).

### Shewhart Control Chart (`shewhart-chart.ts`)
- Per-category rolling window of deviation values.
- UCL/LCL at ±3σ (standard Shewhart rule).
- Checks PRIOR control limits before adding new point (standard SPC practice).
- Destructive: 20pp agreement change trigger (D7-Q7, AC-W2-5).
- State persistence: `serialize()` / `deserialize()` for cross-session continuity.
- Reset on Phase3Config change.

### Safety Properties
- No authority transfer (C44-5, Rule #55, Rule #57).
- Disabled when Phase3Config.enabled = false (C44-6, naturally via no events).
- No `pushInput`, no `gearArbiters` hooks — purely observational.

## Wave 3: NC3 Integration Tests + Benchmark

### NC3 Tests (all use factory chain, C44-12, AC-W3-7)
- **W3-1 (COND-1)**: `createGearArbiterDynamicPlugin()` → emit invalid payload → `DEFAULT_LOGGER.warn` called via factory→bridge→logger chain.
- **W3-2 (COND-3)**: `createGearArbiterDynamicPlugin({ coldStartGear: 2, phase3: { enabled: true } })` → verify initialGear=2 and shadow events fire.
- **W3-3 (COND-4, CRITICAL)**: `createGearArbiterDynamicPlugin({ coldStartGear: 2 })` → feed MIN_N observations → `evaluate().action === 2` (NOT just id check). Closes Gen4 gap (Rule #63 L4).
- **W3-4**: Shadow computation latency benchmark: 1000 iterations, < 1ms mean (C44-3).
- **W3-5**: SPC plugin integration: mock shadow decisions → Shewhart chart → anomaly detection.

## Constraint Compliance

| ID | Constraint | Status |
|----|-----------|:------:|
| C44-1 | computeShadowDecision isolated from ManoAggregator | ✅ PASS |
| C44-2 | Phase3Config.enabled not read in evaluate() | ✅ PASS |
| C44-3 | Shadow overhead < 5% | ✅ PASS (<1ms) |
| C44-4 | Destructive NEVER delegated (Rule #55) | ✅ PASS |
| C44-5 | SPC monitoring-only | ✅ PASS |
| C44-6 | SPC disabled when Phase3=false | ✅ PASS |
| C44-7 | All thresholds = HYPOTHESIS | ✅ PASS |
| C44-8 | Zero Core modifications (Tenet #7) | ✅ PASS |
| C44-9 | All changes plugin-scoped (Tenet #2) | ✅ PASS |
| C44-10 | Rule #47 even Plan = full traceability | ✅ PASS |
| C44-11 | ENG-FAB v1.3 G APPLICABLE | ✅ PASS |
| C44-12 | NC3 integration tests via factory | ✅ PASS |
| C44-13 | Rule #63 L4 config→behavior | ✅ PASS |

## DEV-1b NC Satisfaction

| NC | How Satisfied | Status |
|----|--------------|:------:|
| NC1 | M4a non-empty (W1 shadow decisions produce M4a data) | ✅ SATISFIED |
| NC2 | ENG-FAB v1.3 G APPLICABLE (Phase 3 shadow) | ✅ SATISFIED |
| NC3 | COND integration tests via factory chain (W3) | ✅ SATISFIED |
| NC4 | Dual verification (TURING + GUARDIAN) | ⏳ PENDING 03-9 |

## Phase 3 Code Review

Verdict: CONDITIONAL → resolved (1 rework item).

| Finding | Severity | Resolution |
|---------|----------|------------|
| COND-A: NC3 COND-1 test lacks behavioral verification | Code Fix | Fixed: spy on `DEFAULT_LOGGER.warn` to verify factory→bridge→logger chain |
| COND-B: coldStartGear ?? 1 location documentation | Note | No rework (functionally identical to Plan43 pattern) |

## Phase 3.5 Security Audit

Verdict: PASS (0 Critical, 0 High).

| Finding | Severity | Status |
|---------|----------|--------|
| SEC-001 | LOW (carry-forward) | ILogger injection surface unchanged |
| SEC-002 | LOW (carry-forward) | Vacuous negMean guard now in 2 files |
| SEC-003 | INFO (new) | ShewhartChart.deserialize() unvalidated JSON.parse (no call site in Plan44) |

## New Rules

| Rule | Content |
|------|---------|
| #63 | L4: Config Activates — new config params require factory-level behavioral verification (config→factory→runtime). Addresses Gen4 fabrication. |

## Source

Cycle 03-8 R3 decisions D8-Q28~Q32. Full specification: O5_plan44_engineering_spec.md.

---

*Plan44 — Cycle 03-8, 2026-04-12*
