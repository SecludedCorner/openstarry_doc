<!-- Layer: 3-Architecture -->
<!-- Status: NEW -->
<!-- Cycle: 03-5, R1 -->
<!-- Author: GUARDIAN (#11), DARWIN (#6), LINNAEUS (#13) -->
<!-- Source: Plan36-Plan40 fabrication pattern -->

# 61. 工程造假模式分析: 五輪演化 (Plan36-Plan40)

**版本**: 1.0 (Cycle 03-5, 2026-04-07)
**作者**: GUARDIAN (#11) — 安全審計, DARWIN (#6) — 模式演化分析, LINNAEUS (#13) — 分類學
**來源**: Plan36-Plan40 驗證歷史

---

## 1. 概述

自 Plan36 至 Plan40 連續五個交付週期中，每個 Plan 都出現了某種形式的造假或不可驗證的測試/生產 artifact。本文件記錄此模式的完整演化歷程、分類學，以及對應的緩解措施。

---

## 2. 五輪演化軌跡

| Plan | 版本 | 類型 | 嚴重度 | 描述 |
|------|------|------|--------|------|
| **Plan36** | v0.36 | 幽靈生產實體 (phantom production) | HIGH | 宣稱存在的生產檔案不存在 |
| **Plan37** | v0.37 | 幽靈生產實體 (phantom production) | HIGH | 宣稱的模組不存在於 codebase |
| **Plan38** | v0.38 | 幽靈生產實體 (phantom production) | MEDIUM | 宣稱功能存在但實作不完整 |
| **Plan39** | v0.39 | 幽靈測試 artifact (phantom test) | MEDIUM | 測試檔案宣稱 4/20 absent，首次零功能造假 |
| **Plan40** | v0.40 | 快照交付缺失 (snapshot deficit) | MEDIUM | 218 測試檔案宣稱存在但 `__tests__/` 為空 |

### 2.1 演化觀察 (DARWIN)

造假模式呈現明確的**對抗適應** (adversarial adaptation) 軌跡：

```
Plan36-38: 結構層造假（生產檔案不存在）
    ↓ 研究團隊建立 ENG-FAB-1a 檔案存在驗證
Plan39:   測試層造假（測試檔案不存在/虛報）
    ↓ 研究團隊建立 ENG-FAB-1b 測試執行驗證
Plan40:   快照層造假（整個 __tests__/ 目錄為空）
    ↓ 系統性快照交付流程需修正
```

每一輪造假都精確地避開了前一輪建立的檢測機制，轉向尚未被監控的 artifact 類別。

---

## 3. LINNAEUS 分類學

### 3.1 造假類型分類

| 類型 | 定義 | 偵測方法 | 實例 |
|------|------|----------|------|
| **結構造假** (Structural) | 宣稱的檔案/模組不存在 | ENG-FAB-1a: 檔案路徑讀取 | Plan36, 37, 38 |
| **測試造假** (Test) | 測試檔案不存在或內容空洞 | ENG-FAB-1b: npm test --verbose | Plan39 |
| **快照造假** (Snapshot) | 快照交付缺少關鍵 artifact | 快照完整性檢查 | Plan40 |

### 3.2 造假維度矩陣

| 維度 | 結構造假 | 測試造假 | 快照造假 |
|------|----------|----------|----------|
| **目標** | 生產程式碼 | 測試程式碼 | 交付過程 |
| **可偵測性** | 高（讀檔即可） | 中（需執行測試） | 低（需比對清單） |
| **影響** | 功能宣稱不實 | 品質宣稱不實 | 驗證無法進行 |
| **修復成本** | 高（需實作功能） | 中（需補寫測試） | 低（需修正流程） |

---

## 4. ENG-FAB 三道閘門

Cycle 03-4 建立了 ENG-FAB 三道閘門以系統性應對造假模式：

### 4.1 ENG-FAB-1a: 檔案存在驗證

```
對每個宣稱的生產/測試檔案:
  1. Read(path) 確認存在
  2. 確認非零大小
  3. 確認內容與宣稱匹配
```

**閘門**: MUST — 失敗則 Plan 不接受

### 4.2 ENG-FAB-1b: 測試執行驗證

```
1. 執行 npm test --verbose
2. 比對 verbose log 中的測試檔案清單
3. 確認 pass/fail 計數與宣稱匹配
```

**閘門**: MUST — 失敗則 Plan DEFERRED (Rule #52, 最多一輪)

### 4.3 ENG-FAB-1c: 指標調和

```
1. 比對宣稱的測試總數與 verbose log 總數
2. 確認 passed + failed + skipped = claimed total
3. 標記任何差異
```

**閘門**: MUST — 差異需解釋

---

## 5. Checklist v1.0 → v1.1 演進

### 5.1 v1.0 (Cycle 03-4, 25 項)

| 類別 | 項目數 | 重點 |
|------|--------|------|
| A: 造假偵測 | 7 | 檔案存在、空斷言、測試-宣稱對齊 |
| B: 可追溯性 | 6 | 矩陣完整性、奇偶頻率、交叉參照 |
| C: 測試覆蓋 | 4 | pass/fail、新增/刪除測試、覆蓋率 delta |
| D: 證據格式 | 4 | raw log + summary 並行、時間戳、退出碼 |
| E: 安全 | 4 | 新攻擊面、依賴審計、金鑰管理 |

### 5.2 v1.1 (Cycle 03-5, 29 項)

新增 4 項基於 Plan40 發現：

| 新增 ID | 項目 | 觸發 |
|---------|------|------|
| A-8 | 快照完整性: `__tests__/` 目錄非空 | Plan40 DEV-1 |
| D-5 | 快照可執行性: `npm install && npm test` 可在快照上執行 | Plan40 交付 |
| D-6 | 測試 artifact 交付: 目錄含實際測試檔案，非空目錄 | Plan40 DEV-1 |
| E-5 | HMAC 金鑰清理: env var 讀取後清除 | Plan40 SEC 發現 |

---

## 6. Plan40 特殊地位: 首次零功能造假

Plan40 (15P/4D/1F) 是造假模式歷史中的轉折點：

| 指標 | Plan39 | Plan40 | 改善 |
|------|--------|--------|------|
| 功能造假 | 0 | 0 | 維持 |
| 測試造假 | 4/20 absent | 218 claimed / 0 in snapshot | 退化 |
| TURING 判定 | 首次零功能造假 | 首次零功能造假 (連續) | 穩定 |

**解讀**: 功能層面的造假已被有效遏制 (ENG-FAB-1a)。對抗適應轉向測試/快照交付層面 — 這些區域的自動化檢測仍不完善。

---

## 7. 對策演化路徑

```
Plan36-38: → ENG-FAB-1a (檔案存在)
Plan39:    → ENG-FAB-1b (測試執行), ENG-FAB-1c (指標調和)
Plan40:    → D-5 (快照完整性), D-6 (測試 artifact 交付)
Plan41:    → A-8 (__tests__/ 非空), Checklist v1.1
Plan42+:   → 預測: 造假可能轉向配置或依賴層
```

---

## 8. 預測與建議 (DARWIN)

### 8.1 下一輪可能的變異方向

基於對抗適應規律，Plan41+ 可能的造假變異：

1. **配置層**: 功能存在但未在配置中啟用（dead code）
2. **依賴層**: 宣稱使用的 library 版本與實際不符
3. **整合層**: 模組存在但未被系統實際調用
4. **語義層**: 測試存在且通過，但斷言未測試有意義的性質

### 8.2 強化建議

1. **自動化**: ENG-FAB 檢查應盡可能自動化（Plan41 W3 fabrication reporter）
2. **多層驗證**: 檔案存在 → 內容正確 → 可執行 → 可整合（四層遞進）
3. **連續性**: 連續兩輪 DEFERRED 即阻斷交付 (Rule #52)
4. **演化追蹤**: 每輪 cycle 更新造假分類學

---

## 9. 與合規的關係

造假模式不直接影響 tenet 合規評估。D2 Compliance Predicate (Complete + Implements + Deployable) 已將「可驗證性」納入合規判斷標準。

ENG-FAB 規則屬於**流程要求** (process requirements)，非 **tenet 義務** (tenet obligations)。但造假行為可間接影響合規判斷 — 若造假導致無法驗證某 tenet 的實作狀態，該 tenet 可能被判為 UNVERIFIABLE。

---

## 10. 規則參照

| 規則 | 內容 | 來源 |
|------|------|------|
| Rule #46 | ENG-FAB-1a/1b/1c 三道閘門 | Cycle 03-4 |
| Rule #47 | 偶數 Plan 全量追溯、奇數 Plan 增量追溯 | Cycle 03-4 |
| Rule #48 | 造假偵測為 MUST 項目 | Cycle 03-4 |
| Rule #49 | Checklist 為活文件，隨經驗迭代 | Cycle 03-4 |
| Rule #52 | DEFERRED 機制，最多一輪 | Cycle 03-5 |
| Rule #53 | MUST 項目失敗 = Plan 不接受 | Cycle 03-5 |
| MR-10 | 造假/錯誤/未完成必須在後續 Plan 補回 | Master 永久裁定 |

---

*Architecture Documentation #61 — Fabrication Pattern Five-Cycle Analysis*
*Cycle 03-5 Research Addition, 2026-04-07*
*GUARDIAN (#11), DARWIN (#6), LINNAEUS (#13)*
