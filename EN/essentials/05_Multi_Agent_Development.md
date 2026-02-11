# How OpenStarry was Built: Multi-Agent Development

> OpenStarry is not just an AI Agent framework. It is an AI Agent framework **built by** AI Agents.

## The 6-Agent Team

OpenStarry is developed by a team of six professional AI Agents following a formal Standard Operating Procedure (SOP). Each Agent has a clear role, a dedicated workspace, and structured output requirements. This is not a toy demonstration—it is a real software engineering process producing architecture-reviewed, quality-validated, and fully documented code.

### Team Members

| Agent | Role | Model | Workspace | Output |
|-------|------|------|--------|------|
| **Architect** | Architecture guardian | Sonnet | Docs + Code (Read-only) | Design Specs, Code Reviews, Security Audits |
| **Dev-Core** | Core developer | Sonnet | `agent_dev/openstarry/` | SDK, Core, Shared, Runner implementation |
| **Dev-Plugin** | Plugin developer | Sonnet | `agent_dev/openstarry_plugin/` | Transport, Provider, Tool, Guide plugins |
| **QA** | Quality Assurance | Sonnet | `agent_test/` (Isolated copy) | Build validation, Test reports, Purity checks |
| **Doc-Keeper** | Documentation keeper | Haiku | `share/openstarry_doc/` | Decision records, Iteration logs, Doc updates |
| **Researcher** | Technical researcher | Sonnet | `share/ref/` | Pre-research reports, Reference project analysis |

A human **Coordinator** uses Claude Opus to direct the team, make final decisions, resolve disputes, and manage cycle transitions.

### Why this structure?

Every role exists for a reason:

- **Architect** prevents architectural erosion. Without it, the purity of the Five Aggregates would degrade as shortcuts accumulate. The Architect enforces compliance checks on every commit.
- **Independent Dev-Core and Dev-Plugin** Agents can work in parallel because the SDK interfaces are frozen. Dev-Core builds the framework, and Dev-Plugin develops on top of it—synchronously.
- **QA works in an isolated copy** (`agent_test/`), not the development directory. This prevents the classic "it works on my machine" problem—if it passes in the test environment, it truly passes.
- **Doc-Keeper uses a lighter model** (Haiku) because documentation updates do not require the reasoning power of code generation but need consistency and completeness.
- **Researcher** performs pre-research before implementation begins, studying reference projects (OpenClaw for Agent routing, OpenCode for TUI patterns, OpenOctopus for MCP integration) to inform design decisions.

## Iteration Cycles

Every feature goes through a rigorous 8-stage cycle with quality gates:

### Phase 0: Planning
```
Coordinator: "Cycle 1 Scope: Session Isolation + HTTP SSE + Health Check"
Researcher: Researches Transport patterns, SSE best practices, Session management strategies.
Doc-Keeper: Records the plan in Iteration_Log.md with a cycle ID (20260210_cycle1).
```

### Phase 1: Design
```
Architect: Produces Architecture_Spec_Cycle1.md, including:
  - New/modified interfaces (ISession, ISessionManager, InputEvent.sessionId)
  - Sequence diagrams for Session lifecycle
  - Security considerations (Session isolation preventing cross-session interference)

⚠️ Interface Freeze: Once published, interfaces are immutable except via a formal Spec Addendum.
```

### Phase 1.5: Baseline Backup
```
Coordinator: Executes scripts/baseline.sh → saves a snapshot before implementation.
  └─ Safety net: Roll back to this point if things go wrong.
```

### Phase 2: Implementation
```
Phase 2a: Dev-Core implements SDK interface changes first.
  └─ ISession, ISessionManager added to @openstarry/sdk.
  └─ InputEvent gains optional sessionId field (backward compatible).
  └─ pnpm build must pass to enter Phase 2b.

Phase 2b: Dev-Core + Dev-Plugin work in parallel.
  └─ Dev-Core: Session manager, Session routing in Execution Loop.
  └─ Dev-Plugin: WebSocket Session binding, HTTP SSE endpoints.
  └─ Both produce structured DevLogs recording decisions, challenges, and solutions.
```

