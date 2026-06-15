---
title: R4 Folder Convention Discipline — Master Directive 2026-05-01
author: Master directive 2026-05-01 + coordinator (canonical doc create)
date: 2026-05-01
cycle: post-03-18 (Master directive 2026-05-01；forward-binding cycle 03-19+)
status: BINDING
authority: Master directive 2026-05-01
supersedes: extends G4-folder-3 (cycle 03-15 ratified；本 directive 加嚴 措辭) + G4-folder-7 (cycle 03-17 ratified；持續)
cross_refs:
  - openstarry_doc/Research_Methodology/13_F_15_Scope_Expansion_Governance_Docs.md
  - openstarry_doc/Research_Methodology/14_G4_folder_6_O7_Freshness.md
  - openstarry_doc/Research_Methodology/16_F_15_v3_Third_Tier_Amendment.md
  - openstarry_doc/CHANGELOG_RESEARCH_TEAM.md (cycle 03-18 §retrospective)
  - claude research/research record/cycle03-18/deliver/Master_Ratification/Master_Confirmation.md
binding_until: superseded by future Master directive
---

# R4 Folder Convention Discipline — Master Directive 2026-05-01

**Status**: BINDING（forward-binding cycle 03-19+）
**Authority**: Master directive 2026-05-01
**Trigger**: cycle 03-17 + cycle 03-18 連兩輪 R-team 漏 openstarry_doc baseline mirror + 連帶漏 BINDING canonical docs（cycle 03-18 漏 Plan56 EN/TW + Reference/15）

---

## §1 Background — 兩輪連續漏的根因

### §1.1 觀察事件

| Cycle | Gap | Coordinator response |
|:-----:|-----|----------------------|
| 03-17 | research record/cycle03-17/openstarry_doc only 2 files (CHANGELOG + README)；應 ≥ 258 baseline | post-R4 backfill |
| 03-18 | 同 03-17 漏 baseline + **連帶漏 3 個 BINDING canonical docs**（Plan56 EN spec / Plan56 TW sibling / Reference/15 Security Retroactive Precedent） | post-R4 backfill (baseline) + 2026-05-01 backfill (3 BINDING canonical docs) |

### §1.2 4 個根因

1. **G4-folder-3 措辭 gap**：原檢查只說「cycle-specific diff 對應 + CHANGELOG entry」 — 未明文要求 baseline 完整（≥ canonical files）
2. **R-team subagent 責任歸屬不清**：openstarry_doc 維護不歸任何 chapter subagent；R1/R2/R4 5+1 平行設計下，無人主筆 = 漏
3. **Coordinator post-R4 backfill (reactive)**：每輪漏完才補；無事前防呆
4. **BINDING canonical doc 創建責任未明文**：ratified BINDING items（如 Plan56 spec）必須有對應 canonical-format doc，但 R-team 把 spec 內容放 deliver/O1 即交差，未重組為 canonical

---

## §2 Master Directive 2026-05-01 — 4 道防線（forward-binding cycle 03-19+）

### §2.1 防線 1 — Coordinator Pre-R4 Baseline Audit (Operational)

**規範**: R4 dispatch 前 coordinator **預驗** `research record/cycleXX/openstarry_doc` 檔案數；若 < canonical 自動於 dispatch 同時 backfill。

**目的**: 從 reactive 改 proactive；不依賴 R-team self-discipline。

**Scope**: coordinator operational（不需 R3 ratify）

### §2.2 防線 2 — R4 Dispatch Task Explicit Step (Operational)

**規範**: coordinator R4 dispatch task description 必含明文：

> openstarry_doc/ 完整 baseline (≥ canonical) + cycle-specific BINDING 新 docs (per Master_Ratification batch ratified items) **MUST** be present at R4 close; G4-folder-3 hard fail otherwise.

**目的**: 把 baseline + BINDING doc creation 寫進 dispatch instruction，避免 R-team 用 spec 文字漏洞躲。

**Scope**: coordinator operational

### §2.3 防線 3 — G4-folder-3 措辭加嚴 (BINDING Master directive)

**原 G4-folder-3** (cycle 03-15 ratified):
> openstarry_doc/ cycle-specific diff 對應 Master Ratification batch + CHANGELOG entry

**修訂後 G4-folder-3** (Master directive 2026-05-01):
> **baseline 完整 (≥ canonical files)** + cycle-specific diff 對應 Master Ratification ratified items（含 BINDING canonical doc creation per ratified spec/Reference/Plan items）+ CHANGELOG entry — **三項缺一 G4 fail**

**目的**: 技術上堵漏；R-team 不能再用「只放 diff」應付。

**Scope**: 結構性 gate 增改 → Master directive immediate；canonical Folder Convention 引用此 doc。

