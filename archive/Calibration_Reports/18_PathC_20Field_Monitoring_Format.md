# Path-C Per-Round 20-Field Monitoring Record Format (BINDING)

> **Front-matter**
> - **Cycle**: 03-12
> - **Date**: 2026-04-25
> - **Authors**: BABBAGE (audit-completeness lens) + ATHENA (operational lens) + KNUTH (statistical lens) + SCRIBE (G6.8/G6.9/G6.10 authority) + SYNTHESIST (consolidation)
> - **Binding R3 decisions**: **CV-35** AGREE 15 → 20 field upgrade · **CV-36** AGREE N increment rules · **CV-39** UNANIMOUS STRONG ENDORSE N = R12 = N = 1 starting point · **D-06(a)** pooled_std predicate · **D-07(a)** two-stage N = 10 + N = 20 · **D-08(a)** model-reset + code-flag deployment fingerprint · **D-18(a)** SCRIBE-authority G6.8/G6.9/G6.10
> - **Status**: **BINDING** — adopted as the canonical per-round W2 monitoring record format from cycle 03-12 R4 forward; attached to Master Ratification Request Batch 7 (S-3 downgrade) and Batch 8c (Rule #72 N≥10 sub-clause) as a binding annex.
> - **File location**: per cycle, `research record/<cycle>/discussions/path_c_log.md`; one row per W2 round; ENG-FAB F-10 Pre-Delivery-Gate audit includes population-check on this file.
> - **Cross-refs**: `Research_Methodology/08_Rule_72_N10_SubClause_Draft.md` · `deliver/O6_PathC_monitoring_framework.md`

---

## §1 Purpose

Upgrade the R1 15-field Path-C monitoring record (6 mandatory Master-letter fields + 9 supplementary) to a **20-field layout** (9 mandatory + 11 supplementary) per F-§6-R2-1. The upgrade reflects:

- **BABBAGE audit-completeness** — deployment-fingerprint (model hash + code fingerprint) mandatory for ZT-3 distributional homogeneity audit and for D-08(a) model-reset + code-flag.
- **ATHENA operational** — field-source mapping table in `todo_test_instructions.md` to reduce per-round filling time from ≈ 40 min to ≈ 10 min via structured source pointers.
- **KNUTH statistical** — `pooled_std`, `event_count_n_per_round`, and `per_category_sigmas` required for FR-2 pooled-mode SPC recalibration + Simpson's-paradox detection (03-10 lesson).
- **§6 R2 cross-chapter consistency** — fields 16/17 (fingerprints) required for Rule #72 deployment-fingerprint rule cross-chapter consistency with §6 PathC framework.

---

## §2 Mandatory Fields (9)

Per F-§6-R2-1 + §6-1.2 R2 upgrade. Every W2 round MUST populate these 9 fields.

| # | Field | Type | Source (typical `test_report_w2rN.md` section) | Rationale |
|:-:|-------|------|------------------------------------------------|-----------|
| **1** | `round_id` | string `W2-RNN` | `test_report §0` | Master letter §六 |
| **2** | `sigma_6dp` | decimal 6dp | `test_report §1.1` (overall event-level σ) | Master letter §六; Rule #72 Step 1 |
| **3** | `rolling_reference` | decimal 6dp | `test_report §1.2` (mean of last 2 same-family) | Rule #71 FR-1 |
| **4** | `v11` | decimal 4dp | `test_report §1.3` (= σ / rolling_ref) | Master letter §六; Rule #61; see `Reference/07_V11_Wording_Binding.md` for interpretation binding |
| **5** | `regime` | enum `{deterministic_LTI, stochastic_drift, transitional}` | derived per §6-1.3 decision tree + Rule #72 §72.5 | Rule #72 §72.5 |
| **6** | `N_cumulative` | integer ≥ 1 | derived per §3 increment rule | Rule #72 §72.2 starting point + §72.3 increment |
| **7** | `spc_ucl_lcl` | `(decimal 6dp, decimal 6dp)` | `test_report §1.4` (current provisional limits) | R2 upgrade per KNUTH (§6-1.2 field 7) |
| **8** | `spc_status` | enum `{in_control, warning, out_of_control}` | derived vs UCL/LCL | R2 upgrade per ATHENA (§6-1.2 field 9); G6.7 gate input |
| **9** | `shadow_agreement_pct` | decimal % | `test_report §2` (e.g., 47/47 or equivalent) | R2 upgrade per §2 consistency (§6-1.2 field 12) |

### §2.1 Mandatory-field invariants

- **Field 2 `sigma_6dp`** MUST be rounded to exactly 6 decimal places; underlying float retained for FR-2 pooled computation but the record holds 6dp.
- **Field 3 `rolling_reference`** computed per Rule #71 FR-1; when the round lies in the transition window (e.g., W2-R10, W2-R11 for gpt-5.4), `rolling_reference` is computed from mixed-family predecessors and `role` in §3 below = `lti_anchor`.
- **Field 4 `v11`** interpretation is bound per `Reference/07_V11_Wording_Binding.md` — when `v11 = 1.0000x`, use canonical phrasing "numerical artifact / tautological 1.0000x"; do NOT use "dead-center" / "converged" / "perfect stability" in any surrounding narrative.
- **Field 5 `regime`** is assigned via the §6-1.3 decision tree + Rule #72 §72.5 predicate; hysteresis rule per §72.5.5 applies (regime switch requires 2 consecutive rounds).
- **Field 6 `N_cumulative`** starts at N = 1 at W2-R12 per CV-39 UNANIMOUS; transition-window rounds record `N_cumulative = 0` and `role = lti_anchor`.

---

## §3 Supplementary Fields (11)

Per F-§6-R2-1 + §6-1.2 R2 upgrade. These are populated where applicable; some depend on FR-2 activation or canary scenarios.

| # | Field | Type | Populated when | Purpose |
|:-:|-------|------|----------------|---------|
| **10** | `spc_delta_pct` | signed % | every round | trend monitoring (§6-1.2 field 8); SPC drift detection |
| **11** | `anomaly_count_4tuple` | `(L2_count, L3_count, spc_breach_count, rerun_count)` | every round | flag-not-drop audit (§2.3 Rule #72); §72.3 increment input |
| **12** | `canary_status` | enum `{pass, fail, n/a}` | rounds with canary | Plan47/48 positive-control audit |
| **13** | `pooled_mode_active` | bool | every round (true from R13+ once FR-2 active) | FR-2 audit |
| **14** | `pooled_std` | decimal 6dp | when `pooled_mode_active = true` | regime predicate input per Rule #72 §72.5.2; D-27(b) divisor `n_total − N` |
| **15** | `event_count_n_per_round` | integer (80 under current config) | every round | FR-2 SE denominator (§6-2.3) |
| **16** | `model_version_hash` | string (model family + build) | every round | ZT-3 distribution-homogeneity audit (F-§6-R2-1); Rule #69; D-08(a) |
| **17** | `code_deployment_fingerprint` | string (git-rev or build hash) | every round | D-08(a) code-flag audit; Rule #72 §72.7 deployment fingerprint |
| **18** | `per_category_sigmas` | map `{category → σ_6dp}` | when `pooled_mode_active = true` | Simpson's-paradox detection (03-10 lesson) |
| **19** | `timestamp_utc` | ISO-8601 | every round | rate-of-change analysis; audit (F-§6-R2-1) |
| **20** | `test_team_sign_off` | `{signer, timestamp, test_instructions_ref}` | every round | Rule #74 L1' compliance audit trail; F-§6-R2-1 |

### §3.1 Supplementary-field invariants

- **Fields 16 + 17** are **jointly mandatory** for ZT-3 homogeneity + D-08(a) model-reset-vs-code-flag: missing either → SCRIBE G6.10 FAIL (see §7).
- **Field 14 `pooled_std`** uses the D-27(b) divisor `n_total − N` (NOT `n_total − 1`); see `Research_Methodology/08_Rule_72_N10_SubClause_Draft.md` §72.6 for the formula.
- **Field 18 `per_category_sigmas`** non-empty when `pooled_mode_active = true` is a hard requirement — Simpson's-paradox detection requires per-category visibility.
- **Field 12 `canary_status`** = `n/a` outside canary scenarios; `pass` / `fail` only when a canary (Plan47 Canary 1 deny / Canary 2 allow / Canary 3 default-permissive) is actually executed in the round.

---

## §4 N Increment Rules (per §3 of this doc, CV-36 AGREE + ATHENA audit-trail caveat)

| Round outcome | N increment | `path_c_log.md` annotation |
|---------------|:-----------:|----------------------------|
| GO + `in_control` + no anomaly | **+1** | `role = recalibration_input` |
| NO-GO attributable to dev-layer fix (Rules #64–66) | **0 → re-run scheduled** | `role = suspended; rerun_pending` |
| Re-run after fix | **last-clean + 1** with audit footnote | `role = recalibration_input_rerun; footnote = <fix_reference>` |
| Anomaly (L2/L3 > 0 OR SPC breach OR infra fault) | **+1 flagged** | `role = flagged_retained; flag_reason = <enum>` |
| Aborted mid-round | **0** | `role = aborted; reason = <string>` |
| Transition-window round (R10, R11 for gpt-5.4) | **0** | `role = lti_anchor, not_recalibration_input` |

### §4.1 Re-run audit-trail caveat (ATHENA)

"**Re-run last clean**" semantics require an **explicit fix-reference** in the footnote (pointing to the commit / Plan ID / ENG-FAB item that repaired the issue). Without this, a skilled adversary could silently re-run until a desired σ appeared — undermining ZT-3 same-distribution homogeneity. SCRIBE **G6.10** enforces footnote presence (see §7).

### §4.2 BABBAGE flag-not-drop invariant

Every round outcome is **recorded**. No round is silently dropped. Aborted/suspended rounds stay in the log with `N_increment = 0` + reason string. Consistent with MR-10 + ZT-1.

---

## §5 N = R12 = N = 1 Starting Point (CV-39 UNANIMOUS)

### §5.1 Binding starting point

**W2-R12 = N=1.** R10 and R11 are retained in `path_c_log.md` with `role = lti_anchor, not_recalibration_input`. Their role is **exterior LTI validation**, not recalibration input.

### §5.2 Fixed-point argument (BABBAGE + KNUTH + ATHENA UNANIMOUS per §6 R2 §4.3)

1. **KNUTH rigor** — ZT-3 "same distribution" requires samples drawn from the regime the recalibration will govern. R12's rolling reference = mean(R10 gpt-5.4, R11 gpt-5.4) = pure gpt-5.4 reference; only R12 onward has distributional homogeneity.
2. **BABBAGE formal** — Fixed-point property: inputs to recalibration must be drawn under the regime the recalibration governs. R10/R11 were evaluated under the *transition* regime; using them as recalibration inputs violates the fixed-point.
3. **ATHENA operational** — R10/R11 not wasted: the three-round 6dp σ identical match (R10 = R11 = R12 = 0.023753 to 6 decimals, P(coincidence) ≤ 10⁻⁴⁸ per `Reference/06`) is itself the strongest LTI evidence. Role = exterior validation.
4. **Master-letter-literal alignment** — Master letter §六 line 2.2: "計算起點 = R12 (第一個 post-gpt-5.4 W2 round)". Path A (R12 = N=1) aligns with literal text.

### §5.3 Zero-data-loss claim

| Round | Raw σ used | Recalibration baseline | LTI-anchor validation |
|-------|:----------:|:----------------------:|:---------------------:|
| R10 | YES (consumed in rolling ref for R12) | **NO** (transition window) | **YES** (first post-change sample) |
| R11 | YES (in rolling ref for R12/R13) | **NO** (transition window) | **YES** (2nd post-change sample) |
| R12+ | YES | **YES** (path-C counter) | YES |

Both independent roles (rolling-reference seed; LTI validation) preserved. Zero information discarded.

---

## §6 Two-Stage N = 10 Provisional + N = 20 Finalize Integration

### §6.1 Stage 1 — N = 10 provisional

At cumulative N = 10 (same-family, post-transition-window) rounds:
1. PASCAL + WIENER co-signed pre-check (ZT-3 triple + anomaly ratio + regime classification + G6.7 auto-veto).
2. Produce **provisional** recalibrated limits — recorded in a dedicated `path_c_log.md` annotation row with `role = provisional_recalibration_record`.
3. Master Ratification Request per cadence (§72.4.4).
4. During N = 10 → N = 20 window, test-team continues monitoring against **existing** Rule #61 limits.

### §6.2 Stage 2 — N = 20 finalize

At cumulative N = 20 rounds:
1. Re-run pre-check on full N = 20 sample.
2. Compute final recalibrated limits.
3. **Consistency predicate** — finalize iff provisional and final mean/UCL/LCL differ by ≤ 10%.
4. If > 10% → trigger ad-hoc R3 retrospective.
5. Submit **consolidated** Master Ratification Request (N = 10 + N = 20 combined).

### §6.3 Rule #72 §72.4 cross-reference

Full two-stage verification machinery defined in `Research_Methodology/08_Rule_72_N10_SubClause_Draft.md` §72.4 (D-07(a) BINDING). This section provides the path_c_log record structure.

---

## §7 SCRIBE G6.8 / G6.9 / G6.10 Authority (D-18(a) BINDING)

Per R3 D-18(a) 23/0 UNANIMOUS (non-chair), SCRIBE is granted authority for three parallel process gates. These are **SCRIBE-authority** (not Master route); SCRIBE maintains the logs and can declare FAIL independently.

### §7.1 G6.8 — Pre-Delivery Gate Alignment

- **Scope**: cross-artefact alignment of Pre-Delivery Gate attestation per Rule #75 across cycle-end artefacts (delivery report, engineering delivery, CHANGELOG, test_instructions).
- **Trigger**: every Plan delivery.
- **Check**: attestation sections consistent across all cycle-end documents.
- **Failure**: G6.8 FAIL → dev + research joint remediation before R4 deliver.

### §7.2 G6.9 — 25-Sub-Item Verification Completeness

- **Scope**: verify all 25 atomic acceptance criteria for Plan48 (per `Calibration_Reports/19_Plan48_25_SubItems_Binding.md`) are accounted for.
- **Trigger**: Plan48 delivery (and analogously for subsequent Plans with binding acceptance tables).
- **Check**: each of the 25 criteria has a PASS/FAIL attestation + evidence reference in the delivery report.
- **Failure**: G6.9 FAIL → missing criteria identified; dev remediates before Rule #75 transfer.

### §7.3 G6.10 — `path_c_log.md` Population Audit

- **Scope**: verify `path_c_log.md` correctness per this document.
- **Trigger**: every W2 round (per round commit to the log).
- **Check items**:
  - Presence of all 9 mandatory fields for the round.
  - Presence of fields 16 (model hash) + 17 (code fingerprint) — **missing either → G6.10 FAIL**.
  - Non-empty `per_category_sigmas` when `pooled_mode_active = true`.
  - Re-run footnote presence on `role = recalibration_input_rerun` rows.
  - `evaluator_touched = true` → research-team attests test-surface-only; failure → mandatory N reset per Rule #72 §72.7.2.
- **Failure**: G6.10 FAIL → round disputed; test-team remediates before next round proceeds.

---

## §8 Deployment Fingerprint Rule (D-08(a))

Per Rule #72 §72.7 + D-08(a) BINDING, `path_c_log.md` records fields 16 + 17 on every row. N-reset semantics:

| Change type | N behavior | Action |
|-------------|:----------:|--------|
| **Model-only change** (e.g., gpt-5.4 → gpt-5.5) | **RESET N to 1** | Old-family log sealed with `role = closed_family_log`; new transition window begins |
| **Code-only change** (Plan48, Plan49 delivery) | **DO NOT RESET N** | Row annotated with `code_deployment_fingerprint`; plan ID in notes |
| **WIENER evaluator-touch** (code change touches σ computation, FR-2 pooled logic, or Rule #71/#72 machinery) | Flag `evaluator_touched = true` | SCRIBE G6.10 review mandatory; research-team attests test-surface-only; failure → N reset |

---

## §9 Operational Burden + Field-Source Mapping

Per ATHENA operational flag (§6 R2 §3.3), manual filling of 20 fields ≈ 40 min/round without structured aid. **Binding mitigation** per F-§6-R2-3: `todo_test_instructions.md` for W2-R13+ MUST include a **per-field source-pointer table** mapping each of the 20 fields to its line range in `test_report_w2rN.md`. Reduces fill-time to ≈ 10 min via structured source pointers.

The reference template is provided in the R4 `todo_test_instructions.md` §W2-R13-PathC-fields section (outside the scope of this document but cross-referenced here for completeness).

---

## §10 Example Row (W2-R12, illustrative)

```
round_id: W2-R12
sigma_6dp: 0.023753
rolling_reference: 0.023753
v11: 1.0000                 # NOTE: "numerical artifact / tautological 1.0000x" per Reference/07
regime: deterministic_LTI
N_cumulative: 1             # N = R12 = N=1 per CV-39
spc_ucl_lcl: (0.024200, 0.019540)
spc_status: in_control
shadow_agreement_pct: 100.0
spc_delta_pct: -1.8
anomaly_count_4tuple: (0, 0, 0, 0)
canary_status: pass         # Canary 1 deny 5/5 + 2 allow 3/3 + 3 default-permissive 4/4
pooled_mode_active: false   # FR-2 pooled activates from R13+
pooled_std: n/a
event_count_n_per_round: 80
model_version_hash: gpt-5.4-build-20260420
code_deployment_fingerprint: v0.47.0-alpha@git:abc123
per_category_sigmas: n/a    # pooled_mode_active = false
timestamp_utc: 2026-04-20T14:23:56Z
test_team_sign_off: {signer: "test-team-lead", timestamp: "2026-04-20T14:35:10Z", test_instructions_ref: "todo_test_instructions.md#W2-R12"}
role: recalibration_input
N_increment: +1
```

---

## §11 Status

**BINDING** per R3 CV-35 / CV-36 / CV-39 reaffirmed + D-06(a) / D-07(a) / D-08(a) / D-18(a).

- **Adoption**: cycle 03-12 R4 forward; W2-R13 onward uses this 20-field format.
- **Grandfather for W2-R12**: W2-R12 retroactively populated into the 20-field layout (test-team + research-team back-fill from existing test_report_w2r12.md); populated entry retained as reference row.
- **Annex status**: attached to Master Ratification Request Batch 7 (S-3 downgrade) and Batch 8c (Rule #72 N≥10 sub-clause) as binding annex.

## §12 Author attribution

- **Audit lens**: BABBAGE (#9).
- **Operational lens**: ATHENA (#5).
- **Statistical lens**: KNUTH (#21).
- **SCRIBE authority**: SCRIBE (#2) for G6.8/G6.9/G6.10.
- **Consolidation**: SYNTHESIST (#1).
- **Coordination**: SUNYATA (#0).
- **R3 votes** (ground truth `R3_decision_log.md §2.1/§2.2`): CV-35 AGREE + CV-36 AGREE + CV-39 UNANIMOUS STRONG ENDORSE + D-06(a) 22/1 (1 MESH relative-threshold dissent) + D-07(a) 22/1 (1 TURING single-stage dissent) + D-08(a) 21 (a) / 1 (b) / 1 (c) — 2 non-chair dissents (safety + holistic alternatives); WIENER evaluator-touch refinement adopted into (a) + D-18(a) 23/0 UNANIMOUS SCRIBE-authority G6.8/G6.9/G6.10.
