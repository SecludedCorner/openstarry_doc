# 72. WIENER L2+L3 Safety Framework (Plan45 Design)

`[Cycle 03-9 新增]`

> **來源**: Cycle 03-9 Plan45 WIENER L2+L3 設計
> **核心學者**: WIENER (#12), ARCHIMEDES (#16), BABBAGE (#9)
> **相關文件**: 44_Safety_Architecture_Overview.md (Tier 1/2/3)、07_SPC_Monitor_Plugin.md (L1)

---

## 1. 概述

WIENER 安全框架是**控制理論層級的觀測-反饋-響應結構**，與 doc #44 的三層安全框架（Tier 1/2/3）**正交**：
- 三層安全框架：**行動許可**（action permission）的分層
- WIENER L1/L2/L3：**觀測-異常-響應**的分層

WIENER 層的完整設計：

```
L3: Emergency Safety Gate        — 系統級保守干預（opt-in, 單次, 冷卻）
 ↑ 觸發：L2 critical count ≥ threshold
L2: Escalation Monitor           — 異常計數 + 時窗 + 分級
 ↑ 觸發：audit:spc_anomaly
L1: ShewhartChart (Plan44)       — 統計過程控制 ±3σ
 ↑ 觸發：audit:shadow_decision
L0: WIENER Gear Arbiter          — 磁滯 + Dwell + 負值約束
```

Plan44 已交付 L1（SPC Monitor）。Plan45 交付 **L2+L3**，完成 Tenet #8 feedback loop。

---

## 2. 為什麼需要 L2+L3？

### L1 單獨的侷限

Plan44 的 ShewhartChart 偵測異常點，但**無人響應**。`audit:spc_anomaly` 事件被 emit 但無 consumer，形成「開環」（open loop）。

從控制理論觀點（WIENER #12）：
- **觀測**（ShewhartChart）✅
- **異常識別**（±3σ 規則）✅
- **響應決策** ❌（缺）
- **動作執行** ❌（缺）

這違反 Tenet #8（完整的 control theory feedback loop）。

### L2 填補「觀測 → 決策」缺口

L2 不做個別點的異常判斷（那是 L1），而是**累積觀測**形成 escalation 級別。Rationale：單點異常可能是噪音，累積多個才是真訊號。

### L3 填補「決策 → 動作」缺口（條件性）

L3 是**最終安全層**，在極端情況下可強制系統進入保守模式。Rationale：若多個類別同時 critical，系統可能處於不安全狀態，需即時緩解。

**關鍵限制**：L3 是**單向保守**（gear 只能降，不能升），是**opt-in**（預設關閉），有**冷卻時間**（避免震盪）。

---

## 3. L2 Escalation Monitor 設計

### 責任

將 L1 的個別異常點聚合為 escalation 級別。

### 狀態

| 級別 | 條件（HYPOTHESIS） | 語意 |
|------|-------------------|------|
| `normal` | 異常數 < watch 閾值 | 系統正常 |
| `watch` | 異常數 ≥ 2 | 輕微觀察 |
| `warning` | 異常數 ≥ 4 | 警戒 |
| `critical` | 異常數 ≥ 7 | 嚴重 |

所有閾值為 **HYPOTHESIS**（Rule #59），可隨 R10+ 數據調整。

### 時窗

- **預設 window**：5 分鐘（HYPOTHESIS）
- **老化機制**：超過 window 的異常從計數中移除
- **分類獨立**：informational/read_only/state_modifying/destructive 各自計數

### 輸出

- **狀態變化時** emit `audit:spc_escalation` 事件
- 事件包含：category, previousLevel, currentLevel, anomalyCount, windowMs, timestamp
- **監控用**，不觸發任何自動動作（Rule #57）

### 不應用於 destructive？

Destructive 類別**計入**但**不觸發 L3**。這是 Rule #55（destructive NEVER delegated）的體現。L2 對 destructive 純粹為 monitoring-only。

### 程式介面（草稿）

```typescript
export interface EscalationConfig {
  readonly windowMs?: number;              // Default: 300000 (5 min)
  readonly thresholds?: {
    readonly watch?: number;               // Default: 2
    readonly warning?: number;             // Default: 4
    readonly critical?: number;            // Default: 7
  };
}

export class EscalationMonitor {
  processAnomaly(anomaly: SpcAnomaly): EscalationEvent | null;
}
```

---

## 4. L3 Emergency Safety Gate 設計

### 責任

在多類別同時 critical 時，**暫時強制系統降低 gear**（更保守）。

### 觸發條件

HYPOTHESIS：L2 critical-level 類別數 ≥ 2 同時發生（即多個風險面同時惡化）。

### 動作

**一次性強制 gear 降級**：
- 透過 `pushInput` 系統事件通知 gear-arbiter-dynamic
- 目標 gear = `max(1, currentGear - 1)`
- 保守方向（UP 禁止，DOWN 允許）

### 冷卻機制

**冷卻時間**：50 shadow decisions（HYPOTHESIS）。冷卻期間：
- L3 不再觸發
- 若狀況改善（critical → warning/watch），冷卻加速結束
- 若狀況持續，冷卻結束後可再次觸發

### Opt-in 設計

```typescript
{
  safetyGate: {
    enabled: false  // 預設關閉
  }
}
```

**為什麼預設關閉？**
- MR-8 品質第一：R10 是首次實證評估，不應預設打開
- 保守主義：自動干預系統行為需要實證基礎
- 符合 S-3（Full verification 持續）：需多輪 R 確認 L3 邏輯正確

### Master 決策責任

L3 的**啟用**屬 Master 決策。研究團隊僅提供設計與實作，不自動啟用。

---

## 5. 為什麼 L3 可以「干預行為」而 L2 不可以？

### 區別原則

| 層級 | 行為 | 依據 |
|------|------|------|
| L2 | 監控（記錄） | Rule #57 (monitoring-only for SPC) |
| L3 | 保守干預（單向，opt-in） | Master 直接指定 |

L3 的行為干預**不違反** Rule #57，因為：
1. 方向單向（保守）
2. 一次性（非持續）
3. 冷卻後停止
4. Master 選擇啟用

### Rule #55 的保留

Destructive 類別的決策**永遠不**由 L3 影響。L3 的 forceGear 只降級，不觸及 destructive 特定邏輯。

---

## 6. 與既有系統整合

### 事件流

```
Gear evaluate()
    ↓
audit:shadow_decision (Plan44)
    ↓
SPC Monitor (L1, Plan44)
    ↓
audit:spc_anomaly (Plan44)
    ↓
Escalation Monitor (L2, Plan45)
    ↓
audit:spc_escalation (Plan45, NEW)
    ↓
Safety Gate (L3, Plan45)
    ↓
audit:spc_safety_gate (Plan45, NEW)
    ↓
pushInput system event
    ↓
DynamicArbiter.forceNextGear() (Plan45 one-shot)
```

### Tenet 符合

| Tenet | L2 | L3 |
|-------|:--:|:--:|
| #2 (Plugin 架構) | ✅ plugin-scoped | ✅ plugin-scoped |
| #7 (Core 純淨) | ✅ 零 Core 修改 | ✅ 零 Core 修改 |
| #8 (Control Theory) | ✅ 完成觀測→決策 | ✅ 完成決策→動作 |

---

## 7. BIBO 穩定性分析

**BIBO（Bounded Input, Bounded Output）** 穩定性是 WIENER 的核心要求。

### L2 BIBO

- 輸入：有限數量的異常點
- 輸出：有限種狀態 + 有限事件
- 無正反饋（狀態不影響自身輸入）
- **BIBO stable** ✅

### L3 BIBO

- 輸入：L2 critical 事件
- 輸出：**一次性** gear 降級
- 冷卻機制防止快速循環
- Gear 降級 ≠ 放大異常 → 不形成正反饋
- **BIBO stable** ✅

### 風險：L3 + Phase 3 shadow 循環？

疑慮：L3 降 gear → shadow 觀測變化 → SPC 觸發 → L2 升級 → L3 再降？

**分析**：
1. Gear 降級後 dwell counter 重置（既有邏輯）
2. 冷卻期間 L3 不觸發
3. SPC window 滾動，歷史異常逐漸老化
4. 因此：**無法形成穩態振盪**

WIENER #12 的結論：設計符合 BIBO 原則。

---

## 8. 實驗性質的明確承認

### L2 閾值 = HYPOTHESIS

```
watch=2, warning=4, critical=7 (within 5 min window)
```

這些是**猜測的起點**（D7-Q18 精神）。實際值需 R10+ 多輪數據驗證。

### L3 冷卻 = HYPOTHESIS

```
cooldown = 50 shadow decisions
```

這是根據 W2 MIN_N=10 的 5 倍推算。實際值待觀察。

### 調整機制

- R10 執行後：評估 L2 是否 over/under triggering
- R12+ 後：基於累積數據調整閾值
- 所有調整須經 R3 debate

---

## 9. 與 Tier 1/2/3 的區分

重申（doc #44）：
- **Tier 1/2/3** 是**行動許可**的分層（SafetyMonitor, risk-weighted threshold, reduce complexity）
- **WIENER L1/L2/L3** 是**觀測-異常-響應**的分層

兩者**正交**。一個 action 可能同時：
- 被 SafetyMonitor (Tier 1) 阻擋
- 被 risk-weighted threshold (Tier 2) 約束
- 被 WIENER L3 降 gear 間接影響

但它們各司其職，無重疊。

---

## 10. 驗證計劃

### Plan45 交付後（v0.45.0-alpha）

| 測試 | 預期 |
|------|:----:|
| L2 unit tests | PASS (count, window pruning, level transitions) |
| L3 unit tests | PASS (trigger, cooldown, disabled state) |
| L2 + L3 integration | PASS (end-to-end chain) |
| L3 default=disabled | 驗證：R10 無 L3 gate 事件 |

### W2-R10 執行

**預期結果**：
- L2 可能 emit 少量 watch/warning（正常波動）
- L3 **不** emit（預設關閉）
- 無 gear forceDown 發生

### 實證評估（R12+）

累積 3 輪數據後：
- 評估 L2 閾值校準度
- 評估 L3 觸發條件的合理性
- 決定是否調整 HYPOTHESIS 值

---

## 11. 對 Tenet #10 (Awakened Foundation) 的意義

Tenet #10 要求系統對自身狀態有「覺察」（awareness）。WIENER L1/L2/L3 是此覺察的**技術體現**：

- L1 覺察個別事件
- L2 覺察累積模式
- L3 覺察系統級危機（並響應）

完成 Plan45 後，OpenStarry 首次具備**完整的自我觀察與響應循環**，這是 Tenet #10 從 PARTIAL 邁向 FULL 的關鍵一步。

（MR-5：Tenet #10 FULL compliance 待 Phase 6 完成；當前 Phase 3 active, Plan45 是 Phase 6 進程的一部分）

---

*Architecture Documentation #72 — WIENER L2+L3 Safety Framework*
*Cycle 03-9 R1 (WIENER + ARCHIMEDES), 2026-04-15*
