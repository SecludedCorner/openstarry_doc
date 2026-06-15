---
title: plugin-loader Cycle 03-17 5-Criterion + 5×4 Decision Matrix Evaluation Framework
author: SUSSMAN (#22) + DARWIN (#6) + ARCHIMEDES (#16) + GUARDIAN (#11) + LEIBNIZ (#14) + RUSSELL (#23)
date: 2026-04-28
cycle: 03-16
status: CANDIDATE (pending Master Ratification Batch 13 #2 — APPROVE evaluation framework)
authority: research-team (R4 final draft); Master (procedural ratification)
supersedes: cycle 03-15 R3 informally-positioned 4-criterion structure (extended to 5 with explicit C5)
language: en
cross_refs:
  - research record/cycle03-16/deliver/O4_Plan51_retro_pluginloader_final.md §11 + §12 + §13
  - research record/cycle03-16/deliver/Master_Ratification/Batch_13_Request.md §3.2
  - research record/cycle03-16/openstarry_doc/Technical_Specifications/Plan54_AC9_Binding.md (C1 conditioning input)
  - research record/cycle03-16/R3/R3_decision_log.md (D-02 / D-19)
  - research record/cycle03-15/openstarry_doc/Technical_Specifications/Plan51_Zod_Gate_Binding.md (Plan51 first-shipping retrospective baseline)
  - research record/cycle03-13/openstarry_doc/Technical_Specifications/Plan49_Zod_Gate_Baseline.md (MRB-06 plugin-loader 0-site greenfield origin)
binding_until: cycle 03-21 R0 disposition (MRB-06 close-with-rationale OR re-ratify-as-open-gap per D-19)
prefix_discipline: verified | inferred | speculative
---

# plugin-loader Cycle 03-17 5-Criterion + 5×4 Decision Matrix Evaluation Framework

## 1. Status

**CANDIDATE** at cycle 03-16 R4 close. Upgrades to **APPROVED evaluation framework** (procedural binding for cycle 03-17 R0 invocation) upon Master Ratification Batch 13 Item #2.

**Application scope**: cycle 03-17 R0 invokes the 5-criterion + 5×4 matrix to determine whether plugin-loader (5th Plan51 module candidate; DEFERRED at cycle 03-15 D-§5-A 9/11/3 vote) should re-enter scope for Plan-spec dispatch, or remain DEFERRED. Cycle 03-21 R0 invokes MRB-06 bilateral disposition logic per D-19 (close-with-rationale OR re-ratify-as-open-gap).

## 2. R3 Provenance

- **D-02 (Round A apex; §4 D-§4-B + F-§4-R2-07 RUSSELL agent rationality)** — explicit **C5 5th criterion** (DSS-A4 procedural-caution honour clause), **18/5**. DSS-CY16-02 5-vote minority preserved verbatim per MR-11 (SUSSMAN + DARWIN + LINNAEUS + 2 others; row-1-amendment to existing 4-criterion preferred over explicit C5 — abstraction parsimony argument).
- **D-19 (Round D; MRB-06)** — pre-position both options for cycle 03-21 R0 to invoke (close-with-rationale vs re-ratify-as-open-gap bilateral structure), **UNANIMOUS 23/0**.
- **CV-19** — DSS-3 LEIBNIZ ossification CLOSED-by-execution at cycle 03-16 §4 D-§5-D trigger met, **UNANIMOUS reaffirm with NAGARJUNA Madhyamaka caveat**.
- **MRB-06 RESOLVED** — cycle 03-13 Plan49 spec MRB-06 plugin-loader 0-site greenfield disposition pre-positioned per D-19.

## 3. Background — Why a 5-Criterion Framework

### 3.1 Cycle 03-15 D-§5-A Outcome (carryover)

Plan51 cycle 03-15 ratified 4-of-5 modules (D-§5-A 17/6) — WebSocket / checkpoint-store / event-bus / hook-registry shipped at v0.51.0-alpha. The 5th module (plugin-loader) DEFERRED 9/11/3 (推薦/反對/棄權). DSS-A4 6-dissent (LEIBNIZ + KNUTH + SUSSMAN + 3 others; "Plan51 entire 應 defer cycle 03-17+ post-AC-9; 4-module commit 過早") preserved verbatim per MR-11.

### 3.2 DSS-A4 Three Rationale Branches

The DSS-A4 6-dissent has three rationale branches:
1. **Procedural-caution branch** ("4-module commit 過早") — partially refuted by Plan51 first-shipping STRUCTURAL PASS at v0.51.0-alpha.
2. **Post-AC-9 conditioning branch** ("Plan51 entire 應 defer post-AC-9") — operationalised via C1 (AC-9 Plan54 spec stable as conditioning input).
3. **Caution against bundled-decision-making** (RUSSELL R2 §2.11 implicit-third-branch identification) — operationalised via C5 explicit honour clause.

C1-C4 all-GREEN does NOT auto-trigger plugin-loader resumption. C5 is **not subsumable** under C1-C4 — it operates at procedural-honour discipline layer.

## 4. The 5 Criteria

### 4.1 C1 — AC-9 Plan54 spec stable

- Per R3 D-01 (Round A apex; Stage 2 runoff 16/7): **AC-9 Plan54 Candidate A (full plugin Plan52 isomorph) ratified**.
- Cycle 03-16 Master Ratification Batch 13 Item #1 APPROVE BINDING.
- Conditioning input for plugin-loader re-evaluation.
- **C1 status at cycle 03-16 close**: ✅ AC-9 Plan54 spec ratified; cycle 03-17 R0 evaluation can proceed to C2/C3/C4 with AC-9 spec known.
- Cross-ref: `Technical_Specifications/Plan54_AC9_Binding.md` + Master Ratification Batch 13 Item #1.

### 4.2 C2 — ~70-110 LOC budget validated

- Per cycle 03-15 R1 §5 estimate + cycle 03-15 O4 §7.4 pro-forma: 70-110 LOC factored, median ~90.
- LOW migration risk (greenfield 0-site).
- **Cycle 03-17 R0 should re-run LOC factor analysis** with post-AC-9 surface area folded in (does AC-9 introduce additional plugin-config schema fields requiring expansion?).
- Cycle 03-16 first-shipping of 4 modules **validates the per-module factoring methodology** at structural layer.
- **R4 LOC methodology guidance applies** (per cycle 03-16 O4 §7.5): D-G-class binding helpers default +30-40% over R1 baseline; shared dispatcher contract +30% over hand-counted estimate; boilerplate re-export plumbing explicitly estimated OR explicitly excluded as non-counting.

### 4.3 C3 — 0-site greenfield → first call-site emerge post-AC-9?

- Per cycle 03-13 Plan49 spec MRB-06: plugin-loader currently has **0 schema-drift `safeParse()` call-sites (greenfield)**.
- AC-9 Plan54 sub-agent composition spec (Candidate A full plugin per D-01) MAY introduce **first call-site** if AC-9 requires plugin-level config validation at sub-agent spawn.
- **Cycle 03-17 evaluation MUST determine**: does AC-9-introduced plugin-config surface justify schema gate immediately, or remain deferred?
- LEIBNIZ R1 framing: plugin-loader has strongest **architectural-gap-closure case**; AC-9 may make this *operational-necessity-case* if first call-site emerges.

### 4.4 C4 — MR-12 既有不破壞 verified for migration

- Per cycle 03-15 O4 §9 cross-version-skew helpers MUST precedent: any *new* plugin-loader schema must verify MR-12 forward-only honoured.
- If AC-9 introduces new plugin-config schema with version-tagging, cycle 03-17 evaluation MUST verify per-version migration helpers analogous to checkpoint-store v0.42-onwards pattern (cross-version-skew helpers 6/6 PASS at cycle 03-15 Plan51 first-shipping).
- If AC-9 is purely additive (new config fields, no rename of existing fields), MR-12 forward-only honoured by construction.
- CV-08 R3 reaffirm: forward-only MR-12; cycle 03-13 backfill = MR-10 one-off precedent non-invocable.
- CV-17 R3 reaffirm: MR-12 forward-only (no retrofit).

### 4.5 C5 — DSS-A4 procedural-caution honour clause (NEW per D-02 18/5)

**Statement**: even with C1-C4 GREEN, cycle 03-17 R0 MUST explicitly weigh DSS-A4 procedural-caution before resuming plugin-loader; explicit weighing SHALL be documented in cycle 03-17 R0 orientation artefact.

**Rationale (RUSSELL agent rationality per F-§4-R2-07 MED-HIGH)**:

The DSS-A4 6-dissent has three rationale branches (per §3.2 above):
1. Procedural-caution — partially refuted by first-shipping success at structural layer.
2. Post-AC-9 conditioning — operationalised via C1.
3. Caution against bundled-decision-making (DSS-A4's spirit) — operationalised via C5.

C1-C4 all-GREEN does NOT auto-trigger plugin-loader resumption. Cycle 03-17 R0 MUST **independently weigh** DSS-A4 procedural-caution against the 4 criteria GREEN, and documented weighing SHALL be archived in cycle 03-17 R0 artefact.

### 4.6 5-Criterion Semantic Precedence

| Criterion | Type (per RUSSELL agent rationality) | Logical role |
|-----------|--------------------------------------|--------------|
| C1 | Conditioning input | Necessary information for decision |
| C2 | Feasibility envelope | Within scope envelope |
| C3 | Trigger condition | Action-triggering condition |
| C4 | Constraint compliance | MR-12 既有不破壞 honoured |
| **C5** | **Dissent-honour cycle disposition** | **Even with C1-C4 GREEN, weigh DSS-A4 procedural-caution explicitly** |

C5 is **not subsumable** under C1-C4. It operates at a different semantic layer: C1-C4 are technical/compliance criteria; C5 is procedural-honour discipline.

## 5. The 5×4 Decision Matrix

The 5×4 matrix enumerates 5 cycle disposition outcomes against 4 criteria (C1-C4). C5 (DSS-A4 honour clause) operates as **gate condition** modulating *all* dispositions.

| Row | C1 (AC-9) | C2 (LOC) | C3 (call-site) | C4 (MR-12) | C5 (DSS-A4) | Recommended cycle 03-17 outcome |
|:---:|:---------:|:--------:|:--------------:|:----------:|:-----------:|---------------------------------|
| **1** | AC-9 ratified ✅ | within budget | first call-site emerges | MR-12 honoured | weighing documented | **Plan-spec candidate plugin-loader Zod gate** for cycle 03-17 R0 evaluation |
| **2** | AC-9 ratified ✅ | within budget | greenfield persists | (n/a; no surface) | weighing documented | **Continue DEFERRED**; revisit cycle 03-18+ |
| **3** | AC-9 ratified ✅ | over-budget | (any) | (any) | weighing documented | **Continue DEFERRED**; reconsider scope |
| **4** | AC-9 否決 / deferred ⏳ | (n/a) | (n/a) | (n/a) | (n/a; C1 short-circuits) | **Continue DEFERRED**; revisit cycle 03-18+ post-AC-9-ratification |
| **5** | AC-9 ratified ✅ | within budget | first call-site emerges | MR-12 violated ⚠️ | (n/a; C4 short-circuits) | **Continue DEFERRED**; resolve MR-12 path first |

### 5.1 KNUTH Algorithm Completeness

R1 5-row enumeration correctly covers 16-combination space (2⁴ = 16) via short-circuit pruning (AC-9 否決/over-budget/MR-12 violated invariant under remaining; greenfield invariant under C4). **No algebraic gap**.

### 5.2 C5 Honour Clause Modulation

C5 modulates Rows 1, 2, 3 explicitly:
- **Row 1**: even with C1-C4 GREEN, cycle 03-17 R0 MUST document DSS-A4 weighing before recommending Plan-spec candidate.
- **Row 2**: documenting DSS-A4 weighing supports DEFERRED disposition (consistent with DSS-A4 post-AC-9 conditioning).
- **Row 3**: documenting DSS-A4 weighing supports DEFERRED disposition (consistent with DSS-A4 caution-against-bundled-decision-making).

Rows 4 + 5 short-circuit on C1 / C4 respectively; C5 weighing not required (subsumed under technical/compliance failure).

### 5.3 NAGARJUNA Madhyamaka Decision-Coupling Caveat

5-criterion + 5×4 matrix should **NOT be applied mechanically** at cycle 03-17 R0. Madhyamaka-correct: criteria as **decision-support, not decision-replacement**. Independent agent reasoning + DSS-A4 honour cycle (C5) + AC-9 spec specifics (D-01 Candidate A full plugin) + 0-site call-site shape all condition final outcome.

## 6. Cycle 03-16 Close C1 Status Update (informational)

At cycle 03-16 R3 close (2026-04-28):
- C1 = ✅ AC-9 Plan54 Candidate A ratified (D-01 Stage 2 runoff 16/7).
- C2 = pending cycle 03-17 R0 LOC factor re-analysis (post-AC-9 surface).
- C3 = pending cycle 03-17 R0 first-call-site analysis (post-AC-9 plugin-config schema introduction question).
- C4 = pending cycle 03-17 R0 MR-12 verification (depends on AC-9 schema additivity).
- C5 = pending cycle 03-17 R0 DSS-A4 weighing documentation.

**Cycle 03-17 R0 carries explicit responsibility to evaluate C2-C5 with C1=GREEN as conditioning input.**

## 7. Cycle 03-17 R0 Invocation Procedure

1. **C1 short-circuit check**: if AC-9 Plan54 implementation cycle 03-17 (Dev) hits blocker → C1 not stable → Continue DEFERRED (Row 4 logic).
2. **C2 LOC factor re-analysis**: re-run pro-forma 70-110 LOC budget with AC-9 surface area (apply R4 LOC methodology guidance per cycle 03-16 O4 §7.5).
3. **C3 first-call-site analysis**: examine AC-9 plugin-config schema introduction → does plugin-loader gain a `safeParse()` call-site naturally?
4. **C4 MR-12 verification**: if C3 = first-call-site emerges, verify per-version migration helpers analogous to checkpoint-store pattern.
5. **C5 DSS-A4 weighing**: explicitly document weighing in cycle 03-17 R0 orientation artefact regardless of C1-C4 outcome.
6. **Disposition selection**: apply matrix Row 1 / Row 2 / Row 3 / Row 5 per C2-C5 evaluation; Row 4 already short-circuit-resolved if C1 stable.

## 8. MRB-06 Cycle 03-21 Disposition Logic (D-19 UNANIMOUS 23/0)

If §5-§7 5-criterion + 5×4 matrix evaluation cycle 03-17 produces "Continue DEFERRED" (Rows 2/3/4/5) and the cycle 03-21 soft 6-cycle floor anchor (per cycle 03-15 O4 §13.2) is reached:

### 8.1 Option A — Close-with-rationale

- Explicit decision that plugin-loader 0-site greenfield is acceptable indefinitely (NOT architecturally required).
- Rationale documented: e.g., AC-9 Plan54 Candidate A surface remained plugin-config-orthogonal; greenfield naturally appropriate.
- MRB-06 trail closes; cycle 03-13 architectural-gap acknowledgement formally retired.

### 8.2 Option B — Re-ratify-as-open-gap

- Explicit acceptance that the architectural gap persists with documented reason.
- Rationale documented: e.g., AC-9 surface introduced first-call-site but cycle 03-17/18/19/20 evaluation each produced DEFERRED for technical/compliance reasons; gap remains.
- MRB-06 trail continues with re-ratified status; soft floor next cycle (e.g., cycle 03-27).

### 8.3 Cycle 03-21 R0 Invocation Procedure

1. **Status check**: confirm plugin-loader status at cycle 03-21 (DEFERRED active? resumed? schema-shipped already?).
2. **Bilateral selection**:
   - Apply **Option A** if AC-9 Plan54 surface remained plugin-config-orthogonal across cycle 03-17~03-20 (greenfield naturally appropriate).
   - Apply **Option B** if AC-9 surface introduced first-call-site but evaluations produced DEFERRED for technical/compliance reasons.
3. **Document selection rationale** in cycle 03-21 R0 artefact + Master visibility item per Batch dispatch.
4. **MRB-06 closure or re-ratification** recorded in coordinator G5 obligation per cycle 03-15 O4 §7.3.

### 8.4 Coordinator G5 Sustained Obligation

Per cycle 03-15 O4 §7.3 + R1 F-§4-R1-10: coordinator G5 obligation per O4 §7.3 sustained at cycle 03-16. **MRB-06 (cycle 03-13 Plan49 spec) remains open** until cycle 03-21 disposition.

## 9. DSS-3 Closure-by-Execution + Plan51 Sunset (CV-19 + Madhyamaka caveat)

Per cycle 03-15 R3 D-§5-D UNANIMOUS 23/0: 推薦 sunset-by-execution. Active per D-§5-A 推薦 outcome (4-of-5 modules implemented in cycle 03-16). At first-shipping verification:
- 4 modules **shipped** in v0.51.0-alpha = **execution trigger condition MET**.
- DSS-3 carryover (cycle 03-14 LEIBNIZ ossification) **closed by R3 D-§5-D**; cycle 03-16 confirms execution triggered → DSS-3 closure confirmed.

**NAGARJUNA Madhyamaka caveat** (per F-§4-R2-05 + CV-19 R3 reaffirm): DSS-3 closure is **specific to D-§5-D ratification text**, NOT a generic doctrine that "shipping = ossification doctrine resolved". Sunset-by-execution applies **case-by-case** to R3-ratified-推薦-and-implement outcomes; it does NOT generalise as a generic shortcut.

**Plan51 sunset trail status table**:

| Trail | Sunset clause | Status at cycle 03-16 first-shipping |
|-------|---------------|--------------------------------------|
| 推薦 4-of-5 (D-§5-A 17/6) | sunset-by-execution per D-§5-D UNANIMOUS | ✅ TRIGGERED + CONFIRMED at v0.51.0-alpha |
| DEFERRED plugin-loader (D-§5-A 9/11/3) | soft 6-cycle floor anchor (cycle 03-21) | ⏳ pending; cycle 03-17 evaluation 5-criterion + 5×4 matrix |
| DSS-3 (cycle 03-14 LEIBNIZ ossification) | resolved by D-§5-D UNANIMOUS | ✅ CLOSED with NAGARJUNA Madhyamaka caveat per F-§4-R2-05 + CV-19 |

## 10. DSS Preservation (per MR-11 verbatim)

| DSS ID | Source | Vote | Status |
|--------|--------|:----:|--------|
| **DSS-A4** (cycle 03-15 D-§5-A carryover) | LEIBNIZ + KNUTH + SUSSMAN + 3 others | 6 (cycle 03-15 vote 17/6) | PRESERVED — procedural branch partially refuted by first-shipping success; post-AC-9 branch operationalised via C1; caution-against-bundled-decision-making operationalised via C5 |
| **DSS-B7** (cycle 03-15 D-§5-B carryover) | 4 personas | 4 (cycle 03-15 vote 17/4/2) | PRESERVED — operationally aligned with shipping outcome (4 modules; identical to DSS-B7 preferred 4-module variant) |
| **DSS-C4** (cycle 03-15 D-§5-E carryover) | 2 personas | 2 (cycle 03-15 vote 21/2) | PRESERVED — stylistic / granularity dissent; shipping verified 1-module-2-artefacts pattern implementable |
| **DSS-3** (cycle 03-14 R3 LEIBNIZ ossification carryover) | 1 (cluster) | — | **CLOSED-by-execution** at cycle 03-16 §4 D-§5-D trigger met (per CV-19) **with NAGARJUNA Madhyamaka caveat** |
| **DSS-CY16-02** (cycle 03-16 R3 NEW; D-02 minority) | SUSSMAN + DARWIN + LINNAEUS + 2 others | 5 (R3 D-02 vote 18/5) | PRESERVED — row-1-amendment minority; abstraction parsimony argument for embedding DSS-A4 honour inline in C1 rather than explicit C5 |

**Verbatim DSS-CY16-02 (cycle 03-16 R3 NEW)**: "Row 1 amendment to existing 4-criterion preferred over explicit C5; abstraction parsimony; DSS-A4 honour can be embedded inline rather than 5th criterion".

R3 majority 18 chose explicit C5 for operational clarity at cycle 03-17 R0 level (RUSSELL agent rationality per F-§4-R2-07).

## 11. MR/ZT Compliance Audit

| Constraint | Status | Evidence |
|------------|:------:|----------|
| MR-2 + MR-4 (Tenet 措辭不改) | PASS | No Tenet wording change |
| MR-5 hard (Tenet #10 status no change) | PASS | governance / architectural retrospective; no Phase 6 functional implication |
| MR-6 (Core 零) | PASS | 4 modules peripheral; verified at file-system spot-check + R2 PENROSE F-§4-R2-04 |
| MR-9 | PASS | no preemptive MUST claim; F-16 SHOULD initial inherited per CV-09 |
| MR-11 | PASS UNCONDITIONAL | DSS-CY16-02 R3 NEW + DSS-A4/B7/C4/3 carryover all preserved verbatim |
| MR-12 既有不破壞 | PASS | cross-version-skew helpers MUST D-§5-G honoured; structural verified |
| ZT-1 / ZT-2 / ZT-3 | PASS reaffirmed | endpoint 10/0/0★ unchanged; 9/0/1★ ACTIVE preserved |

## 12. Forward Schedule

- **Cycle 03-17 R0**: invoke 5-criterion + 5×4 matrix; document DSS-A4 weighing per C5; report to research-team per cycle convention.
- **Cycle 03-17~03-20**: each R0 may apply matrix as plugin-loader status evolves with AC-9 surface materialisation.
- **Cycle 03-21 soft floor**: MRB-06 disposition logic invocation per §8 above; bilateral structure A vs B.

---

*plugin-loader Cycle 03-17 5-Criterion + 5×4 Matrix Evaluation Framework — CANDIDATE pending Master Ratification Batch 13 #2*
*Authors: SUSSMAN + DARWIN + ARCHIMEDES + GUARDIAN + LEIBNIZ + RUSSELL*
*R3 D-items absorbed: D-02 (18/5; explicit C5) + D-19 (UNANIMOUS bilateral structure) + CV-19 (DSS-3 closure-by-execution with Madhyamaka caveat)*
*DSS preservation: DSS-A4 (6) + DSS-B7 (4) + DSS-C4 (2) + DSS-CY16-02 (5) verbatim; DSS-3 closed-by-execution with caveat*
*Plan51 sunset-by-execution triggered (D-§5-D); plugin-loader trail soft 6-cycle floor cycle 03-21*
*Compliance: MR-5 hard / MR-6 / MR-9 / MR-11 UNCONDITIONAL / MR-12 + ZT-1/2/3 全 PASS*
