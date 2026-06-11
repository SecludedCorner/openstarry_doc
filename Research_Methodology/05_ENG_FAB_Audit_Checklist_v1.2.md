# ENG-FAB Audit Checklist (v1.0 -- v1.3)

**版本**: 1.3 (Cycle 03-7, 2026-04-10)
**前版**: v1.2 (Cycle 03-6, 30 items); v1.1 (Cycle 03-5, 29 items); v1.0 (Cycle 03-5, 22 items)
**性質**: 活文件 (Rule #49)，隨經驗迭代更新
**來源**: D1-Q5 (03-6), D3-Q3 (03-5), D4-Q2 (03-6), D4-Q3/Q4 (03-7), D7-Q3 (03-7)

---

## 概述

ENG-FAB Audit Checklist 是研究團隊對工程交付進行驗證的最低標準。完成 checklist 後，審計者仍可自由補充額外發現。

**執行規則**:
- **MUST** 項目失敗 = Plan 不接受 (Rule #53)
- **SHOULD** 項目失敗 = 記錄為 INFO/LOW finding
- **Conditional** 項目視條件啟用
- 偶數 Plan: 全量驗證 (Rule #47)
- 奇數 Plan: 增量驗證 (Rule #47)

---

## Version Evolution

| Version | Cycle | Items | Key Addition |
|:-------:|:-----:|:-----:|-------------|
| v1.0 | 03-5 | 22 | Initial: Categories A-E |
| v1.1 | 03-5 | 29 | +7: MUST/SHOULD升級; DEFERRED mechanism; assertion density, cross-cycle regression, snapshot completeness, test artifact |
| v1.2 | 03-6 | 30 | +1: A-9 integration verification (Rule #54, 因 Plan41 phantom integration) |
| **v1.3** | **03-7** | **38** | **+8: G section (dynamic verification, 4) + H section (configuration correctness, 4); A-0 applicability judgment; Rule #60 checklist fatigue control** |

Each version addresses a specific fabrication generation while maintaining backward compatibility with earlier detection methods.

---

## A-0: Applicability Judgment (1 item, v1.3 新增)

| # | Item | MUST/SHOULD | Pass Criteria |
|---|------|-------------|---------------|
| **A-0** | **TURING 判定本 Plan 適用之 section (A-H)；GUARDIAN 審查完整性** | **MUST** | **明確列出適用/不適用的 section 及理由** |

> **A-0 新增原因** (03-7 D4-Q3): 無明確適用性判定時，審計者可能無根據地標記 section 為 N/A，或在無關 section 浪費精力。A-0 確保每次審計從明確的範圍決策開始。

---

## A: Fabrication Detection (9 items)

| # | Item | MUST/SHOULD | Pass Criteria |
|---|------|-------------|---------------|
| A-1 | 每個聲稱的 production file 存在 | MUST | `ls` 確認存在 |
| A-2 | 每個 production file 非空且含聲稱的修改 | MUST | diff 確認 |
| A-3 | 每個新增 test file 存在 | MUST | `ls` 確認 |
| A-4 | 每個 test file 測試其聲稱的功能 | MUST | 程式碼審查 |
| A-5 | 無幻影檔案（聲稱存在但不存在） | MUST | 交叉對照 |
| A-6 | 無幻影修改（聲稱修改但未修改） | MUST | diff 對照 |
| A-7 | 無幻影功能（聲稱實現但未實現） | MUST | 功能驗證 |
| A-8 | Assertion density ≥ 1.0 per test | SHOULD→MUST* | *partial fabrication 時升級 |
| **A-9** | **Integration verification: 從系統入口可達** | **MUST** | **未接入 = FAIL; 接入但未觸發 = INFO** |

> **A-9 新增原因** (03-6 D1-Q5): Plan41 Finding 2-3 "phantom integration" — reporter 程式碼存在、測試通過、但從未被系統呼叫。A-9 防止此類情況。

---

## B: Traceability (8 items)

| # | Item | MUST/SHOULD | Pass Criteria |
|---|------|-------------|---------------|
| B-1 | Traceability matrix 完整（每個需求→source→test） | MUST | 逐行確認 |
| B-2 | 偶數 Plan: 全量矩陣驗證 | MUST (even) | Rule #47 |
| B-3 | 奇數 Plan: 增量矩陣驗證（改動 + 依賴鏈路） | MUST (odd) | Rule #47 |
| B-4 | Source file 行號與實際對應 | MUST | 抽查確認 |
| B-5 | Test file 與 source file 對應關係正確 | MUST | 交叉對照 |
| B-6 | Acceptance criteria 逐一可驗證 | MUST | AC 對照 |
| B-7 | LOC 報告一致（摘要 vs 明細） | SHOULD | 交叉對照 |
| B-8 | Diff 驗證（聲稱的改動 vs 實際 diff） | MUST | git diff |

---

## C: Test Coverage (4 items)

| # | Item | MUST/SHOULD | Pass Criteria |
|---|------|-------------|---------------|
| C-1 | npm test --verbose pass/fail 統計 | MUST | 數字匹配 |
| C-2 | 新增/刪除 test 清單 | MUST | 對照 delivery report |
| C-3 | Test churn ratio 監控 | SHOULD | 異常標記 |
| C-4 | Coverage delta | SHOULD→MUST* | *coverage 下降 > 5% 時升級 |

---

## D: Evidence Format (6 items)

| # | Item | MUST/SHOULD | Pass Criteria |
|---|------|-------------|---------------|
| D-1 | Raw log 存在（npm test --verbose） | MUST | 檔案存在 |
| D-2 | 結構化摘要存在 | MUST | 格式正確 |
| D-3 | Raw log 與摘要數據一致 | MUST | 交叉對照 |
| D-4 | Timestamp (ISO 8601 UTC) | MUST | 格式確認 |
| D-5 | Snapshot 包含 test files | MUST (DEFERRED max 1) | Rule #52 |
| D-6 | Test artifact delivery 完整 | MUST (DEFERRED max 1) | 檔案存在 |

---

## E: Security (4 items)

| # | Item | MUST/SHOULD | Pass Criteria |
|---|------|-------------|---------------|
| E-1 | 新 attack surface 評估 | Conditional (diff-triggered) | 安全審查 |
| E-2 | Dependency audit | Conditional (new deps) | 版本確認 |
| E-3 | Key management 評估 | Conditional (crypto changes) | GUARDIAN 審查 |
| E-4 | Hardcoded secrets 檢查 | Conditional (new files) | grep 確認 |

---

## G: Dynamic Feature Verification (4 items, v1.3 新增, Phase 3+)

| # | Item | MUST/SHOULD | Pass Criteria |
|---|------|-------------|---------------|
| **G-1** | Shadow decision count >= MIN_N per category | **MUST** | W2 數據確認每個 category 達到統計最低樣本數 |
| **G-2** | Shadow vs actual schema completeness | **MUST** | Schema 欄位完整，不完整 = 不可比較數據 |
| **G-3** | Feature flag enabled confirmation | **MUST** | 功能旗標確認啟用；關閉 = config-layer fabrication (Gen4 預防) |
| **G-4** | M4a agreement rate computability | **SHOULD** | 分母非零；SHOULD 因失敗可能是時序問題 |

> **G section 新增原因** (03-7 D4-Q4): TURING 第七變體預測 (configuration-layer fabrication) — 程式碼正確、測試通過、但配置使功能在運行時不活躍。G section 要求運行時執行軌跡，不僅僅是靜態程式碼存在。

**適用性**: 無 dynamic features 的 Plan 標記 N/A（需在 A-0 明確判定）。

---

## H: Configuration Correctness (4 items, v1.3 新增, dual-track per Master)

### Relative Track — 鎖定參數必須匹配 W2 驗證值

| # | Item | MUST/SHOULD | Pass Criteria |
|---|------|-------------|---------------|
| **H-1** | 全部 7 個鎖定參數匹配 W2 驗證值 | **MUST** | 逐一數值對照 |
| **H-2** | 參數值可追溯至鎖定的 W2 round | **MUST** | 來源文件確認 |

**7 個鎖定參數**: DELTA_SCALING_FACTOR (0.055), MIN_N (10), GEAR_UP/DOWN thresholds, MIN_DWELL, static/dynamic priority weights.

### Process Track — 新參數需要文件化理由

| # | Item | MUST/SHOULD | Pass Criteria |
|---|------|-------------|---------------|
| **H-3** | 新參數理由引用 W2 數據、設計原則或安全約束 | **MUST** | 理由文件存在且具體 |
| **H-4** | 新參數預設值有正當理由（非任意值） | **MUST** | 數值來源可追溯 |

> **H section 新增原因** (03-7 D4-Q4): Master §8-2 dual-track detection — relative track (W2 數據匹配) + process track (新參數理由)。防止 calibration 參數被任意設定或偷偷修改。

**適用性**: 無 calibration 參數新增/修改的 Plan 標記 N/A（需在 A-0 明確判定）。

---

## Checklist Fatigue and Growth Management (Rule #60)

**ARCHIMEDES R2 warning**: 超過 ~30 個 mandatory items 後，checklist 效果因審計者疲勞而下降。每增加一個 MUST 項目都會稀釋既有項目的注意力。

**Rule #60** 建立兩個控制:
1. 新 mandatory 項目需要 R3 投票通過（不可單方面新增）
2. 每個新 MUST 項目必須引用其防止的具體失敗模式

**MR-11 connection**: 有效的檢查 > 更多的檢查。Checklist 尊重架構選擇，不做官僚堆積。v1.3 的 38 項接近疲勞閾值。未來版本應優先升級既有 SHOULD 項目，而非新增項目，並考慮退役從未觸發 finding 的項目。

---

## 統計 (v1.3)

| 分類 | Items | MUST | SHOULD | Conditional |
|------|:-----:|:----:|:------:|:-----------:|
| A-0 (Applicability) | 1 | 1 | 0 | 0 |
| A (Fabrication) | 9 | 8 | 1* | 0 |
| B (Traceability) | 8 | 7 | 1 | 0 |
| C (Coverage) | 4 | 2 | 2* | 0 |
| D (Evidence) | 6 | 5 | 0 | 0 (+1 DEFERRED) |
| E (Security) | 4 | 0 | 0 | 4 |
| G (Dynamic) | 4 | 3 | 1 | 0 (N/A if no dynamic features) |
| H (Configuration) | 4 | 4 | 0 | 0 (N/A if no param changes) |
| **Total** | **38** (+A-0) | **30** | **5** | **4** |

> **Note**: A-0 is a procedural gate item that precedes all others. The checklist contains **38 verification items** plus A-0 applicability judgment. 有效 MUST 數量依 Plan 適用性約 30-38 項。

### v1.2 統計 (歷史參考)

| 分類 | MUST | SHOULD | Conditional | Total |
|------|------|--------|-------------|-------|
| A | 8 | 1* | 0 | 9 |
| B | 7 | 1 | 0 | 8 |
| C | 2 | 2* | 0 | 4 |
| D | 5 | 0 | 0 | 5 (+1 DEFERRED mechanism) |
| E | 0 | 0 | 4 | 4 |
| **Total** | **22** | **4** | **4** | **30** |

---

## 變更歷史

| 版本 | 日期 | 變更 |
|------|------|------|
| v1.0 | 03-5 R1 | GUARDIAN 初版設計，22 items (A-E sections) |
| v1.1 | 03-5 R3 | +7 items (assertion density, MUST/SHOULD 升級, DEFERRED mechanism, snapshot completeness, test artifact), 29 items |
| v1.2 | 03-6 R3 | +A-9 integration verification, 30 items. 因 Plan41 "phantom integration" finding (Rule #54) |
| **v1.3** | **03-7 R3** | **+A-0 applicability judgment, +G section (dynamic feature verification, 4 items), +H section (configuration correctness, 4 items), 38 items. 因 Gen4 prediction (config-layer fabrication) + Master §8-2 dual-track detection. Rule #60 (fatigue control)** |

---

*ENG-FAB Audit Checklist v1.0 -- v1.3*
*Rule #49 (活文件) + Rule #53 (MUST fail = not accepted) + Rule #54 (A-9) + Rule #60 (fatigue control)*
*GUARDIAN (#11), ARCHIMEDES (#16), TURING (#17), WIENER (#12)*
*Cycle 03-7, 2026-04-10*
