<!-- Status: DELIVERED -->
<!-- Layer: 1-Engineering -->
<!-- Applies to: v0.46.0-alpha -->
<!-- Source: Architecture_Spec published 2026-04-16, cycle 20260416_cycle03-10 -->

# Plan46: Deferred Cleanup — SEC + Tool Filtering + K-3 SDK Hook

**Version**: v0.45.0-alpha → v0.46.0-alpha
**Cycle**: 20260416_cycle03-10
**Master Ratified**: 2026-04-16
**Architecture Spec**: `share/engineering_delivery/cycle03-10_plan46/architecture_spec.md`
**Delivery Report**: `share/engineering_delivery/cycle03-10_plan46/delivery_report.md`

---

## Overview

Plan46 completes three deferred items from the previous cycles:

1. **Wave 0 — Security Cleanup**: SEC-001 nonce counter (ISeed replay protection), SEC-004 vector clock pruning, SEC-R1 transport-http JSON validation, SEC-R2 transport-websocket JSON validation, StateTracker.fromSnapshot runtime type guards.
2. **Wave 1 — Tool-Level Capability Filtering** (Plan45 W3 deferred, Master priority): runtime enforcement of `PluginCapabilities.allowedTools` via a factory-wrapper proxy in the runner. Zero Core modifications (C46-1).
3. **Wave 2 — K-3 SDK Framework Hook** (Plan45 CC-2 completion): `PluginSnapshot` type + `PluginHooks.onCheckpoint`/`onRestore`. Rule #45 re-freeze authorized (R3 D10-Q32, 24/24 unanimous). Migrates SafetyGate (L3), StateTracker, ShewhartChart (L1). Adds CheckpointManager orchestrator in the runner.

