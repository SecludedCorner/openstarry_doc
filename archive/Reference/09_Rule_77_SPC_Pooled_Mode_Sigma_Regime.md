# Rule #77 — SPC Pooled-Mode σ-Regime Conjunct (FR-2 in-place revise)

> **Front-matter**
> - **Cycle**: 03-13
> - **Date**: 2026-04-24
> - **Authors**: WIENER (#12, control theory + FR-2 derivation) + LEIBNIZ (#14, protocol-evolution lens — in-place amendment rationale) + ARCHIMEDES (#16, engineering spec + Plan50 σ_regime target) + KNUTH (#21, statistical methodology) + SCRIBE (#2, rule-text addition discipline) + SYNTHESIST (#1, consolidation) · **dissent preserved per MR-11**: MESH (#4) / PASCAL (#19) / PENROSE (#18) / KNUTH (#21 co-dissenting) / RUSSELL (#23)
> - **Binding R3 decisions**: **D-04 18/5** (in-place σ_regime conjunct primary; 5-agent dissent for DEACTIVATE-and-reformulate preserved) · **D-05 UNANIMOUS 23/0** (σ_regime inclusion in Plan50) · **D-06 UNANIMOUS 23/0** (7 R1 + 2 R2 = 9-item impact list) · **D-09 UNANIMOUS 23/0** (batched via Batch 10, not standalone Batch 8c') · **CV-07 UNANIMOUS tacit** (σ = composition index) · **CV-08 UNANIMOUS tacit** (V11 = numerical artifact / tautological)
> - **Status**: **BINDING as Rule #77** (post-Master ratification 2026-04-24 via Batch 10 Item 8). **Effective Plan50 delivery** for implementation; **effective R4 close 2026-04-24** for textual authority (rule-level text enforceable immediately).
> - **Supersedes**: nothing — Rule #77 is a **new rule**. Rule #72 FR-2 predicate receives in-place **amendment** (conjunct `AND (σ_regime ∈ {llm_variance, mixed})`), not replacement; Rule #72's other clauses (§72.1–§72.7) are untouched.
> - **Cross-refs**: `Reference/06_P_Coincidence_Binding_Convention.md` (Rule #76 §76.1–§76.6) · `Reference/08_Rule_76_Section_76_7_Caveat.md` (σ-deterministic regime caveat companion) · `Research_Methodology/08_Rule_72_N10_SubClause_Draft.md` (cycle 03-12 Rule #72 §72.1–§72.7 — predecessor FR-2 text) · `Research_Methodology/11_ENG_FAB_v1.8_Binding.md` §E (F-13 Plugin Hook Dispatch; σ_regime detection via plugin introspection) · `Technical_Specifications/Plan50_Sigma_Regime_Binding.md` (σ_regime ~50 LOC machinery Option γ) · `research record/cycle03-13/deliver/O3_core_trio_final.md §B` (σ metric semantic clarification) · `research record/cycle03-13/todo/Plan50_sigma_regime_spec.md` (Dev-facing spec)

---

## §1 Rule #77 — Authoritative Text

### §1.1 Rule #77 statement

> **Rule #77 — SPC Pooled-Mode σ-Regime Conjunct**
>
> **§77.1 Rule**: Any SPC machinery that activates a pooled-mode / regime-switch / recalibration-trigger predicate on the basis of σ variance or σ stability **MUST** condition activation on **`σ_regime ∈ {llm_variance, mixed}`**. Under **`σ_regime = composition_index`**, such predicates remain textually defined but are **dormant** (evaluate to false) because the underlying σ carries no stochastic signal (σ = deterministic composition index per cycle 03-13 §四 finding).
>
> **§77.2 σ_regime tag values**:
> - `composition_index` — σ is a deterministic composition index (event-count vector × constant-value-lookup). No stochastic signal under H_variance. **Default tag for pre-Plan50 observations**; **current tag under the 03-13-era pipeline**.
> - `llm_variance` — σ is driven by LLM output variance (per-event risk scores non-constant; stochastic signal present).
> - `mixed` — σ composition combines static-lookup events with LLM-variance events.
>
> **§77.3 Rule #72 FR-2 amendment (in-place)**: FR-2 pooled-mode activation predicate becomes:
> ```
> FR-2 pooled-mode activates iff
>   (max(σᵢ) − min(σᵢ) ≤ 0.0001 for N ≥ 3 consecutive rounds)
>   AND (σ_regime ∈ {llm_variance, mixed})
> ```
> The first conjunct preserves the original R1/R2 Rule #72 text; the second conjunct is the σ_regime gate per this rule.
>
> **§77.4 Version-history annotation mandate**: every in-place amendment to a SPC rule per Rule #77 **MUST carry a version-history annotation** in the rule's revision log capturing (i) original text, (ii) amended text, (iii) R3 vote record with dissent entries, (iv) migration rationale. See §6 for the template.
>
> **§77.5 3-round identity preservation**: the mathematical identity "σ_R(N-2) = σ_R(N-1) = σ_R(N) under deterministic pipeline ⇒ deterministic output" is preserved as a **factual property** of the pipeline, not as LTI regime evidence. Deterministic-regime observations remain recorded with their σ values; interpretation is governed by Rule #76 §76.7 caveat.
>
> **§77.6 Scope**: applies to all SPC rules, pooled-mode predicates, regime classifiers, drift detectors, and recalibration triggers that consume σ as an input. Does **not** apply to: (a) σ-as-output observational rules (simply recording σ values without inferring regime), (b) Rule #76 P(coincidence) itself (which receives its own σ-regime caveat via §76.7).

### §1.2 Rule-level semantics summary

Under **current 03-13-era pipeline** (σ_regime = `composition_index` per cycle 03-13 §四 CV-07 UNANIMOUS):
- FR-2 pooled-mode: **dormant** (first conjunct can satisfy via 3-round identity, but second conjunct is false → rule inactive).
- No implicit stochastic-regime recalibration is triggered by 3-round σ identity.
- The identity itself is recorded and cited per Rule #76 §76.7 caveat (sampling-floor statement, not literal coincidence).

Under **future LLM-reflective pipeline** (Plan51+ per §四 Option α / Phase 6+ AC-9 per O6 §9 roadmap):
- σ observations tagged `llm_variance` or `mixed` satisfy the §77.3 second conjunct.
- FR-2 pooled-mode activates naturally when the first conjunct fires (σ variance collapse across ≥ 3 rounds).
- Rule #77 machinery gates FR-2 correctly without further rule rewrite.

---

## §2 In-Place Revise Path — Rationale and Method

### §2.1 R3 D-04 decision (18/5)

R3 D-04 adopted **in-place revise with σ_regime conjunct** (R2 LEIBNIZ primary elevated via R3 vote) over DEACTIVATE-and-reformulate (R1 §B.4 primary).

### §2.2 Majority rationale (18 agents)

1. **Preserves ratification trail** (LEIBNIZ primary rationale, R2 §B.4). The Rule #72 FR-2 amendment is Batch-3A-adjacent; in-place preserves continuity. Two-step "deactivate → wait → reformulate → re-ratify" sequence burns governance cycles unnecessarily.
2. **σ_regime tag = governance artifact**. Once Plan50 σ_regime ships (~50 LOC Option γ), the machinery is sufficient to gate FR-2 activation without additional infrastructure.
3. **Metadata extension > rule replacement** (protocol-evolution pattern). Adding a conjunct is a robust forward-compatible pattern; replacing rules introduces versioning overhead for downstream consumers.
4. **Testability preserved**. Rule #72 predecessor text remains in place; readers can understand the evolutionary path via the revision log (§6 annotation template).
5. **Version-history annotation mandate** (§77.4, per SYNTHESIST aggregation on D-04) addresses 5-dissenters' transparency concern without reverting the in-place decision.

### §2.3 Dissent (5 agents, preserved per MR-11)

Per MR-11, the D-04 5-agent minority is preserved with full rationale (see §7 below for individual rationales).

---

## §3 σ_regime Annotation Specification (Plan50 ~50 LOC Option γ)

### §3.1 Data-model field

Every σ observation record (SPC store, audit:completed event, W2 per-round summariser output, Path-C 20-field record) carries a **`sigma_regime` field**:

```
sigma_observation = {
  round_id: string,
  sigma: float,
  ucl: float,
  lcl: float,
  N_events: int,
  mean: float,
  ...
  sigma_regime: "composition_index" | "llm_variance" | "mixed"  # NEW
}
```

**Default**: `composition_index` (retroactive default for all pre-Plan50 observations; migration ~7 LOC per `Plan50_sigma_regime_spec.md §2`).

### §3.2 Regime inference logic (~15 LOC)

```python
def infer_sigma_regime(sigma_computation_inputs):
    """
    Determine σ_regime tag at σ computation time.

    Current pipeline (all tool_audited:* values drawn from static lookup table;
    gear-arbiter-dynamic contributes clamped=0):
      → composition_index

    Future pipelines (Phase 6+):
      - per-event LLM-risk-score injection into v_k → llm_variance
      - mixed static + LLM-risk events → mixed
    """
    has_llm_variance_source = any(
        source.is_llm_derived() for source in sigma_computation_inputs.sources
    )
    has_static_source = any(
        source.is_static_lookup() for source in sigma_computation_inputs.sources
    )
    if has_llm_variance_source and has_static_source:
        return "mixed"
    if has_llm_variance_source:
        return "llm_variance"
    return "composition_index"
```

See `Technical_Specifications/Plan50_Sigma_Regime_Binding.md §3` for full Plan50 specification (all change surfaces enumerated).

### §3.3 Rule consumer branching (~8 LOC total across FR-2 + Rule #76 checks)

- **FR-2 pooled-mode activation check**: conjunct evaluation per §77.3. ~3 LOC.
- **Rule #76 §76.7 caveat trigger** at P(coincidence) claim-evaluation boundary: if `claim.sigma_regime == "composition_index"` → emit §76.7 caveat annotation. ~5 LOC.

---

## §4 3-Round Identity Property Preservation (§77.5)

### §4.1 Factual property retained

The 3-round σ identity R10 = R11 = R12 = 0.023753 (6-decimal identical, observed 2026-04-20) is retained as a **factual observation**:
- Each σ value is recorded in `test_report_w2rN.md` verbatim.
- Path-C 20-field records per `Calibration_Reports/18_PathC_20Field_Monitoring_Format.md` (cycle 03-12) preserve the identity across rounds.
- The identity is noted in `MEMORY.md` factually (see cycle03-13 post-R3 snapshot).

### §4.2 Interpretation governed by Rule #76 §76.7

The **interpretation** of the 3-round identity is governed by Rule #76 §76.7 caveat:
- In σ-deterministic regime: "sampling-floor statement reflecting pipeline deterministic reproduction; not literal coincidence probability; NOT LTI regime evidence in the stochastic sense."
- In σ-stochastic regime (future): "literal 3-round identity observation; if combined with §76.1–§76.5 requirements, evidence of deterministic pipeline or intensive compositional invariance worth investigation."

### §4.3 No re-baseline; baseline σ = 0.023753 preserved

Per §B.7 of O3 + Plan50 spec §2, the existing σ = 0.023753 baseline is preserved verbatim. σ_regime annotation **does not** trigger a re-baseline; it only adds a metadata field. W2-R13 shift to σ = 0.023993 (Plan48 composition change, +1.01%) is recorded as **composition-shift re-baseline for Plan48+ codebase**, not drift.

---

## §5 Impact List — 9 Items (D-06 UNANIMOUS)

Per R3 D-06 UNANIMOUS, the following 9 artifacts require σ_regime-aware back-fill or procedural update (reproduced from `O3_core_trio_final.md §B.6` for canonical reference):

### §5.1 Tier A — In-place revisions (MR-10 back-fill, SCRIBE-owned)

1. `MEMORY.md` σ W2 references — keep factual σ = 0.023753; replace "LTI strong evidence" → "deterministic pipeline baseline"; replace "P(coincidence) ≈ 10⁻⁴⁸" → "P(coincidence) ≤ 10⁻⁴⁸ conservative upper bound; §76.7 caveat: σ_regime = composition_index; not informative for this pipeline".
2. `baseline_rules.md` Rule #76 — append §76.7 caveat (SCRIBE).
3. `research record/cycle03-11/deliver/O2_W2R11_full_analysis.md §3.3` — append footnote: "σ interpretation revised cycle03-13 §四; calculation correctness preserved under assumed model; applicability scope narrowed per Rule #76 §76.7."
4. `research record/cycle03-12/deliver/O2_W2R12_full_analysis.md §2.3 + §2.4` — same footnote; §2.4 Rule #76 candidate text aligned with ratified Rule #76 + §76.7.
5. `research record/cycle03-12/deliver/O6_PathC_monitoring_framework.md §1.2` — update `pillars` table `P(coincidence) ≤ 10⁻⁴⁸ binding` row with §76.7 caveat pointer.
6. `research record/cycle03-12/deliver/O7_Compliance_12_03.md §4` — update 9/0/1★ σ-narrative; remove "σ-LTI evidence" language; 9/0/1★ stands on non-σ evidence per CV-11.
7. `research record/cycle03-12/deliver/Master_Ratification/Batch_3A_P_Coincidence_Binding.md` — append cycle 03-13 §五 addendum note (this cycle's revisions).
8. **R2 addition** — `openstarry_doc/Principles/02_SPC_Discipline.md` (cycle 03-10 establishment) — if doc references σ as stochastic-SPC statistic, append σ_regime-aware framing footnote; if purely procedural, no action. SCRIBE verifies in R4 sweep.
9. **R2 addition** — `research record/cycle03-9/deliver/O7_Phase3_Baseline.md` PH3-SPC-1~4 — add footnote referencing §76.7 / σ_regime without rewriting the cycle 03-9 artifact (MR-11 historical integrity). SCRIBE verifies in R4 sweep.

### §5.2 Tier B — Procedural updates

- **B-1** Rule #72 FR-2 amendment (WIENER + ARCHIMEDES) — submit revision in Batch 10 (per D-09 UNANIMOUS, batched, not standalone Batch 8c'). Predicate per §77.3.
- **B-2** Plan48 W2-R13 GO verdict retention — retain GO verdict (valid on non-σ evidence: all-MUST PASS + canary 3/3). Add footnote documenting σ re-interpretation.
- **B-3** `Reference/06_P_Coincidence_Binding_Convention.md` (predecessor) — append §76.7 via companion file (`Reference/08_Rule_76_Section_76_7_Caveat.md` this cycle); update front-matter status cross-ref.

### §5.3 Tier C — Read-only (no action)

- `research input/**` (frozen).
- Master letters (cycle 02-12 → cycle 03-13).
- Prior R1/R2/R3 drafts (preserved as audit trail per MR-11).

---

## §6 Version-History Annotation (§77.4 Template)

Every in-place amendment to a SPC rule per Rule #77 carries this annotation in its revision log (SYNTHESIST aggregation requirement per R3 D-04):

```
Rule #XX — Revision History

- 2026-04-DD: original activation predicate
    <verbatim original text>
  (ratified under <cycle> Master Ratification Batch <ID>).

- 2026-04-DD (cycle <N> R3 D-<ID>, Batch <M>): in-place amendment
    <verbatim amended text>
  per <reference-chapter> finding.
  Dissent preserved (MR-11): <N> agents (<agent-IDs>) preferred <alternative> for <rationale>.
  R3 vote: <tally>.
```

### §6.1 Worked example — Rule #72 FR-2 revision log (per this cycle)

```
Rule #72 FR-2 — Revision History

- 2026-04-18: original activation predicate
    max(σᵢ) − min(σᵢ) ≤ 0.0001 for N ≥ 3 consecutive rounds
  (ratified under cycle03-11 Master Ratification Batch 2-A).

- 2026-04-24 (cycle03-13 R3 D-04, Batch 10): in-place amendment
    AND (σ_regime ∈ {llm_variance, mixed})
  per §四 finding (σ = composition index, not stochastic variance estimator).
  Dissent preserved (MR-11): 5 agents (MESH / PASCAL / PENROSE / KNUTH / RUSSELL)
  preferred DEACTIVATE + reformulate for clean-break rationale.
  R3 vote: 18/5.
```

This annotation satisfies D-04 5-agent dissenters' transparency concern without reverting the in-place decision.

---

## §7 Dissent Roll-Up (MR-11 preservation, D-04 5-agent minority)

Per MR-11, the D-04 minority is preserved with full rationale:

### §7.1 MESH (#4, Distributed Systems) — clean-break

Clean deactivation + re-introduction clarifies the distributed-protocol contract boundary. In-place amendment risks ambiguity in cross-cycle comparisons where rule versioning matters for downstream consumers.

### §7.2 PASCAL (#19, Decision Theory) — statistical mixed-regime

In-place conjunct mixes regime semantics within a single rule; clean break avoids mixed-regime contamination in future SPC audits when Phase 6+ LLM-variance enters.

### §7.3 PENROSE (#18, Meta-Principle) — learning-oriented

Deactivation forces reassessment, which aligns with the learning-not-punitive spirit of MR-11. In-place amendment can be perceived as papering-over rather than engaging the lesson.

### §7.4 KNUTH (#21, Algorithms) — simpler tools

Two-state `active / dormant` via regime tag is algorithmically more complex than one-state `replace`; simpler tools reduce bug surface.

### §7.5 RUSSELL (#23, Agent Theory) — honest re-derivation

In-place amendment leaves the pre-σ_regime derivation partially load-bearing; honest re-derivation post-§四 finding favors a structural break that makes the conceptual shift visible.

### §7.6 Why majority chose in-place revise (reiteration)

Per §2.2: ratification-trail preservation + σ_regime tag as forward-compatible governance artifact + metadata-extension > rule-replacement pattern + version-history annotation mandate (§77.4) addresses transparency. D-04 18/5 adopted; 5 dissent entries formally preserved here + in `R3_D_items_voting.md §Round B D-04`.

---

## §8 Relationship to ZT-3 (Control-Range Triple-Check)

Per R3 Compliance Audit (§8 of R3_decision_log), **ZT-3 strengthens (not loosens) under σ_regime**:

- ZT-3 triple-check (K-S + Anderson-Darling + QQ) remains applicable when σ_regime ∈ {llm_variance, mixed} (stochastic signal present → distributional tests meaningful).
- Under σ_regime = composition_index, ZT-3 triple-check is rule-textually retained but **not triggered** because the pipeline is deterministic (distributional tests are uninformative when there is no stochastic variance to test).
- Plan50 σ_regime machinery makes this explicit at the predicate level; SCRIBE G6.7 auto-veto gate evaluates only when σ_regime ∈ {llm_variance, mixed}.

---

## §9 Cross-References and Author Attribution

### §9.1 Cross-refs within `openstarry_doc/`

- `Reference/06_P_Coincidence_Binding_Convention.md` (Rule #76 §76.1–§76.6 predecessor, cycle 03-12).
- `Reference/08_Rule_76_Section_76_7_Caveat.md` (companion σ-regime caveat for Rule #76, this cycle).
- `Research_Methodology/08_Rule_72_N10_SubClause_Draft.md` (cycle 03-12 Rule #72 §72.1–§72.7 — §77.3 amends §72's FR-2 predicate).
- `Research_Methodology/11_ENG_FAB_v1.8_Binding.md` §E (F-13 Plugin Hook Dispatch Verified — σ_regime detection via plugin introspection; Plan50+).
- `Calibration_Reports/18_PathC_20Field_Monitoring_Format.md` (cycle 03-12, `per_category_sigmas` field #18 → Plan50 σ_regime tagging extends the per-category structure).
- `Technical_Specifications/Plan50_Sigma_Regime_Binding.md` (this cycle, σ_regime ~50 LOC Option γ machinery).

### §9.2 Research record (cycle 03-13)

- `deliver/O3_core_trio_final.md §B` — §四 σ metric semantic clarification; D-04 in-place decision + σ_regime specification.
- `deliver/Master_Ratification/Batch_10_Request.md Item 8` — Rule #77 σ_regime ratification request.
- `deliver/R3_decision_log.md §4.2 D-04 + §4.5 D-05 + D-06 + D-09 + §3 CV-07 + CV-08` — R3 voting record.
- `deliver/R3_D_items_voting.md §Round B D-04 + §Remaining D-05/06/09` — 23-agent vote tables.
- `todo/Plan50_sigma_regime_spec.md` — Dev-facing implementation spec.

### §9.3 Baseline rules

- `baseline_rules.md` Rule #77 — SCRIBE appends Rule #77 text verbatim from §1 above on Master ratification complete.
- `baseline_rules.md` Rule #72 FR-2 — SCRIBE amends in-place per §77.3 + §6.1 revision-history annotation.

### §9.4 Author attribution

- **Primary derivation and FR-2 control-theory lens**: WIENER (#12).
- **Protocol-evolution / in-place rationale**: LEIBNIZ (#14) — R2 primary elevated to R3 vote.
- **Engineering spec (Plan50 ~50 LOC Option γ)**: ARCHIMEDES (#16).
- **Statistical methodology**: KNUTH (#21) — co-authored majority path while preserving dissent on algorithmic simplicity grounds (§7.4).
- **Rule-text discipline**: SCRIBE (#2).
- **Consolidation**: SYNTHESIST (#1), SUNYATA (#0, chair).
- **Dissent agents preserved**: MESH (#4) / PASCAL (#19) / PENROSE (#18) / KNUTH (#21) / RUSSELL (#23).
- **R3 vote record**:
  - **D-04 18/5** (in-place σ_regime conjunct / DEACTIVATE-and-reformulate).
  - **D-05 UNANIMOUS 23/0** (Plan50 inclusion).
  - **D-06 UNANIMOUS 23/0** (+2 impact list additions: Principles/02 + cycle 03-9 O7).
  - **D-09 UNANIMOUS 23/0** (Batched via Batch 10).
  - **CV-07 UNANIMOUS tacit** (σ = composition index).
  - **CV-08 UNANIMOUS tacit** (V11 = numerical artifact).

---

*Rule #77 — SPC Pooled-Mode σ-Regime Conjunct — Cycle 03-13 — 2026-04-24 R4 close*
*BINDING new Rule per Master ratification 2026-04-24 via Batch 10 Item 8*
*D-04 18/5 (5 dissent MESH / PASCAL / PENROSE / KNUTH / RUSSELL preserved) + D-05/06/09 UNANIMOUS + CV-07/08 tacit*
