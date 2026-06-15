---
title: Reference/21 — 計數器登記簿（專案所有計數器的單一真相來源）
title_en_sibling: Reference/21 — Counter Registry (Single Source of Truth for All Project Counters)
date: 2026-05-09 (v0.1) / 2026-05-11 (v0.1.1 修訂)
status: RATIFIED v0.2 (Master Ratification Batch 24 sign-off 2026-05-13；substance §3.7 Counter Sunset Clause + C-S1-roll12 NEW counter + 整合 v0.1.4 mechanical patch；詳細條文見 EN 版 §3.7 + §2.1 C-S1-roll12)
authority: Master directive 2026-05-09 (audit/fix non-mixing binding §2.3 condition 2 mechanical correction; coordinator may execute pre-R0)
effective: cycle 03-26 R0 起 (v0.1) + §3.4/§3.5 cycle 03-27 close 起 (v0.1.1) + §3.4.1 cycle 03-26 retroactive reconcile 2026-05-12 起 (v0.1.2) + §3.6 mirror discipline single-target rule 2026-05-12 起 (v0.1.3)
amendment_history:
  - v0.1 → v0.1.1 (2026-05-11)：新增 §3.4 release tag 精確定義（解決 §75.X 第 5 次 flagging Dev=18 vs R-team=19）+ §3.5 hygiene-only cycle counter 累進規則（解決 C-C6/C-C7 解讀爭議 per cycle 03-27 R-team verification §7 F-CY27-§R-V-08）；既有 row 不修改 per MR-12 forward-only
  - v0.1.1 → v0.1.2 (2026-05-12)：新增 §3.4.1 retroactive reconcile after cycle 03-26 v0.57.3-alpha doc-only patch 補做 per Master directive 2026-05-12（cycle 03-26 release dispatch was missed → coordinator dispatch Dev task #190 補做 → release/cycle03-26_v0.57.3-alpha/ now exists → §3.4 release tag 充要條件 satisfied → C-C4 reconciled = 19 not 18; v0.57.3-alpha = 18th, v0.57.4-alpha = 19th）；§3.4 規則 unchanged（規則沒錯；v0.1.1 reconciled value 18 是基於當時的 silent-failure 缺席事實；v0.1.2 reconciles to true 19 after 補做）
  - v0.1.2 → v0.1.3 (2026-05-12)：新增 §3.6 Mirror Discipline Single-Target Rule after Master directive 2026-05-08 真正落地（Dev task #191 — `openstarry/docs/` 從 4 location 移除：agent_dev + cycle03-26 release + cycle03-27 release + test codebase；agent_dev pnpm build/test/purity PASS；sibling-naming-check.mjs retarget Rule #62 root-cause fix bundled）；single-target = canonical 鏡同步唯一目標 = `openstarry_doc/`，禁止 mirror 到 `openstarry/docs/canonical/`；典型 mirror count 6-9 locations
supersedes: scattered counter definitions across Plans, References, Calibration_Reports, Architecture_Documentation, CHANGELOG, and Master_Confirmation docs (these continue to exist but reference Reference/21 as canonical algorithm)
cross_refs:
  - openstarry_doc/Reference/10_Rule_75_Section_75_X_pnpm_build.md (§75.X enforce counter — first reconciled application; see §3.1)
  - openstarry_doc/Reference/16_R4_Folder_Convention_Discipline.md (and v2 consolidation; 4 防線 + amendment apply counters)
  - openstarry_doc/Reference/19_Audit_Methodology_Codification.md (M1 BINDING; methodology counters)
  - openstarry_doc/Reference/20_Audit_Verdict_Format_Codification.md (A9 BINDING; verdict cell counters)
  - openstarry_doc/CHANGELOG_RESEARCH_TEAM.md (cycle-by-cycle counter snapshots)
  - claude research/research record/cycle03-25/deliver/Master_Ratification/Master_Confirmation.md (Batch 22; latest counter snapshot)
  - C:/Users/yulin/.claude/projects/C--Users-yulin-Desktop-openstarry/memory/MEMORY.md (cycle 03-25 progress entry)
