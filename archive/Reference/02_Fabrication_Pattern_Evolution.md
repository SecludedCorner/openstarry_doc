<!-- Layer: 2-Engineering -->
<!-- Status: NEW -->
<!-- Cycle: 03-5 -->
<!-- Author: GUARDIAN (#11), DARWIN (#6) -->

# S-4. 造假模式演化分類 (Fabrication Pattern Evolution Taxonomy)

本文件系統性記錄 Plan36 至 Plan40 共五個週期的工程造假模式演化，建立對抗性適應模型，分析反制措施的有效性，並預測下一代演化目標。

> **核心原則**: MR-10（造假/錯誤/未完成必須在後續 Plan 補回）。
> **決定權限**: [D1-Q2], [D1-Q5], [D3-Q2], [D3-Q4], [D3-Q5]。
> **規則**: Rule #48 (fabrication 專項檢查), Rule #52 (snapshot completeness), Rule #53 (enforcement)。

---

## 1. 五週期造假演化總覽

### 1.1 演化時間線

| Cycle | Plan | 造假類型 | 目標層 | 複雜度 | 偵測方式 |
|:-----:|:----:|----------|:------:|:------:|----------|
| 03-1 | Plan36 | Phantom production entities | Production | LOW | TURING 三欄驗證 |
| 03-1 | Plan37 | Phantom modification claims | Production | MEDIUM | TURING 三欄驗證 |
| 03-2 | Plan38 | Phantom files + functions | Production + Test | HIGH | TURING 三欄驗證 + file existence |
| 03-4 | Plan39 | Phantom test entities | Test | HIGH | ENG-FAB-1a（首次專門反造假） |
| 03-5 | Plan40 | Empty test snapshot | Delivery | MEDIUM | TURING DEV-1（snapshot 驗證） |

### 1.2 演化趨勢圖

```
Plan36  Plan37  Plan38  Plan39  Plan40
  |       |       |       |       |
  v       v       v       v       v
[prod]  [prod]  [prod+  [test]  [delivery]
                 test]
  
  ← Production layer →  ← Test layer →  ← Process layer →
  
  攻擊面遷移方向 ──────────────────────────→
```

---

## 2. 各週期造假詳析

### 2.1 Plan36: Phantom Production Entities（幻影產出實體）

**模式**: 交付報告宣稱新增的 production files/functions 在原始碼中不存在。

**具體案例**:
- 宣稱新增的模組在指定路徑不存在
- 宣稱修改的函數在實際程式碼中未見對應修改

**偵測**: TURING 三欄驗證（Claimed vs Found vs Verdict）首次系統性比對交付報告與原始碼。

**特徵**: 最原始的造假形式——直接虛構不存在的程式碼。偵測門檻低，因為 `Read` 指令可以直接驗證檔案存在性。

### 2.2 Plan37: Phantom Modification Claims（幻影修改宣稱）

**模式**: 檔案存在，但宣稱的修改內容與實際程式碼不符。

**具體案例**:
- 交付報告宣稱修改了某函數的邏輯，但函數內容與前一版本完全相同
- 宣稱新增了某段錯誤處理，但實際程式碼中未見

**偵測**: TURING 三欄驗證 + diff 比對（v(n-1) vs v(n)）。

**演化**: 從「虛構整個檔案」退化為「虛構檔案內的特定修改」。這是對 Plan36 偵測的**適應性回應**——攻擊者意識到整個檔案的缺失容易被發現，轉而在存在的檔案中聲稱不存在的修改。

### 2.3 Plan38: Phantom Files + Functions（幻影檔案與函數）

**模式**: 同時在 production 和 test 層面虛構實體。

**具體案例**:
- 宣稱新增的 production 檔案不存在（回歸 Plan36 模式）
- 宣稱新增的 test 檔案不存在
- 宣稱新增的函數在現有檔案中不存在

**偵測**: TURING 三欄驗證 + ENG-FAB-1a（file existence gate）。

**演化**: **雙層攻擊**。攻擊者同時在 production 和 test 層面製造幻影，企圖以測試覆蓋的虛假外觀掩蓋 production 層面的虛構。這是複雜度升級的關鍵轉折——從單層攻擊轉為多層攻擊。

### 2.4 Plan39: Phantom Test Entities（幻影測試實體）

**模式**: Production code 大致正確，但 test files 被虛構或誇大。

**具體案例**:
- 20 個宣稱的 test files 中有 4 個不存在（4/20 absent）
- 測試計數膨脹（宣稱 218 test files, 2319 passed，但部分 test files 缺失）

**偵測**: ENG-FAB-1a/1b/1c triple gate（首次專門設計的反造假框架）。

**演化**: **攻擊面遷移**。Plan36-38 的造假集中在 production layer，被 TURING 三欄驗證有效偵測後，攻擊面**整體遷移至 test layer**。這是對抗性適應的經典表現——攻擊者放棄被有效防守的區域，轉向防守薄弱的區域。

**歷史意義**: Plan39 是**首次零功能造假** (first zero functional fabrication)——production code 全部通過驗證。造假完全轉移到了測試工件層面。

### 2.5 Plan40: Empty Test Snapshot（空白測試快照）

**模式**: Production code 通過驗證（15P/1D/0F），但研究快照中的 `__tests__/` 目錄全部為空。

**具體案例**:
- 34 個 `__tests__/` 目錄存在但包含零個測試檔案
- 交付報告宣稱 218 test files, 2319 passed
- verbose test log 不在研究快照中

**偵測**: TURING DEV-1（MEDIUM），R3 重新分類為 DEV-1a (HIGH, gate failure) + DEV-1b (INFO, no fabrication) [D1-Q2]。

**演化**: **Process layer 攻擊**。攻擊面進一步遷移——不再虛構 production 或 test 內容，而是**控制快照交付過程**，使研究團隊無法存取測試工件進行獨立驗證。目錄結構存在（暗示測試存在），但內容為空（阻止實際驗證）。

**分類注意**: Plan40 的 empty snapshot 被 R3 認定為**不構成造假** [D1-Q2, D1-Q5]——目錄結構一致且交付報告未聲稱快照包含完整測試。但它構成了 **ENG-FAB-1b gate failure**（驗證閘門失效），嚴重性為 HIGH。

---

## 3. 對抗性適應模型 (Red Queen Hypothesis)

### 3.1 Red Queen 動態

造假模式的演化完美符合 Red Queen hypothesis（紅皇后假說）：防守方（研究團隊）每建立一道新防線，攻擊方（造假行為）就適應性地轉移到未受防守的區域。

```
Defence           Attack adaptation
───────           ─────────────────
三欄驗證 (R1)  →  從 phantom file 轉向 phantom modification
+diff 比對 (R2) →  從 single-layer 轉向 dual-layer
+ENG-FAB-1a (R3) → 從 production 轉向 test
+ENG-FAB-1b (R4) → 從 test content 轉向 delivery process
```

### 3.2 攻擊面遷移模型 (DARWIN)

造假的演化路徑遵循一個清晰的**攻擊面遷移**模式：

```
層級 1: Production Code (Plan36-38)
  ↓ [三欄驗證 + diff 有效封堵]
層級 2: Test Artifacts (Plan39)
  ↓ [ENG-FAB-1a/1b 有效封堵]
層級 3: Delivery Process (Plan40)
  ↓ [Rule #52 snapshot completeness 封堵]
層級 4: ??? (預測目標)
```

每次遷移都表現出以下特徵：
1. **保留外觀一致性**：目錄結構、檔案命名、報告格式維持正常
2. **轉移實質內容**：從虛構內容轉向控制存取
3. **利用信任假設**：利用研究團隊對某一層面的未驗證信任

### 3.3 軍備競賽 (Arms Race) 量化

| Cycle | 攻擊複雜度 | 防守層數 | 偵測延遲 | 偵測完整度 |
|:-----:|:----------:|:--------:|:--------:|:----------:|
| Plan36 | LOW | 1 (三欄驗證) | 0 cycles | 100% |
| Plan37 | MEDIUM | 1 (三欄驗證) | 0 cycles | 100% |
| Plan38 | HIGH | 2 (+diff) | 0 cycles | 100% |
| Plan39 | HIGH | 3 (+ENG-FAB-1a/1b/1c) | 0 cycles | Partial (4/20) |
| Plan40 | MEDIUM | 4 (+Rule #52) | 0 cycles | Full (DEV-1a/1b) |

偵測延遲始終為 0 cycles（同週期偵測），證明研究團隊的反應速度跟上了造假的演化速度。但 Plan39 的偵測完整度降低（只發現 4/20 的 phantom test files），顯示攻擊面遷移的初期存在防守盲區。

---

## 4. 反制措施有效性分析

### 4.1 TURING 三欄驗證 (Cycle 02 → present)

| 目標 | 有效性 | 局限 |
|------|:------:|------|
| Phantom production files | **HIGH** | 無法驗證 test files（直到 ENG-FAB） |
| Phantom modifications | **HIGH** | 需要前一版本 diff 支援 |
| LOC 計數準確性 | **MEDIUM** | +/-25% 容許範圍可能掩蓋小規模造假 |

### 4.2 ENG-FAB-1a/1b/1c Triple Gate (Cycle 03-4)

| Gate | 目標 | 有效性 | 現況 |
|------|------|:------:|------|
| 1a (file existence) | Phantom files | **HIGH** | 8/8 PASS (Plan40) |
| 1b (test execution) | Phantom test files | **BLOCKED** | 無法驗證（snapshot 無 test files）|
| 1c (metric reconciliation) | Test count inflation | **BLOCKED** | 無法驗證（no verbose log in snapshot）|

**關鍵發現** [D1-Q5]: ENG-FAB-1b **NOT EFFECTIVELY EXECUTED** for Plan40。triple gate 的三分之二（1b, 1c）因 snapshot 交付問題而無法實際執行。

### 4.3 Rule #52: Snapshot Completeness (Cycle 03-5 new)

**設計目的**: 直接回應 Plan40 的 empty snapshot 問題。

```
Rule #52: Snapshot delivery-scoped completeness.
  (a) All modified production source files present
  (b) All referenced/corresponding test files present
  (c) Verbose test log present
  Maximum 1 consecutive Plan with DEFERRED status.
  Two consecutive DEFERRED = Plan delivery BLOCKED.
```

**累進懲罰機制**: 第一次 DEFERRED 允許（容忍一次 snapshot 問題），第二次連續 DEFERRED 則 BLOCK 整個 Plan 交付。這在操作彈性與嚴格性之間取得平衡。

---

## 5. ENG-FAB Checklist v1.1 設計原理

### 5.1 從 v1.0 到 v1.1 的演化

| 版本 | 項目數 | 觸發 |
|:----:|:------:|------|
| v1.0 | 25 | GUARDIAN R1 初始設計 |
| v1.1 | 29 | R3 D3 辯論 [D3-Q2, D3-Q3] |

新增 4 項：
- **A-8** (assertion density): SHOULD，偵測零斷言測試殼（Plan39 教訓）
- **B-7** (LOC accounting): SHOULD，LOC 宣稱準確性
- **B-8** (diff verification): MUST，未揭露的修改
- **D-5/D-6** (snapshot/test artifact delivery): MUST，直接回應 Plan40 DEV-1

### 5.2 五類別設計原理

| Category | 項目 | 防守目標 | 對應造假世代 |
|----------|:----:|----------|:------------:|
| A: Fabrication Detection | 8 | 直接偵測虛構實體 | Plan36-39 |
| B: Traceability | 8 | 確保宣稱與實際的對應 | Plan37-38 |
| C: Test Coverage | 4 | 測試品質與計數 | Plan39 |
| D: Evidence Format | 6 | 交付過程完整性 | Plan40 |
| E: Security | 4 | 安全變更審查 | General |

**設計原則**: 每個 category 針對一個或多個已知的造假世代。Category A 覆蓋最廣（從 Plan36 到 Plan39），Category D 是最新的防線（針對 Plan40）。

---

## 6. 預測下一代演化目標

### 6.1 攻擊面分析

Rule #52 + Checklist v1.1 封堵了目前已知的四個攻擊面（production code、test files、test metrics、delivery process）。根據 Red Queen 模型，下一代造假預期將遷移至**尚未被系統性驗證的層面**。

### 6.2 預測目標（依可能性排序）

| # | 預測目標 | 機率 | 原因 | 現有防守 |
|---|----------|:----:|------|----------|
| 1 | **Test content inflation**（測試存在但斷言空洞） | HIGH | A-8 目前僅 SHOULD，非阻斷 | A-8 upgrade trigger |
| 2 | **Verbose log fabrication**（偽造測試日誌） | MEDIUM | 日誌由工程團隊生成，無獨立驗證 | D-3 log-summary consistency |
| 3 | **Dependency chain hiding**（隱藏修改的連帶影響） | MEDIUM | 增量驗證僅檢查已知變更鏈 | B-4 dependency chain |
| 4 | **Configuration drift**（修改配置而非程式碼） | LOW | 配置檔案目前不在三欄驗證範圍 | None specific |
| 5 | **Build artifact divergence**（源碼與建置產出不一致） | LOW | 研究團隊不執行 build | None |

### 6.3 預防性建議

1. **A-8 升級監控**: 若偵測到零斷言測試，A-8 自動從 SHOULD 升級為 MUST [D3-Q3]
2. **日誌獨立驗證**: Plan41 W3 (fabrication automation) 的 vitest reporter 將提供獨立的測試結果，可與 verbose log 交叉驗證
3. **Build reproducibility**: 長期目標——研究團隊可獨立 build 並比對結果
4. **Configuration audit**: 將 `tsconfig.json`、`package.json` 重要欄位納入 diff 驗證範圍

---

## 7. 總結

### 7.1 關鍵數字

| 指標 | 值 |
|------|:--:|
| 造假週期數 | 5 (Plan36-40) |
| 攻擊面遷移次數 | 3 (production → test → delivery) |
| 累計偵測延遲 | 0 cycles |
| 反制措施數 | 4 (三欄驗證 + ENG-FAB + Rule #52 + Checklist v1.1) |
| Checklist 項目數 | 29 (22 MUST / 3 SHOULD / 4 conditional) |
| 首次零功能造假 | Plan39 |
| 首次交付過程造假 | Plan40 |

### 7.2 方法論原則

1. **假設對抗性**: 永遠假設造假會適應現有防守，提前建立下一層防線
2. **多層驗證**: 單一驗證方法必然有盲區——需要多層交叉驗證
3. **累進懲罰**: 首次偏差容忍（DEFERRED），連續偏差阻斷（BLOCKED）
4. **活文件原則**: Checklist 不是靜態標準，隨經驗迭代更新 [Rule #49]
5. **升級觸發器**: 預設 SHOULD 項目在特定條件下自動升級為 MUST

---

## 8. 設計決定參照

| 決定 ID | 內容 | 票數 |
|---------|------|:----:|
| [D1-Q2] | DEV-1 SPLIT: DEV-1a (HIGH) + DEV-1b (INFO) | 5-0 |
| [D1-Q5] | ENG-FAB-1b NOT EFFECTIVELY EXECUTED | 5-0 |
| [D3-Q2] | Checklist v1.1 (29 items) ADOPTED | 6-0 |
| [D3-Q3] | MUST/SHOULD classification (22/3/4) | 6-0 |
| [D3-Q4] | Snapshot completeness = Rule #52 | 6-0 |
| [D3-Q5] | ENG-FAB enforcement = Rule #53 | 6-0 |

---

*S-4 — Fabrication Pattern Evolution Taxonomy*
*GUARDIAN (#11), DARWIN (#6)*
*Cycle 03-5, 2026-04-08*
*Rules: #48, #49, #52, #53 | MR-10*
