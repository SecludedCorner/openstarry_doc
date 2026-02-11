# 00. Project Roadmap & Milestones

This document defines the implementation path for OpenStarry, from core prototype to a complete ecosystem. This roadmap is designed to ensure orderly evolution and provide clear guidance for open-source contributors.

---

## 🏷️ Version Milestones Overview

| Version | Content | Status |
|------|------|------|
| v0.1.0-alpha | MVP Foundation (Plan01-04) | ✅ Completed |
| v0.2.0-beta | Multi-channel Transport — WebSocket + HTTP (Plan05) | ✅ Completed |
| v0.2.1-beta | Session Isolation + SSE + TraceId (Plan05.1, 05.2, 05.5) | 📋 Planned |
| v0.2.2-beta | E2E Testing Scaffolding (Plan05.4) | 📋 Planned |
| v0.3.0-beta | MCP Protocol Integration (Plan06 + DevTools) | 📋 Planned |
| v0.3.5-beta | Management Zone Infrastructure | 📋 Planned |
| v0.4.0-beta | Persistence & State Restoration (Plan07) | 📋 Planned |
| v0.5.0-beta | TUI Dashboard + Plugin Coordination Layer (Plan08-09) | 📋 Planned |

---

## 📅 Phase 1: Genesis
**Goal:** Establish the Monorepo physical foundation and shared conventions.
**Status:** 🟡 In Progress — 1.3 CI/CD not yet established.

- [x] **1.1 Monorepo Initialization**
    - [x] Create `apps/` and `packages/` directory structures.
    - [x] Configure `pnpm-workspace.yaml` and root `package.json`.
    - [x] Unify TypeScript, ESLint, and Prettier conventions.
- [x] **1.2 Core SDK & Type Definitions**
    - [x] Establish `packages/sdk` to define full **Five Aggregates Plugin Interfaces**.
        *   **Form (IUI)**: Interface and manifestation (Form/UI).
        *   **Sensation (IListener)**: Sensory monitoring (Perception/Input).
        *   **Perception (IProvider)**: Cognitive generation (Cognition/LLM).
        *   **Volition (ITool)**: Tool execution (Action/Will).
        *   **Consciousness (IGuide)**: Soul and persona (Consciousness/Persona).
    - [x] Establish `packages/shared` for global error handling and logging modules.
- [ ] **1.3 Engineering & Testing Infrastructure**
    - [x] Configure unit testing framework (Vitest). *(Plan02 Phase C — 56 tests passed)*
    - [x] Purity check script (`pnpm test:purity`). *(Plan03 Phase A4)*
    - [ ] Establish CI/CD workflows (GitHub Actions).

## 🧠 Phase 2: The Conscious Kernel
**Goal:** Implement a "Headless," event-driven `Agent Core`.
**Status:** 🟢 Completed

- [x] **2.1 The Execution Loop**
    - [x] Implement `packages/core/execution` (tick mechanism based on event queue).
    - [x] Event-driven refactor of the ExecutionLoop: pulling events from EventQueue, state machine includes WAITING_FOR_EVENT / SAFETY_LOCKOUT. *(Plan02 Phase A)*
    - [x] Standardize `InputEvent` payload (source, inputType, data, replyTo). *(Plan02 Phase A4)*
- [x] **2.2 State & Memory Management**
    - [x] Implement `packages/core/state` (supports snapshots and persistence interfaces).
    - [x] Implement `packages/core/memory` (pluggable context strategies, e.g., sliding window).
- [x] **2.3 Safety & Correction**
    - [x] Implement Token budget limits and infinite loop detection (Circuit Breakers). *(Plan02 Phase B — SafetyMonitor)*
    - [x] Implement the "Error as Pain" mechanism, transforming runtime errors into Context feedback to trigger Agent self-correction. *(Plan02 Phase B — frustration counter + repetitive call detection + error cascade breaker)*

## 🦾 Phase 3: Body and Senses
**Goal:** Build complete Five Aggregates plugin infrastructure and standard library; establish security boundaries.
**Status:** 🟡 In Progress — Core plugins completed; advanced features pending.