### Phase 2.5: Synchronization
```
Coordinator: Executes scripts/sync-to-test.sh.
  └─ Atomic copy from agent_dev/ to agent_test/.
  └─ Ensures the test environment has a clean copy of the dev code.
  └─ pnpm build must pass in the test environment.
```

### Phase 3: Verification (Parallel)
```
QA (in agent_test/):                      Architect (reading agent_dev/):
├─ pnpm build → all 11 packages pass      ├─ Five Aggregates compliance check
├─ pnpm test → 118+ tests pass            ├─ Microkernel purity review
├─ pnpm test:purity → zero violations     ├─ Security audit
└─ Produces QA_Report_Cycle1.md           └─ Produces CodeReview_Cycle1.md
```

### Phase 4: Convergence
```
PASS: Coordinator executes scripts/snapshot.sh → saves versioned snapshot.
FAIL: Issue classification:
  ├─ Code Fix → Back to Phase 2 (implementation bugs)
  ├─ Design Fix → Back to Phase 1 (architectural issues)
  └─ Plan Fix → Back to Phase 0 (requirement issues)

Max 2 rework loops before escalating to human intervention.
```

## Quality Gates

### Interface Freeze
Once the Architect publishes the Architecture Spec, interfaces are frozen. Dev-Core can implement them but cannot change them. This prevents the scope creep and interface instability that plagues most projects. Changes require a formal Spec Addendum—a deliberate, reviewed process.

### Microkernel Purity Verification
QA runs `pnpm test:purity`, scanning the compiled Core binary for any plugin imports. **Zero violations allowed.** This automated gate prevents the "leaky abstraction" that erodes microkernel architectures over time.

### Test Isolation
QA never tests in the development directory. The sync script copies code to a clean `agent_test/` environment. If a developer leaves a transient modification that makes tests pass only in their local state, QA will catch it.

### Rework Classification
Not all failures are equal:
- **Code Fix** → Fast patch, back to Phase 2.
- **Design Fix** → Requires interface change, back to Phase 1 (requires Spec Addendum).
- **Plan Fix** → Wrong requirements, back to Phase 0 (rare but serious).

## Structured Deliverables

Every cycle produces a traceable, structured trail of reports:

```
share/test/reports/
├── research/20260210_cycle1/
│   └── Research_Transport_Enhancement.md      ← Researcher
├── arch_reviews/20260210_cycle1/
│   ├── Architecture_Spec_Cycle1.md            ← Architect (Frozen)
│   └── CodeReview_Cycle1.md                   ← Architect
├── dev_logs/20260210_cycle1/
│   ├── DevLog_Phase2a_SDK_Core.md             ← Dev-Core
│   └── DevLog_Phase2b_Transport_Plugins.md    ← Dev-Plugin
├── qa_results/20260210_cycle1/
│   └── QA_Report_Cycle1.md                    ← QA
└── sys_summary/20260210_cycle1/
    └── Convergence_Summary.md                 ← Coordinator
```

All files are named with the cycle ID (`{YYYYMMDD}_cycle{N}`), ensuring full traceability. Six months later, you can trace any line of code back to the research that inspired it, the spec that defined it, the dev log that implemented it, and the QA report that validated it.

## What This Means for the Community

When OpenStarry is released, contributors inherit more than just code:

1. **A full development process** — not just code, but a way of working.
2. **Architectural guardrails** — automated purity tests prevent well-intentioned contributions from eroding the architecture.
3. **Traceable decisions** — every "why" is documented, not lost in chat history.
4. **Parallel-safe development** — Dev-Core and Dev-Plugin can be different contributors working at the same time because SDK interfaces are frozen.
5. **Reproducible quality** — the SOP is the same whether the developer is human or AI.

> The process is part of the product.
