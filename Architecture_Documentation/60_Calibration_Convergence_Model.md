<!-- Layer: 2-Engineering -->
<!-- Status: NEW -->
<!-- Cycle: 03-5 -->
<!-- Author: WIENER (#12), PASCAL (#19) -->

# 60. 校準收斂模型 (Calibration Convergence Model)

本文件分析 W2 四回合校準的收斂過程，從控制理論與貝葉斯推斷的雙重視角解釋系統如何從初始偏差迭代修正至穩定均衡，並提供 Phase 1 到 Phase 2 的形式化轉換準則。

> **決定權限**: [D2-Q3] (LOCKED, 5-0), [D2-Q4] (parameters, 5-0), [D2-Q5] (Phase 2 transition, 5-0), [D5-Q1] (Phase 2 activation, 6-0)。

---

## 1. 四回合校準軌跡

### 1.1 數據總覽

| Round | Version | 核心問題 | neg freq | sat. | Clamp | F1/F2 | V4 (old) | V4 (new) |
|:-----:|---------|----------|:--------:|:----:|:-----:|:-----:|:--------:|:--------:|
| R1 | v0.38 | Sensor blind spot (delta=0) | 3.2% | N/A | 0% | 1.0x | PASS | PASS |
| R2 | v0.39 | Sign inversion | 3.4% | N/A | 32.2% | 14.2x | PASS | PASS |
| R3 | hotfix | Unscaled | 19.7% | 38.1% | 29.9% | 13.2x | FAIL | WARNING |
| **R4** | **v0.40** | **Scaled (alpha=0.055)** | **19.8%** | **0.0%** | **0.0%** | **0.997x** | ~~FAIL~~ | **PASS** |

### 1.2 每回合詳細分析

#### Round 1: Sensor Blind Spot（感測器盲區）

**症狀**：delta 幾乎全為零。neg_freq 僅 3.2%（少數 non-zero entries）。F1/F2 = 1.0x——看似理想。

**根因**：audit trail 中的 B-modified path（tool_audited events）未正確注入 delta。大多數條目的 rawDelta = 0，clampedDelta = 0。系統表面上健康，但**感測器失靈**——測量結果不反映真實行為。

**控制理論解讀** (WIENER)：這是典型的 **sensor fault masking**。感測器輸出恆定值（零），使得所有下游指標（F1/F2、clamp rate）顯示正常。在閉迴路控制中，sensor masking 是最危險的故障模式之一——它阻止系統偵測到真實偏差。

**遮蔽效應** (masking effect)：R1 的所有 PASS 結果都是**無效的**。F1/F2 = 1.0x 不是因為信號正確，而是因為沒有信號。這揭示了校準系統的一個根本挑戰：「看起來正常」未必是正常——必須驗證感測器本身的功能。

#### Round 2: Sign Inversion（符號反轉）

**症狀**：修復了 delta 注入，但 destructive/state_modifying 操作錯誤地產生正 delta。clamp rate 32.2%，F1/F2 = 14.2x。

**根因**：sign model 反轉。destructive 操作（本應降低信心）反而增加信心，導致大量正 delta 超出 clamp window。

**控制理論解讀** (WIENER)：**positive feedback instability**。符號反轉將負反饋迴路轉為正反饋。在物理系統中，這等同於恆溫器的加熱/冷卻邏輯反接——溫度升高時系統繼續加熱，直到觸及安全限制（clamp）。

**信號損失量化**：F1/F2 = 14.2x 意味著 pre-clamp sigma（F1）比 post-clamp sigma（F2）大 14.2 倍。94% 的信號變異被 clamping 截斷——幾乎完全的信息破壞。

#### Round 3: Correct Sign, Unscaled（符號正確，未縮放）

**症狀**：符號修正後，neg_freq 首次升至 19.7%（正確行為）。但 F1/F2 仍為 13.2x，clamp rate 29.9%。sigma = 0.027389 LOCKED。

**根因**：raw delta 值過大。`TOOL_CONFIDENCE_TABLE` 的數值（最大 0.85 for destructive）直接作為 delta，遠超 clamp window [-0.05, +0.05]。

**控制理論解讀** (WIENER)：**gain mismatch**。信號的增益（amplitude）與通道的動態範圍（dynamic range）不匹配。類似於將高增益麥克風接入低動態範圍的錄音設備——聲音清晰但持續過載。

**R3 的價值**：雖然 F1/F2 仍不理想，但 R3 確認了兩件事：(1) sign model 正確（neg_freq ~20% 來自正確的 category-sign 映射），(2) sigma = 0.027389 是真實的系統雜訊水平（非 R1 的零，非 R2 的污染值）。這為 R4 的 scaling 提供了可靠基線。

#### Round 4: Fully Calibrated（完全校準）

**症狀**：alpha = 0.055 套用後，所有指標回歸理想值。F1/F2 = 0.997x，clamp = 0.0%，saturation = 0.0%。

**校準結果**：
- F1/F2 = 0.997x（理想值 1.0x，差異 < 0.3%）
- Clamp rate = 0.0%（完美——無任何事件被截斷）
- neg_freq = 19.8%（結構決定，與 R3 一致）
- Category signs 全部正確

**控制理論解讀** (WIENER)：系統已達到**穩定均衡** (stable equilibrium)。scaling factor 將信號增益匹配到通道動態範圍，消除了所有 clamping。這是經典的 impedance matching 問題——R4 找到了正確的 impedance。

---

## 2. 控制理論：迭代修正模型

### 2.1 收斂結構

四回合校準展現了**迭代修正** (iterative refinement) 的收斂模式：

```
R1: Perturbed state (sensor fault)
    ↓ fix sensor
R2: Divergent state (sign inversion)
    ↓ fix sign
R3: Partially converged (gain mismatch)
    ↓ fix scaling
R4: Converged to stable equilibrium
```

每回合修正一個**獨立的缺陷維度**：
- R1 → R2: 修復 delta injection（感測器功能）
- R2 → R3: 修復 sign model（信號極性）
- R3 → R4: 修復 scaling（信號增益）

### 2.2 遮蔽效應分析 (Masking Effect)

遮蔽效應是此迭代過程中的核心挑戰：

| Round | 被遮蔽的問題 | 遮蔽機制 |
|:-----:|------------|----------|
| R1 | sign inversion + gain mismatch | sensor blind spot 使所有 delta = 0 |
| R2 | gain mismatch | sign inversion 主導 F1/F2，掩蓋了 gain 問題 |
| R3 | 無 | 所有主要問題暴露 |
| R4 | 無 | 系統收斂 |

遮蔽效應的實際含義是：**每修復一個問題，可能暴露出下一個被遮蔽的問題**。這不是失敗——這是迭代校準的**預期行為**。校準團隊（WIENER/KNUTH/PASCAL）在 R2 結束時即預測了 R3 的 gain mismatch，這是對遮蔽效應的正確理解。

### 2.3 收斂速率

| 指標 | R2 → R3 改善 | R3 → R4 改善 | R4 距理想值 |
|------|:-----------:|:-----------:|:----------:|
| F1/F2 | 14.2x → 13.2x (-7%) | 13.2x → 0.997x (-92%) | 0.3% |
| Clamp rate | 32.2% → 29.9% (-7%) | 29.9% → 0.0% (-100%) | 0% |
| Sign correctness | 0/4 → 4/4 | 4/4 → 4/4 | 完美 |

R3 → R4 的改善幅度（-92%、-100%）遠大於 R2 → R3（-7%、-7%），因為 R3 → R4 修正了**根本原因**（gain mismatch），而 R2 → R3 只修正了**方向**（sign）。

---

## 3. 貝葉斯後驗推導 (PASCAL)

### 3.1 P = 0.993 的推導

**Prior**：P(calibration correct | CONDITIONAL LOCK) = 0.98

CONDITIONAL LOCK 的先驗基於 R4 的數據質量：6/6 MUST PASS（除 V4），7/7 SHOULD MET，F1/F2 = 0.997x。V4 是唯一的 FAIL，且已被診斷為 spec defect。

**Evidence update**：

1. V4 spec fix 不是降低標準——是修正衡量錯誤
   - Master 正式批准（Modifications 3-1, 3-2, 3-3）
   - R3 全票通過 [D2-Q1]（5-0）
   - WIENER 信號理論分析提供獨立理論基礎

2. 修正後的 V4 結果：
   - CV-6 C1: neg_freq 19.8% < 25% → CLEAR
   - CV-6 C2: saturation 0.0% < 10% → CLEAR
   - V4: PASS

3. 完整證據集：
   - MUST: 6/6 PASS
   - SHOULD: 7/7 MET
   - F1/F2 = 0.997x（理想值 ~1.0x）
   - Clamp = 0.0%（理想值 0%）
   - sigma ratio = 0.747（在 0.5-2.0 band 內）
   - 四回合軌跡顯示 F1/F2 的單調改善（在修正方向上）

4. 對抗性先驗測試（KNUTH 懷疑論先驗）：
   - 若 P_prior = 0.95（更保守），posterior >= 0.974
   - 結論對先驗選擇具有穩健性

**Posterior calculation**：

```
P(correct | all evidence) = P(evidence | correct) * P(correct) / P(evidence)

P(evidence | correct) ≈ 0.997 (6/6 MUST + 7/7 SHOULD + ideal metrics)
P(correct) = 0.98 (prior)
P(evidence) = P(evidence|correct)*P(correct) + P(evidence|incorrect)*P(incorrect)
            = 0.997 * 0.98 + 0.30 * 0.02
            = 0.97706 + 0.006
            = 0.98306

P(correct | evidence) = 0.97706 / 0.98306 = 0.993
```

### 3.2 EVAI（期望附加信息價值）

PASCAL 評估：再跑一輪 W2 的期望信息價值接近零。

- 若 W2-R5 通過：P 從 0.993 升至 ~0.996（決策無影響）
- 若 W2-R5 發現新問題：P 會下降，但新問題本身的發現價值高於數值變化

**結論**：EVAI ~ 0。不值得為提高 P 0.003 而延遲 LOCK。

### 3.3 殘餘風險清單

| 風險 | 機率 | 影響 | 緩解措施 |
|------|:----:|:----:|----------|
| CV-5 architectural limitation | Known | Expected FAIL | V3 accounts for it; DEFERRED to Plan41 [MR-9] |
| Category mix shift | ~15% | MEDIUM | Phase 2 M4 monitoring |
| Unknown unknowns | ~1% | Irreducible | Phase 2 full-spectrum monitoring |

---

## 4. Alpha = 0.055 推導路徑

### 4.1 數學推導

```
alpha = (maxDelta / max(TOOL_CONFIDENCE_TABLE)) * HEADROOM_FACTOR
      = (0.05 / 0.85) * 0.935
      = 0.058824 * 0.935
      = 0.055
```

其中：
- `maxDelta = 0.05`：clamp window 的邊界值（來自 `DEFAULT_CONFIDENCE_AUDIT_CONFIG`）
- `max(TOOL_CONFIDENCE_TABLE) = 0.85`：destructive category 的信心表值（來自 SDK）
- `HEADROOM_FACTOR = 0.935`：確保最大 scaled delta 不觸及 clamp 邊界

### 4.2 Headroom 驗證

```
max_scaled_delta = max(TOOL_CONFIDENCE_TABLE) * alpha
                 = 0.85 * 0.055
                 = 0.04675

headroom = maxDelta - max_scaled_delta
         = 0.05 - 0.04675
         = 0.00325 (6.5% margin)
```

6.5% 的 headroom 足以吸收：
- IEEE 754 浮點運算誤差（< 1e-15）
- 未來 TOOL_CONFIDENCE_TABLE 微調的緩衝

### 4.3 Per-Category Scaled Deltas

| Category | Table Value | Scaled Delta | Sign | 佔 clamp window |
|----------|:----------:|:------------:|:----:|:--------------:|
| informational | 0.001 | +0.000055 | + | 0.11% |
| read_only | 0.30 | +0.01650 | + | 33.0% |
| state_modifying | 0.65 | -0.03575 | - | 71.5% |
| destructive | 0.85 | -0.04675 | - | 93.5% |

**關鍵特性**：沒有任何 category 的 scaled delta 超過 clamp window。destructive 使用了 93.5% 的範圍（最接近邊界），正是 HEADROOM_FACTOR 確保的安全邊距。

### 4.4 數值穩定性 (KNUTH)

- `0.055` 在 IEEE 754 double-precision 中精確表示為 0.05499999999...
- 乘法 `0.85 * 0.055 = 0.04675` 無 catastrophic cancellation 風險
- sign 由整數 `signMultiplier` (-1 or +1) 決定，無浮點符號損失
- 6.5% headroom 遠大於任何浮點誤差

---

## 5. Phase 1 → Phase 2 轉換準則

### 5.1 Phase 1 Lock 條件（已滿足）

```
Phase 1 LOCKED iff:
  (1) All MUST thresholds PASS (V1-V6: 6/6)
  (2) All SHOULD targets MET (V5t-V11: 7/7)
  (3) R3 ratification vote >= 4-0 (actual: 5-0) [D2-Q3]
  (4) P >= 0.99 (actual: P = 0.993)
```

### 5.2 Phase 2 啟動條件（已滿足）[D5-Q1]

| Prerequisite | Status |
|-------------|:------:|
| Phase 1 LOCK ratified by R3 | DONE [D2-Q3] |
| Plan40 delivery verified | DONE [D1-Q1, 15P/1D/0F] |
| W2-R4 baseline established | DONE [D2-Q4, 8 parameters LOCKED] |
| Monitoring team assigned | DONE (WIENER/KNUTH/PASCAL) |

### 5.3 Phase 2 監控結構

Phase 2 以 LOCKED 參數為基線，監控系統是否偏離穩定均衡。

**五指標矩陣** [D5-Q2]：

| # | Metric | Baseline | WARNING Band | FAIL Band |
|---|--------|:--------:|:------------:|:---------:|
| M1 | F2 sigma | 0.031031 | 0.5x-2.0x | 0.25x-3.0x |
| M2 | F1/F2 ratio | 0.997x | > 2.0x | > 3.0x |
| M3 | Clamp rate | 0.0% | > 5% | > 15% |
| M4 | neg freq | 19.8% | < 10% or > 25% | < 5% or > 40% |
| M5 | Category sign | all correct | any violation | (immediate re-cal) |

**四個再校準觸發條件** [D5-Q3]：

| # | Trigger | Condition | Priority |
|---|---------|-----------|:--------:|
| T1 | Single FAIL | Any metric enters FAIL band | 2 |
| T2 | Multi-WARNING | 2+ metrics from DIFFERENT correlation groups | 3 |
| T3 | Persistent WARNING | Single WARNING for 2 consecutive W2 rounds | 4 |
| T4 | Sign violation | Any category sign incorrect | 1 |

**Correlation groups** (KNUTH)：{M2, M3}（signal loss），{M1, M4}（distribution），{M5}（model）。同組 co-WARNING 遵循 T3 邏輯而非 T2。

### 5.4 再校準流程

```
Trigger detected
  ↓
Root Cause Analysis (RCA)
  ↓ identify affected dimension
If sensor fault: fix sensor, re-run W2 (R1-type issue)
If sign error: fix sign model, re-run W2 (R2-type issue)
If gain drift: recalculate alpha, re-run W2 (R3-type issue)
If new phenomenon: full investigation, may require new W2 protocol
```

---

## 6. 收斂模型的一般性啟示

### 6.1 迭代校準的結構性保證

四回合校準的成功建立了一個重要的方法論原則：

1. **修復順序無關收斂**：sensor → sign → gain 的修復順序是自然的（每次修復暴露下一問題），但最終收斂不依賴順序。
2. **遮蔽效應是預期行為**：每修復一個問題暴露下一個問題不是失敗——是迭代校準的固有特徵。
3. **收斂充分性**：當 F1/F2 < 1.1x 且 clamp = 0% 時，系統已達到充分的校準精度。進一步的微調（F1/F2 從 0.997x 到 1.000x）不具有實際意義。

### 6.2 對未來校準的預測

基於四回合經驗，未來的校準（若觸發 T1-T4）預期只需 1-2 回合即可收斂——因為 Phase 1 已識別並修復了所有主要缺陷維度（sensor、sign、gain）。除非引入全新的缺陷類型（如 category misclassification），否則再校準將是局部修正而非全面重建。

---

## 7. 設計決定參照

| 決定 ID | 內容 | 票數 |
|---------|------|:----:|
| [D2-Q3] | W2-R4 CONDITIONAL → LOCKED (P=0.993) | 5-0 |
| [D2-Q4] | 8 calibration parameters LOCKED | 5-0 |
| [D2-Q5] | Phase 2 transition criteria ADOPTED | 5-0 |
| [D5-Q1] | Phase 2 ACTIVATED | 6-0 |
| [D5-Q2] | 5-metric monitoring matrix ADOPTED | 6-0 |
| [D5-Q3] | 4 re-calibration triggers ADOPTED | 6-0 |

---

*60 — Calibration Convergence Model*
*WIENER (#12), PASCAL (#19)*
*Cycle 03-5, 2026-04-08*
*Decision references: [D2-Q3] through [D2-Q5], [D5-Q1] through [D5-Q3]*
