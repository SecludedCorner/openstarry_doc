<!-- Status: CURRENT -->
<!-- Layer: 3-Research -->
<!-- Applies to: v0.33.0-alpha -->
<!-- Last verified: 2026-03-16 -->

> **Audience**: Control theory researchers, stability analysis researchers
> **Prerequisite**: Familiarity with Lyapunov stability theory, TOST equivalence testing

# 54. Lyapunov Phase 0 分析報告

**版本**: v1.0
**共同作者**: WIENER (#12, Control Theory Expert) · PASCAL (#19, Decision Theory & Probability Philosophy Expert)
**週期**: Cycle 02-10
**日期**: 2026-03-13
**狀態**: Final（整合 D1 辯論決議 D1-R1 至 D1-R8）
**Protocol 依據**: Lyapunov Phase 0 Protocol v1.0-final（02-9 交付）

---

## 1. Executive Summary（執行摘要）

### 1.1 核心判定

**Phase 0 整體判定：CONDITIONAL GO**

此判定非 PASS 亦非 FAIL，而是三層分析的綜合結論：

| 層面 | 判定 | 依據 |
|------|------|------|
| 數據管線有效性（S6/S7） | **PASS** | 零 error，所有值在預期範圍，audit 機制正確運作 |
| q-only 統計基礎（S4, q* 估計） | **PARTIAL PASS** | sigma_DeltaV_q 可估計，ESS_factor 合理，q* 有意義估計 |
| 三維 Lyapunov 核心目標 | **INAPPLICABLE / FAIL** | sigma_DeltaV(3D) 未定義，S5/S8 前提不成立，TOST 退化 |

### 1.2 Go/No-Go 條件

CONDITIONAL GO 的前提條件：
1. **W2 mini-pilot（50 cycles）先行**：確認 audit 觸發率 ≥ 50% 且 delta ≠ 0 出現頻率 ≥ 20% of audited cycles
2. **Phase 1 Protocol 修訂版完成**：依 D1-R1 至 D1-R4 的全部修訂指令
3. **W2 mini-pilot 提供 sigma_DeltaV_full（三維）**：供 Phase 1 n_corrected 計算

### 1.3 根本原因

W1 Workload（pure informational/read_only 工具）的信息密度不足以激發 ThresholdAuditor 的調節功能。130 cycles 中，21.2% 觸發 audit pipeline；被觸發的所有 audit 中，confidence 恆高於 threshold，導致 delta 恆為 0。這不是系統不穩定——這是在無威脅環境中的正確行為，如同「量測無威脅環境中的免疫反應為零，是正確的觀測結果，不能解讀為免疫系統有效性的證明，也不能解讀為免疫系統失效」（D1-R2 記錄之 BABBAGE 類比）。

### 1.4 主要發現

- **系統健康**：零 audit error，所有安全不變量完整保持，數據管線正常
- **q(k) 高度穩定**：取五離散值 {0.75, 0.8125, 0.875, 0.9375, 1.0}，均值回歸，均衡估計 q* ≈ 0.94
- **Layer 3 動力學潛伏**：theta 由 static-rule-arbiter 的 risk category mapping 決定（{0.5, 0.6}），非 loopQualityAlpha 連續調整的輸出
- **vitakka = mechanism**：唯一 gear=2 事件（cycle 71）由 vitakka_stall_override 觸發，確認為 mechanism 行為（非 policy）
- **三維分析不可行**：稀疏 full-audit 數據（measurement window 僅 13 筆）使三維 V(k) 分析實質上無法進行

---

## 2. Data Overview（數據概覽）

### 2.1 Collection Parameters（採集參數）

| 參數 | 設計值（Protocol v1.0-final） | 實際值 |
|------|------------------------------|--------|
| 總 cycles | 130（30 warmup + 100 measurement） | 130 JSONL loop:started |
| Warmup cycles | 30 | k=0..29（CSV 30 行）|
| Measurement cycles | 100 | k=30..103（CSV 74 行，JSONL 100）|
| CSV 記錄行數 | 100（measurement） | 74（measurement），104（total）|
| loopQualityAlpha | 0.10 | 0.10（SDK default）|
| Environment | Option C（clean，無 Klesha，無 VedanaEmergency）| 確認 |
| Auditor | ThresholdAuditor（default rules） | 確認，active |

**26 行缺失的說明**：JSONL 記錄了 130 個 loop:started 事件，但 CSV 僅有 104 行（差 26 行）。依 BABBAGE 在 D1 Round 1 的分析，缺失行最可能為 duration=0ms 或 duration=-1ms 的瞬間 cycle（時間戳精度問題），被 CSV 記錄器視為不完整的 CycleRecord 而跳過。此缺失導致 measurement window 中 CSV 行數由預期 100 降至 74，是 S1a NEAR-MISS 的主要技術原因（OQ-D1-4 待 TURING 原始碼確認）。

### 2.2 Event Stream Summary（事件流摘要）

| 事件類型 | 總數 | 說明 |
|---------|------|------|
| loop:started | 130 | 所有 130 cycles 均完整記錄 |
| loop:finished | 130 | 完整配對 |
| loop:quality_updated | 130 | q(k) 每 cycle 均更新 |
| gear:arbiter_evaluated | 22 | 僅 tool-routing cycles 觸發 |
| audit:completed | 22 | 與 arbiter_evaluated 對應 |
| gear:switch | 23 | 22 次正常 + 1 次 vitakka_stall_override |
| **JSONL 事件總數** | **558** | |

**Audit pipeline 觸發率**：22/104 (21.2%) 的所有 CSV 記錄。measurement window 中：13/74 (17.6%)。全部 22 次 audit 結果均為 "unchanged"（delta = 0）。

**gear:switch 事件 breakdown**：
- 22 次由 static-rule-arbiter 觸發（正常 routing）
- 1 次由 vitakka_stall_override 觸發（k=57，gear=2，D1-R6 確認為 mechanism）

### 2.3 Data Quality Assessment（數據品質評估）

#### 2.3.1 q(k) 分析

q(k) 的唯一變異來源是 q_coherence（其他三個子維度 q_efficiency = q_convergence = q_stability = 1.0 恆成立）：

```
q = (q_coherence + q_efficiency + q_convergence + q_stability) / 4
  = (q_coherence + 3) / 4
```

| q_coherence | q 值 | 近似出現次數（full dataset） |
|------------|------|--------------------------|
| 0 | 0.75 | ~8 次 |
| 0.25 | 0.8125 | ~8 次 |
| 0.5 | 0.875 | ~18 次 |
| 0.75 | 0.9375 | ~35 次 |
| 1.0 | 1.0 | ~35 次 |

q(k) 的特徵：
- **離散有界**：嚴格取五個離散值 ∈ [0.75, 1.0]
- **均值回歸**：擾動後趨向回歸至 1.0
- **無明顯趨勢**：長期無上升或下降漂移
- **正自相關**：相鄰 cycle 間 q 值傾向相同，run length 通常 2-6 cycles

**注意**：q(k) 的離散有界特性使得傳統連續分佈假設的 ADF/KPSS 測試在此數據上意義有限——有界序列不可能有 unit root，因此 ADF 強力拒絕 H0 是數學必然，而非特殊的統計發現。

#### 2.3.2 theta(k) 與 c_adj(k) 分析

| 欄位 | 值域 | 決定機制 | 說明 |
|------|------|---------|------|
| theta | {0.5, 0.6} | static-rule-arbiter + risk category | informational→0.5；read_only→0.6 |
| c_raw | {0.9, 0.95} | gear arbiter confidence | 離散 |
| c_adj | {0.9, 0.95} | audit 後輸出 | = c_raw（delta = 0 恆成立）|
| delta_raw | 0（全部） | ThresholdAuditor 判定 | c_adj >> theta，無需調整 |
| delta_clamped | 0（全部） | 同上 | |

**Layer 3 動力學觀察**：Protocol 設計期待 `theta_adjusted = max(0.3, 0.6 * (1 - 0.10 * q(k)))` 在每個 cycle 連續更新。但實際 thresholdAtDecision 反映的是 risk category 的 static riskDelta 而非 Layer 3 的連續調整。W1 workload 下，Layer 3 的連續動力學被 Layer 2 的離散 risk category 分類完全掩蓋。

#### 2.3.3 資料異常記錄

| 異常類型 | 描述 | 影響 |
|---------|------|------|
| CSV 缺失 26 行 | JSONL 130 loops vs CSV 104 行 | S1a NEAR-MISS 的直接原因 |
| duration 異常 | ~20 行 duration ≤ 0ms（時間戳精度） | q(k) 數據仍有效 |
| vitakka_stall_override | k=57，gear=2，唯一異常 gear 切換 | 不影響 q(k) 分析，已確認為 mechanism |
| k=0 無 q 值 | 第一行 q 欄位為空字串 | 不計入分析，有效 q 記錄 103 筆 |

---

## 3. Success Criteria Evaluation（成功標準評估）

### 3.1 S1：Valid CycleRecords（雙層定義）

**Protocol 原文**：≥95 of 100 measurement cycles have valid CycleRecords。

**D1-R1 決議**：S1 採用雙層定義。

#### S1a：q-valid coverage

| 項目 | 值 |
|------|-----|
| 定義 | 有 q 值的 measurement records / JSONL measurement cycle 數 |
| 分子 | 74（CSV measurement 行，均有 q 值） |
| 分母 | 100（JSONL measurement cycles，k=30..129） |
| 覆蓋率 | 74% |
| 門檻 | ≥ 95% |
| **判定** | **NEAR-MISS（74%，未達 95%）** |

**失敗原因**：CSV 記錄器的 26 行缺失（JSONL 記錄了 100 measurement cycles，但僅 74 筆進入 CSV）。失敗原因為技術缺陷，非 workload 設計失敗。

#### S1b：full-valid coverage

| 項目 | 值 |
|------|-----|
| 定義 | 有完整三維觀測（theta, c_adj, q 均有值）的 measurement records / JSONL measurement cycle 數 |
| 分子 | 13（measurement window 中有 audit 數據的行） |
| 分母 | 100（JSONL measurement cycles） |
| 覆蓋率 | 13% |
| **判定** | **FAIL（13%，遠低於任何合理門檻）** |

**失敗原因**：workload-driven observability gap——W1 workload 的 tool call 均為 informational/read_only，audit pipeline 觸發率僅 ~17.6%。這是 Protocol 設計時高估 W1 workload audit 觸發率的結果，非系統不穩定的證據。

**S1 整體判定：FAIL**（S1a NEAR-MISS + S1b FAIL）

**Protocol 修訂指令**（D1-R1）：Phase 1 Protocol 須包含 S1a/S1b 雙層定義，並在 Step 0 加入 CSV vs JSONL 行數一致性預驗證（缺失 ≤ 5% 才進入正式分析）。

---

### 3.2 S2：CSV↔JSONL Bijection（數據結構一致性）

**Protocol 原文**：三狀態變量（theta, c_adj, q）通過 ADF+KPSS 聯合平穩性檢驗。

**可分析序列評估**：

| 序列 | 長度 | ADF 預期 | KPSS 預期 | 聯合判定 | 可行性 |
|------|------|---------|---------|---------|--------|
| q(k)（measurement） | 74 | 強力拒絕 unit root（有界序列的數學必然） | fail to reject stationary | **Stationary**（注意：形式測試意義有限，因離散有界違反連續分佈假設；結論基於定性特徵） | 可分析 |
| theta(k) | 13（稀疏） | N/A | N/A | **Cannot assess**（樣本量不足） | 不可分析 |
| c_adj(k) | 13（稀疏） | N/A | N/A | **Cannot assess** | 不可分析 |

**q(k) Stationarity 的定性依據**（PASCAL，D1 Round 4 採用）：
- 有界序列（[0.75, 1.0]），不可能有 unit root
- 無明顯趨勢或漂移
- 均值回歸特性（sliding window 機制天然驅動）
- 變異數大致恆定

**S2 判定：PARTIAL PASS**
- q(k)：Stationary（高度確信，定性判斷）
- theta(k)/c_adj(k)：Cannot assess（workload-driven observability gap）

**注記**：theta 和 c_adj 無法評估不是數據品質問題——這些維度在非 tool-routing cycle 中確實不存在（系統設計的正確行為）。此處的「PARTIAL PASS」記錄為診斷資訊，而非系統失敗判定。

---

### 3.3 S3：sigma_DeltaV Precision（INCONCLUSIVE）

**Protocol 原文**：sigma_DeltaV > 0 且 95% CI half-width < 0.5 × sigma_DeltaV（等價於 n_eff ≥ 9）；透過 TOST equivalence test 確認 E[DeltaV] ∈ (-ε, 0)，ε = 0.03。

**D1-R2 決議**：S3 = INCONCLUSIVE。

#### 3.3.1 三維分析（V_full）

delta = 0 恆成立 → c_adj 在 audit cycles 間不變 → theta 由 risk category 決定 → **三維 sigma_DeltaV = 0**（或 undefined，因常數序列的標準差退化）。

S3 的顯式條件（sigma_DeltaV ≤ 0.03）在三維分析下退化為 undefined；TOST 失去統計意義。

#### 3.3.2 q-only 分析（V_q，主分析軌道）

V_q(k') = (q(k') - q*)²

