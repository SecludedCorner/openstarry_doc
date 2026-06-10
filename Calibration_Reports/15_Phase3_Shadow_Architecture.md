<!-- Status: CURRENT -->
<!-- Layer: 1-Engineering -->
<!-- Applies to: v0.44.0-alpha (Plan44) -->

# 15. Phase 3 Shadow Decision Architecture

`[Cycle 03-8 新增: Phase 3 design specification]`

## Purpose

Phase 3 confirms non-inferiority: the dynamic arbiter's decisions are not inferior to the static arbiter's decisions (D7-Q17).

## Architecture: Option B — Pure Function Shadow

### Design

A pure function `computeShadowDecision()` computes what the dynamic arbiter would decide, without mutating any state:

```typescript
function computeShadowDecision(
  deltas: number[], totalObs: number,
  gear: number, dwell: number
): { shadowGear: number; abstains: boolean }
```

### Isolation Guarantees

1. **State isolation**: Pure function with immutable inputs. No `this`, no mutation.
2. **Temporal isolation**: Shadow fires AFTER routing decision in CalibrationBridge event handler.
3. **Path isolation**: MUST NOT be reachable from any path that feeds into `RouteResult`.

### M4a Schema (Rule #59)

Per-event: `ShadowDecisionRecord` — timestamp, category, shadowGear, actualGear, agrees, deviation, trackerSnapshot, computeTimeMs

Per-round: `M4aReport` — per-category agreement rate + mean deviation, hypothesis thresholds

## Non-inferiority Thresholds (ALL HYPOTHESIS)

| Category | Agreement | Deviation | Status |
|----------|:---------:|:---------:|--------|
| informational | H-1: > 0.85 | H-5: < 0.5 | HYPOTHESIS |
| read_only | H-2: > 0.80 | H-6: < 0.5 | HYPOTHESIS |
| state_modifying | H-3: monitoring-only | H-7: monitoring-only | Rule #57 |
| destructive | H-4: monitoring-only | H-8: monitoring-only | Rule #57 |

Observation: 3 W2 rounds. Exit: research team decision.

## Latency

Estimated < 0.002% overhead (pure function ~0.1ms vs ~5500ms cycle). Benchmark required.

---

*Source: Cycle 03-8 R3 decisions D8-Q11~Q20, WIENER (#12)*
