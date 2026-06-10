# 43. 認知迴路品質監控架構 (Cognitive Loop Quality Monitoring)

`[Cycle 02-4 新增]` `[Cycle 02-5 修訂: D1 佛學映射清理 + D2 蘊歸屬重新分類 + D4 命名矯正]` `[Cycle 02-6 新增: D3 四維計算公式 + 事件訂閱分階 + extras 整合 + 權重配置]`

> Cycle 02-4 R3 D2 全部 8 項決議 + D4 三層穩定架構。Cycle 02-5 D1 佛學映射邊界裁定 + D2 蘊歸屬重新分類 + D4 命名矯正 (ISatiMonitor → ILoopQualityMonitor)。
>
> **實作狀態**: Phase 1 (Plan27b) — SafetyMonitor.liveness + VitakkaWatchdog 接線。Phase 2+ (Plan29, v0.29.0-alpha) — PluginHooks `monitors` 槽位 + MonitorRegistry + ILoopQualityMonitor SDK 型別 + DEFAULT_LOOP_QUALITY_WEIGHTS 已完成。LoopQualityMonitor event-driven plugin 實體留待 Plan30+。

---

## 1. 概述

LoopQualityMonitor 是五蘊認知迴圈的**品質監控 plugin**（quality monitoring plugin），衡量整個認知迴圈的運作品質——認知事件是否被持續觀察、異常是否被即時偵測、觀察是否平衡不偏。

### 命名歷史

本元件原名「SatiMonitor」（源自佛學正念 smṛti 的類比），Cycle 02-5 D4 延長討論中發現命名矛盾：正念在佛學中屬行蘊 (samskara)，但本元件的蘊歸屬明確排除行蘊（D2-R2）。依第四項測試（定義責任測試），更名為 `LoopQualityMonitor`。

[來源: Cycle 02-5 D4-R1, 13/20 通過]

### 核心原則

LoopQualityMonitor 採用**事件驅動監控**（event-driven monitoring）——當認知事件發生時，監控器透過 EventBus 訂閱接收通知，不需要額外的 polling 動作。Polling 是退化行為（degradation behavior），事件驅動監控在設計上優於 polling。

### 蘊歸屬

LoopQualityMonitor 的五蘊組成為 **skandha: ['vedana', 'samjna', 'vijnana']**（受蘊+想蘊+識蘊），明確**排除行蘊 (samskara)**。

| 功能 | 蘊歸屬 | 說明 |
|------|--------|------|
| EventBus 事件訂閱（11 種事件類型的持續感知） | vedana | 領納認知迴圈信號——LoopQualityMonitor 的感知通道 |
| 滑動窗口 + 模式識別（重複/異常/時間模式） | samjna | 從事件流中辨認行為模式並標記類別 |
| LoopQualityVector 計算 + SPC 異常判斷 | vijnana | 評估性的品質判斷，超越描述性的模式辨認 |
| **不執行工具、不修改狀態、不切換齒輪** | **排除 samskara** | LoopQualityMonitor 是純觀測器，不做造作 |

受蘊成分相對較薄（事件訂閱是「領納」而非完整的苦/樂/捨三受評價），但其獨立退化策略（§9）和持續廣域感知特徵使其足以獨立聲明——LoopQualityMonitor 的「持續訂閱、主動感知」特徵使其區別於被動接收參數的 IGearArbiter。

[來源: Cycle 02-5 D2-R2, 18/20 通過]

---

## 2. 設計背景

### 設計約束

LoopQualityMonitor 的核心設計約束是：監控必須基於事件驅動（event-driven），而非定時輪詢（polling）。Polling 在兩次檢查之間存在覺知空白，事件驅動消除了此空白。

### 決議歷史

本架構的工程規格源自兩輪辯論：

**Cycle 02-4 D2**（8 項決議）——建立了 LoopQualityVector、Level/Depth 架構、退化策略、MRA 資源配置等核心工程規格。

**Cycle 02-5 D1 + D2**——佛學映射清理 + 蘊歸屬重新分類：

| # | 決議 | 核心內容 | 票數 |
|---|------|---------|------|
| D1-Q2-A | 佛學映射清理 | 「event-driven = 正念」等映射 REMOVE | 20/20 |
| D2-R1 | 文件標題修正 | 標題改為「認知迴路品質監控架構」 | 19/20 |
| D2-R2 | 蘊歸屬重分類 | skandha: ['vedana', 'samjna', 'vijnana']，排除行蘊 | 18/20 |
| D2-R3 | PluginHooks 擴展 | 新增 `monitors?: ILoopQualityMonitor[]` 槽位 (Plan29) | 20/20 |

### 三層穩定化架構 (D4)

LoopQualityMonitor 位於 D4 三層穩定化架構的頂層，VitakkaWatchdog 為中層，SafetyMonitor 為底層（見 §11）。

### 微核心原則

LoopQualityMonitor 是 **plugin**（非核心內建）。Agent 在沒有 LoopQualityMonitor plugin 的情況下仍可運作（只是缺乏品質監控），安裝 LoopQualityMonitor plugin 後品質監控能力提升。這符合微核心的「零內建能力」原則——品質監控是可選的增強功能，不是核心的內建組件。

---

## 3. 架構模型：品質屬性 + 漸進深化

LoopQualityMonitor 是一個**可透過 plugin 配置漸進增強的品質屬性**，而非獨立的認知組件。

**(D) 品質屬性**：LoopQualityMonitor 不獨立產生認知——它衡量整個五蘊迴圈的運作品質。沒有五蘊迴圈就沒有品質可監控。

**(C) 漸進深化**：監控能力可漸進式深化——從粗略的計數器監控（VitakkaWatchdog）到精細的事件驅動監控（LoopQualityMonitor plugin），再到連續流分析（未來 Phase 3）。