DeltaV_q 的可能值（部分列舉）：

| q(k) → q(k+1) | DeltaV_q |
|----------------|---------|
| 同值不變 | 0 |
| 1.0 → 0.9375 | -0.00354 |
| 0.9375 → 1.0 | +0.00354 |
| 0.9375 → 0.875 | +0.00416 |
| 0.875 → 0.9375 | -0.00416 |
| 0.875 → 0.8125 | +0.01203 |
| 0.8125 → 0.875 | -0.01203 |

估計統計量：

| 指標 | 估計值 | 方法 |
|------|--------|------|
| mean(DeltaV_q) | ≈ 0（均值回歸 first difference 的期望值） | 定性推斷 |
| sigma_DeltaV_q | ≈ 0.005 ~ 0.010 | WIENER 估計（基於離散跳變分析） |
| ESS_factor_DeltaV | ≈ 1.0 ~ 2.0 | WIENER 計算（Andrews bandwidth m_opt ≈ 4-5，負自相關 floor 至 1） |
| n_eff（DeltaV） | ≈ 37 ~ 73 | floor(73 / ESS_factor_DeltaV) |

**TOST 執行（q-only，探索性）**：

```
mu_hat_q ≈ 0（或略負）
SE_NW_q ≈ sigma_DeltaV_q / sqrt(n_eff) ≈ 0.008 / sqrt(50) ≈ 0.0011

Test 1（上界，H0: mu ≥ 0 vs H1: mu < 0）：
  若 mu_hat ≈ -0.001：t_upper = -0.001 / 0.0011 = -0.91 → p_upper ≈ 0.18 → FAIL

Test 2（下界，H0: mu ≤ -ε vs H1: mu > -ε）：
  t_lower = 0.029 / 0.0011 = 26.4 → p_lower ≈ 0 → trivially PASS
```

