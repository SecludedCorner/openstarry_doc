# How OpenStarry is Built: Multi-Agent Development

> OpenStarry is not just an AI agent framework. It's an AI agent framework **built by AI agents**.

## The 6-Agent Team

OpenStarry is developed by a team of 6 specialized AI agents operating within a formal Standard Operating Procedure (SOP). Each agent has a defined role, a dedicated workspace, and structured output requirements. This isn't a toy demo — it's a genuine software engineering process that produces architecture-reviewed, quality-verified, documented code.

### The Team

| Agent | Role | Model | Workspace | Output |
|-------|------|-------|-----------|--------|
| **Architect** | Architecture Guardian | Sonnet | Docs + code (read-only) | Design specs, code reviews, security audits |
| **Dev-Core** | Core Developer | Sonnet | `agent_dev/openstarry/` | SDK, Core, Shared, Runner implementations |
| **Dev-Plugin** | Plugin Developer | Sonnet | `agent_dev/openstarry_plugin/` | Transport, provider, tool, guide plugins |
| **QA** | Quality Assurance | Sonnet | `agent_test/` (isolated copy) | Build verification, test reports, purity checks |
| **Doc-Keeper** | Documentation Keeper | Haiku | `share/openstarry_doc/` | Decision records, iteration logs, doc updates |
| **Researcher** | Technical Researcher | Sonnet | `share/ref/` | Pre-research reports, reference project analysis |

A human **Coordinator** orchestrates the team using Claude Opus, makes final decisions, resolves disputes, and manages the cycle transitions.

### Why This Structure?

Each role exists for a reason:

- **Architect** prevents architectural erosion. Without it, the Five Aggregates purity would slowly degrade as shortcuts accumulate. The Architect enforces compliance on every commit.
- **Separate Dev-Core and Dev-Plugin** agents can work in parallel because the SDK interfaces are frozen. Dev-Core builds the framework while Dev-Plugin builds on it — simultaneously.
- **QA works in an isolated copy** (`agent_test/`), not the development directory. This prevents the classic "it works on my machine" problem — if it passes in the test environment, it really passes.
- **Doc-Keeper uses a lighter model** (Haiku) because documentation updates don't need the reasoning power of code generation, but they need to be consistent and thorough.
- **Researcher** does pre-research before implementation begins, studying reference projects (OpenClaw for agent routing, OpenCode for TUI patterns, OpenOctopus for MCP integration) to inform design decisions.

## The Iteration Cycle

Every feature goes through a disciplined 8-phase cycle with quality gates:

### Phase 0: Planning
```
Coordinator: "Cycle 1 scope: Session Isolation + HTTP SSE + Health Check"
Researcher: Studies transport patterns, SSE best practices, session management strategies
Doc-Keeper: Records the plan in Iteration_Log.md with cycle ID (20260210_cycle1)
```

### Phase 1: Design
```
Architect: Produces Architecture_Spec_Cycle1.md with:
  - New/modified interfaces (ISession, ISessionManager, InputEvent.sessionId)
  - Sequence diagrams for session lifecycle
  - Security considerations (session isolation prevents cross-talk)

⚠️ INTERFACE FREEZE: Once published, interfaces cannot change without a formal Spec Addendum
```

### Phase 1.5: Baseline
```
Coordinator: runs scripts/baseline.sh → saves pre-implementation snapshot
  └─ Safety net: if anything goes wrong, restore to this point
```

### Phase 2: Implementation
```
Phase 2a: Dev-Core implements SDK interface changes first
  └─ ISession, ISessionManager added to @openstarry/sdk
  └─ InputEvent gains optional sessionId field (backward compatible)
  └─ pnpm build must pass before Phase 2b begins

Phase 2b: Dev-Core + Dev-Plugin work in parallel
  └─ Dev-Core: Session manager, execution loop session routing
  └─ Dev-Plugin: WebSocket session binding, HTTP SSE endpoints
  └─ Each produces a structured DevLog with decisions, challenges, solutions
```

### Phase 2.5: Sync
```
Coordinator: runs scripts/sync-to-test.sh
  └─ Atomic copy from agent_dev/ → agent_test/
  └─ Ensures test environment has exact copy of development code
  └─ pnpm build in test environment must pass
```

### Phase 3: Verification (Parallel)
```
QA (in agent_test/):                    Architect (reading agent_dev/):
├─ pnpm build → all 11 packages pass   ├─ Five Aggregates compliance check
├─ pnpm test → 118+ tests pass         ├─ Microkernel purity review
├─ pnpm test:purity → zero violations  ├─ Security audit
└─ Produces QA_Report_Cycle1.md        └─ Produces CodeReview_Cycle1.md
```

### Phase 4: Convergence
```
PASS: Coordinator runs scripts/snapshot.sh → versioned snapshot saved
FAIL: Issues classified:
  ├─ Code Fix → back to Phase 2 (bug in implementation)
  ├─ Design Fix → back to Phase 1 (architecture issue)
  └─ Plan Fix → back to Phase 0 (requirements problem)

Max 2 rework cycles before human escalation
```

## Quality Gates

### Interface Freeze
Once the Architect publishes an Architecture Spec, the interfaces are frozen. Dev-Core can implement them, but can't change them. This prevents the scope creep and interface instability that plagues most projects. If an interface needs to change, it requires a formal Spec Addendum — a deliberate, reviewed process.

### Microkernel Purity Verification
QA runs `pnpm test:purity` which scans the compiled Core binary for any plugin imports. **Zero violations allowed.** This automated gate prevents the gradual contamination that erodes microkernel architectures over time.

### Test Isolation
QA never tests in the development directory. The sync script copies code to a separate `agent_test/` environment. If a developer left a temporary hack that makes tests pass only with local state, QA catches it.

### Rework Classification
Not all failures are equal:
- **Code Fix** → Quick patch, back to Phase 2
- **Design Fix** → Interface change needed, back to Phase 1 (requires Spec Addendum)
- **Plan Fix** → Requirements were wrong, back to Phase 0 (rare but serious)

## Structured Artifacts

Every cycle produces traceable, structured reports:

```
share/test/reports/
├── research/20260210_cycle1/
│   └── Research_Transport_Enhancement.md      ← Researcher
├── arch_reviews/20260210_cycle1/
│   ├── Architecture_Spec_Cycle1.md            ← Architect (FROZEN)
│   └── CodeReview_Cycle1.md                   ← Architect
├── dev_logs/20260210_cycle1/
│   ├── DevLog_Phase2a_SDK_Core.md             ← Dev-Core
│   └── DevLog_Phase2b_Transport_Plugins.md    ← Dev-Plugin
├── qa_results/20260210_cycle1/
│   └── QA_Report_Cycle1.md                    ← QA
└── sys_summary/20260210_cycle1/
    └── Convergence_Summary.md                 ← Coordinator
```

All named with cycle IDs (`{YYYYMMDD}_cycle{N}`) for full traceability. Six months from now, you can trace any line of code back to the research that motivated it, the spec that defined it, the dev log that implemented it, and the QA report that verified it.

## What This Means for the Community

When OpenStarry is released, contributors inherit:

1. **A complete development process** — not just code, but a way of working
2. **Architectural guardrails** — automated purity tests prevent well-meaning contributions from eroding the architecture
3. **Traceable decisions** — every "why" is documented, not lost in chat history
4. **Parallel-safe development** — Dev-Core and Dev-Plugin can be separate contributors working simultaneously, because the SDK interfaces are frozen
5. **Reproducible quality** — the SOP is the same whether the developer is human or AI

> The process is part of the product.
