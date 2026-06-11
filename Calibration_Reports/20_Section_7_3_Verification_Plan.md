# §7.3 Coordinator 7-Step Procedure Verification Plan (Consultative)

> **Front-matter**
> - **Cycle**: 03-13
> - **Date**: 2026-04-24
> - **Authors**: SCRIBE (#2, CK-1..CK-7 authorship) + TANENBAUM (#20, R2 byte-faithful verification) + ARCHIMEDES (#16, EC edge-case review) + LINNAEUS (#13, taxonomy discipline) · Master letter v2.0 §七 + coordinator CLAUDE.md lines 140-154 source anchors
> - **Binding R3 decisions**: **CV-17 UNANIMOUS 23/0 tacit** (coordinator 7-step procedure byte-faithful to CLAUDE.md 145-152) · **D-22 UNANIMOUS 23/0** (F-11 now, Rule #79 deferred 1-2 cycles; §7.3 verification sits underneath F-11 discipline) · R3 §5.1 Batch 10 Item 11 **CONSULTATIVE** (not a ratification request)
> - **Status**: **CONSULTATIVE — Master awareness only, not ratification**. Per Batch 10 Item 11, Master acknowledgement requested as awareness. The 7-step procedure is coordinator operating pattern per CLAUDE.md, not a Rule. Recording here enables future audit.
> - **Scope**: CK-1..CK-7 research-side post-coordinator audit checklist + EC-1..EC-8 edge-case handlers. Intended to operationalise ENG-FAB v1.8 F-11 canonical-sync verification without duplicating F-11's gate text.
> - **Cross-refs**: `Research_Methodology/11_ENG_FAB_v1.8_Binding.md §C` (F-11 Canonical Sync Verified spec) · `research record/cycle03-13/deliver/O5_doc_sync_engfab_v18.md §3 + §3.4 + §3.6` (CK-1..CK-7 + EC-1..EC-8 full exposition) · `research record/cycle03-13/deliver/Master_Ratification/Batch_10_Request.md Item 11` (consultative status) · coordinator CLAUDE.md lines 140-154 (byte-faithful source)

---

## §1 Purpose and Scope

### §1.1 Purpose

This document records the **research-side post-coordinator audit checklist** (CK-1 through CK-7) and **edge-case handlers** (EC-1 through EC-8) for the coordinator's 7-step Suggestion → Canonical sync procedure.

The 7-step procedure itself is specified in `coordinator/CLAUDE.md` lines 145-152 + 153-154 (conflict-report path) — it is **Master-authored coordinator operating pattern**. This §7.3 document provides the **research verification layer** that audits coordinator execution.

### §1.2 Consultative status (not ratification)

Per Master Batch 10 Item 11 **CONSULTATIVE**: this document is **Master awareness only, not Master-ratified**. The 7-step procedure is already in force via CLAUDE.md; this verification checklist enables research-side independent audit without altering the procedure itself.

**Distinction from F-11**:
- **F-11 (ENG-FAB v1.8 MUST item)** = mechanical gate at cycle close; Master-ratified via Batch 10 Item 2; enforcement at SCRIBE G-gate level.
- **§7.3 (this document)** = research-side verification checklist that supports F-11 audit; provides CK-*/EC-* vocabulary and checklists; consultative only.

### §1.3 Byte-faithful to CLAUDE.md (CV-17 UNANIMOUS tacit)

CV-17 UNANIMOUS tacit confirms this document is byte-faithful to `CLAUDE.md` lines 145-152 + 153-154. Any discrepancy between this document and `CLAUDE.md` is resolved in favour of `CLAUDE.md` (the authoritative source); research notifies coordinator for remediation.

---

## §2 Coordinator 7-Step Procedure (CLAUDE.md 145-152, verbatim paraphrase)

Per `CLAUDE.md` lines 140-154, added 2026-04-23 by Master:

### §2.1 Trigger point

> "每輪 R4 G4 PASS + Master Ratification 完成後（cycle 收尾）"

For cycle 03-13, this is the moment after:
1. R4 O-series deliver files written to `cycle03-13/deliver/`.
2. G4 SCRIBE process gate PASS logged.
3. Master Ratification for Batch 10 (ENG-FAB v1.8 + Rules #76 §76.7 + #77 + §7.3 verification) signed off.

### §2.2 Seven steps

| Step | Action (CLAUDE.md verbatim paraphrase) | Verification artefact |
|:----:|-----------------------------------------|------------------------|
| 1 | Diff `openstarry_eco/share/research_team_suggestion/{cycle}/openstarry_doc/` (研究建議) vs `openstarry_eco/share/openstarry_doc/` (canonical) | `diff -rq` report saved to `cycle03-13/discussions/coordinator_canonical_diff.txt` |
| 2 | Copy into canonical the suggestion-internal Master-ratified diffs (新 Rules / 新 Binding / 升版 ENG-FAB 等) that canonical does not have | `git diff`-equivalent log with per-file attribution |
| 3 | **Filename collision handling**: if suggestion new file collides with canonical existing (e.g., two `17_*.md`), apply **"先到先得" principle** — canonical existing file retains its number, suggestion new file renumbered; update all internal cross-references | collision-resolution manifest (old_number, new_number, affected_files triple per entry) |
| 4 | Dev-this-cycle-new 架構級 docs (`agent_dev/openstarry/docs/TW/` or EN) incorporate into canonical `Technical_Specifications/` as Master decides | Master decision log entry |
| 5 | Sync `CHANGELOG_RESEARCH_TEAM.md` to latest (suggestion-internal version) | byte-identical check manifest |
| 6 | **Post-sync verify**: canonical == `release/{cycle}_v{X}.Y.Z-alpha/openstarry_doc/` byte-identical | sha256sum manifest (every canonical file) |
| 7 | Dispatch Dev task: pull canonical into `agent_dev/openstarry/docs/TW/` (prevent next-cycle stale) | Dev acknowledgement in dev-inbox + coordinator log |

### §2.3 Conflict-report path (CLAUDE.md 153-154)

> "若發現 suggestion vs canonical 有含義衝突（非單純檔號），coordinator 不自行裁決，送 Master"

Coordinator does NOT self-adjudicate semantic conflicts beyond mechanical renumbering. F-11 treats semantic conflict as **cycle-close blocker** pending Master ruling.

---

## §3 CK-1..CK-7 Research-Side Post-Coordinator Audit Checklist

Per `O5_doc_sync_engfab_v18.md §3.4` + CV-17 UNANIMOUS tacit.

### §3.1 CK-1 — Diff completeness

Every ratified diff (list: `cycle03-13/discussions/ratified_diff_manifest.md`) is present in canonical, not only in suggestion.

**Verification**: grep per ratified-diff-manifest entry; assert canonical entry exists.

**Target outcome**: **zero missing**. Any missing = CK-1 FAIL → F-11 FAIL cascade.

### §3.2 CK-2 — Byte-identical

`sha256sum` of every canonical new-addition file equals suggestion counterpart (modulo renumber adjustments per collision manifest).

**Verification**: `sha256sum` per file; assert equal.

**Target outcome**: **zero mismatch** outside renumber scope. Renumber-induced file-number changes are explicitly whitelisted via the collision manifest.

### §3.3 CK-3 — Collision resolution audit

Per renumber-manifest entry, verify (a) first-come retained number, (b) newcomer renumbered, (c) all cross-references in other files updated.

**Verification**: grep old number, count=0 per operator note using **word-boundary regex** `\b{NN}_` per F-03-13-§7-R1-6, to avoid partial-word false-positives (e.g., `17_` matching `172_...`).

**Target outcome**: **zero unmigrated cross-reference**. Any residue = CK-3 FAIL.

### §3.4 CK-4 — release == canonical (byte-identical)

`diff -rq` between `release/cycle03-13_v{X}.Y.Z-alpha/openstarry_doc/` and canonical = empty (per D-21 UNANIMOUS scope).

**Verification**: `diff -rq` + `sha256sum` complementary paths.

**Target outcome**: **empty diff**. Release and canonical must match byte-for-byte after coordinator sync complete.

### §3.5 CK-5 — Dev pull confirmation

`agent_dev/openstarry/docs/` contains new ratified diffs (post Dev pull; Dev step-7 acknowledgement).

**Verification**: coordinator verifies Dev ack in dev-inbox; research-side checks delta between pre-pull and post-pull snapshots.

**Target outcome**: **Dev pull confirmed** in cycle close log. **Failure mode**: if Dev pull not confirmed within cycle-close window, F-11 FAIL but with sub-severity "Dev-pull-pending" rather than "coordinator-sync-incomplete".

### §3.6 CK-6 — CHANGELOG propagation

`CHANGELOG_RESEARCH_TEAM.md` at canonical == suggestion version (byte-identical).

**Verification**: file-level sha256.

**Target outcome**: **byte-identical match**.

### §3.7 CK-7 — Cross-reference graph freshness

`grep -r "ENG-FAB v1\." canonical/` returns only v1.8 (or current ratified version); no lingering v1.7/v1.6 refs in active docs (archive docs may retain for historical record).

**Verification**: grep recursive across canonical; count mentions of deprecated versions.

**Target outcome**: **zero deprecated-version references** in active docs (documentation of prior versions in archive / history sections permitted with explicit tag).

---

## §4 EC-1..EC-8 Edge-Case Handlers

Per `O5_doc_sync_engfab_v18.md §3.6`. Each edge case has an explicit mitigation. **Experience feedback** (§3.6): if ≥ 3 distinct edge-case classes emerge during 03-13 first application, SCRIBE + coordinator draft a rule-refinement PR for Master's Batch 11 review.

### §4.1 EC-1 — Multi-number-collision cascade

**Scenario**: suggestion contains multiple new files that collide with multiple canonical existing files (e.g., `17_A.md`, `17_B.md`, `18_A.md` all colliding simultaneously).

**Mitigation**: coordinator resolves collisions deterministically in alphabetical order; renumber manifest captures all re-numberings; cross-references updated batch-wise.

### §4.2 EC-2 — Cross-reference loop

**Scenario**: file A references file B; file B references file A; both are renumbered; update order matters.

**Mitigation**: two-pass update — first pass updates referent numbers without propagating; second pass propagates via `sed`-equivalent replacement. Pass-2 idempotent.

### §4.3 EC-3 — Partial-word number match

**Scenario**: `17_*` referenced in text; naive regex also matches `172_*`, `170_*`, etc.

**Mitigation**: use word-boundary regex `\b{NN}_` or filename-anchored regex `^{NN}_` per CK-3 operator note (F-03-13-§7-R1-6).

### §4.4 EC-4 — CHANGELOG merge semantics

**Scenario**: suggestion CHANGELOG has newer cycle entries; canonical CHANGELOG has older cycle entries; merge strategy unclear.

**Mitigation**: suggestion wins (one-way replacement). Canonical CHANGELOG is always refreshed from suggestion. No content merge conflict — each cycle appends its entries to the top; no rewrite of historical entries.

### §4.5 EC-5 — Dev in-flight edits

**Scenario**: Dev mid-cycle edits landed in `agent_dev/openstarry/docs/`; canonical pull would overwrite.

**Mitigation**: coordinator checks with Dev before step-7 dispatch; if Dev has in-flight edits, coordinate merge order (Dev commits edits to suggestion first; coordinator re-syncs canonical; then Dev pulls).

### §4.6 EC-6 — Master mid-flight ratification

**Scenario**: Master ratifies mid-coordinator-sync (new Master letter arrives during step 2-5).

**Mitigation**: coordinator completes current sync run; appends new Master ratification to next cycle's sync queue. Do NOT mid-cycle re-open.

### §4.7 EC-7 — Unicode filename collision

**Scenario**: suggestion filename has unicode character; canonical has ASCII variant; OS may treat as distinct on some filesystems.

**Mitigation**: normalize all filenames to NFC Unicode (per F-12 NFR-6c); disallow filenames with Unicode characters in numeric-prefix segments.

### §4.8 EC-8 — High-velocity cycle compression

**Scenario**: consecutive cycles (03-13 R4 close → 03-14 R0 open) overlap; next cycle begins before coordinator sync completes.

**Mitigation**: coordinator sync is cycle-close blocker per F-11; next cycle cannot open its R0 until prior cycle's sync completes + F-11 PASS.

---

## §5 Failure-Mode Handling (Severity Tiers)

Per `O5_doc_sync_engfab_v18.md §3.5`. Severity count-dimension; content-importance dimension flagged for future refinement per F-03-13-§7-R2-8 INFO (non-blocking this cycle):

- **Small gap (1-3 files)**: SCRIBE writes `{cycle}/discussions/canonical_sync_remediation.md`; coordinator re-runs missing copies; verification re-run.
- **Medium gap (4-10 files)**: cycle-close blocker; R4 close postponed until remediation complete; Master notified in next Ratification batch.
- **Large gap (> 10 files, 03-12-class)**: process-failure; Master notified immediately; retrospective formally re-opened; v1.8 F-11 flagged for tightening.
- **Collision failure** (renumber incorrect or cross-refs dangling): coordinator halt; Master consulted on rename plan; no cycle close until resolved.

---

## §6 Master Awareness Record

Per Batch 10 Item 11 **CONSULTATIVE**, this document records:

1. The coordinator 7-step procedure is byte-faithful to CLAUDE.md lines 145-152 + 153-154 (CV-17 UNANIMOUS tacit).
2. The 7-step procedure is first-ever-field-applied at 03-13 R4 close itself (this very cycle).
3. The CK-1..CK-7 checklist and EC-1..EC-8 edge-case handlers provide research-side audit vocabulary that supports F-11 MUST enforcement without duplicating F-11.
4. Master acknowledgement of this document enables future audit tracking.

### §6.1 First-application observations (cycle 03-13 R4)

The first-ever field application of the coordinator 7-step procedure is **this very cycle's R4 close**. Observations will be captured in `cycle03-13/discussions/coordinator_7step_first_application_log.md` upon completion:
- Number of new ratified diffs synced: 8 (this cycle's openstarry_doc additions).
- Number of renumber collisions handled: TBD (depends on cycle 03-12 state).
- Edge cases encountered: TBD (enumerate post-application).
- Execution time: TBD.

If ≥ 3 distinct EC-* classes emerge, SCRIBE + coordinator draft a rule-refinement PR for Master's Batch 11 review per §4 closing note.

---

## §7 Status and Author Attribution

- **Status**: **CONSULTATIVE** (Master awareness only, not ratified).
- **Effective**: 2026-04-24 R4 close onward; first field application concurrent with this cycle's close.
- **Authority**: SCRIBE + coordinator jointly maintain; Master-awareness recorded via Batch 10 Item 11 APPROVE.
- **Primary authorship**:
  - SCRIBE (#2) — CK-1..CK-7 authorship + EC-1..EC-8 cataloguing.
  - TANENBAUM (#20) — R2 byte-faithful verification of CLAUDE.md conformance.
  - ARCHIMEDES (#16) — EC edge-case review.
  - LINNAEUS (#13) — taxonomy discipline.
  - SYNTHESIST (#1) — consolidation.
  - SUNYATA (#0) — chair (procedural abstain).
- **R3 vote record** (ground truth `R3_decision_log.md §3 CV-17 + §5.1 Batch 10 Item 11 + §4.4 D-22`):
  - **CV-17 UNANIMOUS 23/0 tacit** (byte-faithful to CLAUDE.md 145-152).
  - **D-22 UNANIMOUS 23/0** (F-11 now; Rule #79 defer 1-2 cycles).
  - **Batch 10 Item 11** CONSULTATIVE marker.

---

*§7.3 Coordinator 7-Step Verification Plan — Cycle 03-13 — 2026-04-24 R4 close*
*CONSULTATIVE: Master awareness only, not ratified; CV-17 UNANIMOUS byte-faithful*
*Complement to F-11 ENG-FAB v1.8 MUST item (ratified); provides CK-*/EC-* audit vocabulary*
