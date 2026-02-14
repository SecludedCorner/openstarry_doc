# 00. Project Roadmap and Milestones

This document defines OpenStarry's implementation path from core prototype to full ecosystem. This plan ensures orderly project evolution and provides clear guidance for open-source contributors.

---

## Version Milestones Overview

| Version | Content | Status |
|---------|---------|--------|
| v0.1.0-alpha | MVP Foundation Architecture (Plan01-04) | ✅ Complete |
| v0.2.0-beta | Multi-Channel Transport — WebSocket + HTTP (Plan05) | ✅ Complete |
| v0.2.1-beta | Session Isolation + SSE + Health Check (Plan05.1, 05.2, 05.5-①) | ✅ Complete (Cycle 1, 2026-02-10) |
| v0.2.2-beta | Metrics/Logging + Error Handling (Plan05.5-②③) | ✅ Complete (Cycle 2, 2026-02-11) |
| v0.3.0-beta | MCP Protocol Integration (Plan06) | ✅ Complete (Cycle 3, 2026-02-11) |
| v0.3.5-beta | Management Zone Infrastructure | Planning |
| v0.4.0-beta | Runtime Sandbox MVP (Plan07) | ✅ Complete (Cycle 5, 2026-02-11) |
| v0.4.1-beta | Sandbox Restart + Worker Pool (Plan07.1) | ✅ Complete (Cycle 6, 2026-02-11) |
| v0.4.2-beta | Import Analysis + PKI Verification (Plan07.2) | ✅ Complete (Cycle 7, 2026-02-11) |
| v0.4.3-beta | Audit Logging + Module._load (Plan07.3) | ✅ Complete (Cycle 8, 2026-02-11) |
| v0.5.0-beta | TUI Dashboard MVP (Plan08) | ✅ Complete (Cycle 9, 2026-02-11) |
| v0.6.0-beta | DevTools Plugin + E2E Testing (Plan11) | ✅ Complete (Cycle 12, 2026-02-12) |
| v0.7.0-beta | DevTools Debugging + E2E Framework | ✅ Complete (Cycle 12, 2026-02-12) |
| v0.8.0-beta | Daemon Mode MVP (Plan12) | ✅ Complete (Cycle 13, 2026-02-12) |
| v0.9.0-beta | Seamless Attach (Plan13) | ✅ Complete (Cycle 14, 2026-02-12) |
| v0.10.0-beta | MCP Resources + OAuth 2.1 (Plan06-P3) | ✅ Complete (Cycle 15, 2026-02-12) |
| v0.11.0-beta | Multi-client Attach & Session Management (Plan14) | ✅ Complete (Cycle 16, 2026-02-12) |
| v0.12.0-beta | MCP Sampling & Advanced Protocol Extensions (Plan06-P4) | ✅ Complete (Cycle 17, 2026-02-12) |
| v0.13.0-beta | SDK Context Extensions & Provider Integration (Plan15) | ✅ Complete (Cycle 18, 2026-02-12) |
| v0.14.0-beta | Security Hardening & Quality Polish (Plan16) | ✅ Complete (Cycle 19, 2026-02-12) |
| v0.15.0-beta | Plugin Developer Experience (Plan17) | ✅ Complete (Cycle 20, 2026-02-12) — 970 tests |
| v0.16.0-beta | Plugin Sync & System Plugin Directory (Plan18) | ✅ Complete (Cycle 21, 2026-02-12) — 1009 tests |
| v0.17.0-beta | Plugin Dependency Wiring & Cross-Plugin Services (Plan19) | ✅ Complete (Cycle 22, 2026-02-12) — 1067 tests |
| v0.18.0-beta | Workflow Engine MVP (Plan20) | ✅ Complete (Cycle 23, 2026-02-12) — 1104 tests |
| v0.19.0-beta | Web-based Remote Attach (Plan21) | ✅ Complete (Cycle 24, 2026-02-12) — 1132 tests |
| v0.20.0-beta | Plugin Marketplace MVP (Plan22) | ✅ Complete (Cycle 25, 2026-02-13) — 1330 tests |
| v0.20.1-beta | Windows Cross-Platform Fixes + Attach UX Improvements (Hotfix) | ✅ Complete (Cycle 25-26, 2026-02-13~14) — 1339 tests |

---

## Phase 1: Genesis
**Goal:** Establish the Monorepo physical foundation and shared conventions.
**Status:** In Progress — 1.3 CI/CD not yet established

- [x] **1.1 Monorepo Initialization**
    - [x] Create `apps/` and `packages/` directory structure.
    - [x] Configure `pnpm-workspace.yaml` and root `package.json`.
    - [x] Unify TypeScript, ESLint, and Prettier conventions.
