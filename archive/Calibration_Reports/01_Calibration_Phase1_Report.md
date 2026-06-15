# 63. Calibration Phase 1 Report -- W2-R1/R2/R3 Complete Trajectory

`[Cycle 03-4 新增: Phase 1 校準完整技術紀錄]`

> Phase 1 校準歷經三輪測試（W2-R1、W2-R2、W2-R3），從 sensor 盲點到閉迴路恢復，最終建立 confidence delta 的統計基線。本文件記錄完整軌跡、鎖定參數、SC/CV 結果、sensor 修復歷程、F1/F2 signal loss 根因，以及 CV-5 DEFERRED to Plan41 和 CV-6 TRIGGERED 的決策。
>
> **核心學者**: WIENER (#12), KNUTH (#21), PASCAL (#19), ARCHIMEDES (#16), SUSSMAN (#22)
> **依賴文件**: #50 Calibration Methodology, #60 Audit Path B-Modified Delta, #42 IGearArbiter Interface Spec
> **研究來源**: R1 WIENER calibration, R1 KNUTH+PASCAL statistics, R3 D3 calibration debate, 測試團隊 coordinator report

---

## 1. 校準目標

Phase 1 校準的目標是建立 **confidence delta 的統計基線** -- 驗證五層信心度模型（C1）的 L2 (IConfidenceAuditor) 層在受控條件下，對四個風險類別（informational, read_only, state_modifying, destructive）產生方向正確且統計穩定的 delta 信號。

具體而言：

1. **Sigma > 0**: 確認系統輸出的 clamped delta 具有可量測的變異性（非全零）
2. **n_eff 充分**: 有效樣本量足以進行統計推論
3. **符號正確**: 低風險操作產生正 delta（增加信心），高風險操作產生負 delta（降低信心）
4. **雙向分布**: 正負 delta 共存，確認系統具備雙向回饋能力
5. **F1/F2 信號損失可控**: pre-clamp 與 post-clamp sigma 比率在可接受範圍

Phase 1 採用 5-block 50-cycle 測試設計，每輪涵蓋全部四個風險類別的獨立和混合操作場景。

[來源: Calibration_Reports/03_Calibration_Methodology.md#Section 4]

---

## 2. 三輪軌跡

### 2.1 總覽

```
W2-R1 (v0.38)   2026-04-04     GO    sensor blind spot -- 62% delta=0
    |
    v   Plan39 sensor fix (B-modified delta injection)
    |
W2-R2 (v0.39)   2026-04-04     IF    delta sign inversion -- SC-3 FAIL
    |
    v   signMultiplier hotfix (1-2 LOC)
    |
W2-R3 (hotfix)   2026-04-04    GO    Phase 1 completed -- 6/6 SC, 6/7 CV
```

### 2.2 三輪演進總表

| 指標 | W2-R1 (v0.38) | W2-R2 (v0.39) | W2-R3 (hotfix) | 趨勢 |
|------|:---:|:---:|:---:|:---:|
| Verdict | GO | IF | **GO** | 恢復 |
| sigma (entry) | 0.001869 | 0.023276 | **0.027389** | 收斂 |
| sigma F1 (pre-clamp) | =F2 | 0.382425 | **0.561286** | 增（雙向） |
| sigma F2 (post-clamp) | 0.001869 | 0.026840 | **0.042659** | 增 |
| n_eff/N | 37.9% | 100% | **100%** | 鎖定 |
| destructive n_eff | 0 | 11 | **12** | 修復 |
| negative freq | 3.2% | 3.4% | **19.7%** | sign fix |
| Block 4 mean | 0.000 | +0.021 | **-0.016** | 修復 |
| SC | 6/6 | 5/6 | **6/6** | 恢復 |
| CV | 1/5 | 5/7 | **6/7** | 改善 |
| Clamp ceiling/floor | 0/0 | 38/0 | **17/21** | 平衡 |
| F1/F2 ratio | 1.0x | 14.2x | **13.2x** | 高（待解決） |

[來源: R1_WIENER_calibration.md#Section 1.2]

### 2.3 W2-R1 (v0.38): GO -- Sensor Blind Spot

**Verdict**: GO（滿足 SC，但嚴重 sensor 缺陷）

W2-R1 揭示了 B-modified path 的根本問題：62% 的 `tool_audited` 事件產生 delta=0。在控制理論中，這等價於 **sensor 輸出為零的開迴路運行** -- 系統對多數事件沒有回饋修正能力。

- **影響範圍**: 所有 B-modified path 事件（read_only, state_modifying, destructive 的 tool_audited）
- **僅 confidence_audited path**（arbiter route）能產生非零 delta
- **destructive 完全不可見**（n_eff=0）

sigma=0.001869 是 sensor 輸出幾乎全為零的假低值（measurement floor noise）。真正的系統 sigma 始終在 0.02-0.03 範圍，只是量測不到。

[來源: R1_WIENER_calibration.md#Section 2.2]

### 2.4 W2-R2 (v0.39): IF -- Delta Sign Inversion

**Verdict**: INFORMATIVE FAILURE（SC-3 FAIL）

Plan39 修復了 sensor 盲點（B-modified delta injection），100% 事件產生非零 delta。但暴露了更微妙的問題：**delta 符號反轉**。

Destructive 操作的 delta = +0.05（應為負），意味著每次 destructive 操作**增加**系統信心，而非降低。在控制理論中這是 **正回饋**（positive feedback）：

- `sign(feedback) = sign(disturbance)` -- 回饋放大擾動而非抑制
- Block 3 (state_modifying) mean = +0.025（應為負）
- Block 4 (destructive) mean = +0.021（應為負）
- SC-3 FAIL: destructive 和 state_modifying 的 delta 符號錯誤

根因：confidence 值（正數 0.75, 0.85）被直接用作 delta，未考慮方向。

[來源: R1_WIENER_calibration.md#Section 2.3, coordinator_report_w2r2r3.md]

### 2.5 W2-R3 (hotfix): GO -- Phase 1 Completed

**Verdict**: GO（6/6 SC, 6/7 CV）

signMultiplier hotfix（1-2 行程式碼）修正了符號問題：

```typescript
const signMultiplier = (riskCat === 'destructive' || riskCat === 'state_modifying') ? -1 : 1;
const rawDelta = TOOL_CONFIDENCE_TABLE[riskCat] * signMultiplier;
```

修正後各類別 delta 方向：

| Category | Raw delta | Clamped delta | 方向 |
|----------|:---------:|:------------:|:----:|
| informational | +0.001 | +0.001 | 正確: 低風險 -> 微增 |
| read_only | +0.50 | +0.05 (clamped) | 正確: 低風險 -> 正 |
| state_modifying | -0.75 | -0.05 (clamped) | 正確: 高風險 -> 負 |
| destructive | -0.85 | -0.05 (clamped) | 正確: 高風險 -> 負 |

**閉迴路負回饋已恢復**。Lyapunov 穩定性在正常操作頻率分布下成立。

[來源: R1_WIENER_calibration.md#Section 2.4-2.5, coordinator_report_w2r2r3.md]

### 2.6 Sigma 收斂分析

```
W2-R1  sigma = 0.001869   (sensor blind: measurement floor noise)
W2-R2  sigma = 0.023276   (+12.5x, sensor fix -- 量測能力恢復)
W2-R3  sigma = 0.027389   (+1.18x, sign fix -- 雙向分布展開)
```

R1 -> R2 的 12.5x 跳躍不是系統動態變化，是量測能力恢復（類比：溫度計從壞的換成好的）。R2 -> R3 的 18% 微增完全可歸因於 signMultiplier 帶來的雙向分布。**sigma 已收斂**。

[來源: R1_WIENER_calibration.md#Section 1.1]

---

## 3. 鎖定參數表

W2-R3 校準參數經三方獨立驗證（測試團隊、WIENER、KNUTH），由 R3 D3 辯論 5-0 全票正式鎖定。

### 3.1 Overall Parameters

| 參數 | 值 | 狀態 |
|------|-----|:---:|
| sigma (entry-level, clamped) | **0.027389** | LOCKED |
| sigma F1 (pre-clamp) | **0.561286** | LOCKED |
| sigma F2 (post-clamp) | **0.042659** | LOCKED |
| F1/F2 signal loss ratio | **13.2x** | LOCKED |
| n_eff/N | **100%** (127/127) | LOCKED |
| N (total entries) | **127** | LOCKED |
| Overall mean | **-0.001272** | LOCKED |

### 3.2 Per-Category Parameters

| Category | n | Mean delta | Sigma | Min | Max |
|----------|:---:|:---:|:---:|:---:|:---:|
| informational | 72 | +0.001000 | ~0 | +0.001 | +0.001 |
| read_only | 30 | +0.028550 | 0.024529 | +0.0005 | +0.05 |
| state_modifying | 13 | -0.037692 | 0.018462 | -0.05 | -0.01 |
| destructive | 12 | -0.050000 | ~0 | -0.05 | -0.05 |

### 3.3 Per-Block Statistics (W2-R3)

| Block | Cycles | Focus | Events | Mean | Sigma |
|-------|:------:|-------|:------:|:----:|:-----:|
| Block 1 | 1-10 | Informational | 33 | +0.001000 | 0.000000 |
| Block 2 | 11-20 | Read-Only | 20 | +0.027775 | 0.024571 |
| Block 3 | 21-30 | State-Modifying | 12 | -0.027250 | 0.023080 |
| Block 4 | 31-40 | Destructive | 30 | -0.016000 | 0.024042 |
| Block 5 | 41-50 | Mixed | 32 | +0.001781 | 0.029323 |

### 3.4 Delta Source Composition

| Category | confidence_audited (arbiter) | tool_audited (B-modified) |
|----------|:---:|:---:|
| informational | -- | 72 entries @ +0.001 |
| read_only | 13 entries @ +0.0005 | 17 entries @ +0.05 |
| state_modifying | 4 entries @ -0.01 | 9 entries @ -0.05 |
| destructive | -- | 12 entries @ -0.05 |

[來源: R1_KNUTH_PASCAL_statistics.md#Section 1-4, R3_D3_calibration.md#Issue 1]

---

## 4. SC-1 ~ SC-6 結果

全部 6 項 Success Criteria 經 KNUTH 從 raw audit trail (127 entries) 逐條獨立驗證，與測試團隊判定完全一致。

| SC | Criterion | W2-R3 Result | 獨立驗證 | 說明 |
|----|-----------|:---:|:---:|------|
| SC-1 | sigma > 0 | **PASS** | CONFIRMED | sigma=0.027389 > 0 |
| SC-2 | n_eff >= 10 | **PASS** | CONFIRMED | n_eff=127 >> 10 |
| SC-3 | Correct delta sign per category | **PASS** | CONFIRMED | 全部四類別方向正確 |
| SC-4 | No structural anomaly | **PASS** | CONFIRMED | hash chain 完整，無異常 |
| SC-5 | All 4 risk categories present | **PASS** | CONFIRMED | 72/30/13/12 |
| SC-6 | Bidirectional delta | **PASS** | CONFIRMED | 102 positive + 25 negative |

**SC 獨立驗證結論: 6/6 PASS**

SC-3 在 W2-R2 中 FAIL（destructive 和 state_modifying delta 為正值），signMultiplier hotfix 後在 W2-R3 完全修正。

[來源: R1_KNUTH_PASCAL_statistics.md#Section 1]

---

## 5. CV-1 ~ CV-7 結果

| CV | Criterion | W2-R1 | W2-R2 | W2-R3 | 獨立驗證 |
|----|-----------|:---:|:---:|:---:|:---:|
| CV-1 | sigma > 0 | PASS (0.002) | PASS (0.023) | **PASS** (0.027) | CONFIRMED |
| CV-2 | n_eff/N > 80% | FAIL (37.9%) | PASS (100%) | **PASS** (100%) | CONFIRMED |
| CV-3 | destructive n_eff > 0 | FAIL (0) | PASS (11) | **PASS** (12) | CONFIRMED |
| CV-4 | Block 4 mean < 0 | FAIL (0.000) | FAIL (+0.021) | **PASS** (-0.016) | CONFIRMED |
| CV-5 | Per-category sigma monotonic | N/A | FAIL | **DEFERRED to Plan41** | See Section 8 |
| CV-6 | Asymmetric clamp trigger | N/T | N/T | **TRIGGERED** | See Section 9 |
| CV-7 | F1/F2 reported separately | N/A | PASS | **PASS** | CONFIRMED |

**CV 獨立驗證結論: 6/7 PASS, CV-5 DEFERRED to Plan41 (Master: MR-9 品質第一，不因成本 WAIVE)**

三輪軌跡中的 CV 改善：
- **CV-2**: W2-R1 的 37.9% 因 sensor 盲點導致多數 delta=0，sensor 修復後恢復至 100%
- **CV-3**: W2-R1 的 destructive n_eff=0 因 B-modified path 不發射 delta，修復後恢復
- **CV-4**: W2-R2 的 Block 4 mean=+0.021 因 delta 符號反轉，hotfix 後修正為 -0.016

[來源: R1_KNUTH_PASCAL_statistics.md#Section 2, coordinator_report_w2r2r3.md]

---

## 6. Sensor Fix 歷程

### 6.1 問題發現 (W2-R1)

W2-R1 測試揭示 B-modified path 存在 **audit path blind spot**：

- 62% 的 `tool_audited` 事件產生 delta=0
- 根因：B-modified path 在 `tool_audited` 事件中未注入 delta 值
- 僅 `confidence_audited` path（arbiter route）能產生非零 delta
- **destructive 類別完全不可見**（n_eff=0）

### 6.2 Plan39 Sensor Fix (W2-R2)

Plan39 引入 B-modified delta injection，修復內容：

| 修復項 | 說明 | 驗證狀態 |
|--------|------|:---:|
| fs.delete confidence 0.50 -> 0.85 | 修正 destructive 操作的 confidence 值 | VERIFIED |
| fs.write confidence 0.70 -> 0.75 | 修正 state_modifying 操作的 confidence 值 | VERIFIED |
| B-modified delta injection | tool_audited 事件注入基於 TOOL_CONFIDENCE_TABLE 的 delta | VERIFIED (100% non-zero) |
| Per-cycle clamp reset | cumulativeClampedDelta 在 processInput() 頂部重置 | VERIFIED |

修復後 W2-R2 達成 100% non-zero delta（n_eff/N=100%），但暴露了 delta 符號問題。

### 6.3 SignMultiplier Hotfix (W2-R3)

W2-R2 的 INFORMATIVE FAILURE 根因：confidence 值（正數）被直接用作 delta，未考慮高風險操作應產生負 delta。

Hotfix 在 loop.ts 加入 signMultiplier：

```typescript
const signMultiplier = (riskCat === 'destructive' || riskCat === 'state_modifying') ? -1 : 1;
const rawDelta = TOOL_CONFIDENCE_TABLE[riskCat] * signMultiplier;
```

修復效果（量化）：

| 指標 | W2-R2 (pre-hotfix) | W2-R3 (post-hotfix) | 變化 |
|------|:---:|:---:|------|
| Overall mean | +0.016347 | -0.001272 | 趨近零均值 |
| Block 3 mean | +0.024846 | -0.027250 | 符號翻轉 (delta=0.052) |
| Block 4 mean | +0.020580 | -0.016000 | 符號翻轉 (delta=0.037) |
| Negative entries | 4 (3.4%) | 25 (19.7%) | +21 entries |
| Negative floor hits | 0 | 21 | +21 |
| Positive ceiling hits | 38 | 17 | -21 |
| SC-3 | FAIL | PASS | 修復 |
| CV-4 | FAIL | PASS | 修復 |

Clamp 總啟動次數從 38 維持為 38，但從 100% ceiling 重新分布為 45% ceiling / 55% floor，確認 hotfix 將相同數量的高量級 delta 重導至正確符號。

[來源: R1_WIENER_calibration.md#Section 1.3, R1_KNUTH_PASCAL_statistics.md#Section 7]

---

## 7. F1/F2 Signal Loss: 13.2x 根因與含義

### 7.1 現狀

| 指標 | W2-R2 | W2-R3 |
|------|:---:|:---:|
| F1 sigma (pre-clamp) | 0.382 | **0.561** |
| F2 sigma (post-clamp) | 0.027 | **0.043** |
| F1/F2 | 14.2x | **13.2x** |
| Pre-clamp range | [-0.009, +1.350] | [-0.850, +1.003] |
| Post-clamp range | [0.000, +0.100] | [-0.050, +0.101] |

### 7.2 根因: Dynamic Range Mismatch

F1/F2 = 13.2x 意味著 clamp 過程消除了原始信號中 **92.4% 的資訊**（1 - 1/13.2）。

從資訊理論角度，這代表每個測量週期損失 log2(13.2) = **3.72 bits** 的類別區分資訊（PASCAL 分析）。

根因是 TOOL_CONFIDENCE_TABLE 的值（0.001-0.85）與 clamp 窗口 +/-0.05 之間的 **17:1 dynamic range mismatch**：

```
Raw delta range:     [-0.85, +1.003]    (寬度 1.853)
Clamp window:        [-0.05, +0.05]     (寬度 0.100)
Dynamic range ratio: 18.5:1
```

TOOL_CONFIDENCE_TABLE 的值原設計為 **confidence 分數**（「屬於此風險類別的概率」），其數值尺度（0-1）遠大於合理的 delta 增量尺度（~0.001-0.05）。signMultiplier hotfix 修正了方向，但未修正量級。

### 7.3 Per-Category Compression

| Category | Raw delta | Clamped delta | 壓縮率 |
|----------|:---------:|:------------:|:------:|
| informational | +0.001 | +0.001 | 1:1（無壓縮） |
| read_only | +0.50 | +0.05 | **10:1** |
| state_modifying | -0.75 | -0.05 | **15:1** |
| destructive | -0.85 | -0.05 | **17:1** |

在控制理論術語中，這是 **actuator saturation**（致動器飽和）。四個類別中有三個被 clamp 壓縮到相同絕對值 0.05，系統只能區分 informational（+0.001）和「其他」（+/-0.05）。類別區分度幾乎完全喪失。

### 7.4 Plan B4 解決方案

R3 D3 辯論 5-0 全票採納 **Plan B4: Scaling Factor（alpha=0.055, 僅 tool_audited path）**：

```typescript
// DELTA_SCALING_FACTOR: scales tool_audited rawDelta to fit within clamp window
// Derivation: (maxDelta / max(TOOL_CONFIDENCE_TABLE)) * HEADROOM_FACTOR
//           = (0.05 / 0.85) * 0.935 ~ 0.055
// Ensures: max scaled delta (destructive) = 0.85 * 0.055 = 0.04675 < 0.05
const DELTA_SCALING_FACTOR = 0.055;
```

預期改善：

| 指標 | W2-R3 (current) | Plan B4 (predicted) | 改善 |
|------|:---:|:---:|:---:|
| F1/F2 ratio | 13.2x | ~1.0x | **-92%** |
| Clamp activation rate | ~30% | ~0% | **-100%** |
| Category diff (dest vs sm) | 不可區分 | -0.047 vs -0.041 (13% gap) | **restored** |
| Signal retention | 7.6% | ~100% | **+92pp** |

Plan B4 以 ~2 LOC 實作解決 Cycle 03-3 遺留的最大校準問題。

[來源: R1_WIENER_calibration.md#Section 4, R1_KNUTH_PASCAL_statistics.md#Section 5, R3_D3_calibration.md#Issue 6]

---

## 8. CV-5 DEFERRED to Plan41: 根因分析

### 8.1 問題描述

CV-5 要求 per-category sigma 單調遞增：informational < read_only < state_modifying < destructive。

W2-R3 觀測值：[0.000, 0.025, 0.019, 0.000] -- 不滿足單調性。

### 8.2 根因: Single-Source Delta Architecture

| Category | Delta 來源 | 值 | Sigma 原因 |
|----------|-----------|-----|-----------|
| informational | 僅 TOOL_CONFIDENCE_TABLE (+0.001) | 單一值 | 零方差（常數序列） |
| read_only | arbiter (+0.0005) + table (+0.05) | 兩個來源 | 0.025（有意義） |
| state_modifying | arbiter (-0.01) + table (-0.05) | 兩個來源 | 0.018（有意義） |
| destructive | 僅 table (-0.85 clamped to -0.05) | 單一值 | 零方差（常數序列） |

根因是 **gear-arbiter-static 架構決策**：informational 和 destructive 操作不產生 `audit:completed` 事件，只有 `tool_audited` 事件走 B-modified path。因此這兩個類別各只有一個 delta 值，sigma 數學上必然為零。

### 8.3 Alpha Scaling 無法解決 CV-5

引入 alpha=0.055 後，informational 仍為單一值（+0.000055），destructive 仍為單一值（-0.04675 或 clamped -0.05）。根因在架構層，非 scaling 層。

### 8.4 R3 D3 決議

**DECISION Q5: CV-5 正式判定為 DEFERRED to Plan41 (Master: MR-9 品質第一，不因成本 WAIVE)**。

> **Master Override (2026-04-06)**: R3 原始決議為 WAIVED (known limitation)，5-0 全票。Master 依 MR-9「所有 Plan 確保穩健實現，不因成本而 WAIVE」否決 WAIVED，改為 DEFERRED to Plan41，列為 P0。

- 分類：`DEFERRED to Plan41 (P0)`
- 根因：single-source delta for informational and destructive（架構限制）
- 安全影響：無 -- sigma=0 是確定性回饋，在控制理論中比隨機回饋更容易分析
- 穩定性影響：無 -- 只要 mean delta 方向正確（已通過 SC-3），Lyapunov 穩定性不受影響
- Criterion 定義保留（不刪除），Plan41 必須修復
- 修復成本：90-150 LOC, 7-12 hrs, HIGH risk

[來源: R1_WIENER_calibration.md#Section 5, R1_KNUTH_PASCAL_statistics.md#Section 6, R3_D3_calibration.md#Issue 5]

---

## 9. CV-6 TRIGGERED: 非對稱 Clamp 條件滿足

### 9.1 觸發條件

CV-6 定義了兩個 D3-R2 條件，任一滿足即觸發：

| 條件 | 門檻 | W2-R3 觀測值 | 狀態 |
|------|------|:---:|:---:|
| 負 delta 頻率 | >= 5% | **19.7%** (25/127) | **MET** |
| Block 3-4 floor 飽和率 | >= 10% | **38.1%** (16/42) | **MET** |

兩個條件皆滿足。CV-6 TRIGGERED。

### 9.2 方案評估

**方案 A (Floor Expansion): REJECTED**

擴大 clamp floor 從 -0.05 到 -0.10 或 -0.15 無法解決問題：

1. raw delta = -0.85 被 clamp 到 -0.10 仍是 8.5:1 壓縮。信號損失根因未解決。
2. 破壞對稱性。非對稱 clamp [-0.15, +0.05] 引入非對稱增益，使穩定性分析複雜化。
3. destructive (-0.85 -> -0.10) 和 state_modifying (-0.75 -> -0.10) 在 floor=-0.10 時仍然不可區分。
4. TOOL_CONFIDENCE_TABLE 值每次改變都需重新調整 floor。

**方案 B4 (Scaling Factor): ADOPTED**

R3 D3 辯論 5-0 全票採納 Plan B4（alpha=0.055, 僅 tool_audited path）。

關鍵工程事實（ARCHIMEDES 澄清）：方案 A 和方案 B4 在 **code diff 上路徑不同** -- B-modified path 天然只處理 tool_audited 事件，confidence_audited 走另一條路徑（arbiter route），不需要額外 if 判斷。架構天然隔離。

PASCAL 期望效用分析：EU(B4) = 0.81 > EU(A1) = 0.37。B4 在所有合理權重配置中勝出。

[來源: R1_WIENER_calibration.md#Section 3, R1_KNUTH_PASCAL_statistics.md#Section 8, R3_D3_calibration.md#Issue 2]

---

## 10. Bayesian Posterior Assessment

PASCAL 以 Bayesian 框架評估校準有效性：

```
Prior:      P(valid) = 0.7 (moderate, reflecting W2-R2 failure + root cause identified)
Likelihood: P(data | valid)   ~ 0.95
            P(data | invalid) ~ 0.05

Posterior:  P(valid | W2-R3 data)
          = (0.95 * 0.7) / (0.95 * 0.7 + 0.05 * 0.3)
          = 0.665 / 0.680
          = 0.978
```

即使以保守先驗 P(valid) = 0.5 計算，posterior 仍達 **0.95**。

W2-R3 數據強烈支持校準有效性。W2-R4 通過後預計更新至 >= 0.99，Phase 1 校準正式鎖定。

[來源: R1_KNUTH_PASCAL_statistics.md#Section 5.4, R3_D3_calibration.md#Issue 1]

---

## 11. 控制迴路完整性評估

### 11.1 閉迴路結構

OpenStarry 的 confidence audit loop 建模為經典離散時間控制迴路：

```
Reference (ideal risk) -> [+] -> Controller (clamp) -> Plant (confidence state) -> Sensor (audit) -> [-]
                           ^                                                             |
                           +------------------------ feedback ----------------------------+
```

### 11.2 三輪控制迴路狀態

| 輪次 | Sensor | Controller | Feedback Path | Loop Status |
|------|--------|-----------|---------------|-------------|
| W2-R1 | **BROKEN** (62% blind) | Functional | Partial (38%) | **Open loop** for 62% |
| W2-R2 | Fixed (100% non-zero) | Functional | **Inverted** | **Positive feedback** for high-risk |
| W2-R3 | Fixed | Functional | Correct | **Closed loop restored** |

### 11.3 Lyapunov 穩定性

WIENER 以 Lyapunov 候選函數 V(k) = (confidence(k) - confidence_target)^2 分析穩定性：

W2-R3 四類別加權平均 delta（以類別頻率 72:30:13:12）：

```
(72 * 0.001 + 30 * 0.029 + 13 * (-0.038) + 12 * (-0.050)) / 127
= (0.072 + 0.870 - 0.494 - 0.600) / 127
= -0.152 / 127
= -0.001197
```

整體平均 delta 趨近零且微負（-0.0012），表明系統在均衡附近振盪而非發散。

**結論：W2-R3 控制迴路在 Lyapunov 意義下局部漸近穩定（locally asymptotically stable），前提是操作頻率分布在合理範圍內。**

[來源: R1_WIENER_calibration.md#Section 2.5]

---

## 12. R3 D3 辯論決議索引

全部 7 個議題均以 5-0 全票通過（WIENER, KNUTH, PASCAL, SUSSMAN, ARCHIMEDES）。

| # | 議題 | 決議 | 投票 |
|---|------|------|:---:|
| Q1 | W2-R3 校準參數鎖定 | **LOCKED** (sigma=0.027389, F1=0.561286, F2=0.042659) | 5-0 |
| Q2 | Scaling factor 方案 | **Plan B4** (alpha scaling, tool_audited only) | 5-0 |
| Q2b | Informational 豁免 | **不豁免**，保持 B-modified path 統一 scaling | 5-0 |
| Q3 | Alpha 精確值 | **0.055** (hardcoded Phase 1, 動態計算列 Phase 2) | 5-0 |
| Q4 | W2-R4 驗證標準 | **V1-V11 雙層標準** (門檻 + 目標) | 5-0 |
| Q5 | CV-5 處置 | **DEFERRED to Plan41** (Master override MR-9; R3 原決議 WAIVED) | 5-0 |
| Q6 | F1/F2 預期值 | **~1.0x** (theoretical, 待 W2-R4 實測) | 5-0 |
| Q7 | Plan40 排程 | **W0 實作 (15min) + W1 驗證 (2-3hrs)** | 5-0 |

[來源: R3_D3_calibration.md#Debate Summary]

---

## 13. W2-R4 驗證標準

Phase 1 鎖定需 W2-R4 驗證通過。採用雙層標準（門檻 MUST + 目標 SHOULD）：

| # | 指標 | 門檻 (MUST) | 目標 (SHOULD) | 來源 |
|---|------|------------|-------------|------|
| V1 | SC-1~SC-6 | 6/6 PASS | 6/6 PASS | ARCHIMEDES |
| V2 | CV-1~CV-4 | 4/4 PASS | 4/4 PASS | ARCHIMEDES |
| V3 | CV-5 | FAIL (expected) | FAIL (expected) | ARCHIMEDES |
| V4 | CV-6 | NOT TRIGGERED | NOT TRIGGERED | ARCHIMEDES |
| V5 | F1/F2 ratio | < 3.0x | < 2.0x | ARCHIMEDES |
| V6 | Clamp rate | < 15% | < 5% | ARCHIMEDES |
| V7 | Category diff (dest vs sm) | dest != sm | \|dest-sm\| > 10% | ARCHIMEDES |
| V8 | Informational delta | > 0 | > 0.0001 | ARCHIMEDES |
| V9 | Mean delta sign per category | 與 W2-R3 同向 | -- | WIENER |
| V10 | Per-category sign correctness | 全部正確 | -- | WIENER |
| V11 | sigma(R4)/sigma(R3) | > 0.5 且 < 2.0 | -- | KNUTH |

**Phase 1 鎖定條件**（PASCAL）：
- 全部門檻通過 + 全部目標達到: P(valid) >= 0.99, Phase 1 **LOCKED**
- 全部門檻通過 + 部分目標未達: P(valid) 0.97-0.99, Phase 1 **CONDITIONAL LOCK**
- 任一門檻未通過: Phase 1 **NOT LOCKED**, 需調查

[來源: R3_D3_calibration.md#Issue 4]

---

## 14. Phase 2 開放項目

1. **Alpha 動態計算**: `maxDelta / max(TOOL_CONFIDENCE_TABLE) * HEADROOM_FACTOR`，消除 TOOL_CONFIDENCE_TABLE 變更時的手動同步風險
2. **Informational TOOL_CONFIDENCE_TABLE 值調整**: 如果 W2-R4 顯示 +0.000055 信號不足
3. **CV-5 重新評估**: 待 arbiter 重構後重新評估 per-category sigma 單調性
4. **F1/F2 長期目標**: < 1.5x
5. **Multi-session 穩定性驗證**: W2-R3 為 single-session 校準，Phase 2 需跨 session/model/prompt 驗證
6. **動態異常偵測閾值**: scaling 後 sigma 降低，anomaly detection 可能需更緊的閾值

[來源: R3_D3_calibration.md#Open Items for Phase 2, R1_KNUTH_PASCAL_statistics.md#Section 10]

---

## 決議索引

| ID | 決議 | 票數 | 內容 |
|----|------|------|------|
| D3-Q1 | W2-R3 參數鎖定 | 5-0 | sigma=0.027389, F1=0.561286, F2=0.042659 |
| D3-Q2 | Plan B4 採納 | 5-0 | alpha=0.055, tool_audited only |
| D3-Q2b | Informational 不豁免 | 5-0 | 保持 B-modified path 統一 scaling |
| D3-Q3 | Alpha=0.055 確認 | 5-0 | Phase 1 hardcoded + 推導公式註釋 |
| D3-Q4 | W2-R4 驗證標準 | 5-0 | V1-V11 雙層標準 |
| D3-Q5 | CV-5 DEFERRED to Plan41 | 5-0 | Master override MR-9; R3 原決議 WAIVED (known limitation: single-source delta) |
| D3-Q6 | F1/F2 預期 1.0x | 5-0 | 待 W2-R4 實測確認 |
| D3-Q7 | Plan40 W0+W1 排程 | 5-0 | 15min 實作 + 2-3hrs 驗證 |

---

## 引用來源

- [來源: R1_WIENER_calibration.md] -- W2 校準三輪軌跡分析、控制迴路完整性、非對稱 clamp 推薦
- [來源: R1_KNUTH_PASCAL_statistics.md] -- SC/CV 獨立統計驗證、F1/F2 分析、方案決策分析
- [來源: R3_D3_calibration.md] -- W2-R3 校準參數鎖定辯論、Plan B4 確認、CV-5/CV-6 處置
- [來源: test_data/cycle03-3/coordinator_report_w2r2r3.md] -- 測試團隊 W2-R2/R3 回報
- [來源: Calibration_Reports/03_Calibration_Methodology.md] -- 校準方法論（C1/C2 框架）

---

*本文件為 Cycle 03-4 Phase 1 校準完整技術紀錄。*
*紀錄時間：2026-04-04*
