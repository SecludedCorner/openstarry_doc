# OpenStarry Implementation Plan 02 — Architectural Completion & Quality Improvement

> **Status**: ✅ Completed (2026-02-05)

## Background

Plan01 completed the basic skeleton of MVP v0.1 Alpha (compilable, launchable, and capable of receiving instructions).
However, through verification reports from the testing team (`test/20260204/`) and comparisons with `openstarry_doc` design documents,
several **gaps between implementation and design** have been identified.

This plan defines the scope of improvements for Plan02, aiming to **fill the missing components of Phase 2.3 / Phase 3.1 and fulfill the requirements of the design documents**.

---

## 1. Issues Identified in Test Reports

| # | Source | Issue | Severity |
|---|------|------|--------|
| G1 | Developer_Handoff_Report / Engineering_Implementation_Gaps | CLI plugin loading is hardcoded (`switch-case`), unable to dynamically `import()` third-party plugins. | 🔴 High |
| G2 | Engineering_Implementation_Gaps | `ExecutionLoop.run(string)` accepts strings and is not event-driven; inconsistent with the document stating "the event queue is the sole input source for the core." | 🔴 High |
| G3 | QA_MultiInput_Verification_Report | Asynchronous competition risks during tool execution—potential state conflicts between new input and the current loop. | 🟡 Medium |
| G4 | Developer_Handoff_Report / QA_Report | Lack of concurrent unit tests; stability of `EventQueue` FIFO not verified. | 🟡 Medium |

---

## 2. Design Document Comparison — Unimplemented Features

| # | Source Document | Missing Feature | Belonging to Phase |
|---|----------|----------|-----------|
| D1 | `02_Headless_Agent_Core.md` | ExecutionLoop should be a `WAITING_FOR_EVENT` state machine pulling events from EventQueue, rather than receiving strings directly. | 2.1 |
| D2 | `12_Capabilities_Injection_Mechanism.md` | PluginLoader should support dynamic `import()` loading based on `agent.json` path resolution. | 3.1 |
| D3 | `07_Safety_Circuit_Breakers.md` | Resource-level breakers: Token budget limits, Loop Cap. | 2.3 |
| D4 | `07_Safety_Circuit_Breakers.md` | Behavioral breakers: Repetitive tool call detection, error cascade breakers. | 2.3 |
| D5 | `12_Error_Handling_and_Self_Correction.md` | "Error as Pain" mechanism: Tool error normalization + frustration counter + mandatory help request. | 2.3 |
| D6 | `09_Observability_and_Tracing.md` | Structured JSON logging (including agent_id, trace_id, module). | 3.1 |
| D7 | `15_Testing_Strategy_and_Infrastructure.md` | Unit test infrastructure (Vitest), microkernel purity checks, MockHost. | 1.3 |
| D8 | `01_Execution_Loop.md` | Output routing: Returning results to the correct channel based on the event `source`. | 2.1 |

---

## 3. Implementation Steps for Plan02

### Phase A: Event-Driven Transformation (Corresponding to G1, G2, D1, D2, D8)

**Goal**: Transition ExecutionLoop away from directly receiving strings to listening to the EventQueue; support dynamic plugin loading in the CLI.

#### A1. Event-Driven Refactoring of ExecutionLoop
- Add `start()` method to `ExecutionLoop`: continuously retrieves events via `EventQueue.pull()`.
- Standardize event payload: `{ source: string, type: string, data: unknown }`.
- Downgrade `run(userInput)` to an internal method, invoked only by event triggers.
- Implement `isProcessing` lock to prevent simultaneous multi-event processing (resolving G3 competition issues).
- Output Routing: Determine the reply channel based on `event.source`.

#### A2. AgentCore Integration
- Change `AgentCore.processInput()` to push to EventQueue instead of calling the loop directly.
- Listener plugins push events to EventQueue (instead of calling AgentCore directly).
- Retain the fast path for slash commands (bypassing the LLM cycle).

#### A3. Dynamic Plugin Loading in CLI
- Add `DynamicPluginLoader`: uses `import()` to load based on `plugins[].path` in `agent.json`.
- Retain fast path for builtin plugins (switch-case fallback when path is absent).
- Support `node_modules` package resolution (`import(pluginName)`).

#### A4. Event Payload Standardization
- Define `InputEvent` type:
  ```typescript
  interface InputEvent {
    source: string;      // "cli", "webhook", "mcp"
    type: string;        // "user_input", "system_command"
    data: unknown;
    replyTo?: string;    // Reply channel identifier
  }
  ```

---

### Phase B: Safety Breakers & Self-Correction (Corresponding to D3, D4, D5)

**Goal**: Implement the `SafetyMonitor` module to prevent runaway states and infinite loops.