- [x] **1.2 Core SDK and Type Definitions**
    - [x] Create `packages/sdk` defining the complete **Five Aggregates plugin interfaces**.
        *   **Rupa (IUI)**: Interface and presentation (Form/UI).
        *   **Vedana (IListener)**: Sensory listening (Perception/Input).
        *   **Sanna (IProvider)**: Thought generation (Cognition/LLM).
        *   **Sankhara (ITool)**: Tool execution (Action/Will).
        *   **Vinnana (IGuide)**: Soul and persona (Consciousness/Persona).
    - [x] Create `packages/shared` providing global error handling and logging modules.
- [ ] **1.3 Engineering & Testing Infrastructure**
    - [x] Configure unit test framework (Vitest). *(Plan02 Phase C — 56 tests passing)*
    - [x] Purity check script (`pnpm test:purity`). *(Plan03 Phase A4)*
    - [ ] Set up CI/CD pipeline (GitHub Actions).

## Phase 2: The Conscious Kernel
**Goal:** Implement a "Headless", event-driven `Agent Core`.
**Status:** Done

- [x] **2.1 The Execution Loop**
    - [x] Implement `packages/core/execution` (tick mechanism based on event queue).
    - [x] ExecutionLoop event-driven refactor: pull events from EventQueue, state machine includes WAITING_FOR_EVENT / SAFETY_LOCKOUT. *(Plan02 Phase A)*
    - [x] InputEvent payload standardization (source, inputType, data, replyTo). *(Plan02 Phase A4)*
- [x] **2.2 State and Memory Management**
    - [x] Implement `packages/core/state` (supports Snapshot and persistence interface).
    - [x] Implement `packages/core/memory` (pluggable context strategies, e.g., sliding window).
- [x] **2.3 Safety Circuit Breakers & Error Feedback**
    - [x] Implement Token consumption limits and infinite loop detection (Circuit Breakers). *(Plan02 Phase B — SafetyMonitor)*
    - [x] Implement "Error as Pain" mechanism, transforming runtime errors into Context feedback to trigger Agent self-correction. *(Plan02 Phase B — frustration counter + duplicate call detection + error cascade circuit breaker)*

## Phase 3: Body and Senses
**Goal:** Build the complete Five Aggregates plugin infrastructure and standard library, establish security boundaries.
**Status:** In Progress — Core plugins complete, advanced features pending

- [x] **3.1 Core Loading Protocol**
    - [x] Implement `PluginLoader` supporting **Factory Pattern** initialization. *(Plan01)*
    - [x] Implement `IPluginContext`: includes Logger, Config, and **cross-plugin service injection (dependencies field)**. *(Plan19)*
    - [x] **Implement Dependency Wiring:** Based on document 20, automatically wire OODA loop connections during loading. *(Plan19 — topologicalSort, cycle detection, AgentCore.serviceRegistry)*
    - [x] **Implement CommandRegistry:** Support mapping and execution logic for CLI `run-tool` and Chat `/slash` commands. *(Plan01)*
    - [x] **Implement Dynamic Plugin Loading:** CLI supports `agent.json` `plugins[].path` field, dynamically loading third-party plugins via `import()`. *(Plan02 Phase A3)*
    - [x] Implement runtime validator for `agent.json` (Zod schema validation). *(v0.1.5 - Plan03 enhancement complete)*
- [ ] **3.2 Coordination Layer & Registry Mechanism**
    - [x] **Dual-Path Scanning:** Simultaneously scan system and project private plugin directories.
    - [ ] **One-Click Sync:** Implement `openstarry plugin sync` command to sync official **`openstarry_plugin`** repository contents to the system directory.
    - [x] Implement **Plugin Registry Service** in Daemon, building an in-memory index.
- [ ] **3.2 Standard Aggregate Plugin Library**
    > **Development Location:** `openstarry_plugin` repository (Ecosystem Repo)
    - [x] `@openstarry-plugin/standard-function-stdio`: Aggregates Vedana (Stdio Listener) + Rupa (CLI Body) + Vinnana (Default Guide). *(Plan01)*
    - [x] `@openstarry-plugin/provider-gemini-oauth`: Aggregates Sanna (Gemini Provider). PKCE + OAuth 2.0 authentication. *(Plan01)*
    - [x] `@openstarry-plugin/standard-function-fs`: Aggregates Sankhara (FS Tools) + Vinnana (Path Scoping Policy). *(Plan01)*
    - [ ] `@openstarry-plugin/guide-mcp`: Aggregates Vinnana (MCP Guide). Provides standardized communication capability.
    - [ ] `@openstarry-plugin/guide-pain-mechanism`: Implements anthropomorphic pain interpretation logic.
    - [x] `@openstarry-plugin/standard-function-skill`: Aggregates Vinnana (Skill Guide). **Core dependency**: responsible for parsing Markdown skill files, the foundation for workflows and complex Agents. *(Plan03 Phase B1)*
