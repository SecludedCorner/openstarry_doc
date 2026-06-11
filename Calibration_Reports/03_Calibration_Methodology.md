# 50. 校準方法論 — 五層信心度模型與四階段校準協議 (Calibration Methodology)

`[Cycle 02-8 新增: C1/C2 校準決議 | Cycle 03-7 更新: V11 Range Narrowing (Rule #61)]`

> Cycle 02-8 D4a-R1 ~ D4e-R1 共 5 項決議。本文件定義 OpenStarry 信心度路由系統的校準方法論——C1 五層信心度模型的驗證，以及 C2 四階段校準協議的完整規格。
>
> **核心學者**: PASCAL (#19), WIENER (#12), BABBAGE (#9), ATHENA (#5), KNUTH (#21)
> **依賴文件**: #37 Klesha 增益排程, #42 IGearArbiter 介面規格, #43 認知迴路品質監控, #44 安全架構總覽, #46 AuditContext 與 Extras 協議

---

## 1. 概述

OpenStarry 的路由決策依賴信心度 (confidence) 與閾值 (threshold) 的比較。信心度由五層模型計算，每層獨立可配置。校準 (calibration) 的目標是確認五層模型的組合輸出在統計上穩定且有意義。

### C1: 五層信心度模型

```
最終信心度 = θ_base + L1(Klesha) + L4(VedanaEmergency) → L3(quality) → L2(audit) → routing
```

| 層 | 名稱 | 作用 | 範圍 |
|----|------|------|------|
| L0 | SafetyMonitor | 硬性安全閘門 | boolean (pass/fail) |
| L1 | Klesha 增益排程 | 閾值偏置 (Sneha/Dvesha/Moha/Mana) | ±Σwᵢμᵢ |
| L2 | IConfidenceAuditor | 信心度微調 | ±0.05 (clamp) |
| L3 | ILoopQualityMonitor | 閾值 α 偏移 | ±α (預設 0.10) |
| L4 | VedanaEmergency | 緊急閾值提升 | +boost (max 0.15) |

### C2: 四階段校準協議

```
Phase 0 (Pilot) → Phase 1 (C1-only) → Phase 2 (C2-only) → Phase 3 (Joint Validation)
```

---

## 2. TOST 作為主要穩定性測試 [D4a-R1, 24/24 全票]

### 2.1 TOST (Two One-Sided Tests)

TOST 檢驗系統輸出是否在等價帶 (equivalence band) 內穩定：

```
H₀: |μ_observed - μ_reference| ≥ ε
H₁: |μ_observed - μ_reference| < ε
```

| 參數 | 值 | 說明 |
|------|-----|------|
| ε (等價帶寬) | 0.03 | 信心度空間的 3% |
| α_stat (顯著性) | 0.05 | 標準顯著水準 |

### 2.2 輔助測試

| 測試 | 用途 | 地位 |
|------|------|------|
| Mann-Kendall | 趨勢偵測 | 次要（偵測漂移） |
| Autocorrelation | 時間相依性 | 輔助（決定 Newey-West 頻寬） |

### 2.3 Newey-West HAC 標準誤差 [D4c-R1, 23/24]

所有最終統計推論**必須**使用 Newey-West HAC (Heteroskedasticity and Autocorrelation Consistent) 標準誤差。認知迴圈的連續輸出存在自相關 (autocorrelation)，傳統標準誤差會低估不確定性。

---

## 3. 先導實驗 (Pilot Run) [D4b-R1, 24/24 全票]

### 3.1 規格

| 項目 | 值 |
|------|-----|
| 迴圈數 | 100 cycles |
| 顯著性 | α = 0.10（寬鬆，用於估計） |
| 波數 | W1 (最基礎配置) |
| 環境 | 乾淨環境（無 Klesha 歷史） |

### 3.2 產出

| 估計量 | 用途 |
|--------|------|
| σ_DV (因變量標準差) | 計算正式實驗所需樣本量 |
| ρ (自相關係數) | 決定 Newey-West 頻寬 |
| ADF 統計量 | 確認序列平穩性 |
| 效果大小 (Cohen's d) | 校準正式實驗的 power |

### 3.3 樣本量修正

```
n_corrected = n_iid × (1 + 2Σᵏ ρ(k))
```

若修正後的有效樣本量不足，啟用 O'Brien-Fleming 序貫分析作為替代方案（上限：n_corrected × 15 ≤ 10000 cycles）。

---

## 4. C1→C2 序列校準 [D4d-R1, 24/24 全票]

### 4.1 四階段時間表

| 階段 | 內容 | 時程 | 前提 |
|------|------|------|------|
| Phase 0 | Pilot Run | 1 週 | 乾淨環境 |
| Phase 1 | C1-only (五層模型驗證) | 2-3 週 | Phase 0 完成 |
| Phase 2 | C2-only (校準協議驗證) | 2-3 週 | Phase 1 TOST 通過 |
| Phase 3 | Joint Validation (C1+C2) | 1 週 | Phase 1 + Phase 2 各自通過 |

### 4.2 Phase 1: C1-only

驗證五層信心度模型的各層獨立行為：

```
測試 C1-1: L1(Klesha) 關閉 → 輸出穩定 (TOST ε=0.03)
測試 C1-2: L1(Klesha) 開啟 → 輸出在 θ_base ± Σwᵢμᵢ 帶內
測試 C1-3: L4(VedanaEmergency) 觸發 → 閾值提升 ≤ 0.15
測試 C1-4: L2(Auditor) 關閉 → L3 單獨穩定
測試 C1-5: L3(Monitor) 關閉 → L2 單獨穩定
```

### 4.3 Phase 2: C2-only

驗證校準協議本身的統計有效性：

```
測試 C2-1: 重複 Pilot 3 次 → σ_DV 的 CV < 0.20
測試 C2-2: TOST 的 type I error rate ≤ α_stat (模擬)
測試 C2-3: Newey-West 在已知自相關下的覆蓋率 ≥ 0.95 (模擬)
```

### 4.4 Phase 3: Joint Validation

C1 和 C2 同時運行，驗證整合行為：

```
測試 J-1: 全層開啟 → 路由決策穩定 (TOST ε=0.03)
測試 J-2: Klesha 極端值 → 安全不變量維持
測試 J-3: 連續 1000 cycles → 無累積漂移 (Mann-Kendall p > 0.05)
```

---

## 5. 安全不變量 α-獨立 [D4e-R1, 24/24 全票]

### 核心原則

所有安全不變量在**任何** α 值下都必須成立。α 是校準參數，不是安全參數。

```
∀α ∈ [0.0, 1.0]:
  SafetyMonitor.isSafe() = true  // L0 不受 α 影響
  destructive_delta ≤ 0          // Cycle 02-7 D1-R1 永久規則
  clamp(audit_delta, -0.05, +0.05)  // L2 限幅不受 α 影響
```

### 特殊情況

| α 值 | 行為 | 說明 |
|------|------|------|
| α = 0 | 純配置回退（Config-only） | 所有 L3 調節歸零，僅 Config 閾值生效 |
| α ≥ 0.30 | 限制為 load-testing 環境 | 生產環境禁止使用 |

---

## 6. 與 Plan32 的關係

Plan32 Wave 5 (Schema Extension) 為校準提供必要的資料基礎：

| Plan32 變更 | 校準用途 |
|------------|---------|
| `audit:completed` 增加 `riskCategory` | Phase 1 C1-4/C1-5 需要按風險類別分組分析 |
| `audit:completed` 增加 `thresholdAtDecision` | Phase 2 C2-1 需要追蹤閾值隨時間的變化 |

Plan32 是校準的**前置依賴**——在 Plan32 部署前，校準只能在模擬環境中進行。

---

## 決議索引

| ID | 決議 | 票數 | 內容 |
|----|------|------|------|
| D4a-R1 | TOST 為主要穩定性測試 | 24/24 | 全票，ε=0.03, α=0.05 |
| D4b-R1 | Pilot Run 必須 | 24/24 | 全票，100 cycles, α=0.10 |
| D4c-R1 | Newey-West + 序貫分析 | 23/24 | HAC 標準誤差必須 |
| D4d-R1 | C1→C2 序列校準 | 24/24 | 全票，四階段框架 |
| D4e-R1 | 安全不變量 α-獨立 | 24/24 | 全票，任何 α 下安全成立 |

---

## 7. V11 Acceptance Range Update (Rule #61, Cycle 03-7)

> **核心學者**: NAGARJUNA (#7), WIENER (#12), DARWIN (#6), BABBAGE (#9)
> **依賴文件**: #60 Calibration Convergence Model, #63 W2 Phase2 Calibration Architecture, Rule #61

### 7.1 Range Change

| Parameter | Old Value | New Value | Effective |
|-----------|-----------|-----------|-----------|
| V11 lower bound | 0.50x | **0.70x** | W2-R7 |
| V11 upper bound | 2.00x | **1.50x** | W2-R7 |

Rule #61 codifies this narrowing. The old range [0.5x, 2.0x] spanned 0.03324 units of sigma space (given baseline sigma=0.02216). The new range [0.7x, 1.5x] spans 0.01773 units -- a **47% reduction** in acceptance width.

### 7.2 Rationale: Discriminatory Power

NAGARJUNA's R2 cross-review challenge identified the core problem: the old range was so wide that it accepted virtually any sigma value as "stable." A test that accepts everything is not a test; it is a formality. The narrowed range transforms V11 from a near-automatic pass into a judgment with genuine discriminatory power, aligning with MR-11 (quality controls must be meaningful).

### 7.3 Empirical Justification

All historical rounds pass under both the old and new ranges:

| Round | Sigma | Ratio to Baseline (0.02216) | Old Range | New Range |
|-------|-------|----------------------------|-----------|-----------|
| R4 | 0.020 | 0.903x | PASS | PASS |
| R5 | 0.022 | 0.993x | PASS | PASS |
| R6 | 0.021 | 0.948x | PASS | PASS |

The narrowing does not retroactively invalidate any prior result.

### 7.4 DARWIN: Damped Oscillation Observation

DARWIN's pattern analysis identified that sigma follows a **damped oscillation** pattern (0.020 -> 0.022 -> 0.021) with decreasing amplitude around a settling point, rather than monotone convergence. This is consistent with a system approaching equilibrium through alternating overshoots. The model predicts future sigma values will remain close to the baseline, supporting the feasibility of the narrower range. However, a transient spike is expected after significant system changes (e.g., provider version update).

### 7.5 Rolling Reference Diagnostic

A rolling reference diagnostic is introduced: the mean sigma of the last 2 rounds is computed and reported alongside the fixed-baseline V11 ratio. This captures recent trend information that the fixed baseline cannot.

The rolling reference is **diagnostic only** and does not gate any decisions. It serves as an early warning: if the rolling reference diverges from the fixed baseline while V11 still passes, it may indicate a slow drift.

### 7.6 Provider Change Protocol

A provider version change (e.g., inference model upgrade) may fundamentally alter sigma characteristics. Rule #61 includes:

1. **Reset**: The fixed baseline is invalidated upon provider version change.
2. **Recalibration**: 3 consecutive W2 rounds required to establish a new baseline.
3. **Interim**: During recalibration, V11 is reported but not gated (informational only).

### 7.7 Baseline as Sunya (空) -- NAGARJUNA Philosophical Note

NAGARJUNA provided a Madhyamaka (Middle Way) analysis of the baseline concept. The baseline sigma value (0.02216) is a **conventional designation** (prajnapti / 假名), not an inherently existing stable value:

- **Depends on conditions** (pratityasamutpada): the specific provider version, test data composition, plugin set, and system state at the time of measurement
- **Lacks inherent existence** (svabhava-sunya): there is no "true sigma" that the baseline approximates; the baseline is the best available conventional reference
- **Functions through dependent origination**: the baseline is useful precisely because it was produced by the same causal chain that produces current measurements

This analysis cautions against treating the baseline as a fixed truth. It is a tool for comparison, valid within its conditions of origination, and must be reset when those conditions change (Section 7.6).

---

*本文件為 Cycle 02-8 C1/C2 校準方法論交付物，Cycle 03-7 新增 V11 範圍收窄 (Rule #61)。*
*紀錄時間：2026-03-10 | 更新時間：2026-04-10*