- [ ] **3.1 Loading Protocol**
    - [x] Implement `PluginLoader` supporting **Factory Pattern** initialization. *(Plan01)*
    - [ ] Implement `IPluginContext`: includes Logger, Config, and **Cross-plugin Service Injection (dependencies field)**.
    - [ ] **Implement Dependency Wiring**: Automatic OODA loop connection during loading based on Document `20`.
    - [x] **Implement CommandRegistry**: Support for CLI `run-tool` and Chat `/slash` command mapping and execution. *(Plan01)*
    - [x] **Implement Dynamic Loading**: CLI supports `agent.json` `plugins[].path` field, dynamically loading third-party plugins via `import()`. *(Plan02 Phase A3)*
    - [x] Implement runtime validation for `agent.json` (Zod schema validation). *(Plan03 completed)*
- [ ] **3.2 Coordination Layer & Registration**
    - [x] **Dual-Path Scanning**: Simultaneously scan system and project-private plugin directories.
    - [ ] **Sync Command**: Implement `openstarry plugin sync` to sync official **`openstarry_plugin`** repository content to the system directory.
    - [x] Implement **Plugin Registry Service** in the Daemon with in-memory indexing.
- [ ] **3.3 Standard Aggregate Plugin Library**
    > **Development Location:** `openstarry_plugin` repository (Ecosystem Repo).
    - [x] `@openstarry-plugin/standard-function-stdio`: Aggregates Sensation (Stdio Listener) + Form (CLI Body) + Consciousness (Default Guide). *(Plan01)*
    - [x] `@openstarry-plugin/provider-gemini-oauth`: Aggregates Perception (Gemini Provider) with PKCE + OAuth 2.0. *(Plan01)*
    - [x] `@openstarry-plugin/standard-function-fs`: Aggregates Volition (FS Tools) + Consciousness (Path Scoping Policy). *(Plan01)*
    - [ ] `@openstarry-plugin/guide-mcp`: Aggregates Consciousness (MCP Guide) for standardized communication.
    - [ ] `@openstarry-plugin/guide-pain-mechanism`: Implements anthropomorphic pain interpretation logic.
    - [x] `@openstarry-plugin/standard-function-skill`: Aggregates Consciousness (Skill Guide). **Core Dependency**: Parses Markdown skill files, foundational for workflows and complex Agents. *(Plan03 Phase B1)*
- [ ] **3.4 Developer Experience (DX)**
    - [ ] Implement `openstarry create-plugin` scaffolding.
    - [ ] Provide `MockHost` test environment for independent plugin development and verification.

> **🔒 Security Note:**
> While Phase 3 `fs` tools do not run in Docker, they must use the path normalization component from `packages/shared` to strictly limit their operational scope. This allows Agents to perform necessary operations while protecting critical system directories.

## 👶 Phase 4: The First Breath
**Goal:** Implement the first standalone CLI Agent and verify end-to-end workflows.
**Status:** 🟡 In Progress — Runner operational; E2E LLM verification pending manual testing after OAuth login.

- [x] **4.1 Local Runner**
    - [x] Implement `apps/runner` bootstrap program (Pure Bootstrap Runner) supporting `agent.json`. *(Plan01)*
    - [x] Support dynamic plugin resolution (path / node_modules tiered strategy; BUILTIN_FACTORIES removed). *(Plan02 Phase A3)*
- [ ] **4.2 Integration Verification**
    - [ ] Achieve full "perceive -> think -> tool call -> correct -> respond" closed loop. *(Pending manual OAuth test)*

