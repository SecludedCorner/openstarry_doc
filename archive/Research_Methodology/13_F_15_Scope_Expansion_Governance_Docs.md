---
title: F-15 Scope Expansion — Governance Doc Front-Matter Schema
author: SCRIBE (#2 records lead) + LINNAEUS (#13 taxonomy) + ARCHIMEDES (#16 engineering) + SYNTHESIST (#1 aggregator)
date: 2026-04-25
cycle: 03-14
status: CANDIDATE (pending Master Ratification Batch 11 #2)
authority: research-team (R4 final draft); Master (ratification)
supersedes: null
cross_refs:
  - research record/cycle03-14/deliver/O4_R_input_5_candidates_final.md §3 (lines 220-386)
  - research record/cycle03-14/R3/R3_decision_log.md §4 (D-§4-05 + D-§4-06)
  - research record/cycle03-14/deliver/Master_Ratification/Batch_11_Request.md §3.2 (Item #2)
  - research record/cycle03-13/openstarry_doc/Research_Methodology/11_ENG_FAB_v1.8_Binding.md (F-15 baseline)
binding_until: Master Ratification Batch 11 close
---

# F-15 Scope Expansion — Governance Doc Front-Matter Schema

## 1. Status

**CANDIDATE** at cycle 03-14 R4 close. Upgrades to **BINDING** if Master ratifies Batch 11 Item #2 (post-R4). Coordinator G5 sync upgrades canonical mirror to BINDING upon Master ratification.

**ENG-FAB v1.8 = 48 items canonical PRESERVED**: this document is an **F-15 scope amendment**, not an item-count increment. The ENG-FAB roster does not change cardinality. CV-§4-α (R3 cycle 03-14) reaffirmed UNANIMOUS.

## 2. R3 Provenance

- **D-§4-05 (A-5)** 21/2 — bounded enumeration of 8 IN-SCOPE doc classes adopted over R1's "all governance docs" essentialist scope (per NAGARJUNA Madhyamaka critique + ASANGA Yogācāra critique; DSS-6 absorbed verbatim in §10).
- **D-§4-06 (C-4)** UNANIMOUS 23/0 — 7-status-constant disambiguation (KNOWLEDGE_ONLY post-hoc archival vs CONSULTATIVE pre-decision advisory) ratified as the canonical status enum.
- **CV-§4-α** reaffirmed (no item-count change).
- **CV-§4-β** reaffirmed (MR-12 既有 not retrofit honoured).

## 3. Binding Text

### 3.1 Required front-matter fields (7 mandatory + 2 optional)

Every IN-SCOPE governance doc (per §3.2 enumeration) authored on or after the Master ratification effective date MUST carry the following YAML front-matter block at the top of the file:

```yaml
---
title: <human-readable doc title>
author: <persona(s) with authorship + R-stage role; comma-separated>
date: <ISO-8601 YYYY-MM-DD; date of authorship-close>
cycle: <"03-NN" | null>
status: <BINDING | CANDIDATE | KNOWLEDGE_ONLY | CONSULTATIVE | SCRIBE_INTERNAL | DEPRECATED | SUPERSEDED>
authority: <who has decision rights; e.g. "research-team"; "Master"; "Dev">
cross_refs:
  - <relative path or anchor to related doc; one entry per line>
# Optional:
supersedes: <relative path to superseded doc OR null>
binding_until: <event-anchored expiration; e.g. "Master Ratification Batch 11 close" OR null>
---
```

**`cycle:` field discipline**: legacy docs predating the cycle convention use YAML-native `null` (NOT the `"(pre-cycle)"` magic string from earlier transitional drafts). The `null` sentinel is unambiguous YAML and parses identically across YAML 1.1 / 1.2 implementations. All linter logic treats `null` as "no cycle anchor; treat as static reference".

### 3.2 Bounded enumeration of 8 IN-SCOPE doc classes

The schema applies — and only applies — to documents in the following 8 classes. The enumeration is **closed** under cycle 03-14 R4 close; future expansion requires Master Ratification of an explicit class addition.

| # | Class | Locator pattern | Notes |
|:-:|-------|-----------------|-------|
| 1 | **O-series deliverables** | `research record/cycle**/deliver/O*.md` | All numbered O-series files (O1, O2, ..., O7+) |
| 2 | **R-stage artefacts** | `research record/cycle**/R{0,1,2,3,4}/**/*.md` | R0 orientation; R1/R2 per-chapter; R3 decision log; R4 synthesis |
| 3 | **Plan specs** (research-side) | `research record/cycle**/deliver/Plan*_spec.md`; `research record/cycle**/openstarry_doc/Technical_Specifications/Plan*.md` | research-team Plan-level specs (NOT Dev `agent_dev/openstarry/docs/` operational docs which retain Dev's own header style per MR-12) |
| 4 | **Canonical openstarry_doc files** | `openstarry_eco/share/openstarry_doc/**/*.md`; `research record/cycle**/openstarry_doc/**/*.md` | All baseline mirror + cycle-specific diff entries |
| 5 | **Master Ratification documents** | `research record/cycle**/deliver/Master_Ratification/Batch_*.md` | Batch dispatch + addenda |
| 6 | **R3 decision logs** | `research record/cycle**/R3/R3_decision_log.md`; `research record/cycle**/deliver/R3_decision_log_copy.md` | Both author copy and deliver/ copy |
| 7 | **todo specs** | `research record/cycle**/todo/**/*.md`; `research record/cycle**/todo/todo_README.md` | Dev-facing concise specs + transfer manifests |
| 8 | **test_instructions** | `research record/cycle**/test_instructions/**/*.md` | Test-team-facing W-round instructions |

**OUT-OF-SCOPE** (NOT subject to F-15 schema):

- inbox doorbell files (`inbox/*-inbox.txt`) — content is meaningless ring-trigger
- monitor logs / process logs (`discussions/SCRIBE_*_log.md` for runtime traces)
- scratch notes / transient discussion drafts < 500 lines unless cited by a deliver/ artefact
- Dev-side `agent_dev/openstarry/docs/**` (Dev retains own header style per MR-12 cross-team boundary)
- novel content (`research record/cycle**/results/novel/**` retains literary front-matter, not governance)

**EDGE case**: `discussions/**/*.md` files INCLUDE under F-15 schema **iff cited by** `deliver/`, `todo/`, or `test_instructions/`. The citation creates governance-grade cross-reference; the citation pulls the discussion into the audit graph. Uncited side-bar drafts remain OUT-OF-SCOPE.

### 3.3 Status enum — 7 constants with SCRIBE 1-sentence definitions

Per D-§4-06 (C-4) UNANIMOUS, the status enum is closed at 7 constants:

| # | Constant | SCRIBE 1-sentence definition |
|:-:|----------|------------------------------|
| 1 | `BINDING` | Decision is currently in force; consumers MUST follow; supersession requires another BINDING-level act |
| 2 | `CANDIDATE` | Decision is drafted and ratification-routed; pending Master / authority confirmation; consumers should NOT yet rely |
| 3 | `KNOWLEDGE_ONLY` | Post-hoc archival informational record; not a decision instrument; references for context only |
| 4 | `CONSULTATIVE` | Pre-decision advisory artefact intended to inform a future ratification; not itself binding |
| 5 | `SCRIBE_INTERNAL` | Research-team self-binding (G6.* / G4-folder-* gate authority); Master courtesy-noticed but not Master-route ratification |
| 6 | `DEPRECATED` | Formerly BINDING; now retired; consumers must migrate per supersession path; left in place for trace |
| 7 | `SUPERSEDED` | Replaced by a newer BINDING doc; readers redirected to `supersedes` field of the replacement |

**KNOWLEDGE_ONLY vs CONSULTATIVE distinction** (per D-§4-06 UNANIMOUS clarification):
- `KNOWLEDGE_ONLY` = retrospective; documents what happened
- `CONSULTATIVE` = prospective; advises a future decision

The two cannot be substituted: a doc seeking to influence a Batch ratification is `CONSULTATIVE`; a doc capturing post-decision context is `KNOWLEDGE_ONLY`.

### 3.4 `cycle:` field semantics

| Value | Meaning |
|-------|---------|
| `"03-14"` | Doc was authored within cycle 03-14 R-stage |
| `"03-13"` | Doc anchors decisions made at cycle 03-13 (typical for canonical openstarry_doc baseline-mirror entries) |
| `null` | Doc predates the cycle convention OR is a perpetual static reference (e.g. `Project_Structure_and_Conventions/*`) |

The `null` sentinel replaces the earlier transitional `"(pre-cycle)"` parenthesised magic string. YAML loaders treat `null` and the absence of the key identically; the explicit `cycle: null` keeps the field present for linter parsing.

### 3.5 F-15 linter `tools/f15_check.py` extension

The existing F-15 linter (cycle 03-13 ENG-FAB v1.8 baseline) is extended with:

1. **Class-routing**: walk the 8 IN-SCOPE locator patterns from §3.2; skip OUT-OF-SCOPE paths.
2. **Schema validation**: parse YAML front-matter; assert 7 mandatory fields present; assert `status` value in 7-enum; assert `cycle` ∈ `{"03-NN", null}`.
3. **Cross-ref resolution**: for each entry in `cross_refs`, attempt path resolution; emit MED finding for unresolvable references (allow soft-pass for cycle-future references).
4. **Supersession chain**: when `supersedes` is set, assert target exists and target's `status` ∈ `{DEPRECATED, SUPERSEDED}`.
5. **`null` discipline**: emit LOW informational on any `cycle: "(pre-cycle)"` legacy-magic-string occurrence (back-fill recommendation; no MUST retrofit per MR-12).

Linter exit code: 0 default; `--strict` exits non-zero on any HIGH finding. JSON-lines output via `--json` flag.

## 4. Strength

**MUST forward (cycle 03-14+)**.

- All cycle 03-14 R-stage artefacts authored on/after Master ratification effective date MUST adopt the schema.
- All cycle 03-14 deliver/ + openstarry_doc/ + todo/ + test_instructions/ MUST adopt the schema.
- **既有 (pre-cycle-03-14) docs are NOT retrofitted per MR-12** (CV-§4-β reaffirm). Existing docs continue to work; future modifications to those docs MAY (not MUST) adopt the schema on the modified file.

## 5. Apply Scope

### 5.1 In scope

- All 8 IN-SCOPE doc classes per §3.2.
- Forward-only from cycle 03-14 R4 close (effective post-Master-ratification).
- New authoring; pre-existing content not retrofitted.

### 5.2 Out of scope

- All 5 OUT-OF-SCOPE patterns per §3.2.
- Pre-cycle-03-14 docs (legacy preserved as-is).
- Dev-side documentation system (cross-team boundary per MR-12).

### 5.3 Edge case discipline

When an existing pre-cycle-03-14 doc is **modified** in cycle 03-14+ for any substantive reason (content addition, decision update, errata), the author SHOULD add the F-15 front-matter at modification time. This is the standard MR-12 "modify-touch only" discipline — the unmodified surrounding docs remain exempt.

## 6. Effective Date

- **Cycle 03-14 R4 close** (this dispatch): all cycle 03-14 R-stage / deliver / openstarry_doc / todo / test_instructions adopt the schema (already adopted in this cycle's R4 outputs).
- **Cycle 03-15 onward**: F-15 linter MUST run as part of G4-folder gates; failures block G4 close.
- **Backfill**: NONE required for pre-cycle-03-14 docs.

## 7. Compliance Attestation

| Constraint | Status | Evidence |
|------------|:------:|----------|
| MR-2 / MR-4 (Tenet wording unchanged) | ✅ PASS | No Tenet wording proposed |
| MR-5 hard (Tenet #10 status unchanged) | ✅ PASS | F-15 is doc-layer schema; no compliance status implication |
| MR-6 (Core 零) | ✅ PASS | Doc layer; no Core surface |
| MR-7 (timing audit) | ✅ PASS | Effective only post-ratification; back-fill via natural authoring |
| MR-9 (no MUST WAIVE) | ✅ PASS | MUST forward; back-fill exemption is per MR-12 not waiver |
| MR-10 (back-fill) | ✅ PASS | Per-doc-class fields back-fillable; modify-touch discipline |
| MR-11 (dissent preservation) | ✅ PASS | DSS-6 verbatim §10 |
| MR-12 (既有 not retrofit) | ✅ PASS | Pre-cycle-03-14 docs exempt |
| ZT-1 / ZT-2 / ZT-3 | ✅ PASS | No 10-Tenet violation; endpoint unchanged; control-range unaffected |

## 8. Cross-References

- `research record/cycle03-14/deliver/O4_R_input_5_candidates_final.md §3` — F-15 amendment narrative source
- `research record/cycle03-14/R3/R3_decision_log.md §4 D-§4-05 + D-§4-06` — R3 voting record
- `research record/cycle03-14/deliver/Master_Ratification/Batch_11_Request.md §3.2 Item #2` — ratification dispatch
- `research record/cycle03-13/openstarry_doc/Research_Methodology/11_ENG_FAB_v1.8_Binding.md` — F-15 ENG-FAB v1.8 baseline (which this amends without count change)

## 9. Authority Notes

- **Authority route**: Master ratification (F-15 amendment; no item-count change).
- **Effective once Master ratifies**: coordinator G5 sync upgrades canonical mirror status from CANDIDATE → BINDING.
- **SCRIBE responsibility**: F-15 linter `tools/f15_check.py` extension implementation falls to coordinator post-R4 task (not Dev) since it is internal research-team tooling on research-team docs. Final tooling location may be `research agent corps/shared/tools/f15_check.py`.

## 10. Dissent Slot — DSS-6 Verbatim (per MR-11)

**DSS-6 (D-§4-05, 2 minority votes)**:

> NAGARJUNA + ASANGA philosophical objection to bounded enumeration:
>
> "The R1 'all governance docs' formulation honoured the no-self-existence (anātman / śūnyatā) principle that doc-class boundaries are themselves dependently-arising and resist essentialist enumeration. Bounding the schema to 8 classes risks reifying the class boundaries as if they had inherent existence (svabhāva). The Madhyamaka preference is for an open intension ('all governance docs as conventionally identified') with case-by-case adjudication, allowing the dependent-arising of new classes without requiring Master Ratification each time."
>
> ASANGA Yogācāra refinement: "The eight-consciousness lens distinguishes ālaya-vijñāna (storehouse) artefacts from manas / pravṛtti-vijñāna (active deliberation) artefacts; an enumeration that conflates these conceals the epistemic-status difference."
>
> R3 majority (21/2) preferred bounded enumeration on engineering grounds: closed enumeration enables linter automation, audit completeness, and coordinator G5 sync mechanical verification. Future class additions remain open via Master Ratification.

Both dissenting positions are documented; the bounded enumeration adopted is conventional (saṃvṛti-satya) per the Madhyamaka two-truth framing — at the operational level, the linter requires a closed set; at the philosophical level, the closure is acknowledged as conventional, not essentialist.

---

*Cycle 03-14 R4 final — 2026-04-25*
*Authors: SCRIBE (#2) + LINNAEUS (#13) + ARCHIMEDES (#16) + SYNTHESIST (#1)*
*Status: CANDIDATE (pending Master Ratification Batch 11 #2)*