All logic is plugin-scoped or runner-local. **Zero Core modifications (Tenet #7 / C46-1)**. PluginHooks interface is **RE-FROZEN** immediately after W2 delivery (Rule #45).

**DEV-1b Status**: CLOSED after Plan46 delivery (CC-1 completed in Plan45; CC-2 K-3 completion now landed in Plan46 W2).

---

## Wave Structure (Master Ratified 2026-04-16)

| Wave | Content | Target LOC | Actual LOC | Priority |
|------|---------|:----------:|:----------:|:--------:|
| W0 | SEC-001 nonce + SEC-004 vector clock + SEC-R1/R2 transport + StateTracker validation | ~50–100 | ~168 | MUST |
| W1 | Tool-Level Capability Filtering (runtime enforcement) | ~100–200 | ~117 | MUST |
| W2 | K-3 SDK framework hook + L3/StateTracker/ShewhartChart migration | ~180–290 | ~248 | MUST |
| **Total** | | **~330–590** | **~534** | |

Plan46 LOC sits at the upper end of the S-1 range. W0 overshoot stems from StateTracker validation requiring per-entry tuple guards rather than the simpler whole-object check SafetyGate uses.

---

## Hard Constraints (Delivered)

| ID | Constraint | Verification |
|----|-----------|--------------|
| C46-1 | Zero Core modifications | No `packages/core/src` or `packages/shared/src` source touched. |
| C46-2 | PluginHooks re-frozen after W2 | JSDoc `@since v0.46.0-alpha` + re-freeze note on `PluginSnapshot` and both new methods. |
| C46-3 | SEC-003 validation preserved in ShewhartChart migration | `ShewhartChart.onRestore` delegates to existing `deserialize()`; corrupt-JSON test exercises the SEC-003 path. |
| C46-4 | Default permissive `allowedTools` | `wrapPluginWithToolFilter` returns plugin unchanged when `allowedTools` is undefined or empty. |
| C46-5 | HYPOTHESIS constants labeled | `MAX_VECTOR_CLOCK_AGENTS` labeled `HYPOTHESIS (Rule #59)` with calibration note. |
| C46-6 | Rule #68 two-path verification for W1 config | Integration test `Path A (manifest) → Path B (ctx.tools) filtered` asserts both ends. |
| C46-7 | F-8 carry-forward-must if any wave deferred | N/A — all waves delivered. |

---

## Wave 0 — Security Cleanup

### W0-1 SEC-001 Nonce Counter (seed-signature.ts)
- `SeedSignatureServiceImpl` grows an internal `lastNonce: Map<string, number>`.
- New public methods on the impl class (ISeedSignatureService SDK interface remains **FROZEN and untouched**):
  - `verifyNonce(agentId: string, nonce: number): boolean` — enforces `nonce > lastSeen[agentId]`; advances the counter on success; rejects non-finite nonces.
  - `getLastNonce(agentId): number | undefined` — diagnostic accessor.
- `clear()` now resets both the HMAC secret (SEC-002) and the nonce map (SEC-001).
- **Opt-in semantics**: callers that never invoke `verifyNonce()` keep the pre-Plan46 behaviour — backward-compatible.

### W0-2 SEC-004 Vector Clock Pruning (bija-store.ts)
- Exports `MAX_VECTOR_CLOCK_AGENTS = 100` (HYPOTHESIS, Rule #59).
- `mergeVectorClock()` now checks the key count after the element-wise max merge; if it exceeds the cap, the lowest-counter non-self peers are dropped.
- **Own `agentId` always preserved** — even when its counter is the lowest in the clock.

### W0-3 SEC-R1 Transport HTTP JSON Validation (transport-http)
- Module-level `isValidInputMessage()` type guard: object shape, `text: string`, optional `requestId`/`sessionId: string`.
- POST `/api/input` validates before destructuring; returns HTTP 400 on schema failure (fail-closed).

### W0-4 SEC-R2 Transport WebSocket JSON Validation (transport-websocket)
- Exported module-level `isValidWsMessage()` type guard: object, `type: string (non-empty)`, optional `sessionId: string`, opaque `payload`.
- `ws.on("message")` handler validates before destructuring; sends `{ type: "error", error: "Invalid message" }` on schema failure.
- Payload `text` narrowed safely: `typeof payload.text === "string" ? payload.text : ""`.

### W0-5 StateTracker.fromSnapshot Validation (state-tracker.ts)
- `StateTrackerSnapshot` gains `schemaVersion: 1` (aligned with `SafetyGateSnapshot`).
- `fromSnapshot()` rewritten with runtime guards mirroring `SafetyGate.fromSnapshot()`:
  - Null / non-object / array rejection.
  - Unknown `schemaVersion` throws (migration guard).
  - Per-field array shape checks; per-entry tuple + element type checks.
  - Malformed rows are skipped leniently so a partially-corrupt snapshot still yields a usable tracker (consistent with `ShewhartChart.deserialize` lenient-skip pattern).

---

## Wave 1 — Tool-Level Capability Filtering

### Design: Factory Wrapper (not Core extension)

C46-1 prohibits Core changes. Existing `allowedProviders` enforcement (sandbox-manager.ts:474–491) lives in Core, but for tools we chose a **runner-side factory-wrapper** to keep Core pristine:

```
resolvePlugins() → wrapPluginWithToolFilter() → capturePluginHooks() → core.loadPlugin()
```

### Components

| File | Role |
|------|------|
| `apps/runner/src/utils/tool-filter-proxy.ts` (NEW) | `createToolFilterProxy` + `wrapPluginWithToolFilter` + `capturePluginHooks` (W2 hook-capture helper co-located) |
| `apps/runner/src/commands/start.ts` | Insert the wrapper chain between `resolvePlugins()` and `core.loadPlugin()` |

### Key semantics

- `allowedTools` undefined **or** empty → plugin returned unchanged (C46-4).
- `allowedTools` non-empty → factory sees a cloned `ctx` whose `tools` is the filtered proxy.
- Proxy behaviour:
  - `list()` returns only tools whose `id` is in the allowed set.
  - `get(id)` returns `undefined` for blocked ids and invokes `onDenied(event)` where event is:
    ```json
    {
      "type": "audit:capability_denied",
      "plugin": "<plugin name>",
      "tool": "<requested id>",
      "allowedTools": ["<list copy>"],
      "timestamp": "<ISO 8601>"
    }
    ```
- Runner wires `onDenied` to `core.bus.emit({ type, timestamp, payload: event })`.

### Rule #68 Two-Path Verification

Integration test asserts the full path in a single case:

1. **Path A (manifest)**: `plugin.manifest.capabilities.allowedTools = ["fs:readFile"]`.
2. **Path B (ctx.tools runtime)**: after wrapping, `ctx.tools.list()` returns only `fs:readFile`; `ctx.tools.get("net:fetch")` returns `undefined` and an `audit:capability_denied` event fires.

---

## Wave 2 — K-3 SDK Framework Hook

### Rule #45 Re-freeze Authorization

- **R3 Vote**: D10-Q32 — 24/24 unanimous (see `research_team_suggestion/cycle03-10/deliver/R3_decision_log.md`).
- **N=0**: Pre-Plan46 grep for `onCheckpoint|onRestore` across both monorepos returned zero hits — purely additive, BCT-trivially safe.
- **Re-freeze**: Immediate upon Plan46 delivery. Any future additions require a fresh unfreeze cycle.

### SDK Surface

```ts
// packages/sdk/src/types/plugin.ts — new
export interface PluginSnapshot {
  readonly pluginName: string;
  readonly schemaVersion: number;
  readonly state: Readonly<Record<string, unknown>>;
  readonly timestamp: number;
}

// Added to existing PluginHooks
onCheckpoint?: () => PluginSnapshot | null;   // never throws; null = skip
onRestore?: (snapshot: PluginSnapshot) => void;  // may throw; framework catches
```

### Component Migrations

| Component | Plugin name (wire identity) | Delegation |
|-----------|:---------------------------:|------------|
| `SafetyGate` (L3) | `spc-safety-gate` | `onCheckpoint` wraps `serialize()`; `onRestore` validates pluginName + schemaVersion then delegates to `fromSnapshot()`. |
| `StateTracker` | `state-tracker` | `onCheckpoint` wraps `serialize()`; `onRestore` delegates to `fromSnapshot()` then copies the restored state into `this` (mutation semantics). |
| `ShewhartChart` (L1) | `shewhart-chart` | `onCheckpoint` wraps `serialize()` (JSON string) under `state.windows`; `onRestore` delegates to `deserialize()` — **C46-3: SEC-003 validation preserved end-to-end**. |

### CheckpointManager

New file `apps/runner/src/utils/checkpoint-manager.ts`:

```ts
createCheckpointManager(plugins: Map<string, PluginHooks>): {
  checkpoint(): Map<string, PluginSnapshot>;
  restore(snapshots: Map<string, PluginSnapshot>): void;
}
```

- `checkpoint()` iterates plugins, calls `onCheckpoint()` on each (wrapped in try/catch for defense), skips plugins that return `null` or throw.
- `restore()` looks up each snapshot by plugin name and calls `onRestore()` (try/catch → fresh-state fallback on failure).

### Hook-Capture Wrapper (CONDITIONAL-2 resolution)

`core.loadPlugin()` returns `Promise<void>` and does not surface the hooks. The runner needs live hook references to build a CheckpointManager, so `capturePluginHooks()` wraps each plugin's factory and records any plugin exposing `onCheckpoint`/`onRestore` into a name-keyed `Map`. Same pattern as W1 tool-filter wrapper — zero Core changes.

The wrapping chain in `apps/runner/src/commands/start.ts`:

```
resolvePlugins() → wrapPluginWithToolFilter() → capturePluginHooks() → core.loadPlugin()
```

After the loop, `createCheckpointManager(hookMap)` produces the manager. Plan46 constructs it but does not yet expose `checkpoint()`/`restore()` via a runner command — that wiring is the natural Plan47 follow-up once a concrete persistence consumer (daemon session resume, CLI resume flag) is scheduled.

---

## ENG-FAB v1.5 Compliance

| Item | Applicability | Disposition |
|------|:------------:|-------------|
| A-0 G/H | New configs + HYPOTHESIS constants | `MAX_VECTOR_CLOCK_AGENTS` labeled; no other tuning constants introduced. |
| F-7 Rule #68 | W1 tool-filtering config path | Verified via integration test (Path A manifest → Path B ctx.tools filtered). |
| F-8 carry-forward-must | N/A | All 3 waves delivered — no deferrals. |

Rule #47 (even Plan → full traceability + full test suite): satisfied. 2580 passed / 3 skipped / 1 pre-existing flaky timeout.

---

## Test Plan (Delivered)

### W0 Acceptance Criteria

| AC | Evidence |
|----|----------|
| AC-W0-1 (replay) | `seed-signature.test.ts` → `rejects duplicate nonce (replay attack)` + reorder test |
| AC-W0-2 (vector clock cap) | `bija-store.test.ts` → `mergeVectorClock() prunes when exceeding MAX_VECTOR_CLOCK_AGENTS` |
| AC-W0-3 (HTTP validation) | `transport-http/src/index.test.ts` → `HTTP POST /api/input — SEC-R1 input validation` (4 tests) |
| AC-W0-4 (WS validation) | `transport-websocket/__tests__/input-validation.test.ts` (5 tests) |
| AC-W0-5 (StateTracker) | `dynamic-arbiter.test.ts` → `fromSnapshot() rejects…` + schemaVersion round-trip |

### W1 Acceptance Criteria

`apps/runner/__tests__/integration/tool-capability-filtering.test.ts` — 10 tests covering filtered `list`, allowed/blocked `get`, audit event shape + ISO timestamp, default-permissive (undefined / empty), factory return preservation, missing `ctx.tools` safe path, Rule #68 Path A → Path B verification.

### W2 Acceptance Criteria

| AC | Evidence |
|----|----------|
| AC-W2-1/2 (types) | Build passes with `PluginSnapshot` + `onCheckpoint`/`onRestore` in SDK |
| AC-W2-3 (SafetyGate) | `safety-gate.test.ts` → `Plan46 W2 onCheckpoint/onRestore` suite (3 tests) |
| AC-W2-4 (StateTracker) | `dynamic-arbiter.test.ts` → onCheckpoint/onRestore round-trip + pluginName mismatch |
| AC-W2-5 (ShewhartChart + SEC-003) | `shewhart-deserialize.test.ts` → `onRestore preserves SEC-003: corrupt JSON rejected` |
| AC-W2-6/7 (CheckpointManager + integration) | `apps/runner/__tests__/integration/checkpoint-manager.test.ts` (9 tests) |
| AC-W2-8 (re-freeze) | JSDoc on `PluginSnapshot` + both new methods records re-freeze per Rule #45 |

### Test Totals

- Workspace: **239 files / 2580 passed / 3 skipped / 1 pre-existing flaky** (plugin-install `--all` timeout, NOT Plan46).
- Delta vs baseline: +14 files, +155 tests.

---

## Known Issues / Follow-ups

1. Flaky `plugin-install.test.ts --all` — pre-existing environment-sensitive timeout; not Plan46. Recommend hotfix to raise timeout or gate behind `INTEG_SLOW`.
2. Existing purity violations — two pre-existing comment/string references in `packages/core/` (Plan33 era). Unchanged by Plan46.
3. CheckpointManager wire-in is latent — `checkpoint()`/`restore()` not yet surfaced via a runner command. Plan47 follow-up.

---

## Cross-References

- Architecture Spec: `share/engineering_delivery/cycle03-10_plan46/architecture_spec.md`
- Delivery Report: `share/engineering_delivery/cycle03-10_plan46/delivery_report.md`
- Architecture Note: `share/openstarry_doc/Architecture_Documentation/73_Plan46_Tool_Filtering_And_Checkpoint_Hook.md`
- Research recommendations: `share/research_team_suggestion/cycle03-10/deliver/O7_plan46_engineering_spec.md`
- R3 Re-freeze decision: `share/research_team_suggestion/cycle03-10/deliver/R3_decision_log.md` (D10-Q32)

*Plan46 Implementation Plan — Cycle 20260416_cycle03-10, 2026-04-17.*
