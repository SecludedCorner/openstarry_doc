---
title: Reference/21 — Counter Registry (Single Source of Truth for All Project Counters)
date: 2026-05-09 (v0.1) / 2026-05-11 (v0.1.1 amendment)
status: CANDIDATE v0.2.3 (cycle 03-33 R4 close — §10.1 C-C8 cycle 03-33 close value update 2 → 3 per cycle 03-33 R3 §4.5 D-§A6.5 Test team Stream C tier-4 PASS attestation; §10.2.2 NEW cycle 03-33 snapshot block; §10.3 forward outlook expansion dual-counter forward observation NOT BINDING this cycle; Master Ratification Batch 28 forward dispatch routing) / RATIFIED v0.2.2 (Master Ratification Batch 26 sign-off 2026-05-14 — substance addition §10 C-C8 NEW counter Phase 7 T4 cycle count + §3.9 evidence-tier ladder canonical per cycle 03-31 R3 D-§A6.5 Master Ratification Batch 26 Item #18; v0.2.1 RATIFIED Batch 25 sign-off 2026-05-13 consolidated)
authority: Master directive 2026-05-09 §2.3 condition 2 (mechanical correction) + Master directive 2026-05-12 (cycle 03-26 release dispatch补做 + research input transfer补齊 + Master directive 2026-05-08 真正落地) + cycle 03-28 R3 D-§A0 (canonical Tenet authority reconcile) + cycle 03-30 R3 D-§A1.4-P2 (Option γ Hybrid forward strategy + C-LEIBNIZ-N2-observation NEW counter Master Ratification Batch 25 Item #7)
effective: cycle 03-26 R0 onward (v0.1) + §3.4/§3.5 cycle 03-27 close onward (v0.1.1) + §3.4.1 cycle 03-26 retroactive reconcile 2026-05-12 onward (v0.1.2) + §3.6 mirror discipline single-target rule 2026-05-12 onward (v0.1.3) + §2.5 C-E1 phantom doc 51 reference removal cycle 03-28 R4 close 2026-05-12 onward (v0.1.4) + §9 C-LEIBNIZ-N2-observation NEW counter + §3.8 Option γ Hybrid cycle 03-30 R3 close 2026-05-13 onward (v0.2.1)
amendment_history:
  - v0.1 → v0.1.1 (2026-05-11): added §3.4 release tag definition (resolves §75.X 5th-flagging Dev=18 vs R-team=19) + §3.5 hygiene-only cycle counter increment rule (resolves C-C6/C-C7 governance interpretation per cycle 03-27 R-team verification §7 F-CY27-§R-V-08); 既有 row 不修改 per MR-12 forward-only
  - v0.1.1 → v0.1.2 (2026-05-12): added §3.4.1 retroactive reconcile after cycle 03-26 v0.57.3-alpha doc-only patch补做 per Master directive 2026-05-12 (cycle 03-26 release dispatch was missed at the time → coordinator dispatch Dev task #190 retroactive packaging → release/cycle03-26_v0.57.3-alpha/ now exists → §3.4 release tag 充要條件 satisfied → C-C4 reconciled = 19 not 18; v0.57.3-alpha = 18th, v0.57.4-alpha = 19th); §3.4 rule unchanged (rule was correct; v0.1.1 reconciled value 18 was based on then-true silent-failure absence of v0.57.3-alpha; v0.1.2 reconciles to true 19 after补做)
  - v0.1.2 → v0.1.3 (2026-05-12): added §3.6 Mirror Discipline Single-Target Rule after Master directive 2026-05-08 真正落地 (Dev task #191 — `openstarry/docs/` 從 4 location 移除：agent_dev + cycle03-26 release + cycle03-27 release + test codebase; agent_dev pnpm build/test/purity PASS; sibling-naming-check.mjs retarget Rule #62 root-cause fix bundled); single-target = canonical 鏡同步唯一目標 = `openstarry_doc/`，禁止 mirror 到 `openstarry/docs/canonical/`；典型 mirror count 6-9 locations（v0.1.3 之後 9 mirrors，v0.1.1/v0.1.2 時誤 mirror 到 13 locations）
  - v0.1.3 → v0.1.4 (2026-05-12 cycle 03-28 R4 close): mechanical patch §2.5 C-E1 (Tenet canonical authority) — phantom `Architecture_Documentation/51_Ten_Tenets_Compliance_Report.md` reference removed per cycle 03-28 R3 D-§A0 reconcile (canonical 不存在此檔；實際 doc 51 = `51_Codex_Review_Response.md`，與 Tenet 無關)；sole canonical authority = README §十大核心宣言；C-E2 同步更正。Substance additions §3.7 (counter sunset clause per B-1 ratify) + C-S1-roll12 NEW counter (per B-3 ratify) **deferred to v0.2 post-Master-Ratification Batch 24** per O4 §5 promotion path.
  - v0.2 → v0.2.1 (2026-05-13 cycle 03-30 R3 close + Master Ratification Batch 25 Item #7): substance addition §9 C-LEIBNIZ-N2-observation NEW state counter (current value = 2 cells at LEIBNIZ N=2 ceiling: p10_t6 distributed-alaya × Tenet#6 + p13_t6 guide-character-init × Tenet#6) + §3.8 Option γ Hybrid forward strategy (Option β sustain-CP-w-sunset default + Option α forced-escalate flag if specific evidence emerges); ratifies cycle 03-30 R3 D-§A1.4-P2 LEIBNIZ N=2 ceiling escalation strategy; DSS-CY30-§A1.4-P2-α preserved per MR-11 (LEIBNIZ + KNUTH 2 votes NO; preferred Option α strict)
  - v0.2.1 → v0.2.2 (2026-05-14 cycle 03-31 R3 close + Master Ratification Batch 26 Item #18): substance addition §10 C-C8 NEW cardinality counter (Phase 7 T4 cycle count;current value = 1 inaugural at cycle 03-31 via Test team Stream 1 cell-1 full-daemon tier-4 execution PASS) + §3.9 evidence-tier ladder canonical (tier-1 text-only / tier-2 designed-not-executed / tier-3 inline-contract real-LLM / tier-4 full-daemon real-LLM); ratifies cycle 03-31 R3 D-§A6.5 NEW counter + evidence-tier ladder codification; **C-LEIBNIZ-N2-observation update §9.4 cycle 03-31 close**: 2 cells → 0 cells (window 雙 CLOSED;p10_t6 Option α activate via Stream 2 multi-host evidence + p13_t6 Option α activate via Stream 3 FIX-A/B evidence per Batch 26 Items #14-#16)
  - v0.2.2 → v0.2.3 (2026-05-15 cycle 03-33 R4 close + Master Ratification Batch 28 forward dispatch routing): §10.1 C-C8 cycle 03-33 close value update **2 → 3** per cycle 03-33 R3 §4.5 D-§A6.5 (Test team Stream C tier-4 PASS attestation:cycle 03-33 third consecutive cycle Tier-4 execution achieved + cycle 03-31 inaugural + cycle 03-32 second cycle preserved as immutable history per MR-12 forward-only); §10.2.2 NEW cycle 03-33 close snapshot block (forward-only addition; §10.2 cycle 03-31 inaugural snapshot preserved verbatim);§10.3 forward observation expansion to include dual-counter forward observation NOT BINDING this cycle (C-C8 cardinality + Tenet#8 column closure progress; Tenet#8 column 9/38 candidate;cross-Tenet anchor pattern N=3 empirical validation graduation per Reference/20 v3 §6 CY33-NOTE-2 amendment scope mark);**C-LEIBNIZ-N2-observation §9.4 cycle 03-33 close**: 0 cells sustained (window CLOSED sustained 4-cycle 03-31~33;DSS-CY30-§A1.4-P2-α sunset clause 永久 standby NOT triggered per MR-11);ratifies cycle 03-33 R3 §4.5 D-§A6.5 + 永久 forward-only attestation;forward C-C8 inheritance cycle 03-34+ Path 2 mixed continuation or column pivot per R0 chair
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
---

# Reference/21 — Counter Registry (Single Source of Truth)

**Status**: **CANDIDATE v0.2.3** (cycle 03-33 R4 close; §10.1 C-C8 cycle 03-33 close value update 2 → 3 + §10.2.2 NEW cycle 03-33 snapshot block + §10.3 forward outlook expansion dual-counter forward observation NOT BINDING this cycle; Master Ratification Batch 28 forward dispatch routing) / **RATIFIED v0.2.2** (Master Ratification Batch 26 sign-off 2026-05-14; substance §10 C-C8 NEW counter Phase 7 T4 cycle count + §3.9 evidence-tier ladder canonical + §9.4 C-LEIBNIZ-N2-observation cycle 03-31 close update 2→0; consolidates v0.2.1 Batch 25 ratification)
**Authority**: Master directive 2026-05-09 §2.3 condition 2 (mechanical correction; coordinator pre-R0 execute) + Master directive 2026-05-12 (cycle 03-26 release dispatch补做 + Master directive 2026-05-08 真正落地) + cycle 03-28 R3 D-§A0 ratify
**Effective**: cycle 03-26 R0 onward (v0.1) + §3.4/§3.5 cycle 03-27 close onward (v0.1.1) + §3.4.1 cycle 03-26 retroactive reconcile 2026-05-12 onward (v0.1.2) + §3.6 mirror discipline single-target rule 2026-05-12 onward (v0.1.3) + §2.5 C-E1 phantom doc 51 reference removal cycle 03-28 R4 close 2026-05-12 onward (v0.1.4)
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
| C-S1-roll12 | **HEALTHY rolling 12-cycle rate** (v0.2 NEW per cycle 03-28 R3 D-§B.3 21/2 ratify; Master Ratification Batch 24 Item #9) | Last 12 cycles HEALTHY rate; max(min(cycle_count, 12), 1) divisor | R4 close attestation per Master_Confirmation §healthy_metrics | each R4 close; recompute rolling sum | never reset; rolling pure window | **12/12 = 100%** (cycle 03-17~28 all HEALTHY; back-fill from existing R4 attestations) | cycle 03-28 (codification) |

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
| C-E1 | **Tenet #1-10 status (canonical)** | `COMPLIANT` / `NC PENDING` / `NC` / `WAIVED` | **`openstarry_doc/README.md` § 十大核心宣言** (sole canonical authority per cycle 03-28 Master letter §0.1 reconcile + R3 D-§A0 ratify); `doc 76` + `77` 用 drift list (DRIFT-FLAGGED per Master directive 2026-05-11 §3) — 不為 canonical source. **Note**: cycle 03-26 Master directive 2026-05-11 referenced `Architecture_Documentation/51_Ten_Tenets_Compliance_Report.md` but this doc is **phantom** (never canonically deployed; canonical has `51_Codex_Review_Response.md` instead); cycle 03-28 R3 D-§A0 reconcile retains README §十大核心宣言 as sole authority. | **#1-#10 per canonical**: #1-#10 ALL COMPLIANT; cycle 03-24 endpoint 10/0/0★ FINAL substance preserved; cycle 03-26 audit re-verify forward 用 canonical axis |
| C-E2 | **Endpoint X/0/Y★** | `X/Y/Z★ ACTIVE` / `X/Y/Z★ FINAL` (X+Y+Z=10) | **`openstarry_doc/README.md` § 十大核心宣言** (sole canonical authority per cycle 03-28 R3 D-§A0 reconcile); doc 76 + 77 DRIFT-FLAGGED; doc 51 phantom note per C-E1 above | **10/0/0★ FINAL** substance preserved (cycle 03-24 + canonical 對齊；canonical Tenet #10 分形社會結構 早 v0.39.0-alpha 已 COMPLIANT；cycle 03-24 真正完成 canonical Tenet #7 final 缺口 by Phase 6 7/7) |
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

### §3.4 「Release tag」精確定義（v0.1.1 amendment；resolves cycle 03-27 §75.X 5th-flagging）

**觸發事件**: cycle 03-27 W2-R38 + R-team verification report `cycle03-27/R-team_verification/verification_report.md §6` flagged §75.X counter discrepancy 第 5 次：
- **Dev-side count (canonical per C-C4)**: 18（v0.50.0-alpha 起累計到 v0.57.4-alpha = 18 個 release tag）
- **R-team-side reading (per Reference/21 v0.1 algorithm)**: 19（誤算入 cycle 03-26 假設的 v0.57.3-alpha；實際 cycle 03-26 為 audit cycle，無 Dev release）

**根因分析**: §2.2 C-C4 row「每個 release tag (含 hotfix + doc-only) Dev 通過 §75.X 驗證 → +1」中「release tag」缺乏精確定義，導致 R-team 把 cycle 03-26 G5 canonical-doc-deploy 誤算為 release tag。

**Resolution**: 「release tag」精確定義如下（forward-binding cycle 03-27 close 起）：

| 屬性 | 定義 |
|------|------|
| **唯一充要條件** | `openstarry_eco/release/cycleXX_vY.Y.Z-alpha/` 目錄實體存在（含完整 release artefact：openstarry/ + openstarry_plugin/ + openstarry_doc/ + delivery_report.md + release notes if any） |
| **充分條件** | Dev `delivery_report.md` 含 §release_tag 段且 §75.X enforce evidence（pnpm build PASS hash）attest |
| **不算 release tag 的情況** | (1) Canonical doc G5 sync only（無 release dir 創建）— e.g. cycle 03-26 R0 → R5 無 v0.57.3-alpha (2) Suggestion/agent_dev mirror sync (3) `_archive/` 內歷史檔（非 active release） (4) Plan / Reference / Master directive doc-only ratification (5) Hotfix mid-cycle 預備版本（未 cut release dir）|
| **驗證方法** | Coordinator G5 sync 驗 `release/cycleXX_*` glob 與 Dev `delivery_report.md §release_tag` 一致；R-team verification 用同一 source |

**Cycle 03-27 close 後 C-C4 reconciled value (v0.1.1 original assertion — superseded by v0.1.2)**: ~~**18**（v0.50, 0.51, 0.51.1, 0.52, 0.53, 0.54, 0.54.1, 0.55, 0.55.1~5, 0.56, 0.57, 0.57.1, 0.57.2, **v0.57.4** = 18; v0.57.3-alpha 不存在故不算）。~~ — **見 §3.4.1 v0.1.2 retroactive reconcile**

**§75.X 5th-flagging 結案 (v0.1.1 partial; v0.1.2 final)**: Dev 算法為 canonical truth；R-team v0.1 algorithm reading 因「release tag」歧義而誤算；本 §3.4 釐清後 forward 不再產生 discrepancy。

### §3.4.1 v0.1.2 Retroactive Reconcile — v0.57.3-alpha 補齊後 C-C4 真實值（mechanical correction 2026-05-12）

**觸發事件**: 2026-05-12 Master directive — cycle 03-26 release dispatch 漏派經 audit 發現；coordinator dispatch Dev task #190 補做 v0.57.3-alpha doc-only patch retroactive；release dir `release/cycle03-26_v0.57.3-alpha/` + delivery_report.md + 342-canonical-snapshot 全部 packaged complete 2026-05-12T08:15Z。

**v0.1.1 §3.4 reconcile assumption invalidated**: v0.1.1 §3.4 assumed「v0.57.3-alpha 不存在」基於當時的 silent failure（cycle 03-26 release dispatch 漏派 = 沒派 = 假設沒有）。實際 root cause 為「coordinator 漏派 = bug」而非「不該派」。Bug 補齊後 v0.57.3-alpha 確實存在，符合 §3.4 release tag 充要條件（`release/cycle03-26_v0.57.3-alpha/` 目錄實體存在 + delivery_report.md 含 §75.X enforce evidence）。

**Reference/21 §3.4 release tag 精確定義本身保留 unchanged**（規則沒錯；錯的是當時 v0.57.3-alpha 真的不存在的事實）。本 §3.4.1 為 mechanical reconcile 因應補做後事實變更。

**Cycle 03-27 close + cycle 03-26 retroactive backfill 後 C-C4 reconciled value (v0.1.2 final)**: **19** — release tag 全列表（v0.50.0-alpha 起按 cut 時序）：

| # | Release tag | Cycle | Cut date | §75.X enforce |
|:-:|-------------|-------|----------|:--------------:|
| 1 | v0.50.0-alpha | 03-14 | — | ✅ |
| 2 | v0.51.0-alpha | 03-15 | — | ✅ |
| 3 | v0.51.1-alpha | 03-16 | — | ✅ |
| 4 | v0.52.0-alpha | 03-17 | — | ✅ |
| 5 | v0.53.0-alpha | 03-18 | — | ✅ |
| 6 | v0.54.0-alpha | 03-19 | 2026-05-01 | ✅ |
| 7 | v0.54.1-alpha | 03-20 | 2026-05-02 | ✅ doc-only |
| 8 | v0.55.0-alpha | 03-21 | 2026-05-03 | ✅ |
| 9 | v0.55.1-alpha | 03-21 | 2026-05-03 | ✅ hotfix |
| 10 | v0.55.2-alpha | 03-21 | 2026-05-03 | ✅ hotfix |
| 11 | v0.55.3-alpha | 03-21 | 2026-05-03 | ✅ NEW plugin |
| 12 | v0.55.4-alpha | 03-21 | 2026-05-03 | ✅ hotfix |
| 13 | v0.55.5-alpha | 03-21 | 2026-05-03 | ✅ hotfix |
| 14 | v0.56.0-alpha | 03-22 | 2026-05-04 | ✅ |
| 15 | v0.57.0-alpha | 03-23 | 2026-05-05 | ✅ |
| 16 | v0.57.1-alpha | 03-24 | 2026-05-07 | ✅ |
| 17 | v0.57.2-alpha | 03-25 | 2026-05-07 | ✅ |
| **18** | **v0.57.3-alpha** | **03-26** | **2026-05-12 retroactive** | ✅ doc-only |
| **19** | **v0.57.4-alpha** | **03-27** | **2026-05-11** | ✅ hygiene fix |

**Cycle 03-27 close + retroactive backfill 後 reconciled values**:
- **C-C4 = 19** (was v0.1.1 incorrectly claimed 18)
- **C-C4a = 14** (cycle 03-26 + cycle 03-27 each contributed 1; total cycles with ≥1 release tag = 14)
- **C-C4b = 4 sustained** (no spot-check change)

**Forward-binding cycle 03-28 起**: §3.4 release tag 精確定義 + §3.4.1 retroactive reconcile precedent。Future cycle close 必須驗證 release dispatch 完成 (per §3.4 充要條件) 才能 close cycle；coordinator G5 stage 加 release-dispatch-completion check (per Master directive 2026-05-12 implied)。

### §3.6 Mirror Discipline — Single-Target Rule（v0.1.3 amendment 2026-05-12；resolves Master directive 2026-05-08 真正落地後 mirror 範圍歧義）

**觸發事件**: 2026-05-12 Master directive 2026-05-08 真正落地（Dev task #191 — `openstarry/docs/` 從 4 個 location 移除：agent_dev + cycle03-26 release + cycle03-27 release + test codebase）。落地後 coordinator G5 mirror discipline 須 codify「不再 mirror canonical doc 到 `openstarry/docs/canonical/`」單一目標規則，否則下次 G5 sync 又會 propagate 違規。

**根因分析**: cycle 03-27 close coordinator G5 mirror Reference/21 v0.1.1 時，把 canonical doc copy 到 13 個 location 包含 `openstarry/docs/canonical/Reference/21_Counter_Registry.md`（4 個 release / agent_dev / test codebase）。這些 path 本來就違反 directive 2026-05-08 §1（release `openstarry/` 不該有 `docs/`）。Dev cleanup #191 移除整個 `docs/` 等於同時刪了 coordinator 的違規 mirror copy（無內容流失，本應如此），但若無單一目標規則，下次 G5 又會自動 propagate violation。

**Resolution — Single-Target Mirror Rule**（forward-binding cycle 03-28 起）：

| Mirror discipline | 規則 |
|------|------|
| **單一目標 canonical 路徑** | Canonical doc 唯一鏡同步目標 = `openstarry_doc/`（不論在 share/ / agent_dev/ / release/ / test codebase/ / research_version/ 哪個 mirror tree 中）。**不再 mirror 到 `openstarry/docs/canonical/`** |
| **核心 mirror set** | `share/openstarry_doc/`（canonical authority） + `agent_dev/openstarry_doc/`（Dev sync target） + `release/cycleXX_*/openstarry_doc/`（per release snapshot） + `test research/cycleXX/codebase_*/openstarry_doc/`（test codebase snapshot） + `research input/research_version/cycleXX_*/openstarry_doc/`（R-team input snapshot） + `test research/cycleXX/research_reference/`（R-team reference flat copy） |
| **典型 mirror count** | 6-9 locations（依 cycle artefact 範圍；單一 Reference doc 可能 6-7 mirrors；canonical-wide CHANGELOG 可能 9 mirrors） |
| **絕對禁止** | mirror canonical doc 到 `openstarry/docs/canonical/`（per directive 2026-05-08；該 path 不應存在） |
| **驗證 protocol** | Coordinator G5 完成後 `find . -path '*openstarry/docs/canonical/Reference/*'` 應 return empty（任何 release / agent_dev / test codebase 內）；若 non-empty → violation → 移除 + audit dispatch source |

**Pre-2026-05-12 既有 release 不溯及修改**（per MR-12）：
- cycle 03-25 v0.57.2-alpha 及之前 release 含 `openstarry/docs/canonical/` 為歷史 snapshot，不動
- cycle 03-26 v0.57.3-alpha + cycle 03-27 v0.57.4-alpha 為 post-directive cycles，已 in-place cleanup（task #191）使其符合新規

**Mirror count 變化**:
- v0.1.1/v0.1.2 sync 時錯誤 mirror = 13 locations
- v0.1.3 sync 時 corrected mirror = 9 locations（移除 4 個 `openstarry/docs/canonical/Reference/` path）

### §3.5 Hygiene-only Cycle Counter Increment Rule（v0.1.1 amendment；resolves C-C6/C-C7 governance interpretation）

**觸發事件**: cycle 03-27 R-team verification report §7 F-CY27-§R-V-08 flagged C-C6 (/simplify) + C-C7 (subagent permanent-mode) counter increment ambiguity — cycle 03-27 為 hygiene-only fix cycle（無 R0-R3 vote、無 full R4 G4 gate、無 full subagent batch parallel）。

**根因分析**: Reference/21 v0.1 §2.2 C-C6/C-C7 row 之 update trigger 假設 cycle 含完整 R0-R3 + R4 G4 gate + subagent permanent-mode batch；hygiene-only cycle scope 縮減未明文涵蓋。

**Resolution**: Hygiene-only cycle counter increment 規則（forward-binding cycle 03-27 close 起）：

| 規則 | 內容 |
|------|------|
| **Default = deferred** | Hygiene-only cycle 對需「該 counter 對應動作實際發生」的 counter（C-C6 / C-C7 / C-C2 amendment apply / C-C1 4 防線 / C-C3 isolation sustained 等）默認 **不 +1**，保持 sustained 值；除非該動作在此 hygiene cycle 確實發生（per Dev `delivery_report.md` 或 R-team verification report 明文紀錄） |
| **Always-tracked counters** | C-S1 (HEALTHY)、C-S4 (F1/F2/F3 zero)、C-S6 (correction rounds 0)、C-U1 (GREEN cells)、C-Q2 (W2-Rn)、C-Q4 (release tag) 等 **依舊 +1**（不論 hygiene/full cycle；只要 W-round + release tag 出現即 +1） |
| **§75.X enforce (C-C4)** | **依舊 +1**（hygiene cycle 出 release tag → +1；hygiene cycle 不出 release tag → 0；本規則同 §3.4 release tag 精確定義） |
| **Hygiene cycle definition** | (1) 無 R0-R3 vote（無 R3_decision_log generation） (2) 無 full subagent permanent-mode batch（partial subagent or main session only） (3) Master directive 明文歸類為「hygiene-only fix cycle」or「post-audit fix cycle」(per Master directive 2026-05-09 §3.1 PASS path branch) |
| **Verification protocol** | Hygiene cycle counter snapshot 段須註明「（hygiene-only cycle deferred-by-default per §3.5）」字樣，與 full cycle snapshot 區分 |

**Cycle 03-27 close 後 C-C6/C-C7 reconciled value**:
- **C-C6 = 12 sustained**（cycle 03-26 R4 close = 12；cycle 03-27 hygiene + Dev `delivery_report.md §7.4` 無 organic /simplify → deferred → sustained）
- **C-C7 = 12 sustained**（cycle 03-26 R4 close = 12；cycle 03-27 hygiene + 無 full subagent batch → deferred → sustained）

**Cycle 03-28+ reform cycle counter increment**: Reform cycle 為 full cycle（含 R0-R3 vote），applies §2.2 default rule (+1)；hygiene cycle deferred-by-default 規則僅適用 hygiene scope。

### §3.7 Counter Sunset Clause (v0.2 substance addition 2026-05-13 — Master Ratification Batch 24 Item #7 ratified per cycle 03-28 R3 D-§B.1 22/1)

**Forward-binding cycle 03-29+** (post Master Confirmation 2026-05-13):

A counter (`C-S*` / `C-C*` / `C-U*` / `C-T*` / `C-E*` / `C-Q*`) becomes **sunset-eligible** if any of:
- **(a)** 5+ consecutive cycles unchanged AND unreferenced in any new dispatch / Master_Confirmation / R3_decision_log
- **(b)** explicitly superseded by new counter (e.g., cycle 03-30+ replaces C-Sn with C-Sn-v2)
- **(c)** reform-cycle consolidation (Master directive or R3 ratify)

**Sunset workflow**:
1. Sunset proposal at next R3 D-§ (substantive vote) OR coordinator G5 (mechanical correction per Master directive 2026-05-09 §2.3 condition 2)
2. R3 vote (substantive) OR coordinator G5 attestation (mechanical)
3. If ratified:
   - status enum → `SUNSET` (C-E3 extension; per §2.5 C-E3 SUNSET added value)
   - Move counter definition to `Reference/21_archive/sunset_counters/` (forward-only per MR-12; existing references unchanged)
   - Update cross-refs in dispatching docs (CHANGELOG, R3_decision_log, Master_Confirmation)
   - Preserve history (no delete; archive)

**Non-sunsetable classes** (NEVER eligible per project axiom):
- **C-U2 (cumulative DSS)**: NEVER sunsetable per **MR-11** (dissent preservation forever)
- **ZT-related counters** (e.g., if added in future): require Master explicit ratification per ZT-special rule
- **HISTORIC trophy counters** (current value = HISTORIC longest record): Sunset proposal must justify why HISTORIC milestone replaced/superseded; high bar

**DSS-CY28-§B.1 verbatim** (preserved per MR-11; relevant to this clause):

> Dormancy threshold should be uniform 5-cycle across all classes (Rule + MR + Plan + Counter), NOT differentiated 5/7/3/5. MR's "higher authority weight" rationale for 7-cycle conservative dormancy is unfounded; differentiated thresholds add governance complexity without operational justification.

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
6. **§75.X 5th-flagging (Dev=18 vs R-team=19)**: ✅ Resolved per §3.4 (v0.1.1 amendment 2026-05-11 cycle 03-27 close); release tag 精確定義 = `release/cycleXX_*` 目錄實體存在；~~C-C4 reconciled = 18 (Dev canonical algorithm)~~ — superseded by §3.4.1 v0.1.2 retroactive reconcile (C-C4 = 19 after v0.57.3-alpha cycle 03-26 retroactive packaging 2026-05-12)
7. **C-C6/C-C7 hygiene cycle counter increment ambiguity**: ✅ Resolved per §3.5 (v0.1.1 amendment 2026-05-11 cycle 03-27 close); hygiene-only cycle counter increment = deferred-by-default; cycle 03-27 C-C6/C-C7 = 12 sustained (no organic action)
8. **Cycle 03-26 release dispatch missed at the time → retroactive packaging 2026-05-12**: ✅ Resolved per §3.4.1 (v0.1.2 retroactive reconcile; coordinator dispatch Dev task #190 → release/cycle03-26_v0.57.3-alpha/ doc-only patch packaged complete; C-C4 = 19; v0.57.3-alpha = 18th, v0.57.4-alpha = 19th); §3.4 rule unchanged (rule was correct; v0.1.1 reconciled value was based on then-true silent-failure absence)
9. **Master directive 2026-05-08 release structure simplification 真正落地 (`openstarry/docs/` 移除)**: ✅ Resolved per §3.6 (v0.1.3 mirror discipline post-cleanup; coordinator dispatch Dev task #191 → 4 location 整個 `openstarry/docs/` 移除：agent_dev + cycle03-26 release + cycle03-27 release + test codebase; agent_dev pnpm build/test/purity PASS; ε-surface 8-cycle preservation; sibling-naming-check.mjs retarget Rule #62 root-cause fix bundled; mirror count 13 → 9)

---

---

## §9 C-LEIBNIZ-N2-observation NEW Counter (v0.2.1 Substance Addition per Master Ratification Batch 25 Item #7)

### §9.1 Counter Definition

| Field | Value |
|-------|-------|
| **ID** | C-LEIBNIZ-N2-observation |
| **Name** | LEIBNIZ DSS-CY25-§13-A N=2 ceiling at-cells observation list |
| **Type** | State counter (current state: dynamic membership of cells at-ceiling) |
| **Definition** | 當前 verdict=`conditional_pass` AND `roll_forward_count=2` AND under Reference/20 v2 §5.2 Tier-4 + LEIBNIZ DSS-CY25-§13-A N=2 stricter reading 的 cell list |
| **Source of truth** | per-cell `verdict.yaml` + Reference/20 v2 §5.2 (Tier-4 roll-forward rule) + LEIBNIZ DSS-CY25-§13-A (N=2 stricter sustained reading per MR-11) |
| **Update trigger** | R4 close per-cycle attestation; cells entering (CP at roll=2) or exiting (FP / FAIL / N/A) ceiling state |
| **Reset condition** | Cell transitions to `full_pass` / `fail` / `not_applicable` → exits list; roll_forward_count resets to 0; cell removed from C-LEIBNIZ-N2-observation snapshot |
| **Current value (cycle 03-30 close)** | **2 cells**: (a) `p10_t6` distributed-alaya × Tenet#6 (multi-host caveat material + LWW L2 test gap) + (b) `p13_t6` guide-character-init × Tenet#6 (L2 plugin-local test gap material) |
| **First established** | cycle 03-30 (per R3 D-§A1.4-P2 Option γ Hybrid ratification 2026-05-13 + Master Ratification Batch 25 Item #7) |
| **Disambiguation** | 與 C-S1 (HEALTHY consecutive — cycle-level) 不同：本 counter 為 cell-level；與 C-C7 (subagent permanent-mode — process) 不同：本 counter 為 verdict-trajectory；與 C-U2 (cumulative DSS — never reset) 不同：本 counter 為 dynamic membership state |

### §9.2 Option γ Hybrid Forward Strategy Anchor

Per §3.8 (new section ratified Batch 25 Item #7; see §3.8 below for details), cells in C-LEIBNIZ-N2-observation list must follow:

- **Option β default** (sustain CP with sunset trigger): cells may sustain CP a third time → sunset clause activates → cell-level reconsideration + Reference/20 v2 §5.2 Tier 4 amendment proposal trigger
- **Option α flag** (forced escalate; FP via fix-scope or FAIL): if specific evidence emerges (fix scope completion makes FP achievable) → cycle 03-31 forced escalate
- **Forward-binding cycle 03-31+**: per-cell observation per R4 attestation; coordinator G5 sync updates list

### §9.3 Cycle 03-30 R4 close snapshot

```yaml
C-LEIBNIZ-N2-observation:
  cycle: "03-30"
  count: 2
  cells:
    - cell_id: p10_t6
      plugin: distributed-alaya
      tenet_axis: "#6"
      roll_forward_count: 2
      caveats:
        - "multi-host genuine-interleaving evidence requires cycle 03-31+ multi-host harness"
        - "LWW L2 plugin test gap: test_alaya_t6_blackboard_race_last_write_wins absent"
        - "tier-3 inline-contract real-LLM vs tier-4 full-daemon"
      forward_strategy: "Option γ Hybrid — Option β default (cycle 03-31 may sustain CP) + Option α flag if multi-host harness FP-achievable"
    - cell_id: p13_t6
      plugin: guide-character-init
      tenet_axis: "#6"
      roll_forward_count: 2
      caveats:
        - "L2 plugin-local test gap material (no own __tests__/ directory; cross-plugin attestation only at SHALL_REVIEWED tier)"
        - "tier-3 inline-contract real-LLM vs tier-4 full-daemon"
      forward_strategy: "Option γ Hybrid — Option β default + Option α flag if FIX-A test scope + FIX-B silent overwrite mitigation FP-achievable"
```

---

## §3.8 Option γ Hybrid Forward Strategy (v0.2.1 BINDING per Master Ratification Batch 25 Item #7)

### §3.8.1 Motion

For cells in C-LEIBNIZ-N2-observation list (currently 2: p10_t6 + p13_t6), forward-binding cycle 03-31+ follows Option γ Hybrid:

1. **Option β default applies** (sustain CP with explicit sunset trigger):
   - Cycle 03-31 may sustain CP a third time without forced escalation
   - If sustained → sunset clause TRIGGERS for cell-level reconsideration
   - Sunset clause activation creates Reference/20 v2 §5.2 Tier 4 amendment proposal trigger (defer to reform cycle)

2. **Option α flag applies if specific evidence emerges**:
   - For p10_t6: multi-host harness deployment OR DSS-CY26-§A.3-3 path (c) cross-cell carry resolution → FP-achievable → forced escalate to FP-or-FAIL
   - For p13_t6: FIX-A test scope expansion (`__tests__/guide-character-init.test.ts`) + FIX-B silent guideId overwrite mitigation (`guide-registry.ts`) → FP-achievable → forced escalate to FP-or-FAIL

3. **Cycle 03-31 dispatch consideration**: coordinator + R-team R0 jointly decide per-cell strategy (Option β default vs Option α flag); record in `R0/R0_orientation.md` §LEIBNIZ-N2-observation block

### §3.8.2 Dissent preserved per MR-11

**DSS-CY30-§A1.4-P2-α** (LEIBNIZ + KNUTH; 2 votes NO; preferred Option α strict):
> LEIBNIZ + KNUTH 主張嚴格 Option α: "1 cycle grace; 2nd cycle is final" 之 DSS-CY25-§13-A 原本邏輯；三個 cycle 的 caveat-drift 等於 sprint-window 無法接受；Option γ Hybrid 之 Option β default 等於放寬 N=2 為 N=3 implicit。Master Ratification Batch 25 ratify Option γ Hybrid 21/2 super-majority；本 dissent 保留 per MR-11；cycle 03-31 R4 + cycle 03-32 R4 close 階段觀察是否 evidence supports Option β default 持續適用 vs sunset 觸發。

### §3.8.3 Cross-references

- Reference/20 v2 §5.2 (Tier-4 roll-forward rule)
- Reference/20 v2 §7 (LEIBNIZ DSS-CY25-§13-A N=2 verbatim preservation)
- cycle 03-30 R3 D-§A1.4-P2 (vote tally 21/2 → Master Ratification Batch 25 Item #7)
- C-LEIBNIZ-N2-observation §9 (state counter)

---

---

## §3.9 Evidence-Tier Ladder Canonical (v0.2.2 BINDING per Master Ratification Batch 26 Item #18)

### §3.9.1 Tier-Tier definitions (Phase 7 verification evidence hierarchy)

| Tier | Name | Description | First cycle | Example |
|:----:|------|-------------|:----------:|---------|
| **Tier 1** | text-only L4 | doc citation + source structural review;no runtime evidence | cycle 03-26 baseline | cycle 03-26 §B.6 §B.8 column-wide CP entries |
| **Tier 2** | designed-not-executed | scenario design + synthetic-placeholder trace；no actual API/runtime invocation | cycle 03-30 R1 Phase 1 | cycle 03-30 R1 5-cell first pass synthetic-marked trace |
| **Tier 3** | inline-contract real-LLM | real LLM API calls + plugin contract replicated inline (no full daemon) | cycle 03-30 R1 Phase 2 (Test team Stream 2 method） | cycle 03-30 Test team `live_traces/` 5 cell + cycle 03-31 Test team Stream 2 multi-host |
| **Tier 4** | full-daemon real-LLM | real LLM API + `pnpm install + pnpm build + daemon start` actual OpenStarry runtime via apps/runner | cycle 03-31 Test team Stream 1 cell-1 inaugural | cycle 03-31 `test research/cycle03-31/live_traces_tier4/T6-§24-C1_provider-claude-cli/` |

### §3.9.2 Tier transition rules

- Tier-1 → Tier-2 via scenario design exercise (cycle 03-30 R1 Phase 1 pattern)
- Tier-2 → Tier-3 via Test team live execute (cycle 03-30 R1 Phase 2 + Test team Stream 2 multi-host pattern)
- Tier-3 → Tier-4 via Test team full-daemon execute (cycle 03-31 Stream 1 pattern)
- Tier-4 → next: real production traces (cycle 04+ candidate)
- Per Reference/20 v3 §11 + cycle 03-30 R3 §1.1 Option B RELOCATE: tier-3 + tier-4 caveats are forward-binding-clarifications (NOT evidence-gaps); placement = `notes:` block;not `caveats[]`

### §3.9.3 Per-cell tier tracking (per Reference/20 v2/v3 §3 schema extension forward)

Verdict.yaml `notes:` block SHOULD include `evidence_tier:` field (e.g. `evidence_tier: 3` or `evidence_tier: 4`) for Phase 7 forward auditability;defer to Reference/20 v3 §3 sub-amendment (or v3 → v4 candidate per cycle 03-32+).

---

## §10 C-C8 NEW Counter (v0.2.2 Substance Addition per Master Ratification Batch 26 Item #18)

### §10.1 Counter Definition

| Field | Value |
|-------|-------|
| **ID** | C-C8 |
| **Name** | Phase 7 T4 cycle count |
| **Type** | Cardinality counter (forward-only) |
| **Definition** | 累計 cycles in which Tier-4 (full-daemon real-LLM) execution is achieved by Test team OR equivalent infrastructure |
| **Source of truth** | Test team `test research/cycle{XX}/live_traces_tier4/` directory existence + at least 1 cell execution_report.md PASS attestation |
| **Update trigger** | R4 close attestation;1+ cell Tier-4 PASS in current cycle → C-C8 += 1 |
| **Reset condition** | N/A (cardinality forward-only) |
| **Current value (cycle 03-33 close)** | **3** (cycle 03-31 inaugural + cycle 03-32 second cycle + cycle 03-33 third consecutive cycle per cycle 03-33 R3 §4.5 D-§A6.5 Test team Stream C tier-4 PASS attestation;Master Ratification Batch 28 forward dispatch routing) |
| **Previous value (cycle 03-32 close)** | 2 (cycle 03-31 inaugural + cycle 03-32 Stream A 4 cells tier-4 + Stream B 2 cells tier-4 per Master Ratification Batch 27 Item #12 ratified 2026-05-14) |
| **Previous value (cycle 03-31 close)** | 1 inaugural (Test team Stream 1 cell-1 T6-§24-C1 provider-claude-cli tier-4 PASS) |
| **First established** | cycle 03-31 (per R3 D-§A6.5 ratification + Master Ratification Batch 26 Item #18) |
| **Disambiguation** | 與 C-C7 (subagent permanent-mode) 不同：本 counter 為 evidence-tier ladder progression；與 C-S4 (F1F2F3 Popperian zero) 不同：本 counter 為 cycle-level cardinality NOT streak |

### §10.2 Cycle 03-31 close snapshot

```yaml
C-C8:
  cycle: "03-31"
  count: 1
  inaugural_cycle: "03-31"
  inaugural_evidence:
    cell: T6-§24-C1
    plugin: provider-claude-cli
    test_team_artifact: "test research/cycle03-31/live_traces_tier4/T6-§24-C1_provider-claude-cli/execution_report.md"
    test_team_attestation: "PASS — tier-4 evidence captured via real plugin loader path"
    notable_findings:
      - "inline-replication caveat (cycle 03-30 dispatch sandbox limit) REMOVED at tier-4"
      - "C-S4 10th Popperian milestone candidate sustained at tier-4 (F1/F2/F3 = 0/0/0)"
      - "3 P3 LOW/INFO findings (mapStreamEvent quirks + workspace symlink) NOT FP-blocking"
```

### §10.2.2 Cycle 03-33 close snapshot (v0.2.3 substance addition; cycle 03-33 R3 §4.5 D-§A6.5)

```yaml
C-C8:
  cycle: "03-33"
  count: 3
  inaugural_cycle: "03-31"
  cycle_03-31_close: 1   # inaugural via Stream 1 cell-1 T6-§24-C1 provider-claude-cli
  cycle_03-32_close: 2   # Stream A 4 cells tier-4 + Stream B 2 cells tier-4
  cycle_03-33_close: 3   # Stream C tier-4 PASS attestation (third consecutive cycle)
  third_cycle_evidence:
    cycle: "03-33"
    test_team_artifact: "test research/cycle03-33/live_traces_tier4/"
    test_team_attestation: "PASS — Stream C combined tier-4 execution (Stream B continuation cycle 03-31 cells T8-§41/§43/§24 + W2-R42 v0.57.9-alpha verification)"
    cells_executed:
      - "T8-§41 volition: forward FP sustained + tier-4 strengthening"
      - "T8-§43 workflow-engine: CP sustained + bridging caveat material (FIX-CY34-WORKFLOW unified scope cycle 03-34+)"
      - "T8-§24 provider-claude-cli: forward FP cross-Tenet anchor FIRST + tier-4 strengthening (provider × {T6+T7+T8} all FP at tier-4)"
    cross_tenet_anchor_at_tier_4:
      provider: "T6+T7+T8 all FP at tier-4 (cross-Tenet anchor FIRST cycle 03-31 + cycle 03-33 tier-4 strengthened)"
    notable_findings:
      - "C-S4 12th Popperian zero milestone candidate sustained at tier-4 (F1/F2/F3 = 0/0/0 across 36/36 instances 12 cycles)"
      - "Stream C tier-4 PASS validates cycle 03-31 cells under tier-4 (carryover-from-prior-cycle inheritance at tier-4)"
      - "Cross-Tenet anchor pattern provider × {#6, #7, #8} all FP at tier-4 — N=3 empirical validation at tier-4 evidence ladder"
  cumulative_evidence_count: "3 cycles × 1+ cells = ≥3 tier-4 cell-cycles cumulative (cycle 03-31 = 1 + cycle 03-32 = 6 + cycle 03-33 = ≥3)"
```

**Three-cycle consecutive C-C8 increment HISTORIC** (cycle 03-31 inaugural + cycle 03-32 + cycle 03-33): tier-4 evidence ladder progression sustains as the **default evidence tier** for Phase 7 cell verification per cycle 03-32 R0 ratification. Sustained 3-cycle increment validates §3.9 evidence-tier ladder canonical (tier-3 inline-contract → tier-4 full-daemon real-LLM promotion path mechanically reproducible).

### §10.3 Forward observation (v0.2.3 expansion)

**Cycle 03-34+ dual-counter forward observation** (NOT BINDING this cycle; observational guidance only):
1. **C-C8 cardinality forward**: Cycle 03-34+ Path 2 mixed continuation OR Tenet column pivot per R0 chair likely; each cycle with 1+ cell Tier-4 PASS increments C-C8. Expected sustained ≥1 cell tier-4 per cycle given tier-4 default since cycle 03-32 R0; observation: streak break if tier-4 default deprecated or Test team infrastructure regression.
2. **Tenet#8 column closure progress paired observation**: Cycle 03-33 close = 9/38 candidate (6/38 → 9/38 +3 mid-range Path 2 pacing). Cumulative cross-Tenet anchor pattern N=3 empirical validation reached (provider FIRST + alaya SECOND + api-runtime THIRD); cycle 03-34+ SIBLING substrate-invariance pair pattern (replay-cache + alaya) forward observation candidate per Reference/20 v4 forward amendment scope mark (CY33-NOTE-4).
3. **Substrate-class coverage forward observation**: Cycle 03-33 inaugurated 識蘊 Vijnana structural-observability class (api-runtime;first non-consciousness-stream substrate). Cycle 03-34+ candidate substrate classes for column closure: Vijnana-substrate continuation OR consciousness-stream-substrate SIBLING pairs (cycle 03-32 ratified pattern extension).

**Dual-counter forward observation status**: **NOT BINDING this cycle** per cycle 03-33 R3 §4.5 D-§A6.5 ratification scope (counter value update + snapshot block + forward outlook expansion only; cross-counter cardinality interaction NOT codified BINDING). Forward BINDING amendment routing: Reference/20 v4 forward amendment cycle 03-34+ per Master Ratification Batch 28 dispatch.

**Sunset clause forward**: C-C8 cardinality forward-only (no reset / no sunset); each cycle attestation cumulative.

---

## §9.4 C-LEIBNIZ-N2-observation Cycle 03-31 Close Update (per Master Ratification Batch 26 Items #14-#16)

Per cycle 03-31 R3 D-§A6.1 (p10_t6 Option α activate via Stream 2 multi-host evidence) + D-§A6.2 (p13_t6 Option α activate via Stream 3 FIX-A/B evidence + L2 gap CLOSED) + D-§A6.3 (window 雙 CLOSED):

**Cycle 03-31 close snapshot**:
```yaml
C-LEIBNIZ-N2-observation:
  cycle: "03-31"
  count: 0
  cells: []   # window CLOSED; both p10_t6 + p13_t6 exited via Option α activate CP→FP transition
  cycle_03-30_close: 2 cells (p10_t6 + p13_t6 at-ceiling)
  cycle_03-31_close: 0 cells (window 雙 CLOSED)
  sunset_clause_triggered: false   # DSS-CY30-§A1.4-P2-α (LEIBNIZ + KNUTH preferred Option α strict) sunset clause NOT triggered;Option γ Hybrid validated via 2 cells successfully exiting via Option α activate
  forward_observation: cycle 03-32+ no cells in observation window;forward state = empty unless new cells enter at roll=2 ceiling
```

**DSS-CY30-§A1.4-P2-α status update** (per MR-11 verbatim preserved): LEIBNIZ + KNUTH cycle 03-30 R3 D-§A1.4-P2 preferred Option α strict for both p10_t6 + p13_t6. Cycle 03-31 R3 D-§A6.1 + §A6.2 activated Option α (not Option β default) for both cells via specific evidence emergence — empirically supports LEIBNIZ + KNUTH original strict preference (evidence-anchored escalation > 3-cycle CP drift). Option γ Hybrid validated as flexible umbrella absorbing both Option β default + Option α specific-evidence escalation.

---

*Reference/21 — Counter Registry — CANDIDATE v0.2.3 cycle 03-33 R4 close 2026-05-15 (Master Ratification Batch 28 forward dispatch) — RATIFIED v0.2.2 — 2026-05-14 (v0.2.1 ratified Batch 25 sign-off 2026-05-13; v0.2.2 ratified Batch 26 sign-off 2026-05-14)*
*Single Source of Truth for All Project Counters*
*Authority: Master directive 2026-05-09 §2.3 condition 2 (mechanical correction) + Master directive 2026-05-12 (cycle 03-26 release dispatch补做 + Master directive 2026-05-08 真正落地) + cycle 03-28 R3 D-§A0 + cycle 03-30 R3 D-§A1.4-P2 (Master Ratification Batch 25 Item #7)*
*Effective cycle 03-26 R0 起 (v0.1) + §3.4/§3.5 cycle 03-27 close 起 (v0.1.1) + §3.4.1 cycle 03-26 retroactive reconcile 2026-05-12 起 (v0.1.2) + §3.6 mirror discipline single-target rule 2026-05-12 起 (v0.1.3) + §2.5 C-E1 phantom doc 51 removal cycle 03-28 R4 close 2026-05-12 起 (v0.1.4) + §3.7 + C-S1-roll12 cycle 03-28 Batch 24 sign-off 2026-05-13 起 (v0.2) + §9 C-LEIBNIZ-N2-observation + §3.8 Option γ Hybrid cycle 03-30 Batch 25 sign-off 2026-05-13 起 (v0.2.1)*
*First application: §75.X 三邊釐清 (C-C4 = 17 canonical; C-C4a = 12 per-cycle; C-C4b = 4 spot-check)*
*v0.1.1 amendment: §3.4 release tag 精確定義 + §3.5 hygiene-only cycle counter increment rule*
*v0.1.2 amendment: §3.4.1 retroactive reconcile after cycle 03-26 v0.57.3-alpha 补做 (C-C4 = 19 final)*
*v0.1.3 amendment: §3.6 mirror discipline single-target rule (canonical 鏡同步唯一目標 = `openstarry_doc/`)*
*v0.1.4 amendment: §2.5 C-E1 phantom doc 51 removal (canonical authority = README §十大核心宣言 sole)*
*v0.2 amendment: §3.7 Counter Sunset Clause + C-S1-roll12 NEW counter (Batch 24)*
*v0.2.1 amendment: §9 C-LEIBNIZ-N2-observation NEW state counter + §3.8 Option γ Hybrid forward strategy (Batch 25 cycle 03-30 R3 D-§A1.4-P2)*
*v0.2.2 amendment: §10 C-C8 NEW counter Phase 7 T4 cycle count + §3.9 evidence-tier ladder canonical + §9.4 cycle 03-31 close window 雙 CLOSED (Batch 26 cycle 03-31 R3 D-§A6.5)*
*v0.2.3 amendment (CANDIDATE): §10.1 C-C8 cycle 03-33 close 2 → 3 + §10.2.2 NEW cycle 03-33 snapshot block + §10.3 dual-counter forward observation expansion (NOT BINDING this cycle); Master Ratification Batch 28 forward dispatch (cycle 03-33 R3 §4.5 D-§A6.5)*
*25+ counters codified across 6 categories: streak / cardinality / cumulative / state / status / sequence; C-LEIBNIZ-N2-observation = state counter (cell-level membership)*
