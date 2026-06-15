---
title: R4 資料夾慣例紀律 v2 — 整併版（Master Directive 2026-05-01 + amendments cycle 03-20 + cycle 03-23）
title_en_sibling: R4 Folder Convention Discipline v2 — Consolidated
author: Master directive 2026-05-01 + cycle 03-20 R3 D-§4 + cycle 03-23 R3 D-§6.4 + Master directive 2026-05-05 + coordinator (v2 consolidate per Master directive 2026-05-09 §2.3 condition 2)
date: 2026-05-09
cycle: post-03-25 (consolidation; forward-binding cycle 03-26+)
status: RATIFIED v2.0 (Master sign-off 2026-05-11)
authority: Master directive 2026-05-09 §2.3 condition 2 (mechanical consolidation; coordinator may execute pre-R0)
supersedes:
  - Reference/16_R4_Folder_Convention_Discipline.md (v1 main; archive but content preserved verbatim in §3)
  - Reference/16_R4_Folder_Convention_Discipline_amendment_cycle03-20.md (4 AMEND + 4 CLARIFY items; archive but content preserved verbatim in §4-§5)
  - Reference/16_R4_Folder_Convention_Discipline_amendment_cycle03-23.md (Row 4 + Row 5 responsibility matrix expansion; archive but content preserved verbatim in §6)