---

## 4. Level 固定 / Depth 動態 — 核心架構決策

這是 D2 辯論的**核心成果**——解決了 R1+R2 階段最大的分歧（SYNTHESIST Divergence D-6）。

### HERACLITUS 的 CPU 類比

| 概念 | CPU 類比 | 監控映射 | 可變性 |
|------|---------|----------|--------|
| **Level** | ISA（指令集架構） | 監控架構能力——能看到什麼 | 部署配置時固定 |
| **Depth** | Clock Speed（時脈頻率） | 監控精細度——看得多快多細 | Runtime 動態可調 |

**Level = 架構能力，部署時固定**（如同 CPU 指令集）

- Level 1: VitakkaWatchdog only — 計數器能力（最小覺察）
- Level 2: LoopQualityMonitor plugin — 事件驅動覺察（標準生產環境）
- Level 3: 連續流分析 — 進階能力（Phase 3 預留）

Level 1 的 Agent 不能在 runtime「升級」到 Level 2——因為它沒有載入 LoopQualityMonitor plugin。要升級需要重新部署或動態載入 plugin，而這本身是部署級操作。

**Depth = 資源配置，Runtime 動態可調**（如同 CPU 時脈頻率）

- 在固定 Level 內調整監控精細度
- Level 1 的 Depth：Gear 1 計數閾值
  > **Plan27a 修整**: N-Gear 泛化後，齒輪監控深度不限於 Gear 1。`gear:switch` 事件 payload 包含齒輪號（`{ gear: number, decidedBy?, confidence }`），LoopQualityMonitor 可追蹤任意齒輪的切換頻率和停留時間。
- Level 2 的 Depth：取樣率、事件過濾範圍、分析窗口長度
- Level 3 的 Depth：分析演算法複雜度、回溯深度

Depth 受 Level 能力所限——Level 2 無論 Depth 如何提高，都做不到 Level 3 的連續流分析。

[來源: Cycle 02-4 D2-R2]

### 控制理論形式化 (WIENER)

Level 對應**觀測器結構**（Observer Structure），Depth 對應**觀測器增益**（Observer Gain）：

```
Level: 觀測器結構 O_L = (A_L, C_L, K_L)
  L=1: C_1 = [gear_count]  (只觀測齒輪計數)
  L=2: C_2 = [gear_switch, vedana_assessment, klesha_update, ...]  (事件驅動)
  L=3: C_3 = [continuous_state_stream]  (連續流)

Depth: 觀測器增益 K(t) — 在 O_L 結構內動態調整
```

**關鍵定理**：可觀測性由 $(A_L, C_L)$ 決定，與增益 $K$ 無關。如果 Level 1 的觀測矩陣 $C_1$ 不包含某個狀態變量，無論增益如何調整，該狀態變量都是不可觀測的——這從數學上證明了 Level 切換和 Depth 調整是本質不同的操作。

[來源: Cycle 02-4 D2-R2, WIENER 控制理論分析]

### 三級模型

| Level | 名稱 | 能力 | Depth 可調項 | 實作時機 |
|-------|------|------|-------------|---------|
| 1 | VitakkaWatchdog | 計數器監控 | Gear 1 閾值 | Plan27b |
| 2 | LoopQualityMonitor Plugin | 事件驅動覺察 | 取樣率、事件過濾、分析窗口 | Plan28 |
| 3 | 連續流分析 | 連續流處理 | 分析複雜度、回溯深度 | Phase 3 |

Phase 3 預留了 Level 動態切換的設計空間——不在 Phase 1/2 實作，但不從架構上永久排除。

[來源: Cycle 02-4 D2-R2, 16/18 通過]

---

## 5. LoopQualityVector — 四維品質度量

WIENER 從控制理論出發提出四維品質向量，用於量化認知迴圈的監控品質。

```typescript
/**
 * LoopQualityVector — 認知迴圈品質四維向量
 *
 * 四維度：
 * - Continuity: 監控覆蓋的時間連續性
 * - Granularity: 監控覆蓋的事件粒度
 * - Speed: 監控的響應速度
 * - Equanimity: 監控對正面/負面事件的覆蓋平衡性
 *
 * 各維度值域 [0, 1]。
 */
export interface LoopQualityVector {
  /** 連續性 Continuity — 監控在觀測窗口內的活躍時間比例 */
  readonly continuity: number;

  /** 精細度 Granularity — 訂閱事件類型數 / 系統總事件類型數 */
  readonly granularity: number;

  /** 速度 Speed — 1 - (平均響應延遲 / 最大可容忍延遲) */
  readonly speed: number;

  /** 平衡性 Equanimity — 1 - |正面事件覆蓋率 - 負面事件覆蓋率| */
  readonly equanimity: number;
}
```

### 四維定義

**C (Continuity / 連續性)**：監控在時間上的無間斷程度。

$$C = \frac{\sum t_{active}}{\Delta T}, \quad C \in [0, 1]$$

監控覆蓋率越高，連續性越好。

**G (Granularity / 精細度)**：監控能觀察到的事件細節程度。

$$G = \frac{|\text{subscribed event types}|}{|\text{total event types}|}, \quad G \in [0, 1]$$

觀察的事件類型越多，精細度越高。

**S (Speed / 速度)**：從事件發生到監控器覺察的延遲。

$$S = 1 - \frac{\bar{\delta}_{response}}{\delta_{max}}, \quad S \in [0, 1]$$

$\delta_{max}$ 為可容忍的最大延遲（configurable）。

**E (Equanimity / 平衡性)**：監控對正面和負面事件的觀察是否平衡。

