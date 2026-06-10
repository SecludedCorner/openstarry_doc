# 16. Phase 3 Baseline Established at W2-R9

`[Cycle 03-9 新增]`

> **來源**: Cycle 03-9 R1 BABBAGE + R3 Debate D2
> **核心學者**: BABBAGE (#9), WIENER (#12), TURING (#17)
> **狀態**: PROVISIONAL（9 parameters），upgrade criteria 見 §7

---

## 1. 概述

W2-R9 是 Phase 3 shadow decision 架構的**首次實際運作**。本文件正式建立 Phase 3 Baseline，作為後續 R10+ 的比較基礎。

所有參數狀態為 **PROVISIONAL HYPOTHESIS**（Rule #59），升級為 CONFIRMED 需額外數據支持（見 §7）。

---

## 2. W2-R9 Raw Data Summary

| 指標 | 值 | 備註 |
|------|----|------|
| 完成率 | 50/50 | 2nd consecutive perfect |
| 錯誤 | 0/50 | 乾淨執行 |
| Sigma | 0.021976 | 近 R5 baseline |
| V11 | 0.9917x | W2 歷史最接近 1.0x |
| Shadow decisions | 54 | Phase 3 FIRST operational |
| Agreement rate | 100% (54/54) | 見 §3 分析 |
| SPC anomalies | 0 | 預期 |
| Plugins loaded | 9 | 含 gear-arbiter-dynamic + spc-monitor（首次） |
| ENG-FAB | 39/39 (0 N/A) | 首次全段適用 |

---

## 3. PH3-B1: Agreement Baseline

| Parameter | Value | Status |
|-----------|-------|:------:|
| Total shadow decisions | 54 | |
| Agreements | 54 | |
| Disagreements | 0 | |
| Agreement rate | **100.0%** (54/54) | **PROVISIONAL** |

### 為什麼 100% 是預期且非可疑？

在當前配置下，100% agreement 是**唯一數學可能的輸出**：

1. `coldStartGear = 1`（Rule #63 L4 Config Activates）
2. `computeShadowDecision()` 在無 transition trigger 時返回 `gear = currentGear`
3. 整個 R9 期間 `dwellCounter = 0`（無 transition），shadow 始終返回 `gear = 1`
4. Static arbiter 在當前配置下對所有類別返回 `gear = 1`
5. 因此：`shadowGear == actualGear == 1` 對所有 54 次決策必然成立

### Phase 3 的**未來**診斷

100% rate **在未來 round 才會成為診斷指標**（當 transition triggers 引入後）。當前此 rate 確認：
- Shadow path **正確連接**
- 無產生意外分歧

### HYPOTHESIS Invariant

若 coldStartGear=1 且無 transition triggers 發生的 round 出現 agreement rate < 100%，即為 shadow computation bug。此作為 invariant check。

---

## 4. PH3-B2: Latency Baseline

| Parameter | Value | Status |
|-----------|-------|:------:|
| Mean latency | **0.0169ms** | PROVISIONAL |
| P95 latency | **0.0265ms** | PROVISIONAL |
| Max latency | **0.1292ms** | PROVISIONAL（含 cold-start outlier） |
| Min latency | 0.0072ms | PROVISIONAL |

### 解讀

- Shadow computation overhead **可忽略**（mean 0.0169ms vs cycle ~5500ms = 3×10⁻⁶）
- Max 0.1292ms 為 cold-start outlier（首次調用，JIT warmup）
- **符合** D7-Q36 5% 閾值要求（遠低於）

### 未來比較基準

R10+ 須報告：
- Mean latency，偏差 < 20% 視為穩定
- P95 latency，偏差 < 50% 視為穩定
- Max latency，可能因 GC/JIT 波動，不強制穩定

---

## 5. PH3-B3: Shadow/Audit Coverage

| Parameter | Value | Status |
|-----------|-------|:------:|
| Total audit entries | 107 | |
| Total shadow decisions | 54 | |
| Shadow/Audit ratio | **50.5%** | PROVISIONAL |

### 為什麼不是 100%？

Shadow 從 **cycle 10** 才開始 fire（MIN_N=10 觀察閾值）。Cycles 1-9（Block 1 informational）無 shadow。

### 未來趨勢

若使用 `calibrationState` snapshot（Plan44 Fix 12c）跨 cycle 保留狀態，R10 的 shadow/audit ratio 應更接近 100%。R10 可驗證此預期。

---

## 6. PH3-SPC: Provisional Control Limits

### PH3-SPC-1: Sigma Control Chart

基於 R5+R6+R8+R9 四輪 rolling Shewhart 計算（R7 excluded）：

| Parameter | Value | Status |
|-----------|-------|:------:|
| Center Line (CL) | 0.02187 | PROVISIONAL |
| UCL (+3σ) | **0.02420** | PROVISIONAL |
| LCL (-3σ) | **0.01954** | PROVISIONAL |

**R9 position**: 0.02198，接近 CL，在控制範圍內。

### PH3-SPC-2: Agreement Control Chart

**未建立**。100% agreement 為 coldStartGear=1 下的數學必然，無變異可計算 control limits。待 transition triggers 引入後建立。

### PH3-SPC-3: Latency Control Chart

基於 54 個 data points（排除 cold-start outlier）：

| Parameter | Value | Status |
|-----------|-------|:------:|
| Center Line (CL) | ~0.014ms | PROVISIONAL |
| UCL | **0.05ms** | PROVISIONAL |
| LCL | 0 | 結構約束（latency ≥ 0） |

### PH3-SPC-4: Per-Category n Control Chart

| Category | R8 n | R9 n | Moving range | Provisional CL | Warning if below |
|----------|:----:|:----:|:------------:|:--------------:|:----------------:|
| informational | 50 | 41 | 9 | 45.5 | 30 |
| read_only | 30 | 27 | 3 | 28.5 | 20 |
| state_modifying | 16 | 14 | 2 | 15.0 | 10 (= MIN_N) |
| destructive | 12 | 11 | 1 | 11.5 | 10 (= MIN_N) |

**極度 provisional**（僅 2 data points per category）。建議累積 5+ rounds 後正式建立。

### 標記為 HYPOTHESIS 的意義

所有 SPC limits **皆為 HYPOTHESIS**（Rule #59）。不用於自動決策（Rule #57 monitoring-only）。僅供：
- 文件基準
- 報告參考
- 未來實證評估

SPC Monitor plugin 本身使用**滾動計算**的 UCL/LCL，而非本文件值。本文件值為 **documentation baselines**。

---

## 7. Upgrade Criteria: PROVISIONAL → CONFIRMED

所有 PH3-B* 和 PH3-SPC-* 參數**可升級**為 CONFIRMED 當：

1. **10+ rounds 累積數據**
2. **無重大方法論變更**（Phase 3 架構未被 Plan46+ 顯著修改）
3. **近 5 rounds 穩定**（limits 變動 < 10%）
4. **R3 debate 批准**

升級流程：
- 測試團隊在測試報告中報告 parameter stability
- 研究團隊在 R1/R2 評估
- R3 debate 決定是否升級
- Master 批准最終升級

---

## 8. Rolling Reference Update

| Round | Sigma | Rolling Ref Post-Round |
|-------|-------|:---------------------:|
| R5 | 0.02216 | 0.02216 (locked baseline) |
| R6 | 0.02087 | mean(R5,R6) = 0.02152 |
| R7 | 0.01650 | EXCLUDED (infrastructure) |
| R8 | 0.02247 | mean(R6,R8) = 0.02167 |
| **R9** | **0.02198** | **mean(R8,R9) = 0.02223** |

R9 後 rolling reference = **0.02223**（+2.6% 從 R8 後的 0.02167）。

### V11 計算

V11 = R9 sigma / R5 baseline = 0.02198 / 0.02216 = **0.9917x**

Rule #61 範圍 [0.7x, 1.5x] **通過**。這是 W2 歷史最接近 1.0x 的 V11。

---

## 9. Per-Category Readiness

| Category | n | n/MIN_N | Readiness |
|----------|:-:|:-------:|:---------:|
| informational | 41 | 4.1x | **HIGH** |
| read_only | 27 | 2.7x | **HIGH** |
| state_modifying | 14 | 1.4x | **MODERATE** |
| destructive | 11 | 1.1x | **LOW** |

### 與 R8 比較

全部類別維持同一 readiness tier（無降級）。

### Destructive sigma 退化

Destructive sigma = 0.000000（所有 11 個 destructive delta 皆為 -0.04675 硬編碼）。這是**第 5 個連續 round** 的 degenerate destructive sigma。

根因：`fs.delete` 在 tool auditor 中的 delta 為固定 -0.04675。除非引入 destructive prompt diversity（不同刪除場景），此 sigma 將持續為零。

**決策**：per Rule #57，destructive 為 monitoring-only，此限制**可接受**。

---

## 10. Known Findings

從 BABBAGE 的 R1 分析：

| ID | 嚴重度 | 描述 |
|----|:------:|------|
| F-B9-R9-1 | INFO | Dynamic arbiter passthrough 產生 14 個零 delta 項目 |
| F-B9-R9-2 | LOW | Shadow activation gap (cycles 1-9) |
| F-B9-R9-3 | LOW | Destructive sigma 持續退化 |
| F-B9-R9-4 | MEDIUM | Plugin config propagation fragility (→ Plan45 W0-3 fix) |
| F-B9-R9-5 | INFO | Cycle 40 path scope error (正確處理) |
| F-B9-R9-6 | INFO | Rolling reference 收斂至 baseline |

---

## 11. 對 Phase 3 下一步的意義

### R10 期望

- 若 gear transitions 發生：首次獲得 non-trivial agreement 數據
- Sigma 應在 [0.02, 0.024] 範圍內
- SPC 可能首次偵測真實異常（而非 0）
- L2 Escalation Monitor 首次運作（Plan45 W1）

### R12 期望（Phase 3 Round 3 / 3）

- 累積足夠數據升級 PROVISIONAL → CONFIRMED
- Phase 3 non-inferiority hypothesis 首次評估（H-1, H-2）
- SPC limits 多輪確認

### Plan45 對 Phase 3 的強化

- L2+L3 填補觀測-響應循環
- Perturbation diagnostic 提供敏感度量測
- Context-dependent deltas 精細化 calibration

---

*Calibration Report #16 — Phase 3 Baseline Established*
*Cycle 03-9 R1 BABBAGE + R3 D2, 2026-04-15*
