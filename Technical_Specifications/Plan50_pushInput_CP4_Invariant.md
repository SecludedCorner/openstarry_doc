# Plan50 pushInput — CP-1 / CP-2 / CP-3 / CP-4 Invariants + CR-SCK + CR-PARETO (SPEC BINDING)

> **Front-matter**
> - **Cycle**: 03-12
> - **Date**: 2026-04-25
> - **Authors**: TANENBAUM (#20 Microkernel primary) + ASANGA (#8 唯識, CP-4 author) + GUARDIAN (#11 Security allow-list) + RUSSELL (#23 Agent Theory, least-commitment) + LEIBNIZ (#14 Multi-Agent, Pareto + hook ordering) + SYNTHESIST (consolidation)
> - **Binding R3 decisions**: **D-09(b)** B + B' conditional (17/23 super-majority, GUARDIAN allow-list bound) · **D-10(a)** Plan numbering: §4 pushInput = **Plan50**; §5 SHOULD = **Plan49** (UNANIMOUS) · **D-24(a)** CR-PARETO retrospectively ratified (UNANIMOUS 23/0) · **D-25(b)** B' Phase 6 forward-compat score = 4.0/5 · **D-26(b)** CR-SCK SDK-exported TypeScript type + "SHOULD follow or document deviation" (UNANIMOUS 23/0) · **CP-4 MRB-06** TANENBAUM COMPATIBLE with CP-1/2/3 (implicit clause batched with D-09)
> - **Status**: **SPEC BINDING** per R3 D-09(b) + D-10(a) + D-24(a) + D-25(b) + D-26(b) + MRB-06 compound. No implementation this cycle; implementation scheduled for a future cycle per MR-12 ordering.
> - **MR-5 explicit note**: This document references Tenet #10 (分形社會結構) **solely for contextual reference**; **no Tenet #10 status change** is implied. Tenet #10 remains NON-COMPLIANT PENDING per MR-5.
> - **MR-12 ordering**: Plan49 (δ SHOULD supplement) precedes Plan50 (new feature). Plan50 spec is frozen here; implementation waits for Plan49 close-out per MR-12 priority of 回補 over new feature.
> - **Cross-refs**: `deliver/O4_Plan50_pushInput_R4_engineering_spec.md` · `Reference/07_V11_Wording_Binding.md` · `Research_Methodology/09_Rule_75_PreDelivery_Gate_Draft.md`

---

## §1 Background and Status

Plan50 specifies the pushInput source authorisation architecture based on cycle 03-12 §4 R3 decisions. Three candidates (A = full allow-list sole, B = full plugin + opaque sourceContext, C = plugin layered under mandatory Core verify) were evaluated; **Candidate A REJECTED**, **Candidate B PRIMARY**, **Candidate B' VIABLE** (conditional enhancement), **Candidate C DEFERRED** per R3 §4 debate.

### §1.1 Candidate verdicts (R3 D-09(b), 17/23 super-majority)

- **A REJECTED**: 5/5 MR-6 FAIL (Core policy bleed); TANENBAUM + KERNEL + ASANGA + RUSSELL + LEIBNIZ 5-specialty concurrence.
- **B PRIMARY**: 3-way consensus (GUARDIAN + TANENBAUM + KERNEL); GUARDIAN allow-list complementary; TANENBAUM microkernel preserved; KERNEL kernel-level invariants undisturbed.
- **B' VIABLE**: optional enhancement (hook observation + plugin-chain verification); Phase 6 forward-compat score 4.0/5 per D-25(b); implementation at future cycle iff re-confirmation of 5-specialty signatures at that point.
- **C HIGH SECURITY GAP**: deferred (not pre-vetoed; architectural gap requires an F-CP-1 回補 that Plan49 W1 absorbs per MR-10 regardless of candidate choice).

### §1.2 Scope of this spec

This document specifies (a) the four invariants CP-1/2/3/4, (b) the CR-SCK SDK-exported sourceContext recommended-keys type, (c) the CR-PARETO R3 fallback procedure with three-tier red-line taxonomy, and (d) the binding roles of each at Plan50 implementation time. This is a **specification document**; no code is produced this cycle.

---

## §2 Invariant CP-1 — Core Opacity

> **CP-1**: Core knowledge of `sourceContext` payload = **0 bytes**. Core holds `sourceContext?: Record<string, unknown>` as **opaque bytes**; no key enumeration, no schema awareness, no parsing.

### §2.1 Derivation and scope

Originated in TANENBAUM R1 §4-6.8 (cycle 03-11 referenced); reaffirmed and binding at Plan50 scope. CP-1 ensures microkernel purity — Core surface is identity-free with respect to plugin-layer policy. Any plugin's internal sourceContext schema is opaque to the kernel.

### §2.2 Enforcement

- **Dev-side check**: static grep of `packages/core/**` for `sourceContext.*` key access. Result MUST be **zero**.
- **Research-side L2 review**: diff Core surface against Plan47 baseline; confirm only `Record<string, unknown>` remains.
- **Layer discipline**: `sourceContext` type declared in `@openstarry/sdk`, NOT `@openstarry/core`.

---

## §3 Invariant CP-2 — No Binary Decision in Core

> **CP-2**: Core MUST NOT render any boolean verdict on source verification. Policy verdicts live in **plugins**.

### §3.1 Derivation

Originated in TANENBAUM R1 §4-6.8 (cycle 03-11). CP-2 preserves the microkernel principle that policy decisions live outside the kernel; the kernel provides mechanism (event dispatch), not policy.

### §3.2 Enforcement

- **Dev-side check**: type-check — no `verify(): boolean` function on Core surface.
- **Research-side L1'/L2 review**: PR-level architecture review.
- **Hook dispatch**: Core hook dispatch emits the event to observers; the event is NOT gated by hook return values. This preserves CP-2; policy enforcement remains in downstream policy plugins.

---

## §4 Invariant CP-3 — verify/enforce Separation

> **CP-3**: Source attestation is **transport-plugin** responsibility. Policy enforcement is **policy-plugin** responsibility. Core performs neither.

### §4.1 Derivation

Originated in TANENBAUM R1 §4-6.8 (cycle 03-11). CP-3 separates "who attests identity" from "who decides authorisation" — both are plugin responsibilities, in different plugin layers.

### §4.2 Enforcement

- **Dev-side check**: architecture review at PR level.
- **Research-side L3 trace**: verify attestation code path lives in transport plugin; verify enforcement logic lives in policy plugin; Core path is transparent event dispatch.

---

## §5 Invariant CP-4 — Subject Disambiguation Invariant (NEW, cycle 03-12)

> **CP-4 (Subject Disambiguation Invariant)**: `source: string` represents the **transport-identity only**. Agent-identity and capability-identity MUST live in `sourceContext` fields (plugin-controlled; SDK-recommended keys `parentAgentId`, `capabilitySet`, `cert`). Core MUST NOT derive agent-identity from `source` string parsing. Plugins that conflate these (e.g., emitting `source = "agent:X"` and interpreting it as agent-identity without `sourceContext`) are architecturally incorrect and MUST be flagged by ENG-FAB F-9 / Rule #74 doc check.

### §5.1 Authorship and certification

- **Authored by**: ASANGA (#8 唯識), per §4 R2 §6.4 (2026-04-22).
- **Compatibility certified by**: TANENBAUM (#20 Microkernel), per MRB-06 §6.3 — **COMPATIBLE** with CP-1/2/3; **STRENGTHENS** CP-1; **zero Core cost**.
- **R3 adoption**: batched implicitly with D-09(b) under the CP-4 clause ("candidate B primary adopts CP-1/2/3/4 invariant set"), per MRB-06 §6.4. Implicit consensus at R3 §4 vote = 17/23 (same as D-09(b)).

### §5.2 Rationale

**From 唯識 identity-model (ASANGA)**: `source` string conflation treats transport-identity and agent-identity as a single observable signifier. 唯識 analysis requires subject-layer separation: agent-identity is an 意根 projection (subjective reference) distinct from transport-identity (物質 / sensory reference). CP-4 codifies this separation at the architectural level.

**From microkernel discipline (TANENBAUM)**: Identity and ontology decisions happen in plugin/SDK, not kernel. CP-4 reinforces the microkernel principle by prohibiting `source`-string parsing at Core; identity-layer data lives in `sourceContext`, which Core holds opaque per CP-1.

**Zero Core cost**: CP-4 is a discipline, not a runtime check. SDK layer adds ~10–30 documentation LOC (JSDoc + TypeScript comment block) in Plan50 implementation round. ENG-FAB F-9 doc-check extension is minimal.

### §5.3 Interaction with CP-1 / CP-2 / CP-3

| Axis | Effect |
|------|--------|
| **vs CP-1 (Core opacity)** | **STRENGTHENS** — adds explicit prohibition on Core deriving agent-identity from `source` parsing. Core opacity was previously "don't know sourceContext internals"; now also "don't parse source for subject layers". Opacity now covers both fields. |
| **vs CP-2 (no binary in Core)** | **ORTHOGONAL** — CP-4 does not introduce any Core-level verdict. Plugin-level category assignment unchanged. |
| **vs CP-3 (verify/enforce)** | **LAYERED** — CP-3 governs who verifies; CP-4 governs where subject data lives. Orthogonal dimensions; composable. |

### §5.4 Core cost

**Zero Core LOC**. No Core surface change. SDK layer: ~10–30 doc LOC (JSDoc + TypeScript comment block). ENG-FAB F-9 doc-check extension (see `Research_Methodology/10_ENG_FAB_v1.7_Candidate.md` §C.4) adds a plugin-author documentation sub-check at the ENG-FAB audit layer.

### §5.5 Enforcement

- **Dev-side check** (Plan50 implementation round): ENG-FAB F-9 doc-check extension per MRB-06 §6.4 — "Plugin author documents whether `source` string carries agent-identity semantics beyond transport-identity; if so, MUST cross-reference `sourceContext.parentAgentId` in plugin README or JSDoc."
- **Research-side**: Rule #74 L1' doc-check extension; cross-chapter coherence verified at R4 SCRIBE sweep.

### §5.6 TANENBAUM signature (MRB-06 §6.3)

> "CP-4 is COMPATIBLE with CP-1/2/3 and reinforces microkernel discipline. I sign off on its inclusion as the 4th invariant in Plan50 R4 engineering spec §5 Invariants." — TANENBAUM, 2026-04-22.

---

## §6 CR-SCK — sourceContext Recommended Keys (D-26(b), UNANIMOUS 23/0)

Per R3 D-26(b) UNANIMOUS, **CR-SCK** is ratified: the `RecommendedSourceContextKeys` type is **SDK-exported TypeScript type** + **"SHOULD follow or document deviation"** discipline (LEIBNIZ §5.2 position adopted).

### §6.1 SDK-exported TypeScript type (spec frozen here; publication at Plan50 implementation round)

Location: `@openstarry/sdk` (NOT `@openstarry/core`). Plan50 implementation round (deferred) publishes; this spec specifies the shape.

```typescript
// @openstarry/sdk — Plan50 implementation-round publication (spec frozen here per D-26(b))
// SHOULD-follow-or-document per CR-SCK

/**
 * RecommendedSourceContextKeys — plugin-ecosystem Schelling point.
 *
 * Per CP-1, Core does NOT read any of these keys. This type is an
 * SDK-level convention to reduce O(N×M) coordination cost
 * (LEIBNIZ §5.2 analysis, §4 R2 F-§4-R2-2).
 *
 * Plugins SHOULD populate these keys where semantically meaningful;
 * plugins MAY use additional keys; plugins that deviate from these
 * names MUST document the deviation in the plugin README or JSDoc.
 *
 * Per CP-4, agent-identity and capability-identity live here,
 * NOT in `InputEvent.source: string`.
 */
export interface RecommendedSourceContextKeys {
  /** Durable agent identity across sessions. Not transport-identity. */
  readonly parentAgentId?: string;

  /** Capability set authorised for this input. Opaque to Core. */
  readonly capabilitySet?: readonly string[];

  /** Transport-layer certificate or identity token (e.g., JWT). */
  readonly cert?: string;

  /** Transport-layer signature over (source, ts, nonce, capabilityHash). */
  readonly tokenSig?: string;

  /** Cryptographic nonce for replay protection. */
  readonly nonce?: string;

  /** ISO timestamp of source attestation. */
  readonly ts?: string;

  /** Plugin-computed trust score in [0,1]; plugin-specific discipline. */
  readonly trust_score?: number;

  /** Arbitrary additional keys — plugin-specific, MUST document. */
  readonly [key: string]: unknown;
}
```

### §6.2 "SHOULD follow or document deviation" discipline

Per LEIBNIZ §5.2 and D-26(b):

- **SHOULD follow**: plugin authors SHOULD populate the above keys when semantically meaningful for their transport / policy. Deviation is permitted but MUST be documented.
- **Document deviation**: if a plugin uses `{ user_id }` instead of `{ parentAgentId }`, it MUST note this in its README or JSDoc (e.g., "This plugin emits `sourceContext.user_id` in lieu of `parentAgentId` because the HTTP user model is a proper superset of the Schelling `parentAgentId` abstraction.").
- **ENG-FAB F-9 / F-10 check**: doc-audit verifies deviation documentation where applicable (per MRB-06 §6.4 and MRB-08 Rule #75 gate).

### §6.3 NOT Core-enforced (preserves CP-1)

The type is declared in `@openstarry/sdk`, NOT `@openstarry/core`. Core has **zero knowledge** of the key names. Core holds only `Record<string, unknown>`. CP-1 preserved.

### §6.4 Example key schema (plugin-specific extensible)

```typescript
// Example: transport-websocket plugin
const event: InputEvent = {
  source: "ws://hub.example.com:443#connection-abc123",  // transport id (CP-4)
  sourceContext: {
    parentAgentId: "agent-supervisor-42",                // per CP-4 + CR-SCK
    capabilitySet: ["pushInput", "pushInput.file"],
    cert: "<JWT...>",
    tokenSig: "HMAC-SHA256:<...>",
    nonce: "abc-def-123",
    ts: "2026-04-25T12:34:56.789Z",
    trust_score: 0.87,
    // Plugin-specific additions (documented in plugin README):
    ws_connection_id: "abc123",
    ws_origin: "https://supervisor.example.com",
  },
  // ...
};
```

### §6.5 Alternatives considered and rejected

- **"JSDoc-only suggestion"** (lighter-touch): rejected — insufficient LEIBNIZ O(N×M) coordination-cost reduction.
- **"MUST-enforce at dev lint"** (heavier): rejected — would require Core awareness (violates CP-1) or tooling scope beyond Plan50.
- The chosen middle path (SDK-exported + SHOULD-document) preserves CP-1 while reducing coordination cost — unanimous R3 adoption.

---

## §7 CR-PARETO — Pareto-Frontier R3 Fallback (D-24(a), UNANIMOUS 23/0)

Per R3 D-24(a) UNANIMOUS, **CR-PARETO** is retrospectively ratified — pre-declared at R3 opening per MRB-01 and applied throughout R3 §4 voting. Scope: standing R3 procedure for multi-candidate debates with potential red-line exhaustion.

### §7.1 Voting procedure (verbatim from MRB-01)

**Trigger** (any of):
- **T1**: After 2 rounds of R3 primary vote, no candidate achieves ≥ 17/24 (2/3 super-majority).
- **T2**: 24-agent red-line declarations eliminate the entire candidate set under AND-composition of hard red-lines.
- **T3**: SYNTHESIST aggregation produces two equally-ranked composite scores differing by less than 5%.

**Procedure**:

1. **Step P-1 (Dominance filter)**: For each candidate `c` and every 5-specialty lens `L ∈ {TANENBAUM, KERNEL, GUARDIAN, RUSSELL, ASANGA}`, collect the lens-specific rank (1–4). Candidate `c` is *dominated* iff `∃c'` such that `c'.rank_L ≤ c.rank_L` for all `L` and strict inequality for at least one `L`. Remove dominated candidates.
2. **Step P-2 (Pareto frontier)**: The remaining set is the Pareto frontier. Typically 1–2 candidates.
3. **Step P-3 (Frontier voting)**: If `|frontier| = 1`, adopt. If `|frontier| ≥ 2`, hold tie-break vote with **simple majority (≥ 13/24)** over frontier only.
4. **Step P-4 (Empty-frontier escape)**: If `|frontier| = 0`, escalate to Master direct ratification with SYNTHESIST memo summarising cyclic preference structure. No further R3 voting.

**SCRIBE obligation**: Pareto computation table recorded in `discussions/cycle03-XX/SCRIBE_pareto_log.md`. Tie-break deterministic rule for Step P-3: SYNTHESIST + SUNYATA joint recommendation + Master tiebreaker.

### §7.2 Three-tier red-line taxonomy

| Tier | Definition | Aggregation | Override |
|------|-----------|:-----------:|:--------:|
| **Hard** | ZT-1 trigger / Tenet #1–10 violation / MR-6 Core zero violation / MR-3/4/5 violation | AND (any agent's hard red-line → candidate excluded) | **Never** (ZT-1 immutable) |
| **Soft** | S-1..S-4 deviation; architectural concern; forward-compat risk; maintenance burden | Score (count soft red-lines; higher = worse) | Super-majority (≥ 17/24) can override |
| **Specialty** | Lens-specific concern from one of the 5 specialty lenses (TANENBAUM, KERNEL, GUARDIAN, RUSSELL, ASANGA) | Rank per lens (1–4); used in Pareto dominance | Super-majority + specialty-author's written waiver |

### §7.3 Application to §4 D-09

At §4 D-09(b) voting, the three-tier taxonomy resolved the A vs B vs B' vs C choice:
- **A**: 5 hard red-lines (MR-6 violations) → excluded under AND-composition.
- **B**: 0 hard; 0 soft (no S-1 violation); GUARDIAN specialty concern (require allow-list) resolved via explicit condition.
- **B'**: 0 hard; 1 soft (architectural complexity); 5-specialty concurrence achieved at spec level.
- **C**: 0 hard; HIGH specialty concern (TANENBAUM microkernel) + F-CP-1 回補 required; deferred.

Result: **B primary** + **B' viable** + **C deferred** + **A rejected**. See `deliver/O4_Plan50_pushInput_R4_engineering_spec.md` §3 for full detail.

---

## §8 Plan50 Engineering Spec Summary (§8 Invariants list)

The Plan50 implementation-round engineering spec §8 Invariants list is:

1. **CP-1** Core Opacity — Core knowledge of `sourceContext` = 0 bytes.
2. **CP-2** No Binary Decision in Core — policy verdicts in plugins only.
3. **CP-3** verify/enforce Separation — attestation in transport plugin; enforcement in policy plugin; Core neither.
4. **CP-4** Subject Disambiguation Invariant — `source: string` is transport-identity only; agent/capability identity lives in `sourceContext`.

All four are **binding on implementation-round dev-team** and on **any future plugin author** working with pushInput. CP-1/2/3 carried forward from cycle 03-11; CP-4 newly introduced cycle 03-12 (ASANGA-authored, TANENBAUM-signed COMPATIBLE per MRB-06).

---

## §9 Interactions and Dependencies

| Reference | Relationship to Plan50 spec |
|-----------|-----------------------------|
| **MR-6 Core zero policy** | All four invariants preserve MR-6; Plan50 adds **zero Core policy constants** |
| **Rule #74 L1'** | CP-4 requires plugin-author doc; Rule #74 L1' verifies the doc |
| **Rule #75 Pre-Delivery Gate** (draft; `Research_Methodology/09`) | Plan50 implementation round MUST attest Rule #75 §75.3.1–§75.3.5 |
| **ENG-FAB v1.7** (candidate; `Research_Methodology/10`) | F-9 sub-check (i)/(ii)/(iii) + F-10 Pre-Delivery Gate Verified apply to Plan50 delivery |
| **MR-5** | Tenet #10 reference only; no status change implied |
| **MR-12** | Plan49 (δ SHOULD) precedes Plan50; Plan50 implementation waits for Plan49 close-out |
| **MR-10** | F-CP-1 回補 absorbed into Plan49 W1 regardless of candidate choice (ASANGA §6.6 subject hygiene); §4 R2 F-§4-R2-8 |
| **D-09(b)** | B primary + B' viable; GUARDIAN allow-list condition bound at spec level |
| **D-10(a)** | §4 pushInput = Plan50; §5 SHOULD supplement = Plan49 |

---

## §10 Dissent (MR-11 preservation)

- **D-09 (a) conservative minority** — 6 dissent votes favouring candidate A (allow-list sole). Rejected: 5/5 MR-6 FAIL. Dissent preserved in R3_decision_log §7.
- **D-09 (d) C forward-ready** — 0 votes; C DEFERRED per §4 R2 consensus.
- **D-10 alternatives (b) Plan49a/b split / (c) both in Plan49** — rejected unanimously; (a) Plan50 naming for cohesion.
- **D-24 CR-PARETO objection to retrospective ratification** — 0 dissent; UNANIMOUS 23/0.
- **D-25 B' forward-compat 4.0/5** — UNANIMOUS 23/0.
- **D-26 CR-SCK alternatives (JSDoc-only / MUST-enforce)** — both rejected unanimously; SDK-exported + SHOULD-document chosen.

### §10.1 R4 obligation per D-09 (a) minority (conservative)

Per MR-11 minority-voice preservation: the GUARDIAN allow-list (§3.2.2 in Plan50 implementation round) plus TANENBAUM + KERNEL hook-context purity conditions bind complexity risk. Conservative dissent is acknowledged; majority position is adopted with GUARDIAN allow-list as an explicit MR-11 minority-voice concession.

---

## §11 Status and Implementation Timing

**Status**: **SPEC BINDING** — specification frozen this cycle; implementation deferred to a future cycle per MR-12.

- **Plan50 implementation prerequisites**:
  1. Plan49 (δ SHOULD supplement) close-out PASS.
  2. Plan48 → Plan49 sequence complete per MR-12.
  3. Re-confirmation at implementation-cycle entry of the 5-specialty signatures (TANENBAUM + KERNEL + GUARDIAN + RUSSELL + ASANGA) against the then-current SDK hook infrastructure.
- **Plan50 implementation targets**: 500–800 LOC prod (pushInput dedicated plan) + SDK-exported `RecommendedSourceContextKeys` (~50–100 LOC) + ENG-FAB F-9 CP-4 sub-check extension + Rule #74 L1' doc sync.
- **Master Ratification Request**: Batch 9 (`Master_Ratification_Plan50_Scope.md`) includes the CP-1/2/3/4 invariant set + CR-SCK type export + CR-PARETO procedure reference.

---

## §12 MR-5 Explicit Statement (Tenet #10 reference, no status change)

This document references Tenet #10 (分形社會結構, fractal social structure) only as **contextual architecture reference** — the Plan50 pushInput source-authorisation architecture is forward-compatible with a future fractal multi-agent topology via the plugin-layer `sourceContext` mechanism. **No Tenet #10 status change is implied by Plan50 spec ratification or implementation**. Tenet #10 remains NON-COMPLIANT PENDING per MR-5 until Phase 6 completion.

Per ZT-2, the endpoint is **10/0/0★** (Phase 6 completion; Plan48–Plan55 roadmap per `MEMORY.md` Master guidance). 9/0/1★ (transitional) is not the final state; Plan50 is one contributing spec toward the Phase 6 roadmap but does **not** itself cause Tenet #10 reclassification.

---

## §13 Author Attribution and R3 Vote Summary

- **Primary architectural lenses**: TANENBAUM (#20) + ASANGA (#8) + GUARDIAN (#11) + RUSSELL (#23) + LEIBNIZ (#14).
- **CP-4 authorship**: ASANGA.
- **CP-4 compatibility certification**: TANENBAUM (MRB-06 §6.3).
- **CR-SCK design**: LEIBNIZ (coordination cost analysis) + TANENBAUM (microkernel discipline).
- **CR-PARETO procedure**: LEIBNIZ + SYNTHESIST (MRB-01).
- **Consolidation**: SYNTHESIST (#1).
- **Coordination**: SUNYATA (#0).
- **R3 vote summary**:
  - D-09(b) B + B' conditional: 17/23 super-majority (6 dissent → A conservative)
  - D-10(a) Plan50/Plan49 naming: UNANIMOUS 23/0
  - D-24(a) CR-PARETO: UNANIMOUS 23/0
  - D-25(b) B' 4.0/5: UNANIMOUS 23/0
  - D-26(b) CR-SCK: UNANIMOUS 23/0
  - CP-4 MRB-06 batched with D-09: implicit 17/23
