# Plan50 — σ_regime In-Place Annotation Binding (Option γ ~50 LOC)

> **Front-matter**
> - **Cycle**: 03-13
> - **Date**: 2026-04-24
> - **Authors**: ARCHIMEDES (#16, Plan50 spec lead + Option γ) + WIENER (#12, σ semantics + FR-2 amendment) + LEIBNIZ (#14, in-place rationale) + SCRIBE (#2, spec discipline) + SYNTHESIST (#1, consolidation)
> - **Binding R3 decisions**: **D-04 18/5** (in-place σ_regime conjunct on FR-2 predicate; 5 dissent MESH/PASCAL/PENROSE/KNUTH/RUSSELL preferred DEACTIVATE-and-reformulate, preserved per MR-11) · **D-05 UNANIMOUS 23/0** (σ_regime inclusion in Plan50) · **D-06 UNANIMOUS 23/0** (9-item impact list; 7 R1 primary + 2 R2 additions) · **CV-07 UNANIMOUS tacit** (σ = composition index per §四 finding) · **D-09 UNANIMOUS** (Batched via Batch 10 Item 8)
> - **Status**: **BINDING for Plan50 delivery** per Rule #77 σ_regime conjunct (ratified via Master Batch 10 Item 8, 2026-04-24). Plan50 spec document (this file) anchors Dev implementation target = Plan50 (cycle 03-14 Dev cycle kick-off).
> - **Supersedes**: nothing — new σ_regime machinery layer on top of existing σ computation; existing σ = 0.023753 baseline preserved per §5 below.
> - **Cross-refs**: `Reference/09_Rule_77_SPC_Pooled_Mode_Sigma_Regime.md` (Rule #77 authoritative text) · `Reference/08_Rule_76_Section_76_7_Caveat.md` (§76.7 caveat trigger per §3.4 below) · `Research_Methodology/11_ENG_FAB_v1.8_Binding.md` §E (F-13 plugin hook dispatch — plugin introspection for σ_regime tagging) · `Technical_Specifications/Plan50_pushInput_CP4_Invariant.md` (cycle 03-12 Plan50 companion spec; σ_regime may bundle with pushInput or stand alone) · `Technical_Specifications/Plan49_MR6_Conditional_Gates.md` (Plan49 precedes Plan50 per MR-12) · `research record/cycle03-13/deliver/O3_core_trio_final.md §B.4 + §B.5` (FR-2 amendment derivation + Plan50 machinery) · `research record/cycle03-13/todo/Plan50_sigma_regime_spec.md` (Dev-facing spec, 320 lines)

---

## §1 What — σ_regime In-Place Annotation

### §1.1 Objective

Annotate every σ observation with a tag `σ_regime ∈ {composition_index, llm_variance, mixed}` so that downstream consumers (SPC rules, FR-2 pooled-mode, Rule #76 §76.7 caveat machinery) can branch on the regime without requiring ad-hoc heuristics.

### §1.2 Approach — In-Place Revise (R3 D-04 adopted)

**In-place revise**: extend σ metric machinery with a regime tag field; keep existing σ computation unchanged. Do **NOT** deactivate-and-reformulate (rejected by R3 majority 18/5; 5-agent dissent preserved per `Reference/09_Rule_77_SPC_Pooled_Mode_Sigma_Regime.md §7`).

### §1.3 Rationale (§四 R1 §B.1 / §B.4; R2 §B.4 LEIBNIZ; R3 D-04)

- σ, as currently computed, is a deterministic composition index over an event-count vector × constant-value-lookup table. It carries **no stochastic signal** under the current pipeline (per `O3_core_trio_final.md §B.1 + CV-07 UNANIMOUS`).
- Explicit regime tagging surfaces this structural fact to every downstream consumer, replacing the implicit assumption that σ is stochastic.
- When Phase 6+ LLM-reflective plugins or per-event LLM-risk-score injection arrives (§三 Option 2 / §四 Option α per O6), new σ observations can be tagged `llm_variance` or `mixed`, and the FR-2 + Rule #76 machinery naturally branches without further rule rewrites.

---

## §2 LOC Target — ~50 LOC (Option γ)

Per R1 §B.4.2 Option γ estimate + `todo/Plan50_sigma_regime_spec.md §2`:

| Change surface | Estimated LOC | Type |
|----------------|--------------:|------|
| σ-metric struct field addition (`σ_regime: string`) | ~5 | data-model |
| σ emission paths (`audit:completed` event carrier, W2 test harness summarisers, Path-C 20-field record) | ~15 | data-flow |
| FR-2 activation predicate conjunct guard | ~3 | rule evaluation |
| Rule #76 P(coincidence) evaluation caveat trigger | ~5 | rule evaluation |
| σ_regime inference helper (default = `composition_index`; upgrade when LLM-risk signal present) | ~15 | logic |
| Migration / retroactive tagging for pre-Plan50 σ observations | ~7 | migration |
| **Total prod** | **~50 LOC** | |
| Tests (unit tests per surface + regression fixture for R10/R11/R12 baseline) | ~40 LOC | test |

**Combined**: ~50 prod + ~40 test = ~90 LOC. Well within Plan50 S-1 (500–1000 LOC) budget; leaves ample budget for Plan50 `gear-arbiter-dynamic` rename bundle (D-01 20/3 tabled) + `pushInput` spec (cycle 03-12 precedent).

---

## §3 Specific Modification Points

### §3.1 σ-metric data-model

**File** (Dev to confirm exact path during Plan50 kickoff): `openstarry-plugin/spc-monitor/src/sigma_metric.ts` (or equivalent σ-computation module).

Add field to σ observation record:

```
sigma_observation = {
  round_id: string,
  sigma: float,
  ucl: float, lcl: float,
  N_events: int,
  mean: float,
  ...
  sigma_regime: "composition_index" | "llm_variance" | "mixed"  # NEW
}
```

**Default tag** = `composition_index`. Upgrade decision logic (§3.5) sets the tag when LLM-variance signal enters the computation.

**Type enforcement** — the σ_regime field is required (no nullable); schema validation (C-series contract compliance per ENG-FAB v1.8) enforces presence at emission time.

### §3.2 σ emission paths

Consumers / emitters that write σ observations:
- **`audit:completed` event carrier** (if σ is surfaced on event level — Dev to verify; may be separate from per-round σ).
- **W2 test-harness per-round summariser** (emits `w2rN_sigma = <value>` to `test_report_w2rN.md`).
- **Path-C 20-field record** (each per-round record in SPC store per `Calibration_Reports/18_PathC_20Field_Monitoring_Format.md` cycle 03-12).

All emitters must propagate the σ_regime tag from the computation to the output artifact. **No emitter may write σ without the tag** (enforced via data-model type).

### §3.3 FR-2 activation predicate

**File** (Dev to confirm): `openstarry-plugin/spc-monitor/src/rule72_fr2.ts` (or wherever the FR-2 pooled-mode activation check lives).

**Current predicate (pre-Plan50)**:
```
FR-2 pooled-mode activates iff
  max(σᵢ) − min(σᵢ) ≤ 0.0001 for N ≥ 3 consecutive rounds
```

**Amended predicate (Plan50 conjunct per Rule #77 §77.3)**:
```
FR-2 pooled-mode activates iff
  (max(σᵢ) − min(σᵢ) ≤ 0.0001 for N ≥ 3 consecutive rounds)
  AND (σ_regime ∈ {llm_variance, mixed})
```

**Current pipeline**: σ_regime = `composition_index`, so the second conjunct is **false**. FR-2 is dormant without requiring a separate deactivation step.

### §3.4 Rule #76 §76.7 caveat trigger

**File** (Dev to confirm): wherever P(coincidence) claims are checked against §76.6 reproducibility block structure (likely tied to `audit_calc.py scan-log` mode integration, §八).

**Claim-evaluation flow**:
```
if claim.type == "P(coincidence)":
    if claim.sigma_regime == "composition_index":
        emit_caveat("§76.7 applies: σ_regime = composition_index; "
                    "bound is a sampling-floor statement, not literal coincidence probability.")
    else:  # llm_variance or mixed
        # literal stochastic coincidence interpretation applies
        standard_p_coincidence_check(claim)
```

Annotation does **NOT block or FAIL** the claim; it annotates it with the §76.7 caveat for audit readers.

### §3.5 σ_regime inference helper

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

### §3.6 Plugin introspection rules (Dev to flesh out during Plan50)

- A plugin emitting `clamped` values that are NOT drawn from a static constant table (e.g., reads from an LLM response, samples from a stochastic distribution, uses time-of-day entropy) MUST declare `is_llm_derived = true` (or equivalent flag).
- `gear-arbiter-dynamic` is by-design `is_llm_derived = false` until Phase 6+ O2 enhancement (per `O3_core_trio_final.md §A.4.4`).
- `static-rule-arbiter` / `tool_audited:*` are trivially `is_static_lookup = true`.
- See `Research_Methodology/11_ENG_FAB_v1.8_Binding.md §E` (F-13 Plugin Hook Dispatch) for plugin introspection cross-link — F-13.2.3 manifest check can validate `is_llm_derived` declaration consistency.

---

## §4 3-Round Identity Property Preservation

### §4.1 Mandate (per Rule #77 §77.5)

The 3-round σ identity R10 = R11 = R12 = 0.023753 (6-decimal identical, observed 2026-04-20) is retained as a **factual observation** of the pipeline:
- σ values recorded verbatim in `test_report_w2rN.md`.
- Path-C 20-field records preserve the identity across rounds.
- The identity is noted in `MEMORY.md` factually (see cycle03-13 post-R3 snapshot).

### §4.2 Interpretation governed by Rule #76 §76.7

Under σ-deterministic regime (σ_regime = `composition_index`), the 3-round identity is interpreted as:

> "Sampling-floor statement reflecting pipeline deterministic reproduction; not literal coincidence probability; NOT LTI regime evidence in the stochastic sense."

Under future σ-stochastic regime, the identity would carry stronger interpretation per Rule #76 §76.1–§76.5 (literal probability upper bound). Transition is handled automatically via σ_regime tag flip.

### §4.3 Acceptance criterion

The Plan50 regression test suite MUST include a **fixture** reproducing the R10/R11/R12 σ = 0.023753 baseline + asserting σ_regime = `composition_index` tag on each observation. This ensures the in-place amendment does not accidentally change σ values.

---

## §5 Migration and Backward-Compatibility

### §5.1 Baseline preservation

Per `O3_core_trio_final.md §B.7` + Plan50 spec §2: **the existing σ = 0.023753 baseline is preserved verbatim**. σ_regime annotation does NOT trigger a re-baseline; it only adds a metadata field.

### §5.2 Retroactive tagging (~7 LOC migration)

All pre-Plan50 σ observations are retroactively tagged `composition_index` by the migration helper. Rationale: every pre-Plan50 σ observation derives from the static-rule-arbiter + tool_audited lookup-table pipeline (CV-07 UNANIMOUS); none has LLM-variance source.

### §5.3 W2-R13 shift (0.023753 → 0.023993) is composition-shift, not drift

Per R1 §B.3: W2-R13 observed σ = 0.023993 (vs R10/R11/R12 = 0.023753, +0.000240 / +1.01%) is explained by Plan48 event-type additions changing the event-count composition vector `c`:
- Plan48 added `structured-log` events, `audit-sink` events, and `hmac-cleanup` events.
- Composition shift in `c` alters closed-form σ output without invoking any stochastic signal.

This shift does NOT trigger any drift-detection rule (Rule #72 / Rule #77 drift predicates apply to stochastic signals; composition_index shifts are re-baseline events). R13 establishes a new deterministic σ baseline = 0.023993 for the Plan48+ codebase.

Plan50 regression fixture asserts both baselines (0.023753 pre-Plan48; 0.023993 post-Plan48) with σ_regime = `composition_index` on both.

---

## §6 Acceptance Criteria (Plan50 Delivery Time)

### §6.1 PA-level criteria (all MUST PASS for Plan50 δ δelivery)

1. **PA-1** — σ_regime field present on every σ observation record. Test: schema validation passes on 100% of observations in a full W2-R14 round.
2. **PA-2** — σ_regime inference helper correctly tags `composition_index` for all 03-13-era and pre-Plan48 observations; correctly tags `llm_variance` / `mixed` in simulated LLM-injection test fixtures.
3. **PA-3** — FR-2 activation predicate correctly evaluates to **false** in σ-deterministic regime (dormant); correctly evaluates to **true** in simulated LLM-variance regime with 3-round identity simulation.
4. **PA-4** — Rule #76 §76.7 caveat emission correctly triggers on P(coincidence) claims with σ_regime = `composition_index`; does NOT trigger for `llm_variance` / `mixed`.
5. **PA-5** — Baseline σ = 0.023753 (R10/R11/R12) and σ = 0.023993 (R13) preserved byte-identical post-Plan50.
6. **PA-6** — MR-6 Core zero-policy constraint preserved (no Core additions; Plan50 modifications all at SPC-monitor / plugin / emission layer).
7. **PA-7** — Rule #74 L1' doc sync (README + CHANGELOG + Architecture doc for σ_regime machinery).
8. **PA-8** — ENG-FAB v1.8 audit PASS for Plan50 (v1.8 baseline from Plan50+ per D-21 timing).
9. **PA-9** — F-15 research acceptance report front-matter block present (§C49-M8 analog for Plan50 research verification).
10. **PA-10** — Version-history annotation per Rule #77 §77.4 template recorded in `baseline_rules.md` Rule #72 FR-2 revision log (SCRIBE writes; post-ratification sync).

### §6.2 Test coverage criteria

- ≥ 90% branch coverage for σ_regime inference helper.
- ≥ 1 integration test per emission path (§3.2): audit:completed + W2 harness + Path-C record.
- ≥ 1 W2-R14 runtime verification: σ_regime present on every observation + correct composition_index tag.

---

## §7 Relationship to Plan49 and Plan50 Companion Specs

### §7.1 Plan49 precedence

Plan49 (cycle 03-14 Dev kickoff) precedes Plan50 per MR-12 back-fill priority (CV-14 UNANIMOUS). Plan49 addresses:
- C49-M7e O4 doc artefact for `gear-arbiter-dynamic` identity (per D-11 UNANIMOUS).
- Plan49 W1 `riskCategory` wire-in (per D-03 UNANIMOUS, O3).

Plan50 then builds on Plan49's identity clarification to apply σ_regime machinery.

### §7.2 Plan50 `pushInput_CP4_Invariant` companion (cycle 03-12 precedent)

Plan50 as specced in cycle 03-12 (`Technical_Specifications/Plan50_pushInput_CP4_Invariant.md`) covers:
- pushInput CP-1/2/3/4 invariants.
- CR-SCK SDK-exported sourceContext keys.
- CR-PARETO three-tier red-line taxonomy.

The cycle 03-13 σ_regime (this document) may **bundle with** or **stand alone from** Plan50 pushInput — coordinator / Dev discretion at Plan50 kick-off. The two spec documents are compatible (non-overlapping change surfaces: pushInput at core/SDK boundary; σ_regime at SPC-monitor/plugin boundary).

### §7.3 `gear-arbiter-dynamic` rename option (D-01 dissent)

Per D-01 20/3 dissent preservation (WIENER / LEIBNIZ / TANENBAUM minority preferred in-Plan49), the rename `gear-arbiter-dynamic` → `gear-arbiter-wiener` is **tabled to Plan50**. Plan50 may bundle:
- σ_regime annotation (this document, ~50 LOC).
- pushInput CP-4 invariant (cycle 03-12 companion, ~LOC TBD).
- Optional `gear-arbiter-dynamic` rename (~50 LOC + migration shims + dual-name support).

Total Plan50 budget remains within S-1 (500–1000 LOC) with all three.

---

## §8 Dissent Roll-Up (MR-11 preservation, D-04 5-agent minority)

Per MR-11, the D-04 5-agent minority is preserved (brief form; full rationale at `Reference/09_Rule_77_SPC_Pooled_Mode_Sigma_Regime.md §7`):

| Agent | Lens | Rationale (abridged) |
|-------|------|----------------------|
| MESH (#4) | Distributed systems | Clean-break clarifies cross-cycle contract boundary |
| PASCAL (#19) | Decision theory | Mixed-regime semantics risk in single rule |
| PENROSE (#18) | Meta-principle | Deactivation forces reassessment aligned with MR-11 learning spirit |
| KNUTH (#21) | Algorithms | Two-state rule algorithmically more complex than replacement |
| RUSSELL (#23) | Agent theory | Honest re-derivation favours structural break |

**Majority (18) rationale**: see Rule #77 §2.2 — ratification-trail preservation + σ_regime tag as forward-compatible governance artifact + version-history annotation (§77.4) addresses transparency without reverting in-place decision.

---

## §9 Status and Author Attribution

- **Status**: **BINDING spec for Plan50 delivery** per Rule #77 (Master Batch 10 Item 8 APPROVED 2026-04-24).
- **Dev implementation target**: Plan50 (cycle 03-14 Dev kickoff or later per coordinator Plan cadence).
- **Research verification target**: Plan50 delivery-time acceptance audit.
- **Primary authorship**:
  - ARCHIMEDES (#16) — Plan50 spec + Option γ ~50 LOC design.
  - WIENER (#12) — σ semantics + FR-2 amendment derivation.
  - LEIBNIZ (#14) — in-place rationale (primary R2 elevated via R3 vote).
  - SCRIBE (#2) — spec discipline + MRB-14 resolution.
  - SYNTHESIST (#1) — consolidation.
  - SUNYATA (#0) — chair (procedural abstain).
- **Dissent agents preserved** (D-04 5-agent minority): MESH (#4), PASCAL (#19), PENROSE (#18), KNUTH (#21), RUSSELL (#23).
- **R3 vote record** (ground truth `R3_decision_log.md §4.2 + §4.5`):
  - **D-04 18/5** (in-place σ_regime conjunct on FR-2).
  - **D-05 UNANIMOUS 23/0** (Plan50 inclusion).
  - **D-06 UNANIMOUS 23/0** (9-item impact list).
  - **D-09 UNANIMOUS** (batched via Batch 10).
  - **CV-07 UNANIMOUS tacit** (σ = composition index).

---

*Plan50 σ_regime In-Place Annotation Binding — Cycle 03-13 — 2026-04-24 R4 close*
*BINDING for Plan50 delivery per Master Batch 10 Item 8 + Rule #77 ratification 2026-04-24*
*D-04 18/5 (5 dissent preserved) + D-05/06/09 UNANIMOUS + CV-07 tacit*
*Dev target: cycle 03-14 Plan50 kickoff; ~50 LOC prod + ~40 LOC test*
