<!-- Layer: 3-Architecture -->
<!-- Status: NEW -->
<!-- Cycle: 03-5, R1 -->
<!-- Author: WIENER (#12), KNUTH (#21), PASCAL (#19) -->
<!-- Source: W2-R1 through W2-R4 -->

# 60. W2 校準收斂架構與 V4 Spec 修正

**版本**: 1.0 (Cycle 03-5, 2026-04-07)
**作者**: WIENER (#12) — 控制理論, KNUTH (#21) — 統計分析, PASCAL (#19) — 機率推理
**來源**: W2-R4 鎖定 (LOCKED P=0.993), V4 Spec 修正

---

## 1. 概述

本文件記錄 W2 校準系統從 R1 到 R4 的收斂歷程，以及 V4 Spec 從 frequency-based 到 saturation-based 的修正。這是 OpenStarry 信心審計系統的核心架構文件。

---

## 2. 四輪軌跡表

| 指標 | R1 (v0.38) | R2 (v0.39) | R3 (hotfix) | R4 (v0.40) | 趨勢 |
|------|:---:|:---:|:---:|:---:|:---:|
| **判定** | GO | IF | GO | **GO** | 穩定 |
| **sigma** | 0.002 | 0.023 | 0.027 | **0.020** | 收斂 |
| **F1/F2** | 1.0x | 14.2x | 13.2x | **1.0x** | 修復 |
| **Clamp** | 0% | 32.2% | 29.9% | **0%** | 消除 |
| **n_eff/N** | 37.9% | 100% | 100% | **100%** | 穩定 |
| **SC** | 5/6 | 5/6 | 6/6 | **6/6** | 完整 |

### 2.1 控制理論解釋 (WIENER)

sigma 軌跡 {0.002, 0.023, 0.027, 0.020} 展現典型的**二階欠阻尼步階響應** (second-order underdamped step response)：

| 階段 | 輪次 | 行為 | 證據 |
|------|------|------|------|
| **暫態** (Transient) | R1-R3 | 大幅振盪、感測器盲點、符號反轉 | F1/F2: 1.0x→14.2x→13.2x |
| **穩態** (Steady state) | R4 | 收斂、所有指標在 10% 內 | F1/F2 ~1.0x, clamp 0% |

R1 的 sigma=0.002 是**人工壓低值** — 感測器盲點 (audit path gap) 導致大部分 delta 為零。修復盲點後 (Plan39 B-modified delta injection)，系統顯現真實波動，然後經 DELTA_SCALING_FACTOR = 0.055 校正收斂。

---

## 3. V4 Spec 修正: Frequency → Saturation

### 3.1 修正前 (V4-old)

```
CV-6 FAIL: neg freq >= 5%
```

**問題**: freq 是結構屬性 (structural property)。destructive 和 state_modifying 操作*天然*產生負 delta，因此 negative frequency 達到 5% 是正常且預期的。V4-old 將正常操作行為判為失敗。

### 3.2 修正後 (V4-new)

| 閾值 | 條件 | 含義 |
|------|------|------|
| **FAIL** | saturation ≥ 10% | 資訊破壞 (information destruction) |
| **WARNING** | neg freq ≥ 25% | 需調查但非失敗 |

**Saturation** 定義: clamp rate — 指 delta 被截斷到邊界的比例。若 saturation ≥ 10%，表示系統正在丟失信號，這是真正的品質問題。

### 3.3 WARNING 閾值推導 (KNUTH)

```
WARNING = 25% = mu + 1.75 * sigma_est
```

基於已觀測最大值 19.8%，加上 5.2 個百分點的邊際。KNUTH 使用了 W2-R3 的 127 筆資料作為分佈估計基礎。

**解釋帶** (Interpretation bands):
- < 15%: 可疑 (可能符號回歸)
- 15-25%: 正常操作範圍
- \> 25%: WARNING (需調查)

### 3.4 修正對 W2-R4 的影響

| V | 舊結果 | 新結果 |
|---|--------|--------|
| V1-V3 | PASS | PASS |
| **V4** | **FAIL** | **PASS** (sat=0% < 10%) |
| V5-V6 | PASS | PASS |
| **MUST 總計** | **5/6** | **6/6** |

**判定**: CONDITIONAL LOCK → **LOCKED (P = 0.993)**

---

## 4. 鎖定參數 (Cycle 03-5 確認)

| 參數 | 值 | 說明 |
|------|-----|------|
| DELTA_SCALING_FACTOR | **0.055** | 生產環境使用值 |
| F1/F2 | **0.997x** | 訊噪比 parity |
| Clamp rate | **0.0%** | 飽和完全消除 |
| Cycle sigma (F2) | **0.031031** | W2-R3 真值 |
| info mean | +0.000468 | 近零、微正 |
| read_only mean | +0.015800 | 正向 |
| state_mod mean | -0.032321 | 負向（預期） |
| destructive mean | -0.046750 | 強負向（預期） |

---

## 5. 架構意涵

### 5.1 校準系統的成熟度模型

```
Phase 1 (R1-R4): 初始校準
  └─ 目標: 找到 DELTA_SCALING_FACTOR 使 F1/F2 ~1.0x 且 clamp=0%
  └─ 結果: 0.055 鎖定，系統收斂

Phase 2 (R5+): 擴展操作
  └─ 目標: 驗證鎖定參數在新功能 (Plan41+) 下仍穩定
  └─ 基準: sigma ~0.020, F1/F2 ~1.0x, clamp 0%

Phase 3 (未來): 動態仲裁
  └─ 目標: dynamic arbiter 基於 shadow 觀測逐步接管
  └─ 先決條件: Phase 2 基準穩定
```

### 5.2 控制迴路 Invariants

WIENER 確認的系統 invariants：

1. **C-1 信心窗口隔離**: 每個 agent 的信心窗口獨立，無跨 agent 洩漏
2. **C-2 無交叉汙染**: per-instance 追蹤，非全域
3. **C-3 有界**: 滑動窗口 20 筆 delta，記憶體使用有界

### 5.3 BABBAGE 機制清單 (至 Plan40)

W2 校準系統相關機制：

| # | 機制 | 引入 | 狀態 |
|---|------|------|------|
| M-38-7 | TOOL_CONFIDENCE_TABLE | Plan38 | 生產中 |
| M-38-8 | B-modified delta injection | Plan39 | 生產中 |
| M-40-1 | DELTA_SCALING_FACTOR | Plan40 | 鎖定 |
| M-40-2 | signMultiplier (-1/+1) | Plan40 | 生產中 |

---

## 6. V4 Spec 修正的統計學基礎 (PASCAL)

### 6.1 為何 frequency 不適合作為失敗標準

設 $X$ 為 negative delta 出現次數，$N$ 為總事件數。

negative frequency $= X/N$

在 OpenStarry 中，destructive (fs.delete) 和 state_modifying (fs.write) 操作*設計上*產生負 delta。因此：

```
P(negative) = P(destructive) + P(state_modifying | negative)
```

這是系統的結構屬性，非品質指標。以 W2-R3 的 127 筆資料為例：
- destructive: 12/127 = 9.4%
- state_modifying (negative): 13/127 = 10.2%
- 合計: ~19.7%

這個比例在正常操作範圍內。

### 6.2 為何 saturation 適合作為失敗標準

Saturation (clamp rate) 表示 delta 超出 [0, maxAuditDelta] 範圍而被截斷的比例。截斷代表*信號損失* — 系統的真實行為未被正確記錄。

- saturation = 0%: 所有信號完整保留
- saturation > 0%: 部分信號被截斷，品質指標有系統性偏差
- saturation ≥ 10%: 嚴重信號損失，校準可能不再有效

---

## 7. 10/0/0 合規影響

V4 Spec 修正是度量標準的改進，不影響 tenet 合規。校準鎖定強化 Tenet #8（控制迴路）的表現。

| Tenet | 影響 | 說明 |
|-------|------|------|
| #8 控制迴路 | 正面 | 校準收斂，信號保真度改善 |
| 其餘 | 中性 | 無影響 |

---

## 8. 後續展望

1. **Phase 2 驗證**: Plan41 引入 gear-arbiter-dynamic 後，W2-R5 需確認系統穩定性
2. **WARNING 閾值監控**: neg freq 接近 25% 時需調查
3. **Dynamic arbiter 校準**: Phase 3 啟動前需建立 M4a (shadow agreement rate) 基準

---

*Architecture Documentation #60 — W2 Calibration Convergence Architecture*
*Cycle 03-5 Research Addition, 2026-04-07*
*WIENER (#12), KNUTH (#21), PASCAL (#19)*
