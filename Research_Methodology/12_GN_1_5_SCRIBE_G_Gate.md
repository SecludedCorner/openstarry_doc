# GN.1–GN.5 SCRIBE G-Gate (Impact-Matrix Discipline)

> **Front-matter**
> - **Cycle**: 03-13
> - **Date**: 2026-04-24
> - **Authors**: SCRIBE (#2, gate authority) + SYNTHESIST (#1, consolidation) + RUSSELL (#23, agent-theoretic refinement) + NAGARJUNA (#7, two-truth framing + "structured failure" wording) + ARCHIMEDES (#16, engineering practice cross-ref to F-15) · dissent preserved per MR-11
> - **Binding R3 decisions**: **D-15 UNANIMOUS 23/0** (GN.2 genuinely discriminating hypotheses, RUSSELL refinement) · **D-16 19/4** (GN.3 mandatory with documented-escape clause; 4 dissent ATHENA/GUARDIAN/RUSSELL/DARWIN preferred mandatory-always) · **D-17 20/3** (F-15(e) explicit GN.2 cross-reference; 3 dissent WIENER/SUSSMAN/RUSSELL preserved — cross-referenced in F-15 chapter) · **D-18 UNANIMOUS 23/0** ("structured failure high repeat-likelihood" NAGARJUNA amendment) · **D-19 UNANIMOUS 23/0** (GN.5 extends to system-behaviour/metric, RUSSELL refinement) · **D-20 17/6** (two-truth framing explicit in GN.*; 6 dissent ATHENA/GUARDIAN/ARCHIMEDES/TURING/KNUTH/SUSSMAN philosophical-drift concern) · **CV-05 UNANIMOUS tacit** (GN.1+GN.4+GN.5 core structure agent-theoretically sound)
> - **Status**: **SCRIBE-internal authority** (not Master-route Rule per R3 §6 pattern; analogous to 03-12 G6.8/G6.9/G6.10 which are SCRIBE-internal). Master acknowledges via Batch 10 Item 10 (APPROVE recorded 2026-04-24). Enforcement authority = SCRIBE at G5 level. Rule-level generalisation (Rule #79 content-drift) deferred 1-2 cycles per D-22 UNANIMOUS.
> - **Effective**: **2026-04-24 R4 close onward; all impact matrices and R2+ published claims bound**. Cycle 03-13 artefacts grandfathered (first field application of GN.1-5 at 03-13 R4 close itself, specifically F-15 self-attestation in O5 §7 front-matter).
> - **Cross-refs**: `Research_Methodology/11_ENG_FAB_v1.8_Binding.md` §G (F-15 Code-Path Verified; D-17 GN.2 cross-ref) · `research record/cycle03-13/discussions/research_over_reaction_retrospective.md §3` (GN.1-5 originating context, Task #49 5-cause analysis) · `research record/cycle03-13/deliver/O6_phase_6_roadmap.md` (Phase 6 / AC-9 Claim E linkage under D-27 MED-HIGH) · `research record/cycle03-13/deliver/R3_decision_log.md §4.3 + §4.4 + §4.5 D-15/16/17/18/19/20`

---

## §1 Purpose — Code-Path Verified Before Inference, Impact-Matrix Discipline

The GN.1–GN.5 gates are **SCRIBE-internal-authority** structural checks that strengthen research deliverables at the **inference**, **alternative-hypothesis**, **peer-review**, **humility-discipline**, and **two-truth-framing** axes.

**Origin**: Task #49 over-reaction retrospective (cycle 03-13 §六, Master letter v2.0 §六). Five specific causes (C-1 through C-5) were diagnosed; GN.1–GN.5 are the corresponding SCRIBE G-gates. MR-11 framing: **learning-oriented, not punitive**.

**Naming**: `GN.*` = "G-gate New" additions — distinct from coordinator-route Rules (which follow Rule #NN numbering) and distinct from existing SCRIBE process gates G1-G7 (structural), G6.8/G6.9/G6.10 (SCRIBE authority per 03-12 D-18a).

**Scope**: these gates govern **published impact assessments, R2/R3/R4 claims touching ≥ 2 deliverables or ratified Rules, and retraction proposals**. Internal brainstorming (R1 sketches) is exempt per F-15(e) distinction + GN.2 Stage 1 symptom-catalog-only carve-out.

---

## §2 GN.1 — Code-Path Verification Required Before Impact Assessment

### §2.1 Rule

Any "failure", "invalidation", or "retraction" claim whose scope affects **≥ 2 deliverables** OR **touches a ratified Rule (≥ Rule #50)** MUST cite in the supporting document:

- **(a) Source code read** — path + commit hash + line numbers of the code that exhibits / implements the claimed failure.
  - **(a)' If source is opaque** (third-party binary, vendor-closed): cite documented behavioural contract (README, published API, vendor spec) with equivalent authority [R2 §5.1 RUSSELL refinement adopted; CV-coherent].
- **(b) Author intent understood** — reference to inline comments, README, or design-doc section that declares the code's *intended* behavior (quoted text).
- **(c) Alternative hypotheses enumerated** — **≥ 2 genuinely discriminating hypotheses** per D-15 UNANIMOUS (strawman or trivial alternatives excluded).

### §2.2 Failure mode prevented

- **C-1** (code-not-read before inferring "LLM silent failure"): warning-log-as-specification. Task #49 inferred "silent hook-registration failure" from PluginLoader INFO warnings alone, without reading plugin source.
- **C-2** (naming-as-behavior "dynamic" = LLM-arbitration-dynamic): `gear-arbiter-dynamic` read as "LLM-driven" by name convention.
- **C-4** (single-hypothesis lock-in): "plugin-by-design-not-LLM" hypothesis never enumerated.

### §2.3 R3 dissent — none (UNANIMOUS adoption)

D-15 UNANIMOUS 23/0 adoption — no dissent on GN.2 "genuinely discriminating" refinement (which crosses into GN.1's (c) clause).

---

## §3 GN.2 — Impact Matrix Three-Stage Gate (≥ 2 Genuinely Discriminating Hypotheses per D-15)

### §3.1 Rule

Impact matrices produced in response to a putative failure MUST be produced in three named stages, with a gate between each stage:

- **Stage 1 — Symptom catalog**: list observations only. No hypotheses. No actions. Internal brainstorming permitted here.
- **Stage 2 — Hypothesis registry**: list **≥ 2 genuinely discriminating hypotheses** (D-15 UNANIMOUS RUSSELL refinement) for each symptom cluster, with discriminating-evidence checklist. **Trivial alternatives or strawman alternatives are excluded** — each listed hypothesis must produce a distinct observable under the proposed diagnostic evidence.
- **Stage 3 — Invalidation/action matrix**: produced ONLY after root-cause confirmation from implementation-authoritative team (Dev for code issues, Test for runtime issues).

### §3.2 "Genuinely discriminating" operational test

A hypothesis *H₂* is **genuinely discriminating** relative to *H₁* iff there exists an observable *X* such that:
- `P(X | H₁)` and `P(X | H₂)` differ by at least one order of magnitude (or are qualitatively incompatible: one predicts presence and the other predicts absence).
- The observable *X* is accessible within the project's evidence-gathering budget.

Strawman test: if *H₂* is a weaker, obviously-false, or intentionally-weakened version of *H₁*, it fails the discriminating test — SCRIBE FAILs GN.2 at Stage 2 gate.

### §3.3 Failure mode prevented

- **C-3** (pre-confirmation action prescriptions): Task #49 produced a 14-item invalidation matrix within ~2h of first observation and **before** Dev Task #50 returned root cause. §4 of Task #49 was operational — "RETRACT", "DEACTIVATE", "RE-VERIFY", "RE-SUBMIT to Master" — despite the caveat header "subject to Dev Task #50 root-cause confirmation".
- **C-4** partial (alternative hypothesis enumeration at Stage 2 strengthens GN.1(c)).

### §3.4 R3 dissent — none (D-15 UNANIMOUS)

No dissent on "genuinely discriminating" refinement.

---

## §4 GN.3 — Peer Review Before Retraction Issuance (with Documented Escape per D-16 19/4)

### §4.1 Rule

Retraction proposals (items marked "RETRACT", "INVALIDATE", or "RE-SUBMIT to Master" in an impact matrix) MUST route through **minimum one second reviewer** — either Dev team lead (if code-causal) or another Research persona with independent standing (if interpretation-causal) — **before filing as a research artifact**.

- **Primary path**: synchronous (≤ 30 min) or asynchronous (≤ 2h) review.
- **Escape clause** (per D-16 19/4 adoption): if peer reviewer is unavailable, escape is permitted with **documented escape** capturing **all three** of:
  - **(i) URGENT timeline specificity** — explicit citation of the URGENT dispatch, the deadline driving urgency, and the irrecoverable cost of waiting for review.
  - **(ii) Single-agent-has-read-source condition** — the author personally read the relevant source at declared path + line range (GN.1(a) standard). This is the **lowest-risk condition** because direct source evidence substitutes somewhat for second-opinion.
  - **(iii) Post-hoc peer review within 48h commitment** — commitment to obtain second-reviewer sign-off within 48 hours of artifact filing; failure to meet commitment = **evidence of erosion** flagged by SCRIBE.

### §4.2 Failure mode prevented

- Solo-authored retractions bypass institutional checks (C-3 contributing factor — Task #49 was effectively solo-authored SCRIBE + GUARDIAN + PASCAL + KNUTH + TURING + SYNTHESIST within the main session, but no external second-reviewer sign-off before filing).
- Time-pressure abrogation of review discipline (GN.4 failure mode).

### §4.3 R3 dissent — 4 agents (ATHENA / GUARDIAN / RUSSELL / DARWIN preserved per MR-11)

**D-16 19/4 tally**. Minority position: **mandatory-always without escape clause**, citing:
- **Escape-clause erosion risk over time** — once an escape is codified, invocation expands until the default becomes escape.
- **URGENT-abuse risk** — URGENT flag may be applied too liberally, collapsing the primary path in practice.
- **Removing judgment burden** — mandatory-always forces the author to confront the reviewer-availability constraint directly rather than exercising discretion.

**Majority (19 agents) rationale**:
- Real-world URGENT situations (vendor incidents, Master-observation-triggered investigations) require rapid response; mandatory-always can push authors toward skipping GN.3 entirely rather than invoking escape.
- 48h post-hoc commitment is itself an audit signal — missed commitment = erosion alarm.
- Documentation requirement (i/ii/iii simultaneously) makes escape non-trivial to invoke.

**SCRIBE obligation per MR-11**: monitor escape-invocation patterns in cycles 03-14/03-15/03-16. If:
- Escape invoked > 2 times per cycle, OR
- Post-hoc 48h commitment missed ≥ 1 time, OR
- Post-hoc review systematically weaker than primary-path review,

→ **escape clause revocable at cycle-end retrospective**; revert to mandatory-always per D-16 minority position.

---

## §5 GN.4 — Urgency ≠ Finality

### §5.1 Rule

URGENT task status (as tagged in coordinator dispatch) does **NOT** exempt a research output from GN.1 code-path verification, GN.2 staged matrix, or GN.3 peer review. URGENT flag means **elevated priority for scheduling**, not **reduced evidentiary standard**.

Research outputs produced under URGENT flag MUST include a statement in their front-matter:

> "This URGENT-flagged output was produced under full GN.1-GN.3 discipline with evidentiary backing at standard threshold."

OR, if shortcuts were necessary:

> "URGENT-flagged output produced under partial GN.1-GN.3 discipline. The following shortcuts were taken: [explicit enumeration]. Post-hoc remediation scheduled for: [date/deadline]."

### §5.2 Failure mode prevented

Time-pressure dilution of verification discipline (C-3 + general URGENT cultural hazard). If URGENT status becomes an implicit waiver, the entire GN.1-GN.5 framework is subverted via abuse of the flag.

### §5.3 R3 dissent — none

GN.4 recognized by R2 (§5.4 RUSSELL) as "STRONGLY SOUND" — adopted as-drafted. No R3 dissent.

---

## §6 GN.5 — Humility Discipline (Component / Metric / System-Behaviour Intent) per D-19 UNANIMOUS

### §6.1 Rule (per D-19 UNANIMOUS extension beyond plugin/module)

When characterizing the **intent** or **design** of a plugin, module, **system component**, **metric**, **or system behaviour** in a failure-analysis context, default phrasing MUST be:

> "I have not yet confirmed the source's design intent."

rather than:

> "The plugin is [broken / failing / silently failed]"
> "The metric is [converging / diverging]"
> "The system is [broken]"

### §6.2 D-19 UNANIMOUS extension — scope broadened

Per D-19 UNANIMOUS (RUSSELL refinement), GN.5 scope extends from plugin/module-only to **system behaviour + metrics**:
- Metric claims: "V11 = 1.0000 dead-center" → forbidden (see `Reference/07_V11_Wording_Binding.md` cycle 03-12 RATIFIED); use "V11 = 1.0000 numerical artifact / tautological" per GN.5 humility.
- System behaviour claims: "The system converged" → forbidden absent dynamical-systems-level evidence; use "system entered steady-state observation under these fixture conditions".

### §6.3 Specific wording discipline (cycle 03-13 cross-refs)

- **σ**: use "composition index (deterministic event-count vector × constant-value lookup)" — NOT "variance estimator" or "LTI evidence". Per CV-07 UNANIMOUS tacit + `O3_core_trio_final.md §B`.
- **V11 = 1.0000x**: use "numerical artifact / tautological 1.0000x due to rolling-reference formula mathematical identity" — NOT "dead-center / converged / Lyapunov V → 0". Per cycle 03-12 R3 D-03(b) BINDING `Reference/07_V11_Wording_Binding.md`.
- **`gear-arbiter-dynamic`**: use "by-design pure-WIENER-math plugin; `cat=undefined raw=0 clamped=0` is intended null output" — NOT "broken / silent failure". Per MRB-02 RESOLVED.
- **`P(coincidence) ≤ 10⁻⁴⁸`**: use "conservative upper bound under the assumed H_coincidence model" + Rule #76 §76.7 caveat annotation when σ_regime = composition_index. Per `Reference/08_Rule_76_Section_76_7_Caveat.md`.

### §6.4 Failure mode prevented

- **C-2** (naming conflation — "dynamic" = "LLM-arbitration-dynamic").
- **C-5** (Master-observation framing bias — senior-voice-as-ground-truth; epistemic asymmetry).

### §6.5 R3 dissent — none (D-19 UNANIMOUS)

---

## §7 "Structured Failure High Repeat-Likelihood" Framing (D-18 UNANIMOUS, NAGARJUNA amendment)

### §7.1 Per D-18 UNANIMOUS adoption

Replace the cycle 03-13 R1 draft's "pattern" language with **"structured failure high repeat-likelihood"** framing (NAGARJUNA amendment). Rationale:

- "Pattern" carries a deterministic-recurrence connotation that may overstate the cross-cycle repeat probability.
- "Structured failure high repeat-likelihood" frames the observation as (a) **structural** (has a system-level cause, not random fluctuation), (b) **high repeat-likelihood** (will likely recur absent intervention), (c) **failure** (something went below expected standard — not blame-assignment).

### §7.2 Usage in GN.* retrospectives

SCRIBE retrospective narratives describe observed failure patterns as:

> "Structured failure with high repeat-likelihood observed in §X: author-intent-inquiry missing in N of M cases during cycle Y. GN.1(b) re-emphasis scheduled for next-cycle kick-off."

NOT as:

> "Pattern X observed. Agent Y responsible. Remediation: personal retraining."

### §7.3 D-18 UNANIMOUS — no dissent

---

## §8 Two-Truth Framing Explicit in GN.* (D-20 17/6, NAGARJUNA)

### §8.1 Per D-20 17/6 adoption

Two-truth framing (conventional truth / ultimate truth, per NAGARJUNA Madhyamaka distinction) is **explicitly adopted in GN.*** failure analysis.

- **Conventional truth** — propositional-level fault: e.g., "plugin naming convention was misread" (C-2). Remediable by GN.5 humility + GN.1(b) author-intent check.
- **Ultimate truth** — methodological-level fault: e.g., "method of hypothesis-formation bypassed strongest available evidence source" (C-1, C-3, C-4). Remediable by GN.1 / GN.2 / GN.3 structural gates.

Two-truth framing maps each cause (C-1 through C-5) to the appropriate gate tier:

| Cause | Truth level | Primary remedy | Secondary remedy |
|-------|:-----------:|----------------|------------------|
| C-1 code not read | Ultimate | GN.1 | — |
| C-2 naming conflation | Conventional | GN.1(b) | GN.5 |
| C-3 matrix before confirmation | Ultimate | GN.2 | GN.3 |
| C-4 alternative hypothesis missed | Ultimate | GN.1(c) + D-15 refinement | GN.2 |
| C-5 Master-observation framing bias | Conventional | GN.1(a)' | GN.5 |

Three ultimate-level causes → methodological gates (GN.1-3). Two conventional-level causes → phrasing discipline + source verification (GN.5 + GN.1(a)').

### §8.2 R3 dissent — 6 agents (ATHENA/GUARDIAN/ARCHIMEDES/TURING/KNUTH/SUSSMAN preserved per MR-11)

**D-20 17/6 tally**. 6 dissenters concerned about **philosophical drift** — the risk that explicit two-truth framing imports Madhyamaka-flavored language into technical documents and invites further philosophy-drift in cycles ahead.

**Majority (17 agents) rationale**:
- Two-truth framing maps naturally to methodological/propositional distinction that already exists in engineering epistemology; it provides a vocabulary rather than novel content.
- Mapping each C-cause to truth-level gives SCRIBE concrete remediation targets; no framing equivalent is as crisp.
- The distinction is cycle-local (applies to failure analysis); not a general doc-language change.

**SCRIBE monitoring obligation per MR-11**: monitor cycles 03-14/03-15/03-16 for drift signs — philosophical-language expansion beyond failure-analysis context into general technical docs. If drift observed:
- Re-scope two-truth framing to failure-analysis only (not general),
- Or retract D-20 adoption via cycle-end retrospective CR route.

Until drift signal, D-20 stands.

---

## §9 Relationship to F-15 (ENG-FAB v1.8 cross-ref per D-17 20/3)

### §9.1 D-17 cross-ref rule

Per **D-17 20/3** (preserved 3 dissent WIENER/SUSSMAN/RUSSELL in `Research_Methodology/11_ENG_FAB_v1.8_Binding.md §I.1`), F-15(e) explicitly cross-references GN.2:

> F-15(e): alternative hypotheses in F-15.c must meet **GN.2 discriminating-power bar** when assessment is in the R2+ published-claim domain.

### §9.2 Authority separation

- **F-15**: **Master-ratified ENG-FAB MUST item** (Batch 10 Item 6 APPROVED 2026-04-24); applies to research deliverables' impact assessments. Published-claim domain (R2/R3/R4). Reviewer discipline + evidence artefact.
- **GN.1-GN.5**: **SCRIBE-internal-authority process gates** at G5 level; apply to R2+ published claims touching ≥ 2 deliverables. Gate enforcement + process discipline.

### §9.3 Composition in practice

When a research artifact produces an impact assessment:
- F-15 requires (a) source-code read + (b) author-intent + (c) alt-hypothesis + (d) second-reviewer + (e) scope marker with GN.2 cross-ref.
- GN.1-5 SCRIBE audit at G5 verifies the Stage 1/2/3 progression, peer review (possibly with D-16 escape), humility wording, and two-truth framing.
- Complementary: F-15 = author-side attestation; GN.* = SCRIBE-side audit.

### §9.4 Exception — Rule #79 deferral (per D-22 UNANIMOUS)

D-22 UNANIMOUS defers rule-level generalisation to **Rule #79 (content-drift generalisation)** by 1-2 cycles. Rationale: F-11 canonical-sync + GN.1-5 SCRIBE gate + F-15 MUST are sufficient for 03-13; generalise to Rule #79 only after 03-14/03-15 field data accumulates (whether pattern generalises or not).

---

## §10 Authority and Enforcement

### §10.1 SCRIBE-internal authority (not Master-route)

GN.1-GN.5 are **SCRIBE-internal authority gates**. Analogous to:
- 03-12 G6.8/G6.9/G6.10 (SCRIBE authority per D-18a cycle 03-12 UNANIMOUS 23/0 — `Calibration_Reports/18_PathC_20Field_Monitoring_Format.md §7`).
- 03-13 GP.1-GP.5 pre-gate (subagent trial SCRIBE authority per R3 §6 pattern).

**Master acknowledgement ≠ Master-route**. Master APPROVE recorded in `Batch_10_Request.md Item 10` signals awareness; the gates operate under SCRIBE authority, which means SCRIBE can declare FAIL independently and the fail is binding without Master adjudication.

### §10.2 Audit cadence

- **Every R2/R3/R4 artifact touching impact claims**: SCRIBE audits at G5 for GN.1-5 conformance.
- **Every R4 deliver artifact with F-15 front-matter block** (per `Research_Methodology/11_ENG_FAB_v1.8_Binding.md §G.5`): SCRIBE audits for block presence + content non-vacuousness.
- **Every cycle close**: SCRIBE writes `{cycle}/discussions/GN_audit_report.md` summarising GN.* gate status (PASS/FAIL + remediation status).

### §10.3 Pattern-recognition without shaming (per MR-11)

Per F-15.3 and §7.2: where GN.* FAIL occurs ≥ 2 times in a single cycle, SCRIBE notes the pattern **topically (not individually)** in cycle-end retrospective — e.g., "§3 retrospective stage produced 3 F-15 FAILs, suggesting R1 draft-review checklist needs strengthening" NOT "agent X failed F-15 three times".

---

## §11 Dissent Roll-Up (MR-11 preservation summary)

### §11.1 This document's preserved dissent

| D-ID | Dissent agents | Count | Rationale (abridged) | Preservation location |
|------|----------------|:-----:|----------------------|-----------------------|
| D-16 (GN.3 escape clause) | ATHENA / GUARDIAN / RUSSELL / DARWIN | 4 | Mandatory-always without escape preferred; escape-clause erosion risk, URGENT-abuse risk | §4.3 above + `R3_D_items_voting.md §D-16` |
| D-20 (two-truth explicit in GN.*) | ATHENA / GUARDIAN / ARCHIMEDES / TURING / KNUTH / SUSSMAN | 6 | Philosophical drift concern; two-truth language may expand beyond failure-analysis context | §8.2 above + `R3_D_items_voting.md §D-20` |

### §11.2 Cross-referenced dissent (preserved elsewhere)

| D-ID | Dissent agents | Preserved in |
|------|----------------|--------------|
| D-17 (F-15(e) GN.2 cross-ref) | WIENER / SUSSMAN / RUSSELL (3) | `Research_Methodology/11_ENG_FAB_v1.8_Binding.md §I.1` |

**Total direct dissent in this document**: 10 entries (4 D-16 + 6 D-20). Cross-referenced: 3 (D-17). Combined: 13 dissent entries across 3 D items impacting GN.1-5.

---

## §12 Cross-References

### §12.1 Within `openstarry_doc/`

- `Research_Methodology/11_ENG_FAB_v1.8_Binding.md` §G — F-15 Code-Path Verified; F-15(e) GN.2 cross-ref.
- `Reference/08_Rule_76_Section_76_7_Caveat.md` — Rule #76 §76.7 (σ-regime wording discipline aligns with GN.5 humility per §6.3).
- `Reference/07_V11_Wording_Binding.md` (cycle 03-12 predecessor) — V11 binding "numerical artifact / tautological" (GN.5 wording anchor).
- `Calibration_Reports/20_Section_7_3_Verification_Plan.md` (this cycle) — coordinator 7-step verification plan (CK-1..CK-7 consultative).

### §12.2 Research record (cycle 03-13)

- `discussions/research_over_reaction_retrospective.md §3` — GN.1-5 originating context + Task #49 5-cause analysis.
- `deliver/O6_phase_6_roadmap.md` — Phase 6 AC-9 Claim E linkage (per D-27/D-30 MED-HIGH HMAC-signed parentAgentId MVP).
- `deliver/Master_Ratification/Batch_10_Request.md Item 10` — GN.1-5 SCRIBE G-gate additions ratification request.
- `deliver/R3_decision_log.md §4.3 + §4.4 + §4.5 D-15/16/17/18/19/20` — R3 voting record.
- `deliver/R3_D_items_voting.md §D-15/16/17/18/19/20` — 23-agent vote tables.

### §12.3 Historical anchor (cycles 03-11 / 03-12)

- `Research_Methodology/09_Rule_75_PreDelivery_Gate_Draft.md` (cycle 03-12) — Rule #75 Pre-Delivery Gate; analogous SCRIBE G6.8 gate authority.
- `Calibration_Reports/18_PathC_20Field_Monitoring_Format.md §7` (cycle 03-12) — G6.8/G6.9/G6.10 SCRIBE authority precedent per D-18a UNANIMOUS.

---

## §13 Status and Author Attribution

- **Authority**: SCRIBE-internal (not Master-route). Master APPROVE recorded via `Batch_10_Request.md Item 10` 2026-04-24.
- **Effective**: 2026-04-24 R4 close onward; cycle 03-13 self-application via F-15 reflexive attestation (O5 §7 front-matter of ENG-FAB v1.8 binding).
- **Primary authorship**:
  - SCRIBE (#2) — gate authority + rule-text discipline.
  - RUSSELL (#23) — GN.2 "genuinely discriminating" refinement + GN.5 system-behaviour/metric extension.
  - NAGARJUNA (#7) — "structured failure high repeat-likelihood" wording (D-18) + two-truth framing (D-20).
  - ARCHIMEDES (#16) — F-15 cross-reference + engineering-practice lens.
  - SYNTHESIST (#1) — consolidation.
  - SUNYATA (#0) — chair (procedural abstain).
- **Dissent agents preserved**:
  - D-16: ATHENA (#5) / GUARDIAN (#11) / RUSSELL (#23) / DARWIN (#6).
  - D-20: ATHENA (#5) / GUARDIAN (#11) / ARCHIMEDES (#16) / TURING (#17) / KNUTH (#21) / SUSSMAN (#22).
- **R3 vote record** (ground truth `R3_decision_log.md §4.3 + §4.4 + §4.5`):
  - **D-15 UNANIMOUS 23/0** (GN.2 genuinely discriminating).
  - **D-16 19/4** (GN.3 with documented-escape).
  - **D-17 20/3** (F-15(e) GN.2 cross-ref; cross-ref in ENG-FAB file).
  - **D-18 UNANIMOUS 23/0** ("structured failure" wording).
  - **D-19 UNANIMOUS 23/0** (GN.5 system-behaviour/metric extension).
  - **D-20 17/6** (two-truth framing explicit).
  - **CV-05 UNANIMOUS tacit** (GN.1+GN.4+GN.5 agent-theoretically sound).

---

*GN.1–GN.5 SCRIBE G-Gate — Cycle 03-13 — 2026-04-24 R4 close*
*SCRIBE-internal authority (not Master-route); Master APPROVE via Batch 10 Item 10 2026-04-24*
*6 R3 decisions: D-15/18/19 UNANIMOUS + D-16 19/4 + D-17 20/3 + D-20 17/6; 10 direct dissent + 3 cross-ref dissent preserved per MR-11*
