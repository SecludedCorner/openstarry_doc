# 42. IGearArbiter 介面規格 (IGearArbiter Interface Specification)

`[Cycle 02-4 新增]` `[Plan27a 修整]` `[Cycle 02-6 新增: AuditContext 型別 + Model Delta Option C + WIENER 約束]`

**來源**: Pre-DC VasanaEngine 架構矛盾討論 + Cycle 02-4 D1 全部決議 (D1-1A~1D, D1-R1~R5) + D5 風險加權閾值 + D6 VasanaEngine 安全閘門
**日期**: 2026-03-04（初版）, 2026-03-05（Plan27a 修整）
**實作狀態**: ✅ SDK 型別 + Core 骨架已實作 (Plan27a, v0.27.0-alpha, 1611 tests)
**核心學者**: KERNEL (#10), GUARDIAN (#11), ATHENA (#5), BABBAGE (#9), TURING (#17), WIENER (#12), ASANGA (#8), NAGARJUNA (#7), ARCHIMEDES (#16), PASCAL (#19), DARWIN (#6)
**依賴文件**: #04 插件基礎設施, #29 五蘊連結, #35 雙齒輪意時鐘, #37 Klesha 增益排程, #38 IVolition 雙階段審議

---

## 一、概述 (Overview)

IGearArbiter 是 Cycle 02-4 引入的全新 plugin 介面，定義 N-Gear 齒輪選擇的評估能力。它取代了原本錯誤放置在核心中的 VasanaEngine 組件，使 OpenStarry 的微核心架構回歸「零內建能力」原則。`[Plan27a 修整: Gear 1 快速路徑 → N-Gear 齒輪選擇]`

### 核心定位

| 屬性 | 說明 |
|------|------|
| **性質** | Plugin 介面 (capability interface)，非核心機制 |
| **功能** | 為 ManoAggregator 提供 N-Gear 齒輪選擇的信心度評估 `[Plan27a: N-Gear 泛化]` |
| **取代** | VasanaEngine（原核心組件） — VasanaEngine 轉為 IGearArbiter 的第一個參考實作 plugin |
| **模式** | Chain of Responsibility — first-win, early termination |
| **退化** | 無 IGearArbiter plugin 安裝時，Agent 退化為 `config.defaultGear`（預設 Gear 2，永遠走 IProvider/LLM），仍完全可運行 `[Plan27a: defaultGear]` |

### 佛學基礎

VasanaEngine 外部化的佛學根據：**習氣 (vasana) 是後天薰習，不是先天結構**。

[STATED] 唯識學認為，習氣（vasana）是「種子」（bija）在阿賴耶識中經由經驗反覆薰習而形成的傾向。它不是與生俱來的固有結構，而是因緣和合的後天產物。

[INFERRED] 既然習氣是後天薰習的能力，不是先天的認知機制，那麼它在工程架構上應該是 plugin（可選、可替換、後天安裝），而非核心組件（必備、固定、先天存在）。VasanaEngine 僅是 IGearArbiter 介面的其中一種實作——其他開發者可以寫出完全不同的快速路徑評估 plugin。

> [來源: DC_vasana_engine_architectural_contradiction.md#§3]

---

## 二、設計背景 (Design Context)

### 2.1 Pre-DC 決議：VasanaEngine 架構矛盾

Master 在 Pre-Cycle 02-4 討論中識別了關鍵矛盾：

| 核心組件 | 性質 | 知識 | 語義判斷 |
|---------|------|------|---------|
| EventBus | 純路由 | 無 | 無 |
| ServiceRegistry | 純註冊 | 無 | 無 |
| PluginLoader | 純載入 | 無 | 無 |
| ExecutionLoop | 純排程 | 無 | 無 |
| **VasanaEngine (舊)** | **規則匹配** | **有** | **有** |

VasanaEngine 是唯一具備知識和語義判斷的核心組件——它有規則庫、匹配邏輯、信心度評分。這使它成為**能力** (capability) 而非**機制** (mechanism)，違反微核心的零內建能力原則。

> [來源: DC_vasana_engine_architectural_contradiction.md#§2]

### 2.2 解決方案

矛盾的根源在於齒輪切換機制和 VasanaEngine 能力被混為一體。分離方案：

- **核心保留**: ManoAggregator = 純路由機制 (if/else)，與 EventBus 同等性質
- **外部化**: VasanaEngine 作為 plugin，實作新介面 `IGearArbiter`
- **新增介面**: IGearArbiter — 任何 plugin 都可以實作，VasanaEngine 只是第一個

> [來源: DC_vasana_engine_architectural_contradiction.md#§3]

### 2.3 Master's Letter M-1 確認

Master's Letter 的 M-1（嚴格微核心原則）明確要求核心不得包含任何內建能力。IGearArbiter 的介面化設計正是此原則的落實。

### 2.4 D1 辯論確認

Cycle 02-4 Debate 1 全部九項決議（四項快速通過 + 五項辯論決議）確認了 IGearArbiter 的完整規格。方案 (B) 跨蘊雙層策略以 20/20 全票通過，無任何根本反對。

> [來源: D1_IGearArbiter_skandha_attribution.md#§第一節]

---

## 三、介面定義 (Interface Definition)

### 3.1 IGearArbiter 核心介面

```typescript
/**
 * IGearArbiter — N-Gear 齒輪選擇評估介面。
 *
 * Plugin 透過此介面提供齒輪選擇能力。ManoAggregator 按 priority 順序
 * 呼叫 IGearArbiter chain，第一個超過動態閾值的結果獲勝，
 * 否則退回 config.defaultGear（預設 Gear 2 / IProvider/LLM）。
 *
 * 蘊歸屬: Interface 層獨立（不繼承 ISamjna 或 IVijnana），
 *         Manifest 層 skandha: ['samjna', 'vijnana']
 * 契約: evaluate() 為觀測函數，禁止副作用
 * 時間預算: per-arbiter 100ms (configurable)
 *
 * @since Cycle 02-4 D1
 * @sealed 此介面的方法簽名不應擴展
 *
 * [Plan27a 修整]: evaluate() 允許同步或異步回傳，棄權用 action: 'abstain'
 */
export interface IGearArbiter {
  /** 仲裁者唯一識別碼 */
  readonly id: string;

  /** 評估優先級（數字越小越優先） */
  readonly priority: number;

  /**
   * 評估當前上下文，回傳信心度和齒輪選擇。
   *
   * 約束:
   * - 觀測函數: 可讀取外部狀態，禁止修改任何狀態
   * - 確定性: 相同輸入 + 相同外部狀態 → 相同輸出
   * - 不得自行發射事件: 核心在 evaluate() 完成後自動同步發射
   *   gear:arbiter_evaluated 事件
   * - 必須在 per-arbiter timeout 內完成（預設 100ms）
   * - 允許同步或異步回傳 — Core 用 Promise.resolve() 統一包裝
   *
   * @param context 當前認知上下文（CoarisingBundle + KleshaSignals + 歷史）
   * @returns 評估結果（信心度 + 齒輪選擇），棄權時 action: 'abstain'
   */
  evaluate(context: GearContext): GearEvaluation | Promise<GearEvaluation>;
}
```

> `[Plan27a 修整]`: evaluate() 簽名從 `Promise<GearEvaluation | null>` 改為 `GearEvaluation | Promise<GearEvaluation>`。棄權語義從 `null` 改為 `action: 'abstain'`。允許同步回傳（OQ-A5）。

> [來源: D1_IGearArbiter_skandha_attribution.md#§第六節, D1-R1]

### 3.2 為何是單一 evaluate() 方法

D1 辯論第二節的核心衝突：GUARDIAN 主張 `recognize()/resolve()` 兩步分離以允許中間驗證，ATHENA 主張 EventBus 同步事件提供等效安全保障。

**關鍵轉折**: KERNEL 指出 Node.js EventEmitter.emit() 是**同步**呼叫所有 listener 的。GUARDIAN 原本的「異步驗證來不及」論點在 Node.js 模型中不成立。

**佛學支持**: ASANGA 引用《成唯識論》卷三「五遍行心所與心同時起、同所依、同所緣、同行相」——想蘊的辨認和識蘊的了別是俱時的，不是序列的。單一 evaluate() 更忠實地映射五遍行俱生結構。

**最終決議**: 20/20 通過（GUARDIAN、LEIBNIZ 在附加條件後同意），保持單一 evaluate()，安全驗證通過三道外部機制實現（同步事件、閾值安全因子、最小事件集）。

> [來源: D1_IGearArbiter_skandha_attribution.md#§第二節, D1-R1]

---

## 四、相關型別定義 (Related Type Definitions)

> **Plan27a 修整 (2026-03-05)**: 本節全面修訂以反映 N-Gear 泛化、Tenet #7 合規修正、和工程實作對齊。
> 主要變更：GearAction 改為 `number | 'abstain'`；GearContext 移除 threshold；GearEvaluation 新增 riskCategory；
> 新增 RouteResult、RiskDeltaConfig、ManoAggregatorConfig 等型別。

### 4.1 基礎型別

```typescript
/**
 * GearAction — 齒輪選擇結果。
 *
 * 正整數表示齒輪編號（如 1=快速路徑, 2=LLM 路徑），
 * 'abstain' 表示此 arbiter 棄權（交由下一個 arbiter 或 defaultGear）。
 *
 * 設計決策 (OQ-A1): 齒輪選擇與行動載荷時序分離。
 * arbiter 只回答「走哪個齒輪」，行動載荷由後續步驟處理（Plan27b）。
 * 佛學依據：想蘊辨認（samjna）與行蘊執行（samskara）是不同心所活動。
 */
type GearAction = number | 'abstain';

/**
 * RiskCategory — 風險分類。
 *
 * 由 arbiter plugin 在 GearEvaluation 中宣告（Tenet #7: Core 不做語義推斷）。
 * SafetyMonitor 可在 postRouteCheck() 中覆寫/升級（Plan28）。
 */
type RiskCategory = 'destructive' | 'state_modifying' | 'read_only' | 'informational';

/**
 * AgentConfig — arbiter 可見的最小 Agent 配置。
 */
export interface AgentConfig {
  readonly id: string;
  readonly model?: string;
}

/**
 * GearToolCall — 提議中的工具調用資訊。
 */
export interface GearToolCall {
  /** 工具名稱 */
  readonly name: string;
  /** 工具參數 */
  readonly arguments: Record<string, unknown>;
}

/**
 * ActionRecord — 最近行動歷史記錄。
 */
export interface ActionRecord {
  readonly name: string;
  readonly success: boolean;
  readonly timestamp: number;
}
```

### 4.2 核心介面型別

```typescript
/**
 * GearContext — IGearArbiter 評估的輸入上下文。
 *
 * 包含當前認知狀態的只讀快照。
 * IGearArbiter plugin 可讀取但不得修改任何欄位。
 *
 * 注意：不含 threshold 欄位。閾值由 ManoAggregatorConfig 注入，
 * Core 在 ManoAggregator 內部進行比較——arbiter 不需要知道閾值。
 * （Tenet #7: 閾值計算是策略邏輯，不屬於 arbiter 的評估輸入。）
 */
export interface GearContext {
  /** 當前輸入文本 */
  readonly input: string;
  /** 提議中的工具調用（無則傳空陣列 []） */
  readonly proposedToolCalls: readonly GearToolCall[];
  /** 最近行動歷史 */
  readonly actionHistory: readonly ActionRecord[];
  /** Agent 配置 */
  readonly agentConfig: AgentConfig;
  /** 會話識別碼 */
  readonly sessionId?: string;
  /** 當前 CoarisingBundle（Plan27b 接線後可用） */
  readonly bundle?: CoarisingBundle;
  /** 受蘊評估快照（Plan27b 接線後可用） */
  readonly vedanaAssessment?: VedanaAssessment;
  /** Klesha 信號快照（Plan27b 接線後可用） */
  readonly kleshaSignals?: KleshaSignalBundle;
}

/**
 * GearEvaluation — IGearArbiter 的評估結果。
 *
 * action 為齒輪選擇（number）或棄權（'abstain'）。
 * Core 消費 confidence + action：超過風險加權閾值且 action 為有效齒輪號時路由，
 * 否則繼續評估下一個 arbiter 或 fallback 到 config.defaultGear。
 */
export interface GearEvaluation {
  /** 齒輪選擇：正整數（齒輪號）或 'abstain'（棄權） */
  readonly action: GearAction;
  /** 信心度 [0.0, 1.0] — Core 用於閾值比較 */
  readonly confidence: number;
  /** 風險分類（arbiter 宣告，可選）— 影響風險加權閾值 */
  readonly riskCategory?: RiskCategory;
  /** 仲裁者的推理說明（用於審計日誌和除錯） */
  readonly reasoning?: string;
}
```

### 4.3 ManoAggregator 配置與結果型別

```typescript
/**
 * RiskDeltaConfig — 各風險等級的閾值增量。
 *
 * 正值提高閾值（更嚴格），負值降低閾值（更寬鬆）。
 * SDK 提供 DEFAULT_RISK_DELTA，使用者可透過 ManoAggregatorConfig 覆寫。
 */
export interface RiskDeltaConfig {
  readonly destructive: number;       // 預設 +0.20
  readonly state_modifying: number;   // 預設 +0.10
  readonly read_only: number;         // 預設  0.00
  readonly informational: number;     // 預設 -0.10
}

/**
 * ManoAggregatorConfig — ManoAggregator 的完整配置。
 *
 * 所有策略值由此 config 注入，Core 不硬編碼任何值（Tenet #7）。
 * SDK 提供 DEFAULT_MANO_AGGREGATOR_CONFIG 作為預設值。
 */
export interface ManoAggregatorConfig {
  /** 單一 arbiter 評估超時（毫秒），預設 100 */
  readonly perArbiterMs: number;
  /** 整條 chain 的截止時間（毫秒），預設 200 */
  readonly chainMs: number;
  /** 各齒輪的信心度上限，預設 { 1: 0.95 } */
  readonly maxConfidenceByGear: Record<number, number>;
  /** 預設齒輪（無 arbiter 或全部棄權時使用），預設 2 */
  readonly defaultGear: number;
  /** 基礎閾值 θ_base，預設 0.6 */
  readonly baseThreshold: number;
  /** 風險增量配置 */
  readonly riskDelta: RiskDeltaConfig;
  /** 閾值下限 — 不可低於此值（防止永遠使用 IGearArbiter），預設 0.3 */
  readonly thresholdFloor: number;
  /** 閾值上限 — 不可高於此值（防止永不使用 IGearArbiter），預設 0.9 */
  readonly thresholdCeiling: number;
}

/**
 * RouteResult — ManoAggregator.route() 的回傳結果。
 *
 * 記錄路由決策的完整資訊，供事件發佈和審計使用。
 */
export interface RouteResult {
  /** 最終選擇的齒輪號 */
  readonly gear: number;
  /** 決定此結果的 arbiter ID（若所有 arbiter 棄權則為 undefined） */
  readonly decidedBy?: string;
  /** 決定性 arbiter 的信心度（若無則為 0） */
  readonly confidence: number;
  /** 是否經過風險加權閾值調整 */
  readonly riskAdjusted: boolean;
}
```

### 4.4 SDK 常量

```typescript
/** 預設風險增量 — SDK 提供，Core 不硬編碼 */
const DEFAULT_RISK_DELTA: RiskDeltaConfig = {
  destructive: 0.20,
  state_modifying: 0.10,
  read_only: 0.00,
  informational: -0.10,
};

/** 預設 ManoAggregator 配置 */
const DEFAULT_MANO_AGGREGATOR_CONFIG: ManoAggregatorConfig = {
  perArbiterMs: 100,
  chainMs: 200,
  maxConfidenceByGear: { 1: 0.95 },
  defaultGear: 2,
  baseThreshold: 0.6,
  riskDelta: DEFAULT_RISK_DELTA,
  thresholdFloor: 0.3,
  thresholdCeiling: 0.9,
};
```

> [來源: D1_IGearArbiter_skandha_attribution.md#§第六節; Plan27a implementation_summary §1.1; OQ-A1~A5 決議]

---

## 五、Chain of Responsibility 語義 (Chain Semantics)

### 5.1 語義規則

D1 共識項 1-C (20/20 全票通過) 確立了 Chain of Responsibility 的完整語義：

| 規則 | 說明 |
|------|------|
| **First-win** | 第一個 confidence > adjustedThreshold 且 action 為有效齒輪號的 arbiter 勝出 |
| **Early termination** | 獲勝後不繼續評估後續 arbiter |
| **Deterministic ordering** | 按 priority 數值升序排列（數字越小越先評估） |
| **Same priority** | 按註冊順序 (FIFO) |
| **Abstain** | arbiter 回傳 `action: 'abstain'` → 棄權，繼續下一個 |
| **All fail** | 所有 arbiter 均未超過閾值或棄權 → `config.defaultGear`（預設 2） |
| **No arbiters** | 無 IGearArbiter plugin 安裝 → 永遠 `config.defaultGear` |

> [來源: D1_IGearArbiter_skandha_attribution.md#§第一節, 共識項 1-C]

### 5.2 流程圖

```
processEvent()
  │
  ▼
ManoAggregator receives GearContext
  │
  ├── Klesha.perceive() → KleshaSignalBundle          ← vijnana-clock (~0.03ms)
  │     → config.baseThreshold（由 Klesha 增益排程調整）
  │
  ▼
Sort IGearArbiter chain by priority (ascending)
  │
  ▼
┌──────────────────────────────────────────────────────────────────┐
│  For each arbiter (in priority order):                          │
│                                                                  │
│  ┌─ Promise.resolve(arbiter.evaluate(context)) ──────────┐     │
│  │  ← per-arbiter timeout: config.perArbiterMs (100ms)   │     │
│  │  ← timeout → treat as abstain, continue next          │     │
│  └────────────────────────────────────────────────────────┘     │
│      │                                                          │
│      ▼                                                          │
│  emit gear:arbiter_evaluated (SYNCHRONOUS, MANDATORY)           │
│      │ → SafetyMonitor / LoopQualityMonitor 可在此同步驗證        │
│      │ → 事件在路由決策之前發射 (GUARDIAN 附加條件)                │
│      │                                                          │
│      ▼                                                          │
│  evaluation.action === 'abstain' ? → YES → continue next        │
│      │ NO                                                       │
│      ▼                                                          │
│  adjustedThreshold = computeAdjustedThreshold(                  │
│    config.baseThreshold, evaluation.riskCategory, config.riskDelta, │
│    config.thresholdFloor, config.thresholdCeiling)                 │
│      │                                                          │
│      ▼                                                          │
│  evaluation.confidence > adjustedThreshold ?                    │
│      │                                                          │
│      ├── YES → Winner! route to gear = evaluation.action        │
│      │         → apply maxConfidenceByGear cap                  │
│      │         → STOP (early termination)                       │
│      │         → emit gear:switch { gear: N, decidedBy }        │
│      │                                                          │
│      └── NO  → Continue to next arbiter                         │
│                                                                  │
│  ← chain timeout: config.chainMs (200ms)                        │
│  ← chain timeout reached → break, fallback to defaultGear      │
└──────────────────────────────────────────────────────────────────┘
  │
  ▼ (no winner found)
config.defaultGear (預設 Gear 2): IProvider.chat(context) → LLM inference
  │
  ▼
emit gear:switch { gear: config.defaultGear }
```

### 5.3 ManoAggregator 虛擬碼

> **Plan27a 修整**: 虛擬碼更新以反映工程實作。主要變更：config 注入取代 context.threshold；
> N-Gear routing（action 為齒輪號）；abstain 處理；風險加權閾值由 SDK utility 計算；
> RouteResult 回傳型別。

```typescript
// packages/core/src/mano/mano-aggregator.ts (概念)
// ManoAggregator = 純路由機制，同 EventBus 性質
// 無狀態、無智能邏輯、無領域知識、策略值全部從 config 注入

function createManoAggregator(bus: EventBus, config: ManoAggregatorConfig) {

  async function route(
    context: GearContext,
    arbiters: readonly IGearArbiter[]
  ): Promise<RouteResult> {

    // 空 arbiter → 直接 defaultGear
    if (arbiters.length === 0) {
      return { gear: config.defaultGear, confidence: 0, riskAdjusted: false };
    }

    const chainDeadline = Date.now() + config.chainMs;  // 200ms
    const sorted = [...arbiters].sort((a, b) => a.priority - b.priority);

    for (const arbiter of sorted) {
      if (Date.now() > chainDeadline) break;  // chain timeout

      // Per-arbiter timeout + sync/async 統一包裝
      const evaluation = await Promise.race([
        Promise.resolve(arbiter.evaluate(context)),
        timeout(config.perArbiterMs).then(() => null),
      ]);

      // 超時或異常 → 視為棄權
      if (!evaluation || evaluation.action === 'abstain') {
        bus.emit({
          type: 'gear:arbiter_evaluated',
          timestamp: Date.now(),
          payload: { arbiterId: arbiter.id, confidence: 0, passed: false },
        });
        continue;
      }

      // 風險加權閾值計算（SDK utility，五參數全從 config 注入）
      const adjustedThreshold = evaluation.riskCategory
        ? computeAdjustedThreshold(
            config.baseThreshold, evaluation.riskCategory, config.riskDelta,
            config.thresholdFloor, config.thresholdCeiling
          )
        : config.baseThreshold;

      // Per-gear confidence cap
      const gear = evaluation.action as number;
      const cap = config.maxConfidenceByGear[gear];
      const confidence = cap !== undefined
        ? Math.min(evaluation.confidence, cap)
        : evaluation.confidence;

      const passed = confidence > adjustedThreshold;

      // MANDATORY: 同步發射事件 — 在路由決策之前 (D1-R1 附加條件)
      bus.emit({
        type: 'gear:arbiter_evaluated',
        timestamp: Date.now(),
        payload: {
          arbiterId: arbiter.id,
          confidence,
          adjustedThreshold,
          passed,
          riskCategory: evaluation.riskCategory,
        },
      });

      // First-win: 超過風險加權閾值即勝出
      if (passed) {
        bus.emit({
          type: 'gear:switch',
          timestamp: Date.now(),
          payload: { gear, decidedBy: arbiter.id, confidence },
        });
        return {
          gear,
          decidedBy: arbiter.id,
          confidence,
          riskAdjusted: !!evaluation.riskCategory,
        };
      }
    }

    // No winner → config.defaultGear
    bus.emit({
      type: 'gear:switch',
      timestamp: Date.now(),
      payload: { gear: config.defaultGear, confidence: 0 },
    });
    return { gear: config.defaultGear, confidence: 0, riskAdjusted: false };
  }

  return { route };
}
```

> [來源: D1_IGearArbiter_skandha_attribution.md#§第二節; Plan27a mano-aggregator.ts 實作]

---

## 六、evaluate() 純度契約 (Purity Contract)

D1-R4 (19/20 通過，PASCAL 棄權於數值部分) 確立了 evaluate() 的完整契約。

### 6.1 觀測性純粹 (Observational Purity)

```
若 evaluate 為觀測函數（observational）:
  evaluate 可讀取外部狀態但不修改
  ∀ c1 = c2, 若外部狀態相同: evaluate(c1) = evaluate(c2)
```

| 操作 | 允許？ | 說明 |
|------|--------|------|
| 讀取規則庫 | **允許** | VasanaEngine 需要讀取已沉積的規則 |
| 讀取歷史 context | **允許** | 評估可能需要參考近期互動 |
| 讀取 Klesha 狀態 | **允許** | 信心度計算可能參考煩惱狀態 |
| 修改規則庫 | **禁止** | 學習（imprint）必須在獨立函數中完成 |
| 更新計數器 | **禁止** | 統計追蹤由核心層面處理 |
| 發射 EventBus 事件 | **禁止** | 事件由核心在 evaluate() 完成後自動發射 |
| 呼叫外部服務/API | **禁止** | 評估必須基於本地狀態完成 |

**核心設計原則**: evaluate() 只**觀察**，不**改變**。修改狀態的操作（如規則學習的 `imprint()`）應在單獨的方法中完成，由核心在明確的時機呼叫——而非在 evaluate 的副作用中偷偷進行。

> [來源: D1_IGearArbiter_skandha_attribution.md#§第四節, BABBAGE 形式化分析]

### 6.2 事件發射責任

**重點**: evaluate() 不得自行發射事件。事件發射是**核心的責任**。

```
IGearArbiter plugin:  evaluate() → 回傳 GearEvaluation
                      (不發射任何事件)

核心 (ManoAggregator): 收到 GearEvaluation
                      → 同步 emit gear:arbiter_evaluated
                      → SafetyMonitor / LoopQualityMonitor 在此同步驗證
                      → 才做路由決策 (if confidence > adjustedThreshold)
```

GUARDIAN 的附加條件: `gear:arbiter_evaluated` 事件**不可被延遲或跳過**。這是安全審計的最低要求。

> [來源: D1_IGearArbiter_skandha_attribution.md#§第二節, GUARDIAN 附加條件]

### 6.3 雙層超時機制 (Dual-Layer Timeout)

| 層級 | 配置欄位 | 預設值 | 說明 | 超時行為 |
|------|---------|--------|------|---------|
| Per-arbiter | `config.perArbiterMs` | 100ms | 單一 arbiter 的最大評估時間 | 視為棄權，跳過此 arbiter |
| Total chain | `config.chainMs` | 200ms | 整個 chain 的最大總評估時間 | 直接 fallback 到 `config.defaultGear` |

兩個數值均可通過 `ManoAggregatorConfig` 覆寫——不同部署場景的延遲容忍度不同。PASCAL 保留對數值的校準權，但不阻止框架通過。

> **Plan27a 修整**: 原 `ManoAggregatorTimeoutConfig` 已被 `ManoAggregatorConfig` 完整取代（§四 4.3）。
> 超時配置是 `ManoAggregatorConfig` 的一部分，與 baseThreshold、riskDelta、maxConfidenceByGear、defaultGear 統一注入。

> [來源: D1_IGearArbiter_skandha_attribution.md#§第四節, D1-R4; Plan27a §1.1 ManoAggregatorConfig]

---

## 七、蘊歸屬 (Skandha Attribution)

### 7.1 方案 (B) 跨蘊雙層策略

D1 共識項 1-A (20/20 全票通過) 確認了 IGearArbiter 的蘊歸屬方案：

| 層級 | 策略 | 說明 |
|------|------|------|
| **Interface 層** | 獨立介面 | IGearArbiter 不繼承 ISamjna 也不繼承 IVijnana |
| **Manifest 層** | 雙蘊標注 | `PluginManifest.skandha: ['samjna', 'vijnana']` |

### 7.2 為何不能雙繼承

BABBAGE 在 R04 的型別代數分析中證明了 TypeScript 判別式衝突：

```typescript
// ISamjna 和 IVijnana 各有判別式欄位
interface ISamjna { readonly skandha: 'samjna'; /* ... */ }
interface IVijnana { readonly skandha: 'vijnana'; /* ... */ }

// 如果 IGearArbiter 同時繼承兩者：
interface IGearArbiter extends ISamjna, IVijnana {
  // skandha: 'samjna' & 'vijnana' = never
  // TypeScript 判別式衝突 — 此介面不可實例化
}
```

`'samjna' & 'vijnana' = never` — 型別系統層面就不允許同時繼承。因此：

- **Interface 層使用獨立介面**，不帶 `skandha` 判別式
- **Manifest 層使用多值 `skandha` 陣列**，利用 DC-2 已確認的多值 skandha 機制

### 7.3 佛學語義

ASANGA 在 D1 中的說明：「佛學組原本傾向方案 (A) [IGearArbiter extends ISamjna]，但交叉審閱後接受 R04 的型別代數論證。想蘊確實是主要職責，但識蘊始終在場——方案 (B) 的 Manifest 雙標注精確表達了這一事實。」

IGearArbiter 的認知活動涉及兩個蘊：
- **想蘊 (samjna)**: 辨認——「我認出了這個模式」
- **識蘊 (vijnana)**: 判斷依據——「我對這個辨認結果的信心度是 0.85」

> [來源: D1_IGearArbiter_skandha_attribution.md#§第一節, 共識項 1-A]

### 7.4 型別守衛 (Type Guard)

D1-R2 (20/20 全票通過) 確認了 `isGearArbiter()` 結構型別守衛：

```typescript
/**
 * isGearArbiter — IGearArbiter 結構型別守衛。
 *
 * 因方案 (B) 下 IGearArbiter 不帶 skandha 判別式，
 * isSkandha(arbiter, 'samjna') 返回 false。
 * 必須使用結構型別檢查。
 *
 * TURING 增強: evaluate.length <= 1 提供微妙的簽名驗證。
 */
export function isGearArbiter(obj: unknown): obj is IGearArbiter {
  if (typeof obj !== 'object' || obj === null) return false;
  const candidate = obj as Record<string, unknown>;
  return (
    typeof candidate.id === 'string' &&
    typeof candidate.priority === 'number' &&
    typeof candidate.evaluate === 'function' &&
    (candidate.evaluate as Function).length <= 1  // evaluate 接受最多 1 個參數
  );
}
```

此方案採用 TURING 的增強版，與既有程式碼風格一致（v0.26.0-beta 中的 `isVedanaSensor()`, `isProvider()` 等均為結構型別檢查）。

> **Plan27a 修整**: `isGearArbiter()` 定位為 **SDK utility**（位於 `sdk/src/types/gear-arbiter.ts`），
> 供 plugin 開發者在測試和驗證中使用。它**不是** Core PluginLoader 的發現機制——
> plugin 發現透過顯式 `PluginHooks.gearArbiters` 宣告（見 §十二）。
> Core 不呼叫此函數。

**被拒替代方案**:
- G-2 (Registry 查詢)：4 票。需修改 registry 資料結構。
- G-3 (Symbol brand)：2 票。形式上更嚴格，但與既有風格不一致。

> [來源: D1_IGearArbiter_skandha_attribution.md#§第三節, D1-R2; Plan27a engineering_notes §五 P27-J 修正]

---

## 八、G-0 ~ G-4 退化表 (Degradation Table)

D1-R5 (20/20 全票通過) 確認了完整的退化行為表：

| 等級 | IGearArbiter 配置 | IProvider 配置 | 路由行為 | 說明 |
|------|-------------------|---------------|---------|------|
| **G-0** | 無 | 無 | 核心存活但無法認知 | Phase 2.5 跳過，Phase 3 失敗 |
| **G-1** | 無 | 有 | 永遠 `config.defaultGear` | **與 v0.26.0-beta 行為完全一致** — 向後完全相容 |
| **G-2** | 單一 arbiter | 無 | 僅 arbiter 推薦的齒輪 | 僅能處理 arbiter 匹配到的場景，無 fallback |
| **G-3** | 單一 / 多個 arbiter | 有 | N-Gear routing | 先查 arbiter chain，過閾值走推薦齒輪，否則走 `config.defaultGear` |
| **G-4** | 多個 arbiter + hot-swap | 有 | 運行時動態載入/卸載 | 遠期目標 (Phase 3+)，需 plugin 生命週期管理 |

> **Plan27a 修整 (OQ-A3)**: 退化表描述**能力深度**，不是齒輪數量。
> G-1 的「永遠 Gear 2」改為「永遠 `config.defaultGear`」（預設 2，可配置）。
> G-3 從「完整雙齒輪」改為「N-Gear routing」（支援任意齒輪數）。
> 佛學映射不變——映射描述認知深度，與齒輪數量無關。

### 關鍵保證

**G-1 向後完全相容**: 不安裝任何 IGearArbiter plugin 時，ManoAggregator 的 Phase 2.5 是空操作 (noop)，直接進入 `config.defaultGear` 路徑（預設為 Gear 2 = IProvider/LLM）。行為與 v0.26.0-beta **完全一致**。這是 Plan27a 的關鍵驗收標準。

**零內建能力原則**: G-0/G-1 證明 Agent 不需要任何 IGearArbiter 即可運行。IGearArbiter 提供的是能力（有規則庫、匹配邏輯、信心度評分），不是機制——因此必須是 plugin 而非核心組件。

### 佛學映射

NAGARJUNA 在 D1 中確認退化行為表與唯識學的自洽性：

| 退化等級 | 佛學映射 |
|----------|---------|
| G-0 | 無識無想的空殼——無根本認知能力 |
| G-1 | 初生眾生——無習氣，每件事都要從頭認知 |
| G-2 | 純條件反射——只有習氣驅動的模式匹配，缺乏深層意識 |
| G-3 | 有經驗的眾生——常見情境用直覺，陌生情境深思（凡夫認知標準模式） |
| G-4 | 多層習氣——不同領域、不同深度的經驗累積（種子分位） |

> [來源: D1_IGearArbiter_skandha_attribution.md#§第五節, NAGARJUNA 佛學確認]

---

## 九、ManoAggregator 整合 (Integration with ManoAggregator)

### 9.1 ManoAggregator 的性質

ManoAggregator = **純路由機制**，性質同 EventBus。

| 屬性 | ManoAggregator | EventBus |
|------|---------------|----------|
| 狀態 | 無 | 無 |
| 智能邏輯 | 無 | 無 |
| 領域知識 | 無 | 無 |
| 功能 | if/else 路由 | 事件分發 |

ManoAggregator 不決定任何事——它只根據 IGearArbiter chain 的回報數值做簡單的閾值比較。

### 9.2 ExecutionLoop 中的位置

ManoAggregator 作為 **Phase 2.5** 插入 ExecutionLoop：

```
Phase 1: Assemble context (現有)
Phase 2: Safety check — SafetyMonitor.isSafe() (現有)
───────────────────────────────────────────────────
Phase 2.5: ManoAggregator routing (新增)
  ├── Gear N (非 defaultGear) → 對應齒輪處理路徑
  │            → 進入 Phase 3.5 (IVolition Position B)
  └── config.defaultGear → 繼續走現有 Phase 3
───────────────────────────────────────────────────
Phase 3: Call LLM — Gear 2 path (現有)
Phase 3.5: IVolition Position B (現有)
Phase 4: Execute tools (現有)
```

### 9.3 ExecutionLoopDeps 整合

遵循 v0.26.0-beta 中 `volition` 欄位的內聯結構型別模式，ManoAggregator 的 DI 注入同樣使用內聯型別：

```typescript
// loop.ts — ExecutionLoopDeps 新增
export interface ExecutionLoopDeps {
  // ... 現有依賴 ...
  manoAggregator?: {
    route(context: GearContext): Promise<RouteResult>;
  };
}
```

保持 loop.ts 的低耦合特性——只依賴最小行為契約，不依賴具體模組的完整型別。

### 9.4 完整決策流程

> **Plan27a 修整**: 決策流程更新以反映 config 注入和 N-Gear 路由。
> `computeAdjustedThreshold()` 為 SDK utility，三參數全從 config 注入。
> `maxConfidenceByGear` 取代 `MAX_GEAR1_CONFIDENCE`（per-gear cap）。

```
GearContext 到達 ManoAggregator
  │
  ├── 1. config.baseThreshold 由 Klesha 增益排程計算
  │       （動態閾值注入，ManoAggregator 不參與計算）
  │
  ├── 2. 運行 IGearArbiter chain (Chain of Responsibility)
  │       → first-win, early termination
  │       → per-arbiter config.perArbiterMs, chain config.chainMs
  │
  ├── 3a. Winner found (action = gear number, 非 'abstain'):
  │       adjustedThreshold = computeAdjustedThreshold(
  │           config.baseThreshold, evaluation.riskCategory, config.riskDelta)
  │       effectiveConfidence = min(confidence, config.maxConfidenceByGear[gear])
  │       → effectiveConfidence > adjustedThreshold → route to gear N
  │       → else → continue chain
  │
  ├── 3b. 'abstain' → 跳過此 arbiter，繼續下一個
  │
  └── 3c. No winner → config.defaultGear（預設 Gear 2 = IProvider.chat()）
```

**回傳 RouteResult**: `{ gear, decidedBy?, confidence, riskAdjusted }`

> [來源: D1_IGearArbiter_skandha_attribution.md#§第五節; Plan27a mano-aggregator.ts]

### 9.5 VedanaEmergency 雙輸入調變 `[Plan28 新增]`

Plan28 為 ManoAggregator 新增 VedanaEmergency 佈線，實現 Model Delta Layer 4 的閾值調變：

```typescript
// ManoAggregator 接收 vedanaFn callback + config
function createManoAggregator(
  bus: EventBus,
  config: ManoAggregatorConfig,
  vedanaFn?: (assessment: VedanaAssessment) => number,  // Plan28 新增
  vedanaEmergencyConfig?: VedanaEmergencyConfig,         // Plan28 新增
) {
  async function route(context: GearContext, arbiters: readonly IGearArbiter[]): Promise<RouteResult> {
    // ... arbiter chain evaluation ...

    // VedanaEmergency: 計算 thresholdBoost
    const thresholdBoost = vedanaFn
      ? vedanaFn(context.vedanaAssessment)
      : 0;

    // effectiveBaseThreshold = base + thresholdBoost
    const effectiveBaseThreshold = config.baseThreshold + thresholdBoost;

    // 後續閾值比較使用 effectiveBaseThreshold
    // ...
  }
}
```

**公式**: `effectiveBaseThreshold = config.baseThreshold + thresholdBoost`

其中 `thresholdBoost` 由 `vedanaFn` 純函數計算：
- 輸入: `VedanaAssessment`（當前受蘊評估）
- 輸出: `number`（閾值提升量，[0, maxThresholdBoost]）
- 觸發條件: 連續 `sustainedTicks` 個 tick 的 intensity > `intensityThreshold`
- 冷卻: 觸發後 `cooldownTicks` 個 tick 內不再重複觸發

**VedanaEmergencyConfig DEFAULT**:

| 參數 | 預設值 | 說明 |
|------|--------|------|
| `intensityThreshold` | 0.8 | 受蘊強度觸發閾值 |
| `sustainedTicks` | 5 | 連續超過閾值的 tick 數 |
| `maxThresholdBoost` | 0.15 | 最大閾值提升量 |
| `cooldownTicks` | 10 | 冷卻期（tick 數）|

> VedanaEmergency 為純函數，不維護內部狀態——狀態由 ManoAggregator 管理。這保持了 vedanaFn 的可測試性和 Tenet #7 合規（Core 不做語義推斷，vedanaFn 由 config 注入）。

> [來源: Plan28 implementation_summary W3; D5-R6 Model Delta Layer 4]

---

## 十、風險加權閾值整合 (Risk-Weighted Threshold Integration)

D5-R2 (16/20 通過) 確立了風險加權閾值作為主要機制，取代全域信心度上限。

### 10.1 公式

$$\theta_{\text{adjusted}}(a) = \text{clamp}\left(\theta_0 + \Delta_{\text{klesha}}(t) + \Delta_{\text{risk}}(a), \; \theta_{\text{floor}}, \; \theta_{\text{ceiling}}\right)$$

其中：
- $\theta_0$ = 原始基礎閾值（靜態，default 0.6）
- $\Delta_{\text{klesha}}(t)$ = Klesha gain-scheduled 增量（動態，由四煩惱 Beta 分佈驅動）[詳見 Doc 37]
- $\Delta_{\text{risk}}(a)$ = 行動的風險增量（靜態函數）
- $\theta_{\text{floor}}$ = 閾值下限（config 注入，default 0.3）— 防止閾值過低而永遠使用 IGearArbiter
- $\theta_{\text{ceiling}}$ = 閾值上限（config 注入，default 0.9）— 防止閾值過高而永不使用 IGearArbiter

### 10.2 風險增量表

> **Plan27a 修整**: 風險增量值從 Core 硬編碼改為 SDK 提供的 `DEFAULT_RISK_DELTA`（§四 4.4）。
> 使用者可透過 `ManoAggregatorConfig.riskDelta` 覆寫。Core 不包含任何策略值。
> `riskCategory` 由 arbiter plugin 在 `GearEvaluation` 中宣告（OQ-A4），Core 不做語義推斷。

| 風險等級 (RiskCategory) | SDK DEFAULT Δ_risk | 說明 |
|------------------------|-------------------|------|
| `'destructive'` | +0.20 | 刪除、格式化等不可逆操作 |
| `'state_modifying'` | +0.10 | 修改配置、寫入資料庫等有副作用操作 |
| `'read_only'` | 0.00 | 讀取操作，無副作用 |
| `'informational'` | -0.10 | 純資訊回覆，風險最低 |

### 10.3 穩定性約束

WIENER 的穩定性分析：

> **Δ_risk 必須是動作類別的靜態函數，不得依賴系統歷史狀態。** 如需動態化，必須附帶穩定性證明。

若 Δ_risk 依賴歷史狀態，會引入回饋迴路，可能導致極限環（limit cycle）——系統在兩個狀態之間震盪，永遠不收斂。保持 Δ_risk 為無記憶函數 (memoryless) 可消除此風險。

> [來源: D5_klesha_threshold_vijnana.md#§第三節, WIENER 穩定性分析]

### 10.4 maxConfidenceByGear（Per-Gear 信心度上限）

> **Plan27a 修整 (OQ-A2)**: `maxGear1Confidence` 泛化為 `maxConfidenceByGear: Record<number, number>`，
> 支援 N-Gear——任意齒輪均可配置獨立的信心度上限。

D5-R3 (16/20 通過) 確立信心度上限為可配置的部署安全參數。N-Gear 泛化後：

```typescript
/**
 * maxConfidenceByGear — Per-Gear 信心度上限。
 *
 * 即使 IGearArbiter 回報 confidence = 1.0，
 * 核心也會將有效信心度限制在對應齒輪的上限以下。
 * 這是防止過度自信的安全閥。
 *
 * 預設: { 1: 0.95 }
 * 未列出的齒輪無上限限制。
 * 生產環境可調低（更保守），開發環境可設為 1.0（無限制）。
 */
readonly maxConfidenceByGear: Record<number, number>;
// DEFAULT_MANO_AGGREGATOR_CONFIG 中: { 1: 0.95 }
```

**重要**: IGearArbiter plugin **不需要知道** maxConfidenceByGear 的存在。Plugin 只回報原始信心度和齒輪選擇——核心負責應用風險加權閾值和 per-gear 信心度上限。這保持了 IGearArbiter 介面的簡潔性。

> **Plan28 修整**: riskCategory 傳播路徑已在 Plan28 完成佈線：arbiter → `GearEvaluation.riskCategory` → `RouteResult.riskCategory`。SafetyMonitor.postRouteCheck() 可在此路徑上覆寫/升級風險等級。

> [來源: D5_klesha_threshold_vijnana.md#§第三節, D5-R2/R3; Plan27a N-Gear 泛化]

---

## 十一、安全約束 (Security Constraints)

### 11.1 三層安全架構

```
Layer 0 (硬閘): SafetyMonitor — 所有動作（Gear 1 和 Gear 2）必經
                                 不可妥協的安全底線
Layer 1 (軟影響): Klesha + 風險加權閾值 — 情境性調節
                                         受煩惱狀態調制
Layer 2 (審議): IVolition — Gear 1/2 雙路徑均必經
                            （按風險等級區分強制/可選）
```

**安全性單調性** (BABBAGE): 無論 Klesha 調制如何影響閾值，Gear 1 的 GearEvaluation 仍必須通過 SafetyMonitor.postRouteCheck() 硬檢查。Klesha 調制無法繞過 Layer 0。

### 11.2 Gear 1 最小事件集

D1-R3 (20/20 全票通過) 確立了 Gear 1 路徑的最小可觀測事件集：

| 事件 | 發射時機 | 核心保證 | 用途 |
|------|---------|---------|------|
| `gear:arbiter_evaluated` | evaluate() 完成後 | 強制同步發射 | SafetyMonitor 同步預檢 |
| `gear:switch` | ManoAggregator route() 完成路由決策 | 強制同步發射，payload 含 `{ gear: number, decidedBy?, confidence }` | LoopQualityMonitor 知道系統選了哪個齒輪 |
| `action:proposed` | Gear 1 GearAction 構建完成 | 強制同步發射 | LoopQualityMonitor/IVolition 可審查提議的動作 |
| `action:executed` | 動作實際執行完成 | 強制同步發射 | 審計追蹤 |

**不可妥協**: 這四個事件在核心層面被強制發射，IGearArbiter plugin **無法繞過**。它們構成 WIENER 定義的最小可觀測集——少了任何一個，外部觀察者（LoopQualityMonitor/SafetyMonitor）都無法完整推斷系統狀態。

> Gear 2 路徑額外發射 `llm:started`、`llm:response`。

### 11.3 IVolition 風險分級審議 (D5)

Gear 1 動作經過 IVolition Position B 的審議，按風險等級區分：

| 風險等級 | IVolition 審議 | 說明 |
|---------|---------------|------|
| destructive | **必經** (MUST) | 不可逆操作，即使信心度極高也必須審議 |
| state_modifying | **必經** (MUST) | 有副作用操作 |
| read_only | **可選** (MAY) | 部署策略可配置是否跳過 |
| informational | **可選** (MAY) | 風險最低，SafetyMonitor 保護已足夠 |

> [來源: Doc 38 §十七, D5 決議]

---

## 十二、Plugin 開發指南 (Plugin Development Guide)

### 12.1 最小實作

> **Plan27a 修整**: Plugin 範例全面更新以反映 N-Gear 型別、`action: 'abstain'` 棄權語義、
> `riskCategory` 宣告、和顯式 `PluginHooks.gearArbiters` 宣告。

```typescript
import type {
  IGearArbiter,
  GearContext,
  GearEvaluation,
  RiskCategory,
  PluginFactory,
  inferRiskCategory,  // SDK utility（可選使用）
} from '@openstarry/sdk';

/**
 * 最簡 IGearArbiter plugin 範例。
 *
 * 此 plugin 使用規則表做快速模式匹配。
 * evaluate() 為觀測函數——只讀規則表，不修改任何狀態。
 * evaluate() 可為同步或異步（Core 以 Promise.resolve() 統一包裝）。
 */
const myArbiterPlugin: PluginFactory = (ctx) => ({
  hooks: {
    // 顯式宣告 gearArbiters（Tenet #2: plugin 自行宣告能力）
    gearArbiters: [
      {
        id: 'my-rule-matcher',
        priority: 100,

        // 同步 evaluate()（簡單規則匹配不需 async overhead）
        evaluate(context: GearContext): GearEvaluation {
          // 1. 模式匹配 — 讀取規則表（只讀，無副作用）
          const match = matchRules(context.input, myRules);

          // 2. 無匹配 → 棄權
          if (!match) {
            return { action: 'abstain', confidence: 0 };
          }

          // 3. 有匹配 → 回報齒輪選擇、信心度、風險分類
          //    核心負責閾值比較和風險加權，plugin 不需要知道閾值
          return {
            action: 1,  // 推薦 Gear 1（快速路徑）
            confidence: match.confidence,
            riskCategory: inferRiskCategory(match.toolName),  // SDK utility
            reasoning: `Matched rule: ${match.ruleId}`,
          };
        },
      },
    ],
  },
  // Plugin manifest — 宣告蘊歸屬
  manifest: {
    name: 'my-arbiter',
    skandha: ['samjna', 'vijnana'],  // 方案 (B) Manifest 雙標注
    // ...
  },
});

export default myArbiterPlugin;
```

### 12.2 開發者契約

Plugin 開發者必須遵守以下契約：

| 契約 | 說明 | 違反後果 |
|------|------|---------|
| evaluate() 為觀測函數 | 不得修改任何外部狀態 | 核心無法保證安全審計的正確性 |
| evaluate() 不發射事件 | 事件由核心發射 | 可能導致 SafetyMonitor 驗證時序錯亂 |
| `config.perArbiterMs` 內完成 | 預設 100ms，超時視為棄權 | 不影響系統，但 plugin 被跳過 |
| Manifest 宣告 skandha | `['samjna', 'vijnana']` | 五蘊完備性檢查可能報警 |
| confidence 範圍 [0, 1] | 超出範圍行為未定義 | 核心可能 clamp 或拒絕 |
| action 為正整數或 `'abstain'` | 其他值行為未定義 | 核心可能拒絕 |
| riskCategory 為 RiskCategory 型別 | 若未提供，Core 使用 baseThreshold（無風險加權） | 無安全問題，但略過風險調整 |

### 12.3 PluginManifest 範例

```json
{
  "name": "my-arbiter-plugin",
  "version": "1.0.0",
  "skandha": ["samjna", "vijnana"],
  "hooks": {
    "gearArbiters": [
      {
        "id": "my-rule-matcher",
        "priority": 100
      }
    ]
  }
}
```

### 12.4 PluginLoader 整合

IGearArbiter 的 plugin 載入透過顯式 `PluginHooks.gearArbiters` 宣告（Tenet #2 + #7: plugin 自行宣告能力，Core 不做 skandha 推斷）。

> **Plan27a 修整**: 原設計（P27-J）為 PluginLoader 按 skandha 自動偵測。此為 Tenet #2 + #7 違規——
> 「哪些 skandha 值映射到 gear arbiter」是領域知識。修正為顯式 hook 宣告。
> `isGearArbiter()` 是 SDK utility（供 plugin 開發者在測試中使用），不是 Core 發現機制。

```typescript
// plugin-loader.ts — load() 新增
if (plugin.hooks?.gearArbiters) {
  for (const arbiter of plugin.hooks.gearArbiters) {
    gearArbiterRegistry.register(arbiter);
  }
}
```

> [來源: Doc 04 (Plugin Infrastructure); Plan27a engineering_notes §五 P27-J 修正]

### 12.5 DeliberationContext 使用範例 `[Plan28 新增]`

Plan28 新增 `DeliberationContext` 型別，供 IVolition plugin 在審議時讀取路由結果和行動歷史：

```typescript
/**
 * DeliberationContext — IVolition 審議上下文。
 *
 * 提供路由決策結果和行動歷史，使 IVolition plugin 能做出
 * 風險感知的審議決策。
 *
 * @since Plan28 (v0.28.0-alpha)
 */
export interface DeliberationContext {
  /** ManoAggregator 路由結果（含齒輪號、決定者、信心度、風險類別） */
  readonly routeResult: RouteResult;
  /** 最近行動歷史（供重複偵測、模式分析） */
  readonly actionHistory: readonly ActionRecord[];
}
```

**Plugin 使用範例**:

```typescript
// volition-rule-engine plugin
async deliberateAction(input: ActionDeliberationInput): Promise<ActionDeliberationResult> {
  const { deliberationContext } = input;

  if (deliberationContext) {
    // 讀取路由結果做風險決策
    const { riskCategory } = deliberationContext.routeResult;
    if (riskCategory === 'destructive') {
      // 破壞性動作：檢查行動歷史是否有連續破壞性模式
      const recentDestructive = deliberationContext.actionHistory
        .filter(a => a.name.startsWith('delete') || a.name.startsWith('remove'));
      if (recentDestructive.length > 3) {
        return { veto: true, alternative: null, reasoning: 'Too many consecutive destructive actions' };
      }
    }
  }

  // ... 繼續其他規則評估 ...
}
```

> DeliberationContext 為 optional field（向後相容）。無 Plan28 佈線的環境中，deliberationContext 為 undefined。

> [來源: Plan28 W1 SDK Type Foundations; OQ-B4 回答]

---

## 十三、VasanaEngine 作為參考實作 (VasanaEngine as Reference Implementation)

### 13.1 定位

VasanaEngine 是 IGearArbiter 的**第一個**也是**參考**實作。它不是唯一的——任何開發者都可以寫出自己的 IGearArbiter plugin。

> **Plan27a 修整 (OQ-A6)**: N-Gear 泛化後，VasanaEngine 可推薦**任意齒輪**（不僅限 Gear 1）。
> 其三閘門保護 `imprint()`（學習方法），與齒輪選擇無關——無論推薦哪個 gear，
> 規則庫的安全性由三閘門保障。三閘門設計不變（Plan29+ 範疇）。

| 屬性 | VasanaEngine |
|------|-------------|
| 匹配策略 | Few-shot exemplar matching |
| 規則類型 | exact, regex, intent, tool_retry |
| 蘊歸屬 | Manifest: `['samjna', 'vijnana']` |
| 齒輪推薦 | 可推薦任意齒輪號（N-Gear 泛化），不僅限 Gear 1 |
| 學習機制 | `imprint()` — 從成功結果沉積新規則（延後至 Plan29+） |

### 13.2 三閘門安全 (D6 決議)

D6 第四節確立了 VasanaEngine `imprint()` 方法的三道安全閘門：

| 閘門 | 時機 | 功能 |
|------|------|------|
| Gate 1: 安全分類器 | imprint() 入口 | 拒絕包含破壞性動作模板的規則 |
| Gate 2: 啟動閾值 (N=3/5/20 依風險等級) | 規則激活前 | 規則必須被成功匹配 N 次才能激活：informational N=3, read_only N=5, state_modifying N=20（防止少量惡意輸入下毒）|
| Gate 3: Bayesian 影子驗證 | 規則使用中 | 長期追蹤規則的成功/失敗率，退化表現的規則自動降權 |

### 13.3 imprint() 與 evaluate() 的時序分離

evaluate() 和 imprint() 在不同的時機執行：

```
mano-clock tick N:
  evaluate() 讀取規則表 → 回報信心度 (觀測函數)
  (此處不學習)

mano-clock tick N+K (工具執行完成後):
  VedanaAssessment 回饋到達
  → imprint() 在獨立的學習窗口中執行
  → 經過三閘門安全檢查
  → 若通過，沉積新規則到規則表
```

這個時序分離保證了 evaluate() 的觀測純粹性——它在評估時讀取的規則表不會被自己的評估結果修改。

> [來源: D6_vedana_engineering_OQ.md#§第四節, VasanaEngine 安全閘門]

---

## 十四、跨文件參照 (Cross-Document References)

| 相關文件 | 關係 |
|---------|------|
| Doc 04 (Plugin Infrastructure) | Plugin 註冊機制——IGearArbiter 遵循相同的 PluginLoader 載入路徑 |
| Doc 29 (Five Aggregates Core Connection) | 五蘊整合——IGearArbiter 的蘊歸屬在此框架中的位置 |
| Doc 30 (Sparsha-Coarising Model) | CoarisingBundle 結構——GearContext 中 bundle 欄位的來源 |
| Doc 33 (Multivalue Skandha Design) | 多值 skandha 機制——方案 (B) Manifest 雙標注的基礎 |
| Doc 35 (Dual Gear Mano Clock) | 架構上下文——N-Gear 齒輪模型（原雙齒輪），ManoAggregator 定位 |
| Doc 36 (Vedana Measurement Model) | VedanaAssessment——GearContext 中 vedanaAssessment 欄位的來源 |
| Doc 37 (Klesha Gain Scheduling) | 閾值提供者——θ(t) 計算公式和四煩惱 Beta 分佈 |
| Doc 38 (IVolition Deliberation Pattern) | 後評估檢查——Position B 雙階段審議（風險分級條件化） |
| Doc 40 (Tenet 6 Architecture Mapping) | 四層迴路——IGearArbiter 在 Layer 2 歸依意中的位置 |
| Doc 41 (Design Decisions and Rationale) | 設計決策——VasanaEngine 外部化的完整理由鏈 |

---

## 十五、未解決問題 (Unresolved Questions)

| 編號 | 問題 | 說明 | 預計時程 |
|------|------|------|---------|
| UQ-1 | IGearArbiter hot-swap 機制 | 運行時動態載入/卸載 arbiter plugin，需要 plugin 生命週期管理 | Phase 3+ (G-4) |
| UQ-2 | 跨 arbiter 狀態共享 | 多個 IGearArbiter 若需共享上下文（如共用規則庫），目前無機制 | 需求明確後設計 |
| UQ-3 | VasanaEngine 學習協定 | imprint() 的完整協定 + 三閘門安全實作 | Plan29+ |
| UQ-4 | Chain 長度 > 5 的效能分析 | 200ms chain timeout 在多 arbiter 場景下是否足夠 | Plan27b 量測後校準 |
| UQ-5 | IGearArbiter 與 DC-11 判斷點 1 的整合 | 識蘊的齒輪選擇覆寫（即使 arbiter 匹配成功，識蘊仍可升級到 Gear 2）細節待設計 | 未來 Cycle |
| UQ-6 | N-Gear 語義定義 | 齒輪號 3+ 的具體語義待定義。目前 Gear 1=快速規則路徑、Gear 2=LLM 路徑是唯一確定語義。N-Gear 框架已就緒（`GearAction = number`），但齒輪 3+ 的含義需 plugin 生態和使用案例驅動。 | Plan29+ |

---

## 附錄 A: D1 辯論決議索引

| 編號 | 決議 | 票數 | 本文對應章節 |
|------|------|------|-------------|
| D1-1A | 方案 (B) 跨蘊雙層策略 | 20/20 | §七 |
| D1-1B | ISamjna D-3 不修改（不分裂 fast/slow 子介面） | 20/20 | — |
| D1-1C | Chain of Responsibility 語義 | 20/20 | §五 |
| D1-1D | VasanaEngine 外部化 | 20/20 | §二 |
| D1-R1 | evaluate() 單方法 + EventBus 同步事件 + 閾值安全因子 | 20/20 | §三, §六 |
| D1-R2 | isGearArbiter() 結構型別守衛 | 20/20 | §七 |
| D1-R3 | Gear 1 最小事件集 | 20/20 | §十一 |
| D1-R4 | evaluate() 純度契約 + 雙層時間預算 | 19/20 | §六 |
| D1-R5 | ManoAggregator 純路由化方案 | 20/20 | §八, §九 |

## 附錄 B: 相關 D5/D6 決議索引

| 編號 | 決議 | 票數 | 本文對應章節 |
|------|------|------|-------------|
| D5-R2 | 風險加權閾值為主要機制 | 16/20 | §十 |
| D5-R3 | maxConfidenceByGear 可配置部署安全參數（原 MAX_GEAR1_CONFIDENCE，N-Gear 泛化） | 16/20 | §十 |
| D5-R5 | Gear 1 IVolition 按風險等級區分 | — | §十一 |
| D6 §4 | VasanaEngine 三閘門安全 | — | §十三 |

## 附錄 C: Plan27 工程影響摘要

> **Plan27a 修整**: 更新實際 LOC 和測試數據（v0.27.0-alpha, 1611 tests）。

### Plan27a 實際數據（✅ 已完成）

| 工作項 | 估算 LOC | 實際 LOC | 說明 |
|--------|---------|---------|------|
| SDK `gear-arbiter.ts` 新建 | 60-80 | ~228 | 全部型別 + SDK utility (isGearArbiter, computeAdjustedThreshold, inferRiskCategory) |
| Core `gear-arbiter-registry.ts` | 50-80 | ~55 | register/get/list/listSorted/remove |
| Core `mano-aggregator.ts` | 150-200 | ~120 | createManoAggregator() 純路由 |
| Core `klesha-threshold.ts` | — | ~16 | 純 re-export（Tenet #7: Core 不含策略值） |
| SDK vedana.ts 修改 | — | +3 exports | VedanaDimension, toVedanaDimension, validateVedanaConfig |
| SDK coarising.ts 修改 | — | +3 exports | SparshEvent.timestamp, ManasikaraDimension, fromChannelManasikara |
| SDK plugin.ts 修改 | — | +1 field | PluginHooks.gearArbiters |
| Core plugin-loader.ts 修改 | 30-50 | ~10 | gearArbiters hook 註冊 |
| Core agent-core.ts 修改 | 40-60 | ~15 | Registry + ManoAggregator DI |
| Core loop.ts 修改 | — | ~5 | ExecutionLoopDeps 型別預備 |
| 新測試 | — | +59 tests | 1552 → 1611 全部通過 |

### Plan27b 實際數據（✅ 已完成）

| 工作項 | 實際 LOC | 說明 |
|--------|---------|------|
| VitakkaWatchdog N-Gear 泛化 | ~90 | recordGearCycle(gear): boolean + per-gear state |
| ManoAggregator 擴展 (forceNextGear + baseThresholdFn) | ~30 | 一次性 override + 動態閾值 callback |
| GearContext Construction Helper | ~35 | gear-context-builder.ts (pure function) |
| SparshEvent Construction | ~20 | sparsh-event-builder.ts (Object.freeze) |
| ExecutionLoop Phase 2.5 + Events | ~45 | Phase 2.5 + action:proposed/executed + sparsha:contact |
| AgentCore DI Wiring | ~25 | manoAggregator + gearArbiterRegistry + watchdog bus |
| StaticRuleArbiter Plugin | ~80 | @openstarry-plugin/gear-arbiter-static (10 tests) |
| 新測試 | +43 tests | 1611 → 1654 全部通過 |

> [來源: D1_IGearArbiter_skandha_attribution.md#§第六節; Plan27a/27b implementation]

---

*IGearArbiter 介面規格完成。本文件為 Cycle 02-4 的核心新增架構文件，涵蓋介面定義、Chain of Responsibility 語義、evaluate() 純度契約、蘊歸屬方案 (B)、退化行為表 G-0~G-4、風險加權閾值整合、安全約束、Plugin 開發指南、以及 VasanaEngine 參考實作。所有決議均經 D1 辯論表決確認，方案 (B) 以 20/20 全票通過。*

*跨文件參照: Doc 04 (插件基礎設施), Doc 29 (五蘊連結), Doc 35 (雙齒輪意時鐘), Doc 37 (Klesha 增益排程), Doc 38 (IVolition 雙階段審議)*

---

## Spec Addendum SA-42-001: 'abstain' Sentinel Design Rationale

`[Plan27b DOC SOP, 2026-03-05]`

### 背景

Plan27a 將 evaluate() 的棄權語義從 `null` 改為 `action: 'abstain'`（§三 3.1 + §四 4.1）。此 Addendum 記錄設計決策與實作影響。

### 設計決策

| 方面 | 原設計 | Plan27a 修正 | 理由 |
|------|--------|-------------|------|
| 棄權回傳 | `null` | `{ action: 'abstain', confidence: 0 }` | 結構化回傳 — 可攜帶 reasoning，便於審計日誌 |
| 型別 | `GearEvaluation \| null` | `GearEvaluation`（action 為 `number \| 'abstain'`） | 消除 nullable，簡化消費端程式碼（無需 null check） |
| 同步/異步 | `Promise<GearEvaluation \| null>` | `GearEvaluation \| Promise<GearEvaluation>` | 允許同步回傳（OQ-A5），Core 以 Promise.resolve() 統一包裝 |

### 影響範圍

1. **IGearArbiter.evaluate() 簽名** (§三 3.1): `GearEvaluation | Promise<GearEvaluation>`
2. **GearAction 型別** (§四 4.1): `number | 'abstain'`
3. **ManoAggregator route()** (§五 5.3): 檢查 `evaluation.action === 'abstain'` 而非 `evaluation === null`
4. **StaticRuleArbiter** (Plan27b P27-V): 無匹配時回傳 `{ action: 'abstain', confidence: 0, reasoning: 'No static rule matched' }`
5. **gear:arbiter_evaluated 事件**: abstain 時 payload `{ confidence: 0, passed: false }`

### Tenet 合規

- **Tenet #1 (名相)**: `'abstain'` 語義明確，優於 `null`（null 可能表示錯誤或未完成）
- **Tenet #7 (微核心)**: Core 不解釋 'abstain' 的原因，只執行 skip 邏輯

### 佛學補充

`'abstain'` 對應行蘊中的「不作意」（amanasikāra）——有意識的不介入，非錯誤亦非遺漏。這與 `null`（無法區分「不介入」和「出錯/未完成」）在語義上有根本差異。

### 向後相容

此為介面凍結前的設計修正（Plan27a 首次實作），無向後相容問題。

`[研究團隊確認: SA-42-001 接受。2026-03-05]`

---

## Cycle 02-6 Research Additions

`[Cycle 02-6 新增: D2 辯論五項決議]`

以下內容來自 Cycle 02-6 D2 辯論決議，定義了 AuditContext 型別、Model Delta 五層整合方案 Option C、以及 WIENER 回饋迴路約束。

### AuditContext 型別定義 [D2-R1, 20/20]

IConfidenceAuditor.audit() 的輸入參數從 RouteResult 擴展為完整的 AuditContext：

```typescript
export interface AuditContext {
  readonly version: 1;
  readonly sparshEvent: SparshEvent;
  readonly gearEvaluation: GearEvaluation;
  readonly riskCategory: RiskCategory | undefined;
  readonly routeResult: RouteResult;
  readonly historicalConfidence?: readonly number[];
  readonly extras: ReadonlyMap<string, unknown>;
}
```

設計要點：
- 核心欄位由 ManoAggregator 組裝，每次路由必有
- historicalConfidence 僅含原始 arbiter 信心度（WIENER C-1）
- extras 由 Plugin 透過 EventBus 貢獻，Core 不感知語義
- 完整規格見 **Doc 46** (AuditContext & Extras Protocol)

### Model Delta 五層整合 — Option C [D2-R4, 20/20]

```
θ_base + L1(Klesha) + L4(VedanaEmergency) = θ_intermediate
θ_adjusted = max(thresholdFloor, θ_intermediate × (1 - α × q))     ← L3
confidence_adjusted = confidence + clampAuditDelta(audit.delta)      ← L2
routing = (confidence_adjusted > θ_adjusted) ? arbiter_gear : default
```

- L2 (IConfidenceAuditor) → confidence，±0.05 clamp
- L3 (LoopQualityMonitor) → threshold，α=0.10
- 兩通道獨立，無交叉項，BIBO 穩定
- Layer 順序: L4(VedanaEmergency) → L3(LoopQuality) → 比較
- 多 monitor: 簡單平均; Monitor 過期: 5000ms → q=0

### WIENER 回饋迴路約束 [D2-R1/R3, 20/20]

| 約束 | 規則 | 防止的路徑 |
|------|------|-----------|
| C-1 | historicalConfidence 僅含原始 arbiter 信心度 | auditor → history → auditor |
| C-2 | AuditContext 不含 previousAuditResult | auditor → context → auditor |
| C-3 | extras key 禁止 `audit:` 前綴 | auditor → extras → auditor |

### ConfidenceAuditLog [D2-R3, 20/20]

```typescript
interface ConfidenceAuditLog {
  readonly inputConfidence: number;
  readonly rawDelta: number;
  readonly clampedDelta: number;
  readonly wasClamped: boolean;
  readonly reasoning: string;        // 截斷 500 chars
  readonly outputConfidence: number;
  readonly result: 'adjusted' | 'unchanged' | 'error';
  readonly auditDurationMs: number;
}
```

主通道: EventBus `audit:completed`。**GUARDIAN D5 義務正式兌現。**

### ManoAggregatorConfig 新增欄位

| 欄位 | 型別 | 預設值 | 決議 |
|------|------|--------|------|
| historicalConfidenceSize | number | 10 | D2-R1 |
| loopQualityAlpha | number | 0.10 | D2-R4 |
| monitorStalenessMs | number | 5000 | D2-R4 |

### Auditor 策略方向 [D2-R5, 20/20]

| Phase | 內容 | 時程 |
|-------|------|------|
| Phase 0 | PassthroughAuditor (delta=0, 純日誌) | Plan30 (可選) |
| Phase 1 | ThresholdAuditor (規則式) | Plan31 |
| Phase 2 | LLM-assisted | 長期 |

All phases must comply with WIENER C-1/C-2/C-3.

### 跨文件參照更新

| 相關文件 | 關係 |
|---------|------|
| Doc 43 §17.1-17.4 | LoopQualityMonitor 計算公式 + extras 整合 |
| Doc 44 §12.1-12.3 | Safety Architecture — Layer 2/3 整合 |
| Doc 46 | AuditContext & Extras Protocol 完整規格 |

### Cycle 02-6 D2 決議索引

| 編號 | 決議內容 | 票數 |
|------|---------|------|
| D2-R1 | AuditContext 完整型別 | 20/20 |
| D2-R2 | extras bag 協議 | 19/20 |
| D2-R3 | ConfidenceAuditLog | 20/20 |
| D2-R4 | Layer 整合方案 C | 20/20 |
| D2-R5 | Auditor 策略 | 20/20 |

---

*本文件涵蓋 Cycle 02-4 IGearArbiter 介面設計、Plan27a/27b 修整、Plan28 IVolition 整合、Plan29 IConfidenceAuditor 佈線、及 Cycle 02-6 AuditContext 型別 + Model Delta Option C + WIENER 約束。*
