# Rule #76 §76.7 σ-Regime Caveat (In-Place Addition)

> **Front-matter**
> - **Cycle**: 03-13
> - **Date**: 2026-04-24
> - **Authors**: ARCHIMEDES (#16, §三+§四+§五 trio primary) + PASCAL (#19, decision-theoretic methodology) + KNUTH (#21, reproducible calc) + NAGARJUNA (#7, two-truth framing) + SCRIBE (#2, Rule #76 addition discipline) · ATHENA (#5) dissent preserved per MR-11
> - **Binding R3 decisions**: **D-07 22/1** (O4 retain + §76.7 caveat primary; 1 ATHENA O2 operational lean preserved) · **D-08 UNANIMOUS 23/0** (explicit Master ratification-category request in Batch 3-A' cover letter) · **MRB-14 RESOLVED** (Batch 3-A' cover letter template) · **CV-02 UNANIMOUS** (§76.6 Reproducible Calc Mandate retained unconditionally) · **CV-10 tacit** (§五 O4 retain + §76.7 caveat primary) · **CV-22 tacit** (P-1 arithmetic (10⁻⁴)^12 = 10⁻⁴⁸ PASS under Method A)
> - **Status**: **BINDING in-place addition per Master Option A** (in-place scope clarification, not formal re-ratification). Master determined category on 2026-04-24 Batch 3-A' Addendum. Additive only — nothing in §76.1–§76.6 removed.
> - **Scope**: Rule #76 §76.7, effective **2026-04-24 R4 close onward**. Pre-03-13 applications (cycles 03-11 / 03-12) **regime-annotated retrospectively** (not retroactively overturned) per §5 below.
> - **Supersedes**: nothing — strictly additive. Predecessor convention `Reference/06_P_Coincidence_Binding_Convention.md` (cycle 03-12 RATIFIED 2026-04-23 Path A standalone) remains BINDING; §76.7 appends to it without displacing §76.1–§76.6.
> - **Cross-refs**: `Reference/06_P_Coincidence_Binding_Convention.md` (Rule #76 §76.1–§76.6 predecessor) · `Reference/09_Rule_77_SPC_Pooled_Mode_Sigma_Regime.md` (σ_regime conjunct + Rule #77) · `Research_Methodology/11_ENG_FAB_v1.8_Binding.md` §D (F-12 operational enforcement of §76.6) · `Technical_Specifications/Plan50_Sigma_Regime_Binding.md` (σ_regime tagging machinery) · `research record/cycle03-13/deliver/O3_core_trio_final.md §C` (derivation)

---

## §1 §76.7 Caveat — Authoritative Text (Verbatim)

Per R3 D-07 22/1 adoption + CV-10 + Master Option A Batch 3-A' Addendum ratification 2026-04-24:

> **§76.7 Scope caveat — When σ is demonstrably rule-deterministic (σ is a composition index rather than stochastic variance, e.g. via 3-round identity or event-count vector hash identity), `P(coincidence) ≤ 10⁻⁴⁸` is a conservative upper bound reflecting sampling discipline, not a literal coincidence probability. The bound remains sound as a sampling-floor statement; it is not a statement about LLM stochasticity in these cases. See cycle03-13 R4 O3 §C (core trio chapter) for derivation; see Plan50 σ_regime annotation for the regime-tagging machinery.**

The §76.7 text is added as an **in-place clarification** to Rule #76 — not a substantive re-ratification of §76.1–§76.6. Rule #76's authoritative form statement (§76.1), acceptable phrasings (§76.2), forbidden forms (§76.3), disclosure requirement (§76.4), SCRIBE G5 enforcement (§76.5), and Reproducible Calculation Mandate (§76.6) are **all unchanged**.

### §1.1 Version anchor

Rule #76 prior to 03-13 was ratified 2026-04-23 (Master Batch 3-A APPROVED 2026-04-23) with §76.1–§76.6 content. §76.7 is added 2026-04-24 (Batch 3-A' Addendum + Batch 10 Item 7 APPROVED by Master 2026-04-24) without version-number increment — treated as in-place scope clarification per Master Option A determination.

---

## §2 Applicability — When §76.7 Triggers

§76.7 applies when a `P(coincidence)`-class probability claim is asserted and the underlying σ metric supporting that claim is in a **σ-deterministic regime**.

### §2.1 σ-deterministic regime defined

Per `O3_core_trio_final.md §B.1 (CV-07 UNANIMOUS)`, σ as currently implemented in the W2 pipeline is a **deterministic composition index over an event-count vector**, not a variance estimator over a stochastic signal:

> Event-level σ closed form:
> `N = Σₖ cₖ` (total event count)
> `μ = (Σₖ cₖ · vₖ) / N` (mean)
> `σ² = (Σₖ cₖ · (vₖ − μ)²) / (N − 1)` (sample variance with Bessel correction)
>
> where `cₖ` is the event count per category (static-rule-arbiter informational, tool_audited:fs.list, ..., gear-arbiter-dynamic) and `vₖ` is the per-category constant (0.001, 0.000055, ..., 0) drawn from static rule + tool_audited lookup tables. Neither `cₖ` nor `vₖ` contains a stochastic term under the current pipeline.

Given held-constant fixture + deterministic pipeline, σ output is **deterministic**, and 3-round σ identity (R10 = R11 = R12 = 0.023753 to 6 dp observed 2026-04-20) is a mathematical certainty — **NOT LTI regime evidence in the stochastic sense**.

### §2.2 Trigger criteria (any one sufficient)

A P(coincidence) claim enters §76.7-annotated status iff **any** of the following detection criteria applies:

1. **3-round σ identity**: σ values across ≥ 3 consecutive rounds are byte-identical to 6 decimal places (e.g., R10/R11/R12 = 0.023753 per 2026-04-20 observation).
2. **Event-count vector hash identity**: the per-category event-count vector `c = (c₁, c₂, ..., cₙ)` produces an identical hash across ≥ 2 rounds, indicating structural deterministic composition (no variation in the composition basis).
3. **`audit_calc.py` tier-invariance triple-hash PASS under `@tautology` marker**: the reproducible-calc tooling (Plan49+ per F-12) reports `(corpus_sha256 + script_sha256 + python_env_hash)` match across tiers with the one-line `@tautology` marker appended to the calculation block (D-25 UNANIMOUS editorial).

**Detection responsibility**: per F-12 evidence artefact (`{plan}/delivery_report.md §F-12_evidence`), the author asserting a P(coincidence) claim MUST declare whether the σ supporting the claim is σ-deterministic (§76.7 applies) or σ-stochastic (§76.1–§76.5 apply directly without caveat).

---

## §3 Relationship to §76.1–§76.6

§76.7 is a **scope-narrowing clarification** that does not remove or modify §76.1–§76.6. The relationship is conjunctive:

| Rule §     | Substance                                      | Effect of §76.7                                        |
|-----------:|------------------------------------------------|--------------------------------------------------------|
| **§76.1**  | Authoritative form `P(coincidence) ≤ BOUND` | Unchanged. §76.7 clarifies that under σ-deterministic regime, BOUND is a sampling-floor statement; under σ-stochastic regime, BOUND is a literal probability upper bound. |
| **§76.2**  | Acceptable phrasings (primary: `≤ 10⁻⁴⁸`) | Unchanged. Primary phrasing retained. |
| **§76.3**  | Forbidden forms (`P = 10⁻⁷²`, `P = 0`, etc.) | Unchanged. §76.7 does NOT permit forbidden forms in σ-deterministic regime. |
| **§76.4**  | Protocol D disclosure requirement             | Unchanged. Non-conservative point estimates still require Protocol D disclosure irrespective of regime. |
| **§76.5**  | SCRIBE G5 enforcement                          | Extended. G5 numerical-consistency scan now also validates §76.7 regime-annotation presence when σ-deterministic regime is detected (§2.2 criteria). |
| **§76.6**  | Reproducible Calc Mandate                     | **Retained unconditionally per CV-02 UNANIMOUS**. §76.6 is procedural — enables audit-ability regardless of σ regime; its retention is the mechanism that enabled §四's re-interpretation (§C.5 of O3). |
| **§76.7**  | σ-regime caveat (this document)               | NEW. Layered on top of §76.1–§76.6 — clarifies interpretation in σ-deterministic regime; does not alter procedural discipline. |

### §3.1 §76.6 retained unconditionally (CV-02 UNANIMOUS)

§76.6 Reproducible Calculation Mandate is **retained unconditionally** regardless of σ regime. Rationale (R1 §C.4; R2 §C.5 STRONGLY CONFIRMED; R3 CV-02 UNANIMOUS):

1. §76.6 is **procedural, not substantive**. It requires any P(x) claim in binding documents to carry an adjacent `Inputs / Formula / Steps / Citation` block. This enables audit-ability, which is independent of whether any specific claim's premises hold.
2. §76.6 is the **mechanism that enabled §四's re-interpretation** (cycle 03-13). The reproducible-calc block in `Reference/06` §7.1.1 made H_coincidence's implicit assumptions explicit — which is precisely how PASCAL / KNUTH could formulate the composition-index counterargument. Retracting §76.6 would remove the audit-ability that just caught the over-claim.
3. §76.6 is Master's 補強 #1 (2026-04-18 ZT-1 framing). Retracting it would reverse a Master directive without justification.

**Portability note** (R2 F-R2-§5-c appendix): should a hypothetical future Rule #76 retraction occur on any grounds, §76.6 must port cleanly to a new standalone rule so that §76.6 is never momentarily orphaned. Under current O4 adoption this concern is moot, but the safety net is preserved.

### §3.2 When §76.1–§76.5 apply directly (σ-stochastic regime)

When σ is **genuinely stochastic** (LLM variance surfaces; per-event risk scores non-constant; compositional invariance breaks), §76.1–§76.5 apply directly **without §76.7 annotation**. In that regime:

- `P(coincidence) ≤ 10⁻⁴⁸` or other documented bounds interpret literally as probability upper bounds under H_coincidence.
- The rule machinery (acceptable phrasings, forbidden forms, Protocol D, G5 scan) operates as originally drafted.

Future arrival of σ-stochastic regime (Phase 6+ LLM-reflective plugins per §三 O2; Plan51+ σ_llm per §四 Option α) will trigger this clause naturally via `σ_regime` tagging (see `Reference/09_Rule_77_SPC_Pooled_Mode_Sigma_Regime.md` + `Technical_Specifications/Plan50_Sigma_Regime_Binding.md`).

---

## §4 Relationship to Batch 3-A (cycle 03-12) and Batch 10 Item 7 (cycle 03-13)

### §4.1 Batch 3-A predecessor (cycle 03-12, APPROVED 2026-04-23)

Master APPROVED Rule #76 + §76.6 Reproducible Calc Mandate on 2026-04-23 under Path A (standalone Rule #76). That ratification stands. See `research record/cycle03-12/deliver/Master_Ratification/Batch_3A_P_Coincidence_Binding.md §12` for Master decision record.

### §4.2 Batch 3-A' Addendum (cycle 03-13, this cycle's request)

Per **D-08 UNANIMOUS + MRB-14 RESOLVED**, cycle 03-13 research team submitted Batch 3-A' Addendum on 2026-04-24 R4 close explicitly requesting Master determination of §76.7's ratification category:

- **Option A — in-place scope clarification** (no new formal re-ratification; absorbed into Batch 10 Item 7 as a clarification). **Research team preference** per D-07 22/1.
- **Option B — substantive addition** (formal re-ratification of Rule #76).

Master ratified **Option A (in-place clarification)** on 2026-04-24. §76.7 takes effect as of R4 close; cycles 03-11 / 03-12 applications are **retrospectively regime-annotated** (not retroactively overturned).

See `research record/cycle03-13/deliver/Master_Ratification/Batch_3A_prime_Addendum.md` for full addendum text + ATHENA O2 dissent preservation.

### §4.3 Batch 10 Item 7 (companion ratification)

`Batch_10_Request.md Item 7` (Master APPROVED 2026-04-24) acknowledges the Batch 3-A' Addendum and ratifies the §76.7 caveat text as the primary Rule #76 extension; Batch 3-A' Addendum carried Master's ratification-category determination in the cover letter. The two documents together form the ratification trail.

---

## §5 Retrospective Regime-Annotation (cycles 03-11 / 03-12)

Per Master Option A determination, prior cycles' P(coincidence) applications are **retrospectively annotated with σ-regime**, **not retroactively overturned**.

### §5.1 Cycle 03-11 (W2-R11 analysis)

- `research record/cycle03-11/deliver/O2_W2R11_full_analysis.md §3.3`: PASCAL original P(coincidence) ≤ 10⁻⁴⁸ derivation.
- **Regime annotation (added 2026-04-24)**: σ was σ-deterministic (composition index); §76.7 applies. Calculation correctness preserved under assumed model; applicability scope narrowed. The original derivation block satisfies §76.6 procedurally; R13-interpretation-caveat applies.

### §5.2 Cycle 03-12 (W2-R12 analysis + ratified Rule #76 base)

- `research record/cycle03-12/deliver/O2_W2R12_full_analysis.md §2.3 + §2.4`: 3-round σ identity + Rule #76 candidate text. **Regime annotation**: σ was σ-deterministic; §76.7 applies.
- `research record/cycle03-12/deliver/O6_PathC_monitoring_framework.md §1.2`: pillars table `P(coincidence) ≤ 10⁻⁴⁸ binding` row. **Regime annotation**: §76.7 cross-ref pointer added.
- `research record/cycle03-12/deliver/O7_Compliance_12_03.md §4`: 9/0/1★ σ-narrative. Per **CV-11 UNANIMOUS**, 9/0/1★ transitional achievement rests on **σ-independent evidence** (α/β/γ/δ non-σ): Plan47 K-3 PASS (architecture) / W2-R12 canary (enforcement) / Rule #74 + ENG-FAB v1.6 ratified (governance) / HMAC sync + F-L3-SEC-2/3 Plan48 (architecture). **No 9/0/1★ status change triggered by §76.7** (CV-12 UNANIMOUS endpoint 10/0/0★ unchanged per ZT-2 + MR-5).

### §5.3 Read-only boundaries (no regime annotation applied)

Per MR-11 historical integrity, the following are **not annotated** even retrospectively:

- `research input/research_version/**` (frozen read-only inputs)
- `research input/master_letters/**` (Master letters; preserved verbatim)
- `research input/test_data/**` (test_reports and metrics logs from cycles 03-11 / 03-12)
- Archived R1 / R2 drafts within cycle 03-12 that pre-date R3 ratification (preserved as audit trail)

---

## §6 ATHENA Dissent Preservation (D-07 22/1)

Per MR-11 dissent preservation:

**ATHENA's position (D-07 minority, 1/23 non-chair)**: prefer O2 rephrase (regime-conditional rephrase of §76.1 rather than §76.7 caveat). Operational rationale (R2 §C.4): from ops standpoint, O2 produces cleaner canonical rule text going forward. Caveat-footnote approach (O4) compounds over cycles — future readers encounter original §76.1 text + caveat pointer + cross-reference + trail — versus single clean rephrased §76.1 text. Operational effort ratio O4:O2 ≈ 1:1.5 — O2 ratification-batch cost is a one-time cost; reading-cost differential is ongoing.

**Why majority (22 agents) chose O4**:
1. **Minimal MR-10 back-fill**: existing `P(coincidence) ≤ 10⁻⁴⁸` language in cycle 03-11 / 03-12 / MEMORY can be retained with a caveat footnote pointer; does not require wholesale rewrite.
2. **Preserves full framework for future**: when LLM-variance σ arrives (Plan51+), Rule #76 is already in place; only σ_regime tag evaluation needed.
3. **Honors §76.6**: Master's 補強 #1 Reproducible Calc Mandate is untouched.
4. **Audit-trail-friendly**: original §76.1–§76.5 text preserved verbatim; caveat appended as §76.7 with explicit cycle 03-13 derivation pointer.
5. **ZT-3 compliant**: caveat language is conservative (disclosure of limitation), not a positive over-claim.

**R4 obligation under MR-11**: SCRIBE monitors operational enforcement through cycles 03-14+ for evidence that ATHENA's operational-lean concern materialises. If cycles 03-14/03-15/03-16 show compounded-reading-cost signals exceeding operational tolerance, a CR-route revisit at cycle 03-17+ retrospective is on the docket. Until that signal, O4 stands.

---

## §7 Cross-References

### §7.1 Within `openstarry_doc/`

- `Reference/06_P_Coincidence_Binding_Convention.md` — **predecessor** (cycle 03-12 RATIFIED 2026-04-23 Path A standalone); §76.1–§76.6 authoritative text.
- `Reference/09_Rule_77_SPC_Pooled_Mode_Sigma_Regime.md` — **companion Rule #77** (σ_regime conjunct on FR-2 pooled mode per D-04 in-place revise).
- `Research_Methodology/11_ENG_FAB_v1.8_Binding.md` §D — F-12 Reproducible Calc Verified ENG-FAB enforcement layer for §76.6.
- `Research_Methodology/12_GN_1_5_SCRIBE_G_Gate.md` — GN.1-5 SCRIBE authority gates; cross-ref F-15 / §76.7 detection discipline.
- `Calibration_Reports/18_PathC_20Field_Monitoring_Format.md` (predecessor cycle 03-12) — `per_category_sigmas` field (#18) is the data source for §2.2 criterion #2 (event-count vector hash identity).
- `Technical_Specifications/Plan50_Sigma_Regime_Binding.md` — σ_regime tagging machinery (~50 LOC Option γ; Plan50 delivery).

### §7.2 Research record (cycle 03-13)

- `deliver/O3_core_trio_final.md §C` — §五 Rule #76 P(coincidence) re-assessment; O4 primary decision + §76.7 caveat derivation.
- `deliver/Master_Ratification/Batch_3A_prime_Addendum.md` — Batch 3-A' cover letter + Master Option A determination record.
- `deliver/Master_Ratification/Batch_10_Request.md Item 7` — companion ratification.
- `deliver/R3_decision_log.md §4.5 D-07 + D-08 + §3 CV-02 + §3 CV-10 + §3 CV-22` — R3 voting record.
- `deliver/R3_D_items_voting.md §Remaining D-07 + D-08` — 23-agent vote tables.

### §7.3 Baseline rules (post-ratification state)

- `baseline_rules.md` Rule #76 §76.7 — SCRIBE appends §76.7 text verbatim from §1 above on Master ratification complete.

---

## §8 Author attribution and voting record

- **Trio chapter primary**: ARCHIMEDES (#16).
- **Derivation methodology**: PASCAL (#19 — decision theory; original P(coincidence) derivation + R2 re-derivation under composition-index framing) · KNUTH (#21 — statistical rigor, §四 composition-index argument).
- **Two-truth framing**: NAGARJUNA (#7) — conventional/ultimate level distinction per R3 D-20.
- **Retention mandate**: SCRIBE (#2) — §76.6 unconditional retention (CV-02).
- **Operational dissent**: ATHENA (#5) — D-07 O2 lean preserved per MR-11.
- **Cover letter template**: SCRIBE (#2) — MRB-14 RESOLVED.
- **Consolidation**: SYNTHESIST (#1) + SUNYATA (#0, procedural chair).
- **R3 vote** (ground truth `R3_decision_log.md §4.5` + `R3_D_items_voting.md §Remaining D-07/D-08`):
  - **D-07 22/1** (22 O4 retain + §76.7 caveat / 1 ATHENA O2 operational lean).
  - **D-08 UNANIMOUS 23/0** (explicit Master request in Batch 3-A' cover letter).
  - **MRB-14 RESOLVED** (cover letter template ready).
  - **CV-02 UNANIMOUS 23/0 tacit** (§76.6 unconditional retention).
  - **CV-10 tacit** (§五 O4 primary).
  - **CV-22 tacit** (P-1 arithmetic PASS under Method A).

---

*Rule #76 §76.7 σ-Regime Caveat — Cycle 03-13 — 2026-04-24 R4 close*
*BINDING in-place addition per Master Option A 2026-04-24*
*D-07 22/1 (1 ATHENA dissent preserved) + D-08 UNANIMOUS + MRB-14 + CV-02/10/22 tacit reaffirm*