binding_until: superseded by future Reference/21 v2
author: coordinator (TW sibling backfill 2026-05-11; pending R-team Linnaeus polish review per Rule #78 §78.5 BINDING-tier reflexive)
hard_rule_restated: TW sibling per Rule #78 §78.5; same-PR backfill via coordinator G5 sync stage
---

# Reference/21 — 計數器登記簿（單一真相來源）

**Status**: **RATIFIED v0.2** (Master Ratification Batch 24 sign-off 2026-05-13；substance §3.7 Counter Sunset Clause + C-S1-roll12 NEW；詳見 EN 版完整條文)
**Authority**: Master directive 2026-05-09 §2.3 condition 2 (mechanical correction; coordinator pre-R0 execute)
**Effective**: cycle 03-26 R0 起 (v0.1) + §3.4/§3.5 cycle 03-27 close 起 (v0.1.1)
**Purpose**: 解 §75.X 計數三邊不一致問題；統一所有 project counter 的 canonical algorithm 與 source of truth

---

## §1 Background — 為什麼要訂這份 registry

### §1.1 觸發事件

cycle 03-25 R-team M3 spot-check verification ring 報告：

> **§75.X counter discrepancy**: Dev 17th / spec 12th / spot-check 4th — flagging NON-BLOCKING coordinator/Master reconcile recommended pre-cycle03-26

三邊用不同邏輯算同一個 counter，每輪 R3 + Master Confirmation 都要對一遍。本輪是公認觸發點，但**這是 symptom，不是 isolated bug**。

### §1.2 Symptom of larger problem

掃 canonical doc 發現至少 **25+ counter** 散在不同 doc / process：

- Streak counters（連續 N 輪）
- Cardinality counters（第 N 次發生）
- Cumulative counters（累積 N）
- State counters（當前 N）
- Status enums（status 值）
- Sequence counters（流水號）

**問題**：
1. 同一 counter 在不同 doc 用不同算法（例 §75.X 的 17/12/4）
2. 計算邏輯散在 R3 decision log / Master_Confirmation / progress notes，無 canonical source
3. 每輪 dispatch 須附「current N」snapshot，coordinator 與 R-team 容易 desync
4. Reset 條件不一致（有的 stale 不 reset，有的 reset 後忘記紀錄）

### §1.3 Resolution

本 registry = **唯一 canonical source of truth** for all project counters。每 counter 定義：

- **Name**: 唯一識別字
- **Type**: streak / cardinality / cumulative / state / status / sequence
- **Definition**: 算法（precise；無歧義）
- **Source of truth**: 哪個 doc / event 是 update trigger
- **Update trigger**: 何時 +1 / change
- **Reset condition**: 何時 reset / N/A
- **Current value**: cycle 03-25 close 時的快照（cycle 03-26 R0 起以本表為準）
- **First established**: 第一次出現的 cycle / event
- **Disambiguation**: 與哪些 counter 容易混淆 + 區別

所有其他 doc 引用 counter 時 **MUST** 引用本 registry name + 算法；不可重定義 counter。

---

## §2 Counter Registry Table

### §2.1 Streak Counters（連續 N，發生 break 即 reset）

| ID | Name | Definition | Source of Truth | Update Trigger | Reset Condition | Current Value (cycle 03-25 close) | First Established |
|:--:|------|------------|-----------------|----------------|-----------------|:----------------------------------:|:-----------------:|
| C-S1 | **HEALTHY consecutive cycles** | 連續 N cycles 滿足 9-metric 全 GREEN（S1-S9 per Master directive 2026-04-25 permanent-mode） | R4 close attestation per Master_Confirmation §healthy_metrics | R4 close + 9/9 GREEN cells PASS → +1 | 任一 RED → 0 (S-3 fallback triggered) | **13** (cycle 03-13~03-25) | cycle 03-13 |
| C-S2 | **6-mirror byte-identical sustained** | 連續 N cycles canonical / suggestion / release / research_version / test codebase / agent_dev 6 mirror byte-identical | Coordinator G5 sync verify | G5 PASS → +1 | Any mirror diff → 0 | **5** (cycle 03-21~25) | cycle 03-21 |
| C-S3 | **amendment_cycle03-20 md5 preservation** | 連續 N cycles `Reference/16_amendment_cycle03-20.md` md5 unchanged (canonical: `7ffd0f79cddecb2a764fd0a9fe8d1e3a`) | Coordinator G5 sync verify | G5 PASS + md5 match → +1 | md5 change → 0 | **5** (cycle 03-21~25) | cycle 03-21 |
| C-S4 | **F1/F2/F3 Popperian counter (zero)** | 連續 N cycles F1/F2/F3 各 metric 計數 = 0（Master directive 2026-05-03 5-point isolation v2 evidence collection metrics） | W2-Rn round close attestation | W2-Rn PASS + F1=F2=F3=0 → +1 | Any F1/F2/F3 > 0 → 0 | **5** (cycle 03-21~25; 15/15 zero milestone reached) | cycle 03-21 |
| C-S5 | **9-datapoint pool sustained** | Rule #72 SPC σ 9-datapoint pool 連續 N cycles 維持（不變動 datapoint set） | Master_Confirmation §SPC | W2-Rn excluded from pool (text-only sustained) → +1; pool 不變動 | Pool change (add/remove datapoint) → 0 | **4** (cycle 03-22~25; W2-R28/R30/R32/R34 全 EXCLUDED) | cycle 03-22 (W2-R28) |
| C-S6 | **R3 0/2 correction rounds** | 連續 N cycles R3 deliberation 0 correction rounds (S8 metric) | R3 close per R3_decision_log | R3 close + correction_rounds=0 → +1 | correction_rounds > 0 → 0 | **13** (cycle 03-13~25; 同 C-S1 trajectory) | cycle 03-13 |

### §2.2 Cardinality Counters（第 N 次發生）

| ID | Name | Definition | Source of Truth | Update Trigger | Reset Condition | Current Value (cycle 03-25 close) | First Established |
|:--:|------|------------|-----------------|----------------|-----------------|:----------------------------------:|:-----------------:|
| C-C1 | **4 防線 N-th enforce** | Master directive 2026-05-01 4 防線（coordinator pre-R4 audit + R4 dispatch explicit + G4-folder-3 加嚴 + G5 BINDING doc check）第 N 輪完整 enforce | R4 close attestation | Cycle R4 close + 4 防線 全 PASS → +1 | N/A (forward-only; no reset) | **7** (cycle 03-19, 20, 21, 22, 23, 24, 25) | cycle 03-19 |
| C-C2 | **Reference/16 amendment N-th cycle apply** | amendment_cycle03-20 + amendment_cycle03-23 第 N 輪在 R0/R4 dispatch 同附 + R4 attestation | R4 close attestation | Cycle R4 close + amendment 適用 PASS → +1 | N/A | **5** (cycle 03-21, 22, 23, 24, 25) | cycle 03-21 |
| C-C3 | **5-point isolation v2 N-th sustained** | Master directive 2026-05-03 5-point isolation guarantees v2 第 N 輪 sustained 使用（provider-claude-cli 路徑） | W2-Rn round close attestation | W2-Rn PASS + isolation 5-point 全 sustained → +1 | Any isolation point breached → 0 (treat as streak; 但 cycle 03-25 STRENGTHENED upgrade 不 reset) | **5 STRENGTHENED** (cycle 03-21~25; cycle 03-25 Point 5 升 explicit env-allowlist) | cycle 03-21 |
| C-C4 | **Rule #75 §75.X N-th enforce (canonical)** | Per Rule #75 §75.X §4.2: 任何 release tag (`v*.*.*-alpha` / `v*.*.*-beta` / `v*.*.*` GA) 自 v0.50.0-alpha 起的累計 enforce 次數 | Dev `delivery_report.md §pnpm_build_evidence` (canonical) | 每個 release tag (含 hotfix + doc-only) Dev 通過 §75.X 驗證 → +1 | N/A | **17** (v0.50.0-alpha 起累計：v0.50, 0.51, 0.51.1, 0.52, 0.53, 0.54, 0.54.1, 0.55, 0.55.1~5, 0.56, 0.57, 0.57.1, 0.57.2 = 17) | v0.50.0-alpha (cycle 03-15) |
| C-C4a | **§75.X per-cycle enforce** | Cycles where ≥1 release tag triggered §75.X enforce（不論 tag 數） | Master_Confirmation §release_tags 列表 | Cycle 含 ≥1 release tag → +1 | N/A | **12** (cycle 03-15, 16, 17, 18, 19, 20, 21, 22, 23, 24, 25 + cycle 03-14 v0.50.0-alpha pending = 12) | cycle 03-14 |
| C-C4b | **§75.X spot-check enforce** | Cycles where Master / R-team 對 §75.X 進行 spot-check verification | R-team M-tier audit ring or Master directive | Spot-check ring received → +1 | N/A | **4** (cycle 03-22, 03-23, 03-24, 03-25) | cycle 03-22 |
| C-C5 | **Rule #72 SPC re-cal N-th cycle application** | Rule #72 new SPC limits (μ=0.023793 / UCL=0.023972 / LCL=0.023614) 第 N 輪 cycle application | W2-Rn round close + Master_Confirmation §SPC | W2-Rn round 用 new limits → +1 | N/A | **3** (cycle 03-23 first / 03-24 second / 03-25 third) | cycle 03-23 (W2-R30) |
| C-C6 | **/simplify N-th organic apply** | /simplify standard workflow（per cycle 03-15 ratified；effective 2026-04-28）第 N 輪 Dev cycle 適用 | Dev `delivery_report.md §simplify` | Cycle Dev v*.* shipped + /simplify section 紀錄 → +1 | N/A | **11** (cycle 03-15~25) | cycle 03-15 |
| C-C7 | **Subagent permanent-mode N-th round** | Subagent permanent-mode（per Master directive 2026-04-25）第 N 輪 R-team R0/R1/R2/R4 完整 subagent 執行 | R0 dispatch + R4 close attestation | R0/R1/R2/R4 全用 subagent permanent-mode → +1 | N/A | **11** (cycle 03-15~25; cycle 03-13~14 為 pre-permanent transition) | cycle 03-15 |

**注意 C-C1 vs C-C7**: C-C1 (4 防線) cycle 03-19 first; C-C7 (subagent) cycle 03-15 first; **不同 baseline**。容易混淆。

### §2.3 Cumulative Counters（累積 N，only grow）

| ID | Name | Definition | Source of Truth | Update Trigger | Reset Condition | Current Value (cycle 03-25 close) | First Established |
|:--:|------|------------|-----------------|----------------|-----------------|:----------------------------------:|:-----------------:|
| C-U1 | **GREEN cells aggregate** | C-S1 × 9-metric = 累積 GREEN 細胞數（HEALTHY consecutive cycles × 9 metric per cycle） | 同 C-S1 derive | C-S1 +1 → +9 | C-S1 reset → reset to 0 | **117** (= 13 × 9) | cycle 03-13 |
| C-U2 | **Cumulative dissent (DSS) preserved** | 累計保留 DSS slot 數（per MR-11 dissent preservation；含 DSS-CY{N}-* prefix 全部）| Master_Confirmation §dissent + R3_decision_log §dissent_slots | R3 NEW DSS-CY{N} ratified → +1 | N/A (per MR-11 forever preserved) | **49** (45 carryover + 4 NEW DSS-CY25) | cycle 03-13 (DSS-1~17 baseline) |
| C-U3 | **Master directive count (active)** | Master directives 自 2026-04-25 起 forward-binding 累計數 | Master directive doc per `_tmp/master_directive_*` + canonical sync | New Master directive ratified → +1 | Sunset (rare) → -1 | **10** (2026-04-28~05-08-b 範圍；待 sunset mechanism per cycle 03-XX reform) | 2026-04-25 |
| C-U4 | **canonical doc count (baseline)** | `openstarry_doc/` 全 canonical doc files 累計數 | Coordinator G5 sync verify | New canonical doc create → +1 | Doc archive (rare) → -1 | **310** (cycle 03-26 R0 target: 334 含 Implementation_Reference/{EN,TW}/ 24 NEW) | cycle 01 baseline |

### §2.4 State Counters（當前 N，可變動）

| ID | Name | Definition | Source of Truth | Update Trigger | Reset Condition | Current Value (cycle 03-25 close) | First Established |
|:--:|------|------------|-----------------|----------------|-----------------|:----------------------------------:|:-----------------:|
| C-T1 | **Phase 6 N/7** | Phase 6 七棒功能完工進度（pushInput / AC-9 / D-30-4 / D-30-5 / Mesh / API Runtime / Blackboard-Alaya） | Master_Confirmation §phase_6 | 一棒 functional landing → +1 | N/A | **7/7 完工** (cycle 03-23 R3 D-§5 達成) | cycle 03-13 |
| C-T2 | **σ counter (Bp / Ep)** | Rule #72 SPC σ trajectory shift counter；Bp = beyond pool / Ep = excluded from pool | Master_Confirmation §SPC + W2-Rn delivery | W2-Rn σ trigger → +1 (Bp/Ep 分類) | D-14 trigger (review) → reset 0 | **0 / 0** (cycle 03-25; D-14 deactivated 持續) | cycle 03-13 (Rule #72 baseline) |
| C-T3 | **Cycle 收尾 N 條** | Coordinator G5 cycle 收尾 checklist 條目數 | Reference/16 v2 §cycle_close_checklist | New cycle 收尾 條 added → +1 | Reform consolidate → reset to grouped count | **18** (cycle 03-15 12 條 + 03-17~25 增量；待 reform 改 4 大類) | cycle 03-15 |
| C-T4 | **Replay cache N-contributor** | Replay cache prefix 累積 contributor 數（plan52, plan54, plan56, plan57 'vas:', plan58 'mes:', plan59 'apr:', plan60 'aly:'） | Master_Confirmation §replay_cache | New plan add prefix → +1 | N/A | **7** (cycle 03-23 final; Phase 6 完工 達成) | cycle 03-XX (Plan52 baseline) |

### §2.5 Status Enums（status 值；不是計數）

| ID | Name | Possible Values | Source of Truth | Current Value (cycle 03-25 close) |
|:--:|------|-----------------|-----------------|:----------------------------------:|
| C-E1 | **Tenet #1-10 status (canonical)** | `COMPLIANT` / `NC PENDING` / `NC` / `WAIVED` | **`openstarry_doc/README.md` § 十大核心宣言**（sole canonical authority per cycle 03-28 Master letter §0.1 reconcile + R3 D-§A0 ratify）; `doc 76` + `77` 用 drift list (DRIFT-FLAGGED per Master directive 2026-05-11 §3) — 不為 canonical source. **Note**: cycle 03-26 Master directive 2026-05-11 引用 `Architecture_Documentation/51_Ten_Tenets_Compliance_Report.md` 但此檔為 phantom（canonical 不存在；實際 doc 51 = `51_Codex_Review_Response.md`）；cycle 03-28 R3 D-§A0 reconcile 確認 README §十大核心宣言 為唯一 authority。 | **#1-#10 per canonical**: #1-#10 ALL COMPLIANT；cycle 03-24 endpoint 10/0/0★ FINAL substance preserved；cycle 03-26 audit re-verify forward 用 canonical axis |
| C-E2 | **Endpoint X/0/Y★** | `X/Y/Z★ ACTIVE` / `X/Y/Z★ FINAL` (X+Y+Z=10) | **`openstarry_doc/README.md` § 十大核心宣言**（sole canonical authority per cycle 03-28 R3 D-§A0 reconcile）; doc 76 + 77 DRIFT-FLAGGED; doc 51 phantom note per C-E1 above（cycle 03-29 R-team verification supplementary G5 fix 2026-05-13） | **10/0/0★ FINAL** substance preserved (cycle 03-24 + canonical 對齊；canonical Tenet #10 分形社會結構 早 v0.39.0-alpha 已 COMPLIANT；cycle 03-24 真正完成 canonical Tenet #7 final 缺口 by Phase 6 7/7) |
| C-E3 | **Plan{N} status** | `BINDING` / `CANDIDATE` / `DEFERRED` / `SUNSET` / `OLD` | Technical_Specifications/Plan{N}_*.md | Per-plan basis. Notable: Plan53 SUNSET (cycle 03-25 BINDING ratify); Plan55 DEFERRED FINAL (cycle 03-21); Plan60 BINDING (cycle 03-23) |
| C-E4 | **F-16 status** | `MAY` / `SHOULD` / `MUST` / `RETIRED` | ENG-FAB v1.8 §F-16 + cycle 03-21 R3 D-§3 | **SHOULD initial FINAL** (cycle 03-21 R3 D-§3 binary final terminal); ENG-FAB v1.9 candidate retired |
| C-E5 | **D-14 trigger status** | `ACTIVE` / `DEACTIVATED` | Master_Confirmation §SPC | **DEACTIVATED** (cycle 03-20 W2-R24 σ RESET to baseline; sustained 03-21~25) |

### §2.6 Sequence Counters（流水號）

| ID | Name | Definition | Source of Truth | Update Trigger | Current Value (cycle 03-25 close) | First Established |
|:--:|------|------------|-----------------|----------------|:----------------------------------:|:-----------------:|
| C-Q1 | **Master Ratification Batch N** | Master_Confirmation Batch 流水號 | Master_Confirmation `Batch_{N}_Request.md` | 新 Batch dispatch → +1 | **22** (cycle 03-25 close) | cycle 03-12 (Batch 1 baseline) |
| C-Q2 | **W2-Rn round number** | W2 task round 流水號（even numbers per cycle） | Test team task list per Master directive `W2-R{N}` | 新 round dispatch → +2 (even only) | **34** (cycle 03-25; cycle 03-26 target W2-R36) | cycle 03-13 (W2-R10 baseline) |
| C-Q3 | **Cycle ID** | Project cycle 流水號（format `03-{NN}` for cycle 03-X group） | Master letter per cycle | 新 cycle Master letter dispatch → next cycle | **03-25** (closed); **03-26** in pipeline | cycle 01 baseline |
| C-Q4 | **Release tag** | Dev release tag semver | Dev release procedure | 每次 release → next semver | **v0.57.2-alpha** (cycle 03-25 final) | v0.20.0-beta baseline |

---

## §3 Disambiguation Rules（誤用防呆）

### §3.1 §75.X Counter — 三邊釐清（cycle 03-25 spot-check 觸發）

| 名稱 | Counter ID | 算法 | Cycle 03-25 close value | 哪邊用此值 |
|------|:---:|------|:----------------------:|------------|
| §75.X enforce (canonical) | **C-C4** | 每個 release tag enforce 一次（含 hotfix + doc-only）| **17** | Dev delivery_report 主算法（per Rule #75 §75.X §4.2 spec literal） |
| §75.X per-cycle enforce | **C-C4a** | Cycles 含 ≥1 release tag | **12** | Master_Confirmation §release_tags 列表（cycle 視角；不論一輪幾個 tag） |
| §75.X spot-check enforce | **C-C4b** | Cycles where M-tier spot-check 對 §75.X verification | **4** | R-team spot-check ring（spot-check 視角） |

**Resolution per cycle 03-26 R0**:
- **Dev 算法 = canonical（C-C4 = 17）**；其他兩個算法**改稱不同名字**，不再叫「§75.X N-th enforce」。
- Spec / Master_Confirmation 用 C-C4a 時須明寫「§75.X per-cycle enforce = 12」（不省略 per-cycle qualifier）。
- Spot-check ring 用 C-C4b 時須明寫「§75.X spot-check enforce = 4」（不省略 spot-check qualifier）。

### §3.2 HEALTHY 連續 vs Subagent permanent-mode round 容易混淆

| Counter | ID | Cycle 03-25 close value | 差別 |
|---------|:---:|:------------------------:|------|
| HEALTHY consecutive cycles | C-S1 | **13** | cycle 03-13 起；包含 cycle 03-13~14 pre-permanent transition |
| Subagent permanent-mode N-th round | C-C7 | **11** | cycle 03-15 起；不含 cycle 03-13~14（subagent permanent-mode 還沒上） |

**Resolution**: 兩者 baseline 不同；不可互相代換。文件中提到「N consecutive HEALTHY」必引 C-S1 = 13；提到「Nth subagent permanent-mode round」必引 C-C7 = 11。

### §3.3 4 防線 enforce vs amendment apply vs isolation v2 sustained 容易混淆

| Counter | ID | Cycle 03-25 close value | First established cycle |
|---------|:---:|:------------------------:|:----------------------:|
| 4 防線 N-th enforce | C-C1 | **7** | cycle 03-19 |
| Reference/16 amendment N-th cycle apply | C-C2 | **5** | cycle 03-21 |
| 5-point isolation v2 N-th sustained | C-C3 | **5 STRENGTHENED** | cycle 03-21 |

**Resolution**: 三個 counter 屬不同 directive，不同 first cycle。dispatch payload 引用時必標 ID + value（e.g. 「C-C1 = 7th enforce, C-C2 = 5th apply, C-C3 = 5th sustained STRENGTHENED」）。

### §3.4 「Release tag」精確定義（v0.1.1 修訂；解決 cycle 03-27 §75.X 第 5 次 flagging）

**觸發事件**：cycle 03-27 W2-R38 + R-team 驗證報告 `cycle03-27/R-team_verification/verification_report.md §6` 第 5 次 flag §75.X counter discrepancy：
- **Dev 端計數（per C-C4 canonical）**：18（v0.50.0-alpha 起累計到 v0.57.4-alpha = 18 個 release tag）
- **R-team 端解讀（per Reference/21 v0.1 算法）**：19（誤算入 cycle 03-26 假設的 v0.57.3-alpha；實際 cycle 03-26 為 audit cycle，無 Dev release）

**根因分析**：§2.2 C-C4 row「每個 release tag (含 hotfix + doc-only) Dev 通過 §75.X 驗證 → +1」中「release tag」缺乏精確定義，導致 R-team 把 cycle 03-26 G5 canonical-doc-deploy 誤算為 release tag。

**Resolution**：「release tag」精確定義如下（forward-binding cycle 03-27 close 起）：

| 屬性 | 定義 |
|------|------|
| **唯一充要條件** | `openstarry_eco/release/cycleXX_vY.Y.Z-alpha/` 目錄實體存在（含完整 release artefact：openstarry/ + openstarry_plugin/ + openstarry_doc/ + delivery_report.md + release notes if any） |
| **充分條件** | Dev `delivery_report.md` 含 §release_tag 段且 §75.X enforce evidence（pnpm build PASS hash）attest |
| **不算 release tag 的情況** | (1) 僅 Canonical doc G5 sync（無 release dir 創建）— e.g. cycle 03-26 R0 → R5 無 v0.57.3-alpha (2) Suggestion/agent_dev mirror sync (3) `_archive/` 內歷史檔（非 active release） (4) Plan / Reference / Master directive doc-only ratification (5) Hotfix mid-cycle 預備版本（未 cut release dir）|
| **驗證方法** | Coordinator G5 sync 驗 `release/cycleXX_*` glob 與 Dev `delivery_report.md §release_tag` 一致；R-team verification 用同一 source |

**Cycle 03-27 close 後 C-C4 reconciled value (v0.1.1 原斷言 — 已被 v0.1.2 取代)**：~~**18**~~ — 見 §3.4.1 v0.1.2 retroactive reconcile

**§75.X 第 5 次 flagging 結案 (v0.1.1 partial; v0.1.2 final)**：Dev 算法為 canonical truth；R-team v0.1 algorithm reading 因「release tag」歧義而誤算；本 §3.4 釐清後 forward 不再產生 discrepancy。

### §3.4.1 v0.1.2 Retroactive Reconcile — v0.57.3-alpha 補齊後 C-C4 真實值（mechanical correction 2026-05-12）

**觸發事件**：2026-05-12 Master directive — cycle 03-26 release dispatch 漏派經 audit 發現；coordinator dispatch Dev task #190 補做 v0.57.3-alpha doc-only patch retroactive；release dir `release/cycle03-26_v0.57.3-alpha/` + delivery_report.md + 342-canonical-snapshot 全部 packaged complete 2026-05-12T08:15Z。

**v0.1.1 §3.4 reconcile 假設失效**：v0.1.1 §3.4 假設「v0.57.3-alpha 不存在」基於當時的 silent failure（cycle 03-26 release dispatch 漏派 = 沒派 = 假設沒有）。實際根因為「coordinator 漏派 = bug」而非「不該派」。Bug 補齊後 v0.57.3-alpha 確實存在，符合 §3.4 release tag 充要條件（`release/cycle03-26_v0.57.3-alpha/` 目錄實體存在 + delivery_report.md 含 §75.X enforce evidence）。

**Reference/21 §3.4 release tag 精確定義本身保留 unchanged**（規則沒錯；錯的是當時 v0.57.3-alpha 真的不存在的事實）。本 §3.4.1 為 mechanical reconcile 因應補做後事實變更。

**Cycle 03-27 close + cycle 03-26 retroactive backfill 後 C-C4 reconciled value (v0.1.2 final)**：**19** — release tag 全列表（v0.50.0-alpha 起按 cut 時序）：

| # | Release tag | Cycle | Cut date | §75.X enforce |
|:-:|-------------|-------|----------|:--------------:|
| 1-17 | v0.50 ~ v0.57.2-alpha | 03-14 ~ 03-25 | 2026-04 ~ 2026-05-07 | ✅ × 17 |
| **18** | **v0.57.3-alpha** | **03-26** | **2026-05-12 retroactive** | ✅ doc-only |
| **19** | **v0.57.4-alpha** | **03-27** | **2026-05-11** | ✅ hygiene fix |

**Cycle 03-27 close + retroactive backfill 後 reconciled values**:
- **C-C4 = 19** (v0.1.1 誤稱 18)
- **C-C4a = 14**（cycle 03-26 + cycle 03-27 各貢獻 1；total cycles with ≥1 release tag = 14）
- **C-C4b = 4 sustained**（無 spot-check 變動）

**Forward-binding cycle 03-28 起**：§3.4 release tag 精確定義 + §3.4.1 retroactive reconcile precedent。Future cycle close 必須驗證 release dispatch 完成 (per §3.4 充要條件) 才能 close cycle；coordinator G5 stage 加 release-dispatch-completion check (per Master directive 2026-05-12 implied)。

### §3.6 Mirror Discipline — Single-Target Rule（v0.1.3 修訂 2026-05-12；解決 Master directive 2026-05-08 真正落地後 mirror 範圍歧義）

**觸發事件**：2026-05-12 Master directive 2026-05-08 真正落地（Dev task #191 — `openstarry/docs/` 從 4 個 location 移除）。落地後 coordinator G5 mirror discipline 須 codify「不再 mirror canonical doc 到 `openstarry/docs/canonical/`」單一目標規則。

**根因分析**：cycle 03-27 close coordinator G5 mirror Reference/21 v0.1.1 時，把 canonical doc copy 到 13 個 location 包含 `openstarry/docs/canonical/Reference/21_Counter_Registry.md`（4 個 release / agent_dev / test codebase）。這些 path 本來就違反 directive 2026-05-08。Dev cleanup #191 移除 docs/ 同時刪了違規 mirror copy（無內容流失），但若無單一目標規則，下次 G5 又會自動 propagate violation。

**Resolution — Single-Target Mirror Rule**（forward-binding cycle 03-28 起）：

| Mirror discipline | 規則 |
|------|------|
| **單一目標 canonical 路徑** | Canonical doc 唯一鏡同步目標 = `openstarry_doc/`。**不再 mirror 到 `openstarry/docs/canonical/`** |
| **核心 mirror set** | `share/openstarry_doc/` + `agent_dev/openstarry_doc/` + `release/cycleXX_*/openstarry_doc/` + `test research/cycleXX/codebase_*/openstarry_doc/` + `research input/research_version/cycleXX_*/openstarry_doc/` + `test research/cycleXX/research_reference/` |
| **典型 mirror count** | 6-9 locations |
| **絕對禁止** | mirror canonical doc 到 `openstarry/docs/canonical/` |
| **驗證 protocol** | Coordinator G5 完成後 `find . -path '*openstarry/docs/canonical/Reference/*'` 應 return empty |

**Pre-2026-05-12 既有 release 不溯及修改**（per MR-12）：cycle 03-25 v0.57.2-alpha 及之前 release 含 `openstarry/docs/canonical/` 為歷史 snapshot 不動；cycle 03-26 v0.57.3-alpha + cycle 03-27 v0.57.4-alpha 已 in-place cleanup（task #191）。

**Mirror count 變化**：v0.1.1/v0.1.2 = 誤 mirror 13 locations；v0.1.3 = corrected 9 locations（移除 4 個違規 path）。

### §3.5 Hygiene-only Cycle Counter 累進規則（v0.1.1 修訂；解決 C-C6/C-C7 governance 解讀爭議）

**觸發事件**：cycle 03-27 R-team 驗證報告 §7 F-CY27-§R-V-08 flag C-C6 (/simplify) + C-C7 (subagent permanent-mode) counter 累進歧義 — cycle 03-27 為 hygiene-only fix cycle（無 R0-R3 vote、無完整 R4 G4 gate、無完整 subagent batch parallel）。

**根因分析**：Reference/21 v0.1 §2.2 C-C6/C-C7 row 之 update trigger 假設 cycle 含完整 R0-R3 + R4 G4 gate + subagent permanent-mode batch；hygiene-only cycle scope 縮減未明文涵蓋。

**Resolution**：Hygiene-only cycle counter 累進規則（forward-binding cycle 03-27 close 起）：

| 規則 | 內容 |
|------|------|
| **Default = deferred** | Hygiene-only cycle 對需「該 counter 對應動作實際發生」的 counter（C-C6 / C-C7 / C-C2 amendment apply / C-C1 4 防線 / C-C3 isolation sustained 等）默認 **不 +1**，保持 sustained 值；除非該動作在此 hygiene cycle 確實發生（per Dev `delivery_report.md` 或 R-team verification report 明文紀錄） |
| **Always-tracked counters** | C-S1 (HEALTHY)、C-S4 (F1/F2/F3 zero)、C-S6 (correction rounds 0)、C-U1 (GREEN cells)、C-Q2 (W2-Rn)、C-Q4 (release tag) 等 **依舊 +1**（不論 hygiene/full cycle；只要 W-round + release tag 出現即 +1） |
| **§75.X enforce (C-C4)** | **依舊 +1**（hygiene cycle 出 release tag → +1；hygiene cycle 不出 release tag → 0；本規則同 §3.4 release tag 精確定義） |
| **Hygiene cycle 定義** | (1) 無 R0-R3 vote（無 R3_decision_log 產出） (2) 無完整 subagent permanent-mode batch（partial subagent or main session only） (3) Master directive 明文歸類為「hygiene-only fix cycle」or「post-audit fix cycle」(per Master directive 2026-05-09 §3.1 PASS path branch) |
| **驗證 protocol** | Hygiene cycle counter snapshot 段須註明「（hygiene-only cycle deferred-by-default per §3.5）」字樣，與 full cycle snapshot 區分 |

**Cycle 03-27 close 後 C-C6/C-C7 reconciled value**：
- **C-C6 = 12 sustained**（cycle 03-26 R4 close = 12；cycle 03-27 hygiene + Dev `delivery_report.md §7.4` 無 organic /simplify → deferred → sustained）
- **C-C7 = 12 sustained**（cycle 03-26 R4 close = 12；cycle 03-27 hygiene + 無完整 subagent batch → deferred → sustained）

**Cycle 03-28+ reform cycle counter 累進**：Reform cycle 為 full cycle（含 R0-R3 vote），applies §2.2 default rule (+1)；hygiene cycle deferred-by-default 規則僅適用 hygiene scope。

---

## §4 Update Protocol

### §4.1 Per-cycle counter snapshot

每 cycle R4 close 後 + Master_Confirmation Batch ratification 後，coordinator G5 stage **MUST** 產生 cycle counter snapshot（append 至 cycle CHANGELOG）：

```
### Cycle 03-{NN} Counter Snapshot (per Reference/21)

C-S1 = N (was N-1; +1)
C-S2 = N (was N-1; +1)
C-S3 = N (was N-1; +1)
...
C-C4 = N (was N-X; +X release tags this cycle: [list])
...
```

### §4.2 Counter discrepancy detection

R-team R3 / coordinator G5 / Master spot-check 任一環節若發現 counter 三邊不一致：

1. **NOT** 各自定 patch；**MUST** 引用 Reference/21 algorithm 與算法 source
2. 若發現 Reference/21 本身 algorithm 不清楚 / 不正確 → trigger Reference/21 amendment via R3 vote (Counter Registry amendment 屬 audit cycle 內 R3 議題；cycle 03-26 適用 §75.X reconcile precedent)
3. amendment 須加新 row（不修舊 row；per MR-12 forward-only）

### §4.3 New counter introduction

新 counter（非本 registry 列出者）出現時：

1. R-team / coordinator / Dev / Test 發現 → 立即向 coordinator 申報
2. Coordinator 評估 → drafting 加 row 至 Reference/21（與其他 docs 同步引用 ID + algorithm）
3. R3 ratify → BINDING（merge to Reference/21 v{N+1} 或 amendment doc）

### §4.4 Counter sunset

Counter 連續 5 cycles 未變動 + 未 reference → trigger sunset review（per Master directive 2026-05-XX sunset mechanism；cycle 03-XX reform cycle 議題）。本 v0.1 不含 sunset clause；reform cycle 後升 v2 補。

---

## §5 Pre-Cycle 03-26 Application

### §5.1 §75.X reconcile（cycle 03-26 letter §三）

- **原 letter §三**: 「§75.X counter discrepancy reconcile」
- **本 registry 之後**: §三 改寫為「**Counter Registry consolidation**：採用 Reference/21 v0.1 為 canonical；§75.X 三邊釐清 per §3.1 = first application」
- **Cycle 03-26 R0 dispatch payload**: 附 Reference/21 v0.1 全文 + 強調 §3.1 § §75.X 三邊釐清
- **R3 D-§3**: informational confirm（per Master directive 2026-05-09 §2.3 條件 2 mechanical correction；不投票）

### §5.2 對其他 doc 的影響

- **CHANGELOG_RESEARCH_TEAM.md**: 下次 cycle entry 加 counter snapshot section per §4.1
- **Master_Confirmation Batch 23 (cycle 03-26)**: 引用 counter 必標 ID（e.g. 「C-S1 = 14」非「14 consecutive HEALTHY」）
- **R3_decision_log**: D-item 引用 counter 必標 ID
- **Plan / Reference / Research_Methodology docs 已存在**: **不溯及修改**（per MR-12）；只 forward use ID

---

## §6 Compliance Attestation

| Constraint | Status | Evidence |
|------------|:------:|----------|
| MR-2 / MR-4 (Tenet wording unchanged) | ✅ PASS | Counter registry; no Tenet wording |
| MR-5 hard (Tenet #10 status unchanged) | ✅ PASS | Process / governance doc |
| MR-6 (Core 零) | ✅ PASS | No Core surface |
| MR-7 (post L1-L4) | ✅ PASS | Counter governance strengthens audit reproducibility |
| MR-8 (quality first) | ✅ PASS | Resolves §75.X 多輪 ambiguity；audit verdict 引用更精確 |
| MR-9 (no MUST WAIVE) | ✅ PASS | Update protocol 嚴格；無 WAIVE |
| MR-10 (back-fill) | ✅ PASS | cycle 03-25 spot-check ring identified gap; cycle 03-26 R0 backfill via this doc |
| MR-11 (dissent preservation) | N/A | Master directive 2026-05-09 §2.3 條件 2 mechanical execution; no R3 vote; no new dissent |
| MR-12 (既有不破壞) | ✅ PASS | forward-binding cycle 03-26+; 既有 doc 引用 counter 不溯及修改 |
| ZT-1 / ZT-2 / ZT-3 | ✅ PASS | No tenet rewrite; endpoint unchanged; control-range unaffected |

---

## §7 Cross-References

- `Reference/10_Rule_75_Section_75_X_pnpm_build.md` — Rule #75 §75.X spec literal（C-C4 algorithm source）
- `Reference/19_Audit_Methodology_Codification.md` — M1 BINDING（matrix sweep cell counter 適用）
- `Reference/20_Audit_Verdict_Format_Codification.md` — A9 BINDING（verdict cell aggregate counter 適用）
- `~/.claude/projects/C--Users-yulin-Desktop-openstarry/memory/project_master_directive_2026-05-09_audit_fix_non_mixing.md` — Master directive 2026-05-09 §2.3 條件 2 (本 doc 執行 authority)
- `openstarry_doc/Reference/16_R4_Folder_Convention_Discipline_v2.md` — Reference/16 v2 consolidation (4 防線 / amendment counter source)
- `claude research/research record/cycle03-25/deliver/Master_Ratification/Master_Confirmation.md` — Batch 22 latest counter snapshot
- `C:/Users/yulin/.claude/projects/C--Users-yulin-Desktop-openstarry/memory/MEMORY.md` — cycle 03-25 progress entry full counter values

---

## §8 Resolved Items（Master sign-off 2026-05-11）

1. **Sunset clause**: ❌ NOT included in v0.1; deferred to reform cycle (cycle 03-28+ per Master directive 2026-05-09 §4 Reform cycle table row); v2 will add sunset clause informed by reform deliberation
2. **TW sibling**: ✅ Coordinator G5 sync 階段 create `Reference/21.tw.md` per Rule #78 §78.5 BINDING-tier reflexive same-PR pattern
3. **Status enum (§2.5) 分流 to Reference/22**: ❌ NOT split; maintain in §2.5 (low cardinality; split overhead > benefit)
4. **C-C1 / C-C7 / C-S1 baseline confirmation**: ✅ Master confirmed 2026-05-11 — C-C1 first cycle = 03-19 (4 防線 binding 開始) / C-C7 first cycle = 03-15 (subagent permanent-mode 開始) / C-S1 first cycle = 03-13 (HEALTHY 計數開始)
5. **C-U2 dissent = 49 verification**: ⏳ R-team R0 階段 audit 確認（與 R3_decision_log + DSS-CY{N} prefix 全部 cross-tabulate）；若不符 trigger Reference/21 v0.1 → v0.1.1 patch（mechanical correction per Master directive 2026-05-09 §2.3 condition 2）
6. **§75.X 第 5 次 flagging (Dev=18 vs R-team=19)**：✅ 已解決 per §3.4（v0.1.1 修訂 2026-05-11 cycle 03-27 close）；release tag 精確定義 = `release/cycleXX_*` 目錄實體存在；~~C-C4 reconciled = 18（Dev canonical algorithm）~~ — 已被 §3.4.1 v0.1.2 retroactive reconcile 取代（C-C4 = 19 after v0.57.3-alpha cycle 03-26 retroactive packaging 2026-05-12）
7. **C-C6/C-C7 hygiene cycle counter 累進歧義**：✅ 已解決 per §3.5（v0.1.1 修訂 2026-05-11 cycle 03-27 close）；hygiene-only cycle counter 累進 = deferred-by-default；cycle 03-27 C-C6/C-C7 = 12 sustained（無 organic action）
8. **Cycle 03-26 release dispatch 漏派 → 2026-05-12 retroactive 補做**：✅ 已解決 per §3.4.1（v0.1.2 retroactive reconcile；coordinator dispatch Dev task #190 → release/cycle03-26_v0.57.3-alpha/ doc-only patch packaged complete；C-C4 = 19；v0.57.3-alpha = 18th, v0.57.4-alpha = 19th）；§3.4 規則 unchanged（規則沒錯；v0.1.1 reconciled value 是基於當時的 silent-failure absence）
9. **Master directive 2026-05-08 release structure 真正落地（`openstarry/docs/` 移除）**：✅ 已解決 per §3.6（v0.1.3 mirror discipline post-cleanup；coordinator dispatch Dev task #191 → 4 location 整個 `openstarry/docs/` 移除：agent_dev + cycle03-26 release + cycle03-27 release + test codebase；agent_dev pnpm build/test/purity PASS；ε-surface 8-cycle preservation；sibling-naming-check.mjs retarget Rule #62 root-cause fix bundled；mirror count 13 → 9）

---

*Reference/21 — Counter Registry — RATIFIED v0.1.3 — 2026-05-12（v0.1.1/v0.1.2 ratified 2026-05-11/2026-05-12）*
*Single Source of Truth for All Project Counters*
*Authority: Master directive 2026-05-09 §2.3 condition 2 (mechanical correction) + Master directive 2026-05-12 (cycle 03-26 release dispatch 補做 + Master directive 2026-05-08 真正落地)*
*Effective cycle 03-26 R0 起 (v0.1) + §3.4/§3.5 cycle 03-27 close 起 (v0.1.1) + §3.4.1 cycle 03-26 retroactive reconcile 2026-05-12 起 (v0.1.2) + §3.6 mirror discipline single-target rule 2026-05-12 起 (v0.1.3)*
*First application: §75.X 三邊釐清 (C-C4 = 17 canonical; C-C4a = 12 per-cycle; C-C4b = 4 spot-check)*
*v0.1.1 修訂：§3.4 release tag 精確定義 + §3.5 hygiene-only cycle counter 累進規則*
*v0.1.2 修訂：§3.4.1 retroactive reconcile after cycle 03-26 v0.57.3-alpha 補做（C-C4 = 19 final；v0.57.3-alpha = 18th, v0.57.4-alpha = 19th）*
*v0.1.3 修訂：§3.6 mirror discipline single-target rule（canonical 鏡同步唯一目標 = `openstarry_doc/`；mirror count 13 → 9）*
*25+ counters codified across 6 categories: streak / cardinality / cumulative / state / status / sequence*