- [ ] **3.4 Plugin Developer Experience (DX)**
    - [ ] Implement `openstarry create-plugin` scaffolding.
    - [ ] Provide `MockHost` test environment supporting independent plugin development and verification.

> **Security Note:**
> Although Phase 3's `fs` tools do not run inside Docker, they must go through the `packages/shared` path normalization component to strictly limit their operational scope. This allows Agents to perform necessary operations like "create/delete folders" while protecting critical system directories.

## Phase 4: The First Breath
**Goal:** Implement the first standalone CLI Agent, validating the end-to-end flow.
**Status:** In Progress — Runner operational, end-to-end LLM validation pending OAuth login testing

- [x] **4.1 Local Runner**
    - [x] Implement `apps/runner` bootstrap program (pure startup Bootstrap Runner), supporting `agent.json` startup. *(Plan01)*
    - [x] Support dynamic plugin resolution (path / node_modules dual-layer strategy, BUILTIN_FACTORIES removed, all plugins loaded dynamically). *(Plan02 Phase A3)*
- [ ] **4.2 Integration Verification**
    - [ ] Achieve the complete closed loop of "Perceive -> Think -> Tool Call -> Correct -> Respond". *(Requires manual testing after OAuth login)*

> **Milestone: v0.1 Alpha (MVP)**
> *   ✅ Can run a single Agent in CLI.
> *   ✅ Uses Gemini as Provider (PKCE + OAuth authentication).
> *   ✅ Can operate the local file system (`fs` tool).
> *   ✅ Has basic memory capability (5 conversation turns).
> *   ✅ Safety circuit breaker mechanisms (Token budget, loop limit, error cascade, frustration counter).
> *   ✅ Structured JSON logging (LOG_FORMAT=json, LOG_LEVEL filtering).
> *   ✅ agent.json Zod runtime validation (immediate error on configuration mistakes).
> *   ✅ Tool call Timeout (Promise.race to prevent blocking).
> *   ✅ TraceID mechanism (log tracing for a complete processing cycle).
> *   ✅ Markdown skill loading (standard-function-skill plugin).
> *   ✅ guideFile external file reference (system_prompt loaded from .md).
> *   ✅ Purity check script (pnpm test:purity).
> *   ✅ 56 unit tests passing (Vitest).
> *   ⬜ End-to-end LLM call verification (pending manual testing).

---

> **The above completes the Agent core. Below enters the "Agent Operating System" and ecosystem phases.**

---

## Phase 4.5: Multi-Channel Transport
**Goal:** Validate the extensibility of the IUI/IListener separation architecture, implementing non-stdio channels.
**Status:** Done (v0.2.0-beta)

- [x] **4.5.1 WebSocket Transport Plugin**
    - [x] `@openstarry-plugin/transport-websocket`: WebSocket Listener + UI
    - [x] Support for multi-client connections, directed replies (replyTo)
- [x] **4.5.2 HTTP Webhook Transport Plugin**
    - [x] `@openstarry-plugin/transport-http`: HTTP Listener + UI
    - [x] POST /api/input, GET /api/status, GET /api/response
- [x] **4.5.3 Multi-UI Simultaneous Output**
    - [x] TransportBridge broadcast mechanism verified
    - [x] stdio + WebSocket both receiving events simultaneously

> **Milestone: v0.2.0 Beta (Multi-Channel)**
> *   ✅ WebSocket transport plugin implementation complete
> *   ✅ HTTP Webhook transport plugin implementation complete
> *   ✅ Multi-UI simultaneous output verified

---

## Phase 4.6: Implementation Cycle 1 (v0.2.1-beta)
**Goal:** Resolve multi-user privacy issues, optimize transport efficiency, establish connection health checks.
**Status:** Done (Cycle 1, 2026-02-10) — 118 tests, QA PASS, Architect PASS WITH NOTES

### Plan05.1: Session Isolation & Message Routing ✅
- [x] ISession / ISessionManager interfaces (SDK) + SessionManager implementation (Core)
- [x] InputEvent.sessionId field (optional, backward-compatible)
- [x] IPluginContext.sessions exposes ISessionManager to plugins
- [x] Default session (`__default__`) auto-created, cannot be destroyed
- [x] WebSocket session-aware routing (per-session broadcast)
- [x] 17 new tests covering session isolation scenarios

### Plan05.2: HTTP SSE Support ✅
- [x] New `GET /api/events` SSE endpoint
- [x] EventSource compatible format (text/event-stream)
- [x] Session-scoped event filtering
- [x] SSE heartbeat periodic sending
- [x] Coexists with existing HTTP webhook listener
- [x] 11 new tests covering SSE scenarios

