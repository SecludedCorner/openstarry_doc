<!-- Layer: 3-Architecture -->
<!-- Status: NEW -->
<!-- Cycle: 03-6, R1/R3 -->
<!-- Author: WIENER (#12), KNUTH (#21), PASCAL (#19) -->
<!-- Source: O5 CV-5 Observe Mode, D2 Debate, D3 Debate -->

# 63. Shadow Counting 控制理論設計

**版本**: 1.0 (Cycle 03-6, 2026-04-09)
**作者**: WIENER (#12) — 控制理論, KNUTH (#21) — 演算法分析, PASCAL (#19) — 機率推理
**來源**: CV-5 observe mode 調查, shadow counting 設計
**決議參照**: D2-Q2a (shadow counting 採用, 7-0), D2-Q2b (per-event per-category, 7-0), D2-Q2c (static authoritative, 7-0), D3-Q3 (Phase 2 baseline LOCKED, 5-0)

---

## 1. 概述

本文件從控制理論角度分析 shadow counting 的設計基礎，包括觀測器-控制器分離、Kalman filter 類比、統計需求、遲滯設計，以及收斂條件。

---

## 2. 觀測器-控制器分離 (Observer-Controller Separation)

### 2.1 問題: 耦合設計的死結

原始設計中，dynamic arbiter 的計數（學習）與決策（控制）耦合在同一執行路徑：

```
[耦合設計 — 已失敗]
EventBus → CalibrationBridge → StateTracker
                                    ↓
                              DynamicArbiter.evaluate()
                                    ↓
                              [只有在 arbiter 做決策時才計數]
                                    ↓
                              Priority deadlock: static 攔截所有決策
                                    ↓
                              永遠無法累積觀察
```

**PASCAL 的形式化證明**: P(observe exit) = 0.0

設 $D$ = dynamic arbiter 被授權做決策的事件，$O$ = dynamic arbiter 累積觀察的事件。

在耦合設計中: $O \subseteq D$ (只有在做決策時才累積觀察)

但 priority 排序使 $P(D) = 0$ (static 永遠不 abstain)

因此 $P(O) \leq P(D) = 0$，故 $P(\text{observe exit}) = 0.0$

### 2.2 解決: 解耦學習與決策

Shadow counting 將觀測器 (observer) 從控制器 (controller) 中分離：

```
[解耦設計 — shadow counting]
EventBus → CalibrationBridge → StateTracker
                                    ↓
                              [每個事件都計數 — 觀測器獨立運行]
                                    ↓
                              DynamicArbiter.evaluate()
                                    ↓
                              [shadow decision — 計算但不應用]
                                    ↓
                              [除非 static abstains]
```

在解耦設計中: $O \not\subseteq D$ (觀察獨立於決策)

$P(O) = P(\text{audit event}) > 0$，故 $P(\text{observe exit}) > 0$

---

## 3. 狀態估計: Kalman Filter 類比

### 3.1 類比結構

Shadow counting 在概念上類似於 Kalman filter 的觀測更新步驟：

| Kalman Filter | Shadow Counting | 對應 |
|--------------|----------------|------|
| **狀態向量** $x_k$ | per-category mean delta | 系統真實狀態 |
| **觀測矩陣** $H$ | CalibrationBridge extraction | 事件→delta 映射 |
| **觀測噪音** $R$ | per-event delta 變異 | 單次觀測的不確定性 |
| **先驗估計** $\hat{x}_{k|k-1}$ | gear=1 cold-start | 初始假設 |
| **後驗估計** $\hat{x}_{k|k}$ | sliding window mean | 更新後的估計 |
| **增益** $K_k$ | 1/n (均等加權) | 新觀測的權重 |

### 3.2 簡化決策

shadow counting 不是完整的 Kalman filter — 它使用簡單的滑動窗口均值而非最優狀態估計。這是故意的工程決策：

| 完整 Kalman | 簡化滑動窗口 | 選擇理由 |
|-------------|-------------|----------|
| 最優增益 (optimal gain) | 均等加權 (equal weight) | 系統噪音特性未知 |
| 需要系統模型 (process model) | 無模型 (model-free) | dynamic arbiter 不假設系統動態 |
| 計算密集 | O(1) per update | 插件內不應有顯著計算負擔 |
| 矩陣運算 | 純算術 | 可維護性優先 |

### 3.3 窗口大小 n=20 的選擇 (KNUTH)

StateTracker 使用固定大小 20 的滑動窗口：

```typescript
if (this.deltas.length > 20) this.deltas.shift();
```

**統計依據**:
- 中央極限定理: n=20 提供均值估計的標準誤差約為 $\sigma / \sqrt{20} \approx 0.224\sigma$
- W2-R5 per-category sigma 範圍: 0.02-0.05
- 均值估計的 95% CI 寬度: $\pm 1.96 \times 0.224\sigma \approx \pm 0.44\sigma \approx \pm 0.01$
- 這足以區分 UP (0.047) 和 DOWN (0.031) 閾值

**記憶體考量**: 20 個 `number` = 160 bytes/category, 4 categories = 640 bytes total。微不足道。

---

## 4. Per-Category vs Per-Cycle 計數 [D2-Q2b]

### 4.1 為何 per-category

| 方案 | 優點 | 缺點 |
|------|------|------|
| **Per-cycle** (所有事件混合) | 簡單，收斂快 | 不同 category 的 delta 分佈根本不同 |
| **Per-category** (每類獨立統計) | 反映真實異質性 | 收斂慢（某些 category 事件少） |

**採用 per-category**: 不同 category 有根本不同的 delta 分佈 [D2-Q2b, 7-0]:

| Category | W2-R5 Mean Delta | 特性 |
|----------|:---:|---|
| informational | +0.000055 | 近零、微正 |
| read_only | +0.0275 | 正向 |
| state_modifying | -0.04125 | 負向 |
| destructive | -0.04675 | 強負向 |

混合這些分佈會產生**無意義的均值** — 正負相消，掩蓋真實模式。

### 4.2 Per-event 計數 [D2-Q2b]

| 方案 | 說明 |
|------|------|
| **Per-cycle** | 每個 W2 cycle 結束時統計一次 |
| **Per-event** | 每個 audit event 到達時立即更新 |

**採用 per-event**: 與 StateTracker 的滑動窗口設計匹配。每個 audit event 觸發一次 `recordDelta()`，窗口即時更新。

---

## 5. 遲滯與停滯時間 (Hysteresis and Dwell Time)

### 5.1 為何需要遲滯

無遲滯的 gear switching:
```
mean > threshold → switch up
mean < threshold → switch down
→ 若 mean 在閾值附近振盪 → gear 持續切換 → 系統不穩定
```

### 5.2 雙閾值遲滯

```
gear 1 → gear 2: mean >= UP  (0.047)
gear 2 → gear 1: mean <= DOWN (0.031)
```

UP > DOWN 建立了一個**不行動帶** (dead band)：

```
        DOWN          UP
   ←─────|────────────|─────→
         0.031       0.047

gear=1: ━━━━━━━━━━━━━━━━━━━▶ (直到 mean >= 0.047)
gear=2: ◀━━━━━━━━━━━━━━━━━━━ (直到 mean <= 0.031)
         [不行動帶: 0.016 寬]
```

### 5.3 控制理論解釋 (WIENER)

遲滯是**Schmitt trigger** 的軟體實現 — 一種已知能防止在雜訊輸入下振盪的電路設計。

| 參數 | 值 | 控制理論意義 |
|------|-----|-------------|
| UP | 0.047 | 上限切換閾值 (upper switching threshold) |
| DOWN | 0.031 | 下限切換閾值 (lower switching threshold) |
| 遲滯寬度 | 0.016 | 不行動帶寬度 (dead band width) |
| MIN_DWELL | 5 | 最小停滯時間 (minimum dwell time) |

### 5.4 停滯時間 (Dwell Time) 的必要性

即使有遲滯，以下場景仍可能導致問題：

```
mean 短暫超過 UP → 立即切換 → mean 回落 → 切換回 → ...
```

MIN_DWELL = 5 要求 mean 在閾值同側**持續 5 個 evaluate() 呼叫**才允許切換。這是**一階 IIR filter** 的離散等價物：

```
dwellCounter >= MIN_DWELL → transition allowed
dwellCounter < MIN_DWELL → transition suppressed, counter incremented
mean crosses back → counter resets to 0
```

### 5.5 閾值選擇的統計基礎 (KNUTH)

| 參數 | 值 | 推導 |
|------|-----|------|
| UP = 0.047 | 接近 DELTA_SCALING_FACTOR (0.055) × TOOL_CONFIDENCE_TABLE[read_only] (0.85) ≈ 0.047 | read_only 等級的完整 delta |
| DOWN = 0.031 | ≈ UP × 0.66 | 經驗法則: 遲滯寬度 ≈ 1/3 × range |
| MIN_DWELL = 5 | 5 evaluate() 呼叫 | 在 n=20 窗口下，5 個新觀測替換 25% 的窗口 — 顯著更新 |

---

## 6. 收斂條件

### 6.1 Shadow Estimator 的收斂

**定理** (非正式): 給定 (1) 審計事件持續產生，(2) per-category 事件分佈穩態，(3) RC-1 修復使 delta 正確傳遞，shadow counting 的 per-category mean 在 $n \to \infty$ 時收斂到真實 category mean。

**證明概要**: 滑動窗口均值是最近 20 個觀測的算術平均。在穩態假設下，這是真實均值的一致估計量 (consistent estimator)。

### 6.2 Observe Mode 退出條件

Dynamic arbiter 在以下條件下退出 observe mode:

```
exists category c: count(c) >= MIN_N (10)
```

**PASCAL 的預測 (post-fix)**:

| Category | 預期事件數 (50 cycles) | 達到 n>=10? |
|----------|:---:|:---:|
| informational | 31 | **是** (充裕) |
| read_only | 16 | **是** |
| state_modifying | 10 | **是** (邊際) |
| destructive | 12 | **是** |

- P(all 4 categories exit within 50 cycles) = 0.65-0.80
- P(all 4 categories exit within 100 cycles) > 0.99
- **瓶頸**: state_modifying (n=10, 恰在閾值)

### 6.3 Phase 2 → Phase 3 轉換條件（未來）

Phase 3 的啟動需要 M4a (shadow agreement rate) 達標。M4a 定義為：

```
M4a = (shadow decisions agreeing with static) / (total shadow decisions)
```

M4a 在 Phase 2 結束前需建立基線。轉換條件（尚未正式定義，待 Plan43 設計時決定）：
- M4a ≥ 95% sustained over N rounds (建議 N ≥ 20)
- 所有 4 categories 獨立達標

---

## 7. Phase 2→3→4 轉換的控制器權威遞增

### 7.1 權威模型

```
Phase 2:  Static = 100%, Dynamic = 0% (observer only)
Phase 3:  Static = state_mod + destructive
          Dynamic = informational + read_only (when static abstains)
Phase 4:  Static = destructive ONLY
          Dynamic = informational + read_only + state_modifying
永久:     Destructive = Static ONLY [Rule #55]
```

### 7.2 控制理論觀點: 增量授權 (Incremental Authority)

此設計等同於**增量控制器上線** (incremental controller commissioning)：

1. **Stage 1 (Phase 2)**: 新控制器在 shadow mode 運行，不影響 plant
2. **Stage 2 (Phase 3)**: 新控制器處理低風險 set-points，主控制器保留高風險
3. **Stage 3 (Phase 4)**: 新控制器擴展至中風險 set-points
4. **永久**: 高風險 set-points 永遠由主控制器處理

這是工業控制系統中**controller commissioning** 的標準實踐。在每個 stage，operator 驗證新控制器的表現滿足品質標準 (M4a) 後才授權下一階段。

---

## 8. W2-R5 五輪軌跡的控制理論解讀

### 8.1 Sigma 軌跡分析

```
R1: 0.002  (盲點，人工壓低)
R2: 0.023  (盲點修復後的暫態)
R3: 0.027  (暫態峰值)
R4: 0.020  (DELTA_SCALING_FACTOR 校正)
R5: 0.022  (Phase 2 穩態)
```

R4→R5 穩定性驗證 [D3-Q1]:

| 指標 | R4→R5 比率 | 閾值 | 結果 |
|------|:---:|:---:|:---:|
| sigma 比率 | 1.084 | [0.5, 2.0] | **PASS** |
| F1/F2 | 1.0x→0.99x | ~1.0x | **PASS** |
| clamp | 0%→0% | <5% | **PASS** |
| n_eff/N | 100%→100% | >80% | **PASS** |

### 8.2 Dynamic Arbiter 引入的影響

Plan41 引入 dynamic arbiter (R5)，但對控制迴路的影響為**零擾動** — 因為 dynamic arbiter 處於 observe mode，不產生任何決策。

這驗證了 shadow counting 的核心設計目標：**學習不影響控制** (learning does not perturb control)。

### 8.3 Phase 2 Baseline LOCKED [D3-Q3]

| 參數 | 鎖定值 |
|------|--------|
| sigma | **0.02216** |
| F1/F2 | **0.99x** |
| clamp | **0%** |
| neg freq threshold | **25%** (兩輪平滑規則) |

---

## 9. 三重目的 (WIENER)

| 目的 | 時間範圍 | 度量 |
|------|----------|------|
| **即時**: Dynamic arbiter 退出 observe mode，證明資料管線正常 | Plan42 | M2 (exit event), M2a (counts) |
| **診斷**: Shadow agreement rate 可量測 | Plan43 (M4a) | M4a 基線建立 |
| **未來**: 基於 M4a 驗證的漸進權威轉移 | Plan44+ | M4a 持續同意率 → static abstain 路徑 |

Dynamic arbiter 從「壞掉」轉為「學習中」。權威轉移是獨立的、未來的決策，以 M4a 資料為門檻。

---

## 10. 設計原則摘要

| 原則 | 實現 | 控制理論基礎 |
|------|------|-------------|
| 觀測與控制分離 | Shadow counting 解耦 | Observer-controller separation |
| 有界記憶 | n=20 sliding window | Finite-horizon estimation |
| 異質處理 | Per-category statistics | Multi-mode estimation |
| 防振盪 | Hysteresis + dwell time | Schmitt trigger + IIR filter |
| 增量授權 | Phased authority transfer | Controller commissioning |
| 學習不干擾 | Shadow decisions discarded | Open-loop observer |

---

*Architecture Documentation #63 — Shadow Counting Control Theory Design*
*Cycle 03-6 Research Addition, 2026-04-09*
*WIENER (#12), KNUTH (#21), PASCAL (#19)*
*Decision References: D2-Q2a, D2-Q2b, D2-Q2c, D3-Q1, D3-Q3*
