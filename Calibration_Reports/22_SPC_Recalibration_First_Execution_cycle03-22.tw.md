---
title: SPC 重校準首次執行 — Cycle 03-22 (Rule #72 Path C)
author: Coordinator backfill 2026-05-04 per Master directive cycle 03-23 §6.4
date: 2026-05-04
cycle: 03-22 (W2-R28 首次執行); 03-23 R0+ (canonical backfill per Master directive)
status: BINDING (cycle 03-23+ forward-binding 基線)
authority: Master directive cycle 03-23 §6.4 + cycle 03-22 Batch 19 Item #5 RATIFIED 23/0 UNANIMOUS (D-§2)
supersedes: cycle 03-13 W2-R14 reference legacy 限值 [0.01954, 0.02420] (cycle 03-22 R4 close 後 retired)
binding_until: 未來 Rule #72 SPC 重校準 cycle (path C trigger 或 Master directive) 取代
cross_refs:
  - openstarry_eco/share/openstarry_doc/Reference/baseline_rules.md (Rule #72 path C / Rule #76 §76.7 / Rule #77 σ_regime)
  - openstarry_eco/share/openstarry_doc/Calibration_Reports/17_Phase3_Baseline_R10_R11.md (legacy baseline cycle 03-12 reference)
  - openstarry_eco/share/openstarry_doc/Calibration_Reports/21_WIENER_Thresholds_Hypothesis.md
  - claude research/research record/cycle03-22/deliver/O2_W2_R28_verification_final.md
  - claude research/research record/cycle03-22/deliver/Master_Ratification/Master_Confirmation.md §3.4
  - claude research/research input/master_letters/cycle03-23/Master_letter.md §6.4 (Master directive 補建 directive)
  - openstarry_eco/share/openstarry_doc/Reference/16_R4_Folder_Convention_Discipline.md + amendment_cycle03-20.md
---

# SPC 重校準首次執行 — Cycle 03-22 (Rule #72 Path C)

**狀態**: BINDING (cycle 03-23+ forward-binding 基線)
**權威**: Master directive cycle 03-23 §6.4 + cycle 03-22 Batch 19 Item #5 RATIFIED 23/0 UNANIMOUS
**Backfill 理由**: cycle 03-22 R-team R4 close 漏建 + coordinator G5 防線 4 漏抓 → Master directive cycle 03-23 §6.4 補建 + Reference/16 §3 Responsibility Matrix scope expansion (SPC 基線 canonical 文件創建)

---

## §1 權威數值基線（cycle 03-23+ forward-binding）

per Test team Plan50 50 LOC binding exact form 在 cycle 03-22 W2-R28 9-datapoint pool 上的輸出：

| 量 | 值 |
|----------|:-----:|
| **Pooled μ** | **0.023793** |
| **Pooled σ_pool** | **0.000060** |
| **新 UCL** (μ + 3 σ_pool) | **0.023972** |
| **新 LCL** (μ - 3 σ_pool) | **0.023614** |
| 95% 警告帶 (μ ± 1.96 σ_pool) | [0.023675, 0.023911] |
| 帶寬 (UCL − LCL) | 0.000358 |

**權威來源**: `test research/cycle03-22/deliverables/test_report_w2r28.md`（per cycle 03-22 W2-R28 ring 2026-05-04T04:30Z）

**Plan50 50 LOC binding exact form** = canonical 數值權威（per Batch 19 Item #5 + R3-§2-E STRIKE trial arithmetic ratified）；本基線為 canonical 參考；任何 trial arithmetic / 近似 / rough-cut 計算**不取代**此基線。

### §1.1 與 legacy 比較

| 量 | Legacy (cycle 03-13 W2-R14) | New (cycle 03-22 W2-R28) | 方向 |
|----------|:---------------------------:|:------------------------:|:---------:|
| LCL | 0.01954 | 0.023614 | ↑ 下限收緊 |
| UCL | 0.02420 | 0.023972 | ↓ 上限收緊 |
| 帶寬 | 0.00466 | 0.000358 | **↓ 約 13× 收緊** |
| μ (估計) | ~0.02187 (帶中點) | 0.023793 | ↑ 平均上移 |

**詮釋**: 新限值比 legacy 收緊約 13×（帶寬 0.00466 → 0.000358）；mean 上移至 0.023793。Phase 6 stack 累積 7 個 emit-source 後 σ pool 更緊（對 drift 更敏感）；未來 cycle SPC drift 偵測敏感度提升。

---

## §2 9-Datapoint Pool 組成

### §2.1 Pool 成員 (R3-§2-A 22/1 super-majority ratified)

| # | Round | σ 值 | Cycle | 狀態 |
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

**總計**: 9 datapoints；期間: cycle 03-9 → cycle 03-20 (12-cycle span)；σ_regime: composition_index per Rule #77。

### §2.2 EXCLUDED datapoints (R3-§2-C 22/1 對稱第一類)

| Round | Cycle | σ 值 | 排除軸 | 理由 |
|:-----:|:-----:|:-------:|:--------------:|--------|
| **R5** | 03-5 | (multi-category fragmented) | Axis 2 — Classification (Rule #77 σ_regime methodology era) | Cycle 03-5 multi-category fragmented；早於 cycle 03-13 Rule #77 ratification 8 cycles；pre-Rule-#77 unified composition_index σ 不可得 |
| **R26** | 03-21 | 0 (mechanical Tier 4) | Axis 1 — Distribution (ZT-3 same-distribution conjunct) | Text-only mode (provider-claude-cli text-only sustained) → 0 `audit:tool_audited` events → 退化變異估計；methodology artifact 非 architectural drift |
| **R28** | 03-22 | 0 (mechanical Tier 4) | Axis 1 — Distribution | 與 R26 同 text-only mode methodology artifact (sustained)；pool 維持 9 datapoint baseline |
| **R30** (預估) | 03-23 | TBD (預期 0) | Axis 1 — Distribution (預期) | provider-claude-cli text-only sustained 第三輪；預期 EXCLUDED |

### §2.3 DSS-CY22-§2-A BABBAGE Alternative R5 Historic Strict (per MR-11)

**Verbatim 保留 per MR-11 dissent preservation**：

> BABBAGE alternative R5 historic strict view: cycle 03-5 multi-category fragmented σ data 透過 per-category σᵢ extraction (event-count vector hash identity per Rule #76 §76.7 + Rule #77 σ_regime caveat) **可得**；strict-historic-inclusion view 主張 R5 data 若 pre-aggregation test (R3-§2-D ZT-3 same-distribution conjunct) PASSES 則應該 included。多數 view (R3-§2-C 22/1) elected 對稱第一類 EXCLUSION，理由為 pre-Rule-#77 era classification (Axis 2) 為 non-deterministic regime；BABBAGE dissent preserved 供未來 pre-aggregation methodology 演進時 re-evaluation。

**未來 re-evaluation 路徑**: 若 pre-aggregation methodology 演進至能 enable Rule #77 era retroactive classification (例如 per-category σᵢ retrofit)，R5 inclusion 可per Master directive 重新考慮。在此之前，R5 EXCLUDED 第一類。

---

## §3 雙軸排除分類學 (LINNAEUS R3-§2-C codification)

| Axis | Failure mode | R26 case | R5 case | R28+ case (預估) |
|------|--------------|:--------:|:-------:|:-------:|
| **Axis 1 — Distribution** (ZT-3 same-distribution conjunct) | Type-2 evidence (zero-variance methodology artifact under suppressed function-calling) | ✗ FAILS (text-only σ=0；0 `audit:tool_audited` events) | n/a (per-category σᵢ > 0 但 mixed methodology) | ✗ FAILS (text-only sustained) |
| **Axis 2 — Classification** (Rule #77 σ_regime methodology era) | Pre-Rule-#77 fragmented；no unified composition_index σ available | n/a (R26 = post-Rule-#77 composition_index) | ✗ FAILS (cycle 03-5 早於 Rule #77 8 cycles) | n/a (post-Rule-#77) |

**未來 pool eligibility 規則**: 兩軸皆 clear (ZT-3 distribution + Rule #77 era classification) 為 inclusion 必要條件。單軸 fail → 第一類 exclusion。

---

## §4 Rule #72 Path C Trigger 驗證

per Rule #72 path C 定義：post-feature-stack-extension recalibration 在 emit-source surface materially changes 時觸發。

| Cycle | Emit-source 計數 | Stack increment |
|:-----:|:-----------------:|---|
| 03-13 | 2 | Plan51 cap-bus + Plan52 pushInput (legacy baseline reference) |
| 03-17 | 3 | + Plan54 AC-9 |
| 03-18 | 4 | + Plan56 D-30-4 |
| 03-19 | 5 | + Plan57 D-30-5 (originally runner-level → cycle 03-21 plugin form refactor) |
| 03-21 | 6 | + Plan58 Mesh |
| **03-22** | **7** | + **Plan59 API Runtime (Class I substantive per R3-§2-B)** |

**7-component / 6 emit-source 增量** vs cycle 03-13 legacy 為**強烈 material**。Path C trigger 驗證。Re-calibration 已 overdue (應 cycle 03-19 觸發但延期；cycle 03-22 first execution 補上)。

---

## §5 R3-§2-E STRIKE — Trial Arithmetic Factor-of-13 Scaling Artifact

per R3-§2-E 23/0 UNANIMOUS：R1 §6.1 trial arithmetic σ_pool ≈ 0.000753 為 **factor-of-13 scaling artifact** (非權威)。Plan50 50 LOC binding exact form 輸出 0.000060 為 canonical 數值權威。Trial arithmetic 不取代 binding exact form。

**理由**: R1 trial arithmetic 用 coarse approximation；R3 audit 識別 scaling 差異；Test team 重執行 binding exact form 產生 0.000060 (factor 12.55× 比 trial arithmetic 緊)。Plan50 50 LOC 為 canonical。

**Forward-binding 意涵**: 未來 SPC 重校準 cycles MUST 用 Plan50 50 LOC binding exact form (或 future Master directive equivalent canonical)；trial arithmetic 不允許 canonical baseline establishment。

---

## §6 Forward-Binding 生效範圍

- **Cycle 03-23+ all W2 rounds (R30, R32, R34, ...)** : 用新 μ ± 3σ 評估
- **Cycle 03-22 W2-R28**: 用 legacy [0.01954, 0.02420] 評估 (Phase-I discipline；per O2 §4.2)
- **Legacy [0.01954, 0.02420] 限值**: cycle 03-22 R4 close 後 **RETIRED** (Batch 19 Item #5 ratified)

---

## §7 合規 Attestation

| Constraint | Status | Evidence |
|------------|:------:|----------|
| MR-1~MR-13 | ✅ PASS | MR-9 Plan50 50 LOC canonical 不 WAIVE / MR-11 DSS-CY22-§2-A BABBAGE preserved verbatim §2.3 / MR-12 forward-only (legacy retired post-R4) |
| ZT-1 / ZT-2 / ZT-3 | ✅ PASS | endpoint 10/0/0★ unchanged；Axis 1 ZT-3 same-distribution conjunct enforced (R26+R28 EXCLUDED) |
| Rule #72 path C | ✅ first execution | post-feature-stack-extension recalibration 觸發 + 執行 |
| Rule #76 §76.7 caveat | ✅ applies | composition_index σ rule-deterministic；R5 EXCLUDED per Axis 2；R26+R28 EXCLUDED per Axis 1 |
| Rule #77 σ_regime | ✅ binding | unified composition_index post-cycle 03-13 baseline |
| Master directive cycle 03-23 §6.4 (本 backfill) | ✅ ratified | Master letter cycle 03-23 §6.4 sign-off 2026-05-04 |
| Tenet #10 NC PENDING per MR-5 hard | ✅ unchanged | Phase 6 7/7 完工 cycle 03-23；Tenet #10 升 COMPLIANT cycle 03-24 |

---

## §8 Activation 紀錄

- **Effective**: 2026-05-04 (cycle 03-22 R4 close + Master directive cycle 03-23 §6.4 backfill)
- **Forward-binding**: cycle 03-23+ all W2 rounds
- **Cycle 03-22 W2-R28**: legacy [0.01954, 0.02420] applies (legacy 下最後一輪)
- **Cycle 03-23 W2-R30**: new μ ± 3σ 首次應用
- **Future re-calibration**: Rule #72 path C trigger 或 Master directive

---

## §9 Cross-Reference Index

- `Calibration_Reports/17_Phase3_Baseline_R10_R11.md` — legacy baseline cycle 03-12
- `Calibration_Reports/21_WIENER_Thresholds_Hypothesis.md` — adjacent calibration
- `Reference/baseline_rules.md` — Rule #72 / #76 §76.7 / #77 binding rules
- `Reference/16_R4_Folder_Convention_Discipline.md` + amendment_cycle03-20.md — coordinator G5 防線 4 BINDING canonical doc creation check (本 doc 為 scope expansion 後第一個 SPC baseline canonical case)
- claude research/research record/cycle03-22/deliver/O2_W2_R28_verification_final.md — R-team R4 deliverable source
- claude research/research record/cycle03-22/deliver/Master_Ratification/Master_Confirmation.md §3.4 — Master Ratification source
- claude research/research input/master_letters/cycle03-23/Master_letter.md §6.4 — backfill directive source
- test research/cycle03-22/deliverables/test_report_w2r28.md — Test team 權威數值 source

---

*SPC 重校準首次執行 — Cycle 03-22 — BINDING — 由 cycle 03-23 R0 backfill per Master directive §6.4*
*9-datapoint pool (R10/R11/R12/R14/R15/R18/R20/R22/R24) per R3-§2-A 22/1 super-majority ratified*
*R5+R26+R28 對稱第一類 EXCLUDED per R3-§2-C 雙軸分類學*
*New μ=0.023793 / σ_pool=0.000060 / UCL=0.023972 / LCL=0.023614 — Plan50 50 LOC canonical*
*Forward-binding cycle 03-23+ baseline；legacy [0.01954, 0.02420] cycle 03-22 R4 close 後 RETIRED*
*DSS-CY22-§2-A BABBAGE alternative R5 historic strict 保留 verbatim per MR-11*