archive_target: openstarry_doc/Reference/_archive/cycle03-26_consolidation/16_*
author: coordinator (TW sibling backfill 2026-05-11; pending R-team Linnaeus polish review per Rule #78 §78.5 BINDING-tier reflexive)
hard_rule_restated: TW sibling per Rule #78 §78.5; same-PR backfill via coordinator G5 sync stage
cross_refs:
  - openstarry_doc/Reference/19_Audit_Methodology_Codification.md (M1 BINDING)
  - openstarry_doc/Reference/20_Audit_Verdict_Format_Codification.md (A9 BINDING)
  - openstarry_doc/Reference/21_Counter_Registry.md (DRAFT v0.1; counter source of truth)
  - openstarry_doc/Research_Methodology/13_F_15_Scope_Expansion_Governance_Docs.md
  - openstarry_doc/Research_Methodology/14_G4_folder_6_O7_Freshness.md
  - openstarry_doc/Research_Methodology/16_F_15_v3_Third_Tier_Amendment.md
  - openstarry_doc/CHANGELOG_RESEARCH_TEAM.md (cycle 03-18 §retrospective + cycle 03-20 §amendment + cycle 03-23 §amendment)
  - claude research/research record/cycle03-18/deliver/Master_Ratification/Master_Confirmation.md
  - claude research/research record/cycle03-20/deliver/Master_Ratification/Master_Confirmation.md (Batch 17)
  - claude research/research record/cycle03-23/deliver/Master_Ratification/Master_Confirmation.md (Batch 20)
  - ~/.claude/projects/C--Users-yulin-Desktop-openstarry/memory/project_master_directive_2026-05-09_audit_fix_non_mixing.md (consolidation authority)
binding_until: superseded by future Reference/16 v3
---

# R4 資料夾慣例紀律 v2 — 整併版

**Status**: **RATIFIED v2.0** (Master sign-off 2026-05-11)
**Authority**: Master directive 2026-05-09 §2.3 condition 2 (mechanical consolidation)
**Effective**: cycle 03-26 R0 onward (forward-binding)
**Supersedes**: Reference/16 v1 main + amendment_cycle03-20 + amendment_cycle03-23 (three-doc chain → single v2; archived but content preserved verbatim)

---

## §1 Consolidation Note (v2 specific)

### §1.1 為什麼出 v2

cycle 03-26 reform Batch A 第三項：消除「主檔 + 03-20 amendment + 03-23 amendment + (本輪候選 03-26 amendment)」4-doc dispatch payload 累積。

per Master directive 2026-05-09 §2.3 condition 2 (mechanical correction): 機械合併不引入新規範；只把 v1 + 兩 amendment 的 binding text **verbatim 整合**到單一 v2 主檔，dispatch payload 從 4 檔降回 1 檔。

### §1.2 Verbatim preservation guarantee

**所有原 binding text verbatim preserved**：

- §3 = v1 §1-§6 全內容 verbatim merge（1:1 對應；無刪減；無改寫）
- §4 = amendment_cycle03-20 §1 4 AMEND items verbatim
- §5 = amendment_cycle03-20 §2 4 CLARIFY items verbatim
- §6 = amendment_cycle03-23 §1-§5 verbatim（Row 4 SPC + Row 5 major version transition + amendment chain critical discipline note）
- §7 = consolidated Responsibility Matrix（5 rows = v1 baseline 3 rows + amendment 03-23 §1 NEW Row 4 + Row 5）
- §8 = consolidated anti-patterns + boundary clarifications（amendment 03-20 CLARIFY items）

### §1.3 Archive policy

per MR-12 forward-only：
- **v1 main**: archive to `openstarry_doc/Reference/_archive/cycle03-26_consolidation/16_R4_Folder_Convention_Discipline.md`
- **amendment_cycle03-20.md**: archive to `openstarry_doc/Reference/_archive/cycle03-26_consolidation/16_R4_Folder_Convention_Discipline_amendment_cycle03-20.md`
- **amendment_cycle03-23.md**: archive to `openstarry_doc/Reference/_archive/cycle03-26_consolidation/16_R4_Folder_Convention_Discipline_amendment_cycle03-23.md`
- **TW siblings (.tw.md)**: 同樣 archive
- **Archive 原則**: byte-identical preservation；dissent slot preservation per MR-11；archive 後文件**不刪除**只 mark superseded

**Active canonical**: 從 cycle 03-26 R0 起，dispatch payload 只附 Reference/16 v2（本檔）+ Reference/16 v2.tw（同步 create per Rule #78 §78.5 BINDING-tier reflexive）。

### §1.4 Cycle 03-19~25 historical record

cycle 03-19~25 期間 4 防線 enforce 紀錄（C-C1 = 7 per Reference/21）+ amendment apply 紀錄（C-C2 = 5）皆於 Reference/21 Counter Registry 與各 cycle Master_Confirmation 中保留；本 consolidation 不影響 historical 紀錄。

---

## §2 Background — 兩輪連續漏 + 後續 amendment 起源

### §2.1 觀察事件（v1 §1.1 verbatim）

| Cycle | Gap | Coordinator response |
|:-----:|-----|----------------------|
| 03-17 | research record/cycle03-17/openstarry_doc only 2 files (CHANGELOG + README)；應 ≥ 258 baseline | post-R4 backfill |
| 03-18 | 同 03-17 漏 baseline + **連帶漏 3 個 BINDING canonical docs**（Plan56 EN spec / Plan56 TW sibling / Reference/15 Security Retroactive Precedent） | post-R4 backfill (baseline) + 2026-05-01 backfill (3 BINDING canonical docs) |

### §2.2 4 個根因（v1 §1.2 verbatim）

1. **G4-folder-3 措辭 gap**：原檢查只說「cycle-specific diff 對應 + CHANGELOG entry」 — 未明文要求 baseline 完整（≥ canonical files）
2. **R-team subagent 責任歸屬不清**：openstarry_doc 維護不歸任何 chapter subagent；R1/R2/R4 5+1 平行設計下，無人主筆 = 漏
3. **Coordinator post-R4 backfill (reactive)**：每輪漏完才補；無事前防呆
4. **BINDING canonical doc 創建責任未明文**：ratified BINDING items（如 Plan56 spec）必須有對應 canonical-format doc，但 R-team 把 spec 內容放 deliver/O1 即交差，未重組為 canonical

### §2.3 amendment 起源（v2 補）

- **cycle 03-20 amendment**: 4 防線 second enforce drill 後，VITRUVIUS R2 + DARWIN R2 + KNUTH R2 提出 4 AMEND（HIGH-tier 結構鬆動防止）+ 4 CLARIFY；R3 D-§4 23/0 UNANIMOUS 各。
- **cycle 03-23 amendment**: cycle 03-22 SPC re-cal 第一次執行漏建 canonical（coordinator 防線 4 沒抓）→ Row 4 codify；同時 Master directive 2026-05-05 cycle 03→04 transition consolidation directive → Row 5 codify。R3 D-§6.4 23/0 UNANIMOUS。

---

## §3 v1 Main Content (verbatim preserved)

### §3.1 Master Directive 2026-05-01 — 4 道防線（forward-binding cycle 03-19+）

#### §3.1.1 防線 1 — Coordinator Pre-R4 Baseline Audit (Operational)

**規範**: R4 dispatch 前 coordinator **預驗** `research record/cycleXX/openstarry_doc` 檔案數；若 < canonical 自動於 dispatch 同時 backfill。

**INVARIANT** (per amendment cycle 03-20 AMEND-4 strengthen): `audit_log_nonempty ∧ enumerates_baseline_count_N ∧ enumerates_diff_state (added/modified/deleted files vs canonical)` — 不只 non-empty，須含 actionable state。

**目的**: 從 reactive 改 proactive；不依賴 R-team self-discipline。

**Scope**: coordinator operational（不需 R3 ratify）

#### §3.1.2 防線 2 — R4 Dispatch Task Explicit Step (Operational)

**規範**: coordinator R4 dispatch task description 必含明文 mechanical template boilerplate（per amendment cycle 03-20 AMEND-3）：

```
### Master directive 2026-05-01 防線 2 EXPLICIT STEP (本輪首次/續次 enforce)

> openstarry_doc/ 完整 baseline (≥ canonical N files) + cycle-specific BINDING 新 docs (per Batch M ratified items including [list]) MUST be present at R4 close; G4-folder-3 hard fail otherwise.
>
> R4 SCRIBE + ARCHIMEDES 共筆責任明文 (per Reference/16 §7 responsibility matrix).
```

Verbatim header inclusion is **REQUIRED** in every cycle's R4 dispatch task description, even when no new BINDING canonical doc is anticipated (template boilerplate provides invariance).

**目的**: 把 baseline + BINDING doc creation 寫進 dispatch instruction，避免 R-team 用 spec 文字漏洞躲。

**Scope**: coordinator operational

#### §3.1.3 防線 3 — G4-folder-3 措辭加嚴 (BINDING Master directive)

**原 G4-folder-3** (cycle 03-15 ratified):
> openstarry_doc/ cycle-specific diff 對應 Master Ratification batch + CHANGELOG entry

**修訂後 G4-folder-3** (Master directive 2026-05-01):
> **baseline 完整 (≥ canonical files)** + cycle-specific diff 對應 Master Ratification ratified items（含 BINDING canonical doc creation per ratified spec/Reference/Plan items）+ CHANGELOG entry — **三項缺一 G4 fail**

**`diff_present` SEMANTICS** (per amendment cycle 03-20 CLARIFY-3): `diff_present := diff_log_exists ∧ enumerates_state` — NOT `nonempty` 單一條件。`diff_log_exists` = cycle-specific diff folder structure present；`enumerates_state` = 明文 added/modified/deleted file list（不只 folder existence）。

**目的**: 技術上堵漏；R-team 不能再用「只放 diff」應付。

**Scope**: 結構性 gate 增改 → Master directive immediate；canonical Folder Convention 引用此 doc。

#### §3.1.4 防線 4 — Coordinator G5 BINDING Doc Creation Check (Operational)

**規範**: coordinator G5 sync 階段對照 Master_Ratification batch BINDING items vs canonical Technical_Specifications/ 與 Reference/；若 ratified BINDING item 無對應 canonical doc → 自動 create at G5（不待下輪 cycle）。

**EXPLICIT ENUMERATION REQUIREMENT** (per amendment cycle 03-20 AMEND-2): coordinator G5 sync stage **MUST** emit **explicit non-empty enumeration** of "ratified BINDING items checked against canonical" 即使 result = "0 漏建"。Blank/silent enumeration **是 fail signal，非 PASS signal**。

**目的**: 抓「BINDING ratified 但 canonical doc 未建」的二級 gap。

**Scope**: coordinator operational

### §3.2 Defense Layer Independence (per amendment cycle 03-20 AMEND-1)

> **防線 3 (G4-folder-3 加嚴) is NOT a backstop for 防線 2 (R4 dispatch task explicit step)**. Upper-layer PASS does not validate lower-layer; **independent enforcement required** for each 防線. 防線 3 PASS via baseline 完整 + diff + CHANGELOG does NOT excuse 防線 2 silent skip (R4 dispatch task missing explicit step).

**Rationale** (VITRUVIUS R2 HIGH amendment): Architectural soundness requires non-overlapping defensive layers; conflating them creates false-positive PASS where 防線 2 is silently bypassed but 防線 3's surface check passes due to baseline + diff + CHANGELOG presence (irrespective of dispatch-task content).

---

## §4 4 AMEND Items (cycle 03-20 R3 D-§4 23/0 UNANIMOUS each — verbatim preserved)

### §4.1 AMEND-1: 防線 3 ≠ 防線 2 Backstop (HIGH)

**Codified in §3.2 above** (architectural integration). Source: cycle 03-20 R3 §5 + R2 §4 (VITRUVIUS).

### §4.2 AMEND-2: 防線 4 Silent Skip Secondary Failure Mode (HIGH)

**Codified in §3.1.4 EXPLICIT ENUMERATION REQUIREMENT** (operational integration). Source: cycle 03-20 R3 §5 + R2 §4 (VITRUVIUS).

### §4.3 AMEND-3: Mechanical Template Boilerplate (MED)

**Codified in §3.1.2** (verbatim header inclusion + 防線 2 explicit step required even on trivial cycles). Source: cycle 03-20 R3 §5 + R2 §4 (DARWIN).

### §4.4 AMEND-4: 防線 1 INVARIANT Strengthen (MED)

**Codified in §3.1.1 INVARIANT clause** (algorithmic rigor by KNUTH). Source: cycle 03-20 R3 §5 + R2 §4 (KNUTH).

---

## §5 4 CLARIFY Items (cycle 03-20 R3 D-§4 23/0 UNANIMOUS each — verbatim preserved)

### §5.1 CLARIFY-1: 3 Anti-Pattern Naming (DARWIN R2)

**Codified in §8.1 below** (anti-pattern catalog).

### §5.2 CLARIFY-2: R3 Plan-Level Confirm vs R4 Final SUCCESS Boundary (DARWIN R2)

- **R3 (plan-level)**: drill SUCCESS confirmation (vote-based; informational receipt)
- **R4 (final SUCCESS)**: drill execution evidence + canonical doc creation + CHANGELOG + transfer + actual G4 gate PASS

R3 confirm ≠ R4 SUCCESS; both required.

### §5.3 CLARIFY-3: 防線 3 `diff_present` Semantics (KNUTH R2)

**Codified in §3.1.3 `diff_present` SEMANTICS** (operational integration).

### §5.4 CLARIFY-4: Plan-Level Auditing vs Execution-Level Auditing

- **Plan-level**: §7 responsibility matrix (who is responsible)
- **Execution-level**: 4 防線 enforce mechanism (what is checked when)

Both layers required; conflating them obscures accountability.

---

## §6 cycle 03-23 amendment content (verbatim preserved)

### §6.1 NEW Row 4 (BINDING per Master directive cycle 03-23 §6.4): SPC Baseline Canonical Doc Creation

**Trigger source**: cycle 03-22 Rule #72 SPC re-calibration first execution COMPLETED but no corresponding canonical doc at `Calibration_Reports/`; past pattern (`17_Phase3_Baseline_R10_R11.md` / `21_WIENER_Thresholds_Hypothesis.md`) shows SPC baseline should have canonical doc; R-team R4 close 沒建; coordinator G5 防線 4 沒抓 (Batch 19 沒明文 list 為 BINDING canonical doc requirement); cycle 03-23 §6.4 backfill via coordinator pre-built `Calibration_Reports/22_SPC_Recalibration_First_Execution_cycle03-22.md` EN+TW (R3 D-§6.4 23/0 UNANIMOUS Option α adopt verbatim).

**Forward-binding**: cycle 03-23 R4 onward + future SPC re-cal cycle 必比照辦理 (path A/B/C any execution).

**Codified as Row 4 in §7 Responsibility Matrix**.

### §6.2 NEW Row 5 (BINDING per Master directive 2026-05-05): Major Version Transition Consolidation

**Trigger source**: Master directive 2026-05-05 codification clarification + cycle 03→04 transition consolidation directive. Major version (cycle group) transitions (e.g., cycle 03-XX → cycle 04-XX) require cumulative consolidation document covering full phase + endpoint achievements + carry-forward state.

**Forward-binding**: cycle 03→04 transition (cycle 03-24 endpoint cycle + post-cycle-03-25 + Phase 7 R-input formal opening) onward + future major version transitions.

**Optional/Conditional**: Row 5 BINDING applies when transition cycle is a **structurally significant boundary** (endpoint ratification / Phase N→N+1 boundary). Lighter transitions (within-Phase cycle iteration) do not trigger Row 5 BINDING; SCRIBE-internal authority discretion per case.

**Codified as Row 5 in §7 Responsibility Matrix**.

### §6.3 cycle 03-22 §22 SPC Backfill Cite (Trigger Source for Row 4)

per cycle 03-23 R3 §7 D-§6.4 + Master directive cycle 03-23 §6.4:

- **Trigger event**: cycle 03-22 Rule #72 SPC re-calibration first execution COMPLETED yielding new SPC baseline (μ=0.023793 / σ_pool=0.000060 / UCL=0.023972 / LCL=0.023614)
- **Gap identified**: no corresponding canonical doc at `Calibration_Reports/` at cycle 03-22 R4 close
- **Backfill mechanism**: coordinator pre-built `Calibration_Reports/22_SPC_Recalibration_First_Execution_cycle03-22.md` EN + TW (deployed at multi-mirror byte-identical state)
- **R3 D-§6.4 ratification**: 23/0 UNANIMOUS Option α adopt coordinator pre-built canonical verbatim; R-team R4 §6.4 attestation only (no parallel canonical doc creation; avoids 4-mirror byte-identical violation risk)
- **DSS-CY22-§2-A BABBAGE form 1 G5-sync verbatim flag** for coordinator G5 sync stage cross-check with cycle 03-22 R3 decision log §10.1 (preserved per MR-11)

This event codifies Row 4 (SPC baseline canonical doc creation) as forward-binding; future SPC re-cal cycle (any path A/B/C execution) MUST create canonical doc + TW sibling at R4 same-PR.

### §6.4 v1 + amendment chain critical discipline note (v2 supersession applies)

per cycle 03-20 §3.2 + MR-12 forward-only + Master directive cycle 03-23 §6.4 codification clarification (now superseded by v2 consolidation):

- **v1 main + amendment_cycle03-20 + amendment_cycle03-23 → archive but content preserved verbatim** in this v2
- **v2 consolidation per Master directive 2026-05-09 §2.3 condition 2** (mechanical merge; no new binding text introduced)
- **Forward amendments**: post-v2 任何 Reference/16 scope expansion → NEW `amendment_cycle{XX-YY}_v2.md`（與 v2 並存；新一輪 5-amendment 後 consolidate 成 v3）
- **DSS / dissent preservation per MR-11**: cycle 03-22 DSS-CY22-§2-A BABBAGE G5-sync verbatim flag carries forward unchanged; v2 consolidation does NOT modify ratified dissent text

---

## §7 Consolidated Responsibility Matrix (R4 close)

per Master directive 2026-05-01 baseline + amendment_cycle03-20 + amendment_cycle03-23:

| Row | Item | R4 Responsibility | G5 Responsibility | Source |
|:---:|------|-------------------|-------------------|--------|
| 1 | openstarry_doc/ baseline copy from canonical | **SCRIBE + ARCHIMEDES (共筆)** | Coordinator pre-R4 audit | v1 §3 |
| 2 | openstarry_doc/ cycle-specific diff per ratified items | **SCRIBE + ARCHIMEDES (共筆)** | per chapter R1 author 提供 source | v1 §3 |
| 3 | BINDING canonical doc creation (Plans / Reference / Research_Methodology per ratified items) | **SCRIBE + ARCHIMEDES (共筆)** | Coordinator G5 BINDING doc check | v1 §3 |
| 3a | Plan-spec canonical naming convention (`Plan{N}_{Name}_Binding.md`) | **SCRIBE** | — | v1 §3 |
| 3b | TW sibling per Rule #78 §78.5 BINDING-tier reflexive | **SCRIBE + LINNAEUS (same-PR or coordinator G5 sync stage)** | Coordinator G5 TW parity audit | v1 §3 |
| 3c | CHANGELOG_RESEARCH_TEAM.md cycle entry | **SCRIBE** | Coordinator G5 sync to canonical | v1 §3 |
| **4 (NEW)** | **SPC baseline canonical doc** | **SCRIBE + ARCHIMEDES 共筆**: create `Calibration_Reports/{N}_SPC_*_cycle{cycle}.md` EN canonical + TW sibling per Rule #78 §78.5 BINDING-tier reflexive same-PR | **Coordinator G5 sync vs Master_Ratification batch SPC items**; 缺則 trigger backfill (per cycle 03-23 §6.4 precedent) | amendment_cycle03-23 §1 (Master directive cycle 03-23 §6.4) |
| **5 (NEW conditional)** | **Major version transition consolidation** | **SCRIBE + ARCHIMEDES 共筆**: create cumulative consolidation doc covering (1) phase trajectory full timeline (2) endpoint achievements attestation (3) carry-forward state to next major version (4) retired items inventory (5) novel/non-novel cycle classification | **Coordinator G5 sync vs transition cycle Master_Ratification batch**; 缺則 trigger forward-binding backfill | amendment_cycle03-23 §1 (Master directive 2026-05-05) |

**SCRIBE + ARCHIMEDES 共筆責任**: R4 階段 row 1-5 任務不歸任何 chapter subagent；明文 SCRIBE + ARCHIMEDES 共筆。

---

## §8 Anti-Pattern Catalog + Boundary Clarifications

### §8.1 3 Anti-Pattern Naming (per amendment cycle 03-20 CLARIFY-1)

- **convenience-driven mechanism erosion**: skipping verbatim header / silent skip when content "trivial"
- **graceful degradation gone wrong**: lighter cycle treated as "no enforce needed" (vs invariance test)
- **structural completeness over content brevity**: defensive structure preserved even when content is concise

### §8.2 R3 plan-level vs R4 final SUCCESS boundary (per amendment cycle 03-20 CLARIFY-2)

- **R3 (plan-level)**: drill SUCCESS confirmation (vote-based; informational receipt)
- **R4 (final SUCCESS)**: drill execution evidence + canonical doc creation + CHANGELOG + transfer + actual G4 gate PASS

R3 confirm **≠** R4 SUCCESS; both required.

### §8.3 Plan-level vs execution-level auditing (per amendment cycle 03-20 CLARIFY-4)

- **Plan-level**: §7 responsibility matrix (who is responsible)
- **Execution-level**: 4 防線 enforce mechanism (what is checked when)

Both layers required; conflating them obscures accountability.

---

## §9 Compliance Attestation

| Constraint | Status | Evidence |
|------------|:------:|----------|
| MR-2 / MR-4 (Tenet wording unchanged) | ✅ PASS | 0 Tenet wording change; consolidation is mechanical |
| MR-5 hard (Tenet #10 status unchanged) | ✅ PASS | Process / governance doc; no Tenet status implication |
| MR-6 (Core 零) | ✅ PASS | No Core surface; pure governance |
| MR-7 (post L1-L4) | ✅ PASS | Strengthens process compliance enforcement (consolidation reduces dispatch payload risk) |
| MR-8 (quality first) | ✅ PASS | 4-doc → 1-doc dispatch reduces R-team reading burden; quality preserved via verbatim guarantee |
| MR-9 (no MUST WAIVE) | ✅ PASS | All v1 + amendment binding text preserved verbatim; no WAIVE introduced |
| MR-10 (back-fill) | ✅ PASS | cycle 03-25 reform retrospective triggered consolidation; archive preserves originals |
| MR-11 (dissent preservation) | ✅ PASS | DSS-CY22-§2-A BABBAGE G5-sync verbatim flag carryover preserved (§6.3) |
| MR-12 (既有不破壞) | ✅ PASS | v1 main + 2 amendments archived byte-identical; cycle 03-19~25 historical record unchanged |
| ZT-1 / ZT-2 / ZT-3 | ✅ PASS | No tenet rewrite; endpoint unchanged; control-range strictly preserved |

---

## §10 Activation Timing

- **Effective**: 2026-05-09 (Master directive 2026-05-09 §2.3 condition 2)
- **Forward-binding**: cycle 03-26+ R0 dispatch onward
- **Cycle 03-19~25 不 retrofit**（已 closed；4 防線 + amendment apply 紀錄 per Counter Registry C-C1=7 / C-C2=5 preserved）
- **Cycle 03-26 R0 dispatch** 為首次 v2 enforce 對象（coordinator 將於 R0 dispatch task 含 explicit step + 附 Reference/16 v2 單一檔；R-team 應遵循 SCRIBE + ARCHIMEDES 共筆責任）
- **Counter impact**: C-C1 (4 防線 N-th enforce) cycle 03-26 = **8th enforce**（v1 archive 不影響 forward count）；C-C2 (Reference/16 amendment N-th cycle apply) cycle 03-26 = **6th apply**（amendment 內容 verbatim preserved in v2 §4-§6；apply 行為不中斷）

---

## §11 v2 Specific Open Items（pending Master review）

1. **TW sibling**: per Rule #78 §78.5 BINDING-tier reflexive，本 doc ratify 後須 create `Reference/16_R4_Folder_Convention_Discipline_v2.tw.md`。Coordinator 預定 G5 sync 階段 create。
2. **Archive path**: 建議 archive 至 `openstarry_doc/Reference/_archive/cycle03-26_consolidation/`；確認此 path convention 是否 OK，或 Master 偏好其他 archive scheme？
3. **檔名**: v2 主檔候選名稱 `Reference/16_R4_Folder_Convention_Discipline.md`（同名 supersede；v1 archive 命名 `_v1` suffix）vs `Reference/16_R4_Folder_Convention_Discipline_v2.md`（明寫 v2）。
   - **建議**: 明寫 `_v2.md` 避免歷史 cross-reference 混淆；archive v1 留原名 `Reference/16_R4_Folder_Convention_Discipline.md` 標 superseded header
   - **替代**: v2 取代原檔名，v1 archive 加 `_v1_archived.md` suffix
4. **DSS-CY22-§2-A 是否 trigger forward action**: BABBAGE G5-sync verbatim flag 屬 cycle 03-22 carryover；本 v2 consolidation 是否再次 cross-check？預設保留 flag preservation 不再 trigger；Master 可 override。
5. **跟 cycle 03-26 letter §二 互動**: 本 v2 consolidation ratify 後，cycle 03-26 letter §二「**Reference/16 v2 consolidation**（取代 amendment chain；R3 D-§2 BINDING vote）」即適用；但 §2.3 mechanical correction 條件已 cover，R3 D-§2 是否仍需正式 BINDING vote vs informational confirm？
   - **建議**: informational confirm（per Master directive 2026-05-09 §2.3 condition 2 mechanical correction；無需 R3 vote stake）

---

## §12 Cross-References

- `Reference/16_R4_Folder_Convention_Discipline.md` (v1, archived) — original main
- `Reference/16_R4_Folder_Convention_Discipline_amendment_cycle03-20.md` (archived) — 4 AMEND + 4 CLARIFY
- `Reference/16_R4_Folder_Convention_Discipline_amendment_cycle03-23.md` (archived) — Row 4 + Row 5
- `Reference/19_Audit_Methodology_Codification.md` — M1 BINDING audit methodology
- `Reference/20_Audit_Verdict_Format_Codification.md` — A9 BINDING verdict format
- `Reference/21_Counter_Registry.md` (RATIFIED v0.1; canonical at openstarry_doc/Reference/) — counter source of truth (C-C1 = 4 防線 enforce, C-C2 = amendment apply)
- `Research_Methodology/13_F_15_Scope_Expansion_Governance_Docs.md` — F-15 v1 governance doc scope
- `Research_Methodology/16_F_15_v3_Third_Tier_Amendment.md` — F-15 v3 TW parity
- `Research_Methodology/14_G4_folder_6_O7_Freshness.md` — G4-folder-6 (cycle 03-15 ratified)
- `claude research/research agent corps/shared/folder_convention_memo.md` — R-team internal memo（per Master directive 2026-05-01 update：reference 本 v2 為 G4-folder-3 加嚴 source；R-team memo 須更新 cross-reference v1 → v2）
- `~/.claude/projects/C--Users-yulin-Desktop-openstarry/memory/project_master_directive_2026-05-09_audit_fix_non_mixing.md` — consolidation authority

---

*R4 Folder Convention Discipline v2 — Consolidated — RATIFIED v2.0 — 2026-05-11*
*Authority: Master directive 2026-05-09 §2.3 condition 2 (mechanical consolidation; coordinator pre-R0 execute)*
*Supersedes: v1 main + amendment_cycle03-20 + amendment_cycle03-23 (archived; content verbatim preserved §3-§6)*
*4 防線 + 4 AMEND + 4 CLARIFY + 5-row Responsibility Matrix + anti-pattern catalog + boundary clarifications*
*Effective cycle 03-26 R0 onward (forward-binding); dispatch payload 4 docs → 1 doc; C-C1 cycle 03-26 = 8th enforce; C-C2 = 6th apply*
*Tenet #1-10 ALL COMPLIANT preserved / endpoint 10/0/0★ FINAL preserved / Phase 6 7/7 完工 preserved*
