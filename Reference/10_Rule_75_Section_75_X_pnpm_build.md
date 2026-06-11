---
title: Rule #75 §75.X — pnpm build MUST at Release Tag
author: ARCHIMEDES (#16 engineering practice) + KNUTH (#21 cumulative-evidence calibration) + GUARDIAN (#11 security) + DARWIN (#6 pattern analysis) + SYNTHESIST (#1 aggregator)
date: 2026-04-25
cycle: 03-14
status: CANDIDATE (pending Master Ratification Batch 11 #4)
authority: research-team (R4 final draft); Master (ratification)
supersedes: null
cross_refs:
  - research record/cycle03-14/deliver/O4_R_input_5_candidates_final.md §5 (lines 489-580)
  - research record/cycle03-14/R3/R3_decision_log.md §4 (D-§4-09 + D-§4-10 + MRB-§4-B)
  - research record/cycle03-14/deliver/Master_Ratification/Batch_11_Request.md §3.4 (Item #4)
  - research record/cycle03-12/openstarry_doc/Research_Methodology/09_Rule_75_PreDelivery_Gate_Draft.md (Rule #75 baseline)
binding_until: Master Ratification Batch 11 close
---

# Rule #75 §75.X — pnpm build MUST at Release Tag

## 1. Status

**CANDIDATE** at cycle 03-14 R4 close. Upgrades to **BINDING** if Master ratifies Batch 11 Item #4. Effective **forward-only** at next release tag after Master ratification; existing tags v0.20.0-beta..v0.49.0-alpha exempt per MR-12.

**Rule cardinality**: this is a **§75.X amendment to existing Rule #75 Pre-Delivery Gate** baseline. Rule numbering does NOT change (Rule #75 stays Rule #75); §75.X is a new sub-section appended to the §75.1..§75.10 baseline structure. Per D-§4-09 (A-7) 22/1, the Rule amendment route was preferred over a parallel ENG-FAB F-17 entry (DSS-17 absorbed).

## 2. R3 Provenance

- **D-§4-09 (A-7)** 22/1 — Rule #75 amendment route adopted over parallel ENG-FAB F-17 entry; DSS-17 (1 dissent for F-17 visibility preference) preserved verbatim §10.
- **D-§4-10 (C-5)** UNANIMOUS 23/0 — release-tag-only scope (Wave-internal NOT required per KNUTH cumulative-evidence calibration).
- **MRB-§4-B** RESOLVED — KNUTH cumulative-evidence calibration (1/4 hit rate over Plan46-Plan49 window; 100% true-positive on observed case; cost-asymmetry false-positive << false-negative) supports MUST elevation on 1-incident.

## 3. Incident Evidence (KNUTH Cumulative-Evidence Calibration)

The amendment is incident-driven by Plan49 v0.49.0-alpha shipped-but-broken event:

- **Date**: cycle 03-13 R4 close + Dev release tagging (~2026-04-24).
- **Symptom**: Plan49 v0.49.0-alpha tagged and pushed to release branch with `zod` runtime dependency missing from `package.json`.
- **Visible window**: ~18 hours between tag and incident-discovery.
- **Why vitest/test-suite did not catch it**: vitest runs with hoisting of imports during test setup; `zod` was transitively pulled in via a dev dependency at test time, making the test suite pass even though a fresh `pnpm install --production` followed by `pnpm build` from a clean checkout would have failed at build time on the missing runtime dep.
- **Why incident discovered**: Dev attempted clean checkout for cross-OS verification; `pnpm build` failed with module-not-found.
- **Resolution**: hot-fix tag v0.49.0-alpha+1 with corrected `package.json` deps.

KNUTH cumulative-evidence calibration over Plan46-Plan49 release window:

| Plan | Release tag | Pre-tag fresh-build PASS would have prevented incident? |
|------|-------------|---------------------------------------------------------|
| Plan46 | v0.46.0-alpha | N/A — no incident |
| Plan47 | v0.47.0-alpha | N/A — no incident |
| Plan48 | v0.48.0-alpha | N/A — no incident |
| Plan49 | v0.49.0-alpha | **YES** — fresh build would have caught zod missing at tag time |

**Hit rate**: 1/4 (25%) over Plan46-49 window. **True-positive on observed case**: 100%. **Cost asymmetry**: a release-tag fresh build adds ~1-3 minutes to release procedure; an incident with shipped-broken release adds ~6-18 hours of recovery + reputation cost. **Cost-asymmetry false-positive cost << false-negative cost** by ~2-3 orders of magnitude.

KNUTH discipline: 1-incident MUST elevation justified when (a) hit rate ≥ 25%, (b) true-positive 100% on observed case, (c) cost-asymmetry ≥ 100x. All three conditions met. Per D-§4-09 (A-7) 22/1, R3 ratified MUST elevation.

## 4. Binding Text

### 4.1 §75.X — pnpm build MUST at Release Tag

> **Rule #75 §75.X — pnpm build MUST at Release Tag**
>
> Dev MUST execute `pnpm build` from a fresh repository state before tagging any release version (`v*.*.*-alpha`, `v*.*.*-beta`, or `v*.*.*` GA release tag).
>
> The verification chain is:
>
> ```bash
> pnpm install --frozen-lockfile && pnpm build
> ```
>
> executed from a clean checkout (or ephemeral CI runner) of the to-be-tagged commit. `pnpm install --frozen-lockfile` is REQUIRED — auto-update of the lockfile during the gate is forbidden, since it would mask the very dependency-drift class this gate detects.
>
> Build MUST exit with status 0. **Non-zero exit = delivery block; no PROVISIONAL state**. The release tag MUST NOT be pushed until build succeeds.
>
> Evidence MUST be captured in `delivery_report.md §pnpm_build_evidence` section, including:
>
> - command transcript (or full stdout/stderr capture)
> - tool versions: `pnpm --version`, `node --version`, OS / arch
> - duration: `<wall-clock seconds>`
> - exit status: `0`
> - artefact hash (optional but recommended): SHA-256 of the produced build output tree

### 4.2 Scope (D-§4-10 UNANIMOUS)

**In scope**: release-tag events only. Specifically:

- `v*.*.*-alpha` tags (e.g., `v0.50.0-alpha`)
- `v*.*.*-beta` tags (e.g., `v0.50.0-beta`)
- `v*.*.*` GA release tags (e.g., `v0.50.0`)
- Any future tag-class additions ratified by Master

**Out of scope** (Wave-internal NOT required):

- Wave-internal commits within an in-progress Plan (Wave 1 / Wave 2 / etc.)
- Bug-fix branch HEAD before merge
- Dev-internal feature branches
- Local development workflows

KNUTH cumulative-evidence rationale: the 1/4 hit rate is concentrated at release-tag boundaries (composition + dependency-graph closure happens at release-tag scope, not at intermediate Wave commits). Wave-internal `pnpm build` is good Dev practice but not Rule-mandated; release-tag is where the cost-asymmetry justifies the gate.

### 4.3 Pre-Delivery Gate Integration

§75.X integrates with the existing Rule #75 Pre-Delivery Gate as the **release-tag verification phase**:

- **§75.1..§75.10** — existing Pre-Delivery checks remain unchanged (Wave-internal δ-status declaration, runtime evidence completeness, ENG-FAB audit attestation, etc.).
- **§75.X (new)** — release-tag-specific verification: fresh-state pnpm build PASS.

The two layers compose: Wave-internal §75.1..§75.10 satisfaction does NOT exempt release-tag §75.X; both must pass at release-tag time.

### 4.4 Failure Behaviour

On `pnpm build` exit ≠ 0:

1. Tag operation aborted; release branch HEAD reverted to pre-tag state.
2. Dev investigates failure root cause (typically: missing dep, type-check failure, lint failure escalated to error, build-tool config drift).
3. Fix committed on release branch (or merged from feature branch); fresh checkout repeats §75.X verification.
4. Once §75.X passes, tag operation re-attempted.
5. **No PROVISIONAL state** — build either passes (tag proceeds) or fails (tag blocked). There is no "PROVISIONAL pass with caveat" branch.

The strict pass/fail binary absorbs DARWIN's pattern-analysis observation: gate semantics with PROVISIONAL state historically erode into rubber-stamp habit; binary semantics preserve gate function over cycle-time.

### 4.5 Frozen-Lockfile Rationale

The `--frozen-lockfile` flag is **required**, not optional:

- Without `--frozen-lockfile`, pnpm may auto-update the lockfile to satisfy a missing dependency, masking the very dependency-drift the gate detects.
- With `--frozen-lockfile`, pnpm aborts with non-zero exit if the lockfile cannot be honoured exactly — surfacing dependency declarations that don't match the lockfile.
- This catches the Plan49 zod class precisely: a runtime dep newly imported but not yet added to `package.json` would surface here.

### 4.6 Evidence Capture Format

`delivery_report.md §pnpm_build_evidence` section template:

```markdown
### pnpm_build_evidence (Rule #75 §75.X)

- **Tag**: v0.50.0-alpha
- **Commit**: <full SHA>
- **Date**: <ISO-8601 timestamp>
- **Verification chain**: `pnpm install --frozen-lockfile && pnpm build`
- **Tool versions**:
  - pnpm: <version>
  - Node: <version>
  - OS / arch: <e.g., Linux x86_64>
- **Wall-clock duration**: <seconds>
- **Exit status**: 0
- **Build artefact hash (optional)**: <SHA-256>
- **Notes**: <any non-blocking warnings worth recording>
```

## 5. Routing Rationale: Rule Amendment vs ENG-FAB F-17

Per D-§4-09 (A-7) 22/1, R3 selected the Rule #75 §75.X amendment route over a parallel ENG-FAB F-17 entry. Comparison:

| Aspect | Rule #75 §75.X amendment | ENG-FAB F-17 (rejected) |
|--------|--------------------------|--------------------------|
| Item count | Rule #75 unchanged; §75.X sub-section | ENG-FAB v1.8 = 48 → v1.10 = 50 (with F-16) — multiple increments |
| Authority surface | Single Rule amendment to ratify | Single new ENG-FAB item to ratify |
| Audit traceability | Folded into existing Rule #75 audit | New separate F-17 audit entry per Plan |
| Failure mode | Same Pre-Delivery Gate failure path | Separate ENG-FAB failure path (potential coverage gap if Pre-Delivery Gate runs but F-17 audit doesn't) |
| Decoupled-slot strategy | Decouples release-tag concern (incident-driven) from anticipatory ENG-FAB v1.9 = 49 candidate (F-16) | Couples release-tag to ENG-FAB increment surface |

R3 majority (22/1) preferred Rule amendment on grounds of:
1. Single integrated Pre-Delivery Gate path (less potential for divergent audit logic).
2. Decoupled-slot ratification — Rule amendment ratifies independently of ENG-FAB v1.9 increment.
3. Existing Rule #75 §75.1..§75.10 already covers Pre-Delivery Gate semantics; §75.X is the natural extension.

DSS-17 preserved verbatim §10.

## 6. Effective Date

- **Master ratification effective date**: forward-only at next release tag after Master ratification.
- **Existing tags exempt** (per MR-12): v0.20.0-beta through v0.49.0-alpha NOT retrofitted; the gate begins applying at v0.50.0-alpha or first post-ratification tag.
- **Hot-fix tags following non-compliant existing release**: §75.X applies to the hot-fix tag itself (since it is itself a new tag), but does NOT retroactively block the original existing tag.

## 7. Compliance Attestation

| Constraint | Status | Evidence |
|------------|:------:|----------|
| MR-2 / MR-4 (Tenet wording unchanged) | ✅ PASS | No Tenet wording proposed |
| MR-5 hard (Tenet #10 status unchanged) | ✅ PASS | Pre-Delivery Gate amendment; no compliance status implication |
| MR-6 (Core 零) | ✅ PASS | Delivery gate; no Core surface |
| MR-7 (audit at right moment) | ✅ PASS | Release-tag is the precise audit-moment for fresh-build verification |
| MR-8 (quality first) | ✅ PASS | KNUTH cumulative-evidence 1-incident MUST elevation justified |
| MR-9 (no MUST WAIVE) | ✅ PASS | MUST is final; binary pass/fail semantics |
| MR-10 (back-fill) | ✅ PASS | Release-tag enum extensible (future tag classes) |
| MR-11 (dissent preservation) | ✅ PASS | DSS-17 verbatim §10 |
| MR-12 (既有 not retrofit) | ✅ PASS | Existing tags v0.20-v0.49 exempt; forward-only |
| ZT-1 / ZT-2 / ZT-3 | ✅ PASS | No 10-Tenet violation; endpoint unchanged; control-range unaffected |

## 8. Authority Notes

- **Authority route**: Master ratification (Rule #75 amendment).
- **Coordinator G5 sync**: upon Master ratification, coordinator syncs §75.X amendment text into canonical `Reference/<NN>_Rule75_pnpm_build_amendment.md` (per Batch 11 Request §7 mapping; numbering per 先到先得).
- **Dev acknowledgement**: Dev integrates §75.X into release tagging workflow; Plan52 release tag (cycle 03-15+) is the first effective enforcement point.

## 9. Cross-References

- `research record/cycle03-14/deliver/O4_R_input_5_candidates_final.md §5` — §75.X amendment narrative source
- `research record/cycle03-14/R3/R3_decision_log.md §4 D-§4-09 + D-§4-10` — R3 voting record
- `research record/cycle03-14/deliver/Master_Ratification/Batch_11_Request.md §3.4 Item #4` — ratification dispatch
- `research record/cycle03-12/openstarry_doc/Research_Methodology/09_Rule_75_PreDelivery_Gate_Draft.md` — Rule #75 baseline (which §75.X amends)
- `research record/cycle03-14/deliver/O4_R_input_5_candidates_final.md §6 verify-package-deps.mjs` — complementary fast-feedback Dev tooling (Item #5)

## 10. Dissent Slot — DSS-17 Verbatim (per MR-11)

**DSS-17 (D-§4-09, 1 minority vote)**:

> Single dissent in favour of parallel ENG-FAB F-17 entry over Rule #75 §75.X amendment:
>
> "ENG-FAB visibility argument: ENG-FAB checklist is the single document Dev consults to enumerate required pre-tag verifications. Folding the pnpm build requirement into Rule #75 §75.X means a Dev reading 'ENG-FAB v1.8 checklist for Plan 50 release tag' may not encounter the §75.X requirement as a top-level checklist item — it is buried in Rule #75 sub-section reading. A parallel F-17 entry would surface the requirement at the same level as F-13 / F-14 / F-15 / F-16 in the next-version increment, increasing visibility and reducing the chance of a forgotten gate."
>
> R3 majority (22/1) preferred Rule #75 amendment on grounds:
> - Pre-Delivery Gate (Rule #75) is the canonical pre-tag verification gate; visibility is achieved by Rule #75 itself being the canonical reference at release-tag boundary.
> - ENG-FAB checklist can — as a Dev-side practice — include a one-line reference to "Rule #75 §75.X release-tag pnpm build" in its checklist enumeration, gaining visibility without item-count proliferation.
> - Decoupled-slot strategy advantage: Rule amendment ratifies independently of ENG-FAB v1.9 increment, reducing single-vector ratification risk.

The DSS-17 visibility concern is mitigated by the recommendation that ENG-FAB v1.9 checklist (when ratified) include a one-line cross-reference to Rule #75 §75.X in its release-tag verification listing — this preserves checklist visibility without creating a parallel audit path.

---

*Cycle 03-14 R4 final — 2026-04-25*
*Authors: ARCHIMEDES (#16) + KNUTH (#21) + GUARDIAN (#11) + DARWIN (#6) + SYNTHESIST (#1)*
*Status: CANDIDATE (pending Master Ratification Batch 11 #4)*