### §2.4 防線 4 — Coordinator G5 BINDING Doc Creation Check (Operational)

**規範**: coordinator G5 sync 階段對照 Master_Ratification batch BINDING items vs canonical Technical_Specifications/ 與 Reference/；若 ratified BINDING item 無對應 canonical doc → 自動 create at G5（不待下輪 cycle）。

**目的**: 抓「BINDING ratified 但 canonical doc 未建」的二級 gap。

**Scope**: coordinator operational

---

## §3 Responsibility Matrix (R4 close)

per Master directive 2026-05-01:

| Item | Primary owner | Backup |
|------|---------------|--------|
| openstarry_doc/ baseline copy from canonical | **SCRIBE + ARCHIMEDES (共筆)** | Coordinator pre-R4 audit |
| openstarry_doc/ cycle-specific diff per ratified items | **SCRIBE + ARCHIMEDES (共筆)** | per chapter R1 author 提供 source |
| BINDING canonical doc creation (per ratified Plan/Reference/spec items) | **SCRIBE + ARCHIMEDES (共筆)** | Coordinator G5 BINDING doc check |
| Plan-spec canonical naming convention (Plan{N}_{Name}_Binding.md) | **SCRIBE** | — |
| TW sibling per Rule #78 BINDING-tier | **SCRIBE + LINNAEUS** | Coordinator |
| CHANGELOG_RESEARCH_TEAM.md cycle entry | **SCRIBE** | Coordinator |

**SCRIBE + ARCHIMEDES 共筆責任**: 此 R4 階段任務不歸任何 chapter subagent；明文 SCRIBE + ARCHIMEDES 共筆。

---

## §4 Compliance Attestation

| Constraint | Status | Evidence |
|------------|:------:|----------|
| MR-2 / MR-4 (Tenet 措辭不變) | ✅ PASS | 0 Tenet wording change |
| MR-5 hard | ✅ PASS | Plan56 plugin layer；不動 Tenet #10 |
| MR-6 (Core 零) | ✅ PASS | 純 governance / process discipline doc |
| MR-7 (post L1-L4) | ✅ PASS | 強化 process compliance enforcement |
| MR-8 (quality first) | ✅ PASS | 4 道防線提升 R4 quality |
| MR-9 (no MUST WAIVE) | ✅ PASS | 強化 G4-folder-3；無 WAIVE |
| MR-10 | ✅ PASS | back-fill cycle 03-17/18 漏的 docs（per cycle 03-18 §retrospective）|
| MR-11 (dissent) | N/A | 0 dissent 因為是 Master directive immediate；無 R3 vote |
| MR-12 (既有不破壞) | ✅ PASS | forward-binding cycle 03-19+；既有 cycle 03-15/16/17/18 不 retrofit（已 closed）|
| ZT-1 / ZT-2 / ZT-3 | ✅ PASS | 無 tenet rewrite；endpoint 不變；control 加嚴 |

---

## §5 Cross-References

- `openstarry_doc/Research_Methodology/13_F_15_Scope_Expansion_Governance_Docs.md` — F-15 v1 governance doc scope
- `openstarry_doc/Research_Methodology/16_F_15_v3_Third_Tier_Amendment.md` — F-15 v3 TW parity
- `openstarry_doc/Research_Methodology/14_G4_folder_6_O7_Freshness.md` — G4-folder-6 (cycle 03-15 ratified)
- `claude research/research agent corps/shared/folder_convention_memo.md` — R-team internal memo（per Master directive 2026-05-01 update：reference 本 doc 為 G4-folder-3 加嚴 source）
- `openstarry_doc/CHANGELOG_RESEARCH_TEAM.md` cycle 03-18 §retrospective — 完整事件紀錄

---

## §6 Activation Timing

- **Effective**: 2026-05-01 (Master directive)
- **Forward-binding**: cycle 03-19+
- **Cycle 03-15/16/17/18 不 retrofit**（已 closed；不適用）
- **Cycle 03-19 R4 dispatch** 為首次 enforce 對象（coordinator 將於 R4 dispatch task 含 explicit step；R-team 應遵循 SCRIBE + ARCHIMEDES 共筆責任）

---

*R4 Folder Convention Discipline — BINDING — Master directive 2026-05-01*
*4 道防線：coordinator pre-R4 audit + R4 dispatch explicit step + G4-folder-3 加嚴 + G5 BINDING doc creation check*
*R4 SCRIBE + ARCHIMEDES 共筆責任明文；防 cycle 03-17/18 baseline + BINDING doc gap 重演*
*9/0/1★ ACTIVE preserved / Tenet #10 NC PENDING per MR-5 / endpoint 10/0/0★ unchanged*