### Plan05.5-①: Connection Health Check ✅
- [x] WebSocket protocol ping/pong (server-initiated)
- [x] missedPongs counter, disconnect when exceeding staleThreshold
- [x] HTTP SSE heartbeat periodic check
- [x] HealthCheckConfig configurable (enabled, intervalMs, staleThreshold)
- [x] 6 new tests covering health check scenarios

> **Milestone: v0.2.1-beta**
> *   ✅ Multi-client session isolation (WebSocket + HTTP)
> *   ✅ HTTP SSE real-time streaming
> *   ✅ Connection health checks (ping/pong + heartbeat)
> *   ✅ 118 tests, 11 packages, snapshot saved

---

## Phase 4.7: Implementation Cycle 2 (v0.2.2-beta)
**Goal:** Establish observability infrastructure and standardized error handling.
**Status:** Done (Cycle 2, 2026-02-11) — 165 tests, QA PASS, Architect PASS WITH NOTES

### Plan05.5-②: Metrics / Logging Infrastructure ✅
- [x] MetricsCollector (Core): increment / gauge / getSnapshot / reset
- [x] Logger.time() method (performance.now() timing)
- [x] METRICS_SNAPSHOT event type
- [x] /metrics slash command (AgentCore)
- [x] Transport plugin console.log → createLogger migration
- [x] 19 new tests

### Plan05.5-③: Standardized Error Handling ✅
- [x] ErrorCode const (12 error codes)
- [x] TransportError / SessionError / ConfigError classes
- [x] ES2022 Error cause chain support
- [x] Existing 2-arg constructor backward-compatible
- [x] 16 new tests

> **Milestone: v0.2.2-beta**
> *   ✅ MetricsCollector observability foundation
> *   ✅ Standardized error hierarchy (ErrorCode + cause chain)
> *   ✅ Transport plugin structured logging
> *   ✅ 165 tests, 11 packages, snapshot saved

---

## Phase 4.8: MCP Protocol Integration (v0.3.0-beta)
**Goal:** Implement standardized interconnection between OpenStarry Agent and external MCP Servers.
**Status:** Done (Cycle 3, 2026-02-11) — 86 tests, Architect PASS WITH NOTES

### Plan06: MCP Client Plugin ✅
- [x] `@openstarry-plugin/mcp-client` main implementation
  - [x] Universal MCP transport layer abstraction (stdio, SSE)
  - [x] StdioClientTransport implementation (cross-platform: Windows/Unix)
  - [x] SSEClientTransport implementation
  - [x] RPC communication layer (JSONRPCClient)
- [x] MCP Tool Bridge
  - [x] `mcp:` namespace mechanism
  - [x] `tools/list` → OpenStarry Tool Registry mapping
  - [x] `tools/call` execution and result return
- [x] MCP Prompt Bridge
  - [x] Slash command (`/mcp-prompt-name`) mapping
  - [x] `prompts/list` listing
  - [x] `prompts/get` retrieve full prompt content
