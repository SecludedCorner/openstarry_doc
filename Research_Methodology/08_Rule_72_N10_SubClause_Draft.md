# Rule #72 §72.1–§72.7 N ≥ 10 Sub-Clause (DRAFT)

> **Front-matter**
> - **Cycle**: 03-12
> - **Date**: 2026-04-25
> - **Authors**: WIENER (primary) + PASCAL + SCRIBE, consolidated by SYNTHESIST
> - **Binding R3 decisions**: D-06(a) pooled-event STEP 2 predicate · D-07(a) two-stage N=10 + N=20 · D-08(a) model-reset + code-flag · D-22(a) Batch 2-C merged with FR-1/FR-2 · D-27(b) `n_total − N` divisor fix · CV-39 UNANIMOUS N = R12 = N=1 STRONG ENDORSE
> - **Status**: **CANDIDATE** — awaiting Master ratification via **Batch 8c** (bundled with Rule #75 + ENG-FAB v1.7 per §5 R2 §13.3)
> - **Supersedes**: Rule #72 Step 2 wrapper (existing) — this sub-clause *amends* rather than replaces; existing Step 2 becomes a pointer to §72.1–§72.5 below.
> - **Cross-refs**: Rule #71 (FR-1 rolling reference) · FR-2 (Rule #72 pooled-mode zero-variance handling) · **Rule #76 RATIFIED 2026-04-23** (P(coincidence) ≤ 10⁻⁴⁸ binding + §76.6 reproducible calculation mandate) · `baseline_rules.md` · `Reference/06_P_Coincidence_Binding_Convention.md` · `Calibration_Reports/18_PathC_20Field_Monitoring_Format.md`

---

## §72.1 Scope and purpose

The Path-C SPC recalibration machinery (Rule #61 limits recomputation, V11 range reset, regime classification) triggers on cumulative **N ≥ 10 W2 rounds of the same model family, drawn from the post-transition-window regime** (per Rule #71's rolling-reference definition).

This sub-clause codifies:
1. The **starting point** of N for a given model family (pre-bound per CV-39 UNANIMOUS: N = 1 at W2-R12 for the current gpt-5.4 family).
2. N **increment semantics** per round outcome (GO / NO-GO / aborted / anomaly / transition-window).
3. The **two-stage N=10 provisional + N=20 finalize** verification cadence (per D-07(a)).
4. The **STEP 2 regime predicate** (pooled-event form, D-06(a)) used to detect stochastic drift versus deterministic LTI.
5. The **deployment fingerprint rule** distinguishing model-change N reset from code-change N continuation (D-08(a)).
6. The **Westgard pooled-std divisor** fix (`n_total − N`, per D-27(b)).
7. Cross-reference to the **P(coincidence) ≤ 10⁻⁴⁸ binding** (Reference/06) used to certify N-window distributional homogeneity.

---

## §72.2 N starting point (CV-39 UNANIMOUS, pre-bound)

- For model family *F*, **N = 1** begins at the **first post-transition-window round** whose rolling reference is computed entirely from rounds drawn from *F* (per Rule #71 rolling-reference formula).
- **Worked example (gpt-5.4)**: N = 1 at **W2-R12**. R10 and R11 lie in the transition window and are **retained** in `path_c_log.md` with `role = lti_anchor, not_recalibration_input` (see §72.3 below).

### §72.2.1 Rationale (BABBAGE + KNUTH + ATHENA fixed-point argument, §6 R2 §4.3)

1. **KNUTH rigor** — ZT-3 "same distribution" requires samples drawn under the regime governed by the recalibration. R12's rolling reference = mean(R10, R11) = pure gpt-5.4 reference; only R12 onward satisfies distributional homogeneity in both σ and evaluation context.
2. **BABBAGE formal** — The recalibration baseline must satisfy a **fixed-point property**: inputs to recalibration must be drawn under the regime the recalibration will govern. R10/R11 were evaluated under the *transition* regime; using them as recalibration inputs violates the fixed-point property.
3. **ATHENA operational** — R10/R11 are **not wasted**. Their three-round 6dp σ identical match (R10 = R11 = R12 to 6 decimals, P(coincidence) ≤ 10⁻⁴⁸; see Reference/06) is itself the strongest LTI evidence produced to date. Role = exterior validation, not recalibration input.
4. **Master-letter-literal alignment** — Master letter §六 line 2.2 literally reads "計算起點 = R12 (第一個 post-gpt-5.4 W2 round)". Path A (R12 = N=1) is the literal reading; alternatives would require explicit Master text override.

### §72.2.2 Zero-data-loss invariant

No W2 round is silently deleted from `path_c_log.md`. Both roles of R10/R11 — (a) seed for R12's rolling reference (Rule #71), (b) LTI-anchor exterior validator — are preserved. See `Calibration_Reports/18_PathC_20Field_Monitoring_Format.md` §2.1 fields 16/17 for the deployment-fingerprint audit.

---

## §72.3 N increment rules (per round outcome)

Per CV-36 AGREE + ATHENA audit-trail caveat (§6 R2 §3.2):

| Round outcome | N increment | `path_c_log.md` annotation |
|---------------|:-----------:|----------------------------|
| GO + `in_control` + no anomaly | **+1** | `role = recalibration_input` |
| NO-GO attributable to dev-layer fix (Rules #64–66) | **0 → re-run scheduled** | `role = suspended; rerun_pending` |
| Re-run after fix | **last-clean + 1** with audit footnote | `role = recalibration_input_rerun; footnote = <fix_reference>` |
| Anomaly (L2/L3 > 0 OR SPC breach OR infra fault) | **+1 flagged** | `role = flagged_retained; flag_reason = <enum>` |
| Aborted mid-round | **0** | `role = aborted; reason = <string>` |
| Transition-window round (e.g., R10, R11 for gpt-5.4) | **0** | `role = lti_anchor, not_recalibration_input` |

### §72.3.1 Re-run audit-trail caveat (ATHENA, §6 R2 C-3)

"**Re-run last clean**" semantics require an **explicit fix-reference** in the footnote (pointing to the commit / Plan ID / ENG-FAB item that repaired the issue). Without this constraint, a skilled adversary could silently re-run until a desired σ appeared — undermining ZT-3 same-distribution homogeneity. SCRIBE **G6.10** audits the footnote's presence on every `role = recalibration_input_rerun` row.

### §72.3.2 BABBAGE flag-not-drop invariant

Every round outcome is **recorded**. No round is silently dropped. Aborted/suspended rounds stay in the log with `N_increment = 0` + a reason string. This preserves audit-trail completeness and is consistent with MR-10 (造假/錯誤/未完成補回) + ZT-1 (no ten-tenet violation).

---

## §72.4 Two-stage verification — N = 10 provisional + N = 20 finalize (D-07(a))

### §72.4.1 Stage 1 — N = 10 provisional

At cumulative N = 10 same-family post-transition-window rounds:

1. PASCAL + WIENER co-signed pre-check:
   - **4a**: ZT-3 triple conjunction (K-S + Anderson-Darling + QQ) on the N = 10 sample.
   - **4b**: Anomaly-ratio computation per §72.3.
   - **4c**: Regime classification per §72.5 decision tree.
   - **4d**: G6.7 auto-veto gate (SCRIBE enforces; ZT-3 triple-check failure → auto-veto).
2. Algorithm selection:
   - **Scenario I** (LTI persists + triple-PASS) → **Westgard + FR-2 pooled-mode**.
   - **Scenario II** (regime switch detected) → **Rolling-window IQR** with bidirectional hysteresis (see §72.5.3).
3. Produce **provisional** recalibrated limits (UCL_prov, LCL_prov, mean_prov, V11_range_prov). **Provisional limits do NOT take effect** until Stage 2 consistency check + Master ratification (§72.4.3).
4. Record provisional status in `path_c_log.md`; test-team continues monitoring against **existing** Rule #61 limits during the N=10 → N=20 window.

### §72.4.2 Stage 2 — N = 20 finalize within 10% range tolerance

At cumulative N = 20 same-family post-transition-window rounds:

1. Re-run pre-check on the full N = 20 sample (triple-check + anomaly ratio + regime classification).
2. Compute final recalibrated limits (UCL_final, LCL_final, mean_final, V11_range_final).
3. **Consistency predicate** — finalize iff all three of:
   - `|mean_final − mean_prov| / mean_prov ≤ 10%`
   - `|UCL_final − UCL_prov| / UCL_prov ≤ 10%`
   - `|LCL_final − LCL_prov| / LCL_prov ≤ 10%`
4. If any tolerance exceeds 10% → **trigger ad-hoc R3 retrospective** (investigation of provisional-to-final drift; may indicate late anomaly or regime shift).
5. Submit **consolidated Master Ratification Request** (package covers both N=10 provisional and N=20 final evidence) — **not two separate requests**.

### §72.4.3 Power-analysis rationale (KNUTH — why two-stage)

K-S and Anderson-Darling statistical power at N = 10 is 0.30–0.50 for moderate shift (Cohen 1988; Pettitt 1977 small-sample tables). Two-stage verification raises detection power to 0.75+ at N = 20 while preserving the N = 10 early-warning property. The trade-off is ratification latency (approximately +10 W2 rounds ≈ +15 calendar days) against the risk of provisional adoption of a spurious recalibration.

### §72.4.4 Master ratification timing cadence

- **N = 8** — early-warning pre-review submission (research-initiated; informal heads-up to Master).
- **N = 10** — formal Master Ratification Request (Two-stage provisional per §72.4.1).
- **N = 20** — consolidated final submission (combines provisional + final per §72.4.2).

---

## §72.5 STEP 2 regime classification predicate (D-06(a) pooled-event form)

Extends the §6-1.3 decision tree. Determines the regime of round *k* given cumulative N samples.

### §72.5.1 STEP 1 — Deterministic LTI test

If `(max − min) / mean ≤ 10⁻⁴` (relative tolerance) over the N samples **AND** round *k* shares the same model family as all N samples → **deterministic LTI** regime. Retain current limits; increment N; no recalibration action.

### §72.5.2 STEP 2 — Stochastic drift test (D-06(a) pooled-event, BINDING)

If Westgard 1-3s fires on the rolling reference, compute the **pooled-event regime predicate**:

> **`|σ_current − rolling_ref| > 3 × (pooled_event_std / √n_per_round)`**

where:
- `pooled_event_std` is computed per FR-2 pooled mode (see §72.6 for the divisor fix).
- `n_per_round` is the event count within one W2 round (current config: 80).
- The factor `3` corresponds to the 3σ Westgard threshold.

If the predicate **exceeds** → **stochastic_drift** regime detected; route per §72.4.1 Scenario II (IQR + bidirectional hysteresis).

*Alternative considered and rejected* (D-06(b) relative-threshold): `|σ − ref| > k × ref` with k = 0.10 scale-invariant. R3 adopted (a) pooled-event form for statistical grounding; (b) retained here as the alternative for audit-trail transparency per MRB-07 §7.3 drafting note.

### §72.5.3 STEP 3 — Transitional case

Inside UCL/LCL but outside the deterministic threshold of §72.5.1 → **transitional**; retain existing regime; increment N; no action.

### §72.5.4 STEP 4 — Hard breach

`σ_current > UCL × Rule_70_ceiling_multiplier` OR `σ_current < LCL × Rule_70_floor_multiplier` → **stochastic_drift** + invoke Rule #70 transition tolerance. (Multipliers referenced symbolically to avoid Rule #70 drift.)

### §72.5.5 Regime hysteresis (prevents single-round flipping)

- A **regime-switch decision** requires **2 consecutive rounds** in the new regime.
- **IQR → Westgard switch-back** additionally requires ≥ 5 consecutive Westgard-compatible rounds + K-S/AD PASS in latest 5 + no new 2-2s Westgard fire for ≥ 3 rounds + cooldown ≥ 3 rounds since IQR switch. (Per §6 R2 F-§6-R2-7.)

---

## §72.6 Westgard pooled-std divisor — `n_total − N` fix (D-27(b))

Per D-27(b) BINDING (22/22 ANOVA-consistent), the **FR-2 pooled-mode `pooled_event_std`** is computed with divisor:

> **`pooled_event_std² = Σ_{i=1..N} Σ_{j=1..n_per_round} (x_{ij} − x̄_i)² / (n_total − N)`**

where:
- `N` = cumulative same-family post-transition-window rounds.
- `n_per_round` = event count per round (current: 80).
- `n_total = N × n_per_round`.
- `x̄_i` = within-round mean of round *i*.

The divisor `n_total − N` (not `n_total − 1`) is the **ANOVA within-subject degrees of freedom correction**: each round's within-group residual loses one df to its own mean estimate. Using `n_total − 1` under-estimates variance at small N; using `n_total − N` is the textbook ANOVA unbiased estimator (Scheffé 1959 §2.4; Kutner et al. 2005 §16.4).

### §72.6.1 Numerical example at N = 1 (W2-R12 boundary case)

- N = 1, n_per_round = 80, n_total = 80.
- `pooled_event_std²` denominator = `80 − 1 = 79`.
- This reduces to the familiar single-round sample variance with n−1 correction. No paradox at N = 1.

At N = 2: denominator = `160 − 2 = 158`. Etc.

---

## §72.7 Deployment fingerprint rule (D-08(a))

### §72.7.1 Fields recorded per round

`path_c_log.md` records, on every row (see `Calibration_Reports/18_PathC_20Field_Monitoring_Format.md` fields 16 and 17):

- **Field 16** — `model_version_hash`: model family + build identifier (e.g., `gpt-5.4-build-20260415`).
- **Field 17** — `code_deployment_fingerprint`: git-rev or build hash of the openstarry code under test (e.g., `v0.47.0-alpha @ git:abc123`).

### §72.7.2 N reset semantics

| Change type | N behavior | Additional action |
|-------------|:----------:|-------------------|
| **Model-only change** (e.g., gpt-5.4 → gpt-5.5) | **RESET N to 1** | Old-family `path_c_log.md` sealed with `role = closed_family_log`; new transition window begins; Rule #71 rolling reference reboots. |
| **Code-only change** (e.g., Plan48, Plan49 delivery) | **DO NOT RESET N** | Row annotated with `code_deployment_fingerprint`; plan ID in `notes`. Disclosure in Master Ratification Request package per §72.4.4. |
| **WIENER evaluator-touch** (code change touches σ computation, FR-2 pooled logic, or Rule #71/#72 machinery) | Flag `evaluator_touched = true` | SCRIBE **G6.10** review mandatory; research-team attests test-surface-only non-impact. Failure to attest clean → N reset mandatory. |

### §72.7.3 SCRIBE audit enforcement

SCRIBE **G6.10** enforces presence of fields 16 + 17 on every `path_c_log.md` row. Missing fingerprint → **G6.10 FAIL**, round disputed until repaired.

---

## §72.8 Cross-reference to Rule #76

**§72.8 Cross-reference to Rule #76** — P(coincidence) binding convention and reproducible-calculation mandate are defined in Rule #76 (`Reference/06_P_Coincidence_Binding_Convention.md`). See §76.1–§76.6 for authoritative form, acceptable phrasings, forbidden forms, Protocol D disclosure, SCRIBE G5 enforcement, and reproducible calculation template. The W2-R10 = R11 = R12 6dp identical σ = 0.023753 triple-match is the motivating observation for this sub-clause's N-starting-point (§72.2) and two-stage cadence (§72.4).

(Rule #76 RATIFIED 2026-04-23 Path A standalone per `deliver/Master_Ratification/Batch_3A_P_Coincidence_Binding.md §12`.)

---

## §72.9 Status and ratification path

- **Status**: CANDIDATE — awaiting Master ratification via **Batch 8c** (bundled with Rule #75 Pre-Delivery Gate [see `Research_Methodology/09_Rule_75_PreDelivery_Gate_Draft.md`] + ENG-FAB v1.7 [see `Research_Methodology/10_ENG_FAB_v1.7_Candidate.md`]).
- **Interim guidance**: Until ratification, existing Rule #72 Step 2 wrapper is binding; §72.1–§72.7 govern research-team internal planning and the Path-C monitoring framework (`Calibration_Reports/18_PathC_20Field_Monitoring_Format.md` is already BINDING per R3 CV-35/36/39 + D-06/07/08).
- **Effective date proposal**: on Master ratification, §72.1–§72.7 replace existing Step 2 wrapper immediately for W2-R13 and beyond.

## §72.10 Dissent and audit-trail

Per MR-11 (每輪討論必須以十大宣言為基調) and SCRIBE G5-blindness lesson from 03-11 (not all dissent is visible unless explicitly captured):

- **D-06(a) pooled-event form** — 22/24 majority; 2 dissent votes advocating (b) relative-threshold k = 0.10 form for scale invariance. Dissent preserved in R3_decision_log §7; also noted in §72.5.2 above.
- **D-07(a) two-stage** — 22/23 majority; 1 dissent (TURING: single-stage N=10 minimises round latency). Dissent preserved.
- **D-08(a) model-reset + code-flag** — UNANIMOUS 23/23.
- **D-22(a) Batch 2-C merge** — 22/23; 1 dissent (KERNEL: separate batch for traceability).
- **D-27(b) `n_total − N`** — UNANIMOUS 22/22.
- **CV-39 N = R12 = N=1** — UNANIMOUS STRONG ENDORSE (BABBAGE + KNUTH + ATHENA co-signed).

## §72.11 Author attribution

- **Primary**: WIENER (#12 Control Theory), PASCAL (#19 Decision Theory), SCRIBE (#2 Record) — MRB-07 joint drafters.
- **Statistical methodology sign-off**: KNUTH (#21) + BABBAGE (#9).
- **Operational feasibility sign-off**: ATHENA (#5).
- **SUNYATA coordination**: R3 day-4 agenda item D-06/07/08/22/27 + CV-39 closure.
- **Consolidation for R4**: SYNTHESIST (#1).