> **🏆 Milestone: v0.1 Alpha (MVP)**
> *   ✅ Runs single Agent in CLI.
> *   ✅ Gemini as Provider (PKCE + OAuth).
> *   ✅ Local filesystem operations (`fs` tool).
> *   ✅ Basic memory (5 dialogue turns).
> *   ✅ Safety circuit breakers (Token budget, loop caps, error cascade, frustration counter).
> *   ✅ Structured JSON logs (LOG_FORMAT=json, LOG_LEVEL filtering).
> *   ✅ agent.json Zod validation.
> *   ✅ Tool call Timeout (Promise.race).
> *   ✅ TraceID mechanism.
> *   ✅ Markdown skill loading.
> *   ✅ guideFile external reference.
> *   ✅ Purity check script (`pnpm test:purity`).
> *   ✅ 56 unit tests passed.
> *   ⬜ E2E LLM call verification (manual test pending).

---

> **🚀 Agent Core Complete. Proceeding to Agent OS and Ecosystem phases.**

---

## 📡 Phase 4.5: Multi-Channel Transport
**Goal:** Validate scalability of the IUI/IListener architecture by implementing non-stdio channels.
**Status:** 🟢 Completed (v0.2.0-beta)

- [x] **4.5.1 WebSocket Transport Plugin**
    - [x] `@openstarry-plugin/transport-websocket`: WebSocket Listener + UI.
    - [x] Support for multi-client connections and targeted replies (`replyTo`).
- [x] **4.5.2 HTTP Webhook Transport Plugin**
    - [x] `@openstarry-plugin/transport-http`: HTTP Listener + UI.
    - [x] POST /api/input, GET /api/status, GET /api/response.
- [x] **4.5.3 Multi-UI Simultaneous Output**
    - [x] TransportBridge broadcast verification passed.
    - [x] stdio + WebSocket receive events simultaneously.

> **🏆 Milestone: v0.2.0 Beta (Multi-Channel)**
> *   ✅ WebSocket transport implemented.
> *   ✅ HTTP Webhook transport implemented.
> *   ✅ Multi-UI simultaneous output verified.

---

## 🔐 Phase 4.6: Implementation Cycle 1 (v0.2.1-beta)
**Goal:** Address multi-user privacy, optimize transport efficiency, and establish full-link tracing.
**Status:** 📋 Planned

### Plan05.1: Session Isolation & Routing 🔴 Highest Priority
- [ ] Listener tags `sessionId`.
- [ ] Core propagates `sessionId` to output events.
- [ ] UI filters pushes by `sessionId`.
- [ ] Acceptance: WebSocket User A cannot see User B's dialogue.

### Plan05.2: HTTP SSE Support 🟢 High Priority
- [ ] Add `GET /api/stream` endpoint.
- [ ] Replace polling with Server-Sent Events.
- [ ] Support EventSource clients.

### Plan05.5: Unified TraceId 🔵 Low Priority (Pre-implemented)
- [ ] Core provides unified `TraceId` generator.
- [ ] Automatically attached to all events and logs.

## 🧪 Phase 4.7: Implementation Cycle 2 (v0.2.2-beta)
**Goal:** Establish automated verification and testing scaffolding.
**Status:** 📋 Planned

### Plan05.4: E2E Testing Scaffolding 🟡 Medium Priority
- [ ] `createTestAgent()` utility function.
- [ ] Support for MockLLM plugin.
- [ ] Lower community contribution barriers.

> **⚠️ Note: Plan05.3 DevTools deferred to after Plan06 (MCP).**

---

## 🏰 Phase 5: The Orchestrator Daemon — Plan07
**Goal:** Achieve process-level management and persistence.
**Status:** 📋 Planned (v0.4.0-beta)

- [ ] **5.1 Daemon Core**
    - [ ] Establish `apps/daemon` to manage Agent lifecycles.
    - [ ] Implement scheduling logic in the **Management Zone**.
- [ ] **5.2 API Gateway & Persistence**
    - [ ] HTTP/gRPC API for remote management.
    - [ ] Database integration (SQLite/LevelDB) for Agent long-term state.
- [ ] **5.3 Security & Governance**
    - [ ] Plugin signing and verification.
    - [ ] Runtime sandbox strategies (Node.js vm or WASM).
    - [ ] **Environment isolation & Quota Management**.