$$E = 1 - |f_{positive} - f_{negative}|, \quad E \in [0, 1]$$

平衡性衡量的是覆蓋率的對稱性——監控器不應只關注負面事件而忽略正面事件，或反之。

[來源: Cycle 02-4 D2-R3 19/19 通過]

### SDK 實作維度對照 (Plan29)

Plan29 SDK 實作 (`packages/sdk/src/types/loop-quality-monitor.ts`) 採用重新命名的四維，語義對應如下：

| 設計規格 (Doc 43) | SDK 實作 (Plan29) | 語義映射說明 |
|-------------------|-------------------|-------------|
| Continuity (連續性) | `coherence` | 從時間連續性泛化為邏輯一致性——涵蓋時間與語義兩面 |
| Granularity (精細度) | `efficiency` | 從事件粒度泛化為資源利用效率——涵蓋覆蓋面與效率 |
| Speed (速度) | `convergence` | 從響應速度泛化為目標收斂性——涵蓋速度與方向 |
| Equanimity (平衡性) | `stability` | 從覆蓋平衡泛化為振盪穩定性——涵蓋平衡與穩定 |

重新命名的理由：設計規格的四維偏向監控層面的觀測指標 (C/G/S/E)，SDK 實作的四維偏向認知迴圈本身的品質屬性 (coherence/efficiency/convergence/stability)，更適合作為 Model Delta Layer 3 的輸入信號。兩者值域均為 `[0, 1]`，語義可互通。

---

## 6. Event-Driven 架構

LoopQualityMonitor plugin 透過 EventBus 訂閱監控認知迴圈的健康狀態，而非 polling：

```
EventBus events → LoopQualityMonitor plugin 訂閱：
  - gear:switch             齒輪切換事件
  - gear:arbiter_evaluated  IGearArbiter 評估結果
  - vedana:assessment       受蘊評估變化
  - klesha:update           煩惱狀態更新
  - volition:deliberation   IVolition 審議決策
  - tool:result             工具執行結果
  - coarising:bundle        五遍行快照
  - sparsha:contact          觸事件（原始輸入）[Plan27b 新增]
  - action:proposed          行動提案 [Plan27b 新增]
  - action:executed          行動執行結果 [Plan27b 新增]
  - vitakka:stall            尋伺停滯（看門狗觸發）[Plan27b 新增]
```

LoopQualityMonitor 評估品質後發射報告事件：

```typescript
/**
 * LoopQualityReport — 品質報告事件 payload
 * 定期發射至 EventBus (建議每 10 ticks)。
 * 外部系統（部署監控、日誌分析、人工審計）可訂閱追蹤。
 */
export interface LoopQualityReport {
  readonly vector: LoopQualityVector;
  readonly spcStatus: 'normal' | 'warning' | 'action' | 'disabled';
  readonly degradationStage: 'normal' | 'throttle' | 'sample' | 'emergency_drop';
  readonly ticksSinceLastReport: number;
  readonly depth: {
    readonly samplingRate: number;
    readonly subscribedEvents: number;
    readonly totalEvents: number;
  };
}
```

```
bus.emit({
  type: 'loop:quality_report',
  timestamp: Date.now(),
  payload: LoopQualityReport
});
```

LoopQualityMonitor 採用**異步 buffer + batch 設計**——同步 listener 只做 O(1) 入 buffer（不阻塞 EventBus 主路徑），批次處理在非關鍵路徑異步執行。此設計是退化策略（§9）的工程前提。

---

## 7. Core 最低覺察基線

Agent Core 有獨立於 LoopQualityMonitor plugin 的最低覺察——此覺察不屬於 LoopQualityMonitor 的監控範疇。

### SafetyMonitor 存活性擴展

SafetyMonitor 已提供硬性安全底線（動作安全性檢查）。Cycle 02-4 D2-R1 擴展存活性偵測（loop liveness check），解決 TURING §3.2 發現的殭屍狀態——crash recovery 後 `running` 仍為 `true` 但 ExecutionLoop 已停止處理事件。

```
SafetyMonitor (核心內建)
  ├── isSafe()              — 迴圈入口安全預檢（既有）
  ├── postCheck()           — LLM 回應後安全檢查（既有）
  ├── afterToolExecution()  — 工具執行後安全檢查（既有）
  └── loopLivenessCheck()   — [Cycle 02-4 新增] 殭屍狀態偵測
        └── 每完成一個 tick 更新 lastTickTimestamp
        └── timestamp 超過 N 秒未更新 → 報告異常
```

### 覺察層級

| 層級 | 功能 | 實作 |
|------|------|------|
| 基礎設施 | 存活性偵測（heartbeat / liveness probe） | SafetyMonitor.liveness |
| Level 1 | 計數器監控（粗粒度看門狗） | VitakkaWatchdog |
| Level 2 | 事件驅動監控（品質度量） | LoopQualityMonitor plugin |
| Level 3 | 連續流分析（進階能力） | 未來 Phase 3 |

SafetyMonitor 的 liveness 和 VitakkaWatchdog 是品質監控的基礎設施——它們提供系統存活性和基本看門狗功能，但不提供品質度量。品質監控的起點是 Level 2（LoopQualityMonitor plugin 的事件驅動監控）。

---

## 8. Heartbeat = Liveness Probe（非品質監控）

心跳機制是「存活性探測」（liveness probe），不屬於 LoopQualityMonitor 的品質監控範疇。

### 形式承認

`setInterval(tick, 1000)` 在形式上滿足 polling 的定義——它的計算模型是週期性時間觸發。

```
Polling 的形式定義:
  ∃ timer ∈ PeriodicTimer: at_each_interval(timer, check_state)

setInterval(heartbeat, 1000):
  timer = PeriodicTimer(1000ms), check_state = heartbeat()
  ∴ 形式上滿足 polling 定義
```

