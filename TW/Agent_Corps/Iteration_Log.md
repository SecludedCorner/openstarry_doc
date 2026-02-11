# OpenStarry Agent Corps — Iteration Log

This file tracks each iteration cycle's decisions and outcomes.

---

## 2026-02-09: Agent Corps Establishment

- **Action**: Established the 6-agent development corps
- **Agents**: architect, dev-core, dev-plugin, qa, doc-keeper, researcher
- **Infrastructure**: Created directory structure, agent definitions, project CLAUDE.md, settings
- **Status**: Setup complete, ready for first iteration cycle
- **Next**: Begin Implementation Cycle 1 (Plan05.1 + 05.2 + 05.5 → v0.2.1-beta)

---

## 20260210_cycle1: Implementation Cycle 1

- **Date**: 2026-02-10
- **Target Plans**: Plan05.1 (Session Isolation) + Plan05.2 (HTTP SSE) + Plan05.5-① (Health Check)
- **Target Version**: v0.2.1-beta
- **Scope**:
  - Plan05.1: WebSocket multi-client session isolation (session token, lifecycle, data isolation)
  - Plan05.2: HTTP Server-Sent Events transport (real-time streaming alternative to polling)
  - Plan05.5-①: Connection health check (heartbeat, reconnection, timeout handling)
- **Strategy**: Two-cycle approach
  - Cycle 1 (this): 05.1 + 05.2 + 05.5-① — establish SOP workflow, deliver critical path items
  - Cycle 2 (next): 05.5-② (metrics/logging) + 05.5-③ (error handling) — can parallel with Plan06 pre-research
- **Pre-Conditions**: Plan05 ✅ (v0.2-beta, 2026-02-07)

### Phase 1 — Design

- **Architecture Spec**: `share/test/reports/arch_reviews/20260210_cycle1/Architecture_Spec_Cycle1.md`
- **Interface Freeze**: YES — All Section 2 interfaces frozen on publication
- **Key Frozen Interfaces**:
  - `ISession` (NEW) — Session entity with id, createdAt, updatedAt, metadata
  - `ISessionManager` (NEW) — Session lifecycle: create/get/list/destroy/getStateManager/getDefaultSession
  - `InputEvent.sessionId` (ADDED) — Optional field; omission falls back to default session
  - `IPluginContext.sessions` (ADDED) — Exposes ISessionManager to plugins
  - `AgentEventType.SESSION_CREATED / SESSION_DESTROYED` (ADDED) — Lifecycle events
  - `SSEConnection` (NEW, internal) — HTTP transport SSE connection tracking
  - `HealthCheckConfig` (NEW, per-plugin) — Shared config shape for heartbeat/stale threshold
  - `ClientConnection.alive / sessionId` (ADDED) — WebSocket health and session binding
- **Key Design Decisions**:
  1. SessionManager is **core infrastructure** (like StateManager), NOT a plugin — maintains microkernel purity; session isolation is a framework concern, not a feature
  2. **Default session (`__default__`)** created on construction — backward compatibility; sessionId-less inputs use shared state as before
  3. `getStateManager(sessionId?)` is the bridge between sessions and ExecutionLoop — loop resolves per-session state at start of processEvent(), no restructuring needed
  4. TransportBridge remains **session-agnostic** — intelligence lives in plugin UI implementations, bridge is dumb infrastructure (microkernel principle)
  5. SSE delivery handled via **per-connection bus subscription** in listener, NOT through HTTP UI — separation of polling vs streaming concerns
  6. WebSocket protocol ping/pong **coexists** with existing app-level ping/pong — different purposes (server-initiated health vs client-initiated liveness)
  7. `IPluginContext.sessions` exposes full `ISessionManager` rather than individual methods — single well-typed interface for transport plugins
  8. Default session **cannot be destroyed** — protects backward compatibility
  9. Event payloads enriched with `sessionId` + `replyTo` — enables session-aware routing without changing event interfaces
- **Implementation Sequencing**:
  - **Phase 2a** (dev-core only): SDK interfaces → Core SessionManager → ExecutionLoop changes → AgentCore changes → test updates (~14 new tests)
  - **Phase 2b** (dev-core + dev-plugin parallel): transport-websocket session/ping/routing + transport-http SSE/heartbeat/session (~12 new tests)
  - Phase 2a must complete before Phase 2b (plugins depend on SDK + Core)
- **Expected Test Count**: 82 existing + ~26 new = ~108 total
- **Risk Assessment**: 8 risks identified (see Architecture Spec Section 9); highest: AgentCore.stateManager removal (Medium, mechanical fix) and memory growth from unbounded sessions (Medium, deferred to future cycle)

- **Status**: Phase 1 — Design Complete → Phase 1.5 Baseline

### Phase 2 — Implementation (2026-02-10)

- Phase 2a COMPLETE: SDK interfaces + Core SessionManager (dev-core)
  - 4 new SDK files/changes, 4 new Core files/changes
  - 18 new tests, total 100 → verified
- Phase 2b COMPLETE: Transport plugins (dev-plugin)
  - transport-websocket: session handshake, protocol ping/pong, session-aware routing (6 new tests)
  - transport-http: SSE endpoint, heartbeat, session input (11 new tests)
  - Core tsconfig fix: exclude test files from build
  - Total: 118 tests all passing
- Build: ✅ pnpm build (11 packages)
- Tests: ✅ 118 passed
- Purity: ✅ PASS

### Phase 2.5 — Sync PENDING

- sync-to-test.sh failed due to Windows symlink issue (node_modules symlinks)
- Needs script fix before retry

- **Status**: Phase 2 Complete, Phase 2.5 pending (sync script needs Windows fix)
- **Resume Point**: Fix sync script → Phase 2.5 → Phase 3 (QA + architect review) → Phase 4 (converge)
