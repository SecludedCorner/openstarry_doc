---
title: Reference/16 v3 — R4 Folder Convention Discipline (Consolidated)
date: 2026-05-12
cycle: 03-28
authors: R4-S4 subagent (ARCHIMEDES lead + SCRIBE + LINNAEUS consolidation)
status: CANDIDATE (R3 D-§B.2 ratified; awaiting Master Ratification Batch 24)
authority: cycle 03-28 R3_decision_log §3.2 + §5 (D-§B.2 + cycle close freeze rule)
supersedes: Reference/16 v2 (RATIFIED 2026-05-11 cycle 03-26 R3) + amendment_cycle03-20 + amendment_cycle03-23 (consolidated)
target_deploy: cycle 03-29 G5 hygiene (post-Master Ratification Batch 24)
mr_zt_refs: [MR-12 forward-only (substance preserved; incremental clarification only)]
---

# Reference/16 v3 — R4 Folder Convention Discipline (Consolidated)

## §1 Background (v1 + amendment_03-20 + amendment_03-23 + v2 + cycle 03-28 increments)

### §1.1 v1 → v2 transition (cycle 03-26 R3 ratify)

Reference/16 v2 RATIFIED 2026-05-11 cycle 03-26 R3 indirect via Reform Batch A; consolidates v1 main + amendment_03-20 + amendment_03-23 verbatim per MR-12 substance preservation.

### §1.2 v2 → v3 transition (cycle 03-28 R3 ratify)

Reference/16 v3 CANDIDATE per D-§B.2 (23/0 UNANIMOUS) + D-§4 (§G6 NEW 23/0 UNANIMOUS). Incremental additions:
- §5 cycle 收尾 18-item → 4-group consolidation (B-2)
- §G6 NEW cycle close freeze rule (D-§4)
- §3 responsibility matrix 4th row expansion (SPC baseline canonical doc creation per cycle 03-23 §6.4 + sibling-supersedes per cycle 03-28 §1.3.A)
- amendment_03-20 + amendment_03-23 archived `_archive/cycle03-28_v3_supersession/`

## §2 R4 Folder Structure

| Folder | Content | Typical size | Responsibility | Gate |
|--------|---------|-------------|----------------|------|
| `R0/` | Orientation artefact | 300-500 lines | SUNYATA + ARCHIMEDES | G0 |
| `R1/` | Per-chapter independent research | 4,000-6,000 lines | per-chapter authors | G1 |
| `R2/` | Cross-review per-chapter + consolidation | 3,000-5,000 lines | non-original-author reviewers | G2 |
| `R3/` | Decision log + voting + gate | 800-1,500 lines | SYNTHESIST + SCRIBE | G3 |
| `R4/` | G4 gate + synthesis | varies | SCRIBE + ARCHIMEDES | G4 |
| `deliver/` | R4 final O-series + R3 log + Master_Ratification/ + (if Dev spec) todo index | 8,000-15,000 lines | ARCHIMEDES + subagent | G4 |
| `results/` | Analysis process + data + decision evidence | 1,500-3,500 lines | **SCRIBE (primary)** | G4 soft-min |
| `openstarry_doc/` | canonical baseline snapshot + cycle-specific diff + CHANGELOG | ~342+ files | baseline copy ANY + diff R4 authorship | G4 + G5 |
| `discussions/` | Cross-stage side-bar | 3,000-8,000 lines | issue-owner agent | traceability |
| `todo/` | Dev-facing concise specs + `todo_README.md` | 500-1,500 lines | ARCHIMEDES | G4 |
| `test_instructions/` | Test-team next W-round spec | 400-600 lines/round | HERACLITUS + ARCHIMEDES | G4 |

## §3 G4 / G5 Folder Gate (with 防線 3 + 4 enforce sustained)

- **G4-folder-1**: deliver/ O-series + R3 log + Master_Ratification/
- **G4-folder-2**: results/ soft-min (README + 1 consolidation)
- **G4-folder-3** (Master directive 2026-05-01 防線 3 加嚴): openstarry_doc/ baseline ≥342 + cycle diff + CHANGELOG; **R4 SCRIBE + ARCHIMEDES 共筆 responsibility**
- **G4-folder-4**: todo/ Dev specs + todo_README
- **G4-folder-5**: test_instructions/ next W-round
- **G4-folder-6** (cycle 03-14 ratified; cycle 03-15+ binding): O7 compliance evidence freshness 24/48h

### §3.1 Responsibility Matrix (v3 expanded)

| Item | Responsibility | Source |
|------|---------------|--------|
| 1 | openstarry_doc/ baseline copy + diff | R4 SCRIBE + ARCHIMEDES 共筆 (per 防線 2 cycle 03-19+) |
| 2 | results/ consolidation | SCRIBE (primary) |
| 3 | SPC baseline canonical doc creation | coordinator pre-built (per cycle 03-23 §6.4 Option α) |
| 4 | Sibling-supersedes "active pointer" convention | front-matter `supersedes` chain OR latest-cycle wins (per cycle 03-28 §1.3.A sub-ratify; v3 NEW) |