- [x] `@openstarry-plugin/mcp-server` server implementation
  - [x] StdioServerTransport implementation
  - [x] MCP Server Protocol handling (initialize, tools/list, tools/call, prompts/*)
  - [x] JSON-RPC reply mechanism
  - [x] Comprehensive error handling
- [x] `@openstarry-plugin/mcp-common` shared types
  - [x] MCP Protocol interface definitions
  - [x] Transport abstraction
  - [x] RPC communication types
- [x] 86 unit tests
  - [x] MCP Client: 53 tests
  - [x] MCP Server: 33 tests

> **Milestone: v0.3.0-beta**
> *   ✅ MCP Client Plugin full implementation
> *   ✅ MCP Server Plugin full implementation
> *   ✅ Stdio and SSE transport support
> *   ✅ Tool and Prompt Bridge interconnection
> *   ✅ 86 tests, 3 new packages, snapshot saved

---

## Phase 4.9: Runtime Sandbox (v0.4.0-beta ~ v0.4.3-beta)
**Goal:** Implement a complete runtime sandbox mechanism ensuring plugin security isolation and control.
**Status:** Done (Cycles 5-8, 2026-02-11) — 442 tests total, QA PASS, Architect PASS WITH NOTES

### Plan07: Runtime Sandbox MVP (Cycle 5) ✅
**Goal:** Establish Worker thread isolation and basic signature verification mechanism.
- [x] SandboxManager and Worker thread management
  - [x] Single Worker initialization and execution
  - [x] RPC communication mechanism (cross-thread MessagePort)
  - [x] Timeout and Memory Limit enforcement
- [x] Signature verification mechanism
  - [x] SHA-256 package-level verification
  - [x] Manifest signature checking
  - [x] Clear error reporting on verification failure
- [x] 82 new tests

> **Milestone: v0.4.0-beta**
> *   ✅ Worker thread isolation implementation
> *   ✅ SHA-256 signature verification
> *   ✅ Memory + CPU timeout control

### Plan07.1: Worker Lifecycle + Pool (Cycle 6) ✅
**Goal:** Implement Worker restart strategies, heartbeat monitoring, and connection pool management.
- [x] Worker restart mechanism
  - [x] Exponential backoff restart strategy (100, 200, 400, 800ms → cap)
  - [x] Automatic recovery of failed Workers
- [x] Heartbeat monitoring
  - [x] Periodic heartbeat checks
  - [x] Stall detection and automatic isolation
  - [x] Missing heartbeat counter
- [x] Worker Pool management
  - [x] Pre-spawned Workers (pool sizing)
  - [x] Automatic recycling and reuse
  - [x] Plugin reset via protocol message
- [x] 110 new tests

> **Milestone: v0.4.1-beta**
> *   ✅ Exponential backoff restart strategy
> *   ✅ Heartbeat monitoring and auto-recovery
> *   ✅ Worker Pool connection pool mechanism

### Plan07.2: Import Analysis + PKI (Cycle 7) ✅
**Goal:** Implement static analysis protection and Ed25519 PKI verification.
- [x] Static import analyzer (AST-based)
  - [x] Recursive require/import scanning
  - [x] Whitelist (allowed) / Blocklist (blocked) module checking
  - [x] Refuse to load on violation (strict mode)
- [x] Ed25519 PKI signature verification
  - [x] Public key configuration mechanism
  - [x] Ed25519 signature verification
  - [x] Multiple signer support
- [x] SandboxConfig interface
  - [x] allowed / blocked modules list
  - [x] publicKeys configuration
  - [x] Runtime policy (strict/warn/off)
- [x] Worker Pool interface (initialize/acquire/release/shutdown)
- [x] 135 new tests

> **Milestone: v0.4.2-beta**
> *   ✅ Static import analysis protection
> *   ✅ Ed25519 PKI verification mechanism
> *   ✅ Module allowlist/blocklist control

### Plan07.3: Audit Logging + Final Hardening (Cycle 8) ✅
**Goal:** Implement audit logging and Module._load interception, completing sandbox hardening.
- [x] AuditLogger mechanism
  - [x] Buffered JSONL writing
  - [x] Asynchronous log recording
  - [x] Secret redaction (password/token/key/auth/credential)
- [x] Module._load interception
  - [x] Runtime module loading control
  - [x] Strict / Warn / Off three modes
  - [x] Call stack tracing
- [x] RPC audit integration
  - [x] Tool execution logging
  - [x] Error tracking
  - [x] Performance metrics
- [x] Log rotation and cleanup
- [x] 115 new tests

> **Milestone: v0.4.3-beta**
> *   ✅ Complete audit logging mechanism
> *   ✅ Module._load runtime control
> *   ✅ Secret redaction and security hardening
> *   ✅ 442 tests (Plan07 full line), snapshot saved

---

## Deferred Items (Deferred Plan05.x)

### Plan05.3: DevTools Debug Interface ⬜
> Deferred until after Plan06 (MCP).

### Plan05.4: E2E Testing Scaffold ⬜
- [ ] `createTestAgent()` utility function
- [ ] Support for MockLLM plugin
- [ ] Lower the barrier for community contributions

---

## Phase 5: The Orchestrator Daemon — Plan07
**Goal:** Implement process-level management and persistence.
**Status:** Done (v0.4.3-beta, Cycles 5-8, 2026-02-11)

- [ ] **5.1 Daemon Core**
    - [ ] Build `apps/daemon`, managing the lifecycle of multiple Agent instances.
    - [ ] Implement scheduling logic in the **Agent Management Zone**.
- [ ] **5.2 API Gateway & Persistence**
    - [ ] Provide HTTP/gRPC API for remote management.
    - [ ] Integrate database (SQLite/LevelDB) to store Agent long-term state.
- [ ] **5.3 Security & Governance**
    - [ ] Implement plugin signing and verification mechanism (ensure loaded plugins are trusted).
    - [ ] Implement runtime sandbox strategy (e.g., Node.js vm or WASM containerization).
    - [ ] **Implement environment isolation and resource quota management**.
- [ ] **5.4 System Observability & Hardware Abstraction (HAL)**
    - [x] Structured JSON logging, supporting agent_id / trace_id / module fields. *(Plan02 Phase D)*
    - [ ] Integrate OpenTelemetry or Event Tracing to trace cross-Agent task flows.
    - [ ] Implement log collection supporting Dashboard visualization.
    - [ ] **Implement Hardware Abstraction Layer (HAL) standard sensory data streams**.

## Phase 6: Fractal Society — Plan06
**Goal:** Implement agent collaboration and MCP protocol.
**Status:** Done (v0.3.0-beta → v0.10.0-beta, Cycle 3 → Cycle 15, 2026-02-11 → 2026-02-12)
**Pre-condition:** Plan05.1 Session Isolation complete ✅ (v0.2.1-beta, 2026-02-10)

> **Entry Checklist:**
> - ✅ Plan05.1 Session Isolation implemented (Cycle 1)
> - ✅ Multi-client scenario verified (118 tests)
> - ✅ Messages do not leak across Sessions (session-aware routing verified)

- [x] **6.1 MCP Deep Integration (Plan06-P1: Tools)**
    - [x] Implement `@openstarry-plugin/mcp-client`, giving Agents standardized interconnection capability.
    - [x] Implement `@openstarry-plugin/mcp-server`, supporting multiple external clients connecting simultaneously
    - [x] 86 unit tests verified (v0.3.0-beta, Cycle 3)
- [x] **6.1a MCP Prompts Integration (Plan06-P2: Prompts)**
    - [x] MCP Prompts RFC 0005 Section 5.2 implementation complete
    - [x] `/mcp-prompt-name` Slash command support
    - [x] Dynamic prompt listing and content retrieval
    - [x] 42 new tests (before v0.9.0-beta, Cycle 9+)
- [x] **6.1b MCP Resources Integration (Plan06-P3: Resources)**
    - [x] MCP Resources RFC 0005 Section 5.5 implementation complete
    - [x] listResources / readResource RPC handling
    - [x] OAuth 2.1 token management (PKCE, auto-refresh, TTL)
    - [x] AES-256-GCM encryption + PBKDF2 (100k iterations)
    - [x] `/mcp-resources` / `/mcp-server-resources` Slash commands
    - [x] 45 new tests (v0.10.0-beta, Cycle 15)
- [ ] **6.2 DevTools Debug Interface (Plan05.3)**
    - [ ] Static HTML debug interface `openstarry-devtools`
    - [ ] Left panel: Conversation bubble view
    - [ ] Right panel: State/Context inspector
    - [ ] Bottom panel: Event filter
- [ ] **6.3 Workflow Engine**
    - [ ] Implement `WorkflowEngineTool`, supporting YAML-defined task orchestration.
    - [ ] Implement Agent dynamic spawning and destruction mechanism (Ephemeral Agents).
- [ ] **6.3 Cross-Agent Collaboration & Causal Scheduling (Orchestration)**
    - [ ] Implement task decomposition and distribution to different sub-agents.
    - [ ] **Implement Causality Chain-based event scheduling**.

## Phase 7: Ecosystem and UI
**Goal:** Provide Web graphical management and deployment tools.
**Status:** Not Started

- [ ] **7.1 Dashboard & Multi-Interaction Layer (Interface)**
    - [ ] Build `apps/dashboard` (React/Next.js), visually monitoring all Agents.
    - [ ] **Implement State Projection dashboard**.
- [ ] **7.2 Template Service (Agent Design Service)**
    - [ ] Provide Agent store/template download functionality, achieving "directory-as-protocol" minimal deployment.

## Phase 8: CLI & TUI Evolution — Plan08 / Plan09
**Goal:** Achieve "operating system level" terminal experience, fulfilling the User Scenario Guide vision.
**Status:** In Progress (v0.5.0-beta Plan08 complete, Plan09 pending)

### Plan08: TUI Dashboard MVP (v0.5.0-beta) ✅
**Status:** Done (Cycle 9, 2026-02-11) — 524 tests (+82 new), 43 test files, QA PASS, Architect PASS_WITH_NOTES

- [x] **8.1 Runtime Dashboard**
    - [x] Implement `@openstarry-plugin/tui-dashboard` (using Ink v5 + React 18)
    - [x] Integrate event monitoring, state display, message streaming
    - [x] Support message categorization, tool call tracing, error counting
    - [x] Keyboard shortcuts (q = quit, Tab = toggle event log)

### Plan12: Daemon Mode MVP (v0.8.0-beta) ✅
**Status:** Done (Cycle 13, 2026-02-12) — 714 tests (+44 new), 18 packages, QA PASS, Architect PASS (1 rework)

- [x] **Background Process Management (Daemon)**
    - [x] CLI commands: `daemon start`, `daemon stop`, `daemon ps`
    - [x] Process spawning with detachment (detached process, unref)
    - [x] PID file management (`~/.openstarry/agents/{agent-id}.pid`)
    - [x] Graceful shutdown (SIGTERM/SIGINT cascade)
- [x] **IPC Layer (JSON-RPC over Unix Domain Socket)**
    - [x] Socket communication: `~/.openstarry/agents/{agent-id}.sock`
    - [x] JSON-RPC 2.0 protocol
    - [x] Health check RPC: `agent.health` → `{ok, uptime, version}`
- [x] **Daemon Plugin** (`@openstarry-plugin/daemon`)
    - [x] IPC server integration
    - [x] Health check provider (IProvider)
    - [x] Process lifecycle management
- [x] **Test Coverage**: 44 new tests (process management, IPC communication, health checks)

> **Milestone: v0.8.0-beta**
> *   ✅ Daemon background process implementation
> *   ✅ IPC communication infrastructure
> *   ✅ 714 tests, 18 packages, snapshot saved

### Plan13: Seamless Attach (v0.9.0-beta) ✅
**Status:** Done (Cycle 14, 2026-02-12) — 747 tests (+33 new), 18 packages, QA PASS, Architect PASS (1 rework)

- [x] **IPC Protocol Extensions**
    - [x] `agent.attach(agentId)` → Connect terminal client, return sessionId
    - [x] `agent.input(sessionId, message)` → Send user input from attached client
    - [x] `agent.detach(sessionId)` → Gracefully close terminal session (daemon continues running)
    - [x] Event forwarding: Core.bus → IPC bridge, sessionId filtering
- [x] **CLI Command** (`openstarry attach [agent-id]`)
    - [x] List running daemons
    - [x] Auto-start daemon (if agent.json exists but daemon is not running)
    - [x] Terminal I/O proxy: stdin → agent.input RPC, daemon events → stdout/stderr
    - [x] Graceful Ctrl+C handling (detach, not kill daemon)
- [x] **Event Forwarder** (`event-forwarder.ts`)
    - [x] Session-filtered event delivery
    - [x] Support for LLM responses, tool execution, error and metrics events
    - [x] JSON serialization (with timestamps)
    - [x] Size limits (64KB messages, 1MB events)
- [x] **Security & Validation**
    - [x] sessionId format validation (UUID v4)
    - [x] inputType whitelist (user, system)
    - [x] Agent health check (verify daemon liveness before attach)
    - [x] Data size enforcement
- [x] **Test Coverage**: 33 new tests (attach command, IPC handlers, event forwarding, I/O proxy)

> **Milestone: v0.9.0-beta**
> *   ✅ Seamless connection to running daemon
> *   ✅ Interactive terminal conversation (attach/input/detach)
> *   ✅ Auto-start and event forwarding
> *   ✅ 747 tests, 18 packages, snapshot saved

### Plan20: Workflow Engine MVP (v0.18.0-beta) ✅
**Status:** Done (Cycle 23, 2026-02-12) — 1104 tests (+37 new), QA PASS, Architect PASS

- [x] **Workflow Engine Plugin** (`@openstarry-plugin/workflow-engine`)
  - [x] YAML declarative multi-step workflow orchestration (Zod validation)
  - [x] 4 step executors: tool, service, llm, command
  - [x] Mustache template interpolation (`{{ }}` syntax)
  - [x] LLM streaming integration (direct IProvider AsyncIterable API)
  - [x] EventBus observation (step:start/end, workflow:start/end/error)
  - [x] LRU cache (in-memory, max 100 entries)
- [x] **Test Coverage**: 37 new tests (8 files)

> **Milestone: v0.18.0-beta**
> *   ✅ Workflow Engine MVP complete
> *   ✅ SDK/Core zero changes (microkernel purity maintained)
> *   ✅ 1104 tests, snapshot saved

### Plan21: Web-based Remote Attach (v0.19.0-beta) ✅
**Status:** Done (Cycle 24, 2026-02-12) — 1132 tests, QA CONDITIONAL PASS

- [x] **3 New/Modified Plugins**
  - [x] `@openstarry-plugin/http-static` (new) — Static HTTP file server (path traversal protection, MIME handling)
  - [x] `@openstarry-plugin/web-ui` (new) — Browser agent interface (HTML config injection, session recovery)
  - [x] `@openstarry-plugin/transport-websocket` (modified) — Auth/CORS/proxy IP resolution
- [x] **Test Coverage**: 147 new tests (6 files)

> **Milestone: v0.19.0-beta**
> *   ✅ Web-based Remote Attach complete
> *   ✅ WebSocket Token authentication + CORS
> *   ✅ 1132 tests, snapshot saved

### Plan22: Plugin Marketplace MVP (v0.20.0-beta) ✅
**Status:** Done (Cycle 25, 2026-02-13) — 1330 tests, QA PASS

- [x] **Plugin Catalog & Installer**
  - [x] Plugin catalog (`plugin-catalog.json`): 15 official plugin listings
  - [x] Plugin lock file (`~/.openstarry/plugins/lock.json`): track installed plugins
  - [x] Installer: workspace-first resolution + npm fallback
  - [x] Short name support (e.g. `standard-function-fs` → `@openstarry-plugin/standard-function-fs`)
  - [x] Batch install (`plugin install --all`)
- [x] **5 New CLI Commands**: plugin install/uninstall/list/search/info
- [x] **Test Coverage**: 198 new tests (77 marketplace + infrastructure)

> **Milestone: v0.20.0-beta**
> *   ✅ Plugin Marketplace MVP complete
> *   ✅ 1330 tests, snapshot saved

### Hotfix: Windows Cross-Platform Fixes + Attach UX (v0.20.1-beta) ✅
**Status:** Done (Cycle 25 hotfix + Cycle 26, 2026-02-13~14) — 1339 tests

- [x] **Windows Cross-Platform Fixes** (Cycle 25 hotfix)
  - [x] Path handling: `path.sep`, `basename()`, `pathToFileURL()` replacing hardcoded Unix paths
  - [x] Daemon IPC: `platform.ts` — Windows named pipe / Linux Unix socket
  - [x] Platform guards: SIGHUP, chmod, mkdirSync, unlinkSync skipped on Windows
  - [x] Plugin installer: `cp()` dereference fallback + `rm()` maxRetries
- [x] **Attach UX Improvements** (Cycle 26)
  - [x] Auto-display provider login status after attach connection (`/provider status`)
  - [x] Welcome message with `/help` hint
- [x] **Token Persistence Fix** (Cycle 26)
  - [x] `provider-gemini-oauth`'s `dispose()` no longer mistakenly deletes token files
  - [x] New `cleanup()` method, only cleaning up callback server resources

> **Milestone: v0.20.1-beta**
> *   ✅ Cross-platform (Windows/Linux) stable operation
> *   ✅ Attach auto-displays provider status
> *   ✅ OAuth token persistence across restarts
> *   ✅ 1339 tests, 117 test files, snapshot saved

---

### Plan09: Interactive Designer (Pending) ⬜
- [ ] **8.2 Interactive Designer**
    - [ ] Implement `openstarry design` context-aware menu.
    - [ ] Build "Five Aggregates Configuration" guided Wizard (Inquirer.js).

---

## Plan Dependency Graph

```
                    ┌──────────────────────────────────────────────────┐
                    │           Complete (v0.20.1-beta)                 │
                    │  Plan01 → Plan02 → Plan03 → Plan04 → Plan05    │
                    │  → Plan06-P1/P2/P3/P4 (MCP Tools/Prompts/Res)  │
                    │  → Plan07-07.3 (Sandbox Hardening)             │
                    │  → Cycle1-8: Sessions, MCP, Sandbox Hardening  │
                    │  → Cycle9: Plan08 TUI Dashboard                │
                    │  → Cycle10-11: Plan09-10 Interactive TUI + CLI │
                    │  → Cycle12: Plan11 DevTools + E2E Framework    │
                    │  → Cycle13: Plan12 Daemon Mode MVP             │
                    │  → Cycle14: Plan13 Seamless Attach             │
                    │  → Cycle15: Plan06-P3 Resources + OAuth 2.1    │
                    │  → Cycle16: Plan14 Multi-client Attach & Mgmt  │
                    │  → Cycle17: Plan06-P4 Sampling & Extensions    │
                    │  → Cycle18: Plan15 SDK Context Extensions      │
                    │  → Cycle19: Plan16 Security Hardening          │
                    │  → Cycle20: Plan17 Plugin Developer Experience │
                    │  → Cycle21: Plan18 Plugin Sync                 │
                    │  → Cycle22: Plan19 Dependency Wiring           │
                    │  → Cycle23: Plan20 Workflow Engine MVP          │
                    │  → Cycle24: Plan21 Web Remote Attach           │
                    │  → Cycle25: Plan22 Plugin Marketplace          │
                    │  → Cycle26: Hotfix (Windows + Attach UX)       │
                    └─────────────────────┬───────────────────────────┘
                                          │
                                          ▼
                                    ┌──────────┐
                                    │ Plan09   │
                                    │ Designer │
                                    │ (To be   │
                                    │ scheduled)│
                                    └──────────┘
```

---

## Issues to Discuss

The following issues do not yet have clear resolutions and need to be discussed during development:

| Issue | Status | Related Plan |
|-------|--------|--------------|
| WebSocket authentication mechanism | Pending | Plan05.1 |
| Multi-instance load balancing | Pending | Plan07 |
| Session expiry strategy | Pending | Plan05.1 |
| SSE reconnection | Pending | Plan05.2 |

### Detailed Description

1. **WebSocket Authentication Mechanism**
   - Is Token authentication needed?
   - Verify on connection vs. verify on each message?
   - Relationship with Session isolation?

2. **Multi-instance Load Balancing**
   - How do multiple Agent instances share WebSocket connections?
   - Is Redis PubSub needed?
   - Sticky Session strategy?

3. **Session Expiry Strategy**
   - How long of inactivity before automatic cleanup?
   - Is a heartbeat mechanism needed?
   - How to notify clients on expiry?

4. **SSE Reconnection**
   - How to resume the event stream after client disconnection?
   - Is Last-Event-ID needed?
   - How to replay missed events on reconnection?
