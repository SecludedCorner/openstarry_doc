# 研究團隊轉型提案

**Cycle**: 03-1
**Date**: 2026-03-24
**Authors**: LEIBNIZ (#14) + SUNYATA (#0)
**Status**: ACCEPTED IN PRINCIPLE -- actual switch DEFERRED
**Sources**: D2-R10 (communication architecture debate) + D4-R3 (engineering planning debate)

---

## 1. 現狀: 24 人扁平結構

Current structure: 24 agents in flat R0->R1->R2->R3->R4 Pipeline.

**優勢**:
- Well-understood workflow, proven productive across Cycle 02 series
- Simple coordination (SUNYATA as sole moderator)
- No inter-group overhead

**劣勢**:
- Sequential bottleneck (all 24 agents must complete each round before next)
- No specialization -- every agent participates in every topic
- Does not mirror the multi-agent architecture being designed (missed self-study opportunity)

---

## 2. 提案: 四組結構

### Group A: Architecture (5人)

| # | Agent | Specialty |
|---|-------|-----------|
| 3 | VITRUVIUS (Lead) | Full-stack architecture analysis, doc gap analysis |
| 22 | SUSSMAN | Program abstraction, ICommChannel design |
| 6 | DARWIN | Software pattern analysis, EventBridge |
| 10 | KERNEL | OS process model, Core purity |
| 20 | TANENBAUM | Microkernel IPC, seL4 security model |

**Focus**: System design, interface specification, architecture document authoring. Responsible for ICommChannel, Process Tree, EventBridge design decisions.

### Group B: Philosophy (4人)

| # | Agent | Specialty |
|---|-------|-----------|
| 7 | NAGARJUNA (Lead) | Madhyamaka (Two Truths, emptiness constraints) |
| 8 | ASANGA | Yogacara (eight consciousnesses allocation) |
| 18 | PENROSE | Multi-consciousness binding problem |
| 13 | LINNAEUS | Agent taxonomy, skandha distribution matrix |

**Focus**: Buddhist-philosophy mapping, consciousness model, tenet interpretation, classification ontology. Responsible for D3-R1 hybrid model maintenance, Two Truths Declaration enforcement, tenet compliance interpretation.

### Group C: Verification (5人)

| # | Agent | Specialty |
|---|-------|-----------|
| 9 | BABBAGE (Lead) | Formal methods, continuity test, pi-calculus |
| 11 | GUARDIAN | Security analysis, capability model enforcement |
| 12 | WIENER | Control theory, Lyapunov stability, W2 protocol |
| 21 | KNUTH | Algorithm analysis, auditor coverage |
| 19 | PASCAL | Decision theory, statistical design, resource budgeting |

**Focus**: Formal verification, security review, control theory analysis, statistical testing, compliance assessment. Responsible for BABBAGE continuity test, W2-R1 protocol, TOST evaluation, tenet compliance verification.

### Group D: Engineering (7人)

| # | Agent | Specialty |
|---|-------|-----------|
| 16 | ARCHIMEDES (Lead) | Engineering practice, Plan spec authoring |
| 17 | TURING | Source code analysis, three-column verification |
| 15 | HERACLITUS | Runtime dynamics, failure isolation, graceful shutdown |
| 4 | MESH | Distributed systems, Pipeline/Mesh evaluation |
| 5 | ATHENA | AI/ML systems, Hub-and-Spoke, Channels evaluation |
| 14 | LEIBNIZ | Multi-agent cooperation, fractal composition |
| 23 | RUSSELL | AI agent theory, AIMA framework alignment |

**Focus**: Implementation specifications, code analysis, runtime dynamics, engineering feasibility. Responsible for Plan spec production, LOC estimation, verification planning, engineering recommendations.

**Note**: MESH (#4) abstained on group assignment (D2-R10 vote), noting Architecture group may be more appropriate. Assignment to be finalized by SUNYATA before pilot.

### Cross-group Coordinators (3人)

| # | Agent | Role |
|---|-------|------|
| 0 | SUNYATA | Overall coordination across all groups (Hub) |
| 1 | SYNTHESIST | Cross-group synthesis, integration of group outputs |
| 2 | SCRIBE | Cross-group documentation, record keeping |

---

## 3. 通訊模式

Mirrors the multi-agent communication architecture designed in D2:

| Level | Mode | Description |
|-------|------|-------------|
| **Intra-group** | Pipeline | Sequential handoff within group (e.g., R1 parallel -> internal R2 -> internal summary) |
| **Inter-group** | Hub-and-Spoke | SUNYATA as Hub, group leads (VITRUVIUS, NAGARJUNA, BABBAGE, ARCHIMEDES) as Spokes |
| **Cross-cutting** | Pipeline | Architecture -> Engineering -> Verification (design -> implement -> verify) |
| **System-wide** | Broadcast | SUNYATA announcements, Master directives |

**Group-to-group handoffs**: Structured artifacts (not ad-hoc messages). Each group produces a typed deliverable document that the next group consumes.

This makes the research team itself a case study for multi-agent communication patterns (LEIBNIZ D2-R10).

---

## 4. 切換條件 (D4-R3)

ALL three conditions must be met before actual switch:

1. **Plan37 W2 implemented and verified** -- ICommChannel + PipelineChannel + Process Tree must be working in the codebase. The team should use abstractions that exist, not hypothetical ones.
2. **One full research cycle completed with multi-agent codebase** -- at least one cycle where the research team has concrete multi-agent code to analyze. This provides domain knowledge needed for the grouped structure.
3. **Inter-group communication protocol documented** -- using ICommChannel concepts from D2-R2. The protocol must specify: message format between groups, handoff artifacts, escalation path, conflict resolution.

---

## 5. 時程

| Cycle | Activity |
|-------|----------|
| 03-1 (current) | Proposal drafted and accepted in principle |
| 03-2 | Continue flat structure. Verify Plan37. Execute W2-R1. |
| 03-3 | Continue flat structure. Verify Plan38 if delivered. Evaluate switch conditions. |
| 03-4 | **Pilot**: Run one bounded task (e.g., a single debate topic) using 4-group structure. Evaluate coordination overhead vs productivity gain. |
| 03-5+ | **Full switch** if pilot successful and all three conditions met. |

---

## 6. 風險分析

| Risk | Mitigation |
|------|-----------|
| Premature switch increases coordination overhead | Three conditions gate ensures readiness |
| Group silos reduce cross-pollination | Cross-group coordinators (SUNYATA, SYNTHESIST, SCRIBE) bridge all groups |
| Group lead bottleneck | Each group has internal Pipeline; lead is coordinator, not gatekeeper |
| MESH group assignment ambiguity | SUNYATA finalizes before pilot based on workload analysis |
| Loss of flat-structure's simplicity | Pilot on bounded task before full switch; revert option preserved |

---

## 7. 哲學映射

The four-group structure reflects the multi-agent architecture being designed:

- Each group is an **ICompositeAgent** (D3-R3) with internal agents
- Permission lattice applies: each group's scope is a subset of the full research scope
- Fractal recursion: group -> sub-group possible if groups grow large enough
- Buddhist sangha model (D3-R3): individual researcher complete in themselves; group is conventional designation (prajnapti), not ontologically distinct entity

This alignment between research structure and research subject enables "eating our own dogfood" -- the team experiences the coordination patterns it designs.

---

*LEIBNIZ (#14) + SUNYATA (#0)*
*Cycle 03-1, R4 deliverable*
*2026-03-24*
