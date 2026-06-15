---
title: Reference/21 v0.2.1 Amendment — C-LEIBNIZ-N2-observation Counter
date: 2026-05-13
cycle: 03-30
authors: KNUTH + SUSSMAN + LEIBNIZ (proposal); cycle 03-30 R3 §1.2 + §5.4 ratify
status: CANDIDATE pending Master Ratification Batch 25 Item #7 (LEIBNIZ N=2 ceiling Option γ + counter)
authority: cycle 03-30 R3 §1.2 D-§A1.4-P2 Option γ hybrid (21/2 super-majority); cycle 03-25 R3 D-§13 DSS-CY25-§13-A LEIBNIZ N=2 stricter sustained per MR-11
supersedes: none (additive amendment to Reference/21 v0.2)
cross_refs:
  - Reference/21_Counter_Registry.md (v0.2 baseline)
  - Reference/20_Audit_Verdict_Format_Codification_v2.md §5.2 (roll_forward strict reading)
  - cycle 03-25 R3 D-§13 DSS-CY25-§13-A (LEIBNIZ N=2 stricter verbatim sustained per MR-11)
  - cycle 03-30 R3 §1.2 D-§A1.4-P2 (Option γ hybrid 21/2 super-majority)
  - cycle 03-30 R3 §5.4 (counter ratify)
  - cycle 03-30 R3 §6.1 DSS-CY30-§A1.4-P2-α
---

# Reference/21 v0.2.1 Amendment — C-LEIBNIZ-N2-observation Counter (NEW)

## Purpose

Add observation-window counter to Reference/21 Counter Registry tracking cells currently at LEIBNIZ N=2 ceiling under Reference/20 v2 §5.2 strict reading (DSS-CY25-§13-A stricter verbatim sustained per MR-11). This counter does NOT change the BINDING N=3 ceiling per Reference/20 v2 §5.2 baseline; it provides operational visibility for the LEIBNIZ-stricter minority reading preservation.

## Background

cycle 03-25 R3 D-§13 ratified Reference/20 v2 §5.2 verdict format with roll_forward_count ceiling N=3 (R2-D Conditional PASS endless-loop hardening). LEIBNIZ N=2 stricter alternative reading was preserved as DSS-CY25-§13-A verbatim per MR-11. cycle 03-30 R3 §1.2 D-§A1.4-P2 ratified Option γ hybrid (21/2 super-majority) keeping BINDING N=3 unchanged + adding observational counter for transparency.

## Counter Specification

| Field | Value |
|-------|-------|
| **ID** | `C-LEIBNIZ-N2-observation` |
| **Type** | observation-window (special; tracks cells at LEIBNIZ N=2 ceiling under stricter reading) |
| **Definition** | Count of cells currently at `roll_forward_count=2` under Reference/20 v2 §5.2 strict reading; entries listed by `cell_id` |
| **Source of truth** | R3 ratification per cycle + `verdict.yaml` audit |
| **Update trigger** | R3 close ratification |
| **Reset condition** | Cell-level FP/FAIL transition (Reference/20 v2 §5.2 Tier 4 reset) |
| **Current value (cycle 03-30 R3 close)** | **2 cells**: `p10_t6` distributed-alaya (A1-2) + `p13_t6` guide-character-init (A1-4) |
| **First established** | cycle 03-30 R3 §1.2 D-§A1.4-P2 Option γ + §5.4 counter ratify |
| **Sunset condition** | Cell-level: 0 cells at-ceiling AND no LEIBNIZ N=2 reading invocation 5 consecutive cycles |
| **Cross-ref** | LEIBNIZ DSS-CY25-§13-A (Reference/20 v2 §7 verbatim sustained per MR-11); DSS-CY30-§A1.4-P2-α (R3 §6.1) |

## Observation semantics

- This counter is **observational, not normative**: it does NOT gate Tier transitions; the BINDING ceiling remains N=3 per Reference/20 v2 §5.2
- Each R3 close, R-team verifies which cells are at `roll_forward_count=2` (one cycle below BINDING ceiling) and lists `cell_id` entries
- Master directive 2026-05-09 (audit/fix non-mixing) compatible: this counter is metadata only, no fix injection
- LEIBNIZ N=2 stricter reading invocation requires per-cell explicit MR-11 verbatim citation; not automatic

## Cycle 03-30 R3 close inaugural entries

```yaml
counter_id: C-LEIBNIZ-N2-observation
cycle: 03-30
established: true
current_count: 2
entries:
  - cell_id: p10_t6
    cell_subject: distributed-alaya (A1-2)
    roll_forward_count: 2
    first_at_ceiling_cycle: 03-30
    leibniz_n2_invocation: false (observation only)
  - cell_id: p13_t6
    cell_subject: guide-character-init (A1-4)
    roll_forward_count: 2
    first_at_ceiling_cycle: 03-30
    leibniz_n2_invocation: false (observation only)
notes: |
  Both cells reach roll_forward_count=2 at cycle 03-30 R3 close per CP sustained verdict.
  BINDING ceiling N=3 per Reference/20 v2 §5.2 unchanged.
  LEIBNIZ stricter reading (N=2) preserved verbatim per DSS-CY25-§13-A but not invoked.
```

## Ratification chain

- cycle 03-30 R3 §1.2 D-§A1.4-P2: **21/2 super-majority** Option γ hybrid (BINDING N=3 unchanged + observational counter NEW)
- cycle 03-30 R3 §5.4: counter ratify (separate confirmation vote)
- cycle 03-30 R3 §6.1 DSS-CY30-§A1.4-P2-α: dissent preservation (Option γ minority preferring pure-status-quo OR pure-LEIBNIZ-strict)

## Sunset / promotion path

- **Sunset path**: 5 consecutive cycles with `current_count=0` AND no LEIBNIZ N=2 reading invocation → counter retired (CHANGELOG entry)
- **Promotion path**: If accumulated evidence supports LEIBNIZ N=2 stricter reading (e.g., FN observed at `roll_forward_count=3`), R-team may propose Reference/20 v2 §5.2 amendment to tighten BINDING ceiling N=3 → N=2 (separate Master Ratification batch required)

## Master Ratification target

**Batch 25 Item #7** (LEIBNIZ N=2 ceiling Option γ + counter; pending Master sign-off)

---

## End amendment
