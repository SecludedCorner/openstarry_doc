---
title: F-15 v3 — TW 翻譯對等之第三層修訂（L3 操作機制）
author: SCRIBE (#2) + LINNAEUS (#13) + TURING (#17) + ARCHIMEDES (#16) + ASANGA (#8) + NAGARJUNA (#7) + SYNTHESIST (#1)
date: 2026-04-28
cycle: 03-16 R4 close（cycle 03-15 reflexive obligation 之 TW 配對 sibling）
status: BINDING（cycle 03-15 R4 close 經 Master Ratification Batch 12 Item #1 APPROVED 2026-04-27 升 BINDING；本 TW sibling 同步 BINDING）
authority: research-team（cycle 03-15 R4 final draft）；Master（ratification 2026-04-27）
supersedes: cycle 03-14 F-15 v2（Research_Methodology/13_F_15_Scope_Expansion_Governance_Docs.md） — 累積擴充而**非**取代
language: tw
cross_refs:
  - research record/cycle03-16/openstarry_doc/Research_Methodology/16_F_15_v3_Third_Tier_Amendment.md（cycle 03-15 EN baseline；同步至 cycle 03-16 canonical mirror）
  - research record/cycle03-15/deliver/O1_TW_translation_final.md §3 + §4 + §10
  - research record/cycle03-15/R3/R3_decision_log.md §4.1 + §4.3（D-§2-06 + D-§2-07 + D-§3-05）
  - research record/cycle03-16/openstarry_doc/Reference/11_Rule_78_TW_Translation.tw.md（L1 配對 sibling）
  - research record/cycle03-14/openstarry_doc/Research_Methodology/13_F_15_Scope_Expansion_Governance_Docs.md（v2 baseline，BINDING）
  - research record/cycle03-15/deliver/Master_Ratification/Batch_12_Request.md §3.1（Item #1 bundle）
  - research record/cycle03-16/deliver/O5_Rule78_F15v3_audit_final.md（cycle 03-16 first-shipping linter audit）
binding_until: forward-effective post-Batch 12 ratification（無到期；cycle 03-16 first-shipping 觀察）
prefix_discipline: verified | inferred | speculative
---

# F-15 v3 — TW 翻譯對等之第三層修訂（L3 操作機制）

## 1. 狀態

**BINDING**（cycle 03-15 R4 close 經 Master Ratification Batch 12 Item #1 升 BINDING；2026-04-27 ratified；與 Rule #78 L1 sibling bundled）。本 TW sibling 為 cycle 03-16 R4 close 落地之 reflexive obligation 履行（依 MRB-§5-§78-A R4 critical-path），與 EN sibling 同 PR 上線（per cycle 03-16 R3 D-09 same-PR-strict UNANIMOUS）；status 跟隨 EN sibling。

**前向**：自 cycle 03-15 R3 close 起。

**落地形式**：L1+L3 hybrid 依 D-§2-07（14/4/3/2）。本檔為 **L3 操作機制**半邊（Research_Methodology/，governance-doc front-matter parity 檢查 + linter dispatcher 機制）。**L1 高層政策**半邊為 `Reference/11_Rule_78_TW_Translation.md`。Master Ratification Batch 12 Item #1 將兩者作為 bundle dispatch。

**累積範圍**：F-15 v3 攜帶累積範圍自 v1（cycle 03-14 ratified front-matter discipline） → v2（cycle 03-14 ratified governance-doc scope expansion 至 8 doc classes） → v3（cycle 03-15 候選 TW parity tier）。v1 + v2 功能**保留不變**；v3 為**累加**依 MR-12 既有不破壞。

**ENG-FAB increment policy**：F-15 自身為 **governance-doc discipline bundle**，**非** v1.X 編號之 ENG-FAB 項目。依 cycle 03-14 ratification，F-15 amendment-not-increment 進程保留 ENG-FAB v1.8 = 48 items canonical。v1.9 = 49 候選 F-16 不變。**v1.10 候選 F-19 不創建**依 D-§3-05 + D-§2-07 anchor unification。

## 2. R3 出處

- **D-§2-06 (C-1)** — CI gate = **multi-layer**（F-15 linter + pnpm build + doc gate + G4-folder + coordinator hook）UNANIMOUS 23/0。
- **D-§2-07 (A-1)** — TW 落地點 = L1+L3 hybrid（14 票）。F-15 v3 amendment = L3 半邊。
- **D-§3-05 (B-6)** — Anchor framework = unified Rule #78 + F-15 third-layer（節省 1 candidate slot 相對於 separate ENG-FAB v1.10 F-19） — 16/4/3；DSS-B6 4 dissent §10。**Conditional** 依 Master §3 outcome（A-trail 涵蓋 §2+§3；B/C-trail 僅 §2）。
- **D-§2-04** — quality discipline（glossary epistemic-prefix + 結構保真 + dual-author NO）feeds 進 §3 linter 檢查。
- **MRB-§2-02 RESOLVED** — sibling-naming convention `<basename>.<lang>.md` 編碼化（TURING + SCRIBE）。

## 3. Binding Text

### 3.1 F-15 v3 三層 bundle

> **F-15 v1（cycle 03-14 ratified）** — Tier 1：governance-doc front-matter discipline。Front-matter language fields `title / author / date / cycle / status / authority / cross_refs` MUST 存在。Linter `tools/f15_check.py` 強制。**保留不變**。
>
> **F-15 v2（cycle 03-14 ratified）** — Tier 2：governance-doc scope expansion。F-15 涵蓋 8 doc classes：O-series / R-stage / Master_Ratification batches / SCRIBE process gates / Plan-spec dev-facing / Test instructions / todo_README / openstarry_doc canonical diff entries。Conditional-MUST gates 依 cycle 03-14 ratification text。**保留不變**。
>
> **F-15 v3（cycle 03-15 NEW；本 amendment）** — Tier 3：governance-doc TW 翻譯對等檢查依 Rule #78 §78.1 tier × §78.6 CI gate。

### 3.2 Tier 3 五個子機制

> **子機制 1 — Sibling presence 檢查。**
> 對於每個 §78.1 BINDING / CANDIDATE / KNOWLEDGE_ONLY tier 之 EN-side 文件（依 `tools/f15_config.yaml` tier-config dispatcher），linter 斷言 `<basename>.tw.md` 存在於同目錄。Sibling-naming convention 依 Rule #78 §78.7（編碼化 MRB-§2-02）。失敗模式依 §78.6 layer-1：BINDING 之 block-PR；CANDIDATE / KNOWLEDGE_ONLY 之 warning；CONSULTATIVE / SCRIBE-internal 之 informational。

> **子機制 2 — Front-matter language tag 檢查。**
> TW sibling 必有 `language: tw`（或等價 `language: zh-TW`）front-matter field，與 EN sibling 之 `language: en` 不同。Cross-reference fields 必顯式識別 EN sibling（如 `cross_refs: - <basename>.md`）。Front-matter schema 強制與 v1 baseline 一致（title / author / date / cycle / status / authority / cross_refs）。

> **子機制 3 — Glossary epistemic-prefix 檢查。**
> 若 glossary 文件（`tools/tw_glossary.yaml` 或類似）存在依 Rule #78 §78.4 + O1 §6.1，每個 entry 必有 `prefix:` field 值為 `verified` / `inferred` / `speculative`。Linter 驗證 schema。依 ASANGA Yogācāra 知識論紀律。前綴缺失或無效 = BINDING tier 之 block-PR。

> **子機制 4 — 結構保真 machine-checkable layer。**
> Linter 比較 EN 與 TW siblings 之 heading 階層深度、表格列數、code-block 存在。不匹配 = BINDING + CANDIDATE 之 warning；KNOWLEDGE_ONLY 之 informational。Human-checkable layer（bullet 順序、footnote anchor stability、prose-paragraph-count parity）為 G4-folder gate 之 reviewer 判斷；un-checkable layer（句子層對齊、慣用語 register match）為 author latitude。依 ASANGA + TURING 層分解（Rule #78 §78.4 結構保真 22/1）。

> **子機制 5 — Per-tier dispatcher。**
> Linter 接受 config 文件（`tools/f15_config.yaml`）宣告哪些 paths 為哪 tier 依 Rule #78 §78.1（BINDING / CANDIDATE / KNOWLEDGE_ONLY / CONSULTATIVE / SCRIBE-internal）。Default config 隨 cycle 03-15 ratification 出貨；支援未來 tier 細化而無需重新 coding。

### 3.3 NAGARJUNA Madhyamaka 一致性註

Tier 3 由 enumeration（tier configs 中之特定 file paths）**綁定**，而非由開放本質「所有 governance docs」referent。此中和 R1 §10.3 + R2 F-01（NAGARJUNA + ASANGA dissent on 本質主義 scope）所注本質主義風險。

## 4. F-15 Linter 擴充 Spec

### 4.1 範圍估計

依 F-§2-R1-11 + R2 F-04 TURING 程式碼可行性 cross-check。對 `tools/f15_check.py` 之擴充範圍：

| 元件 | LOC 範圍 | 功能 |
|------|--------:|------|
| 純粹 sibling-file presence 檢查 | 30-50 | 對於 tier-config paths 中每個 EN-side 文件，斷言 `<basename>.tw.md` 存在 |
| Tier-graded scope dispatcher | 50-80 | Config-driven（`tools/f15_config.yaml`）依 Rule #78 §78.1 dispatch per-tier 規則 |
| Glossary 機制整合 | 50-100 | 若 `tools/tw_glossary.yaml` 存在，驗證 schema（含 `prefix:` field 之 entry format）並可選地檢查 TW siblings 中之 term-consistency |
| **合計** | **130-230** | **於 R1 §F-11 估計範圍內** |

### 4.2 實作指引（僅 spec；本文件無 code）

1. **Config-driven design**：tier 定義於外部 YAML，非 hard-coded；支援未來 tier 細化。
2. **Sibling-naming flexibility**：linter 接受 `<basename>.md`（legacy）與 `<basename>.en.md`（preferred）為 EN canonical 依 Rule #78 §78.7。
3. **失敗模式 dispatcher**：per-tier 失敗模式（block / warning / informational）由 config 讀取；允許 per-deployment 客製化。
4. **Front-matter language tag 檢查**：TW sibling 必有 `language: tw`（或 `zh-TW`）front-matter；linter 驗證。
5. **結構保真 machine-checkable 檢查**：EN 與 TW siblings 間之 heading 階層深度 + 計數、表格列數、code-block 計數比較。
6. **Glossary epistemic-prefix 驗證**：每個 glossary entry 必有 `prefix: verified|inferred|speculative`；前綴缺失或無效 = BINDING tier 之 block-PR。
7. **CI 整合**：linter 以 exit code 0（PASS）/ 1（warnings only）/ 2（block-level failure）退出；CI 依 Rule #78 §78.6 layer 1 消費 exit code。

### 4.3 實作 cycle

排程於 **cycle 03-16（Plan51 實作 cycle）** 依 O1 §10 + F-§5-R2-15 coupling。Master Ratification Batch 12 #1 ratifies 政策 + spec；Plan51 cycle 03-16 實作 linter 擴充。

**Cycle 03-16 first-shipping 觀察**：依 cycle 03-16 O5 first-shipping linter audit，shipped `tools/f15_check.py`（117 LOC）為 **legacy Plan49 C49-M8 F-15 v1 claim-block linter**，**非** F-15 v3 §3.2 governance-doc + tier-dispatcher + glossary-prefix linter。Shipped `tools/sibling-naming-check.mjs`（126 LOC）涵蓋子機制 1（sibling presence）+ 子機制 4 之半邊（heading + code-fence count，非 table-column count）於 `agent_dev/openstarry/docs/EN|TW/` folder pair。依 Rule #78 §78.5 forward-only 與 MR-12 既有不破壞，此 gap 派遣至 cycle 03-17 Plan-spec second-tier（Master Ratification Batch 13 Item #6 APPROVE Plan-spec scope）。依 D-17 UNANIMOUS，cycle 03-17 Plan-spec 引入 sibling `tools/f15_v3_check.py`（保留 `tools/f15_check.py` 為 Plan49 v1 legacy）配合 Rule #78 §78.6 layer 1 footnote 區別。

## 5. 五層 Scope（反映自 Rule #78 §78.1）

Linter dispatcher 操作於 Rule #78 五層之上：

| Tier | Strength | F-15 v3 layer 1 mode | Audit |
|------|----------|----------------------|-------|
| **BINDING** | MUST | block-PR | release-tag scheduled audit |
| **CANDIDATE** | SHOULD | warning | release-tag scheduled audit |
| **KNOWLEDGE_ONLY** | SHOULD | warning | release-tag scheduled audit（lower priority） |
| **CONSULTATIVE** | MAY | informational | informational（G4-folder gate） |
| **SCRIBE-internal** | MAY | informational | none |

Tier 轉換（依 Rule #78 §4.2 of O1 / §78.1 of L1 sibling）：
- **CANDIDATE ↔ BINDING**：Master ratification 升級；supersession 降級（forward-only 尊重）。
- **CONSULTATIVE → CANDIDATE**：draft 經 ratification 進 canonical artefact 時。
- **SCRIBE-internal → CANDIDATE**：罕見；僅於 SCRIBE-internal log 被未來 Rule 逐字引用時。

## 6. Strength 矩陣（反映自 Rule #78 §78.2 + §5）

| Strength axis | Governance | Canonical | Plan/operational |
|---------------|------------|-----------|------------------|
| F-15 v3 強制 | **MUST** | **SHOULD** | **MAY** |
| 時序升級 | 24h grace before R3+1 cycle close | 24h grace + R3+1 cycle close | n/a |
| 子機制 1（sibling presence） | block-PR | warning | informational |
| 子機制 2（lang tag） | block-PR | warning | informational |
| 子機制 3（glossary prefix） | block-PR | warning | informational |
| 子機制 4（struct fidelity） | machine MUST + human SHOULD warning | machine MUST + human SHOULD warning | machine MAY |
| 子機制 5（tier dispatcher） | always-on | always-on | always-on |

## 7. Reflexive Application

依 F-§2-R1-10 + F-§2-R2-10 NAGARJUNA self-reference test：F-15 v3 自身遵循 F-15 v3 schema。本檔：

- 攜帶 v1 front-matter discipline（title / author / date / cycle / status / authority / cross_refs ✓）。
- 攜帶 v2 scope（Research_Methodology/ subdir；CANDIDATE tier 依 §78.1）。
- 攜帶 v3 language tag（`language: tw` ✓ for this TW sibling；`language: en` for the EN sibling）。
- 已取得 `16_F_15_v3_Third_Tier_Amendment.tw.md` sibling（**本檔即為**）依 Rule #78 §78.3 BINDING-tier same-PR-strict timing 經 Master Ratification（cycle 03-16 R0 24h grace 撰寫；**cycle 03-16 R4 critical-path 履行依 MRB-§5-§78-A**）。

## 8. Backward Compatibility（v1 + v2 保留）

F-15 v3 為**累加**：
- v1 front-matter discipline：保留不變。所有 cycle 03-14 ratified front-matter 規則仍於效。
- v2 governance-doc scope expansion：保留不變。所有 v2 涵蓋之 8 doc classes 仍涵蓋。
- v3 子機制：於 v1+v2 之上 ADDED layer。Pre-cycle-03-15 docs 祖父化 EN-only 依 Rule #78 §78.5 + MR-12；v3 子機制套用至 cycle 03-15+ docs 前向。

依 R3 投票選擇 L1+L3 hybrid 而非 L1-only 或 L2-only：政策位於 Rule #78（乾淨概念），機制位於 F-15 amendment（基礎設施重用）。DSS-A1 4 dissent（偏好 L2 ENG-FAB v1.10 F-19）依 MR-11 §10 逐字保留。

## 9. MR/ZT Compliance Audit

| 約束 | 狀態 | 證據 |
|------|:---:|------|
| **MR-5 hard**（Tenet #10 status 不變） | PASS | F-15 v3 為 doc-tooling 層；對 Tenet #10 正交 |
| **MR-6**（Core 零） | PASS | Linter 擴充位於 `tools/`；非 Core |
| **MR-9**（無 MUST WAIVE） | PASS | F-16 SHOULD initial 不變；F-15 v3 strength 依 Rule #78 §78.2 分層 |
| **MR-10**（back-fill / retroactive） | PASS | Cycle 03-13 backfill = one-off precedent（sibling Reference doc 12） |
| **MR-11**（dissent 保留） | PASS | DSS-A1 / DSS-B6 逐字 §10 |
| **MR-12**（既有不破壞 / forward-only） | PASS | Pre-cycle-03-15 祖父化；v1+v2 保留不變 |
| **MR-13** standby | PASS | 未啟動 |
| **ZT-1 / ZT-2 / ZT-3** | PASS | 無 Tenet rewrite；endpoint 10/0/0★ 不變；控制範圍未動 |
| **ENG-FAB v1.8 = 48** | 保留 | F-15 amendment-not-increment 依 cycle 03-14 先例 |
| **ENG-FAB v1.9 = 49 F-16 SHOULD** | 保留 | F-16 strength 不變（Batch 12 Item #4 informational only） |
| **v1.10 F-19** | 不創建 | 依 D-§3-05 + D-§2-07 anchor unification |

## 10. Dissent Preservation（依 MR-11 逐字）

### 10.1 DSS-A1（D-§2-07 TW 落地 L1+L3 hybrid；14/4/3/2）

**少數**：NAGARJUNA + DARWIN + 其他 2 位（4 票偏好 L2 ENG-FAB v1.10 F-19）。

**逐字**：「L2（ENG-FAB v1.10 F-19）提供更乾淨的工程化 + MUST/SHOULD/MAY 強度；L1+L3 hybrid 過於分散 governance。」

**R4 義務**：本檔 §10 顯式 dissent slot 依 MR-11。註：此 dissent 套用 L1+L3 hybrid landing 決定。R3 多數（14）勝出但 dissent 作為實質少數立場保留。

### 10.2 DSS-B6（D-§3-05 anchor unified Rule #78 + F-15 third-layer；16/4/3）

**少數**：4 dissent。

**逐字**：「Separate ENG-FAB v1.10 F-19 entry would provide cleaner ENG-FAB-side visibility for the trial-graduation governance integration; anchor unification 過於 saving 1 candidate slot 而失 ENG-FAB-side discoverability.」

**R4 義務**：本檔 footnote 依 MR-11。註：D-§3-05 為 **conditional** 依 Master §3（A）/(B)/(C) outcome — 若 Master 選擇 (A)：Rule #78 + F-15 v3 涵蓋 §2 + §3 graduation governance；若 Master 選擇 (B) 或 (C)：F-15 v3 僅涵蓋 §2。任一情況，F-15 v3 amendment 進行。

### 10.3 合計

§3-tagged dissent（DSS-B6）與 §2-tagged dissent（DSS-A1 在此亦相關作為 L3 vehicle choice）分別保留。其他 §2 dissent entries（DSS-A2 / DSS-B1 / DSS-B1b / DSS-B2 / DSS-B3 / DSS-B3b / DSS-D2）為 L1 sibling Rule #78 doc 之 scope-relevant 並逐字保留於該檔。

---

*F-15 v3 — Cycle 03-15 R4 Final Spec（cycle 03-16 R4 close 之 TW sibling 履行）*
*作者：SCRIBE (#2) + LINNAEUS (#13) + TURING (#17) + ARCHIMEDES (#16) + ASANGA (#8) + NAGARJUNA (#7) + SYNTHESIST (#1)*
*狀態：BINDING（經 Master Ratification Batch 12 #1 APPROVED 2026-04-27；本 TW sibling 同步 BINDING）*
*落地：L3 操作機制半邊（Research_Methodology/）；L1 高層政策半邊 = `Reference/11_Rule_78_TW_Translation.md`*
*三層 bundle：v1 front-matter（cycle 03-14） + v2 scope expansion（cycle 03-14） + v3 TW parity（cycle 03-15 候選）*
*F-15 amendment-not-increment：ENG-FAB v1.8 = 48 保留；v1.10 F-19 不創建*
*合規：MR-5 hard / MR-6 / MR-9 / MR-10 / MR-11 / MR-12 + ZT-1/2/3 全 PASS*
*Reflexive obligation 履行：cycle 03-16 R4 critical-path 依 MRB-§5-§78-A（Rule #78 §78.5 forward-only + §78.8 reflexive；same-PR-strict 同步 EN sibling 依 cycle 03-16 R3 D-09 UNANIMOUS）*