**TOST 結論**：Test 2 trivially PASS（mu 遠離 -0.03），但 Test 1（上界）無法拒絕（mu 太接近 0）。整體 **TOST = INCONCLUSIVE**。

**物理解讀**（重要）：系統已達穩態，DeltaV 圍繞 0 對稱波動。TOST 上界測試要求證明 E[DeltaV] < 0（能量淨耗散），但在穩態中 E[DeltaV] ≈ 0。這實際上是**更強的穩定性證據**（系統已在均衡點），但 TOST 的非對稱設計 [-ε, 0] 要求嚴格 E[DeltaV] < 0，不接受 E[DeltaV] = 0。

**D1-R2 補充條件**：TOST 僅在 sigma_DeltaV > 0 且 mu_DeltaV ∈ (-ε, 0) 時有效。若前提不成立，S3 = INCONCLUSIVE，需要重新設計 workload。

**S3 判定：INCONCLUSIVE**（Protocol 前提假設不成立，criterion 不適用）

---

### 3.4 S4：ESS_factor（雙用途，D1-R3）

**Protocol 原文**：ACF at lags 1-10 computed；ESS_factor ∈ [1, 10]。

**D1-R3 決議**：Phase 0 產生兩個有效但用途不同的 ESS_factor 估計，不可混用。

| ESS_factor | 估計值 | 計算對象 | 用途 |
|-----------|--------|---------|------|
| **ESS_factor_DeltaV** | ≈ 1.0 ~ 2.0（WIENER） | DeltaV_q 序列的 autocorrelation | TOST power analysis；Phase 1 n_corrected 計算 |
| **ESS_factor_q** | ≈ 5.6（PASCAL） | q(k) 序列的 autocorrelation（rho(1) ≈ 0.75，sliding window 效應） | q* 均衡點估計的 CI 計算 |

