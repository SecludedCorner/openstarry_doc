# Cycle 03-2 Scope Notes

**Prepared**: 2026-03-24 (end of Cycle 03-1)
**Phase**: Phase 6 -- Multi-Agent
**Compliance baseline**: 8/1/1 (v0.36.1-alpha)
**Expected baseline after Plan37**: 8/1/1 (deepened, #6/#10 advancing)
**Baseline rules**: #1-#30 + #29 modified + #31-#39 new = 39 Current Baseline

---

## 1. Primary Deliverables

### 1.1 Plan37 Verification

Verify Plan37 delivery (v0.37.0-alpha), four waves ordered by priority:

| Wave | Priority | Content | Est. LOC | Key Verification Points |
|------|----------|---------|----------|------------------------|
| W1 | P0 | BUG-3 fix (worst-risk + per-tool audit) + cumulative clamp +0.05 verification | ~50 | gear-arbiter-static worst-risk scan, `audit:tool_audited` per tool, MAX_CUMULATIVE_POSITIVE=0.05, MAX_CUMULATIVE_NEGATIVE=-0.05 |
| W2 | P1 | ICommChannel SDK type + PipelineChannel plugin + Process Tree (parentAgentId, spawnChildAgent, permission lattice) | ~300 | ICommChannel in SDK (not Core), PipelineChannel messaging capability, BABBAGE zero-child bisimulation, child.allowedPaths subset parent |
| W3 | P1 | AC-6 front-five typed IListener subinterfaces + communication capability in agent.json | ~200 | 5 typed subinterfaces with senseType discriminant, union-compatible with existing IListener, agent.json communication section |
| W4 | P2, conditional | EventBridge Daemon service + L2 Global ServiceRegistry | ~150 | EventBridge in Daemon (not Core), per-agent event type whitelist, L2 Registry DNS-pattern lookup |

**Verification approach**: TURING three-column + BABBAGE BCT + GUARDIAN security review + SUSSMAN architecture review

**Critical constraint**: W1 MUST be independently verified before W2 (BUG-3 is precondition for W2-R1)

### 1.2 Plan38 Specification

Based on D2-R1 ordering (Hub-and-Spoke next) and deferred items:

- **Hub-and-Spoke communication mode**: MessageRouter + structured toolset + capability checking (~400-500 LOC)
- **L2/L3 failure isolation**: Circuit Breaker + Bulkhead (D2-R8, deferred from Plan37)
- **L5 Timeout Hierarchy**: CommMessage timeout propagation (D2-R8)
- **AC-7 (Alaya distributed) initial design**: DistributedAlaya interface + seed propagation protocol (D3-R2, ~800+ LOC)
- **Doc 13 (Orchestrator Daemon)**: Multi-agent lifecycle management update (D4-R2, P1)
- **NEW: Multi-Agent Security Model doc** (D2-R9, P1)

### 1.3 W2-R1 Execution

**Preconditions** (ALL must pass):
1. BUG-3 fix verified in isolation (5-round mini-test, multi-tool audit confirmed)
2. Cumulative clamp = +0.05 verified in code
3. LLM >= 70B (9B PROHIBITED)

**Protocol** (D4-R4):
- 50 cycles, 5 blocks (category-targeted)
- SC-1~SC-6 evaluation (SC-5 category representation + SC-6 bidirectional delta are NEW)
- Parameters recalibrated from fresh data (do NOT preset from W2 v2)
- Three-tier verdict: GO / INFORMATIVE FAILURE / REDESIGN

**Expected outcome**: sigma >> 0.000245 (W2 v2 value under BUG-3 conditions). WIENER projects post-fix sigma ~0.012.

---

## 2. Deferred Items from 03-1

| Item | Source | Priority | Target |
|------|--------|----------|--------|
| Asymmetric clamp (+0.05/-0.10) evaluation | D1-R3 | HIGH | 03-2 (requires W2-R1 empirical data) |
| RUSSELL's Blackboard/Alaya unification | D2-R7 | MEDIUM | Plan39+ (after both implemented) |
| Mesh communication mode | D2-R1 | LOW | Phase 6 late (requires API Runtime) |
| API Runtime implementation | D2-R1 | MEDIUM | Plan39+ |
| Full typed Zod sub-schemas for ICommChannel | D2-R2, Rule #21 | MEDIUM | Plan38 |
| Tool-level capability filtering (FIPA ACL granularity) | D2-R5 | LOW | Plan39+ |
| Doc 06 (Inter-Agent MCP) reposition | D2-R9 | LOW | Plan39 |
| Doc 22 (Coordination Normalization) | D2-R9 | LOW | Plan39 |
| Doc 07 (Management Zone) cross-agent events | D2-R9 | LOW | Plan40+ |
| Unified Comm Interface Spec (new doc) | D2-R9 | MEDIUM | Plan39 |
| Multi-Agent State Consistency (new doc) | D2-R9 | MEDIUM | Plan39+ |
| Must-invoke broader scope (VedanaSensor, ConfirmationGate, multi-agent) | D1-R4 | MEDIUM | Subsequent cycles |
| PROC-MIGRATE-1 second case | Cycle 02-12 | LOW | 03-2+ |

### D1-R3 Required Evidence for Asymmetric Clamp Decision
1. WIENER: Stability analysis under asymmetric clamp -- confirm no "confidence trap" with cold-start confidence=0
2. KNUTH: W2-R1 empirical delta distribution -- actual negative delta frequency and magnitude post-fix
3. BABBAGE: Mechanism/policy classification of floor value with formal justification

---

## 3. Tenet #6 / #10 Upgrade Tracking

### Tenet #6 (Eight Consciousnesses)
- **AC-6 (Plan37)**: Front-five typed IListener subinterfaces -- verify in Plan37 verification
- **AC-7 (Plan38+)**: Alaya distributed -- initial design in Plan38 spec. Dependencies: Process Tree + ICommChannel from Plan37
- **D3-R1 hybrid model**: Per-agent 1-7, shared 8 as samvriti-satya. Architecture constraint for all subsequent plans.
- **GUARDIAN prerequisite**: Security review required before AC-7 implementation

### Tenet #10 (Fractal Social)
- **ICompositeAgent pattern (D3-R3)**: Permission lattice, reserve_ratio=0.3 default, depth=3 default, bisimulation required
- **Plan37 W2**: Process Tree (spawnChildAgent, terminateChild, parentAgentId) -- first implementation step
- **Plan38+**: Full ICompositeAgent with resource budgeting

---

## 4. Architecture Doc Priority (03-2)

| Priority | Document | Action |
|----------|----------|--------|
| P1 | Doc 13 (Orchestrator Daemon) | Multi-agent lifecycle: ProcessTree, Graceful Shutdown, Supervisor Strategy |
| P1 | NEW: Multi-Agent Security Model | Capability model, trust boundaries, attack surface analysis |
| P0 (verify) | Doc 19 + Doc 09 + Doc 57 | Verify Plan37 delivery includes these updates (authored in 03-1) |

---

## 5. Novel

- Three versions (Youth / Academic / Expert) x four languages (TW / EN / CN / JP) -- every sub-cycle per standing directive

---

## 6. Carry-Over Tracking

| # | Item | Source | Priority | Status |
|---|------|--------|----------|--------|
| 1 | Asymmetric clamp evaluation | D1-R3 | HIGH | Awaiting W2-R1 data |
| 2 | W2-R1 execution | D4-R4 | P0 | Precondition: BUG-3 fix verified |
| 3 | Plan38 spec | D4-R1 | P1 | Hub-and-Spoke + L2/L3 isolation + AC-7 design |
| 4 | Blackboard/Alaya unification study | D2-R7 | LOW | Deferred to Plan39+ |
| 5 | Team transformation pilot | D4-R3 / D2-R10 | LOW | Cycle 03-4 pilot, 03-5+ full switch |
| 6 | BUG-3 auditor-per-round investigation | 02-12_final D3-R3 | MEDIUM | May be resolved by D1-R1 fix |
| 7 | Doc 13 + Security Model doc | D4-R2 / D2-R9 | P1 | Plan38 target |
| 8 | Must-invoke broader scope | D1-R4 | MEDIUM | After multi-agent implementation |
| 9 | Mesh + API Runtime | D2-R1 | LOW | Phase 6 late |

---

*Prepared by SUNYATA (#0), Cycle 03-1 R4*
*2026-03-24*
