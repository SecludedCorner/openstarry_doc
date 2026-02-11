# OpenStarry — Plan Dependencies & Definition of Done

Established: 2026-02-09

---

## 1. Plan Dependency Graph

```
Plan01 (MVP Alpha Foundation)           ✅ v0.1-alpha  2026-02-04
  ↓
Plan02 (Event-Driven & Safety)          ✅ v0.1.1      2026-02-05
  ↓
Plan03 (Quality & Skill Parser)         ✅ v0.2-alpha  2026-02-05
  ↓
Plan04 (IUI Interface & Guide Plugin)   ✅ v0.2-alpha  2026-02-06
  ↓
Plan05 (Multi-Channel UI & Listener)    ✅ v0.2-beta   2026-02-07
  ↓
  ├── Plan05.1 (Session Isolation)      ⬜ → v0.2.1-beta
  ├── Plan05.2 (HTTP SSE)              ⬜ → v0.2.1-beta   ← Parallel with 05.1
  ├── Plan05.5 (Others)                 ⬜ → v0.2.1-beta   ← Parallel with 05.1
  │     ↓
  │   Plan06 (MCP Protocol)             ⬜ → v0.3       ← Blocked by Plan05.1
  │     ↓
  │   Plan07 (Runtime Sandbox)          ⬜ → v0.4       ← Blocked by Plan06
  │     ↓
  │   Plan08-09 (TUI Dashboard)         ⬜ → v0.5
  │
  └── (Extended Mode: New plugins, Web UI, Safety Audit)
```

### Critical Path

```
Plan05.1 → Plan06 → Plan07 → Plan08-09
```

Plan05.1 (Session Isolation) is the first uncompleted item on the critical path. Plan06 (MCP) is blocked by it.

### Parallelizable Work

| Parallelizable | Description |
|--------|------|
| Plan05.1 + Plan05.2 + Plan05.5 | These three can be merged into a single iteration (Implementation Cycle 1). |
| Plan06 Pre-research + Plan05.x Implementation | The researcher can perform pre-research on Plan06 while dev agents implement 05.x. |

---

## 2. Definition of Done (DoD)

### 2.1 General DoD (Applies to all Plans)

Each Plan must satisfy **all** of the following conditions before being marked as ✅:

| # | Condition | Verification Method |
|---|------|---------|
| 1 | `pnpm build` passes (monorepo + plugins). | Verified by QA in `agent_test`. |
| 2 | `pnpm test` all pass, count ≥ previous baseline. | Regression check in QA Report. |
| 3 | `pnpm test:purity` passes. | Purity check in QA Report. |
| 4 | All items in Architecture_Spec implemented. | Architect Code Review PASS. |
| 5 | Five Aggregates compliance (components correctly categorized). | Confirmed in Architect Code Review. |
| 6 | pushInput pattern compliance. | Confirmed in Architect Code Review. |
| 7 | No known security vulnerabilities. | Confirmed in Architect Code Review. |
| 8 | Dev logs written. | Coordinator confirms files exist. |
| 9 | QA Report and Code Review are both PASS. | Determined in Phase 4. |
| 10 | doc-keeper updated Plan (✅) + Iteration_Log. | Converged in Phase 4. |
| 11 | Snapshot established. | `scripts/snapshot.sh` complete. |
| 12 | Lessons Learned recorded. | Reviewed in Phase 4. |

### 2.2 Plan-Specific Acceptance Criteria

#### Plan05.1: Session Isolation
- [ ] WebSocket connections support session tokens / auth.
- [ ] Multiple client connections isolated (no cross-session data leakage).
- [ ] A single agent instance can serve multiple sessions.
- [ ] Session lifecycle management (creation, maintenance, destruction).
- [ ] New tests covering session isolation scenarios.

#### Plan05.2: HTTP SSE
- [ ] HTTP Server-Sent Events transport plugin available.
- [ ] SSE connections can receive real-time streaming responses.
- [ ] Coexists with existing HTTP webhook listeners without conflict.
- [ ] New tests covering SSE scenarios.

#### Plan06: MCP Protocol (Model Context Protocol)
- [ ] Implement tool, resource, and prompt capabilities from the MCP spec.
- [ ] MCP server mode: OpenStarry acts as an MCP server for external clients.
- [ ] MCP client mode: OpenStarry connects to external MCP servers to acquire tools.
- [ ] Compliant with the official MCP specification (confirmed by researcher).
- [ ] New tests covering MCP scenarios.

#### Plan07: Runtime Sandbox
- [ ] Plugin execution environment isolation (vm or WASM).
- [ ] Resource limits (CPU, memory, file access).
- [ ] Plugin signature verification mechanism.
- [ ] New tests covering sandbox escape scenarios.

#### Plan08-09: TUI Dashboard
- [ ] Terminal interactive Dashboard (Ink or Blessed).
- [ ] Real-time display of Agent status, dialogues, and tool calls.
- [ ] Support for multi-session switching.
- [ ] New tests covering TUI scenarios.

---

## 3. Iteration Scheduling Recommendations

| Iteration | Covering Plans | Target Version | Pre-conditions |
|------|-----------|---------|---------|
| Cycle 1 | Plan05.1 + 05.2 + 05.5 | v0.2.1-beta | Plan05 ✅ (Satisfied). |
| Cycle 2 | Plan06 (MCP) | v0.3 | Plan05.1 ✅. |
| Cycle 3 | Plan07 (Sandbox) | v0.4 | Plan06 ✅. |
| Cycle 4 | Plan08-09 (TUI) | v0.5 | Plan07 ✅ (or can be moved up). |
| Extended | New features, Safety Audit | v1.0 | All Plans ✅. |

---

## 4. Usage

- **Coordinator**: Before starting a new iteration, consult this document to confirm:
  1. Pre-requisite Plans are complete ✅.
  2. Specific acceptance criteria for the target Plan.
  3. Opportunities for parallel work.
- **architect**: Reference the specific acceptance criteria when designing the Spec to ensure coverage.
- **qa**: Check general DoD + specific criteria item-by-item during verification.
- **doc-keeper**: Confirm all DoD conditions are met before marking a Plan as ✅.
