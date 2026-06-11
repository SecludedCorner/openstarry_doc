---
title: Plan54 — AC-9 Sub-Agent Composition Candidate A BINDING Engineering Spec
author: LEIBNIZ (#14) + RUSSELL (#23) + TANENBAUM (#20) + ARCHIMEDES (#16) + SUNYATA (#0) + SYNTHESIST (#1)
date: 2026-04-28
cycle: 03-16
status: CANDIDATE (pending Master Ratification Batch 13 #1)
authority: research-team (R4 final draft); Master (ratification)
supersedes: null
language: en
cross_refs:
  - research record/cycle03-16/deliver/O1_AC9_Plan54_final.md
  - research record/cycle03-16/deliver/Master_Ratification/Batch_13_Request.md §3.1
  - research record/cycle03-16/openstarry_doc/Technical_Specifications/Plan54_AC9_Binding.tw.md
  - research record/cycle03-16/R3/R3_decision_log.md (D-01 / D-04 / D-11 / D-13 / D-18)
  - research record/cycle03-14/openstarry_doc/Technical_Specifications/Plan52_pushInput_CandidateB.md (Plan52 isomorph baseline)
  - research record/cycle03-15/openstarry_doc/Reference/11_Rule_78_TW_Translation.md
  - research record/cycle03-15/openstarry_doc/Research_Methodology/16_F_15_v3_Third_Tier_Amendment.md
binding_until: Master Ratification Batch 13 close
prefix_discipline: verified | inferred | speculative
---

# Plan54 — AC-9 Sub-Agent Composition Candidate A BINDING Engineering Spec

## 1. Status

**CANDIDATE** at cycle 03-16 R4 close. Upgrades to **BINDING** upon Master Ratification Batch 13 Item #1. Effective **forward-only** from cycle 03-16 R4 close onward; pre-cycle-03-16 Core surface is **grandfathered unmodified** per MR-12 既有不破壞 and MR-6 Core 零.

**Landing pattern**: Phase 6 第二棒 — first formal AC-9 sub-agent composition spec; Plan52 isomorph (pushInput Candidate B v0.50.0-alpha shipped cycle 03-14 R4 close).

**TW sibling**: `Technical_Specifications/Plan54_AC9_Binding.tw.md` MANDATORY same-PR-strict per Rule #78 §78.5 forward-only + §78.8 reflexive + cycle 03-16 R3 D-09 UNANIMOUS.

## 2. R3 Provenance

- **D-01 (Round A apex; AC-9 candidate vote)** — Stage 1 (proceed vs D defer) = 19/4 reject D; Stage 2 plurality A=14, B=4, C=5; Stage 2 runoff (A vs non-A combined) = **16/7 — A ratified at ⅔ super-majority**. DSS-CY16-01 9-vote total minority preserved per MR-11 (4 D-defer + 5 C orchestrator + Stage 2 4-vote B hybrid carry).
- **D-04 (Round A; MAX_SPAWN_DEPTH default)** — `MAX_SPAWN_DEPTH = 4` with operator override, **17/6**. DSS-CY16-03 6-dissent (KERNEL + TANENBAUM + GUARDIAN + 3 others preferring 3-cap) preserved.
- **D-11 (Round B; numbering)** — Plan54 integer numbering vs Plan52a sub-numeral, **19/4**. DSS-CY16-NUM 4-vote chain-consistency minority preserved.
- **D-13 (Round C; LOC ceiling)** — 800 prod + 500 test, **UNANIMOUS 23/0**.
- **D-18 (Round D; philosophy annotations)** — NAGARJUNA Madhyamaka + PENROSE consciousness ADOPT non-binding doc-only, **18/5**. DSS-CY16-06 5-vote philosophical-drift minority preserved.
- **MRB-§1-01 RESOLVED** — §一 (AC-9) MUST precede §四 (plugin-loader) sequencing pin LOCKED.
- **MRB-§1-02 RESOLVED** — Plan54 numbering propagation via SCRIBE + coordinator G5 4-mirror sync.

## 3. Architecture Summary (Candidate A — Full Plugin, Plan52 Isomorph)

### 3.1 ε-Surface Restated (Plan52 inheritance)

Core surface delta for Plan54 implement-round:

```
+ 0 new fields (Plan52 sourceContext?.parentAgentId already present)
+ 0 Zod passthrough additions (Plan52 z.record(z.unknown()).optional() already present)
NOT added (would be MR-6 FAIL):
  - ISpawner interface
  - SubAgent type
  - spawn(): SubAgentHandle method
  - {SPAWNED, ACTIVE, COMPLETED, ABORTED, ORPHANED} enum
  - MAX_SPAWN_DEPTH constant in Core (Candidate B was REJECTED)
  - lifecycle hook registry in Core
Aggregate Core delta vs Plan52 baseline: 0 fields, 0 const → ε-surface PASS (strict equality).
```

### 3.2 Plugin as Composition Runtime

The authoritative AC-9 sub-agent composition runtime is the **`agent-composition` plugin** (working name; final name fixed at cycle 03-17 R0). The plugin lives at `apps/runner/src/agent-composition/` (peripheral; NOT under `@openstarry/core`). The plugin:

1. **Exports** SDK-layer factories via `@openstarry/sdk`:
   - `spawnChild(req: SpawnChildRequest): SpawnChildResponse`
   - `registerLifecycleHook(state: LifecycleState, handler: LifecycleHandler): void`
   - `enforceSpawnDepth(parentDepth: number, maxDepth: number): void`
2. **Attests** parent-agent identity using the **Plan52 HMAC-SHA256 signed-token discipline** (CV-02/04/05 reaffirm; no new crypto).
3. **Constructs** child InputEvent emissions through the existing pushInput pipe — **NO new pipe**.
4. **Emits** through the same Core forwarding pipe established by Plan52. **Core forwards bytes; Core does not read `sourceContext`.**

### 3.3 Plan52 Isomorphism (Verbatim Reuse Map)

| Plan52 contract | AC-9 Plan54 reuse |
|-----------------|------------------|
| `InputEvent.sourceContext?: Record<string, unknown>` | reused verbatim; child inherits |
| `RecommendedSourceContextKeys` (SDK-exported) | extended forward-only (MR-12) with `parentAgentId`, `spawnDepth`, `spawnId` |
| HMAC-SHA256 default tokenSig | reused verbatim (CV-02) |
| Algo-prefix mandatory (e.g., `hmac-sha256:`) | reused verbatim (CV-04) |
| Local-CLI MAY-omit tokenSig | reused verbatim (CV-05) |
| Nonce ≥ 16 bytes entropy | reused verbatim (CV-03) |
| Replay cache TTL ≥ key-rotation overlap | reused verbatim (CV-06) |
| deepFreeze recursive immutability | reused verbatim |
| Tri-party MR-6 audit (TANENBAUM + KERNEL + GUARDIAN) | extended to AC-9 |
| F-13 hook dispatch verifiability | extended to AC-9 spawn / lifecycle hooks |
| F-14 external resource baseline | extended to AC-9 LLM-token-per-child + spawn quota |
| F-15 v3 reflexive scope | extended to this spec doc + plugin code |

## 4. Composition Primitives (Plugin-Layer; NOT Core)

### 4.1 Spawn Primitive

`SpawnChildRequest` Zod schema lives in `@openstarry/sdk`:
- `parentAgentId: string` — signed-token-attested per Plan52 §3.4
- `parentTokenSig: string` — algo-prefix mandatory (CV-04)
- `childAgentSpec: { capability: string; config: Record<string, unknown> }`
- `spawnDepth: number` — child = parent + 1
- `spawnId?: string` — UUID; AC-9 plugin generates if omitted
- `nonce: string` — ≥ 16 bytes entropy per CV-03

`SpawnChildResponse`: `{ success, childAgentId, childTokenSig, state: 'spawned', reason? }`.

### 4.2 Lifecycle Primitive

State machine (plugin string literals; NOT Core enum):
```
spawned → active → {completed, aborted, orphaned}
```

Lifecycle hooks: `onSpawned` / `onActive` / `onCompleted` / `onAborted` / `onOrphaned` (default 30s grace window). Dispatched via Plan51 hook-registry (cycle 03-15 ratified module; first-shipping cycle 03-16 W2-R16 verification per MRB-§1-01 sequencing pin). F-13 hook dispatch verifiability applies.

### 4.3 Boundary Primitive

1. **Cryptographic boundary** — each spawn re-signs `tokenSig` (HMAC-SHA256, fresh nonce, child-specific subject); parent's tokenSig NOT forwarded.
2. **Semantic boundary** — child inherits `parentAgentId` in `sourceContext` for downstream policy plugins.
3. **Lineage integrity** — recursive walk of `sourceContext.parentAgentId` (depth ≤ MAX_SPAWN_DEPTH); opaque to Core (MR-6 invariant).
4. **Capability containment** — plugin policy decides; AC-9 Plan54 provides mechanism only.

### 4.4 IPC Contract

Reuses pushInput pipe established by Plan52. **NO new pipes, NO new transports, NO new event types.**

## 5. ε-Surface MR-6 Reflexive Audit (Tri-Party Sign-off)

At v0.52.0-alpha tag (cycle 03-17 implementation close):
- **TANENBAUM (#20)** — verifies no Core file (`packages/core/src/**`) introduces `agent`, `spawn`, `subagent`, `parent`, `child`, `lifecycle`, `orchestrator` lexical tokens beyond pre-existing Plan52 baseline.
- **KERNEL (#10)** — verifies Core surface diff vs `v0.50.0-alpha` MUST equal 0 lines for AC-9-related changes.
- **GUARDIAN (#11)** — verifies HMAC-SHA256 signed-token discipline preserved end-to-end; replay cache properly inherits Plan52 TTL ≥ key-rotation overlap (CV-06); no plaintext parentAgentId leakage in logs.

Each persona issues a signed verdict committed to `discussions/cycle03-17/MR6_audit_log.md`. Automated CI gate (Plan52 §5 inheritance, extended): block PR + escalate to tri-party manual review on Core surface diff > 0.

Failure disposition: any tri-party FAIL → Plan54 v0.52.0-alpha tag NOT issued; cycle 03-17 R3 reopens AC-9 architecture review; Master letter §五 escalation per Rule #75 §75.X-equivalent halt policy.

## 6. HMAC-SHA256 + Algo-Prefix + Local-CLI MAY-Omit (CV-02/04/05)

- **Default algorithm** HMAC-SHA256 (CV-02 reaffirm); Ed25519 documented but NOT default for AC-9 Plan54 (continuity discipline; ZT-1).
- **Algo-prefix mandatory** (CV-04): `tokenSig = "<algo>:<encoding>:<value>"`, e.g., `"hmac-sha256:hex:7a3b4c2e..."`. Validation in plugin (NOT Core); missing/unrecognised prefix → `reason: "tokenSig_algo_prefix_missing"`.
- **Local-CLI MAY-omit** (CV-05): for sub-agents in same OS process; plugin SHOULD rely on process-local UID + memory-address-space isolation. Does NOT extend to cross-process or capability-restrict plugins.
- **Replay cache TTL** ≥ key-rotation overlap (CV-06; default 24h; configurable). At v0.52.0-alpha, AC-9 plugin MUST share replay cache with Plan52 transport plugins (single-process) OR document coordinated cache topology (multi-process). Forward-only (MR-12) — existing Plan52 deployments NOT retrofitted.
- **Nonce ≥ 16 bytes entropy** from CSPRNG (CV-03 reaffirm).

## 7. MAX_SPAWN_DEPTH = 4 + Operator Override (D-04)

### 7.1 Where the Constant Lives

```
apps/runner/src/agent-composition/config.ts:
  export const MAX_SPAWN_DEPTH_DEFAULT = 4;
```

Plugin-internal const, **NOT** Core. Candidate B (Core const) was REJECTED at R3 D-01.

### 7.2 Operator Override

Precedence: per-spawn > config file > env var > default.
- `OPENSTARRY_MAX_SPAWN_DEPTH=<int>` (range 1..16; outside-range falls back + WARN log).
- `agent-composition.config.json` field `maxSpawnDepth: number`.
- `SpawnChildRequest.overrideMaxDepth?: number` (deferred to Plan55+ requires operator-issued grant token).

### 7.3 Audit Logging

Every override emits structured INFO log (timestamp / source / overridden value / default / operator UID). F-15 v3 reflexive audit applies.

### 7.4 Failure Mode

`spawnDepth + 1 > MAX_SPAWN_DEPTH` →
```
SpawnChildResponse:
  success: false
  reason: "max_spawn_depth_exceeded"
  state: 'aborted'
```
F-16 SHOULD-initial structured error (CV-09 reaffirm; FORBIDDEN-phrasings 6 patterns binding — no "race condition" / "weird state" / "shouldn't happen" / "TODO fix later" / "transient" / "harmless").

## 8. Quota / Resource Limits (Plugin-Layer)

- `MAX_ACTIVE_SUBAGENTS_GLOBAL = 64` (env override 1..1024).
- `MAX_ACTIVE_SUBAGENTS_PER_PARENT = 8`.
- `DEFAULT_LLM_TOKEN_BUDGET_PER_SPAWN = 32_000` tokens (telemetry hooks only; downstream plugin enforces per F-14).
- `ORPHAN_GRACE_WINDOW_MS = 30_000` (parent terminates → child enters orphaned state; 30s flush before forced cleanup).
- Backpressure: `success: false`, `reason ∈ {spawn_capacity_exhausted, parent_quota_exhausted, global_quota_exhausted}`. Retry semantics = plugin-policy (NOT mandated by AC-9 spec).

## 9. LOC Budget (D-13 UNANIMOUS 23/0)

- **Production code: 800 LOC ceiling** (across `apps/runner/src/agent-composition/**` and `@openstarry/sdk` AC-9 schema additions).
- **Test code: 500 LOC ceiling** (unit + integration + cross-OS CI fixtures).

Indicative allocation (R1 §1.3 baseline + R2 refinement):

| Module | Indicative prod LOC | Indicative test LOC |
|--------|--------------------:|--------------------:|
| `agent-composition/spawn.ts` | ~150 | ~120 |
| `agent-composition/lifecycle.ts` | ~120 | ~100 |
| `agent-composition/boundary.ts` | ~80 | ~60 |
| `agent-composition/quota.ts` | ~80 | ~60 |
| `agent-composition/config.ts` | ~40 | ~20 |
| SDK schema additions | ~80 | ~80 |
| Plan51 hook-registry integration | ~60 | ~40 |
| Misc (logging, errors, types) | ~70 | ~20 |
| **Total** | **~680** | **~500** |

Ceiling provides ~120 prod buffer (15%) for cycle 03-17 R0/R1 refinement. Test ceiling exact (500 LOC) — strict.

LOC accounting per cycle 03-16 D-12: primary measure = Δ vs sum-of-medians indicative; secondary = Δ vs Master letter upper 800. Cycle 03-17 delivery_report.md MUST report both.

## 10. F-13 / F-14 / F-15 v3 + Rule #74 + Rule #75 Reflexive

- **F-13 Hook Dispatch Verifiability** — runtime probe at v0.52.0-alpha smoke test triggers spawn → verifies all 5 lifecycle hooks dispatched; `cat=undefined` auto-FAIL (zero-tolerance).
- **F-14 External Resource Baseline** — child sub-agent LLM calls MUST increment provider token counter ≥1 per active spawn (zero-baseline auto-FAIL); `active_subagent_count` gauge increment/decrement; `spawnDepth` gauge.
- **F-15 v3 Reflexive Scope** — front-matter 7 fields ✓; cross_refs to R1+R2+R3+Plan52 ✓; epistemic prefix `BINDING (R4 final; pending Master Ratification Batch 13 #1)` ✓.
- **ENG-FAB v1.8 (48 items) all-MUST PASS pre-condition** at v0.52.0-alpha tag. Per Rule #75 §75.X cross-OS partial-PASS halt policy (D-05 cycle 03-16): cross-OS asymmetric PASS → tag NOT issued.
- **Rule #74 L1' (Code + Doc Equivalence)** — 5 sub-checks per cycle 03-17 delivery_report.md (CV-07 reaffirm).
- **Rule #78 + F-15 v3** — TW sibling MANDATORY same-PR-strict per D-09 UNANIMOUS.

## 11. plugin-loader 5-Criterion Dependency Map (D-02)

Plan54 spec stable = **C1 GREEN** for plugin-loader cycle 03-17 evaluation (`Reference/13_Plugin_Loader_Cycle03_17_Evaluation_Criteria.md`). C2-C5 evaluated cycle 03-17 R0 per matrix Row 1-5 + DSS-A4 honour clause weighing. **§一 (AC-9) MUST precede §四 (plugin-loader) sequencing pin LOCKED** per MRB-§1-01.

## 12. Implementation Order (cycle 03-17 Dev Kickoff)

| Order | Plan | Status entering cycle 03-17 |
|:-----:|------|----------------------------|
| 1 | Plan49 gear-arbiter retrofit | landed (cycle 03-13) |
| 2 | Plan50 σ_regime in-place revise | landed (cycle 03-13) |
| 3 | Plan52 pushInput Candidate B | shipped v0.50.0-alpha |
| 4 | Plan51 4-modules | shipping (cycle 03-16 W2-R16 verification) |
| 5 | **Plan54 AC-9 (THIS SPEC)** | **R4 BINDING; cycle 03-17 Dev kickoff** |

Plan54 implementation tag target: `v0.52.0-alpha`.

Plan53 reserved (per cycle 03-14 D-§1-02 DSS-1 carryover; Plan52 B route success → never invoked → stays reserved). **Plan54 integer numbering per D-11 19/4** (sub-numeral signals errata-like emergency-insert per cycle 03-7 Plan-32.5 precedent; AC-9 is full new Plan).

Phase 6 forward roadmap (informational; no MUST claim per MR-5 hard): Plan55 Multi-IVolition (cycle 03-18 spec target) / Plan56 VasanaEngine (cycle 03-20) / Plan57 Mesh (cycle 03-22) / Plan58 API Runtime (cycle 03-23) / Plan59 Blackboard-Alaya (cycle 03-24). 7/7 functional landing target cycle 03-25 conservative.

## 13. Philosophy Annotations (D-18 ADOPT non-binding doc-only, 18/5)

### 13.1 NAGARJUNA Madhyamaka — Saṃvṛti-Satya / Paramārtha-Satya

The lifecycle states `{spawned, active, completed, aborted, orphaned}` are **conventional truth** (saṃvṛti-satya) — useful-fictions enabling plugin coordination. NOT ultimate truth (paramārtha-satya): no svabhāva (intrinsic essence) to "spawn" — what exists are momentary informational events at particular processing junctures, conventionally labeled. State machine = coordination convenience, NOT metaphysical claim about agents.

Operationally: doc-only. Plugin authors MAY or MAY NOT internalize; correctness does NOT depend on philosophical absorption. F-15 v3 third-tier prefix discipline acknowledged.

(*DSS-5-philosophical carryover preserved per MR-11: paramārtha-satya critique persists; this annotation absorbs saṃvṛti-satya half only.*)

### 13.2 PENROSE Consciousness Disclaimer

AC-9 Plan54 makes **no metaphysical claim** about sub-agent inner experience, qualia, or phenomenal consciousness. Composition primitives are **purely functional** — information flow + resource control, NOT subjective experience. Whether sub-agents are "conscious" is **outside engineering scope**. F-15 v3 epistemic level: `DOC-ONLY-PHILOSOPHY`.

### 13.3 Disposition

Both annotations committed to `Reference/` (not `Technical_Specifications/`). Referenced from this spec but do NOT bind implementation. Cycle 03-17 Dev MUST NOT treat as architectural constraints.

## 14. Compliance Audit (R4 Snapshot)

| Constraint | Status | Evidence |
|------------|:------:|----------|
| MR-2 + MR-4 (Tenet text untouched) | PASS | 0 Tenet wording change |
| MR-5 hard (Tenet #10 status no change) | PASS | §1 explicitly disclaims; §12 7/7 framing informational only |
| MR-6 (Core 零) | PASS | §3.1 ε-surface delta vs Plan52 baseline = 0 lines |
| MR-7 | PASS | No premature COMPLIANT claim |
| MR-9 (no preemptive MUST) | PASS | F-16 SHOULD-initial unchanged; CV-09 reaffirm |
| MR-10 + MR-12 (forward-only) | PASS | §3.3 forward-only; §6 existing Plan52 deployments NOT retrofitted |
| MR-11 (dissent preservation) | PASS | §16 — 24 entries (10 cycle 03-16 NEW + 9 carryover + 5 absorbed/internal) |
| MR-13 standby | PASS | (A) Full retained; not invoked |
| ZT-1 (ten-tenet) | PASS | No tenet text or status modified |
| ZT-2 (endpoint 10/0/0★) | PASS | 9/0/1★ ACTIVE preserved |
| ZT-3 (control-range) | PASS | σ_regime per Rule #77 unchanged; W2-R16 N=5 < 10 not triggering SPC |
| Rule #74 L1' | PASS | §10 — 5 sub-checks delivery_report obligation |
| Rule #75 §75.X | PASS | §10 cross-OS partial-PASS halt referenced (D-05) |
| Rule #78 TW parity | PASS | §1 — TW sibling MANDATORY same-PR-strict (D-09 + MRB-§5-§78-A) |
| ENG-FAB v1.8 (48) MUST | PRE-CONDITIONED | §10 — all 48 MUST PASS at v0.52.0-alpha |
| F-13 / F-14 / F-15 v3 reflexive | PASS | §10 |
| F-16 SHOULD initial | PASS | §7.4 structured error; no preemptive MUST upgrade (CV-09); FORBIDDEN-phrasings 0 hits |
| Wording sweep | PASS | σ / V11 / "CONDITIONAL PASS" only inside MR-6 audit context |
| GP-coherence gate | PASS | front-matter compliant; sibling cross_refs bidirectional |

## 15. Implementation Handoff (cycle 03-17 R0)

R0 dispatch inputs: this spec + plugin-loader 5-criterion (`Reference/13`) + Plan51 first-shipping retrospective + W2-R16/R17 verdicts + F-15 v3 sub-mech 2/3/4-table/5 Plan-spec scope (D-03 ratified) + MR6 audit_log.md scaffold + TW siblings MANDATORY same-PR-strict at v0.52.0-alpha (D-09 + MRB-§5-§78-A; Rule #78 §78.5 + §78.8).

R0 dispatch outputs: AC-9 plugin Plan-spec (Wave breakdown ≤4 Waves per S-1) + cycle 03-17 R0 orientation doc + subagent dispatch matrix.

Risk register (cycle 03-17 entry): MR-6 ε-surface drift HIGH (mitigation: tri-party + automated CI) / Plan51 hook-registry W2-R16 FAIL MED (MRB-§1-01 halts cycle 03-17 Dev) / LOC overshoot LOW (15% buffer) / Cross-OS divergence MED (Rule #75 §75.X halt) / ENG-FAB v1.8 PARTIAL HIGH (F-13/14/15 reflexive checks) / Sub-agent quota race MED (property tests + 30s grace).

## 16. Dissent Preservation (per MR-11 verbatim)

24 entries preserved verbatim per MR-11. Cycle 03-16 R3 NEW = 10 (DSS-CY16-01..09 + DSS-CY16-NUM + DSS-CY16-CHAIR-1) + carryover = 9 + absorbed/closures = 5. **S6 ≥20 floor PASS**.

Highlights (full text in `research record/cycle03-16/deliver/O1_AC9_Plan54_final.md` §16):

- **DSS-CY16-01** (D-01; 9 votes total) — "Candidate A full plugin Plan52 isomorph wins; C orchestrator-as-plugin LEIBNIZ multi-agent symmetry; D defer DSS-1 carryover (LEIBNIZ + RUSSELL B+B' temporal asymmetry)"
- **DSS-CY16-03** (D-04; 6 votes) — "MAX_SPAWN_DEPTH=3 with documented use-case scope preferred; resource isolation tighter; 4 may invite unintended deep-recursion"
- **DSS-CY16-NUM** (D-11; 4 votes) — "Plan52a sub-numeral preserves chain consistency; Plan54 jumps Plan53 reserved"
- **DSS-CY16-06** (D-18; 5 votes) — "Philosophical-drift concern (cycle-03-13 D-20 6-vote precedent); adoption marginal value vs documentation overhead"
- **DSS-CY16-CHAIR-1** (D-01 Stage 2 runoff; 4-vote B carry) — "Candidate B hybrid + 1-2 Core consts offered slippery-slope risk balance; MR-6 Core 零 pristine for A but B explicit at-bounds-but-controlled"
- **DSS-1** (cycle 03-14 carryover) — LEIBNIZ + RUSSELL B+B' temporal asymmetry; preserved verbatim; AC-9 Plan54 ratified ≠ closure of DSS-1 (generic concern persists for Phase 6 5/7 sequencing).

---

*Plan54 — AC-9 Sub-Agent Composition Candidate A — CANDIDATE pending Master Ratification Batch 13 #1*
*Authors: LEIBNIZ + RUSSELL + TANENBAUM + ARCHIMEDES + SUNYATA + SYNTHESIST*
*7 R3 D-items absorbed (D-01 / D-02 / D-03 / D-04 / D-11 / D-13 / D-18) + 5 MRB resolved + 20 CV reaffirmed + 24 dissent preserved (S6 ≥20 PASS)*
*MR-6 Core 零 ε-surface (Plan52 isomorph; 0 new fields beyond Plan52 baseline) / MR-5 hard Tenet #10 status unchanged / endpoint 10/0/0★ unchanged per ZT-2*
*Phase 6 progress: 2/7 functional spec (post cycle 03-17 implementation); 7/7 target cycle 03-25 conservative*
*TW sibling: `Plan54_AC9_Binding.tw.md` MANDATORY same-PR-strict per Rule #78 §78.5 + §78.8 + D-09 UNANIMOUS*