- [ ] **5.4 Observability & HAL**
    - [x] Structured JSON logging. *(Plan02)*
    - [ ] OpenTelemetry/Event Tracing for cross-agent tasks.
    - [ ] Log collection for Dashboard visualization.
    - [ ] **Hardware Abstraction Layer (HAL) sensory data streams**.

## 🕸️ Phase 6: Fractal Society — Plan06
**Goal:** Inter-agent collaboration and MCP protocol.
**Status:** 📋 Planned (v0.3.0-beta)
**Pre-condition:** Plan05.1 Session Isolation complete.

- [ ] **6.1 MCP Integration**
    - [ ] Implement `packages/mcp-protocol` for standardized interconnection.
    - [ ] Support concurrent external client connections.
- [ ] **6.2 DevTools Debugging Interface**
    - [ ] Static HTML interface `openstarry-devtools`.
    - [ ] Chat bubbles, State/Context view, Event filter.
- [ ] **6.3 Workflow Engine**
    - [ ] `WorkflowEngineTool` for YAML-defined task orchestration.
    - [ ] Dynamic spawning/destruction of Ephemeral Agents.
- [ ] **6.4 Orchestration**
    - [ ] Task decomposition and distribution.
    - [ ] **Causality Chain based event scheduling**.

## 📦 Phase 7: Ecosystem and UI
**Goal:** Web GUI management and deployment tools.
**Status:** 🔴 Pending

- [ ] **7.1 Dashboard & Interface**
    - [ ] `apps/dashboard` (React/Next.js).
    - [ ] **State Projection dashboards**.
- [ ] **7.2 Agent Design Service**
    - [ ] Agent store and template downloads ("Directory as Protocol" deployment).

## 🖥️ Phase 8: CLI & TUI Evolution — Plan08 / Plan09
**Goal:** OS-level terminal experience.
**Status:** 📋 Planned (v0.5.0-beta)

- [ ] **8.1 Runtime Dashboard**
    - [ ] TUI interface for `openstarry` (Ink or Blessed).
    - [ ] Daemon monitoring, resource charts, log streaming.
- [ ] **8.2 Interactive Designer**
    - [ ] Context-aware menus for `openstarry design`.
    - [ ] Guided Wizard for aggregate configuration.
- [ ] **8.3 Seamless Attach**
    - [ ] Socket connection logic for `openstarry attach`.
    - [ ] Jump from Dashboard to Agent dialogue (via 'a' key).

---

## 📊 Plan Dependency Graph

```
                    ┌─────────────────────────────────────────┐
                    │           Completed (v0.2.0-beta)        │
                    │  Plan01 → Plan02 → Plan03 → Plan04 → Plan05  │
                    └─────────────────────┬───────────────────┘
                                          │
              ┌───────────────────────────┼────────────────────────────────┐
              │                           │                                │
              ▼                           ▼                                ▼
        ┌──────────┐              ┌──────────────┐                  ┌──────────┐
        │ Cycle 1  │              │ Cycle 2      │                  │ Plan06   │
        │v0.2.1-beta│              │v0.2.2-beta   │                  │ MCP Int. │
        │(Sess+SSE) │              │(E2E Tests)   │                  │(+DevTools)│
        └────┬─────┘              └──────────────┘                  └────┬─────┘
             │                                                            │
             │ (Pre-condition)                                            │
             ▼                                                            ▼
        ┌──────────┐                                                ┌──────────┐
        │ Plan06   │────────────────────────────────────────────────▶│ Plan07   │
        │ MCP Int. │                                                │ Persistence│
        └──────────┘                                                └──────────┘
```

---

## 🟡 Issues to Discuss

The following issues require further discussion during development:

| Issue | Status | Related Plan |
|------|------|----------|
| WebSocket Auth Mechanism | 🟡 Pending | Plan05.1 |
| Multi-instance Load Balancing | 🟡 Pending | Plan07 |
| Session Expiry Policy | 🟡 Pending | Plan05.1 |
| SSE Reconnection | 🟡 Pending | Plan05.2 |
