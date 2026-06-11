# 74. Rule #74 — L1' = Code + Doc 同步性

`[Cycle 03-11 新增]` `[Master Ratified 2026-04-18, MR_REQ_1 Batch 1 APPROVED]`

> **來源**: Cycle 03-11 R3 Debate D3 (24/24 UNANIMOUS, per `R3_decision_log.md §D3.4`)
> **核心學者**: VITRUVIUS (#3, 主提), TURING (#17, L1' supplement), ARCHIMEDES (#16, F-9 互鎖), SUNYATA (#0), BABBAGE (#9), KERNEL (#10), GUARDIAN (#11), TANENBAUM (#20)
> **相關規則**: Rule #58 (Four Necessary Conditions L1), MR-7 (code 完成才 COMPLIANT), MR-10 (回補規範)
> **相關文件**: Doc 75 (Rules #71/#72 FR-1/FR-2), Reference/03 ENG-FAB Audit Checklist v1.6 (F-9 執行對應項)
> **批次**: MR_REQ_1 Batch 1（與 ENG-FAB v1.6 F-8 修訂 + F-9 新增三項互鎖打包同步 ratify）

---

## 1. 概述

Rule #74 是 Cycle 03-11 新增的 baseline rule（A 案，與 Rule #58 同級），將 Rule #58 L1「code exists」延伸涵蓋 doc 同步性，避免「形式上 L1 ✅ 實質上 doc 不完整」的灰色地帶。

| 維度 | 內容 |
|------|------|
| Motivator | Plan46 首交付（2026-04-17）code 完成但 `openstarry_doc/` 鏡像缺漏；coordinator 須事後觸發 `PLAN46-DOC-001` 補回 |
| 結構盲區 | Rule #58 L1「檔案」僅指 source code；不含 doc 同步性 |
| 演化路徑 | Sidecar pattern（doc 與 code 分兩次交付）若不規則化會在 Plan47+ 反覆 |
| 對策 | Baseline Rule #74 + ENG-FAB v1.6 F-9（執行層）+ Coordinator transfer gate（流程層）三層防護 |

---

## 2. Motivating Incident — Plan46 PLAN46-DOC-001

### 2.1 缺漏清單（per `VITRUVIUS_plan46_doc_audit.md §2.2`）

Plan46 首交付當下，`openstarry_doc/` 鏡像缺：

1. `Implementation_Plans/Plan46_*.md`（新 Plan 的實作計畫）
2. `Architecture_Documentation/73_*.md`（新架構決策）
3. `CHANGELOG_RESEARCH_TEAM.md` 本 cycle/Plan 條目
4. `release/{cycle}_{version}/openstarry_doc/` 與 `share/openstarry_doc/` byte-identical 鏡像

Coordinator 觸發 `PLAN46-DOC-001` 補登後，3/3 critical surface（K-3 hook / `wrapPluginWithToolFilter` C46-4 / `CheckpointManager` + `capturePluginHooks`）code 與 doc 100% 對齊，**補登品質本身無問題**，但 root cause 指向 SOP doc gate 缺失。

### 2.2 結構性風險

依 `TURING_plan46_L1_verification.md §10`：「the pattern is structurally invisible at delivery time without the rule」。Sidecar pattern 結構性使 doc 落後一輪。

依 `BABBAGE_plan46_L2_spec.md §11` fabrication 演化分析：「L2 footprint 越來越小但越來越深」。doc-deferral 是下一條 fabrication 演化路徑。Rule #74 是預防性規則。

---

## 3. Rule #74 規則文字（完整）

> **Rule #74（L1' = Code + Doc 同步性）**
>
> **動機**：Plan46 首交付（2026-04-17）程式碼完成但 `openstarry_doc/` 鏡像缺漏，coordinator 須事後觸發 `PLAN46-DOC-001` 補回。Rule #58 L1「code exists」未涵蓋 doc 同步性，導致 dev 可在「code 已交付」聲明下漏掉 doc。MR-7 要求 code 完成才算 COMPLIANT；本規則將「完成」延伸涵蓋 doc，避免「形式上 L1 ✅ 實質上不完整」之灰色地帶。
>
> **L1' 定義**：程式碼交付 COMPLIANT 條件，從 Rule #58 L1（code exists + diff 對 spec）擴張為：
> - **(a) 程式碼存在**：原 L1（不變）
> - **(b) Implementation Plan 存在**：`openstarry_doc/Implementation_Plans/Plan{N}_*.md`
> - **(c) Architecture Doc 存在**（若 Plan 引入新架構決策）：`openstarry_doc/Architecture_Documentation/{NN}_*.md`
> - **(d) CHANGELOG 條目**：`openstarry_doc/CHANGELOG_RESEARCH_TEAM.md` 含本 Cycle/Plan 條目
> - **(e) Release 鏡像**：`release/{cycle}_{version}/openstarry_doc/` 與 `share/openstarry_doc/` byte-identical
>
> **驗證機制**：
> 1. TURING L1' audit 逐項 check（code + (b)(c)(d)(e) 四件 doc artifact）
> 2. ENG-FAB F-9 自動 check（item granularity enumeration, per D4 議題）
> 3. Coordinator transfer gate（dev→research / dev→test transfer 前 pre-check 四件 doc artifact）
>
> **回補規範**（per MR-10）：
> - 發現缺漏立即觸發 `PLAN{N}-DOC-001` 回補 task
> - 記錄於 `delivery_report.md` post-delivery 章節（per Rule #62 Tier 1 分類 + 根因記錄）
> - 補登後 re-verify L1' 全項（五件齊備方為 PASS）
> - Compliance 在補回前維持 **PENDING**（不宣告 COMPLIANT）
>
> **生效時間**：Plan47 起全面適用（未來向）；Plan46 補登後 retroactively 視為 L1' COMPLIANT（歷史向一次性追認）。

---

## 4. 與 Rule #58 / MR-7 的關係

| 維度 | Rule #58 L1 | Rule #74 L1' |
|------|------------|-------------|
| 字面範圍 | 程式碼存在 + diff 對齊 spec | L1 + (b)(c)(d)(e) doc 四件 |
| 適用層級 | Necessary Condition Level 1 | L1 supplement（不改寫 L1 字面） |
| L2/L3/L4 影響 | 不變 | 不變（doc 不參與 runtime/integration 證據） |
| MR-7 「完成」定義 | 程式碼完成 | 程式碼完成 + doc 鏡像同步 |
| 觸發失敗時 | NC（Non-Compliant） | PENDING（補回後 COMPLIANT） |

---

## 5. 觸發條件 / N/A 條件

### 5.1 觸發 — 每次 Plan delivery 後立即

任何 Plan 交付後，自動執行 L1' audit（TURING + F-9）。

### 5.2 N/A 條件（不觸發 L1'）

- **純 reverify Plan**（無新 surface）：例如僅重跑既有測試的 Plan
- **純 hotfix Plan**（無新架構決策、不引入新 Implementation Plan）

---

## 6. 與 ENG-FAB v1.6 F-9 的互鎖

依 `ARCHIMEDES_plan46_L3_engfab.md §6.3`：**Rule #74 是規則，F-9 是執行**。兩者一體：

| 層級 | 角色 | 缺失後果 |
|------|------|---------|
| Rule #74 | Baseline rule（規則層硬性要求） | 缺則 F-9 無規則依據 |
| ENG-FAB F-9 | 執行檢查項（PASS/FAIL criteria + item granularity enumeration） | 缺則 Rule #74 無 audit 機制 |

此即 Batch 1 三項（D1 F-8 修訂 + D3 Rule #74 + D4 F-9 新增）打包 ratify 的核心論據。

完整 F-9 定義見 Reference/03 ENG-FAB Audit Checklist v1.6 §10。

---

## 7. Coordinator Transfer Gate 整合

依 `CLAUDE.md` Transfer 規則章節（dev → research / dev → test）：

```
dev → research transfer pre-check：
  ✓ delivery_report.md 存在
  ✓ codebase release/ 存在（無 node_modules）
  + [新增 per Rule #74] (b)(c)(d)(e) 四件 doc artifact 存在
  + [新增] release/{cycle}_{version}/openstarry_doc/ 與 share/openstarry_doc/ byte-identical
```

缺任一 → transfer 阻擋 + 觸發 PLAN{N}-DOC-001 補回 task。

---

## 8. 升 STRONG 條件 γ

依 `Master_Ratification_9_0_1_Conditions.md`（MR_REQ_3 條件 APPROVED）+ `R3 §D8.5`：

9/0/1† → 9/0/1★ 升級條件 γ 明文要求：
> **「Rule #74 + ENG-FAB v1.6 ratified by Master + Plan47 起首次適用 PASS」**

亦即 Rule #74 不僅是規則層補強，亦為 endpoint 路徑的 critical step。

---

## 9. Plan46 Retroactively COMPLIANT 處置

per Master Q3 ratify「生效時間採 Plan47 起（未來向）+ Plan46 retroactively COMPLIANT（歷史向追認）」：

- Plan46 補登後（PLAN46-DOC-001 完成）已滿足 L1' (a)(b)(c)(d)(e) 五件
- 視為 retroactively COMPLIANT（歷史向一次性追認）
- 此追認**不否定** Plan46 W2 端到端 K-3 disclosed deferral 之 CONDITIONAL COMPLIANT 狀態（兩議題正交）
- Plan47 起 forward apply：所有 Plan delivery 須事前完成 L1' 四件 doc artifact，非事後補回

---

## 10. 參考

### 10.1 R1/R2/R3/R4 來源
- **R1 主提**: `R1_independent/VITRUVIUS_plan46_doc_audit.md §6.1 Rule #74 全文草擬` + `§5.1-5.2 L1' 五件套定義` + `§12.4 升 STRONG 條件 (a)`
- **R1 背書**: `R1_independent/TURING_plan46_L1_verification.md §10 L1' supplement`
- **R1 互鎖**: `R1_independent/ARCHIMEDES_plan46_L3_engfab.md §6.3 L1' audit item: F-9`
- **R1 支持**: `R1_independent/SUNYATA_s3_observation.md §10 F-S3-3` + `BABBAGE_plan46_L2_spec.md §11`
- **R2**: `R2_crossreview/R2_crossreview.md §五.4 打包 Master ratification（D1+D3+D4 同 batch）`
- **R3**: `R3_debates/R3_decision_log.md §D3 全節` + `§A.1 整體合規宣告` + `§C Batch 1 MR_REQ_1`
- **O5**: `deliver/O5_compliance.md §5.1 MR_REQ_1` + `§4.2 Batch 1 互鎖邏輯`
- **R4**: `R4_finalization/R4_synthesis.md §4 6-batch packaging strategy`

### 10.2 Master Ratification
- `discussions/Master_Ratification_Rule_74.md`（Cycle 03-11 standalone, Master 2026-04-18 APPROVED）
- `discussions/Master_Ratification_ENG_FAB_v1_6.md`（同 Batch 1）

### 10.3 Vote
24/24 UNANIMOUS（per R3 §D3.4）

---

*Cycle 03-11 Architecture Documentation #74 — Rule #74 (L1' = Code + Doc Sync)*
*Authored by VITRUVIUS (#3) + ARCHIMEDES (#16); SCRIBE (#2) 收錄*
