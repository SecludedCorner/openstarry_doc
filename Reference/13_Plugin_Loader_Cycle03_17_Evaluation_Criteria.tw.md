---
title: plugin-loader Cycle 03-17 5 準則 + 5×4 決策矩陣評估框架
title_en_sibling: plugin-loader Cycle 03-17 5-Criterion + 5×4 Decision Matrix Evaluation Framework
author: SCRIBE + LINNAEUS (TW translation backfill cycle 03-20)
date: 2026-05-02
cycle: 03-20 (TW backfill per D-§5 ratified + Batch 17 #4 APPROVED)
status: BINDING (TW sibling parity per Rule #78 §78.5 BINDING-tier reflexive)
authority: research-team R4 deliver (cycle 03-20)
supersedes: none (forward-only addendum to cycle 03-16 EN)
language: zh-TW
cross_refs:
  - openstarry_eco/share/openstarry_doc/Reference/13_Plugin_Loader_Cycle03_17_Evaluation_Criteria.md (EN sibling)
  - research record/cycle03-20/openstarry_doc/Reference/TW_backfill_list_cycle03-20.md
  - research record/cycle03-20/deliver/Master_Ratification/Batch_17_Request.md (Item #4)
  - research record/cycle03-16/deliver/O4_Plan51_retro_pluginloader_final.md §11 + §12 + §13
  - research record/cycle03-16/openstarry_doc/Technical_Specifications/Plan54_AC9_Binding.md (C1 conditioning input)
  - research record/cycle03-16/R3/R3_decision_log.md (D-02 / D-19)
  - research record/cycle03-15/openstarry_doc/Technical_Specifications/Plan51_Zod_Gate_Binding.md (Plan51 first-shipping retrospective baseline)
  - research record/cycle03-13/openstarry_doc/Technical_Specifications/Plan49_Zod_Gate_Baseline.md (MRB-06 plugin-loader 0-site greenfield origin)
hard_rule_restated: "TW sibling per Rule #78 §78.5 BINDING-tier reflexive same-PR forward backfill; MR-12 forward-only; 既有 EN sibling content 不 retrofit; this TW sibling provides accessibility parity for TW readers per cycle 03-15 §6.3."
prefix_discipline: verified | inferred | speculative
---

# plugin-loader Cycle 03-17 5 準則 + 5×4 決策矩陣評估框架

## 1. 狀態

於 cycle 03-16 R4 close 為 **CANDIDATE**。經 Master Ratification Batch 13 Item #2 升格為 **APPROVED evaluation framework**（cycle 03-17 R0 援用之程序性 binding）。

**適用範圍**：cycle 03-17 R0 援用 5 準則 + 5×4 矩陣，以判定 plugin-loader（第 5 個 Plan51 模組候選；於 cycle 03-15 D-§5-A 9/11/3 投票 DEFERRED）是否應重新進入 Plan-spec dispatch 範圍，或維持 DEFERRED。Cycle 03-21 R0 依 D-19 援用 MRB-06 雙邊處置邏輯（close-with-rationale 或 re-ratify-as-open-gap）。

## 2. R3 來源

- **D-02（Round A apex；§4 D-§4-B + F-§4-R2-07 RUSSELL agent rationality）** — 明確之 **C5 第 5 準則**（DSS-A4 procedural-caution honour clause），**18/5**。DSS-CY16-02 5 票少數依 MR-11 逐字保留（SUSSMAN + DARWIN + LINNAEUS + 2 others；row-1-amendment 應對既有 4 準則之修正優於明確 C5 — abstraction parsimony 論點）。
- **D-19（Round D；MRB-06）** — 為 cycle 03-21 R0 援用預先定位兩個選項（close-with-rationale vs re-ratify-as-open-gap 之雙邊結構），**UNANIMOUS 23/0**。
- **CV-19** — DSS-3 LEIBNIZ ossification 於 cycle 03-16 §4 D-§5-D trigger met 之 CLOSED-by-execution，**UNANIMOUS reaffirm with NAGARJUNA Madhyamaka caveat**。
- **MRB-06 RESOLVED** — cycle 03-13 Plan49 spec MRB-06 plugin-loader 0-site greenfield 之處置依 D-19 預先定位。

## 3. 背景 — 為何採 5 準則框架

### 3.1 Cycle 03-15 D-§5-A 結果（carryover）

Plan51 於 cycle 03-15 批准 5 個模組中之 4 個（D-§5-A 17/6） — WebSocket / checkpoint-store / event-bus / hook-registry 於 v0.51.0-alpha 出貨。第 5 個模組（plugin-loader）DEFERRED 9/11/3（推薦/反對/棄權）。DSS-A4 6-dissent（LEIBNIZ + KNUTH + SUSSMAN + 3 others；「Plan51 整體應 defer 至 cycle 03-17+ post-AC-9；4 模組 commit 過早」）依 MR-11 逐字保留。

### 3.2 DSS-A4 三項理由分支

DSS-A4 6 dissent 有三項理由分支：
1. **Procedural-caution branch**（「4 模組 commit 過早」） — 部分被 Plan51 first-shipping STRUCTURAL PASS at v0.51.0-alpha 反駁。
2. **Post-AC-9 conditioning branch**（「Plan51 整體應 defer 至 post-AC-9」） — 透過 C1（AC-9 Plan54 spec 穩定作為 conditioning input）操作化。
3. **Caution against bundled-decision-making**（RUSSELL R2 §2.11 implicit-third-branch 識別） — 透過 C5 明確 honour clause 操作化。

C1-C4 全 GREEN **不**自動觸發 plugin-loader resumption。C5 **無法被** C1-C4 涵蓋 — 它操作於 procedural-honour discipline 層次。

## 4. 5 項準則

### 4.1 C1 — AC-9 Plan54 spec 穩定

- 依 R3 D-01（Round A apex；Stage 2 runoff 16/7）：**AC-9 Plan54 Candidate A（full plugin Plan52 isomorph）批准**。
- Cycle 03-16 Master Ratification Batch 13 Item #1 APPROVE BINDING。
- plugin-loader 重新評估之 conditioning input。
- **C1 於 cycle 03-16 close 之狀態**：✅ AC-9 Plan54 spec 已批准；cycle 03-17 R0 評估可在 AC-9 spec 已知之條件下進行 C2/C3/C4。
- 交叉引用：`Technical_Specifications/Plan54_AC9_Binding.md` + Master Ratification Batch 13 Item #1。

### 4.2 C2 — ~70-110 LOC 預算驗證

- 依 cycle 03-15 R1 §5 估計 + cycle 03-15 O4 §7.4 pro-forma：70-110 LOC factored，中位數 ~90。
- LOW migration risk（greenfield 0-site）。
- **Cycle 03-17 R0 應重新執行 LOC 因子分析**，將 post-AC-9 surface area 折入（AC-9 是否引入需要擴張之額外 plugin-config schema fields？）。
- Cycle 03-16 4 模組 first-shipping **驗證了 per-module factoring 方法論**於結構層次。
- **R4 LOC 方法論指引適用**（依 cycle 03-16 O4 §7.5）：D-G-class binding helpers 預設較 R1 baseline +30-40%；shared dispatcher contract 較手算估計 +30%；boilerplate re-export plumbing 明確估計或明確排除為 non-counting。

### 4.3 C3 — 0-site greenfield → AC-9 後是否浮現 first call-site？

- 依 cycle 03-13 Plan49 spec MRB-06：plugin-loader 目前有 **0 個 schema-drift `safeParse()` call-site（greenfield）**。
- AC-9 Plan54 sub-agent composition spec（依 D-01 之 Candidate A full plugin）若 AC-9 要求 sub-agent spawn 時進行 plugin-level config validation，可能引入 **first call-site**。
- **Cycle 03-17 評估 MUST 判定**：AC-9 引入之 plugin-config surface 是否使 schema gate 立即正當化，或維持 deferred？
- LEIBNIZ R1 框架：plugin-loader 有最強之 **architectural-gap-closure case**；若 first call-site 浮現則 AC-9 可能將其變為 *operational-necessity-case*。

### 4.4 C4 — MR-12 既有不破壞於 migration 之驗證

- 依 cycle 03-15 O4 §9 cross-version-skew helpers MUST 先例：任何*新*的 plugin-loader schema 必須驗證 MR-12 forward-only 受到尊重。
- 若 AC-9 引入帶 version-tagging 之新 plugin-config schema，cycle 03-17 評估 MUST 驗證類似 checkpoint-store v0.42-onwards pattern 之 per-version migration helpers（cycle 03-15 Plan51 first-shipping 之 cross-version-skew helpers 6/6 PASS）。
- 若 AC-9 為純加性（新 config fields，無既有 fields rename），MR-12 forward-only by construction 受到尊重。
- CV-08 R3 reaffirm：forward-only MR-12；cycle 03-13 backfill = MR-10 一次性先例 non-invocable。
- CV-17 R3 reaffirm：MR-12 forward-only（無 retrofit）。

### 4.5 C5 — DSS-A4 procedural-caution honour clause（NEW per D-02 18/5）

**陳述**：即使 C1-C4 GREEN，cycle 03-17 R0 MUST 在恢復 plugin-loader 前明確權衡 DSS-A4 procedural-caution；明確權衡 SHALL 記錄於 cycle 03-17 R0 orientation artefact。

**理由（RUSSELL agent rationality 依 F-§4-R2-07 MED-HIGH）**：

DSS-A4 6 dissent 有三項理由分支（依上述 §3.2）：
1. Procedural-caution — 部分被 first-shipping success 於結構層次反駁。
2. Post-AC-9 conditioning — 透過 C1 操作化。
3. Caution against bundled-decision-making（DSS-A4 之 spirit） — 透過 C5 操作化。

C1-C4 全 GREEN **不**自動觸發 plugin-loader resumption。Cycle 03-17 R0 MUST **獨立權衡** DSS-A4 procedural-caution 對應 4 準則 GREEN，並將記錄之權衡 SHALL 歸檔於 cycle 03-17 R0 artefact。

### 4.6 5 準則語意優先性

| 準則 | 類型（依 RUSSELL agent rationality） | 邏輯角色 |
|-----------|--------------------------------------|--------------|
| C1 | Conditioning input | 決策必要資訊 |
| C2 | Feasibility envelope | 在 scope envelope 內 |
| C3 | Trigger condition | 行動觸發條件 |
| C4 | Constraint compliance | MR-12 既有不破壞受到尊重 |
| **C5** | **Dissent-honour cycle disposition** | **即使 C1-C4 GREEN，明確權衡 DSS-A4 procedural-caution** |

C5 **無法被** C1-C4 涵蓋。它操作於不同語意層次：C1-C4 為技術／合規準則；C5 為 procedural-honour discipline。

## 5. 5×4 決策矩陣

5×4 矩陣枚舉 5 個 cycle disposition 結果對應 4 個準則（C1-C4）。C5（DSS-A4 honour clause）作為**閘條件**調節*所有* dispositions。

| Row | C1 (AC-9) | C2 (LOC) | C3 (call-site) | C4 (MR-12) | C5 (DSS-A4) | 建議 cycle 03-17 結果 |
|:---:|:---------:|:--------:|:--------------:|:----------:|:-----------:|---------------------------------|
| **1** | AC-9 ratified ✅ | within budget | first call-site emerges | MR-12 honoured | weighing documented | cycle 03-17 R0 評估之 **Plan-spec candidate plugin-loader Zod gate** |
| **2** | AC-9 ratified ✅ | within budget | greenfield persists | (n/a; no surface) | weighing documented | **Continue DEFERRED**；於 cycle 03-18+ 重審 |
| **3** | AC-9 ratified ✅ | over-budget | (any) | (any) | weighing documented | **Continue DEFERRED**；重新考慮 scope |
| **4** | AC-9 否決 / deferred ⏳ | (n/a) | (n/a) | (n/a) | (n/a; C1 short-circuits) | **Continue DEFERRED**；於 cycle 03-18+ post-AC-9-ratification 重審 |
| **5** | AC-9 ratified ✅ | within budget | first call-site emerges | MR-12 violated ⚠️ | (n/a; C4 short-circuits) | **Continue DEFERRED**；先解決 MR-12 path |

### 5.1 KNUTH Algorithm 完整性

R1 5-row 枚舉透過 short-circuit pruning（AC-9 否決/over-budget/MR-12 violated 在剩餘下不變；greenfield 在 C4 下不變）正確覆蓋 16 組合空間（2⁴ = 16）。**無代數缺口**。

### 5.2 C5 Honour Clause 之調節

C5 明確調節 Rows 1, 2, 3：
- **Row 1**：即使 C1-C4 GREEN，cycle 03-17 R0 MUST 於建議 Plan-spec candidate 前記錄 DSS-A4 weighing。
- **Row 2**：記錄 DSS-A4 weighing 支持 DEFERRED disposition（與 DSS-A4 post-AC-9 conditioning 一致）。
- **Row 3**：記錄 DSS-A4 weighing 支持 DEFERRED disposition（與 DSS-A4 caution-against-bundled-decision-making 一致）。

Rows 4 + 5 分別於 C1 / C4 short-circuit；C5 weighing 不需要（涵蓋於技術／合規失敗下）。

### 5.3 NAGARJUNA Madhyamaka 決策耦合 caveat

5 準則 + 5×4 矩陣**不應**於 cycle 03-17 R0 機械式套用。Madhyamaka-correct：準則作為**決策支援，非決策替代**。獨立 agent reasoning + DSS-A4 honour cycle (C5) + AC-9 spec specifics（D-01 Candidate A full plugin） + 0-site call-site shape 皆條件化最終結果。

## 6. Cycle 03-16 Close C1 狀態更新（informational）

於 cycle 03-16 R3 close（2026-04-28）：
- C1 = ✅ AC-9 Plan54 Candidate A 已批准（D-01 Stage 2 runoff 16/7）。
- C2 = pending cycle 03-17 R0 LOC factor 重新分析（post-AC-9 surface）。
- C3 = pending cycle 03-17 R0 first-call-site 分析（post-AC-9 plugin-config schema 引入問題）。
- C4 = pending cycle 03-17 R0 MR-12 驗證（依 AC-9 schema additivity）。
- C5 = pending cycle 03-17 R0 DSS-A4 weighing 文件化。

**Cycle 03-17 R0 攜帶明確責任，以 C1=GREEN 作為 conditioning input 評估 C2-C5。**

## 7. Cycle 03-17 R0 援用程序

1. **C1 short-circuit check**：若 AC-9 Plan54 implementation cycle 03-17（Dev）遇到 blocker → C1 不穩定 → Continue DEFERRED（Row 4 logic）。
2. **C2 LOC factor 重新分析**：以 AC-9 surface area 重新執行 pro-forma 70-110 LOC 預算（套用 cycle 03-16 O4 §7.5 之 R4 LOC 方法論指引）。
3. **C3 first-call-site 分析**：檢視 AC-9 plugin-config schema 引入 → plugin-loader 是否自然獲得 `safeParse()` call-site？
4. **C4 MR-12 驗證**：若 C3 = first-call-site 浮現，驗證類似 checkpoint-store pattern 之 per-version migration helpers。
5. **C5 DSS-A4 weighing**：無論 C1-C4 結果，明確記錄權衡於 cycle 03-17 R0 orientation artefact。
6. **Disposition selection**：依 C2-C5 評估套用矩陣 Row 1 / Row 2 / Row 3 / Row 5；Row 4 已於 C1 穩定時 short-circuit-resolved。

## 8. MRB-06 Cycle 03-21 處置邏輯（D-19 UNANIMOUS 23/0）

若 §5-§7 5 準則 + 5×4 矩陣 cycle 03-17 評估產生「Continue DEFERRED」（Rows 2/3/4/5）且達到 cycle 03-21 soft 6-cycle floor anchor（依 cycle 03-15 O4 §13.2）：

### 8.1 Option A — Close-with-rationale

- 明確決定 plugin-loader 0-site greenfield 為無限期可接受（架構上**非**必要）。
- 記錄理由：例如 AC-9 Plan54 Candidate A surface 維持 plugin-config-orthogonal；greenfield 自然合適。
- MRB-06 trail 結束；cycle 03-13 architectural-gap acknowledgement 正式退役。

### 8.2 Option B — Re-ratify-as-open-gap

- 明確接受架構缺口持續存在並附文件化原因。
- 記錄理由：例如 AC-9 surface 引入 first-call-site 但 cycle 03-17/18/19/20 評估各因技術／合規原因產生 DEFERRED；缺口仍在。
- MRB-06 trail 以 re-ratified 狀態繼續；soft floor 下一 cycle（例如 cycle 03-27）。

### 8.3 Cycle 03-21 R0 援用程序

1. **狀態檢查**：確認 plugin-loader 於 cycle 03-21 之狀態（DEFERRED active？resumed？schema-shipped already？）。
2. **雙邊選擇**：
   - 若 AC-9 Plan54 surface 跨 cycle 03-17~03-20 維持 plugin-config-orthogonal（greenfield 自然合適），套用 **Option A**。
   - 若 AC-9 surface 引入 first-call-site 但評估各因技術／合規原因產生 DEFERRED，套用 **Option B**。
3. **記錄選擇理由**於 cycle 03-21 R0 artefact + Master 能見度項目依 Batch dispatch。
4. **MRB-06 closure 或 re-ratification** 記錄於 coordinator G5 義務依 cycle 03-15 O4 §7.3。

### 8.4 Coordinator G5 持續義務

依 cycle 03-15 O4 §7.3 + R1 F-§4-R1-10：coordinator G5 義務 per O4 §7.3 於 cycle 03-16 持續。**MRB-06（cycle 03-13 Plan49 spec）保持 open** 直至 cycle 03-21 處置。

## 9. DSS-3 Closure-by-Execution + Plan51 Sunset（CV-19 + Madhyamaka caveat）

依 cycle 03-15 R3 D-§5-D UNANIMOUS 23/0：推薦 sunset-by-execution。依 D-§5-A 推薦結果生效（cycle 03-16 實作 4-of-5 modules）。於 first-shipping 驗證：
- 4 modules **shipped** 於 v0.51.0-alpha = **execution trigger 條件達成**。
- DSS-3 carryover（cycle 03-14 LEIBNIZ ossification）**由 R3 D-§5-D 結束**；cycle 03-16 確認 execution triggered → DSS-3 closure 確認。

**NAGARJUNA Madhyamaka caveat**（依 F-§4-R2-05 + CV-19 R3 reaffirm）：DSS-3 closure **特定於 D-§5-D ratification 文字**，**非**「shipping = ossification doctrine resolved」之通則。Sunset-by-execution **case-by-case** 適用於 R3-ratified-推薦-and-implement 結果；非通則性捷徑。

**Plan51 sunset trail 狀態表**：

| Trail | Sunset 條款 | Cycle 03-16 first-shipping 之狀態 |
|-------|---------------|--------------------------------------|
| 推薦 4-of-5（D-§5-A 17/6） | sunset-by-execution per D-§5-D UNANIMOUS | ✅ TRIGGERED + CONFIRMED 於 v0.51.0-alpha |
| DEFERRED plugin-loader（D-§5-A 9/11/3） | soft 6-cycle floor anchor (cycle 03-21) | ⏳ pending；cycle 03-17 評估 5 準則 + 5×4 矩陣 |
| DSS-3（cycle 03-14 LEIBNIZ ossification） | resolved by D-§5-D UNANIMOUS | ✅ CLOSED with NAGARJUNA Madhyamaka caveat per F-§4-R2-05 + CV-19 |

## 10. DSS 保留（依 MR-11 逐字）

| DSS ID | 來源 | 票數 | 狀態 |
|--------|--------|:----:|--------|
| **DSS-A4**（cycle 03-15 D-§5-A carryover） | LEIBNIZ + KNUTH + SUSSMAN + 3 others | 6（cycle 03-15 vote 17/6） | PRESERVED — procedural branch 部分被 first-shipping success 反駁；post-AC-9 branch 透過 C1 操作化；caution-against-bundled-decision-making 透過 C5 操作化 |
| **DSS-B7**（cycle 03-15 D-§5-B carryover） | 4 personas | 4（cycle 03-15 vote 17/4/2） | PRESERVED — 操作上與 shipping outcome 一致（4 modules；與 DSS-B7 偏好之 4-module variant 相同） |
| **DSS-C4**（cycle 03-15 D-§5-E carryover） | 2 personas | 2（cycle 03-15 vote 21/2） | PRESERVED — 風格／粒度 dissent；shipping 驗證 1-module-2-artefacts pattern 可實作 |
| **DSS-3**（cycle 03-14 R3 LEIBNIZ ossification carryover） | 1（cluster） | — | **CLOSED-by-execution** 於 cycle 03-16 §4 D-§5-D trigger met（依 CV-19）**with NAGARJUNA Madhyamaka caveat** |
| **DSS-CY16-02**（cycle 03-16 R3 NEW；D-02 minority） | SUSSMAN + DARWIN + LINNAEUS + 2 others | 5（R3 D-02 vote 18/5） | PRESERVED — row-1-amendment 少數；abstraction parsimony 論點偏好將 DSS-A4 honour 內嵌於 C1 而非明確 C5 |

**逐字 DSS-CY16-02（cycle 03-16 R3 NEW）**：「Row 1 amendment to existing 4-criterion preferred over explicit C5；abstraction parsimony；DSS-A4 honour can be embedded inline rather than 5th criterion」。

R3 多數 18 選擇明確 C5 以求 cycle 03-17 R0 層次之操作清晰（RUSSELL agent rationality 依 F-§4-R2-07）。

## 11. MR/ZT 合規稽核

| 約束 | 狀態 | 證據 |
|------------|:------:|----------|
| MR-2 + MR-4（Tenet 措辭不改） | PASS | 無 Tenet 措辭變更 |
| MR-5 hard（Tenet #10 status no change） | PASS | governance / 架構 retrospective；無 Phase 6 functional implication |
| MR-6（Core 零） | PASS | 4 modules peripheral；於 file-system spot-check + R2 PENROSE F-§4-R2-04 驗證 |
| MR-9 | PASS | 無 preemptive MUST claim；F-16 SHOULD initial 依 CV-09 繼承 |
| MR-11 | PASS UNCONDITIONAL | DSS-CY16-02 R3 NEW + DSS-A4/B7/C4/3 carryover 全逐字保留 |
| MR-12 既有不破壞 | PASS | cross-version-skew helpers MUST D-§5-G 受到尊重；結構驗證 |
| ZT-1 / ZT-2 / ZT-3 | PASS reaffirmed | endpoint 10/0/0★ 不變；9/0/1★ ACTIVE 保留 |

## 12. 前向時程

- **Cycle 03-17 R0**：援用 5 準則 + 5×4 矩陣；依 C5 記錄 DSS-A4 weighing；依 cycle 慣例向 research-team 回報。
- **Cycle 03-17~03-20**：每 R0 可隨 plugin-loader 狀態與 AC-9 surface 物質化演進套用矩陣。
- **Cycle 03-21 soft floor**：依上述 §8 援用 MRB-06 處置邏輯；雙邊結構 A vs B。

---

*plugin-loader Cycle 03-17 5 準則 + 5×4 矩陣評估框架 — CANDIDATE pending Master Ratification Batch 13 #2*
*Authors: SUSSMAN + DARWIN + ARCHIMEDES + GUARDIAN + LEIBNIZ + RUSSELL*
*R3 D-items absorbed: D-02 (18/5；明確 C5) + D-19 (UNANIMOUS 雙邊結構) + CV-19 (DSS-3 closure-by-execution with Madhyamaka caveat)*
*DSS 保留：DSS-A4 (6) + DSS-B7 (4) + DSS-C4 (2) + DSS-CY16-02 (5) 逐字；DSS-3 closed-by-execution with caveat*
*Plan51 sunset-by-execution triggered (D-§5-D)；plugin-loader trail soft 6-cycle floor cycle 03-21*
*合規：MR-5 hard / MR-6 / MR-9 / MR-11 UNCONDITIONAL / MR-12 + ZT-1/2/3 全 PASS*