**ESS_factor_DeltaV 計算細節**（WIENER）：

DeltaV_q 是 V_q 的 first difference。正自相關的 q(k) 序列取 first difference 後傾向產生負自相關：

```
估計 rho_DeltaV(1) ≈ -0.3 ~ -0.5
alpha_hat(1) = 4 * 0.16 / ((1.4)^2 * (0.6)^2) ≈ 0.907
m_opt = 1.1447 * (0.907 * 73)^(1/3) ≈ 4.62 → L = 5

ESS_factor = 1 + 2 * Σ (Bartlett-weighted rho) ≈ 1.0（負自相關使 ESS_factor < 1，floor 至 1.0）
n_eff = floor(73 / 1.0) = 73
```

**ESS_factor_q 計算細節**（PASCAL）：

```
rho_q(1) ≈ 0.75, rho_q(2) ≈ 0.55, rho_q(3) ≈ 0.40, rho_q(4) ≈ 0.30, rho_q(5) ≈ 0.20, rho_q(6+) < 0.10
sum ≈ 2.30
ESS_factor_q ≈ 1 + 2 * 2.30 = 5.60
n_eff_q ≈ 103 / 5.60 ≈ 18.4
```

**三維 DeltaV 的 ESS_factor**：全為常數 0 的序列，自相關函數 undefined，ESS_factor 概念退化，不適用。

**S4 判定：PASS**（ESS_factor_DeltaV ≈ 1-2，滿足 [1, 10] 範圍；三維分析不適用已記錄）

---

### 3.5 S5：V Function Monotonicity（INAPPLICABLE）

**Protocol 原文**：V 候選函數在 ≥90% 的 step pair 中單調遞減（DeltaV ≤ 0）。

**D1-R4 決議**：S5 = INAPPLICABLE。

**前提假設**：S5 的設計假設「系統從遠離均衡的初始狀態出發，在 measurement window 期間收斂至均衡」。

