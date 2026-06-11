<!-- Layer: 5-Reference -->
<!-- Status: NEW -->
<!-- Cycle: 03-5, R3 -->
<!-- Author: GUARDIAN (#11), SYNTHESIST (#1), ARCHIMEDES (#16) -->
<!-- Source: [D3-Q2], [D3-Q3], [D3-Q5] -->

# R-2. ENG-FAB Audit Checklist v1.1

**版本**: 1.1 (Cycle 03-5, 2026-04-08)
**作者**: GUARDIAN (#11), SYNTHESIST (#1), ARCHIMEDES (#16)
**決議**: [D3-Q2] 採用 6-0, [D3-Q3] MUST/SHOULD 分級 6-0, [D3-Q5] 強制執行 6-0

> **活文件** (Living Document): 依 Rule #49 隨經驗迭代。項目可透過 R3 投票新增、升級或退役。

---

## 1. 執行框架

### 1.1 強制等級 [Rule #53]

| 等級 | 後果 | 閘門性質 |
|------|------|----------|
| **MUST** | 失敗 = Plan 不接受 (或 DEFERRED, 見 1.3) | 阻斷閘門 |
| **SHOULD** | 失敗 = 生成 INFO 或 LOW 嚴重度 finding | 非阻斷 |
| **Conditional MUST** | 觸發時為 MUST；不適用時為 N/A | 視情況 |

### 1.2 適用性 [Rule #47]

| Plan 類型 | 追溯範圍 | B 類項目 |
|-----------|----------|----------|
| **偶數** (40, 42, 44...) | 全量追溯矩陣 | 全部適用 |
| **奇數** (41, 43, 45...) | 增量驗證 (變更 + 受影響依賴鏈) | 僅變更部分 |

### 1.3 DEFERRED 機制 [Rule #52]

若 D-5 或 D-6 失敗，測試相關項目 (A-2 through A-5, B-2, C-1 through C-3) 變為 DEFERRED 而非 FAIL。

**限制**: 最多 1 個連續 Plan 可 DEFERRED。連續兩輪 D-5/D-6 DEFERRED = Plan 交付**阻斷**。

### 1.4 證據要求 [Rule #46]

每次交付**必須**同時包含：
1. **Raw verbose log**: 完整 `npm test --verbose` 輸出，ISO 8601 UTC 時間戳
2. **Structured summary**: pass/fail 統計、覆蓋率、新增/刪除測試清單、造假檢查結果

---

## 2. Category A: 造假偵測 (8 項) [Rule #48]

| ID | 項目 | 通過標準 | 等級 | 適用 |
|----|------|----------|:----:|:----:|
| A-1 | **生產檔案存在**: 每個宣稱的生產檔案在指定路徑存在 | `Read` 確認檔案存在且非零大小 | **MUST** | Even+Odd |
| A-2 | **測試檔案存在**: 每個宣稱的測試檔案在指定路徑存在 | `Read` 確認存在；D-5 DEFERRED 時被阻斷 | **MUST** | Even+Odd |
| A-3 | **測試內容**: 每個測試檔案包含測試其宣稱覆蓋範圍的斷言 | `describe`/`it`/`test` 區塊匹配；每個 test case 至少一個實質 `expect`/`assert` | **MUST** | Even+Odd |
| A-4 | **測試執行**: 每個宣稱的測試檔案出現在 verbose log 中 | grep verbose log 中的測試檔名；確認 PASS/FAIL 狀態 | **MUST** | Even+Odd |
| A-5 | **指標調和**: 宣稱的測試總數與 verbose log 匹配 | passed + failed + skipped in log = claimed total | **MUST** | Even+Odd |
| A-6 | **新檔案審計**: 每個新檔案（不在前版中）獨立驗證 | diff 檔案清單；讀取每個新檔案；內容與交付報告匹配 | **MUST** | Even+Odd |
| A-7 | **刪除檔案審計**: 前版存在但當前版本缺失的檔案已記錄 | diff 清單中的刪除項皆在交付報告中說明 | **MUST** | Even+Odd |
| A-8 | **快照完整性**: `__tests__/` 目錄含實際測試檔案 | 所有 `__tests__/` 目錄非空或已記錄為故意空白 | **MUST** | Even+Odd |

---

## 3. Category B: 可追溯性 (6 項) [Rule #47]

| ID | 項目 | 通過標準 | 等級 | 適用 |
|----|------|----------|:----:|:----:|
| B-1 | **追溯矩陣存在**: 交付包含追溯矩陣 | 矩陣覆蓋所有 wave 的每個項目 | **MUST** | Even+Odd |
| B-2 | **追溯鏈驗證**: 矩陣中每一行可追溯到程式碼 | 每行: 宣稱 → 路徑 → 程式碼行 → 測試案例 | **MUST** | Even |
| B-3 | **依賴鏈驗證**: 變更項的依賴鏈已追溯 | 上游/下游依賴已識別且驗證 | **MUST** | Even |
| B-4 | **LOC 會計**: 宣稱 LOC 與實際 LOC 差異 < 10% | 計數驗證，差異已解釋 | **SHOULD** | Even+Odd |
| B-5 | **Wave 順序合理性**: wave 順序與依賴關係一致 | 無循環依賴，關鍵路徑已識別 | **SHOULD** | Even+Odd |
| B-6 | **前版參照**: 交付報告參照前版本號和 Plan | 明確指出基線版本 | **MUST** | Even+Odd |

---

## 4. Category C: 測試覆蓋 (4 項)

| ID | 項目 | 通過標準 | 等級 | 適用 |
|----|------|----------|:----:|:----:|
| C-1 | **Pass/Fail 比率**: 所有測試通過 | 0 failures in verbose log | **MUST** | Even+Odd |
| C-2 | **新增測試清單**: 新增的測試檔案已列出 | 清單存在且與 A-2 一致 | **SHOULD** | Even+Odd |
| C-3 | **刪除測試清單**: 刪除的測試檔案已列出並說明 | 刪除原因已記錄 | **SHOULD** | Even+Odd |
| C-4 | **覆蓋率 delta**: 覆蓋率變化已報告 | 數值存在，delta 已解釋 | **SHOULD** | Even |

---

## 5. Category D: 證據格式 (6 項) [Rule #46]

| ID | 項目 | 通過標準 | 等級 | 適用 |
|----|------|----------|:----:|:----:|
| D-1 | **Raw log 存在**: 完整 `npm test --verbose` log | Log 存在，非截斷 | **MUST** | Even+Odd |
| D-2 | **Structured summary 存在**: pass/fail 摘要 | 摘要與 log 數據一致 | **MUST** | Even+Odd |
| D-3 | **時間戳**: Raw log 包含 ISO 8601 UTC 時間戳 | 時間戳存在且合理 | **SHOULD** | Even+Odd |
| D-4 | **退出碼**: 測試執行的退出碼已記錄 | 0 = 全通過，非零已解釋 | **SHOULD** | Even+Odd |
| D-5 | **快照可執行性**: `npm install && npm test` 可在快照上執行 | 驗證人可復現 | **Cond. MUST** | Even+Odd |
| D-6 | **測試 artifact 交付**: 目錄含實際測試檔案 | `__tests__/` 非空目錄 | **MUST** | Even+Odd |

---

## 6. Category E: 安全 (5 項)

| ID | 項目 | 通過標準 | 等級 | 適用 |
|----|------|----------|:----:|:----:|
| E-1 | **新攻擊面**: 新增的攻擊面已識別 | 安全審閱覆蓋所有新功能 | **SHOULD** | Even+Odd |
| E-2 | **依賴審計**: 新增/更新的依賴已審計 | 無已知 CVE，版本合理 | **SHOULD** | Even+Odd |
| E-3 | **金鑰管理**: 加密金鑰處理正確 | 金鑰不洩漏、不 log、使用後清理 | **MUST** | Even+Odd |
| E-4 | **權限模型**: 新增的權限/存取控制已審閱 | 最小權限原則 | **SHOULD** | Even+Odd |
| E-5 | **HMAC 金鑰清理**: env var 讀取後 delete | `process.env` 清理已驗證 | **MUST** | Even+Odd |

---

## 7. 統計摘要

| 類別 | 項目數 | MUST | SHOULD | Cond. |
|------|:------:|:----:|:------:|:-----:|
| A: 造假偵測 | 8 | 8 | 0 | 0 |
| B: 可追溯性 | 6 | 4 | 2 | 0 |
| C: 測試覆蓋 | 4 | 1 | 3 | 0 |
| D: 證據格式 | 6 | 3 | 2 | 1 |
| E: 安全 | 5 | 2 | 3 | 0 |
| **合計** | **29** | **18** | **10** | **1** |

---

## 8. 版本歷史

| 版本 | Cycle | 項目數 | 主要變更 |
|------|-------|:------:|----------|
| v1.0 | 03-4 | 25 | 初始版本 |
| v1.1 | 03-5 | 29 | +A-8, +D-5, +D-6, +E-5 (Plan40 發現) |

---

## 9. 使用指引

### 9.1 驗證流程

```
1. 取得交付報告 + 程式碼快照
2. 執行 Category A (造假偵測) — 阻斷閘門
3. 若 D-5 失敗 → 標記 DEFERRED 項目
4. 執行 Category B (可追溯性)
5. 執行 Category C (測試覆蓋)
6. 執行 Category D (證據格式)
7. 執行 Category E (安全)
8. 彙總: MUST 全通過 = Plan 可接受
```

### 9.2 DEFERRED 追蹤

| Plan | D-5/D-6 狀態 | 連續 DEFERRED | 可否 |
|------|:------------:|:-------------:|:----:|
| N | PASS | 0 | 繼續 |
| N | DEFERRED | 1 | 允許 |
| N+1 | DEFERRED | 2 | **阻斷** |
| N+1 | PASS | 0 (重置) | 繼續 |

---

*Reference Document #02 — ENG-FAB Audit Checklist v1.1*
*Cycle 03-5 Research Addition, 2026-04-08*
*GUARDIAN (#11), SYNTHESIST (#1), ARCHIMEDES (#16)*