### 語義區分

作業系統語義區分：

| 特徵 | Polling（輪詢） | Heartbeat（心跳） |
|------|-----------------|-------------------|
| 方向 | 觀察者 → 被觀察者 | 被觀察者 → 觀察者 |
| 目的 | 獲取目標狀態 | 證明自身存活 |
| 缺失時的意義 | 資訊落差 | 存活性問題 |
| 認知地位 | 品質監控的一部分 | 系統基礎設施 |

### 為何心跳不可或缺

事件驅動的 LoopQualityMonitor 有一個根本盲點：**它依賴事件的持續產生**。殭屍場景——`processEvent()` 拋出異常導致 `eventQueue` 內部狀態損壞，`dequeue()` 永久阻塞——此時沒有新事件被 emit，事件驅動的 LoopQualityMonitor 完全失明。

心跳填補了這個盲點：如果 `lastTickTimestamp` 超過預期間隔未更新，外部觀察者可判定系統已進入殭屍狀態。

[程式碼: packages/core/src/execution/loop.ts#run — TURING 殭屍狀態發現]

### 設計一致性

心跳作為 liveness infrastructure 與事件驅動監控不矛盾——心跳不在品質監控的語義範圍內。心跳負責「系統是否存活」，LoopQualityMonitor 負責「系統運作品質如何」。兩者職責正交。

---

## 9. 四階段降級策略

事件風暴或高負載下，LoopQualityMonitor 有控制地降級品質，而非崩潰。

| Stage | 觸發條件 | 行為 | 安全事件 | C/G 影響 |
|-------|---------|------|---------|----------|
| 0 Normal | 負載 < 80% | 全量處理，四維全計算 | 全量 | 正常 |
| 1 Throttle | 80%-95% | 丟棄低優先級事件 | 全量 | G 下降 |
| 2 Sample | 95%-99% | 取樣處理 (10-50%) | **永不丟棄** | C, G 顯著下降 |
| 3 Emergency Drop | > 99% | 只處理安全事件 + 審計日誌 | **永不丟棄** | 最低值 |

### 退化底線

D1-R3 最小事件集在任何 Stage 下都不被丟棄——`gear:arbiter_evaluated`, `gear:switch`, `action:proposed`, `action:executed` 四個事件是退化的絕對底線。SafetyMonitor 的可觀測性不受退化影響。

### 退化判斷機制

基於 LoopQualityMonitor 內部 buffer 佇列深度（異步 buffer + batch 設計的前提下）：

```typescript
// LoopQualityMonitor 內部退化判斷
const queueDepth = this.pendingEvents.length;
const capacity = this.config.maxQueueDepth;  // default: 100

if (queueDepth > capacity * 0.99) return 'emergency_drop';
if (queueDepth > capacity * 0.95) return 'sample';
if (queueDepth > capacity * 0.80) return 'throttle';
return 'normal';
```

### 退化通知

每次 Stage 切換發射 `loop:degradation` 事件：

```typescript
bus.emit({
  type: 'loop:degradation',
  payload: {
    stage: 'throttle' | 'sample' | 'emergency_drop',
    load: number,
    droppedEventTypes: string[],
    auditTrail: boolean  // Stage 3 啟用審計日誌
  }
});
```

### 退化性質

1. **安全事件永不丟棄** — SafetyMonitor 可觀測性不受影響
2. **可逆** — 負載下降時 Stage 自動回升，無需人工干預
3. **漸進** — Stage 0→1→2→3 逐階段過渡，不突然跳到最差情況
4. **有審計追蹤** — Stage 3 強制寫入審計日誌

退化策略本質上是一個**優雅降級控制器**（graceful degradation controller），穩定性由上述四項性質保證。

---

## 10. MRA（監控資源配置）

EVOI (Expected Value of Information) 在 OpenStarry 文件中使用 **MRA (Monitoring Resource Allocation)** 作為設計文件術語，更直接地描述其功能——監控資源的配置與分配。

### 命名分層

| 語境 | 術語 | 理由 |
|------|------|------|
| 數學公式、內部程式碼 | EVOI | 與學術文獻（Howard Raiffa, 1960s）一致 |
| OpenStarry 設計文件 | MRA (Monitoring Resource Allocation) | 描述功能而非預設價值框架 |

### 資源分配邏輯

MRA 在固定 Level 內配置 Depth（取樣率、事件粒度、分析深度）。它不切換 Level——Level 是部署固定的。

```
Depth(t) = argmax_d [ EVOI(d) - Cost(d) ]

其中：
  EVOI(d) = Σ_s P(risk=s|context) × [U(detect|d,s) - U(miss|d,s)]
  Cost(d) = computational_overhead(d)
```

Plan28 第一版使用**簡化版風險評分**代替完整 EVOI 計算：

- 工具呼叫 = 高風險 → Depth 升高（更多訂閱、更高取樣率）
- 純文字回覆 = 低風險 → Depth 降低（節約計算資源）

簡化版預留介面供未來替換為完整 EVOI/MRA 計算。

### 不受 MRA 調節的基礎設施

SafetyMonitor 和 VitakkaWatchdog **不受 MRA 策略調節**——它們的計算預算是固定分配的，不允許被「資源不足」為由削減。MRA 只適用於 Level 2+ 的 LoopQualityMonitor plugin 資源配置。

### 守護維度（Prevention）

MRA 不僅衡量「偵測的資訊價值」（detection），也衡量「預防損害的價值」（prevention）：

$$\text{MRA}_{total} = \text{MRA}_{detection} + \text{MRA}_{prevention}$$

$$\text{MRA}_{prevention} = P(\text{threat}) \times \text{Loss}(\text{undetected\_threat}) \times P(\text{detect} | \text{monitoring\_level})$$

Prevention 維度衡量的是：如果不進行監控，未偵測到的威脅可能造成多大損害。這使 MRA 的 Depth 調整不僅基於當前資訊需求，也基於潛在風險的預防價值。

---

## 11. 三層穩定化架構中的 LoopQualityMonitor

D4 決議確立三層穩定化架構，LoopQualityMonitor 是頂層：

```
┌─────────────────────────────────────┐
│ Layer 3: LoopQualityMonitor (event-driven)  │ ← 品質監控
│   偵測: 品質退化, 模式異常, 漂移     │
│   Level/Depth 可調                   │
├─────────────────────────────────────┤
│ Layer 2: VitakkaWatchdog            │ ← 粗粒度看門狗
│   偵測: 任意齒輪循環卡住 (N-Gear)    │
│   動作: emit event + force config.defaultGear │
├─────────────────────────────────────┤
│ Layer 1: SafetyMonitor              │ ← 硬性安全底線
│   偵測: 安全違規 + 殭屍狀態          │
│   動作: halt / inject prompt        │
│   + loopLivenessCheck [Cycle 02-4]  │
└─────────────────────────────────────┘
```

三層架構的 Lyapunov 穩定性保證：如果三層均正常運作，mano-karma 正回饋迴路有界。各層的職責嚴格分離：

| 性質 | 保證層 | 說明 |
|------|--------|------|
| 安全性 | SafetyMonitor (Layer 1) | 不可能產生不安全的決策 |
| 活性 | VitakkaWatchdog (Layer 2) | 不可能永遠不反省 |
| 品質 | LoopQualityMonitor (Layer 3) | 認知迴圈品質持續被監控 |
| 存活性 | SafetyMonitor.liveness | 殭屍狀態可被偵測 |

---

## 12. VitakkaWatchdog 定位

VitakkaWatchdog 是 Level 1 監控的工程實現——粗粒度的計數器型看門狗，提供最小基線的齒輪停滯偵測。

### 工作原理

純計數器型別：每次齒輪迴圈呼叫 `recordGearCycle(gear)` 累加對應齒輪計數，超過閾值時觸發。它**不是**「每 N 個迴圈檢查一次」的 interval polling，而是逐次計數的事件驅動行為。`[Plan27b 已完成: recordGear1Cycle() → recordGearCycle(gear: number)]`

```typescript
/**
 * VitakkaWatchdogConfig — 防止輪迴停滯
 *
 * KERNEL: 類似 RTOS 看門狗——主迴圈停滯時強制重置。
 * [Plan27b 已完成: N-Gear 泛化，per-gear 配置]
 */
interface VitakkaWatchdogConfig {
  /** 每齒輪最大連續循環數 (default: { 1: 10 }) */
  readonly maxConsecutiveGearCycles: Record<number, number>;
  /** 每齒輪最大時鐘時間 ms (default: { 1: 5000 }) */
  readonly maxGearDurationMs: Record<number, number>;
  /** 啟用狀態 (default: true) */
  readonly enabled: boolean;
}
```

> **Plan27b 已完成**: VitakkaWatchdog 的 API 已泛化為 N-Gear 支援：`recordGearCycle(gear: number)`、`forceNextGear(defaultGear)`、per-gear 配置（`maxConsecutiveGearCycles`、`maxGearDurationMs`）。看門狗偵測任意齒輪的停滯，而非僅限 Gear 1。

[程式碼: packages/core/src/execution/vitakka-watchdog.ts#recordGearCycle] `[Plan27b 已完成]`

### 行為選項（D2-2B 確認）

- **(b) emit `vitakka:stall` 事件** — 通知 LoopQualityMonitor 和其他訂閱者 `[Plan27b: vitakka:stall_detected → vitakka:stall]`
- **(a) force `config.defaultGear`**（預設 2，可配置）— 強制走 IProvider/LLM 路徑
- **(c) 重置 Klesha** — **永久排除** — Watchdog 偵測過程停滯，不開藥方

VitakkaWatchdog 的行為是 (b)+(a)：先 emit 事件通知，同時強制 `config.defaultGear`（預設 2，可配置）。(c) 被永久排除——VitakkaWatchdog 不應直接修改 Klesha，這違反了模組解耦原則和「行蘊→IUI only」決議。

[來源: Cycle 02-4 D2-2B, 20/20 全票通過]

---

## 13. SPC 暖機期（統計過程控制）

ATHENA 的 SPC (Statistical Process Control) 框架為 LoopQualityVector 提供精密的統計判斷。SPC 標記為 **Phase 2+ 可選功能**，不列入 Plan28 第一版的必須交付。

### 控制圖設計

對 C, G, S, E 四維度各建一張控制圖，使用改編的 Western Electric Rules：

| 規則 | 條件 | 意義 |
|------|------|------|
| Rule 1 | 單點低於 Action Limit | 嚴重異常——立即報告 |
| Rule 2 | 連續 3 點低於 LCL | 趨勢惡化——升級 Depth |
| Rule 3 | 連續 7 點同向移動 | 漂移——需要校準 |
| Rule 4 | CL 周圍高頻振盪 | 不穩定——檢查事件風暴 |

### 暖機策略

控制圖需要歷史資料來計算 Center Line (CL) 和 Control Limits。Agent 啟動時沒有歷史資料——不能立即啟用 SPC 判斷。

```typescript
const WARMUP_TICKS = 30;  // 預設 30 個認知 tick，可配置

interface LoopSPCState {
  readonly history: readonly LoopQualityVector[];
  readonly warmupComplete: boolean;
  readonly controlLimits?: {
    cl: LoopQualityVector;   // Center Line
    lcl: LoopQualityVector;  // Lower Control Limit (μ - 3σ)
    ucl: LoopQualityVector;  // Upper Control Limit (μ + 3σ)
  };
}
```

暖機期間使用**固定閾值**（configurable defaults）；暖機完成後切換到基於歷史資料的動態控制限。

PASCAL 的 Bayesian 詮釋：暖機不是「開關切換」，而是從先驗到後驗的連續過渡——隨觀測累積，控制限逐漸收斂到真實值。30 tick 的離散切換是可接受的工程簡化（未來可改為 Bayesian 連續過渡）。

### 退化期間的 SPC

Stage 2 和 Stage 3 退化下，SPC 資料品質不可靠（取樣率不足），SPC 判斷應暫停，使用硬性閾值代替。退化結束後需要短暫的重校準期（re-calibration period）。

[來源: Cycle 02-4 D2-R3]

---

## 14. 分階段實作

| Phase | Plan | 監控能力 | 說明 | 工作量 |
|-------|------|---------|------|--------|
| Phase 1 | Plan27a | LoopQualityVector SDK 類型定義 | SDK 類型預先定義 | ~50-70 LOC |
| Phase 1 | Plan27b | SafetyMonitor.liveness + VitakkaWatchdog 接線 | Level 1 + 存活性偵測 | ~45 LOC |
| Phase 2 | Plan28 | LoopQualityMonitor event-driven plugin + MRA 簡化版 + 退化策略 | Level 2 完整功能 | ~460-700 LOC |
| Phase 2+ | Plan29 ✅ | PluginHooks `monitors` 槽位 + MonitorRegistry + ILoopQualityMonitor SDK 型別 + DEFAULT_LOOP_QUALITY_WEIGHTS | Plugin 註冊基礎設施 (v0.29.0-alpha) | ~120 LOC |
| Phase 3 | Plan30+ | 連續流 + SPC + 完整 MRA + Level 動態切換預留 | Level 3 | 未估算 |

### Plan29 PluginHooks 擴展 (Cycle 02-5 D2-R3)

```typescript
// PluginHooks 擴展
interface PluginHooks {
  listeners?: IListener[];         // rupa (輸入)
  ui?: IUI[];                      // rupa (輸出)
  providers?: IProvider[];         // samjna
  tools?: ITool[];                 // samskara
  guides?: IGuide[];               // vijnana
  arbiters?: IGearArbiter[];       // samjna + vijnana (Cycle 02-4)
  monitors?: ILoopQualityMonitor[];       // vedana + samjna + vijnana (Cycle 02-5)
}

// ILoopQualityMonitor 介面 (Plan29 已實作)
export interface ILoopQualityMonitor {
  readonly id: string;
  start(bus: EventBus): void;
  stop(): void;
  getReport(): LoopQualityReport | null;
}
```

[來源: Cycle 02-5 D2-R3, 20/20 全票通過]

### Plan27 具體交付

| 工程項目 | 來源決議 | 估算 LOC |
|---------|---------|---------|
| SafetyMonitor.liveness (lastTickTimestamp + 殭屍偵測) | Cycle 02-4 D2-R1 | ~30 |
| ExecutionLoop crash recovery 修正 | Cycle 02-4 D2-R1(d) | ~15 |
| LoopQualityVector SDK 類型定義 | Cycle 02-4 D2-R3(a) | ~30-40 |
| LoopQualityReport SDK 類型定義 | Cycle 02-4 D2-R3(c) | ~20-30 |

### Plan28 具體交付

| 工程項目 | 來源決議 | 估算 LOC |
|---------|---------|---------|
| LoopQualityMonitor plugin 骨架 | Cycle 02-4 D2-R2(b) | 100-150 |
| LoopQualityVector 計算邏輯 | Cycle 02-4 D2-R3(a) | 80-120 |
| loop:quality_report EventBus 事件 | Cycle 02-4 D2-R3(c) | 20-30 |
| 退化策略 (4-stage + buffer) | Cycle 02-4 D2-R4 | 80-120 |
| MRA/EVOI Depth 調整引擎 (簡化版) | Cycle 02-4 D2-R2(c) | 50-80 |
| SPC 控制圖 (Phase 2+ 可選) | Cycle 02-4 D2-R3(b) | 100-150 |
| 測試 | — | 100-150 |

---

## 15. 跨文件參照

| 相關文件 | 關係 |
|---------|------|
| Doc 04 (Plugin Infrastructure) | PluginHooks `monitors` 槽位定義 (Plan29) |
| Doc 33 (Multivalue Skandha) | `inferSkandha()` 對 `monitors` 槽位的推斷邏輯 |
| Doc 35 (Dual Gear) | 齒輪切換事件是 LoopQualityMonitor 的主要監控對象 |
| Doc 36 (Vedana) | 受蘊事件是 LoopQualityMonitor 輸入；LoopQualityVector.E 需要 vedana 的正/負分類 |
| Doc 37 (Klesha) | 煩惱狀態是 LoopQualityMonitor 輸入；Klesha 活躍度影響 MRA 風險評分 |
| Doc 38 (IVolition) | IVolition 審議決策是 LoopQualityMonitor 輸入 |
| Doc 42 (IGearArbiter) | IGearArbiter 評估結果是 LoopQualityMonitor 輸入；D1-R3 最小事件集是退化底線 |
| Doc 44 (Safety) | 三層穩定架構：LoopQualityMonitor = Layer 3, SafetyMonitor = Layer 1 |

### 與其他辯論的關係

| 關聯辯論 | 決議的影響 |
|---------|-------------|
| Cycle 02-4 D1（IGearArbiter） | D1-R3 最小事件集是 LoopQualityMonitor 退化策略的底線 |
| Cycle 02-4 D3（觸/作意） | D3 的 EventBus 預留擴展點 → LoopQualityMonitor 可訂閱因果追溯事件 |
| Cycle 02-4 D4（行蘊流向） | D4 三層穩定架構中 LoopQualityMonitor 是頂層 |
| Cycle 02-4 D5（Klesha 閾值） | Klesha 動態閾值 → 高活躍度 = 高風險 → MRA Depth 升高 |
| Cycle 02-4 D6（受蘊工程） | LoopQualityVector.E 需要受蘊正面/負面分類資訊 |
| Cycle 02-5 D1（佛學映射邊界） | §15 佛學映射表移至附錄；佛學引用從正文 REMOVE/RELOCATE |
| Cycle 02-5 D2（蘊歸屬重分類） | skandha 從隱含的行蘊重新分類為 ['vedana', 'samjna', 'vijnana'] |

---

## 16. 未解決問題

| # | 問題 | 說明 | 關聯 |
|---|------|------|------|
| UQ-1 | LoopQualityMetric 介面設計 | LoopQualityMonitor 品質如何回饋至 Klesha 閾值調制？需要定義 LoopQualityMetric → 閾值公式的連接介面 | FC-32, Doc 37 §5 多因素閾值 |
| UQ-2 | LoopQualityMonitor 對五層閾值模型的影響 | LoopQualityVector 如何作為 Model Delta Layer 3 的輸入？監控強度是否影響齒輪信心度閾值？ | Doc 37 DC-12 多因素閾值公式 |
| UQ-3 | 跨 Agent 監控協調 | 多 Agent 場景中，一個 Agent 偵測到異常時如何通知其他 Agent 升級監控？ | MESH D2-R2 反對意見 |
| UQ-4 | SPC 控制限自適應 | 非穩態過程（Agent 行為隨時間演化）下，控制限如何適應？ | ATHENA SPC 框架 Phase 3 |
| UQ-5 | Level 動態切換的安全審計 | Phase 3 預留——動態載入 LoopQualityMonitor plugin 的安全機制設計 | D2-R2(d) Phase 3 預留 |

---

## 附錄 A：品質監控架構總覽圖

```
┌─────────────────────────────────────────────────────┐
│           認知迴路品質監控架構分層                       │
├─────────────────────────────────────────────────────┤
│                                                     │
│  ┌─────────────────────────────────────────────┐   │
│  │  Level 3: 連續流分析 (Phase 3 預留)          │   │
│  │  Depth 可調: 分析演算法複雜度, 回溯深度       │   │
│  └─────────────────────────────────────────────┘   │
│                                                     │
│  ┌─────────────────────────────────────────────┐   │
│  │  Level 2: LoopQualityMonitor Plugin (Plan28)        │   │
│  │  蘊歸屬: vedana + samjna + vijnana           │   │
│  │  能力: 事件驅動監控                           │   │
│  │  Depth 可調: 取樣率, 事件過濾, 分析窗口        │   │
│  │  品質: LoopQualityVector (C,G,S,E)           │   │
│  │  判斷: SPC 控制圖 (Phase 2+ 可選)            │   │
│  │  報告: loop:quality_report EventBus 事件      │   │
│  │  退化: 4-stage degradation                   │   │
│  │  資源: MRA 驅動的 Depth 調整                  │   │
│  │  註冊: PluginHooks.monitors (Plan29)         │   │
│  └─────────────────────────────────────────────┘   │
│                                                     │
│  ┌─────────────────────────────────────────────┐   │
│  │  Level 1: VitakkaWatchdog (Plan27b)          │   │
│  │  能力: 計數器監控                             │   │
│  │  Depth 可調: per-gear 閾值 (N-Gear)           │   │
│  │  行為: (b) emit event + (a) force config.defaultGear │   │
│  └─────────────────────────────────────────────┘   │
│                                                     │
│  ════════════════════════════════════════════════   │
│  品質監控起點 ↑ | 基礎設施 ↓                         │
│  ════════════════════════════════════════════════   │
│                                                     │
│  ┌─────────────────────────────────────────────┐   │
│  │  Layer 0: SafetyMonitor.liveness (Plan27b)   │   │
│  │  功能: 存活性探測 (heartbeat / liveness probe)│   │
│  │  預算: 固定分配, 不受 MRA 調節                 │   │
│  │  新增: lastTickTimestamp 殭屍偵測             │   │
│  └─────────────────────────────────────────────┘   │
│                                                     │
└─────────────────────────────────────────────────────┘
```

## 附錄 B：決議索引

### Cycle 02-4 D2 決議

| 編號 | 決議內容 | 票數 | 對應章節 |
|------|---------|------|---------|
| D2-2A | 事件驅動共識（Polling = 退化行為） | 20/20 | §1, §3 |
| D2-2B | VitakkaWatchdog (b)+(a)，(c) 永久排除 | 20/20 | §12 |
| D2-2C | (D)+(C) 品質屬性 + 漸進監控 | 20/20 | §3 |
| D2-R1 | SafetyMonitor.liveness 擴展 | 10/10 | §7 |
| D2-R2 | Level 固定 / Depth 動態（CPU ISA/Clock 類比） | 16/18 | §4 |
| D2-R3 | LoopQualityVector 四維 + SPC 可選 + EventBus 報告 | 19/19 | §5, §6, §13 |
| D2-R4 | 心跳 = liveness probe + 四階段退化策略 | 20/20 | §8, §9 |
| D2-R5 | EVOI → MRA 改名 | 17/18 | §10 |

### Cycle 02-5 決議

| 編號 | 決議內容 | 票數 | 對應章節 |
|------|---------|------|---------|
| D1-Q2-A (A-4) | 「event-driven = 正念」等映射 REMOVE | 20/20 | §1 |
| D1-Q2-B (B-2) | bhavanga-citta 映射 RELOCATE 至附錄 | 20/20 | §7 |
| D1-Q2-B (B-4) | 佛學映射表 RELOCATE 至 Appendix_A | 20/20 | (原 §15 移除) |
| D1-Q2-B (B-5) | LoopQualityVector 佛學映射註解 RELOCATE | 20/20 | §5 |
| D2-R1 | 標題修正為「認知迴路品質監控架構」 | 19/20 | §1 |
| D2-R2 | skandha: ['vedana', 'samjna', 'vijnana']，排除行蘊 | 18/20 | §1 |
| D2-R3 | PluginHooks 新增 `monitors?: ILoopQualityMonitor[]` (Plan29) | 20/20 | §14 |
| D4-R1 | ISatiMonitor → ILoopQualityMonitor 命名矯正（定義責任測試） | 13/20 | 全文件 |

---

---

## Cycle 02-6 Research Additions

`[Cycle 02-6 新增: D3 四項決議全票通過 (20/20)]`

以下為 Cycle 02-6 D3 辯論確立的 LoopQualityMonitor event-driven plugin 實體工程規格。

### 17.1 四維品質計算公式 [D3-R1, 20/20]

§5 定義了 LoopQualityVector 的四維語義，本節補充**具體計算公式**，全部基於 EventBus 事件的 O(1) 增量計算。

**參數**: W = 10（滑動窗口）, W_warmup = 5（暖機期最小樣本）, Q_default = 0.5

**coherence**: `1 - gearSwitchCount/(W-1)`
- 來源: `gear:switch` 事件。零切換 = 1.0

**efficiency**: `successfulExec / totalProposed`
- 來源: `action:proposed` + `tool:result`。無提案時 = 1.0

**convergence**: `longestConsistentStreak / W`
- 來源: `gear:switch` 事件。全窗口同齒輪 = 1.0

**stability**: `1 - min(1, σ²/0.25)`
- 來源: `gear:arbiter_evaluated`。Welford's online algorithm, O(1)/event

**複合品質分數**: Q = Σ(0.25 × d_i) ∈ [0, 1]

複雜度：每事件 O(1) 時間，O(W) 空間。

[來源: Cycle 02-6 D3-R1, 20/20]

### 17.2 EventBus 事件訂閱分階 [D3-R2, 20/20]

**MINIMAL_QUALITY_EVENTS（Phase 1，6 事件）**:

| 事件 | 用途 |
|------|------|
| `gear:arbiter_evaluated` | stability（信心度方差） |
| `gear:switch` | coherence + convergence |
| `action:proposed` | efficiency（分母） |
| `tool:result` | efficiency（分子） |
| `loop:started` | 生命週期（窗口初始化） |
| `loop:finished` | 生命週期（最終報告 + 清理） |

**EXTENDED_QUALITY_EVENTS（Phase 2，+5 事件）**: `sparsha:contact`, `tool:error`, `tool:blocked`, `vitakka:stall`, `action:executed`

降級: 缺失事件 → 安全預設值，不崩潰。

[來源: Cycle 02-6 D3-R2, 20/20]

### 17.3 extras 整合機制 [D3-R3, 20/20]

Monitor 透過 SDK `emitWithExtras()` 發射：
- `loopQuality:updated` — 攜帶 LoopQualityVector + composite Q
- `audit:context_contribute` — 攜帶 extras 供 L2 auditor

ManoAggregator 訂閱：
- `loopQuality:updated` → loopQualityFn（L3 閾值調制）
- `audit:context_contribute` → extras bag（L2 審計追蹤）

extras keys: `loopQuality:vector`（四維）, `loopQuality:composite`（標量 Q ∈ [0,1]）

生命週期: per-routing-cycle, one-tick delay OK。

[來源: Cycle 02-6 D3-R3, 20/20]

### 17.4 品質權重配置 [D3-R4, 20/20]

**Phase 1**: 固定等權重 0.25 × 4（maximum entropy）

**Phase 2**: 可配置，三種預設：

| 預設 | coherence | efficiency | convergence | stability |
|------|-----------|------------|-------------|-----------|
| balanced（預設） | 0.25 | 0.25 | 0.25 | 0.25 |
| stability-focused | 0.15 | 0.15 | 0.15 | **0.55** |
| efficiency-focused | 0.15 | **0.55** | 0.15 | 0.15 |

約束: Σw_i = 1, w_i ∈ [0,1]。驗證失敗回退至 balanced。

[來源: Cycle 02-6 D3-R4, 20/20]

---

## 附錄 C：Cycle 02-6 D3 決議索引

| 編號 | 決議內容 | 票數 | 對應章節 |
|------|---------|------|---------|
| D3-R1 | 四維品質計算公式 | 20/20 | §17.1 |
| D3-R2 | EventBus 事件訂閱分階 | 20/20 | §17.2 |
| D3-R3 | extras 整合機制 | 20/20 | §17.3 |
| D3-R4 | 品質權重配置 | 20/20 | §17.4 |

*Cycle 02-4 新增文件。Cycle 02-5 修訂：佛學映射清理（D1）+ 蘊歸屬重新分類（D2）+ PluginHooks monitors 槽位（D2-R3）+ 命名矯正（D4-R1）。Cycle 02-6 新增：D3 四維計算公式（§17.1）+ EventBus 事件訂閱分階（§17.2）+ extras 整合機制（§17.3）+ 品質權重配置（§17.4）。*
