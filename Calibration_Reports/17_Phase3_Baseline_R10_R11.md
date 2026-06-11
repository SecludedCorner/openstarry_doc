# 17. Phase 3 Baseline — W2-R10 / R11 (Model Change Transition Window)

`[Cycle 03-11 新增]`

> **來源**: Cycle 03-11 R1 PASCAL/WIENER/BABBAGE + R3 Debate D1/D5/D6
> **核心學者**: PASCAL (#19), WIENER (#12), BABBAGE (#9), ARCHIMEDES (#16)
> **狀態**: PROVISIONAL（Rules #69-#73 Model Change Policy + FR-1/FR-2 適用）
> **相關文件**: Doc 16 (Phase 3 Baseline R9), Doc 75 (Rules #71/#72 FR-1/FR-2), Reference/03 ENG-FAB v1.6

---

## 1. 概述

W2-R11 是 gpt-5.4 模型變更後第二輪資料（R10 為首輪）。本文件記錄 W2-R10/R11 完整數據、Rules #69-#73 Model Change Policy 首輪運作分析、deterministic match 物理基礎、以及 ENG-FAB v1.6 retro 統計。

| 維度 | 內容 |
|------|------|
| 模型 | gpt-5.4（per Master 03-10 model change ratify） |
| Transition window | R10 / R11 / R12（per Rule #71 3-round mixed） |
| 觀察 | R10 與 R11 sigma 6 位小數結構性相等 |
| 預測 | R12 sigma 高機率仍 = 0.023753（deterministic regime） |
| 後續 | R13 起 transition 結束，rolling ref 改 mean(R11, R12) |

---

## 2. W2-R11 Raw Data Summary

| 指標 | 值 | 備註 |
|------|----|------|
| 完成率 | 50/50 | 4th consecutive perfect |
| 錯誤 | 0/50 | 乾淨執行 |
| Sigma | **0.023753** | **= R10 to 6 decimals**（structural deterministic match） |
| V11 ratio | **1.039x** vs new ref 0.02287 | 在 [0.7, 1.5] 控制範圍內 |
| Shadow decisions | 47 | Phase 3 Round 3 post-FINAL |
| Agreement rate | 100% (47/47) | 持續穩定 |
| SPC anomalies | 0 | Provisional limits（pre-recalibration）內 |
| Plugins loaded | 9 | gear-arbiter-dynamic + spc-monitor 同 R9/R10 |
| ENG-FAB v1.5 原評 | 40/41（F-8 N/A） | 字面合規但實質失明 |
| ENG-FAB v1.6 retro | **41/42 with 1 carry-forward-must** | F-8 PASS + 1 item enumerated, F-9 PASS |

---

## 3. R10 / R11 Sigma Deterministic Match Analysis

### 3.1 結構性相等

| 輪 | sigma (6 decimals) | rolling ref source |
|----|-------------------|-------------------|
| R10 | 0.023753 | mean(R8, R9) = 0.02223（pre-R10） |
| R11 | **0.023753** | mean(R9, R10) = 0.02287（pre-R11） |

**6 位小數完全相等**。依 PASCAL §5.4 計算，假設兩輪獨立：

- H_struct（structural deterministic regime）後驗信度 ≈ 0.998
- H_coincidence（統計巧合）後驗信度 ≈ 10⁻⁴⁸
- 結論：**結構性 deterministic match 而非統計巧合**

### 3.2 物理基礎

deterministic 模型（temperature=0 + 固定 prompt + 固定 audit event 序列）下：
- 同樣 input → 同樣 output → 同樣 clampedDelta sequence → 同樣 sigma
- 此非「機率上同」而是「結構上同」
- gpt-5.4 在當前 setup 下進入 fully deterministic regime

### 3.3 R12 預測

- R12 同 model 同 setup → 高機率 sigma 仍 = 0.023753
- 若成立 → max(sigma_i) - min(sigma_i) = 0 → **觸發 FR-2 zero-variance handling**（per Doc 75 §3）
- 進入 pooled event-level sigma mode（n_total = 240 events）

---

## 4. V11 Ratio Trajectory（gpt-5.4 transition window）

| 輪 | sigma | rolling ref | V11 = sigma / ref | 狀態 |
|----|-------|------------|------------------|------|
| R9 (gpt-5.3 last) | 0.021976 | mean(R7, R8) | 0.9917x | gpt-5.3 final |
| R10 (gpt-5.4 first) | 0.023753 | 0.02223 | 1.069x | transition R+1 |
| **R11 (本輪)** | **0.023753** | **0.02287** | **1.039x** | **transition R+2** |
| R12 (next) | 預測 0.023753 | mean(R10, R11) = 0.023753 | 預測 1.000x | transition R+3（最後一輪） |

V11 在 [0.7, 1.5] 控制範圍內（per Master 2026-04-18 MR_REQ_5 路徑 C：維持不收窄）。

---

## 5. Rules #69-#73 Model Change Policy 首輪運作分析

### 5.1 首輪適用結果

| Rule | 條目 | 首輪表現 | 細節 |
|------|------|---------|------|
| #69 | Model Change Classification | ✅ 正確分類 gpt-5.3 → gpt-5.4 為「同 vendor / 同代際 / 微版本」 | per `PASCAL §3.1` |
| #70 | Transition Window Length | ✅ 採 3-round mixed window（CR-1 修訂後） | R10/R11/R12 |
| #71 | Rolling Reference | ⚠️ 例文 ambiguity → FR-1 提案修訂 | 實務 (b)/(c) pattern 正確 |
| #72 | SPC Recalibration Trigger | ⚠️ zero-variance edge case → FR-2 提案 pooled mode | provisional limits 仍 valid |
| #73 | Documentation | ✅ 完整記錄於本 cycle deliver/ | O2_w2r11_analysis.md |

首輪運作**無重大缺陷**（1 minor CR + 1 structural CR）。

### 5.2 Provisional SPC Limits（pre-R12）

依 Rule #72 step 2（per Master 03-10 ratify）：
- sigma UCL = 0.02420
- sigma LCL = 0.01954
- 狀態：**RETAINED, recalibrate at R12**

W2-R11 sigma = 0.023753 在 UCL = 0.02420 內（差 0.000447），未觸發 anomaly。

### 5.3 R12 重校準計畫

- R12 後 new model 累積 3 輪（per Rule #72 條件滿足）
- 若 zero-variance 觸發（FR-2）→ pooled event-level sigma mode（n=240）
- 若 zero-variance 不觸發 → standard SPC recalibration（D2/d3 method）

---

## 6. 47 Shadow Decisions Agreement

| 維度 | 值 |
|------|----|
| Total shadow decisions | 47 |
| Agreements with actual | 47 |
| Disagreements | 0 |
| Agreement rate | **100% (47/47)** |
| Phase 3 Round 3 status | post-FINAL |

依 Doc 16 §3 同邏輯：當前配置下（coldStartGear=1, dwellCounter=0），shadow 必然 agree。100% 為**預期且非可疑**。Phase 3 Round 3 為 post-FINAL（per MEMORY: R11 was Round 3 FINAL, R12+ = post-closure steady-state）。

---

## 7. ENG-FAB v1.6 Retro 統計

### 7.1 v1.5 原評 vs v1.6 retro

| 項目 | v1.5 原評 | v1.6 retro |
|------|----------|-----------|
| Total items | 41 | 42 |
| F-8 (carry-forward-must) | N/A | **PASS + 1 item enumerated** |
| F-9 (openstarry_doc completeness) | (不存在) | **PASS**（Plan46 補登後） |
| 統計 | 40 PASS / 1 N/A | 41 PASS / 1 item enumeration / 1 PASS = **42/42 with 1 carry-forward-must disclosed** |

### 7.2 F-8 N/A → PASS + 1 item 的根因

- v1.5 F-8 字面定義「N/A if no waves deferred」
- Plan46 W2 K-3 hook 雖 wave 全交付，但 spc-monitor + gear-arbiter-dynamic plugin **未 bridge factory hooks**（item-level wire-in deferral）
- v1.5 F-8 wave granularity → 字面 N/A 但實質失明
- v1.6 修訂 PASS criteria：「列舉所有 deferred items, wave OR item granularity」
- Plan46 retro：F-8 PASS + 1 item enumerated（K-3 wire-in to Plan47 as CARRY-FORWARD-MUST, 聚合 C47-K3-M1~M5）

### 7.3 F-9 新增動機

- Plan46 首交付 openstarry_doc 缺漏（per Doc 74 §2.1）
- Coordinator 觸發 PLAN46-DOC-001 補登
- F-9 = ENG-FAB v1.6 對應 Rule #74 的執行檢查項

---

## 8. PROVISIONAL Status

依 Rule #59 HYPOTHESIS 標註：
- W2-R11 sigma 0.023753 為 PROVISIONAL Phase 3 baseline data（gpt-5.4 transition R+2）
- R12 後升 PROVISIONAL → 觀察 deterministic regime 是否 confirmed
- 若 R12 zero-variance → FR-2 pooled mode；新 limits 從 pooled std 推導
- 若 R13+ 持續 deterministic → 啟動 V11 收窄評估（Master_Ratification_V11_Narrowing.md 路徑 C：N≥10 post-model-change）

---

## 9. Cross-Reference

| 文件 | 對應段落 |
|------|---------|
| Doc 16 | Phase 3 Baseline R9（基線比較） |
| Doc 74 | Rule #74 (L1' = code + doc) |
| Doc 75 | Rules #71/#72 FR-1/FR-2 規則文字 |
| Reference/03 (v1.6) | ENG-FAB v1.6 F-8 修訂 + F-9 新增 §10 |
| O2_w2r11_analysis.md | W2-R11 完整分析 |
| O5_compliance.md §5.2 | MR_REQ_2 FR-1/FR-2 |
| O5_compliance.md §5.5 | MR_REQ_5 V11 收窄路徑 C |

---

## 10. Action Items

| # | Owner | Item |
|---|-------|------|
| A1 | PASCAL | W2-R12 報告依 FR-1 修訂例文格式撰寫 |
| A2 | PASCAL + KNUTH | R12 觸發 zero-variance 時 pooled mode 演算法實作 outline |
| A3 | WIENER | R13 V11 收窄評估啟動條件 check（N ≥ 10 post-model-change） |
| A4 | Coordinator + test team | W2-R12 test 流程更新 — 含 zero-variance check |
| A5 | ARCHIMEDES | ENG-FAB v1.6 ledger 新增 W2-R11 retro entry |

---

*Cycle 03-11 Calibration Report #17 — Phase 3 Baseline W2-R10/R11 (Model Change Transition Window)*
*Authored by PASCAL (#19), WIENER (#12), BABBAGE (#9); SCRIBE (#2) 收錄*
*Per R1 PASCAL/WIENER/BABBAGE + R3 D1/D5/D6 + O2_w2r11_analysis.md*
