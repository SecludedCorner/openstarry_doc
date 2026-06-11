<!-- Layer: 5-Reference -->
<!-- Status: NEW -->
<!-- Cycle: 03-6, R1/R3 -->
<!-- Author: BABBAGE (#9), KNUTH (#21), PASCAL (#19) -->
<!-- Source: O5 CV-5 Observe Mode Investigation -->

# R-3. Priority Deadlock 形式分析

**版本**: 1.0 (Cycle 03-6, 2026-04-09)
**作者**: BABBAGE (#9) — 形式驗證, KNUTH (#21) — 演算法分析, PASCAL (#19) — 機率論證
**來源**: CV-5 observe mode 調查, RC-1 + RC-2 雙重根因
**決議參照**: D2-Q1a (雙重根因確認, 7-0), D2-Q1b (兩者皆須修復, 7-0), D2-Q2a (shadow counting 採用, 7-0)

---

## 1. 概述

本文件提供 CV-5 observe mode deadlock 的形式分析，包括：
- Plugin priority 排序機制的精確描述
- Coupled counting-deciding 設計下 P(observe exit) = 0.0 的形式證明
- Shadow counting 作為解耦解決方案的形式規格
- RC-1 (payload) 與 RC-2 (deadlock) 的獨立性證明
- v0.41.0-alpha 機制清單更新

---

## 2. Plugin Priority 排序機制

### 2.1 Arbiter Chain 執行模型

OpenStarry 的 gear arbiter chain 按 priority 升序排列，數字越小優先級越高：

```
arbiters.sort((a, b) => a.priority - b.priority)
for (const arbiter of sortedArbiters) {
  const result = arbiter.evaluate(context)
  if (result.action !== 'abstain') return result  // 第一個非 abstain 的結果生效
}
return defaultDecision  // 所有 arbiter 都 abstain 時的預設
```

### 2.2 Current Priority Assignment

| Arbiter | Priority | 位置 |
|---------|:--------:|------|
| gear-arbiter-static | **10** | `gear-arbiter-static/src/index.ts:98` |
| gear-arbiter-dynamic | **20** | `dynamic-arbiter.ts:18` |

升序排列意味著 static (10) 在 dynamic (20) 之前 evaluate。

### 2.3 Static Arbiter 的 Non-Abstain 性質

**關鍵觀察**: Static arbiter **永遠不回傳 abstain**。它基於 `TOOL_CONFIDENCE_TABLE` 和 risk category 的固定映射，對每個工具呼叫都產生確定性的 gear decision。

形式化:

$$\forall c \in \text{GearContext}: \text{static.evaluate}(c).\text{action} \neq \text{abstain}$$

---

## 3. P(observe exit) = 0.0 的形式證明

### 3.1 定義

- $A_s$: static arbiter 回傳非 abstain 的事件
- $A_d$: dynamic arbiter 的 evaluate() 被調用且其結果被採用的事件
- $O$: dynamic arbiter 累積一次觀察的事件
- $E$: dynamic arbiter 退出 observe mode 的事件
- $N$: 退出 observe mode 所需的觀察次數 (MIN_N = 10)

### 3.2 耦合設計的假設

在原始 (耦合) 設計中，observation counting 發生在 evaluate() 的決策路徑中：

$$O \subseteq A_d \quad \text{(假設 1: 只有在做決策時才計數)}$$

### 3.3 Priority Chain 的結果

由於 static 永遠不 abstain:

$$P(A_s) = 1 \quad \text{(靜態 arbiter 永遠回傳非 abstain)}$$

在 priority chain 中，static 的非 abstain 結果使 dynamic 的結果被忽略（或 evaluate() 不被調用）:

$$P(A_d | A_s) = 0 \quad \text{(static 的結果覆蓋 dynamic)}$$

### 3.4 推導

$$P(O) \leq P(A_d) = P(A_d | A_s) \cdot P(A_s) + P(A_d | \neg A_s) \cdot P(\neg A_s)$$
$$= 0 \cdot 1 + P(A_d | \neg A_s) \cdot 0 = 0$$

因此:

$$P(E) = P(\text{count} \geq N) \leq P(O) = 0$$

$$\boxed{P(\text{observe exit}) = 0.0}$$

### 3.5 證明的完備性

此證明在以下兩個場景下都成立:

| 場景 | RC-1 狀態 | RC-2 狀態 | P(observe exit) |
|------|:---------:|:---------:|:---------------:|
| 兩者皆未修復 | 未修 | 未修 | 0.0 (zero deltas AND zero observations) |
| 僅修 RC-1 | 已修 | 未修 | 0.0 (deltas flow but counting still requires decision authority) |
| 僅修 RC-2 | 未修 | 已修 | 0.0 (counting decoupled but zero deltas reach tracker) |
| **兩者皆修** | **已修** | **已修** | **> 0** (deltas flow AND counting independent) |

**結論**: 只有同時修復 RC-1 和 RC-2，P(observe exit) 才不為零。[D2-Q1b]

---

## 4. RC-1 與 RC-2 的獨立性證明

### 4.1 RC-1: Payload Extraction Bug

**位置**: `calibration-bridge.ts`

```typescript
this.unsub = this.bus.on('audit:tool_audited', (event) => {
  const p = event.payload as { clampedDelta?: number; ... };
  if (!p) return;
  if (typeof p.clampedDelta === 'number') this.tracker.recordDelta(p.clampedDelta);
});
```

**Bug**: `event.payload` 的實際結構中 `clampedDelta` 不是頂層欄位。Guard `typeof p.clampedDelta === 'number'` 靜默失敗，`recordDelta()` 從未被調用。

**影響**: `deltas.length = 0` 永久。

### 4.2 RC-2: Priority Deadlock

**位置**: arbiter chain 的 priority 排序 + 耦合的 counting-deciding

**Bug**: static (priority 10) 永遠不 abstain → dynamic (priority 20) 的決策從不被採用 → 若 counting 耦合於 deciding → 永遠無法累積觀察。

**影響**: 即使 deltas 正確流入，觀察計數仍為零。

### 4.3 獨立性

| | RC-1 已修 | RC-1 未修 |
|---|:-:|:-:|
| **RC-2 已修** | **P > 0** (唯一可行) | P = 0 (deltas 不流入) |
| **RC-2 未修** | P = 0 (counting 被阻) | P = 0 (兩者皆壞) |

RC-1 和 RC-2 在不同的系統層面操作:
- **RC-1**: 資料平面 (data plane) — 事件 payload 提取
- **RC-2**: 控制平面 (control plane) — arbiter 執行順序與 counting 耦合

修復其一不會影響另一的修復。兩者是**獨立的必要條件** (independent necessary conditions) — 都需要修復，但修復順序無關。

---

## 5. Shadow Counting 形式規格

### 5.1 Before (Coupled)

```
evaluate(context):
  if this.observations < MIN_N:
    // Only counted when evaluate() result is actually used
    this.count(context)
    return { action: 'abstain' }
  // decision logic
```

### 5.2 After (Decoupled — Shadow Counting)

```
// CalibrationBridge (observer, runs on every event):
onEvent(event):
  delta = extractDelta(event.payload)  // RC-1 fix: correct extraction path
  tracker.recordDelta(delta)           // Always counted, regardless of priority

// DynamicArbiter (controller):
evaluate(context):
  if tracker.count < MIN_N:
    return { action: 'abstain' }
  // shadow decision logic (computed but may be discarded by priority chain)
```

### 5.3 Counting 語義 [D2-Q2b]

| 屬性 | 規格 |
|------|------|
| 計數單位 | **Per-event** — 每個 audit event 觸發一次 `recordDelta()` |
| 計數範圍 | **Per-category** — 每個 risk category 獨立計數 |
| 權威模型 | **Static 權威，dynamic shadow 學習** [D2-Q2c] |
| 窗口大小 | 20 (固定, StateTracker) |
| 退出條件 | 任一 category count ≥ MIN_N (10) |

---

## 6. RC-3: gear=1 硬編碼 [D2-Q2d, D2-Q4]

### 6.1 問題描述

`CalibrationBridge.recordOutcome()` 永遠記錄 `gear=1`：

```typescript
this.tracker.recordOutcome(1, p.executionResult === 'success');
```

不管實際活躍的 gear 為何。

### 6.2 嚴重度升級

| 舊嚴重度 | 新嚴重度 | 理由 |
|----------|----------|------|
| LOW | **MEDIUM** | 當 dynamic arbiter 開始切換 gear 時，per-gear success rate 將不準確 |

### 6.3 分類

正式列為 **Finding F-latent-1**，納入 Plan42 W2 修復範圍 [D2-Q4]。

---

## 7. 機制清單更新 (M-41-1 through M-41-8)

Plan41 新增的機制清單 (BABBAGE):

| # | 機制 | 位置 | 分類 |
|---|------|------|------|
| M-41-1 | ServiceKey\<T\> phantom type | `sdk/types/service.ts:32` | Mechanism |
| M-41-2 | DynamicArbiter hysteresis state machine | `gear-arbiter-dynamic/dynamic-arbiter.ts:48-54` | Mechanism |
| M-41-3 | StateTracker sliding window (n=20) | `gear-arbiter-dynamic/state-tracker.ts:22` | Mechanism |
| M-41-4 | CalibrationBridge audit subscription | `gear-arbiter-dynamic/calibration-bridge.ts:18` | Mechanism |
| M-41-5 | Destructive delta guard (negMean > 0) | `gear-arbiter-dynamic/dynamic-arbiter.ts:40-43` | Mechanism |
| M-41-6 | snapshot() freeze + clock copy | `distributed-alaya-impl.ts:216-221` | Mechanism |
| M-41-7 | restoreSnapshot() HMAC + freshness + dedup | `distributed-alaya-impl.ts:232-260` | Mechanism |
| M-41-8 | HMAC key env var clearing | `daemon-entry.ts:171` | Mechanism |

### 7.1 機制 vs 策略分類

Plan41 引入的策略常數：

| 常數 | 值 | 分類 | 理由 |
|------|-----|------|------|
| UP threshold | 0.047 | **Policy** | 可調 (tunable) 閾值 |
| DOWN threshold | 0.031 | **Policy** | 可調閾值 |
| MIN_DWELL | 5 | **Policy** | 可調配置 |
| MIN_N | 10 | **Policy** | 可調配置 |
| Static priority | 10 | **Policy** | 順序配置 |
| Dynamic priority | 20 | **Policy** | 順序配置 |
| Freshness threshold | 30000ms | **Policy** | 可調配置 |
| Window size | 20 | **Policy** | 可調配置 |

**Machine count: 8 | Policy count: 8**

Plan41 在 mechanism 與 policy 之間均衡 — 8 個新機制（結構性行為）加 8 個策略常數（可調參數）。策略常數集中在閾值和配置值，符合「Core = zero policy constants」原則（所有策略常數都在 plugin 中，非 Core）。

---

## 8. Acceptance Criteria 形式化 [D2-Q6a, D5-Q3]

Plan42 W2 的驗收條件以形式化方式表述：

| AC | 形式化條件 |
|----|-----------|
| AC-CV5-FIX-1 | $\exists e \in \text{Events}: \text{tracker.deltas.length} > 0 \text{ after } e$ |
| AC-CV5-FIX-2 | $\exists c \in \text{Categories}: \text{count}(c) > 0$ |
| AC-CV5-FIX-3 | $\exists c \in \text{Categories}: \text{count}(c) \geq 10 \implies \text{arbiter.evaluate}(ctx) \neq \text{abstain}$ |
| AC-CV5-FIX-4 | $\sigma_{R6} \in [0.5 \times 0.02216, 2.0 \times 0.02216]$ |
| AC-CV5-FIX-5 | $\text{static.priority} = 10 \wedge \text{dynamic.priority} = 20$ |
| AC-CV5-FIX-6 | $\text{attackSurface}(v_{42}) \setminus \text{attackSurface}(v_{41}) = \emptyset$ |
| AC-CV5-FIX-7 | $\text{modifiedFiles} \subseteq \text{gear-arbiter-dynamic/}$ |
| AC-W2-8 | $\text{CalibrationBridge.recordOutcome}(\text{actualGear}, ...)$ |
| AC-W2-9 | $\text{invalidPayload} \implies \text{log.warn}(\text{message})$ |

---

## 9. PASCAL 統計預測 (Post-Fix)

修復 RC-1 + RC-2 後，基於 W2-R5 的 108 筆資料分佈：

| Category | 事件/50 cycles | P(n ≥ 10 | 50 cycles) | 瓶頸? |
|----------|:---:|:---:|:---:|
| informational | 31 | > 0.99 | 否 |
| read_only | 16 | > 0.95 | 否 |
| state_modifying | 10 | ~0.50 | **是** |
| destructive | 12 | > 0.70 | 否 |

- P(all 4 exit within 50 cycles) = **0.65-0.80**
- P(all 4 exit within 100 cycles) = **> 0.99**

state_modifying 是瓶頸 (n=10 恰在閾值)。

---

## 10. 規則參照

| 規則 | 內容 | 來源 |
|------|------|------|
| **Rule #55** | Destructive operations NEVER delegated to dynamic arbiter (permanent) | D2-Q5b, 7-0 |
| **Rule #56** | Phased authority transfer: P2 shadow → P3 low-risk → P4 state_mod | D2-Q5a, 7-0 |

---

*Reference Document #03 — Priority Deadlock Formal Analysis*
*Cycle 03-6 Research Addition, 2026-04-09*
*BABBAGE (#9), KNUTH (#21), PASCAL (#19)*
*Decision References: D2-Q1a, D2-Q1b, D2-Q2a, D2-Q2b, D2-Q2c, D2-Q6a, D5-Q3*
