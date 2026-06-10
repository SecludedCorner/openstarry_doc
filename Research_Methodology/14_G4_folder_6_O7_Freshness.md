---
title: G4-folder-6 — O7 Compliance Evidence Freshness Check
author: SCRIBE (#2 records lead) + PENROSE (#18 observer-effect) + ARCHIMEDES (#16 engineering) + SYNTHESIST (#1 aggregator)
date: 2026-04-25
cycle: 03-14
status: CANDIDATE (pending Master Ratification Batch 11 #3 courtesy notice)
authority: research-team SCRIBE-internal (self-binding); Master courtesy notice
supersedes: null
cross_refs:
  - research record/cycle03-14/deliver/O4_R_input_5_candidates_final.md §4 (lines 388-485)
  - research record/cycle03-14/R3/R3_decision_log.md §4 (D-§4-07 + D-§4-08 + MRB-§4-C)
  - research record/cycle03-14/deliver/Master_Ratification/Batch_11_Request.md §3.3 (Item #3)
  - research agent corps/shared/folder_convention_memo.md (G4-folder-1..5 baseline)
  - research record/cycle03-13/openstarry_doc/Research_Methodology/12_GN_1_5_SCRIBE_G_Gate.md (precedent SCRIBE-internal authority)
binding_until: cycle 03-15 R4 close (post-calibration data)
---

# G4-folder-6 — O7 Compliance Evidence Freshness Check

## 1. Status

**CANDIDATE — reflexive cycle 03-14 dry-run-non-binding** at this R4 close. Upgrades to **BINDING** at cycle 03-15 R4 close pending coordinator post-R4 calibration data emission (per D-§4-07 + D-§4-08 R3 ratifications and Master Ratification Batch 11 Item #3 courtesy-notice route).

**Authority**: SCRIBE-internal — research-team self-binding parallel to cycle 03-13 G6.8 / G6.9 / G6.10 SCRIBE authority precedent (per `Research_Methodology/12_GN_1_5_SCRIBE_G_Gate.md`). Master is informed; ratification is research-team self-binding. Master may declare otherwise via Batch 11 #3 disposition.

## 2. Background and Motivation

Cycle 03-13 R4 O7 compliance report stated `δ PARTIAL` due to audit-timing staleness — Plan49 delivery (post-O7-input cutoff) and W2-R14 PASS (2026-04-24, post-cutoff) had landed but had not been re-ingested into the O7 evidence chain. The δ status was corrected via post-close addendum (`O7_compliance_addendum_delta_closure.md`, 2026-04-25), with Master confirming 9/0/1★ ACTIVE activation.

This timing lag is structurally avoidable. G4-folder-6 operationalises the principle "audit at the right moment" (MR-7) by making "right moment" verifiable through explicit freshness thresholds on per-source evidence citation timestamps.

## 3. R3 Provenance

- **D-§4-07 (A-6)** 20/3 — 24h soft / 48h hard two-stage data-anchored threshold adopted; DSS-16 (SYNTHESIST minority preference for fixed 24h) preserved verbatim §10.
- **D-§4-08 (B-7)** UNANIMOUS 23/0 — reflexive cycle 03-14 dry-run-non-binding; binding cycle 03-15+; PENROSE observer-effect concern absorbed via the dry-run discipline.
- **MRB-§4-C** RESOLVED — calibration data deferred to coordinator post-R4 task; R3 not blocked.

## 4. Binding Text

### 4.1 Gate definition (G4-folder-6)

A new SCRIBE-internal G4-folder-completeness check, numbered **G4-folder-6**, is added to the existing G4-folder-1..5 series in `research agent corps/shared/folder_convention_memo.md`:

> **G4-folder-6 — O7 Compliance Evidence Freshness**
>
> `O7_compliance.md` MUST cite, by relative path and last-modified timestamp, each of the following evidence sources:
>
> 1. **Dev `delivery_report.md`** — most recent for the cycle in scope
> 2. **Test `test_report.md`** — most recent W-round report relevant to the compliance claim
> 3. **ENG-FAB checklist** — most recent applied checklist (v1.8, v1.9, etc.)
> 4. **Master Ratification log** — most recent Batch dispatch + Master response
>
> Each cited source carries a freshness assessment relative to R4 dispatch wall-clock time. Two-stage threshold:
>
> - **Soft (24h)**: source modified > 24h before R4 dispatch wall-clock → SCRIBE emits a self-reminder flag in O7 §Evidence Provenance section. Audit continues.
> - **Hard (48h)**: source modified > 48h before R4 dispatch wall-clock → coordinator confirmation step REQUIRED. O7 must record either:
>   - one-line coordinator sign-off ("coordinator confirms <source> is current as of <date>"), OR
>   - explicit `re-audit-pending` flag with scheduled re-audit timestamp
>
> Failure to cite any of the 4 evidence sources, or to clear the 48h hard-threshold protocol, blocks G4-folder gate close.

### 4.2 Per-source freshness audit format

O7 §Evidence Provenance section must include the following table:

```yaml
evidence_provenance:
  - source: "Dev delivery_report"
    path: "openstarry_eco/release/<version>/delivery_report.md"
    last_modified: "<ISO-8601>"
    age_hours: <integer>
    threshold_status: <"fresh" | "soft_flag" | "hard_block_pending_coord">
  - source: "Test test_report"
    path: "test research/<cycle>/deliverables/test_report_<round>.md"
    last_modified: "<ISO-8601>"
    age_hours: <integer>
    threshold_status: <as above>
  - source: "ENG-FAB checklist"
    path: "research record/<cycle>/openstarry_doc/Research_Methodology/<NN>_ENG_FAB_v<X>_Binding.md"
    last_modified: "<ISO-8601>"
    age_hours: <integer>
    threshold_status: <as above>
  - source: "Master Ratification log"
    path: "research record/<cycle>/deliver/Master_Ratification/Batch_<NN>_Request.md"
    last_modified: "<ISO-8601>"
    age_hours: <integer>
    threshold_status: <as above>
```

Where `age_hours = floor((R4_dispatch_wall_clock - last_modified) / 1h)` and threshold_status is computed mechanically:

- `age_hours ≤ 24` → `"fresh"` (no action)
- `24 < age_hours ≤ 48` → `"soft_flag"` (SCRIBE self-reminder)
- `age_hours > 48` → `"hard_block_pending_coord"` (coordinator confirmation REQUIRED before G4 close)

### 4.3 Reflexive dry-run discipline (cycle 03-14)

Cycle 03-14 R4 close itself runs G4-folder-6 in **dry-run-non-binding** mode. The cycle 03-14 O7 report includes the freshness table per §4.2 format, but the gate does NOT block on threshold_status outcomes. SCRIBE records all observations (any soft_flag / hard_block_pending_coord findings) into `discussions/cycle03-14/SCRIBE_g4_folder_6_dryrun_observations.md`. These observations feed the calibration data table coordinator emits post-R4.

The dry-run discipline absorbs the PENROSE observer-effect concern (D-§4-08 R1 finding): a freshness check that itself triggers re-audits could feedback-loop into infinite re-audit. The dry-run mode breaks the loop for the calibration cycle.

### 4.4 Binding from cycle 03-15+

Effective at cycle 03-15 R4 close:

- G4-folder-6 binding; failure to clear 48h hard-block without coordinator confirmation blocks G4 close.
- 24h soft-flag emission is informational; does NOT block.
- Coordinator post-R4 calibration table (output of cycle 03-14 dry-run) informs any threshold revision proposed at cycle 03-15+ R0 if substantive false-positive rate is observed.

### 4.5 Threshold revisability

The 24/48h two-stage thresholds are **data-anchored, not essentialist**. If cycle 03-15 R0 calibration data shows:

- Soft-flag rate > 30% across all R4 closes → consider relaxing soft to 36h
- Hard-block triggered with no actual staleness → consider relaxing hard to 72h
- Multiple stale-evidence escapes despite passes → consider tightening soft to 18h

Any threshold revision goes through R0-R4 amendment cycle as a v2 proposal (NOT this CANDIDATE's revision). The current 24/48h is the data-anchored opening calibration.

## 5. Per-Source Freshness Examples

### 5.1 Cycle 03-13 retrospective (illustrative; not binding for cycle 03-13 itself)

Applying G4-folder-6 to cycle 03-13 R4 dispatch (2026-04-24) on a hypothetical retroactive basis:

| Source | Last modified | age_hours | threshold_status |
|--------|---------------|-----------|------------------|
| Plan48 delivery_report.md | 2026-04-23 | ~24h | `fresh` |
| W2-R13 test_report.md | 2026-04-23 | ~24h | `fresh` |
| ENG-FAB v1.7 checklist | 2026-04-22 | ~48h | `soft_flag` |
| Plan49 delivery_report.md | 2026-04-26 (future-relative-to-O7-cutoff) | N/A — late-arrival | retroactive `hard_block_pending_coord` would have triggered |

Conclusion: had G4-folder-6 been operational at cycle 03-13, the late-arriving Plan49 delivery_report would have triggered `hard_block_pending_coord`, and the O7 δ status would have explicitly carried a coordinator confirmation flag — preempting the post-close addendum.

### 5.2 Cycle 03-14 reflexive dry-run (this cycle)

To be filled in by SCRIBE post-R4 dispatch into `discussions/cycle03-14/SCRIBE_g4_folder_6_dryrun_observations.md`. The dry-run table will be appended to coordinator G5 sync calibration data table.

## 6. Effective Date

- **Cycle 03-14 R4 close** (this dispatch): dry-run-non-binding mode active.
- **Cycle 03-15 R4 close**: binding mode active (failure blocks G4 close).
- **Threshold revision (if needed)**: per cycle 03-15 R0 calibration data review.

## 7. Strength

**MUST (SCRIBE-internal authority — research-team self-binding)**.

## 8. Compliance Attestation

| Constraint | Status | Evidence |
|------------|:------:|----------|
| MR-7 (audit at right moment) | ✅ PASS | Operationalises "right moment" via explicit freshness thresholds |
| MR-5 hard (Tenet #10 status unchanged) | ✅ PASS | Audit-process gate; no compliance status implication |
| MR-6 (Core 零) | ✅ PASS | Audit layer; no Core surface |
| MR-9 (no MUST WAIVE) | ✅ PASS | MUST forward at cycle 03-15+; dry-run is intentional calibration discipline |
| MR-10 (back-fill) | ✅ PASS | Threshold revisable per calibration data |
| MR-11 (dissent preservation) | ✅ PASS | DSS-16 verbatim §10 |
| MR-12 (既有 not retrofit) | ✅ PASS | Cycle 03-13 O7 not retrofitted; addendum approach preserved |
| ZT-1 / ZT-2 / ZT-3 | ✅ PASS | No 10-Tenet violation; endpoint unchanged; control-range unaffected |

## 9. Authority Notes

- **Authority route**: SCRIBE-internal (research-team self-binding); Master courtesy notice via Batch 11 #3.
- **Master may declare otherwise**: if Master prefers Master-route ratification with binding effect at cycle 03-14 R4 close (forgoing the dry-run discipline), Master decision overrides; otherwise cycle 03-15+ binding stands.
- **SCRIBE responsibility**: maintains `folder_convention_memo.md` with G4-folder-1..6 series; emits dry-run observations log; coordinates with coordinator post-R4 on calibration data table.

## 10. Dissent Slot — DSS-16 Verbatim (per MR-11)

**DSS-16 (D-§4-07, 1 minority vote — SYNTHESIST)**:

> SYNTHESIST minority preference for fixed 24h threshold:
>
> "An aggregation lens favours operational simplicity. The 24/48h two-stage adds a branching step (`fresh` → `soft_flag` → `hard_block`) where a single 24h cliff edge would suffice: above 24h, coordinator confirmation always required; below, never. The aggregation principle suggests the simpler rule generates fewer false-positives over time once SCRIBE and coordinator habituate to the 24h discipline. The two-stage discipline buys flexibility at the cost of decision-path complexity, and the buying may not be net-positive over enough cycles."
>
> R3 majority (20/3) preferred two-stage on the grounds that:
> - Cycle 03-13 retrospective shows real-world evidence in the 24-48h band (ENG-FAB v1.7 checklist age) where soft-flag is the right outcome (not hard-block, not fresh).
> - The two-stage discipline produces graded data for future calibration; a single cliff produces only binary data.
> - PENROSE observer-effect concern (constant hard-blocks would feedback-loop) is mitigated by the soft-flag intermediate state.

The SYNTHESIST position is preserved; if cycle 03-15+ calibration data shows the soft-flag intermediate adds no value (e.g., > 95% of soft-flagged sources turn out to be effectively fresh), the threshold can collapse to fixed 24h via R0-R4 amendment.

---

*Cycle 03-14 R4 final — 2026-04-25*
*Authors: SCRIBE (#2) + PENROSE (#18) + ARCHIMEDES (#16) + SYNTHESIST (#1)*
*Status: CANDIDATE (dry-run-non-binding cycle 03-14; binding cycle 03-15+ pending calibration)*
*Authority: SCRIBE-internal (research-team self-binding); Master courtesy notice via Batch 11 #3*