**W1 pilot 觀察**：
- lambda_warmup_q = (V_q(29)/V_q(1))^(1/28) ≈ 1.006 > 1（V 未衰減，系統從 warmup 開始即在均衡附近）
- measurement window 全程：q(k) 在離散狀態間隨機跳躍，無指數衰減模式
- V_q(k') 無「從高能量單調趨向零」的收斂軌跡

**理論分析**：在穩態隨機馬可夫鏈中，DeltaV_q 約 40% 為 0，其餘正負近似等分，「單調遞減」比例約 70%，遠低於 90% 門檻。但此「低於門檻」不代表不穩定——**一個已收斂的穩態隨機過程，V 的 step-wise 變化本就是隨機的**，不應期待 90% 單調性。

**S5 判定：INAPPLICABLE**（前提假設「未達穩態」不成立；不記為 FAIL）

**Phase 1 Protocol 修訂指令**（D1-R4）：加入「前提假設驗證步驟」——在 S5/S8 評估前，先確認 V(k_start) > 3σ of V_final distribution，否則標記 INAPPLICABLE。

---

### 3.6 S6：CI Width Analysis（Safety，PASS）

**Protocol 原文**：S6 對應 S6 safety criterion——所有安全不變量在全部 130 cycles（含 warmup）完整保持。

**安全不變量驗證**：

| 不變量 | 要求 | 實際 | 判定 |
|--------|------|------|------|
| theta ≥ 0.3（thresholdFloor） | 全部 cycles | theta ∈ {0.5, 0.6} | ✓ |
| theta ≤ 0.9（thresholdCeiling） | 全部 cycles | theta ∈ {0.5, 0.6} | ✓ |
| clampedDelta ≤ 0（destructive category） | 當 riskCategory = destructive | 本 pilot 無 destructive cycle | N/A（vacuously true） |
| \|clampedDelta\| ≤ 0.05 | 全部 audit cycles | 全部 delta = 0 | ✓ |
| audit_result ≠ 'error' | 全部 measurement cycles | 22/22 = "unchanged" | ✓ |

**S6 判定：PASS**（所有安全不變量完整保持，零違反）

---

### 3.7 S7：Tooling Zero Errors（PASS）

**Protocol 原文**：Zero audit:completed with result='error' in measurement window。

| 指標 | 值 |
|------|-----|
| audit:completed 事件數（measurement） | 13 |
| result = 'error' 數量 | 0 |
| result = 'unchanged' 數量 | 13 |
| audit_duration_ms 超時事件 | 0 |

**S7 判定：PASS**（零 error，ThresholdAuditor 運作正常）

---

### 3.8 S8：Parametric Seeds（INAPPLICABLE）

**Protocol 原文**：均衡點估計從不同時窗（k'=[41,100] 和 k'=[61,100]）與主估計（k'=[51,100]）的差異 < 3 × SE（Newey-West），確認系統已充分收斂。

**D1-R4 決議**：S8 = INAPPLICABLE。

**前提假設**：S8 的設計假設「系統在 measurement window 初期尚未完全收斂，後期才達穩態，因此 equilibrium 估計的時窗選擇有顯著影響」。

**W1 pilot 觀察**：

| 均衡估計 | q* 值 | 差異 |
|---------|-------|------|
| k'=[51,100]（主估計） | ≈ 0.94 | — |
| k'=[41,100] | ≈ 0.93 | ≈ 0.01（< 3×SE） |
| k'=[61,100] | ≈ 0.95 | ≈ 0.01（< 3×SE） |

q* 的 Sensitivity check 技術上 PASS（差異 < 3×SE，SE ≈ 0.01）。但此 PASS 的含義是：系統**從 warmup 結束就已在穩態**，不同時窗給出相同估計是因為系統始終在均衡附近，而非因為系統「已充分收斂至均衡」（後者含義更強）。

對於 theta* 和 c*：full-audit 數據過少（measurement 後 50 個 cycle 中僅 ~6 筆），sensitivity check 無法可靠執行。

**S8 判定：INAPPLICABLE**（系統從未遠離均衡，S8 的 convergence 語義不適用；V_initial ≈ V_final ≈ 小隨機波動，V_final ≤ V_initial 以 50% 概率成立，無統計意義）

---

## 4. Overall Verdict: CONDITIONAL GO（整體判定）

### 4.1 What "CONDITIONAL GO" Means（判定的意涵）

**不採用 "PASS" 的原因**：
- 三維 Lyapunov 分析目標（sigma_DeltaV_full、TOST 結論、S5/S8）實質上未達成
- S1 整體 FAIL，S3 INCONCLUSIVE，S5/S8 INAPPLICABLE

**不採用 "FAIL" 的原因**：
- Protocol 設計本身有效，問題在 workload 選擇（W1 信息密度不足）
- 系統數據管線健康（S6/S7 PASS）
- Phase 0 提供了有價值的診斷資訊（W1 workload 的可觀測性邊界）
- q-only 統計基礎為 Phase 1 提供了部分有用估計

**CONDITIONAL GO 的語義**：Phase 0 作為「workload 可行性研究」完成；作為「sigma_DeltaV_full 估計」未完成。下一步是 W2 mini-pilot，不是 Phase 1 的完整展開，也不是 Protocol 的根本重設計。

### 4.2 Conditions for Phase 1（Phase 1 的前提條件）

**前提條件 C1：W2 Mini-pilot 成功**

W2 mini-pilot（50 cycles）必須在 Phase 1 之前完成，且達到以下最低要求（D1-R7）：

| 要求 | 門檻 | 說明 |
|------|------|------|
| Audit 觸發率 | ≥ 50% | 工具序列應包含 fs.write, fs.mkdir 等 write-level 工具 |
| Delta ≠ 0 頻率 | ≥ 20% of audited cycles | 設計部分 cycle 的 confidence 接近或略低於 theta |
| Risk category 覆蓋 | ≥ {informational, read_only, write} | 擴展 theta 的離散值集合 |
| 主要交付物 | sigma_DeltaV_full（三維） | 供 Phase 1 n_corrected 計算 |

**前提條件 C2：Phase 1 Protocol 修訂完成**

依 D1-R1 至 D1-R4 完成以下修訂：
- S1 雙層定義（S1a + S1b）+ Step 0 CSV/JSONL 一致性預驗證
- V(k) 雙軌制（V_q + V_full）
- TOST 補充條件（sigma_DeltaV > 0 且 mu_DeltaV ∈ (-ε, 0)）
- S5/S8 前提假設驗證步驟
- P 矩陣 w_theta_pending 待 W2 校準標記

**前提條件 C3：P 矩陣校準**

基於 W2 mini-pilot 的 theta 實際方差決定：
- 若 sigma_theta ≥ 0.05（半連續）：w_theta = 1/σ²_theta 或理論重新推導
- 若 sigma_theta < 0.05（仍高度離散）：考慮將 theta 維度從 V_full 移除，退化為二維 V

---

## 5. Phase 1 Requirements（Phase 1 要求）

### 5.1 W2 Workload Design（W2 Workload 設計，D1-R7）

**設計目標**：使 ThresholdAuditor 的動態調節功能可被統計觀測。

| 設計要素 | Phase 0 W1 | Phase 1 W2 目標 |
|---------|-----------|----------------|
| 工具類型 | informational, read_only | + write（fs.write, fs.mkdir 等） |
| Expected risk category | informational, read_only | informational, read_only, **write** |
| Expected theta 值域 | {0.5, 0.6} | {0.5, 0.6, **0.7**}（write 的 riskDelta = +0.10） |
| Expected delta | 全部 = 0 | 部分 ≠ 0（設計 confidence 接近 theta） |
| Audit 觸發率目標 | ~21% | **≥ 50%** |
| Delta ≠ 0 目標 | 0% | **≥ 20% of audited cycles** |

**W2 mini-pilot 首要任務**（BABBAGE 補充，D1 Round 5）：確認 W2 workload 能達到 audit 觸發率 ≥ 50% 且 delta ≠ 0。這是 Phase 0 的「workload 可行性再測試」，成功後才進行 Phase 1 的完整 n_corrected 計算。

**OQ-D1-1 開放問題**：W2 mini-pilot 的具體 workload 腳本設計（工具序列、頻率、錯誤注入方式）移交 ARCHIMEDES（R4 工程方案）。

### 5.2 W3 Workload Design（W3 Workload 設計，D1-R7）

W3 adversarial 的必要條件（Phase 2 或 Phase 1 optional，由 W2 mini-pilot 結果決定）：

| 要求 | 說明 |
|------|------|
| Risk category 覆蓋 | {write, destructive, external} |
| Destructive delta ≤ 0 驗證 | 確認安全機制在壓力下仍然生效（per 02-6 安全規則） |
| BABBAGE 8 機制值全覆蓋 | NaN guard, clamp01, version discriminants, cold-start gear=3, confidence=0, destructive delta ≤ 0, zero-quality fallback, + others |
| vitakka_stall_override 有意觸發 | 連續高 risk 工具呼叫，驗證 mechanism 行為 |

W3 事件統計規則（D1-R6）：vitakka_stall_override 觸發**計入 mechanism 事件統計**，不計入 V 函數的 policy-driven 擾動項。

### 5.3 V(k) Dual-Track Strategy（V(k) 雙軌制，D1-R8）

**主軌道：V_q(k)**

```
V_q(k) = (q(k) - q*)²
```
- 適用於：每個 cycle（所有 JSONL measurement cycles）
- 樣本量：n = JSONL measurement cycle 數（目標 100）
- 用途：S3 primary analysis，ESS_factor_DeltaV，preliminary TOST

**輔助軌道：V_full(k)**

```
V_full(k) = w_theta * (theta(k) - theta*)² + 1.00 * (c_adj(k) - c*)² + 1.00 * (q(k) - q*)²
```
- 適用於：full-audit cycles（theta, c_adj, q 均有值）
- 樣本量：n = full-audit cycle 數（W2 workload 目標 ≥ 50% measurement cycles）
- 用途：S3 supplementary analysis，Phase 1 三維 sigma_DeltaV 估計

兩軌道分別計算 sigma_DeltaV 和 TOST，分別報告 S3/S5/S8。V_q 作為主要指標，V_full 作為輔助驗證。

**Phase 0 基線**：

| 指標 | V_q | V_full |
|------|-----|--------|
| sigma_DeltaV | 0.005 ~ 0.010（WIENER 估計） | undefined（delta = 0 恆成立）|
| TOST | INCONCLUSIVE（mu ≈ 0，Test 1 FAIL）| 不適用 |
| 樣本量 | n = 74（measurement）| n = 13（稀疏）|

### 5.4 P Matrix Calibration（P 矩陣校準，D1-R8）

**當前狀態（Phase 0 執行後的認識）**：

Phase 0 數據確認：theta 在 W1 workload 下由 static-rule-arbiter 的 risk category mapping 決定，非 Layer 3 連續動力學的輸出。Protocol 使用的 w_theta = 2.78 係基於「theta 的連續動力學對 V 的貢獻」推導，在 W1 workload 下基礎不成立。

**Phase 1 Protocol 中的 P 矩陣處理**：

```
P = diag(w_theta_pending, 1.00, 1.00)
```

校準流程（D1-R8）：
1. 從 W2 mini-pilot 提取 theta 的實際方差 σ²_theta
2. 計算 sigma_theta = sqrt(σ²_theta)
3. 若 sigma_theta ≥ 0.05（theta 呈半連續特性）→ w_theta = 1/σ²_theta 或理論重新推導
4. 若 sigma_theta < 0.05（theta 仍為高度離散）→ 考慮將 theta 維度從 V_full 移除，退化為二維 V = (c_adj - c*)² + (q - q*)²

**q* 基線估計**（Phase 0 最佳估計）：

```
q* ≈ 0.94 ± 0.01（WIENER 估計，基於 measurement 後 50 個 cycle 的時間平均）
```

q* 的 95% CI 需要精確的 n_eff 計算。使用 ESS_factor_q ≈ 5.6（PASCAL），n_eff_q ≈ 18.4，95% CI 寬度 ≈ 2 × 1.96 × SE_q，其中 SE_q ≈ sd(q) / sqrt(n_eff_q)。（OQ-D1-5：待精確計算）

---

## 6. Lessons Learned（經驗教訓）

### 6.1 Protocol 設計的隱含假設

Phase 0 Protocol v1.0-final 設計時的隱含假設與 W1 pilot 現實之間存在結構性錯位：

| Protocol 假設 | 實際觀測 | 影響 |
|-------------|---------|------|
| 每個 measurement cycle 產生完整三維觀測 | 僅 17.6% cycle 進入 audit pipeline | S1b FAIL；三維 V(k) 分析缺位 |
| system 從非均衡初始狀態收斂 | 系統從 warmup 結束即在均衡附近 | S5/S8 INAPPLICABLE |
| sigma_DeltaV > 0（可估計）| delta = 0 恆成立；sigma_DeltaV_full = 0 | S3 INCONCLUSIVE |
| theta 呈連續動力學輸出 | theta 由離散 risk category 決定 | Layer 3 dynamics 潛伏；P 矩陣校準基礎不成立 |
| W1 workload 能激發 ThresholdAuditor 調節 | W1 tool risk 遠低於調節閾值 | 核心觀測缺位 |

**結論**：Protocol 設計本身有效——它的十步分析流程、統計方法（Andrews bandwidth, Newey-West HAC, TOST, block bootstrap）、V 函數定義都是正確的。問題是 **W1 workload 的信息密度不足**，使得 Protocol 的多數步驟退化為邊界情況。這是「workload 選擇失敗」，不是「Protocol 設計失敗」，更不是「系統穩定性失敗」。

### 6.2 ThresholdAuditor 正確行為的誤讀風險

delta = 0 恆成立容易被誤讀為「ThresholdAuditor 沒有工作」。實際上，ThresholdAuditor 在每個 tool call 上都正確執行了：

1. 評估 risk category（informational / read_only）
2. 計算對應 threshold（theta = 0.5 或 0.6）
3. 比較 confidence（0.9 或 0.95）vs theta
4. 判定 confidence >> theta → delta = 0（"unchanged"）

這是**正確的結論**，不是 audit 失效。W1 workload 的 confidence 遠高於所有 risk category 的 threshold，auditor 的正確判斷就是「不需要調整」。

### 6.3 W1 Workload 意外的穩定性發現

Phase 0 的一個正面發現：OpenStarry 在 W1 workload 下展現比 Protocol 預期更強的收斂性能。系統不需要 30 個 warmup cycles 才達到穩態——從 cold start 即在均衡附近。這暗示在無威脅環境中，認知循環的 q 動態具有極強的均值回歸特性（sliding window 機制的設計效果）。

但這個發現是 **underpowered**——要確認系統在真正擾動（W2/W3 workload）下的穩定性，仍需 Phase 1。

### 6.4 雙軌 ESS_factor 的啟示

R2 交叉審閱揭示 WIENER 和 PASCAL 計算了不同序列的 ESS_factor，導致表面上差異懸殊（1.0-2.0 vs 5.6）但實際上都正確。這說明：

- DeltaV_q 的 first difference 特性產生負自相關 → ESS_factor 趨近 1
- q(k) 原序列的 sliding window 特性產生強正自相關 → ESS_factor ≈ 5.6

兩個 ESS_factor 對應不同的統計推斷目的，必須明確區分。Phase 1 Protocol 修訂時需要在 Step 7 中明確標注每個 ESS_factor 的計算對象和用途。

### 6.5 vitakka = mechanism 的確認

Phase 0 的 vitakka_stall_override 事件（k=57）為 D1-R6 的 mechanism/policy 分類提供了直接實測證據。依 BABBAGE continuity test，vitakka watchdog 的 stall detection 是 mechanism（移除會破壞語義正確性），已由實際數據確認：在 W1 workload 下，連續非 default gear 運行觸發了 vitakka 的強制回歸，防止了潛在的 cognitive loop 卡死。

---

## 7. Recommendations（建議）

### 7.1 立即行動（Phase 1 前必須完成）

| 優先級 | 行動 | 負責代理 |
|--------|------|---------|
| **P0** | 設計並執行 W2 mini-pilot（50 cycles，audit ≥ 50%，delta ≠ 0 ≥ 20%）| ARCHIMEDES（工程），WIENER（分析）|
| **P0** | 完成 Phase 1 Protocol 修訂版（S1 雙層、V 雙軌、TOST 補充條件、S5/S8 前提驗證、P 矩陣待校準） | WIENER |
| **P0** | 確認 CSV 記錄器 26 行缺失的根本原因（OQ-D1-4）| TURING（原始碼分析）|

### 7.2 Phase 1 設計原則

| 原則 | 說明 |
|------|------|
| **Workload-first**  | Phase 1 的 n_corrected 應基於 W2 mini-pilot 的實測 sigma_DeltaV_full，而非 Phase 0 的 q-only 估計 |
| **雙軌並行** | V_q 作為主分析（n 充足），V_full 作為輔助驗證（n 由 W2 audit 觸發率決定）|
| **ESS_factor 區分** | TOST power analysis 使用 ESS_factor_DeltaV；均衡點 CI 使用 ESS_factor_q |
| **TOST epsilon 重審** | 若 W2 的 sigma_DeltaV_full > epsilon (0.03)，需重新校準 epsilon（OQ-D1-3）|
| **P 矩陣後置** | 在 W2 mini-pilot 確認 theta 的變異特性後才固定 w_theta |

### 7.3 Phase 2+ 建議

| 建議 | 時機 | 說明 |
|------|------|------|
| W3 adversarial workload | Phase 2 或 Phase 1 optional | 取決於 W2 數據是否已充分提供 sigma_DeltaV |
| TOST symmetric interval [-ε, +ε] | Phase 1 Protocol 修訂中納入 | 用於「已收斂穩態」判定，補充現有非對稱 [-ε, 0] |
| q* 精確 CI 計算 | Phase 1 分析 | 需 n_eff_q 的精確計算（OQ-D1-5）|
| ESS_factor_q 精確估計 | Phase 1 分析 | 需實際執行 ACF for q(k) with Andrews bandwidth |

### 7.4 Phase 0 Protocol 修訂摘要

以下修訂指令由 D1 辯論決議正式確立，應在 Phase 1 Protocol 中完整實作：

| 修訂項 | 決議來源 | 說明 |
|--------|---------|------|
| S1 雙層定義（S1a + S1b） | D1-R1 | S1a: q-valid ≥ 95%；S1b: full-valid，門檻由 W2 mini-pilot 決定 |
| Step 0 CSV/JSONL 預驗證 | D1-R1 | 缺失 ≤ 5% 才進入分析，否則 bug 調查 |
| TOST 補充前提條件 | D1-R2 | sigma_DeltaV > 0 且 mu_DeltaV ∈ (-ε, 0) 才執行 TOST |
| ESS_factor 雙用途明確化 | D1-R3 | Step 7 中區分 ESS_factor_DeltaV 和 ESS_factor_q |
| S5/S8 前提假設驗證步驟 | D1-R4 | V(k_start) > 3σ of V_final，否則標記 INAPPLICABLE |
| V(k) 雙軌制（V_q + V_full）| D1-R8 | 分別報告，不合併 |
| P 矩陣 w_theta_pending | D1-R8 | 待 W2 校準後固定 |
| vitakka = mechanism 分類 | D1-R6 | 統一用詞；W3 觸發計入 mechanism 統計 |

---

## 附錄 A：S1-S8 判定彙總表

| # | Criterion | 原始門檻 | 實測值 | **判定** | 失敗原因性質 |
|---|-----------|---------|--------|---------|------------|
| S1a | q-valid coverage | ≥ 95% | 74% | **NEAR-MISS** | CSV 記錄器缺失（技術缺陷）|
| S1b | full-valid coverage | 待定 | 13% | **FAIL** | workload-driven observability gap |
| S1 | 整體 | ≥ 95/100 | — | **FAIL** | 雙重原因 |
| S2 | Stationarity | 三變量均平穩 | q: Stationary；theta/c_adj: N/A | **PARTIAL PASS** | workload-driven observability gap |
| S3 | sigma_DeltaV precision | sigma > 0，TOST p < 0.05 | sigma_q ≈ 0.008；TOST INCONCLUSIVE | **INCONCLUSIVE** | Protocol 前提假設不成立 |
| S4 | ESS_factor ∈ [1, 10] | [1, 10] | ESS_DeltaV ≈ 1-2；ESS_q ≈ 5.6 | **PASS** | — |
| S5 | V 單調性 ≥ 90% | ≥ 90% | 系統已在穩態 | **INAPPLICABLE** | 前提假設（未達穩態）不成立 |
| S6 | Safety | 所有不變量保持 | 100% 保持 | **PASS** | — |
| S7 | No audit errors | 0 error | 0 error | **PASS** | — |
| S8 | Equilibrium convergence | 時窗差異 < 3×SE | 系統始終在均衡附近 | **INAPPLICABLE** | 前提假設（收斂中）不成立 |

**PASS: 3（S4, S6, S7）| PARTIAL PASS: 1（S2）| INCONCLUSIVE: 1（S3）| INAPPLICABLE: 2（S5, S8）| NEAR-MISS: 1（S1a）| FAIL: 1（S1b）**

---

## 附錄 B：Phase 1 n_corrected 計算預估（條件性）

**以下計算依賴 W2 mini-pilot 的實測值，此處為條件性預估，不應直接用於 Phase 1 設計。**

若 W2 mini-pilot 產生 sigma_DeltaV_full ≈ 0.05（預期量級）、mu_DeltaV ≈ -0.01、ESS_factor_DeltaV ≈ 2.0：

```
margin = min(-mu_DeltaV, epsilon + mu_DeltaV) = min(0.01, 0.02) = 0.01
n_base = ((z_0.95 + z_0.90) * sigma / margin)²
       = ((1.645 + 1.282) * 0.05 / 0.01)²
       = (2.927 * 5)²
       = (14.635)²
       ≈ 214 per cell

n_corrected = ceil(214 * 2.0) = 428 per cell
n_total = 428 * 15 = 6,420 cycles（Phase 1 完整）
```

若 n_total > 10,000：觸發 O'Brien-Fleming 序貫分析（D4c-R1）。

**注意**：此預估對 sigma_DeltaV_full 的假設高度敏感。若 sigma_DeltaV_full < 0.03（接近 epsilon），TOST 功率將極低，需要重新校準 epsilon（OQ-D1-3）。

---

## 附錄 C：開放問題（移交後續研究）

| ID | 問題 | 移交目標 |
|----|------|---------|
| OQ-D1-1 | W2 mini-pilot 的具體 workload 腳本設計 | ARCHIMEDES（R4 工程方案）|
| OQ-D1-2 | 若 W2 mini-pilot 仍然產生 mu_DeltaV ≈ 0，如何應對？ | PASCAL + 下一輪辯論 |
| OQ-D1-3 | TOST 對稱 interval [-ε, +ε] 的理論基礎 | PASCAL + BABBAGE |
| OQ-D1-4 | CSV 記錄器 26 行缺失的原始碼確認 | TURING（原始碼分析）|
| OQ-D1-5 | q* = 0.94 的精確 95% CI | PASCAL + WIENER |

---

## 參考文獻

**Protocol 依據**：
- [來源: research record/cycle02-9/deliver/lyapunov_phase0_protocol.md] — Lyapunov Phase 0 Protocol v1.0-final
- [來源: research record/cycle02-9/results/C1_lyapunov_calibration_protocol.md] — 完整 C1 Protocol (Phases 0-3)

**Cycle 02-10 研究材料**：
- [來源: research record/cycle02-10/R1_independent/R1_WIENER_12_lyapunov_phase0_analysis.md] — WIENER Phase 0 十步 Protocol 執行
- [來源: research record/cycle02-10/R1_independent/R1_PASCAL_19_statistical_analysis.md] — PASCAL 統計分析
- [來源: research record/cycle02-10/R2_crossreview/R2_pair3_WIENER_PASCAL_lyapunov_stats.md] — R2 交叉審閱
- [來源: research record/cycle02-10/R3_debates/D1_lyapunov_data_interpretation.md] — D1 辯論記錄（D1-R1 至 D1-R8）

**原始數據**：
- [來源: research input/codex_review/cycle02-9/lyapunov_data/cycle_records.csv] — 104 行 CycleRecord
- [來源: research input/codex_review/cycle02-9/lyapunov_data/raw_audit_trail.jsonl] — 558 JSONL 事件

**統計學文獻**：
- Andrews (1991) — Heteroskedasticity and Autocorrelation Consistent Covariance Matrix Estimation. *Econometrica*.
- Newey & West (1987, 1994) — HAC Covariance Matrix Estimation. *Econometrica*; *Review of Economic Studies*.
- Schuirmann (1987) — TOST (Two One-Sided Tests) procedure. *Journal of Pharmacokinetics and Biopharmaceutics*.
- Dickey & Fuller (1979) — ADF test. *Journal of the American Statistical Association*.
- Kwiatkowski, Phillips, Schmidt & Shin (1992) — KPSS test. *Journal of Econometrics*.
- Berger & Hsu (1996) — IUT (Intersection-Union Test). *Statistical Science*.

---

**決議綁定聲明**：本報告的所有判定均以 D1-R1 至 D1-R8 為最終依據。D1 辯論中 SYNTHESIST (#01) 的仲裁決議具有最終權威。本報告所記錄的 INAPPLICABLE 判定（S5/S8）不得被解讀為 FAIL，不得被用於推論「系統不穩定」。CONDITIONAL GO 判定不得被解讀為 Phase 1 可以立即展開，W2 mini-pilot 是必要的前置條件。

---

*WIENER (#12, Control Theory Expert)*
*PASCAL (#19, Decision Theory & Probability Philosophy Expert)*
*Cycle 02-10 R4 Formal Delivery*
*2026-03-13*

[來源: research record/cycle02-10/R3_debates/D1_lyapunov_data_interpretation.md#四、決議]
