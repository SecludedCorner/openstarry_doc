<!-- Layer: 1-Engineering -->
<!-- Applies to: v0.44.0-alpha+ -->

# Technical Specification: Phase 3 Shadow Decision + M4a Implementation

**Version**: v0.44.0-alpha (Plan44)
**Source**: O4_phase3_design_spec.md (WIENER #12), O5_plan44_engineering_spec.md

---

## 1. Shadow Decision Architecture

### Option B: Pure Function Shadow

The Phase 3 shadow decision system uses a pure function (`computeShadowDecision`) that computes what the dynamic gear arbiter would decide given immutable state inputs, without mutating any state.

```typescript
function computeShadowDecision(
  deltas: readonly number[],
  totalObs: number,
  gear: number,
  dwell: number,
): { readonly shadowGear: number; readonly abstains: boolean }
```

### Isolation Constraints
- **C44-1**: MUST NOT import or reference ManoAggregator. WIENER constants are duplicated locally.
- **C44-2**: Phase3Config.enabled is NOT read in `evaluate()`. Shadow path is completely separate.
- **AC-W1-2**: Shadow decision is NOT reachable from RouteResult path.
- **AC-W1-8**: Shadow fires AFTER routing decision (temporal isolation).

### Integration Point
CalibrationBridge's `fireShadow()` method (Plan44):
1. Records delta to StateTracker (already done in normal event handler).
2. If `shadowConfig.enabled` and `category` present:
   a. Snapshots tracker state (deltas, totalObs) and arbiter state (gear, dwell).
   b. Calls `computeShadowDecision(snapshot)`.
   c. Compares shadow result with actual gear.
   d. Constructs `ShadowDecisionRecord` with latency measurement.
   e. Invokes `shadowConfig.onShadowDecision(record)`.

---

## 2. M4a Data Schema

### Per-Event Record
```typescript
interface ShadowDecisionRecord {
  readonly timestamp: number;
  readonly category: string;           // Risk category
  readonly shadowGear: number;         // What shadow computed
  readonly actualGear: number;         // What arbiter actually has
  readonly agrees: boolean;            // shadowGear === actualGear
  readonly deviation: number;          // |shadowGear - actualGear|
  readonly monitoringOnly: boolean;    // Rule #57 for destructive/state_mod
  readonly trackerSnapshot: TrackerSnapshot;
  readonly computeTimeMs: number;      // Latency instrumentation
}

interface TrackerSnapshot {
  readonly totalObs: number;
  readonly recentDeltaMean: number;
  readonly currentGear: number;
  readonly dwellCounter: number;
}
```

### Per-Round Aggregate
```typescript
interface M4aCategoryRecord {
  readonly category: string;
  readonly totalDecisions: number;
  readonly agreements: number;
  readonly disagreements: number;
  readonly agreementRate: number;       // agreements / totalDecisions
  readonly meanDeviation: number;       // Mean deviation on disagreements only
  readonly monitoringOnly: boolean;
  readonly hypothesisThreshold: string; // HYPOTHESIS label (NOT numeric)
}

interface M4aReport {
  readonly roundId: string;
  readonly timestamp: number;
  readonly categories: readonly M4aCategoryRecord[];
  readonly aggregateAgreementRate: number;
  readonly shadowDecisionCount: number;
}
```

---

## 3. Non-inferiority Framework (Rule #59 Dual-Track)

### HYPOTHESIS Thresholds

All thresholds are labeled "HYPOTHESIS" per AC-W1-5. NOT used for numeric comparison in code.

| Category | Agreement Threshold | Deviation Threshold |
|----------|-------------------|-------------------|
| informational | HYPOTHESIS: A > 0.85 | HYPOTHESIS: D < 0.5 |
| read_only | HYPOTHESIS: A > 0.80 | HYPOTHESIS: D < 0.5 |
| state_modifying | HYPOTHESIS: monitoring-only (Rule #57) | HYPOTHESIS: monitoring-only |
| destructive | HYPOTHESIS: monitoring-only (Rule #57) | HYPOTHESIS: monitoring-only |

### Observation Period
HYPOTHESIS: 3 W2 rounds (n >= 138 info observations for 95% CI width < 0.10).
Exit condition: Research team decision based on accumulated evidence, NOT automatic threshold trigger.

---

## 4. SPC (Statistical Process Control)

### Shewhart Chart Implementation
- Per-category rolling window of deviation values.
- Control limits: UCL = mean + 3σ, LCL = mean - 3σ.
- Point-by-point comparison against PRIOR control limits (before adding new data).
- Destructive anomaly: 20pp agreement rate change (D7-Q7).

### Event Flow
```
audit:tool_audited → CalibrationBridge → recordDelta → fireShadow
  → computeShadowDecision → ShadowDecisionRecord → audit:shadow_decision
  → ShewhartChart.addDataPoint → (anomaly?) → audit:spc_anomaly
```

---

## 5. Configuration

### GearArbiterDynamicConfig (expanded)
```typescript
interface GearArbiterDynamicConfig {
  readonly minSamples?: number;
  readonly coldStartGear?: 1 | 2 | 3;
  readonly phase3?: Phase3Config;        // NEW in Plan44
}

interface Phase3Config {
  readonly enabled: boolean;             // Default: false
}
```

### Factory Wiring
When `phase3.enabled = true`:
1. Factory creates M4aAggregator.
2. Factory builds ShadowConfig with arbiter state accessor.
3. CalibrationBridge receives ShadowConfig and fires shadow on each audit event.
4. Shadow decisions emitted via `audit:shadow_decision` on EventBus.

When `phase3.enabled = false` (default):
- No ShadowConfig created.
- No shadow computation.
- No audit events.

---

## 6. Latency

- Estimated: < 0.002% overhead (pure function, ~0.1ms vs ~5500ms cycle).
- Benchmarked: < 1ms mean over 1000 iterations (AC-W3-4).
- Well below 5% threshold (C44-3, D7-Q36).
- Async unnecessary — shadow fires synchronously in event handler.

---

*Technical Spec 15 — Phase 3 Shadow + M4a Implementation, Plan44*
