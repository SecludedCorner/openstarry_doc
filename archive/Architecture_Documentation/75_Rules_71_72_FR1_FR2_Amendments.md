# 75. Rules #71/#72 FR-1 / FR-2 Amendments — Rolling Reference Clarify + Zero-Variance Handling

`[Cycle 03-11 新增]` `[Master Ratified 2026-04-18, MR_REQ_2 Batch 2 APPROVED A-A]`

> **來源**: Cycle 03-11 R3 Debate D5 (FR-1) + D6 (FR-2)（兩項皆 24/24 UNANIMOUS）
> **核心學者**: PASCAL (#19, 主提兩 FR), WIENER (#12, cross-validate), BABBAGE (#9, 統計 cross-check), KNUTH (#21, invariant 補強), ARCHIMEDES (#16, 觸發精度)
> **基底規則**: Rule #71 (Rolling Reference Window, 03-10 ratify), Rule #72 (SPC Recalibration Trigger, 03-10 ratify)
> **批次**: MR_REQ_2 Batch 2（FR-1 minor + FR-2 structural，邏輯獨立可分別 ratify；Master 2026-04-18 雙方均 APPROVED）

---

## 1. 概述

Cycle 03-11 W2-R11 retro 評估時，PASCAL 揭露 Rule #71 例文與實務 pattern 的字面差異（FR-1）+ Rule #72 zero-variance edge case（FR-2）。兩 FR 皆為 Rules #69-#73 Model Change Policy 的 graceful handling 補強。

| FR | Rule | 性質 | 行為變化 | 即效 |
|----|------|------|---------|------|
| FR-1 | #71 例文 + invariant | minor CR (documentation clarify) | 無 | documentation 立即生效 |
| FR-2 | #72 zero-variance handling | structural CR (新增行為定義) | 有 (pooled mode) | W2-R12 觸發時生效 |

---

## 2. FR-1 — Rule #71 例文 Ambiguity 修訂 + KNUTH Invariant

### 2.1 歧義來源

Rule #71 規則文字本身正確，但例文「R10 rolling ref = mean(R9, R10) = 0.02287」存在三種讀法：

- **(a) 字面**：R10 的 ref = mean(R9, R10) — 但 R10 自己當輸入，self-reference 風險
- **(b) 實務**：R10 的 ref = mean(R8, R9) — 前兩輪 ref（不含當輪）
- **(c) 變體**：R10 的 ref = mean(R9, R10) — R10 已預估（偽 self-reference）

實務上 PASCAL/WIENER 採用 (b)/(c) pattern。例文可能誤標 R 編號（應為 R11 而非 R10）。

依 `PASCAL_w2r11_model_change_retro.md §F-MC-4`：CR minor，非阻斷性。

### 2.2 BABBAGE 統計支持

依 `R3 §D5.3 [17:38] BABBAGE`：mean(R-2, R-1) 是合理的 reference 計算（不含當輪以避免循環）；mean(R-1, R0) 含當輪會引入 self-reference 風險。例文若採 mean(R-1, R0) 字面，數學上有問題。

### 2.3 KNUTH Invariant 補強

依 `R3 §D5.3 [17:50] KNUTH`：rolling reference 的 self-reference 是 numerical instability 的經典 trap。例文修訂應同時加註 invariant：「ref must use prior rounds only, not current round」。PASCAL 接受此補充。

### 2.4 修訂後 Rule #71 完整文字

```markdown
## Rule #71 — Rolling Reference Window (3-round mixed)

**Original ratification**: 2026-04-16 (per Master_Ratification_Rules_69_to_73.md §3 + CR-1)
**FR-1 refinement**: 2026-04-18 (Master APPROVED A-A, Batch 2)

**Rule body**:
- N+1 (transition round): rolling ref = mean(N-1, N)（prior 2 rounds, not current）
- N+2: rolling ref = mean(N, N+1)
- N+3: rolling ref = mean(N+1, N+2) ← all new model
- N+3 onwards (transition ends): rolling ref = mean of last 2 new-model-only rounds

**Invariant** (FR-1 added per KNUTH):
> ref must use **prior rounds only**, not current round
> (prevents numerical self-reference instability; aligns with classical EWMA / SPC moving range)

**Reference example (FR-1 修訂)**:
R10 (transition, N+1): pre-R10 rolling ref = mean(R8, R9) = 0.02223
R11 (N+2):             pre-R11 rolling ref = mean(R9, R10) = 0.02287
R12 (N+3):             pre-R12 rolling ref = mean(R10, R11) = 0.023753
R13+:                  new-model-only ref = mean(R11, R12) (transition ends)
```

### 2.5 Scope 與限制

- **不變更 Rule #71 主體**：3-round mixed window 結構不動
- **不影響 active Plan**：Plan46 / Plan47 不依賴例文字面數字
- **不影響 W2-R10/R11/R12 計算**：實務已採 (b)/(c) pattern，本修訂只 codify 實務
- **CR 性質**：minor，非阻斷性

---

## 3. FR-2 — Rule #72 Zero-Variance Handling via Pooled Event-Level Sigma

### 3.1 退化情境

Rule #72 規定模型變更後 new model 資料累積 3+ 輪啟動 SPC 重校準。但存在未處理的退化情境：

依 `PASCAL_w2r11_model_change_retro.md §5.4`：
- W2-R10 + R11 sigma 已 **6 位小數結構性相等**（H_struct ≈ 0.998 後驗信度）
- 若 R12 仍同 model（gpt-5.4），有非零機率三輪 sigma 全同
- 後果：sigma_R = max(sigma_i) - min(sigma_i) = 0
- 標準 Shewhart Range chart：UCL = D4 × R̄ = 0，LCL = D3 × R̄ = 0
- **結果：UCL = LCL = CL，SPC 完全退化**

依 `PASCAL §F-MC-5`：MEDIUM severity，policy gap，FR-2 structural CR。

### 3.2 Deterministic Regime 物理基礎

依 `MEMORY.md::Current State`：W2-R10 與 W2-R11 sigma = 0.023753 為 6 位小數結構性相等（structural deterministic match），P(coincidence) ~ 10⁻⁴⁸。此為 deterministic 模型（temperature=0 + 固定 prompt）行為，**非統計巧合**。

R12 若同 model 同 setup，繼續 deterministic 全同的機率 high。FR-2 是針對此 known physical regime 的 graceful degradation handler。

### 3.3 BABBAGE 統計 Cross-Check 結論

依 `R3 §D6.3 [18:16] BABBAGE`：
- **240 events statistical power**：對 sigma 估計，n=240 提供 SE ≈ sigma / sqrt(2n) ≈ 4.5% 相對誤差，**充足**
- **d2 / d3 unbiasing**：標準 SPC d2/d3 表格在 n=2~25 範圍內適用；n=240 時 sigma 估計直接用 root-mean-square，**不需 d2/d3**
- **Westgard rules**：建議採 **Westgard 1-3s rule**（單點超 3 sigma）+ **Westgard 2-2s rule**（連續兩點同方向超 2 sigma）作為 zero-variance fallback 判定機制

### 3.4 修訂後 Rule #72 完整文字

```markdown
## Rule #72 — SPC Recalibration Trigger (含 FR-2 Zero-Variance Handling)

**Original ratification**: 2026-04-16 (per Master_Ratification_Rules_69_to_73.md §3)
**FR-2 refinement**: 2026-04-18 (Master APPROVED A-A, Batch 2)

**Rule body (original)**:
- 模型變更即為 SPC 重校準條件
- 延至 new model 資料累積 3+ 輪
- 若新限界與舊限界差 >15%：進入 formal R3 debate
- 在重校準前，provisional limits 繼續生效

**FR-2 增補：Zero-Variance Handling**

**Trigger**:
- 若 N (≥2) 輪 same-model gpt-5.x sigma 滿足 `max(sigma_i) - min(sigma_i) ≤ 0.0001`（4 位小數零差）即進入 pooled event-level sigma mode
- ARCHIMEDES 補強精度（per R3 §D6.3）

**Pooled Event-Level Sigma Mode**:
1. 資料展開：80 events × N rounds (N ≥ 2)，n_total = 80N
2. RMS sigma:
     pooled_std = sqrt(sum((x_i - x_bar)²) / (n_total - 3))
   其中 x_i 為各輪所有 audit events 的 clampedDelta，x_bar 為 pooled mean。
   **不用 d2/d3 unbiasing**（n=240 時 RMS 直接適用，per BABBAGE）
3. SPC limits 重校準:
     UCL = CL + 3 × pooled_std / sqrt(n_per_round)
     LCL = CL - 3 × pooled_std / sqrt(n_per_round)
   其中 n_per_round = 80，CL = pooled mean
4. **Westgard 1-3s rule**：單點超 ±3 sigma → trigger alarm
5. **Westgard 2-2s rule**：連續兩點同方向超 ±2 sigma → trigger alarm
6. Sample size：80 × N events；R12 觸發時 n_total = 240，SE ≈ 4.5%

**Extreme degenerate case (pooled_std = 0)**:
- Retain 既有 limits + 標 "zero-variance baseline"
- Monitoring degrades to fingerprint exact-match check（sigma value match → GO）
```

### 3.5 與 Rule #69-#73 Model Change Policy 的整合

| 規則 | 處理 |
|------|------|
| Rule #69 + Rule #70 | model change disturbance（外因擾動） |
| Rule #71 + Rule #72 | transition window + recalibration cadence（時序處理） |
| **FR-2** | deterministic regime degenerate case（內因 zero-variance） |

三者邏輯獨立且互補；FR-2 為 Rules #69-#73 graceful handling 補強。

---

## 4. FR-2 與 W2-R12 的時序關聯

- **R12 為觸發時機**：若 R12 sigma = 0.023753（與 R10/R11 6 位小數相等預測），即 trigger pooled mode
- **W2-R12 todo_test_instructions** 已含 zero-variance trigger check（per `R4 §5.2`）
- **FR-2 ratify 後 immediate 生效**：W2-R12 觸發時直接套用 pooled mode

---

## 5. 為何 FR-2 屬 Structural CR 而非 Minor

| 維度 | FR-1 (minor) | FR-2 (structural) |
|------|-------------|------------------|
| 行為變化 | 無 | 有（pooled mode 計算路徑） |
| Documentation 變化 | clarify 例文 + 加 invariant | 新增 sub-clause + RMS + Westgard |
| 驗證方法變化 | 無 | W2-R12 觸發時新計算流程 |
| Master ratify 必要性 | 嚴格論可 SCRIBE 自加 note，本輪仍提交 ratify | 必須 Master ratify |

---

## 6. 與其他 Ratification 的 Cross-Impact

- **與 Batch 1 Rule #74**：邏輯無依賴，平行 ratify
- **與 Batch 5 V11 收窄（路徑 C）**：若 R12 進入 pooled mode，N_eff 計算方式變化，需 WIENER 在 V11 收窄評估時調整（per Master_Ratification_FR_2.md A8）
- **與 Batch 3 9/0/1 條件**：FR-2 不影響 9/0/1† → ★ 升級條件 α/β/γ/δ；屬獨立 SPC behavior 補強

---

## 7. Action Items

| # | Owner | Item |
|---|-------|------|
| A1 | PASCAL | W2-R12 報告若觸發 pooled mode，須詳列 trigger 數值 / pooled_std 計算 / 新 UCL/LCL / Westgard rule 觸發狀態 |
| A2 | KNUTH | 審查 pooled mode 演算法實作 outline（W2-R12 test 前完成） |
| A3 | Coordinator + test team | W2-R12 test 流程更新 — 含 zero-variance check 與 pooled mode toggle |
| A4 | WIENER | V11 收窄評估時若 R12 進入 pooled mode，調整 N_eff 計算方式 |
| A5 | PASCAL | 後續 W2 報告依 FR-1 修訂例文格式撰寫 |

---

## 8. 參考

### 8.1 R1/R2/R3/R4 來源
- **FR-1 R1**: `R1_independent/PASCAL_w2r11_model_change_retro.md §5.3` + `§F-MC-4`
- **FR-1 cross-validate**: `R1_independent/WIENER_w2r11_v11_trajectory.md §1.3`
- **FR-2 R1**: `R1_independent/PASCAL_w2r11_model_change_retro.md §5.4` + `§F-MC-5`
- **FR-2 cross-validate**: `R1_independent/WIENER_w2r11_v11_trajectory.md §5.3 + §7.3 場景 A`
- **FR-2 BABBAGE cross-check**: `R1_independent/BABBAGE_plan46_L2_spec.md §10 + §11`
- **R2**: `R2_crossreview/R2_crossreview.md §六.1 D5 + D6 議題`
- **R3**: `R3_decision_log.md §D5 全節（FR-1）` + `§D6 全節（FR-2）` + `§C Batch 2 MR_REQ_2`
- **O5**: `deliver/O5_compliance.md §5.2 MR_REQ_2`
- **R4**: `R4_finalization/R4_synthesis.md §4 6-batch packaging strategy`

### 8.2 Master Ratification
- `discussions/Master_Ratification_FR_1.md`（Cycle 03-11 standalone, Master 2026-04-18 APPROVED A-A）
- `discussions/Master_Ratification_FR_2.md`（同 Batch 2）

### 8.3 Statistical Methodology References
- Wheeler & Chambers (1992), *Understanding Statistical Process Control* 2nd ed.
- AIAG (2005), *Statistical Process Control Reference Manual* 2nd ed. (short-run SPC pooled std methodology)

### 8.4 Vote
24/24 UNANIMOUS × 2（D5 + D6 各別表決，per R3 §D5.4 + §D6.4）

---

*Cycle 03-11 Architecture Documentation #75 — Rules #71/#72 FR-1 / FR-2 Amendments*
*Authored by PASCAL (#19), WIENER (#12), BABBAGE (#9), with KNUTH (#21) invariant supplement*
*SCRIBE (#2) 收錄*
