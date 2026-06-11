<!-- Layer: 1-Engineering -->
<!-- Applies to: v0.44.0-alpha+ -->

# SPC Monitor Plugin — Statistical Process Control

`@openstarry-plugin/spc-monitor`

## Overview

The SPC Monitor Plugin provides real-time Statistical Process Control monitoring for Phase 3 shadow decision data. It implements Shewhart control charts to detect out-of-control signals in the dynamic gear arbiter's shadow decisions.

| Field | Value |
|-------|-------|
| Package | `@openstarry-plugin/spc-monitor` |
| Skandha | vijnana (識蘊 — monitoring/observation) |
| Dependencies | `@openstarry/sdk`, `@openstarry-plugin/gear-arbiter-dynamic` (types only) |
| Core modifications | None (Tenet #7) |
| Authority transfer | None (C44-5, Rule #55, Rule #57) |
| Introduced | Plan44, v0.44.0-alpha |

## Architecture

```
gear-arbiter-dynamic                    spc-monitor
┌──────────────────────┐               ┌──────────────────────┐
│ CalibrationBridge     │               │ ShewhartChart         │
│   ┌─ fireShadow()    │  audit:shadow │   ┌─ addDataPoint()  │
│   │  computeShadow()  │──_decision──→│   │  UCL/LCL check   │
│   │  M4aAggregator   │               │   │  20pp check      │
│   └─ emit record     │               │   └─ emit anomaly    │
│                       │               │                       │
│ Phase3Config.enabled  │               │ SpcMonitorConfig      │
│ = true required       │               │ { enabled, windowSize}│
└──────────────────────┘               └──────────────────────┘
                                               │
                                        audit:spc_anomaly
                                        (informational ONLY)
```

## Configuration

```typescript
interface SpcMonitorConfig {
  /** Enable SPC monitoring. Default: true. */
  readonly enabled?: boolean;
  /** Rolling window size for Shewhart chart. Default: 50. */
  readonly windowSize?: number;
}
```

## Plugin Factory

```typescript
import { createSpcMonitorPlugin } from '@openstarry-plugin/spc-monitor';

const plugin = createSpcMonitorPlugin({
  enabled: true,
  windowSize: 50,
});
```

The factory subscribes to `audit:shadow_decision` events from the EventBus. When Phase 3 is disabled (no shadow events emitted), the plugin has no data to process — naturally satisfying C44-6.

## Shewhart Control Chart

### Algorithm
1. Maintain per-category rolling window of deviation values (configurable size).
2. Compute running mean and standard deviation.
3. Set control limits: UCL = mean + 3σ, LCL = mean - 3σ.
4. Compare each NEW data point against PRIOR control limits (before adding to window).
5. If point beyond UCL or LCL → emit `audit:spc_anomaly` event.

### Destructive Category Special Rule (D7-Q7)
For `destructive` and `state_modifying` categories (monitoringOnly = true):
- Agreement rate is tracked per-round via `snapshotAgreementRates()`.
- If agreement rate changes by ≥ 20pp between rounds → emit anomaly.
- Rule #57: These categories are monitoring-only, no automated response.

### State Persistence
- `serialize()` → JSON string of all category windows.
- `ShewhartChart.deserialize(json, windowSize)` → restored chart.
- `reset()` clears all state (e.g., on Phase3Config change).

## Events

### Input: `audit:shadow_decision`
Emitted by `gear-arbiter-dynamic` CalibrationBridge. Payload: `ShadowDecisionRecord`.

### Output: `audit:spc_anomaly`
Emitted when out-of-control signal detected. Payload:

```typescript
interface SpcAnomaly {
  readonly category: string;
  readonly currentValue: number;
  readonly ucl: number;
  readonly lcl: number;
  readonly mean: number;
  readonly std: number;
  readonly windowSize: number;
  readonly monitoringOnly: boolean;
  readonly reason: string;  // Human-readable explanation
}
```

## Safety Properties

1. **No authority transfer**: Plugin returns only `{ dispose }` from factory. No `gearArbiters`, no `pushInput` calls.
2. **Monitoring-only**: Anomaly events are informational. No consumer in Plan44 triggers automated responses.
3. **Rule #55 compliance**: Destructive operations are never delegated, regardless of SPC findings.
4. **Independent plugin**: No Core modifications. Uses standard EventBus subscription pattern.

## Test Coverage

- `spc-monitor.test.ts`: 13 tests covering Shewhart chart computation, anomaly detection, serialization, plugin integration, and safety properties.

---

*SPC Monitor Plugin Architecture — Plan44, v0.44.0-alpha*