## §4 G5 Coordinator Stage Workflow (per 防線 4 cycle 03-19+ enforce)

- G5-sync-1: 4-mirror byte-identical (suggestion / canonical / release / agent_dev)
- G5-sync-2: CHANGELOG entry across mirrors
- G5-sync-3: Master Ratification Batch dispatch
- G5-sync-4: prior_research cycle{N} update
- G5-sync-5 (Master directive 2026-05-01 防線 4): BINDING canonical doc auto-create at G5 if missing (cycle 03-29 hygiene scope)

## §5 Cycle 收尾 4-Group Checklist (v3 NEW per D-§B.2 ratify 23/0)

Per O2 §2.1 mapping; 19/20 items mapped to 4 groups (#16 deferred per cycle 03-26 §0a).

### §5.1 Group 1 — Dev release dispatch + verification (5 items)

- #4 Dev → Research transfer (release tag)
- #5 Dev → Test transfer
- #10 /simplify standard workflow attestation
- #13 release doc-only patch ceremonial
- #19 release dispatch verification (per Reference/21 §3.4 release tag 充要條件)

### §5.2 Group 2 — 5-fold research input transfer (2 items; Master directive 2026-05-12)

- #6 Test → Research W2-Rn test_data
- #20 5-fold transfer (master_letters + engineering_delivery + research_version + prior_research + test_data)

### §5.3 Group 3 — Research → Dev sync + Master Ratification (8 items)

- #1 Coordinator 合規預驗 (O7 audit)
- #2 Master Ratification Batch dispatch
- #3 Suggestion → Canonical openstarry_doc sync
- #7 prior_research/cycle{N} update
- #8 MEMORY update
- #9 CHANGELOG entry
- #15 matrix verdict canonical (audit cycle special)
- #17 Audit verdict dispatch (cycle 03→04 transition)

### §5.4 Group 4 — Multi-mirror byte-identical + 防線 verification (4 items)

- #11 agent_dev sync canonical
- #12 canonical bilingual TW parity (Rule #78 §78.5)
- #14 4 防線 enforce attestation
- #18 canonical 6-mirror byte-identical verify (7-mirror discipline per Reference/21 v0.1.3 §3.6)

## §6 Per-Group Acceptance Criteria

Each group MUST have explicit acceptance criteria; cycle 03-29 hygiene first formal application of §G6 freeze rule will surface acceptance gaps if any.

## §7 Compliance Cross-Refs

- MR-12 forward-only (v3 = v2 substance preserved + incremental clarification only)
- Reference/21 v0.1.3 §1.3 forward-binding (counter C-C2 interpretation = continuation not reset)
- Master directive 2026-05-01 4 防線 (all 4 防線 v3 codified)
- Rule #78 §78.5 EN+TW sibling BINDING-tier reflexive same-PR

## §8 v2 → v3 Delta Summary

| Section | v2 | v3 | Delta |
|---------|----|----|-------|
| §1 Background | v1 + amendment 集合 | + cycle 03-28 increment | additive |
| §2 R4 folders | unchanged | unchanged | none |
| §3 responsibility matrix | 3-row | 4-row (sibling-supersedes added) | +1 row |
| §4 G5 workflow | 5-step | 5-step | unchanged |
| §5 cycle 收尾 18-item | 18 inventory | 20→4-group | restructure |
| §6 acceptance criteria | implicit | per-group explicit | explicit |
| §7 compliance | sustained | sustained | unchanged |
| §G6 freeze rule | absent | **NEW** | NEW (D-§4 ratify) |

**No substance regression; all v2 substance preserved per MR-12 forward-only.**

## §9 Cycle Close Freeze Rule §G6 (NEW per D-§4 ratify 23/0)

> **§G6 Cycle Close Freeze Rule** (forward-binding cycle 03-29+):
> Between R4 close ratification AND coordinator G5 sync completion, NO new scope additions are permitted to the cycle's deliver/Master_Ratification/ batch. Post-G5 additions require either (a) Master explicit SCOPE CHANGE directive (precedent cycle 03-25 §SCOPE CHANGE 2026-05-07) OR (b) hygiene cycle scope (next cycle hygiene mechanical fixes per A0 fix-class triage pattern).
> Rationale: prevent scope drift between R-team R4 close and coordinator G5 propagation; ensure 4-mirror byte-identical at G5 time matches R4 ratified state.

### §9.1 Examples

- ✅ R4 close 2026-05-12; Master Ratification 2026-05-13; G5 sync 2026-05-13~14 — **freeze active** 5/12~14
- ✅ Master SCOPE CHANGE directive 2026-05-13 → unfreeze for explicit addition
- ❌ Coordinator unilateral addition during freeze window → freeze rule violation; SCRIBE flag

---

*Reference/16 v3 CANDIDATE — consolidated v2 + amendments + cycle 03-28 increments + §G6 NEW cycle close freeze rule; cycle 03-29 G5 deploy target*
