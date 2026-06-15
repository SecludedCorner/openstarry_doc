---
title: SPC Re-Calibration First Execution — Cycle 03-22 (Rule #72 Path C)
author: Coordinator backfill 2026-05-04 per Master directive cycle 03-23 §6.4
date: 2026-05-04
cycle: 03-22 (W2-R28 first execution); 03-23 R0+ (canonical backfill per Master directive)
status: BINDING (cycle 03-23+ forward-binding baseline)
authority: Master directive cycle 03-23 §6.4 + cycle 03-22 Batch 19 Item #5 RATIFIED 23/0 UNANIMOUS (D-§2)
supersedes: cycle 03-13 W2-R14 reference legacy limits [0.01954, 0.02420] (retired post-cycle-03-22 R4 close)
binding_until: superseded by future Rule #72 SPC re-calibration cycle (path C trigger or Master directive)
cross_refs:
  - openstarry_eco/share/openstarry_doc/Reference/baseline_rules.md (Rule #72 path C / Rule #76 §76.7 / Rule #77 σ_regime)
  - openstarry_eco/share/openstarry_doc/Calibration_Reports/17_Phase3_Baseline_R10_R11.md (legacy baseline cycle 03-12 reference)
  - openstarry_eco/share/openstarry_doc/Calibration_Reports/21_WIENER_Thresholds_Hypothesis.md
  - claude research/research record/cycle03-22/deliver/O2_W2_R28_verification_final.md
  - claude research/research record/cycle03-22/deliver/Master_Ratification/Master_Confirmation.md §3.4
  - claude research/research input/master_letters/cycle03-23/Master_letter.md §6.4 (Master directive 補建 directive)
  - openstarry_eco/share/openstarry_doc/Reference/16_R4_Folder_Convention_Discipline.md + amendment_cycle03-20.md
---

# SPC Re-Calibration First Execution — Cycle 03-22 (Rule #72 Path C)

**Status**: BINDING (cycle 03-23+ forward-binding baseline)
**Authority**: Master directive cycle 03-23 §6.4 + cycle 03-22 Batch 19 Item #5 RATIFIED 23/0 UNANIMOUS
**Backfill rationale**: cycle 03-22 R-team R4 close 漏建 + coordinator G5 防線 4 漏抓 → Master directive cycle 03-23 §6.4 補建 + Reference/16 §3 Responsibility Matrix scope expansion (SPC baseline canonical doc creation)

---

## §1 Authoritative Numerical Baseline (cycle 03-23+ forward-binding)

per Test team Plan50 50 LOC binding exact form output on cycle 03-22 W2-R28 9-datapoint pool:

| Quantity | Value |
|----------|:-----:|
| **Pooled μ** | **0.023793** |
| **Pooled σ_pool** | **0.000060** |
| **New UCL** (μ + 3 σ_pool) | **0.023972** |
| **New LCL** (μ - 3 σ_pool) | **0.023614** |
| 95% warning band (μ ± 1.96 σ_pool) | [0.023675, 0.023911] |
| Band width (UCL − LCL) | 0.000358 |

**Authoritative source**: `test research/cycle03-22/deliverables/test_report_w2r28.md`（per cycle 03-22 W2-R28 ring 2026-05-04T04:30Z）

**Plan50 50 LOC binding exact form** = canonical 數值權威（per Batch 19 Item #5 + R3-§2-E STRIKE trial arithmetic ratified）；本 baseline 為 canonical reference；任何 trial arithmetic / approximation / rough-cut 計算**不取代**此 baseline。

### §1.1 Comparison to legacy

| Quantity | Legacy (cycle 03-13 W2-R14) | New (cycle 03-22 W2-R28) | Direction |
|----------|:---------------------------:|:------------------------:|:---------:|
| LCL | 0.01954 | 0.023614 | ↑ tighter floor |
| UCL | 0.02420 | 0.023972 | ↓ tighter ceiling |
| Band width | 0.00466 | 0.000358 | **↓ ~13× tighter** |
| μ (estimated) | ~0.02187 (band midpoint) | 0.023793 | ↑ shifted up |

**Interpretation**: 新 limits 比 legacy 緊縮 ~13×（band width 0.00466 → 0.000358）；mean 上移至 0.023793。Phase 6 stack 累積 7 emit-source 後 σ pool tighter（更敏感 drift）；future cycle SPC drift sensitivity 提升。

---

## §2 9-Datapoint Pool Composition

### §2.1 Pool members (R3-§2-A 22/1 super-majority ratified)

| # | Round | σ value | Cycle | Status |
|:-:|:-----:|:-------:|:-----:|:------:|
| 1 | R10 | 0.023753 | 03-9 | INCLUDED |
| 2 | R11 | 0.023753 | 03-10 | INCLUDED |
| 3 | R12 | 0.023753 | 03-11 | INCLUDED |
| 4 | R14 | 0.023873 | 03-13 | INCLUDED |
| 5 | R15 | 0.023753 | 03-14 | INCLUDED |
| 6 | R18 | 0.023755 | 03-17 | INCLUDED |
| 7 | R20 | 0.023873 | 03-18 | INCLUDED |
| 8 | R22 | 0.023871 | 03-19 | INCLUDED |
| 9 | R24 | 0.023753 | 03-20 | INCLUDED |

**Total**: 9 datapoints; period: cycle 03-9 → cycle 03-20 (12-cycle span); σ_regime: composition_index per Rule #77.

### §2.2 EXCLUDED datapoints (R3-§2-C 22/1 symmetric first-class)

| Round | Cycle | σ value | Exclusion axis | Reason |
|:-----:|:-----:|:-------:|:--------------:|--------|
| **R5** | 03-5 | (multi-category fragmented) | Axis 2 — Classification (Rule #77 σ_regime methodology era) | Cycle 03-5 multi-category fragmented; predates cycle 03-13 Rule #77 ratification by 8 cycles; pre-Rule-#77 unified composition_index σ unavailable |
| **R26** | 03-21 | 0 (mechanical Tier 4) | Axis 1 — Distribution (ZT-3 same-distribution conjunct) | Text-only mode (provider-claude-cli text-only sustained) → 0 `audit:tool_audited` events → degenerate variance estimate; methodology artifact NOT architectural drift |
| **R28** | 03-22 | 0 (mechanical Tier 4) | Axis 1 — Distribution | Same text-only mode methodology artifact as R26 (sustained); pool 維持 9 datapoint baseline |
| **R30** (forecast) | 03-23 | TBD (likely 0) | Axis 1 — Distribution (anticipated) | provider-claude-cli text-only sustained third cycle; 預期 EXCLUDED |

### §2.3 DSS-CY22-§2-A BABBAGE Alternative R5 Historic Strict (per MR-11)

**Verbatim preserved per MR-11 dissent preservation**:

> BABBAGE alternative R5 historic strict view: cycle 03-5 multi-category fragmented σ data is *available* via per-category σᵢ extraction (event-count vector hash identity per Rule #76 §76.7 + Rule #77 σ_regime caveat); the strict-historic-inclusion view holds that R5 data SHOULD be included if pre-aggregation test (R3-§2-D ZT-3 same-distribution conjunct) PASSES. Majority view (R3-§2-C 22/1) elected symmetric first-class EXCLUSION on the basis that pre-Rule-#77 era classification (Axis 2) is non-deterministic regime; BABBAGE dissent preserved for future re-evaluation should pre-aggregation methodology evolve.

**Future re-evaluation pathway**: should pre-aggregation methodology evolve to enable Rule #77 era retroactive classification (e.g., per-category σᵢ retrofit), R5 inclusion may be reconsidered per Master directive. Until then, R5 EXCLUDED first-class.

---

## §3 Two-Axis Exclusion Taxonomy (LINNAEUS R3-§2-C Codification)

| Axis | Failure mode | R26 case | R5 case | R28+ case (forecast) |
|------|--------------|:--------:|:-------:|:-------:|
| **Axis 1 — Distribution** (ZT-3 same-distribution conjunct) | Type-2 evidence (zero-variance methodology artifact under suppressed function-calling) | ✗ FAILS (text-only σ=0; 0 `audit:tool_audited` events) | n/a (per-category σᵢ > 0 but mixed methodology) | ✗ FAILS (text-only sustained) |
| **Axis 2 — Classification** (Rule #77 σ_regime methodology era) | Pre-Rule-#77 fragmented; no unified composition_index σ available | n/a (R26 = post-Rule-#77 composition_index) | ✗ FAILS (cycle 03-5 predates Rule #77 by 8 cycles) | n/a (post-Rule-#77) |

**Future pool eligibility rule**: both-axis clear (ZT-3 distribution + Rule #77 era classification) required for inclusion. Single-axis fail → first-class exclusion.

---

## §4 Rule #72 Path C Trigger Validation

per Rule #72 path C definition: post-feature-stack-extension recalibration triggered when emit-source surface materially changes.

| Cycle | Emit-source count | Stack increment |
|:-----:|:-----------------:|---|
| 03-13 | 2 | Plan51 cap-bus + Plan52 pushInput (legacy baseline reference) |
| 03-17 | 3 | + Plan54 AC-9 |
| 03-18 | 4 | + Plan56 D-30-4 |
| 03-19 | 5 | + Plan57 D-30-5 (originally runner-level → cycle 03-21 plugin form refactor) |
| 03-21 | 6 | + Plan58 Mesh |
| **03-22** | **7** | + **Plan59 API Runtime (Class I substantive per R3-§2-B)** |

**7-component / 6 emit-source increment** vs cycle 03-13 legacy is **strongly material**. Path C trigger validated. Re-calibration overdue (would have been due cycle 03-19 absent earlier deferrals; cycle 03-22 first execution closes the gap).

---

## §5 R3-§2-E STRIKE — Trial Arithmetic Factor-of-13 Scaling Artifact

per R3-§2-E 23/0 UNANIMOUS: R1 §6.1 trial arithmetic σ_pool ≈ 0.000753 was a **factor-of-13 scaling artifact** (not authoritative). Plan50 50 LOC binding exact form output 0.000060 為 canonical 數值權威。Trial arithmetic 不取代 binding exact form。

**Rationale**: R1 trial arithmetic used coarse approximation; R3 audit identified scaling discrepancy; binding exact form re-execution at Test team produces 0.000060 (factor 12.55× tighter than trial arithmetic). Plan50 50 LOC remains canonical.

**Forward-binding implication**: future SPC re-calibration cycles MUST use Plan50 50 LOC binding exact form (or equivalent canonical per future Master directive); trial arithmetic disallowed for canonical baseline establishment.

---

## §6 Forward-Binding Effective Scope

- **Cycle 03-23+ all W2 rounds (R30, R32, R34, ...)** : evaluated against new μ ± 3σ
- **Cycle 03-22 W2-R28**: evaluated against legacy [0.01954, 0.02420] (Phase-I discipline; per O2 §4.2)
- **Legacy [0.01954, 0.02420] limits**: **RETIRED** post-cycle-03-22 R4 close (Batch 19 Item #5 ratified)

---

## §7 Compliance Attestation

| Constraint | Status | Evidence |
|------------|:------:|----------|
| MR-1~MR-13 | ✅ PASS | MR-9 Plan50 50 LOC canonical 不 WAIVE / MR-11 DSS-CY22-§2-A BABBAGE preserved verbatim §2.3 / MR-12 forward-only (legacy retired post-R4) |
| ZT-1 / ZT-2 / ZT-3 | ✅ PASS | endpoint 10/0/0★ unchanged；Axis 1 ZT-3 same-distribution conjunct enforced (R26+R28 EXCLUDED) |
| Rule #72 path C | ✅ first execution | post-feature-stack-extension recalibration triggered + executed |
| Rule #76 §76.7 caveat | ✅ applies | composition_index σ rule-deterministic; R5 EXCLUDED per Axis 2; R26+R28 EXCLUDED per Axis 1 |
| Rule #77 σ_regime | ✅ binding | unified composition_index post-cycle 03-13 baseline |
| Master directive cycle 03-23 §6.4 (本 backfill) | ✅ ratified | Master letter cycle 03-23 §6.4 sign-off 2026-05-04 |
| Tenet #10 NC PENDING per MR-5 hard | ✅ unchanged | Phase 6 7/7 完工 cycle 03-23；Tenet #10 升 COMPLIANT cycle 03-24 |

---

## §8 Activation 紀錄

- **Effective**: 2026-05-04 (cycle 03-22 R4 close + Master directive cycle 03-23 §6.4 backfill)
- **Forward-binding**: cycle 03-23+ all W2 rounds
- **Cycle 03-22 W2-R28**: legacy [0.01954, 0.02420] applies (final round under legacy)
- **Cycle 03-23 W2-R30**: new μ ± 3σ first application
- **Future re-calibration**: Rule #72 path C trigger or Master directive

---

## §9 Cross-Reference Index

- `Calibration_Reports/17_Phase3_Baseline_R10_R11.md` — legacy baseline cycle 03-12
- `Calibration_Reports/21_WIENER_Thresholds_Hypothesis.md` — adjacent calibration
- `Reference/baseline_rules.md` — Rule #72 / #76 §76.7 / #77 binding rules
- `Reference/16_R4_Folder_Convention_Discipline.md` + amendment_cycle03-20.md — coordinator G5 防線 4 BINDING canonical doc creation check (本 doc 為 scope expansion 後第一個 SPC baseline canonical case)
- claude research/research record/cycle03-22/deliver/O2_W2_R28_verification_final.md — R-team R4 deliverable source
- claude research/research record/cycle03-22/deliver/Master_Ratification/Master_Confirmation.md §3.4 — Master Ratification source
- claude research/research input/master_letters/cycle03-23/Master_letter.md §6.4 — backfill directive source
- test research/cycle03-22/deliverables/test_report_w2r28.md — Test team authoritative numerical output

---

*SPC Re-Calibration First Execution — Cycle 03-22 — BINDING — Backfilled cycle 03-23 R0 per Master directive §6.4*
*9-datapoint pool (R10/R11/R12/R14/R15/R18/R20/R22/R24) ratified per R3-§2-A 22/1 super-majority*
*R5+R26+R28 symmetric first-class EXCLUDED per R3-§2-C two-axis taxonomy*
*New μ=0.023793 / σ_pool=0.000060 / UCL=0.023972 / LCL=0.023614 — Plan50 50 LOC canonical*
*Forward-binding cycle 03-23+ baseline; legacy [0.01954, 0.02420] RETIRED post-cycle-03-22 R4 close*
*DSS-CY22-§2-A BABBAGE alternative R5 historic strict preserved verbatim per MR-11*