#### B1. SafetyMonitor Module
- **Location**: `packages/core/src/security/safety-monitor.ts`.
- **Responsibilities**:
  - `beforeLLMCall()`: Token budget check.
  - `afterToolExecution()`: Repetitive call detection + error rate calculation.
  - `onLoopTick()`: Loop count limit check.

#### B2. Resource-Level Breakers
- `MAX_LOOP_TICKS`: Maximum iterations per task (default 50, configurable in `policy`).
- `MAX_TOKEN_USAGE`: Cumulative Token consumption limit (dependent on `usage` returned by Provider).
- Transition state to `SAFETY_LOCKOUT` upon trigger.

#### B3. Behavioral Breakers
- `ToolCallFingerprint` history queue: hash(toolName + args).
- N consecutive identical failure fingerprints → Forcibly inject system prompt: "STOP and analyze why."
- Sliding window error rate (8 failures in last 10 operations → `EMERGENCY_HALT`).

#### B4. Frustration Counter & Self-Correction
- Consecutive failure counter.
- Exceeding threshold (default 5) → Forcibly inject System Prompt: "ask the user for help."
- Standardize tool errors into `{ code, message, suggestion }` structure.

---

### Phase C: Testing Infrastructure (Corresponding to G4, D7)

**Goal**: Establish a Vitest testing environment with coverage for core logic.

#### C1. Testing Framework Configuration
- Add Vitest configuration to the root directory.
- Add `test` scripts to each package.

#### C2. Core Unit Tests
- `EventBus.test.ts`: Multi-handler, wildcard, error isolation.
- `EventQueue.test.ts`: FIFO order, concurrent push/pull, stress testing.
- `StateManager.test.ts`: add/clear/snapshot/restore.
- `ContextManager.test.ts`: Sliding window truncation logic.
- `SecurityLayer.test.ts`: Path validation correctness.
- `SafetyMonitor.test.ts`: Trigger conditions for all breaker levels.

#### C3. Integration Tests
- `MultiSourceEvent.test.ts`: Simulating simultaneous injection from multiple sources.
- `ExecutionLoop.test.ts`: Mock provider, verifying the full loop workflow.
- `PluginLoader.test.ts`: Verification of dynamic loading.

#### C4. Microkernel Purity Checks
- Add `dependency-cruiser` or ESLint rules.
- Ensure `packages/core` does not import from `plugins/*` or `apps/*`.

---

### Phase D: Observability Improvements (Corresponding to D6)

**Goal**: Implement structured JSON logging.

#### D1. Logger Upgrade
- Update `packages/shared/src/logger` to output in JSON format.
- Add `agent_id` and `trace_id` fields.
- Support log level filtering (via `LOG_LEVEL` environment variable).

---

## 4. Implementation Priorities

```
Phase A (Event-Driven Transformation)     ← 🔴 Highest Priority (addresses core issues in test reports)
  └→ A1 ExecutionLoop Eventization
  └→ A2 AgentCore Integration
  └→ A3 Dynamic Plugin Loading
  └→ A4 Event Payload Standardization

Phase B (Safety Breakers)                 ← 🔴 High (missing items from Phase 2.3)
  └→ B1 SafetyMonitor
  └→ B2 Resource-Level Breakers
  └→ B3 Behavioral Breakers
  └→ B4 Frustration Counter

Phase C (Testing Infrastructure)          ← 🟡 Medium (missing items from Phase 1.3)
  └→ C1 Vitest Configuration
  └→ C2 Core Unit Tests
  └→ C3 Integration Tests
  └→ C4 Purity Checks

Phase D (Observability)                   ← 🟢 Low (can proceed in parallel with other phases)
  └→ D1 Logger Upgrade
```

---

## 5. Expected Post-Implementation State

Upon completion of Plan02, the project will achieve:

- **Phase 1.3** ✅ Testing infrastructure + CI rules.
- **Phase 2** ✅ Complete consciousness kernel (including safety breakers, event-driven architecture, self-correction).
- **Phase 3.1** ✅ Dynamic plugin loading + event standardization.
- All design requirements from Phase 1 to 3.1 in `openstarry_doc` implemented.

**Remaining for Phase 3 Completion**:
- 3.2 `openstarry plugin sync` command.
- 3.2 `guide-mcp` + `standard-function-skill` plugins.
- 3.4 `openstarry create-plugin` scaffolding + `MockHost`.

**Remaining for Phase 4 Completion**:
- 4.2 End-to-end LLM call verification (requires manual testing after OAuth login).

---

## 6. Verification Methods

1. `pnpm install && pnpm build` — Successful compilation.
2. `pnpm test` — All unit and integration tests passed.
3. EventQueue stress test passed (1000 events processed in correct FIFO order).
4. Dynamic Loading: CLI successfully loads plugin via custom path in `agent.json`.
5. Safety Breakers: Simulated consecutive failures correctly trigger `SafetyMonitor` halt.
6. Multi-input Simulation: Two event sources injected simultaneously are processed sequentially without conflict.
