# 45. 五蘊 OOP 架構 — Five Skandha Object-Oriented Architecture

`[Cycle 02-5 新增]`

**來源**: Cycle 02-5 R1 獨立研究 (A1-A5) + D3 辯論 (6 項決議, 20 位學者)
**日期**: 2026-03-06
**工程基線**: v0.28.0-alpha
**核心學者**: LINNAEUS (#13), VITRUVIUS (#3), DARWIN (#6), HERACLITUS (#15), LEIBNIZ (#14), KERNEL (#10), TURING (#17), ASANGA (#8), WIENER (#12), BABBAGE (#9), ATHENA (#5), PENROSE (#18), GUARDIAN (#11), ARCHIMEDES (#16), PASCAL (#19)
**依賴文件**: #04 插件基礎設施, #29 五蘊連結, #35 雙齒輪意時鐘, #37 Klesha 增益排程, #38 IVolition 雙階段審議, #42 IGearArbiter 介面規格, #44 安全架構總覽

---

## 一、概述 (Overview)

本文件是 Cycle 02-5 的首要交付物，回答 Master 提出的核心架構問題：

> 「我們需要將五蘊有哪一些子類別，如何依賴注入，如何在 agent core 運行先處理好。架構先用好。agent core 中如何將五蘊 plugin 載入。然後 plugin 要如何做。agent core 中五蘊會如何流轉。是我們的首要任務。」

文件結構對應 Master 的四個問題：

| 問題 | 對應章節 |
|------|---------|
| 五蘊有哪些子類別？ | 二、五蘊型別階層 (Type Hierarchy) |
| 如何依賴注入？ | 三、依賴注入佈線 (DI Wiring) |
| 如何將五蘊 plugin 載入？ | 四、Plugin 載入流程 (Plugin Loading) |
| 五蘊會如何流轉？ | 五、五蘊執行流 (Execution Flow) |

跨蘊互動分析與架構完備性評估見第六、七章。

### 核心程式碼入口

| 檔案 | 角色 |
|------|------|
| `packages/sdk/src/types/aggregates.ts` | 五蘊根介面 + Discriminated Union + type guard |
| `packages/sdk/src/types/plugin.ts` | PluginManifest + PluginHooks + IPluginContext |
| `packages/core/src/agents/agent-core.ts` | Composition Root (createAgentCore) |
| `packages/core/src/infrastructure/plugin-loader.ts` | 拓撲排序 + Hook 註冊 |
| `packages/core/src/execution/loop.ts` | ExecutionLoop FSM (六狀態) |

---

## 二、五蘊型別階層 (Type Hierarchy)

### 2.1 根介面 (Root Interfaces)

五蘊型別系統建立在五個根介面之上，以 `readonly skandha` 屬性作為 Discriminated Union 的辨識器：

```typescript
// 五蘊 Discriminated Union Type
export type Skandha = 'rupa' | 'vedana' | 'samjna' | 'samskara' | 'vijnana';

export interface IRupa {
  readonly skandha: 'rupa';
}

export interface IVedana {
  readonly skandha: 'vedana';
}

export interface ISamjna {
  readonly skandha: 'samjna';
}

export interface ISamskara {
  readonly skandha: 'samskara';
}

export interface IVijnana {
  readonly skandha: 'vijnana';
}
```
[程式碼: packages/sdk/src/types/aggregates.ts#Skandha, IRupa, IVedana, ISamjna, ISamskara, IVijnana]

兩個 type guard 函式提供運行時辨識：

| 函式 | 用途 |
|------|------|
| `isSkandha<S>(obj, skandha)` | 檢查物件是否屬於特定蘊 |
| `hasSkandha(manifest, skandha)` | 檢查 PluginManifest 是否宣告特定蘊（支援單值與多值） |

[程式碼: packages/sdk/src/types/aggregates.ts#isSkandha, hasSkandha]

**D3-R1 (20/20 全票)**：五個根介面在型別代數上覆蓋所有認知相關的 PluginHooks 型別，不需要第六個根介面。

### 2.2 色蘊 (Rupa) — I/O 邊界

色蘊定義 Agent 與外部世界的感官邊界，包含輸入（IListener）和輸出（IUI）兩個子介面。

> DC-6 裁定：色蘊含 IUI + IListener 兩繼承。

#### IListener — 感官輸入通道

```typescript
export interface IListener extends IRupa {
  id: string;
  name: string;
  start?(): Promise<void>;
  stop?(): Promise<void>;
}
```
[程式碼: packages/sdk/src/types/listener.ts#IListener]

| 屬性/方法 | 型別 | 說明 |
|-----------|------|------|
| `skandha` | `'rupa'` | 繼承自 IRupa |
| `id` | `string` | 唯一識別符 |
| `name` | `string` | 顯示名稱 |
| `start()` | `Promise<void>` (optional) | 啟動感官輸入通道 |
| `stop()` | `Promise<void>` (optional) | 停止感官輸入通道 |

**PluginHooks**: `hooks.listeners: IListener[]`
**Registry**: `ListenerRegistry` — Map-backed, `register / get / list`
[程式碼: packages/core/src/infrastructure/listener-registry.ts#ListenerRegistry]

#### IUI — 顯示輸出

```typescript
export interface IUI extends IRupa {
  id: string;
  name: string;
  onEvent(event: AgentEvent): void | Promise<void>;
  start?(): Promise<void>;
  stop?(): Promise<void>;
}
```
[程式碼: packages/sdk/src/types/ui.ts#IUI]

| 屬性/方法 | 型別 | 說明 |
|-----------|------|------|
| `skandha` | `'rupa'` | 繼承自 IRupa |
| `onEvent()` | `(event: AgentEvent) => void \| Promise<void>` | 接收事件並呈現 |

**PluginHooks**: `hooks.ui: IUI[]`
**Registry**: `UIRegistry` — Map-backed, `register / get / list`
**Bridge**: `createTransportBridge(bus, uiRegistry)` 將 EventBus 事件轉發至所有 IUI
[程式碼: packages/core/src/infrastructure/ui-registry.ts#UIRegistry]
[程式碼: packages/core/src/transport/bridge.ts]

### 2.3 受蘊 (Vedana) — 信號處理

受蘊以連續 hedonic 信號模型（非三值列舉）描述苦/樂/捨感受。

#### ChannelVedana — 連續 hedonic 信號

```typescript
export interface ChannelVedana {
  readonly valence: number;    // [-1.0, +1.0]
  readonly intensity: number;  // [0.0, 1.0]
  readonly type: VedanaType;   // 'dukkha' | 'sukha' | 'upekkha' (derived)
  readonly source: string;     // 根門識別符
}
```
[程式碼: packages/sdk/src/types/vedana.ts#ChannelVedana]

離散標籤經由 `VedanaClassificationConfig` 的閾值分類得出（`classifyVedana(valence, config)`）。預設閾值：dukkhaThreshold: -0.1, sukhaThreshold: 0.1。

#### IVedanaSensor — Plugin 提供的感測器

```typescript
export interface IVedanaSensor {
  readonly id: string;
  readonly channel: string;
  sense(event: unknown): ChannelVedana;
}
```
[程式碼: packages/sdk/src/types/vedana.ts#IVedanaSensor]

**注意**: IVedanaSensor 未顯式繼承 IVedana（弱繼承）。蘊歸屬透過 PluginManifest 的 `skandha` 欄位宣告。

**PluginHooks**: `hooks.vedanaSensors: IVedanaSensor[]`
**Registry**: `VedanaRegistry` — Map-backed, `register / get / list / remove`
[程式碼: packages/core/src/infrastructure/vedana-registry.ts#VedanaRegistry]

#### VedanaAssessment — 彙總評估

```typescript
export interface VedanaAssessment {
  readonly aggregate: ChannelVedana;
  readonly channels: readonly ChannelVedana[];
  readonly pidOutput: number;
  readonly timestamp: number;
}
```

VedanaAssessment 彙總多通道信號，供 IVolition 和 IKlesha 消費。

#### VedanaEmergency — 持續苦受狀態機

```typescript
export function checkVedanaEmergency(
  vedana: ChannelVedana,
  state: VedanaEmergencyState,
  config: VedanaEmergencyConfig,
): { thresholdBoost: number; updatedState: VedanaEmergencyState }
```
[程式碼: packages/core/src/vijnana/vedana-emergency.ts#checkVedanaEmergency]

偵測連續高強度苦受（intensity >= 0.8, 持續 5 ticks），觸發閾值提升（+0.15），含 10 ticks cooldown。

#### 受蘊輔助型別

| 型別 | 說明 |
|------|------|
| `VedanaType` | `'dukkha' \| 'sukha' \| 'upekkha'` |
| `VedanaClassificationConfig` | 分類閾值配置 |
| `VedanaTag` | O(1) 快取標籤 |
| `VedanaDimension` | 多維受蘊 (valence + arousal + dominance) |
| `VedanaEmergencyConfig` | 持續苦受觸發配置 |
| `VedanaSensorConfig` | 行蘊 vedana 映射增益配置 (maxSamskaraVedanaGain: 0.3) |

### 2.4 想蘊 (Samjna) — 認知處理

想蘊涵蓋認知處理的完整光譜，以 IProvider 作為 LLM 認知後端介面。

```typescript
export interface IProvider extends ISamjna {
  id: string;
  name: string;
  models: ModelInfo[];
  chat(request: ChatRequest): AsyncIterable<ProviderStreamEvent>;
  isConfigured?(): boolean;
  loginHint?: LoginHint;
}
```
[程式碼: packages/sdk/src/types/provider.ts#IProvider]

| 屬性/方法 | 型別 | 說明 |
|-----------|------|------|
| `skandha` | `'samjna'` | 繼承自 ISamjna |
| `models` | `ModelInfo[]` | 可用模型列表 (id, name, contextWindow, maxOutputTokens) |
| `chat()` | `AsyncIterable<ProviderStreamEvent>` | 串流式認知處理 API |

**PluginHooks**: `hooks.providers: IProvider[]`
**Registry**: `ProviderRegistry` — `register / get / resolveModel / list`
[程式碼: packages/core/src/infrastructure/provider-registry.ts#ProviderRegistry]

`resolveModel(modelId)` 遍歷所有 provider，找到第一個包含指定 model 的 provider。

**Streaming Protocol**: `chat()` 回傳 `AsyncIterable<ProviderStreamEvent>`，事件類型包含 `text_delta`, `tool_call_start`, `tool_call_delta`, `tool_call_end`, `finish`。

### 2.5 行蘊 (Samskara) — 行動執行

行蘊以 ITool 作為可執行行動的唯一子介面。

> DC-6 裁定：行蘊保持開放，不鎖定為身行/語行/意行三類。
> D3-R4 (20/20 全票)：ITool 作為唯一子介面已足夠，無需新增。

```typescript
export interface ITool<TInput = unknown> extends ISamskara {
  id: string;
  description: string;
  parameters: z.ZodType<TInput>;
  execute(input: TInput, ctx: ToolContext): Promise<string>;
}
```
[程式碼: packages/sdk/src/types/tool.ts#ITool]

| 屬性/方法 | 型別 | 說明 |
|-----------|------|------|
| `skandha` | `'samskara'` | 繼承自 ISamskara |
| `parameters` | `z.ZodType<TInput>` | Zod schema 參數驗證 |
| `execute()` | `(input, ctx) => Promise<string>` | 執行行動並回傳結果 |

**PluginHooks**: `hooks.tools: ITool[]`
**Registry**: `ToolRegistry` — `register / get / list / toJsonSchemas`
[程式碼: packages/core/src/infrastructure/tool-registry.ts#ToolRegistry]

`toJsonSchemas()` 將所有 ITool 轉換為 LLM API 可用的 JSON Schema 格式。

**Sandbox Proxy**: Sandbox plugin 透過 `createProxyTool()` 建立代理 ITool，經 RPC 轉發 `execute()` 至 worker thread。
[程式碼: packages/core/src/sandbox/sandbox-manager.ts#createProxyTool]

**ToolContext**: 工具執行上下文，包含 workingDirectory, allowedPaths, signal, bus。

### 2.6 識蘊 (Vijnana) — 控制與決策

識蘊是五蘊中最複雜的一蘊，包含四個子介面：IGuide、IVolition、IGearArbiter、IKlesha。

#### IGuide — 行為約束

```typescript
export interface IGuide extends IVijnana {
  id: string;
  name: string;
  getSystemPrompt(): string | Promise<string>;
}
```
[程式碼: packages/sdk/src/types/guide.ts#IGuide]

IGuide 產生 system prompt，注入每次 LLM 呼叫的 ChatRequest，約束想蘊 (IProvider) 的認知處理範圍。

**PluginHooks**: `hooks.guides: IGuide[]`
**Registry**: `GuideRegistry` — `register / get / list`
[程式碼: packages/core/src/infrastructure/guide-registry.ts#GuideRegistry]

#### IGearArbiter — 齒輪仲裁 (跨蘊: samjna + vijnana)

```typescript
export interface IGearArbiter {
  readonly id: string;
  readonly priority: number;
  evaluate(context: GearContext): GearEvaluation | Promise<GearEvaluation>;
}
```
[程式碼: packages/sdk/src/types/gear-arbiter.ts#IGearArbiter]

**跨蘊設計**: IGearArbiter 未繼承任何根介面（弱繼承），在 PluginManifest 中宣告 `skandha: ['samjna', 'vijnana']`。Cycle 02-4 D1 裁定方案 (B)。

| 屬性/方法 | 型別 | 說明 |
|-----------|------|------|
| `id` | `string` | 仲裁者識別符 |
| `priority` | `number` | 優先順序（數字越小越先評估） |
| `evaluate()` | `GearContext => GearEvaluation` | 評估上下文並推薦齒輪動作 |

**關聯型別**:

| 型別 | 說明 |
|------|------|
| `GearContext` | 評估上下文 (input, proposedToolCalls, actionHistory, agentConfig) |
| `GearEvaluation` | 評估結果 (action: GearAction, confidence, riskCategory?, reasoning?) |
| `GearAction` | `number \| 'abstain'` — N-Gear 泛化 |
| `RiskCategory` | `'destructive' \| 'state_modifying' \| 'read_only' \| 'informational'` |
| `RouteResult` | 最終路由決策 (gear, decidedBy?, confidence, riskAdjusted) |
| `ManoAggregatorConfig` | 完整仲裁配置 (perArbiterMs, chainMs, maxConfidenceByGear, ...) |

**PluginHooks**: `hooks.gearArbiters: IGearArbiter[]`
**Registry**: `GearArbiterRegistry` — `register / get / list / listSorted / remove`
[程式碼: packages/core/src/mano/gear-arbiter-registry.ts#GearArbiterRegistry]

`listSorted()` 按 priority 升序排列，同 priority 以 FIFO 作為 tie-break。

**SDK 工具函式**:
- `isGearArbiter(value)` — 結構型別守衛
- `computeAdjustedThreshold(base, risk, delta, floor, ceiling)` — 風險調整後閾值
- `inferRiskCategory(action)` — SDK 便利工具（Core 不使用）
[程式碼: packages/sdk/src/types/gear-arbiter.ts#isGearArbiter, computeAdjustedThreshold]

#### IVolition — 兩階段審議

```typescript
export interface IVolition extends IVijnana {
  deliberatePlan(input: PlanDeliberationInput): Promise<PlanDeliberationResult>;
  deliberateAction(input: ActionDeliberationInput): Promise<ActionDeliberationResult>;
}
```
[程式碼: packages/sdk/src/types/volition.ts#IVolition]

| 方法 | 時序預算 | 說明 |
|------|---------|------|
| `deliberatePlan()` | 1-3ms (vijnana-clock) | Phase 1: 審查所有提議行動 |
| `deliberateAction()` | 0.5-1ms (vijnana-clock) | Phase 2: 個別行動 veto/approve |

**關聯型別**:

| 型別 | 說明 |
|------|------|
| `PlanDeliberationInput` | proposedActions, kleshaSignals, vedanaAssessment, deliberationContext? |
| `PlanDeliberationResult` | modifiedPlan: null=接受 / array=改寫, + reasoning |
| `ActionDeliberationInput` | proposedAction, kleshaSignals, vedanaAssessment, planContext |
| `ActionDeliberationResult` | veto, alternative, reasoning |
| `DeliberationContext` | routeResult, actionHistory (Plan28 新增) |

**PluginHooks**: `hooks.volition: IVolition` — **單一值**，非陣列。Last-plugin-wins 語義。
[程式碼: packages/sdk/src/types/plugin.ts#PluginHooks]

#### IKlesha — 煩惱框架

```typescript
export interface IKlesha {
  readonly type: KleshaType;
  perceive(context: KleshaContext): KleshaSignal;
}
```
[程式碼: packages/sdk/src/types/klesha.ts#IKlesha]

**注意**: IKlesha 未繼承 IVijnana（弱繼承）。DC-12 裁定 Klesha 屬識蘊。四個 Core 內建實作：

| 類型 | 信號處理模型 | 行為特徵 |
|------|-------------|---------|
| `Moha` (無明) | Low-pass filter | 平滑急速變化，高值=對變化反應遲鈍 |
| `Drishti` (見) | Band-pass filter | 放大重複行為模式，高值=認同固著 |
| `Mana` (慢) | PD controller | 回應正向 valence 值與變化率，高值=過度自信 |
| `Sneha` (愛) | Integrator | 累積正向結果+指數衰減，高值=模式依附 |

[程式碼: packages/core/src/vijnana/klesha.ts#Moha, Drishti, Mana, Sneha]

**Klesha 增益調度 (KleshaModulatedDispatcher)**:

```
theta(t) = clamp(theta_0 + w_sneha * mu_sneha + w_mana * mu_mana, theta_min, theta_max)
```

預設參數: theta_0=0.6, w_sneha=-0.15, w_mana=+0.15, theta_min=0.3, theta_max=0.9
[程式碼: packages/core/src/vijnana/klesha.ts#KleshaModulatedDispatcher]

**VitakkaWatchdog**: 防止齒輪停滯（samsaric stall）。非預設齒輪連續超限時強制切回預設齒輪。
[程式碼: packages/core/src/vijnana/vitakka-watchdog.ts#createVitakkaWatchdog]

### 2.7 跨蘊型別 (Cross-Skandha Types)

#### CoarisingBundle — 五遍行共起

```typescript
export interface CoarisingBundle {
  readonly sparsha: SparshEvent;           // 觸
  readonly vedana: ChannelVedana;          // 受
  readonly samjna: ChannelSamjna;          // 想
  readonly cetana: ChannelCetana;          // 思
  readonly manasikara: ChannelManasikara;  // 作意
  readonly layer: 1 | 2;
  readonly mode: 'fast' | 'slow';
  readonly sahaja: SahajaContract;
  readonly timestamp: number;
}
```
[程式碼: packages/sdk/src/types/coarising.ts#CoarisingBundle]

每個 consciousness moment 五遍行同時共起：

| 五遍行 | 型別 | 內容 |
|--------|------|------|
| 觸 (Sparsha) | `SparshEvent` | root + object + consciousness |
| 受 (Vedana) | `ChannelVedana` | valence + intensity + type |
| 想 (Samjna) | `ChannelSamjna` | label + confidence |
| 思 (Cetana) | `ChannelCetana` | intention + urgency |
| 作意 (Manasikara) | `ChannelManasikara` | focus + intensity |

**SahajaContract**: 品質保證契約

```typescript
export interface SahajaContract {
  readonly mutualConsistency: boolean;    // 各成分互相參照
  readonly atomicPublication: boolean;    // 外部不可見部分 bundle
  readonly stalenessUpperBound: number;   // 最大時間差 (ms)
}
```
[程式碼: packages/sdk/src/types/coarising.ts#SahajaContract]

#### SparshEvent — 觸事件

```typescript
export interface SparshEvent {
  readonly root: string;           // 感官根 ("cli", "mcp", "mano")
  readonly object: unknown;        // 外部對境
  readonly consciousness: string;  // 認知域 ("mano-vijnana")
  readonly timestamp?: number;
}
```
[程式碼: packages/sdk/src/types/coarising.ts#SparshEvent]

`Object.freeze()` 確保建構後不可變。
[程式碼: packages/core/src/mano/sparsh-event-builder.ts#createSparshEvent]

#### SlashCommand — 跨蘊指令 (基礎設施)

```typescript
export interface SlashCommand {
  name: string;
  description: string;
  execute(args: string, ctx: IPluginContext, sessionId?: string): Promise<string | undefined>;
}
```
[程式碼: packages/sdk/src/types/plugin.ts#SlashCommand]

**D3-R2a**: SlashCommand 不歸入任何蘊，因為它 bypass ExecutionLoop，不參與五蘊認知迴路。它走 fast path，直接由 `CommandRegistry.execute()` 處理。

**D3-R2b 安全觀察**: SlashCommand bypass ExecutionLoop 意味著同時 bypass SafetyMonitor、IVolition 審議、Klesha 增益調度。Plugin 透過 SlashCommand 執行的操作僅受 IPluginContext 的能力約束。

**PluginHooks**: `hooks.commands: SlashCommand[]`
**Registry**: `CommandRegistry` — Handler-chain 模式，多 handler 可共享同一命令名。
[程式碼: packages/core/src/infrastructure/command-registry.ts#createCommandRegistry]

#### IPluginService — 跨 Plugin 服務

```typescript
export interface IPluginService {
  name: string;
  version: string;
}
```
[程式碼: packages/sdk/src/types/service.ts#IPluginService]

不歸屬特定蘊，是 plugin 間的服務注入機制。

### 2.8 完整型別樹圖 (Complete Type Tree)

```
aggregates.ts
├── type Skandha = 'rupa' | 'vedana' | 'samjna' | 'samskara' | 'vijnana'
│
├── IRupa { skandha: 'rupa' }
│   ├── IListener extends IRupa          (listener.ts)
│   │   ├── id: string
│   │   ├── name: string
│   │   ├── start?(): Promise<void>
│   │   └── stop?(): Promise<void>
│   └── IUI extends IRupa               (ui.ts)
│       ├── id: string
│       ├── name: string
│       ├── onEvent(event): void | Promise<void>
│       ├── start?(): Promise<void>
│       └── stop?(): Promise<void>
│
├── IVedana { skandha: 'vedana' }
│   ├── [值物件] ChannelVedana           (vedana.ts)
│   │   ├── valence: number [-1.0, +1.0]
│   │   ├── intensity: number [0.0, 1.0]
│   │   ├── type: VedanaType (derived)
│   │   └── source: string
│   ├── [值物件] VedanaAssessment        (vedana.ts)
│   │   ├── aggregate: ChannelVedana
│   │   ├── channels: readonly ChannelVedana[]
│   │   ├── pidOutput: number
│   │   └── timestamp: number
│   ├── [配置] VedanaClassificationConfig
│   ├── [配置] VedanaEmergencyConfig
│   ├── [配置] VedanaSensorConfig
│   └── IVedanaSensor                    (vedana.ts) [弱繼承: 未 extends IVedana]
│       ├── id: string
│       ├── channel: string
│       └── sense(event): ChannelVedana
│
├── ISamjna { skandha: 'samjna' }
│   └── IProvider extends ISamjna        (provider.ts)
│       ├── id: string
│       ├── name: string
│       ├── models: ModelInfo[]
│       ├── chat(request): AsyncIterable<ProviderStreamEvent>
│       ├── isConfigured?(): boolean
│       └── loginHint?: LoginHint
│
├── ISamskara { skandha: 'samskara' }
│   └── ITool<TInput> extends ISamskara  (tool.ts)
│       ├── id: string
│       ├── description: string
│       ├── parameters: z.ZodType<TInput>
│       └── execute(input, ctx): Promise<string>
│
└── IVijnana { skandha: 'vijnana' }
    ├── IGuide extends IVijnana          (guide.ts)
    │   ├── id: string
    │   ├── name: string
    │   └── getSystemPrompt(): string | Promise<string>
    ├── IVolition extends IVijnana       (volition.ts)
    │   ├── deliberatePlan(input): Promise<PlanDeliberationResult>
    │   └── deliberateAction(input): Promise<ActionDeliberationResult>
    ├── IGearArbiter                     (gear-arbiter.ts) [弱繼承: 獨立介面]
    │   ├── id: string                   [manifest skandha: ['samjna','vijnana']]
    │   ├── priority: number
    │   └── evaluate(context): GearEvaluation | Promise<GearEvaluation>
    └── IKlesha                          (klesha.ts) [弱繼承: 獨立介面, DC-12]
        ├── type: KleshaType
        ├── perceive(context): KleshaSignal
        └── [Core 實作]
            ├── Moha  — Low-pass filter
            ├── Drishti — Band-pass filter
            ├── Mana  — PD controller
            └── Sneha — Integrator

[跨蘊型別]
CoarisingBundle (coarising.ts) — 五遍行共起
├── SparshEvent (觸)
├── ChannelVedana (受)
├── ChannelSamjna (想)
├── ChannelCetana (思)
├── ChannelManasikara (作意)
└── SahajaContract (品質保證)

SlashCommand (plugin.ts) — 跨蘊指令 (bypass ExecutionLoop)
IPluginService (service.ts) — 跨 plugin 服務
EventBus (events.ts) — 跨蘊通訊管道
```

---

## 三、依賴注入佈線 (DI Wiring)

### 3.1 Composition Root: createAgentCore()

OpenStarry 採用 **Pure DI**（手動 DI）模式，無 DI 容器框架。所有物件的建立、組裝、佈線在 `createAgentCore(config)` 這個單一 Composition Root 函式中完成。

[程式碼: packages/core/src/agents/agent-core.ts#createAgentCore]

Pure DI 的架構理由：
1. **零內建能力原則** — DI 容器本身就是一種「能力」，微核心應盡量少內建
2. **編譯期安全** — 所有依賴靜態確定，TypeScript 編譯器可捕捉錯誤
3. **Sandbox 安全** — Plugin 無法存取 DI 容器，透過 IPluginContext 限制存取範圍
4. **可測試性** — 直接傳入 mock 依賴，無需配置容器

### 3.2 Registry 架構 — 建構順序與依賴

`createAgentCore()` 中的 21 個元件建構順序（嚴格線性）：

| 順序 | 元件 | 工廠函式 | 依賴 |
|------|------|---------|------|
| 1 | EventBus | `createEventBus()` | 無 |
| 2 | EventQueue | `createEventQueue()` | 無 |
| 3 | SessionManager | `createSessionManager(bus)` | bus |
| 4 | ContextManager | `createContextManager()` | 無 |
| 5 | ToolRegistry | `createToolRegistry()` | 無 |
| 6 | ProviderRegistry | `createProviderRegistry()` | 無 |
| 7 | ListenerRegistry | `createListenerRegistry()` | 無 |
| 8 | UIRegistry | `createUIRegistry()` | 無 |
| 9 | GuideRegistry | `createGuideRegistry()` | 無 |
| 10 | CommandRegistry | `createCommandRegistry()` | 無 |
| 11 | ServiceRegistry | `createServiceRegistry()` | 無 |
| 12 | VedanaRegistry | `createVedanaRegistry()` | 無 |
| 13 | GearArbiterRegistry | `createGearArbiterRegistry()` | 無 |
| 14 | ManoAggregator | `createManoAggregator(bus, config)` | bus |
| 15 | VitakkaWatchdog | `createVitakkaWatchdog(config)` | 無 (bus 用於事件訂閱) |
| 16 | SecurityLayer | `createSecurityLayer(...)` | config, sessionManager |
| 17 | SafetyMonitor | `createSafetyMonitor(...)` | config |
| 18 | MetricsCollector | `createMetricsCollector()` | 無 |
| 19 | PluginSandboxManager | `createPluginSandboxManager(deps)` | bus, sessionManager, registries |
| 20 | PluginLoader | `createPluginLoader(deps)` | 所有 registry, sandboxManager, bus |
| 21 | TransportBridge | `createTransportBridge(bus, uiRegistry)` | bus, uiRegistry |

[程式碼: packages/core/src/agents/agent-core.ts#createAgentCore, lines 97-192]

**關鍵觀察**: Registry (順序 5-13) 全部是無依賴的純 Map 容器，它們的建立不依賴其他元件。

### 3.3 九個 Registry 與蘊的對應

| Registry | 蘊 | 資料結構 | 語義 |
|----------|-----|---------|------|
| ToolRegistry | 行蘊 (samskara) | Map-backed, ID-keyed | 多值累加, 同 ID 覆蓋 |
| ProviderRegistry | 想蘊 (samjna) | Map-backed, ID-keyed | 多值累加, 含 resolveModel |
| ListenerRegistry | 色蘊 (rupa-in) | Map-backed, ID-keyed | 多值累加 |
| UIRegistry | 色蘊 (rupa-out) | Map-backed, ID-keyed | 多值累加 |
| GuideRegistry | 識蘊 (vijnana) | Map-backed, ID-keyed | 多值累加 |
| CommandRegistry | 跨蘊 | Map<string, SlashCommand[]> | Handler-chain pattern |
| ServiceRegistry | 跨蘊 | Map-backed | 拒絕同名重複 (throws) |
| VedanaRegistry | 受蘊 (vedana) | Map-backed, ID-keyed | 多值累加 |
| GearArbiterRegistry | 識蘊 (vijnana) | Map-backed, ID-keyed | priority 排序 |

**讀寫分離模式**: PluginLoader 是所有 Registry 的唯一寫入者；ExecutionLoop、IPluginContext、AgentCore 是讀取者。

### 3.4 IPluginContext — Plugin 的 DI 入口

每個 plugin 在初始化時收到一個 `IPluginContext`，提供對 core 子系統的受限存取：

```typescript
export interface IPluginContext {
  bus: EventBus;
  workingDirectory: string;
  agentId: string;
  config: Record<string, unknown>;
  pushInput: (event: InputEvent) => void;
  sessions: ISessionManager;
  tools?: { list(): ITool[]; get(id: string): ITool | undefined };
  guides?: { list(): IGuide[] };
  providers?: { list(): IProvider[]; get(id: string): IProvider | undefined };
  services?: IServiceRegistry;
  commands?: { list(): SlashCommand[] };
  metrics?: { getSnapshot(): Record<string, unknown> };
}
```
[程式碼: packages/sdk/src/types/plugin.ts#IPluginContext]

**Lazy Accessor 模式**: 每個 registry accessor 使用 closure 封裝 registry reference，提供延遲存取：

```typescript
tools: {
  list: () => toolRegistry.list(),    // closure 捕獲 toolRegistry
  get: (id) => toolRegistry.get(id),
},
```

Plugin 呼叫 `ctx.tools.list()` 時取得**當前**已註冊的所有工具（包含後續載入的 plugin 註冊的工具），而非建構時的快照。

[程式碼: packages/core/src/agents/agent-core.ts#getPluginContext, lines 199-237]

**Provider 能力過濾**: 若 plugin manifest 宣告 `capabilities.allowedProviders`，IPluginContext 中的 providers accessor 僅回傳授權的 provider：

```typescript
providers: {
  list: () => {
    const allProviders = providerRegistry.list();
    if (!shouldFilterProviders) return allProviders;
    return allProviders.filter(p => allowedProviderIds.includes(p.id));
  },
  get: (id) => {
    if (shouldFilterProviders && !allowedProviderIds.includes(id)) return undefined;
    return providerRegistry.get(id);
  },
},
```
[程式碼: packages/core/src/agents/agent-core.ts#getPluginContext, lines 218-228]

### 3.5 從 Registry 到 ExecutionLoop 的佈線

ExecutionLoop 的依賴透過 `ExecutionLoopDeps` 介面注入：

```typescript
export interface ExecutionLoopDeps {
  bus: EventBus;
  queue: EventQueue;
  sessionManager: ISessionManager;
  contextManager: IContextManager;
  toolRegistry: ToolRegistry;
  security: SecurityLayer;
  safetyMonitor: SafetyMonitor;
  providerResolver: (sessionId?: string) => IProvider;
  guideResolver: () => IGuide | undefined;
  modelResolver: (sessionId?: string) => string | undefined;
  manoAggregator?: ManoAggregator;
  gearArbiterRegistry?: { listSorted(): IGearArbiter[] };
  volition?: { ... };
  // + config values: maxToolRounds, slidingWindowSize, temperature, etc.
}
```
[程式碼: packages/core/src/execution/loop.ts#ExecutionLoopDeps]

**Resolver 模式**: Provider/Guide/Model 解析透過 closure 延遲到運行時：

```
resolveProvider(sessionId):
  1. resolveModel(sessionId) → model ID
     a. cognition-config service → getModel(sessionId)  [runtime override]
     b. config.cognition.model                           [static config]
  2. providerRegistry.resolveModel(model) → provider
  3. cognition-config service → getProvider(sessionId)   [runtime override]
  4. config.cognition.provider                           [static config fallback]
  5. throw Error if none found
```
[程式碼: packages/core/src/agents/agent-core.ts#resolveProvider, lines 248-270]

**IVolition 佈線 (Adapter 模式)**: IVolition 透過 `pluginLoader.getVolition()` 取出，包裝為 ExecutionLoopDeps.volition。v0.28.0-alpha 中，`getKleshaSignals` 和 `getVedanaAssessment` 回傳中性預設值（全零 / upekkha）：

```typescript
const volitionDeps = pluginVolition ? {
  deliberatePlan: pluginVolition.deliberatePlan.bind(pluginVolition),
  deliberateAction: pluginVolition.deliberateAction.bind(pluginVolition),
  getKleshaSignals: (): KleshaSignalBundle => ({ moha: 0, drishti: 0, mana: 0, sneha: 0 }),
  getVedanaAssessment: (): VedanaAssessment => {
    const neutral: ChannelVedana = { valence: 0, intensity: 0, type: 'upekkha', source: 'neutral' };
    return { aggregate: neutral, channels: [neutral], pidOutput: 0, timestamp: Date.now() };
  },
} : undefined;
```
[程式碼: packages/core/src/agents/agent-core.ts#start, lines 352-361]

### 3.6 事件驅動佈線 (EventBus Wiring)

除了 DI 的直接引用佈線外，EventBus 提供鬆耦合的事件驅動佈線：

| 事件 | 訂閱者 | 行為 |
|------|--------|------|
| `gear:switch` | VitakkaWatchdog | 記錄齒輪週期，偵測停滯 |
| `STATE_RESET` | safetyMonitor.reset() | 重置安全計數器 |
| `TOOL_EXECUTING` | metrics.increment | 工具呼叫計數 |
| `TOOL_ERROR` | metrics.increment | 工具錯誤計數 |
| `PROVIDER_ERROR` | metrics.increment | Provider 錯誤計數 |

**VitakkaWatchdog 佈線**：

```typescript
bus.on('gear:switch', (event) => {
  const gear = payload.gear;
  if (gear === manoConfig.defaultGear) {
    watchdog.resetOnDefaultGear();
  } else {
    const stalled = watchdog.recordGearCycle(gear);
    if (stalled) {
      bus.emit({ type: 'vitakka:stall', timestamp: Date.now(), payload: { stalledGear: gear } });
      _manoAggregator.forceNextGear(manoConfig.defaultGear);
    }
  }
});
```
[程式碼: packages/core/src/agents/agent-core.ts#createAgentCore, lines 116-133]

### 3.7 完整 DI 依賴圖

```
IAgentConfig (agent.json)
         │
         v
  createAgentCore(config) ─────────────────────────────────────────────┐
  │                                                                    │
  │  ┌── INFRASTRUCTURE ─────────────────────────────────────────────┐ │
  │  │  EventBus          (跨蘊通訊管道)                               │ │
  │  │  EventQueue        (輸入佇列)                                   │ │
  │  │  SessionManager    ← bus                                       │ │
  │  │  ContextManager                                                │ │
  │  └────────────────────────────────────────────────────────────────┘ │
  │                                                                    │
  │  ┌── REGISTRIES (9) ─────────────────────────────────────────────┐ │
  │  │  ToolRegistry         (samskara)                               │ │
  │  │  ProviderRegistry     (samjna)                                 │ │
  │  │  ListenerRegistry     (rupa-in)                                │ │
  │  │  UIRegistry           (rupa-out)                               │ │
  │  │  GuideRegistry        (vijnana)                                │ │
  │  │  CommandRegistry      (cross-skandha)                          │ │
  │  │  ServiceRegistry      (cross-skandha)                          │ │
  │  │  VedanaRegistry       (vedana)                                 │ │
  │  │  GearArbiterRegistry  (vijnana)                                │ │
  │  └────────────────────────────────────────────────────────────────┘ │
  │                                                                    │
  │  ┌── MANO LAYER ─────────────────────────────────────────────────┐ │
  │  │  ManoAggregator       ← bus, config   (純路由: if/else)        │ │
  │  │  VitakkaWatchdog      ← config        (gear stall detection)  │ │
  │  └────────────────────────────────────────────────────────────────┘ │
  │                                                                    │
  │  ┌── SECURITY LAYER ─────────────────────────────────────────────┐ │
  │  │  SecurityLayer        ← config, sessionManager                 │ │
  │  │  SafetyMonitor        ← config                                 │ │
  │  └────────────────────────────────────────────────────────────────┘ │
  │                                                                    │
  │  ┌── PLUGIN LAYER ──────────────────────────────────────────────┐  │
  │  │  PluginSandboxManager ← bus, sessionMgr, registries           │  │
  │  │  PluginLoader         ← all registries, sandboxMgr, bus       │  │
  │  └────────────────────────────────────────────────────────────────┘ │
  │                                                                    │
  │  TransportBridge          ← bus, uiRegistry                        │
  │  MetricsCollector                                                  │
  │                                                                    │
  │  ┌── EXECUTION LOOP (created at start()) ────────────────────────┐ │
  │  │  createExecutionLoop(deps):                                    │ │
  │  │    ← bus, queue, sessionMgr, contextMgr, toolReg               │ │
  │  │    ← security, safetyMonitor                                   │ │
  │  │    ← providerResolver, guideResolver, modelResolver (closure)  │ │
  │  │    ← manoAggregator, gearArbiterRegistry                       │ │
  │  │    ← volition (adapter)                                        │ │
  │  └────────────────────────────────────────────────────────────────┘ │
  └────────────────────────────────────────────────────────────────────┘
```

---

## 四、Plugin 載入流程 (Plugin Loading)

### 4.1 Plugin 定義: IPlugin

```typescript
export interface IPlugin {
  manifest: PluginManifest;
  factory: (ctx: IPluginContext) => Promise<PluginHooks>;
}
```
[程式碼: packages/sdk/src/types/plugin.ts#IPlugin]

Plugin 由兩個部分組成：
1. **Manifest** — 靜態後設資料
2. **Factory** — 非同步工廠函式，接收 IPluginContext，回傳 PluginHooks

工廠模式使延遲初始化和 sandbox 隔離成為可能。

### 4.2 PluginManifest 欄位

```typescript
export interface PluginManifest {
  name: string;
  version: string;
  description?: string;
  author?: string;
  sandbox?: SandboxConfig;
  integrity?: string | PkiIntegrity;
  capabilities?: PluginCapabilities;
  services?: string[];
  serviceDependencies?: string[];
  skandha?: Skandha | readonly Skandha[];
}
```
[程式碼: packages/sdk/src/types/plugin.ts#PluginManifest]

| 欄位 | 型別 | 必填 | 說明 |
|------|------|------|------|
| `name` | `string` | 是 | Plugin 唯一名稱 |
| `version` | `string` | 是 | 語意版本號 |
| `sandbox` | `SandboxConfig` | 否 | Sandbox 配置 (enabled, memoryLimitMb, ...) |
| `integrity` | `string \| PkiIntegrity` | 否 | SHA-512 hash 或 Ed25519/RSA 簽章 |
| `capabilities` | `PluginCapabilities` | 否 | 能力宣告 (allowedProviders) |
| `services` | `string[]` | 否 | 此 plugin 提供的服務 |
| `serviceDependencies` | `string[]` | 否 | 此 plugin 依賴的服務 |
| `skandha` | `Skandha \| Skandha[]` | 否 | 五蘊歸屬 (純 metadata，D3-R3) |

**skandha 欄位語義** (D3-R3, 18/20):
- **單值**: `skandha: 'samjna'` — plugin 屬想蘊
- **多值**: `skandha: ['samjna', 'vijnana']` — 跨蘊 plugin
- **省略**: 合法，不影響載入和路由
- **定位**: 純後設資料 (metadata)，非路由依據。驗證時僅發 warning。

### 4.3 拓撲排序 — Kahn's Algorithm

PluginLoader 使用 Kahn's algorithm 對 plugin 進行 service dependency 排序：

```
輸入: IPlugin[] (按 agent.json 配置順序)
輸出: IPlugin[] (按依賴順序)

Step 1: 建立 service → provider plugin 映射
  for each plugin:
    for each plugin.manifest.services:
      serviceToProvider[service] = plugin

Step 2: 建立依賴圖
  for each plugin:
    for each plugin.manifest.serviceDependencies:
      if serviceToProvider[dep] exists:
        add edge: serviceToProvider[dep] → plugin
        increment inDegree[plugin]

Step 3: Kahn's algorithm
  queue = [plugins with inDegree == 0, sorted by original config index]
  sorted = []
  while queue not empty:
    current = queue.shift()
    sorted.push(current)
    for each dependent of current:
      decrement inDegree[dependent]
      if inDegree[dependent] == 0: queue.push(dependent)

Step 4: 循環依賴偵測
  if sorted.length < plugins.length:
    throw PluginLoadError("circular-dependency")
```
[程式碼: packages/core/src/infrastructure/plugin-loader.ts#topologicalSort, lines 60-148]

**穩定排序保證**: 無依賴關係的 plugin 保持 config 中的原始順序。

### 4.4 載入流程 — Factory 到 Registry

`PluginLoader.load(plugin, ctx)` 的完整步驟：

```
1. validatePluginSkandha(plugin)          — 驗證 skandha 欄位 (warning only)
2. 檢查 serviceDependencies               — 軟驗證 (warning only)
3. 簽章驗證 (如有 integrity 欄位)
4. sandbox.enabled? → Sandbox 載入 : Direct 載入
   ├── Direct: plugin.factory(ctx)         → PluginHooks
   └── Sandbox: sandboxManager.loadInSandbox(plugin, ctx) → Proxy PluginHooks
5. 依序註冊 hooks 到 registries:
   a. hooks.tools        → toolRegistry.register(tool)
   b. hooks.providers    → providerRegistry.register(provider)
   c. hooks.listeners    → listenerRegistry.register(listener)
   d. hooks.ui           → uiRegistry.register(ui)
   e. hooks.guides       → guideRegistry.register(guide)
   f. hooks.commands     → commandRegistry.register(command)
   g. hooks.vedanaSensors → vedanaRegistry.register(sensor)
   h. hooks.gearArbiters  → gearArbiterRegistry.register(arbiter)
   i. hooks.volition      → registeredVolition = hooks.volition  (last-wins)
6. loadedHooks.push(hooks)                — 追蹤已載入 hooks
```
[程式碼: packages/core/src/infrastructure/plugin-loader.ts#load, lines 183-306]

### 4.5 PluginHooks 到 Registry 的完整映射

| PluginHooks 欄位 | 型別 | 蘊 | Registry | 語義 |
|------------------|------|-----|----------|------|
| `providers` | `IProvider[]` | 想蘊 | ProviderRegistry | 多值累加, ID 覆蓋 |
| `tools` | `ITool[]` | 行蘊 | ToolRegistry | 多值累加, ID 覆蓋 |
| `listeners` | `IListener[]` | 色蘊 | ListenerRegistry | 多值累加, ID 覆蓋 |
| `ui` | `IUI[]` | 色蘊 | UIRegistry | 多值累加, ID 覆蓋 |
| `guides` | `IGuide[]` | 識蘊 | GuideRegistry | 多值累加, ID 覆蓋 |
| `vedanaSensors` | `IVedanaSensor[]` | 受蘊 | VedanaRegistry | 多值累加, ID 覆蓋 |
| `gearArbiters` | `IGearArbiter[]` | 跨蘊 (samjna+vijnana) | GearArbiterRegistry | 多值累加, priority 排序 |
| `volition` | `IVolition` | 識蘊 | (PluginLoader 內部) | **單值, last-wins** |
| `commands` | `SlashCommand[]` | 跨蘊 | CommandRegistry | 多值累加, handler chain |
| `dispose` | `() => Promise<void>` | N/A | (PluginLoader 內部) | 清理函式 |

**D3-R2 (20/20 全票)**: 上述映射在原始碼、設計文件、佛學三方面均驗證正確。

### 4.6 Sandbox 與簽章驗證

**Sandbox 載入路徑** (manifest.sandbox.enabled === true):

```
Step 1:   簽章驗證 (SHA-512 hash 或 Ed25519/RSA)
Step 1.5: 靜態 import 分析 (blockedModules / allowedModules)
Step 2:   取得 Worker (pool 或 dedicated)
Step 3:   心跳監控設置 (45s interval)
Step 4:   EventBus 轉發設置
Step 5:   Worker crash handler + restart logic
Step 6:   INIT_PLUGIN RPC → INIT_COMPLETE
Step 7:   建立 Proxy Hooks (createProxyTool, etc.)
```
[程式碼: packages/core/src/sandbox/sandbox-manager.ts#loadInSandbox, lines 355-574]

**Sandbox 隔離機制**:

| 隔離層 | 機制 | 說明 |
|--------|------|------|
| Process 隔離 | Node.js Worker Thread | 獨立 V8 isolate |
| 記憶體限制 | `resourceLimits.maxOldGenerationSizeMb` | 預設 512MB |
| CPU 監控 | Heartbeat + cpuTimeoutMs | 預設 60s |
| Import 封鎖 | `validatePluginImports()` | 靜態分析 |
| RPC 邊界 | postMessage only | Proxy hooks 封裝 |
| Provider 過濾 | capabilities.allowedProviders | RPC handler 中應用 |

**Worker Crash 重啟**: 指數退避策略 (maxRestarts: 3, backoffMs: 500, maxBackoffMs: 10000)。
[程式碼: packages/core/src/sandbox/sandbox-manager.ts#handleWorkerCrash]

### 4.7 Plugin 生命週期

```
              loadPlugin() / loadPlugins()
                       │
                       v
┌──────────┐    ┌─────────────┐    ┌────────────┐
│ CREATED  │───>│   LOADING   │───>│   LOADED   │
└──────────┘    │ validate    │    │ hooks 已註冊 │
                │ verify sig  │    └─────┬──────┘
                │ factory()   │          │ AgentCore.start()
                │ register    │          v
                └─────────────┘    ┌─────────────┐
                                   │  STARTING   │
                                   │ listener.   │
                                   │   start()   │
                                   │ ui.start()  │
                                   └─────┬───────┘
                                         v
                                   ┌─────────────┐
                                   │  RUNNING    │
                                   │ Execution   │
                                   │ Loop active │
                                   └─────┬───────┘
                                         │ AgentCore.stop()
                                         v
                                   ┌─────────────┐
                                   │  STOPPING   │
                                   │ listener.   │
                                   │   stop()    │
                                   │ ui.stop()   │
                                   └─────┬───────┘
                                         v
                                   ┌─────────────┐
                                   │ DISPOSING   │
                                   │ hooks.      │
                                   │  dispose()  │
                                   │ sandbox.    │
                                   │  shutdown() │
                                   └─────┬───────┘
                                         v
                                   ┌─────────────┐
                                   │  DISPOSED   │
                                   └─────────────┘
```

---

## 五、五蘊執行流 (Execution Flow)

### 5.1 ExecutionLoop 狀態機

ExecutionLoop 是一個事件驅動的有限狀態機 (FSM)：

```typescript
export type LoopState =
  | "WAITING_FOR_EVENT"
  | "ASSEMBLING_CONTEXT"
  | "AWAITING_LLM"
  | "PROCESSING_RESPONSE"
  | "EXECUTING_TOOLS"
  | "SAFETY_LOCKOUT";
```
[程式碼: packages/core/src/execution/loop.ts#LoopState]

特性:
- **單執行緒語義**: `processing` flag 確保同一時間只處理一個 InputEvent
- **EventQueue 解耦**: 所有外部輸入透過 `queue.pull()` 進入
- **安全斷路器**: `SAFETY_LOCKOUT` 為吸收態，需 `/reset` 解除

### 5.2 完整九階段流程

以下為一次完整的五蘊流轉：從外部輸入到結果回授，再回到等待狀態。

#### Phase 1: Input Reception — 色蘊 (rupa)

```
IListener (色蘊) → ctx.pushInput(inputEvent) → EventQueue
ExecutionLoop: queue.pull() → inputEvent
SparshEvent = createSparshEvent(inputEvent)  // root + object + consciousness
emit "sparsha:contact"
```

IListener 接收外部輸入（CLI stdin / WebSocket / HTTP），推入 EventQueue。ExecutionLoop 取得事件後，立即建構 SparshEvent 標記根門交會。

Slash command 走快速路徑: bypass 後續所有階段，由 `CommandRegistry.execute()` 直接處理。

#### Phase 2: Context Assembly — 識蘊 (vijnana)

```
setState("ASSEMBLING_CONTEXT")
ISessionManager.getStateManager(sessionId) → IStateManager
stateManager.addMessage(userMessage)
IContextManager.assembleContext(allMessages, slidingWindowSize)
IGuide.getSystemPrompt() → systemPrompt  // 我執框架約束
SafetyMonitor.beforeLLMCall() → budgetCheck  // 安全檢查
```

IGuide (識蘊) 產生 system prompt，注入 ChatRequest，約束想蘊的認知處理範圍。

#### Phase 3: Cognitive Processing — 想蘊 (samjna)

```
setState("AWAITING_LLM")
modelResolver(sessionId) → model ID
providerResolver(sessionId) → IProvider
ChatRequest = { model, messages, systemPrompt, tools, temperature, maxTokens }
provider.chat(chatRequest) → AsyncIterable<ProviderStreamEvent>
串流處理: text_delta, tool_call_start/delta/end, finish
```

IProvider.chat() 執行串流式認知處理，累積 text 和 tool call proposals。LLM timeout 預設 120s。

#### Phase 4: Gear Routing — 識蘊 + 想蘊 (vijnana + samjna)

前提: `stopReason === "tool_use" && pendingToolCalls.length > 0 && manoAggregator`

```
buildGearContext(input, proposedToolCalls, actionHistory, agentConfig)
arbiters = gearArbiterRegistry.listSorted()
routeResult = manoAggregator.route(gearContext, arbiters)
safetyMonitor.postRouteCheck(routeResult)  // v1 passthrough
```

ManoAggregator 路由邏輯（純路由，非能力）:

```
for each arbiter in sorted list:
  if chain deadline exceeded → return defaultGear
  evaluation = arbiter.evaluate(context)     // per-arbiter 100ms timeout
  if evaluation.action === 'abstain' → skip
  adjustedThreshold = computeAdjustedThreshold(base, risk, delta)
  if evaluation.confidence > adjustedThreshold → return { gear, decidedBy }
no arbiter met threshold → return defaultGear (Gear 2)
```

**退化路徑**: 無 arbiter 時直接回傳 `{ gear: defaultGear, confidence: 0 }`。

#### Phase 5: Deliberation — 識蘊 (vijnana)

前提: `stopReason === "tool_use" && pendingToolCalls.length > 0 && volition`

**Phase 5a — Plan-level deliberation** (1-3ms, vijnana-clock):

```typescript
planResult = await volition.deliberatePlan({
  proposedActions: pendingToolCalls.map(tc => ({ name: tc.name, arguments: tc.arguments })),
  kleshaSignals, vedanaAssessment, deliberationContext
});
// modifiedPlan 非 null → 過濾 pendingToolCalls
```

**Phase 5b — Per-action deliberation** (0.5-1ms each, vijnana-clock):
在 Phase 6 的迴圈中，每個 tool call 執行前：

```typescript
actionResult = await volition.deliberateAction({
  proposedAction, kleshaSignals, vedanaAssessment, planContext
});
if (actionResult.veto) continue;  // 跳過此 tool call
```

#### Phase 6: Tool Execution — 行蘊 (samskara)

```
setState("EXECUTING_TOOLS")
for each pendingToolCall (未被 veto):
  emit "action:proposed"
  validateInput(tool.parameters, toolCall.arguments)  // Zod schema
  result = await tool.execute(validatedInput, toolCtx)  // 30s timeout
  emit "action:executed"
  actionHistory.push({ name, result, isError })
  safetyMonitor.afterToolExecution(name, argsJson, isError)
```

#### Phase 7: Sensory Feedback — 受蘊 (vedana)

```
tool:result / tool:error → EventBus → IVedanaSensor.sense(event) → ChannelVedana
VedanaAssessment = aggregate(channels[])  // PID controller
emit "vedana:assessment"
```

增益限制: `maxSamskaraVedanaGain = 0.3`，防止行蘊單次結果對受蘊產生過大影響。

#### Phase 8: Affliction Update — 識蘊 (vijnana)

```
VedanaAssessment → KleshaContext.recentVedana → IKlesha.perceive()
→ KleshaSignalBundle { moha, drishti, mana, sneha }
→ KleshaModulatedDispatcher.computeThreshold(signals)
→ theta(t) = clamp(theta_0 + w_sneha*mu_sneha + w_mana*mu_mana, 0.3, 0.9)
emit "klesha:update"
```

#### Phase 9: Loop Control

**9a. VedanaEmergency (受蘊 → 識蘊)**:
偵測持續苦受 → thresholdBoost → ManoAggregator effectiveBaseThreshold 提升。

**9b. VitakkaWatchdog (識蘊)**:
偵測非預設齒輪停滯 → forceNextGear(defaultGear)。

**9c. 迴圈判斷**:
`stopReason === "tool_use" && toolRound < maxToolRounds` → loop back to Phase 2。
否則 → WAITING_FOR_EVENT。

### 5.3 資料流圖

```
                          ┌──────────────────────────────────┐
                          │       EventBus (跨蘊通訊管道)       │
                          └───────────────┬──────────────────┘
                                          │
┌─────────────────────────────────────────────────────────────────────────┐
│                         ExecutionLoop FSM                               │
│                                                                         │
│  Phase 1            Phase 2              Phase 3                        │
│  ┌──────────┐       ┌──────────────┐     ┌───────────────────┐         │
│  │ INPUT    │──────>│ CONTEXT      │────>│ COGNITIVE         │         │
│  │ 色蘊     │       │ ASSEMBLY     │     │ PROCESSING        │         │
│  │          │       │ 識蘊         │     │ 想蘊              │         │
│  │ IListener│       │ IGuide       │     │ IProvider.chat()  │         │
│  │ SparshEv.│       │ .getSystem   │     │ 120s timeout      │         │
│  └──────────┘       │  Prompt()    │     └────────┬──────────┘         │
│       ^             └──────────────┘              │                    │
│       │                                           v                    │
│       │                                  stopReason=end_turn?          │
│       │ <──────── yes ───────────────────┤                             │
│       │                                  no (tool_use)                 │
│       │                                  │                             │
│       │                         Phase 4  v                             │
│       │                         ┌───────────────────┐                  │
│       │                         │ GEAR ROUTING      │                  │
│       │                         │ 識蘊+想蘊          │                  │
│       │                         │ ManoAggregator    │                  │
│       │                         │ .route(ctx,       │                  │
│       │                         │  arbiters)        │                  │
│       │                         │ 100ms/arbiter     │                  │
│       │                         └────────┬──────────┘                  │
│       │                         Phase 5  v                             │
│       │                         ┌───────────────────┐                  │
│       │                         │ DELIBERATION      │                  │
│       │                         │ 識蘊              │                  │
│       │                         │ IVolition         │                  │
│       │                         │ .deliberatePlan() │                  │
│       │                         │ .deliberateAction()│                 │
│       │                         └────────┬──────────┘                  │
│       │                         Phase 6  v                             │
│       │                         ┌───────────────────┐                  │
│       │                         │ TOOL EXECUTION    │                  │
│       │                         │ 行蘊              │                  │
│       │                         │ ITool.execute()   │                  │
│       │                         │ 30s timeout       │                  │
│       │                         └────────┬──────────┘                  │
│       │                         Phase 7  v                             │
│       │                         ┌───────────────────┐                  │
│       │                         │ FEEDBACK          │                  │
│       │                         │ 受蘊              │                  │
│       │                         │ VedanaSensor      │                  │
│       │                         │ .sense()          │                  │
│       │                         │ VedanaAssessment  │                  │
│       │                         └────────┬──────────┘                  │
│       │                         Phase 8  v                             │
│       │                         ┌───────────────────┐                  │
│       │                         │ KLESHA UPDATE     │                  │
│       │                         │ 識蘊              │                  │
│       │                         │ IKlesha.perceive()│                  │
│       │                         │ threshold modulate│                  │
│       │                         └────────┬──────────┘                  │
│       │                         Phase 9  │                             │
│       │ <──── loop back ─────────────────┘                             │
│       │                                                                │
│  ┌────┴──────┐                                                         │
│  │ SAFETY    │ <── 任何階段的安全違規                                     │
│  │ LOCKOUT   │     (token 超額 / loop 超限 / error cascade)            │
│  │ (吸收態)  │     需 /reset 解除                                       │
│  └───────────┘                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

### 5.4 時序約束

| 約束 | 預設值 | 來源 | 蘊 |
|------|-------|------|-----|
| Per-arbiter evaluation timeout | 100ms | ManoAggregatorConfig.perArbiterMs | 識+想 |
| Arbiter chain total deadline | 200ms | ManoAggregatorConfig.chainMs | 識+想 |
| Plan-level deliberation | 1-3ms | IVolition contract (vijnana-clock) | 識 |
| Per-action deliberation | 0.5-1ms | IVolition contract (vijnana-clock) | 識 |
| Tool execution timeout | 30,000ms | PolicyConfig.toolTimeout | 行 |
| LLM call timeout | 120,000ms | PolicyConfig.llmTimeout | 想 |
| Max tool rounds per event | 10 | CognitionConfig.maxToolRounds | - |
| Max loop ticks per task | 50 | SafetyMonitorConfig.maxLoopTicks | - |
| VitakkaWatchdog gear max cycles | 10 | VitakkaWatchdogConfig | 識 |
| VitakkaWatchdog gear max duration | 5,000ms | VitakkaWatchdogConfig | 識 |
| VedanaEmergency sustained ticks | 5 | VedanaEmergencyConfig | 受 |
| VedanaEmergency cooldown | 10 ticks | VedanaEmergencyConfig | 受 |

### 5.5 ZOH (Zero-Order Hold) 語義

Vedana 信號在 vedana-clock 域產生，被 vijnana-clock 域 (Klesha, IVolition) 和 mano-clock 域 (ManoAggregator) 消費。ZOH 語義：

- **持續有效**: vedana snapshot 在新的 assessment 產生前保持有效
- **不插值**: 消費者讀取最近一次 snapshot，不在兩次 assessment 之間插值
- **原子性發佈**: VedanaAssessment 整體凍結後發佈，不可見部分更新

SahajaContract 的 `mutualConsistency` + `atomicPublication` 保證 ZOH 的品質。

三個時鐘域：

| 時鐘域 | 速率 | 負責蘊 | 時序預算 |
|--------|------|--------|---------|
| vedana-clock | 每次 tool execution 後 | 受蘊 | ~1ms |
| vijnana-clock | 每次 deliberation cycle | 識蘊 | 1-5ms |
| mano-clock | 每次 gear routing | 識蘊+想蘊 | 100-200ms |

---

## 六、跨蘊互動矩陣 (Cross-Skandha Interaction Matrix)

### 6.1 5x5 互動表

```
          ┌──────────┬──────────┬──────────┬──────────┬──────────┐
          │ 色 rupa  │ 受 vedana│ 想 samjna│ 行 samsk.│ 識 vijnana│
 ─────────┼──────────┼──────────┼──────────┼──────────┼──────────┤
 色 rupa  │    -     │    -     │ InputEvt │    -     │ InputEvt │
          │          │          │ → queue  │          │ → session│
 ─────────┼──────────┼──────────┼──────────┼──────────┼──────────┤
 受 vedana│    -     │ self:PID │    -     │    -     │ Vedana   │
          │          │ aggregate│          │          │ → Klesha │
 ─────────┼──────────┼──────────┼──────────┼──────────┼──────────┤
 想 samjna│ stream:  │    -     │    -     │ tool_call│ chat rsp │
          │ text→IUI │          │          │ proposal │ → routing│
 ─────────┼──────────┼──────────┼──────────┼──────────┼──────────┤
 行 samsk.│ result   │ tool rsp │    -     │    -     │ action   │
          │ → IUI    │ → sensor │          │          │ history  │
 ─────────┼──────────┼──────────┼──────────┼──────────┼──────────┤
 識 vijnana│   -     │ Klesha→  │ system   │ veto/    │ self:    │
          │          │ threshold│ prompt   │ approve  │ Klesha↔  │
          │          │          │          │          │ threshold│
 ─────────┴──────────┴──────────┴──────────┴──────────┴──────────┘
```

**互動密度**: 識蘊互動最密（出度 3 + 入度 4 = 7），行蘊最活躍的信號產生者（出度 3）。

### 6.2 直接互動 (方法呼叫)

#### 想蘊 → 行蘊: LLM proposes tool calls

`IProvider.chat()` → tool_call events → `pendingToolCalls` → `ITool.execute()`

想蘊產生意圖 (tool call proposal)，必須經過識蘊審議 (IVolition) 後才能到達行蘊。這實現了想蘊 → 識蘊 → 行蘊的因果鏈。
[程式碼: packages/core/src/execution/loop.ts#processEvent (stream 處理段)]

#### 識蘊 → 想蘊: IGuide sets system prompt

`IGuide.getSystemPrompt()` → `ChatRequest.systemPrompt` → `IProvider.chat()`

識蘊的行為約束透過 system prompt 限定想蘊的認知空間——先驗約束，在想蘊開始處理前已生效。

#### 識蘊 → 行蘊: IVolition veto/approve

`IVolition.deliberatePlan()` → filter pendingToolCalls
`IVolition.deliberateAction()` → veto/approve per tool call

識蘊對行蘊實施雙層控制閘門：Plan-level 可移除整個 tool call，Per-action 可在最後一刻 veto 單個操作。

### 6.3 間接互動 (EventBus)

#### 行蘊 → 受蘊: tool results → VedanaSensor

`tool:result` / `tool:error` → EventBus → `IVedanaSensor.sense()` → `ChannelVedana`

增益限制 `maxSamskaraVedanaGain = 0.3` 確保行蘊單次失敗不會對受蘊產生過大負面信號。

#### 受蘊 → 識蘊: VedanaAssessment → Klesha

VedanaAssessment → KleshaContext.recentVedana → IKlesha.perceive() → KleshaSignalBundle

四個 Klesha 各自從 vedana 提取不同特徵：

| Klesha | 提取特徵 | 信號處理 |
|--------|---------|---------|
| Moha | abs(valence) 平均值 | `alpha * raw + (1-alpha) * smoothed` |
| Drishti | actionHistory 重複率 | `ratio * 0.7 + abs(derivative) * 0.3` |
| Mana | 正向 valence 平均值 | `kp * value + kd * max(0, derivative)` |
| Sneha | 正向 valence 平均值 | `integral += gain * avg; integral *= exp(-lambda)` |

#### 識蘊 → 識蘊: Klesha → threshold → routing (自迴路)

```
KleshaSignalBundle → KleshaModulatedDispatcher.computeThreshold()
→ effectiveBaseThreshold → ManoAggregator.route()
```

Sneha 降低閾值（更容易接受快速 gear），Mana 提高閾值（更保守）。兩者拮抗，形成穩態。

### 6.4 三個回授迴路

#### Inner Loop — 工具結果回授

```
ITool.execute() → tool outcome → VedanaSensor.sense() → ChannelVedana
→ IKlesha.perceive() → KleshaSignalBundle → threshold modulation
→ ManoAggregator.route() → IVolition → ITool.execute() (loop)
```

每個 tool round (~30s worst case) 產生一個完整迴路 tick。

穩定性條件:
- 有界增益: maxSamskaraVedanaGain = 0.3
- 有界閾值: theta(t) in [0.3, 0.9]
- Sneha anti-windup: floor=0.10, maxLevel=0.95, 指數衰減
- Moha 飽和抑制: `delta = alphaM * ratio / (1 + betaM * currentMoha)`
- Confidence cap: maxConfidenceByGear = { 1: 0.95 }

#### Outer Loop — VedanaEmergency 閾值提升

```
sustained dukkha (intensity >= 0.8, 5 consecutive ticks)
→ thresholdBoost = 0.15
→ ManoAggregator effectiveBaseThreshold += boost
→ 更保守的 routing
→ cooldown 10 ticks → boost 歸零
```

#### Safety Loop — VitakkaWatchdog 強制恢復

```
non-default gear persistence
→ consecutiveGearCycles >= 10 OR duration >= 5s
→ emit 'vitakka:stall'
→ manoAggregator.forceNextGear(defaultGear)
→ gear:switch { gear: defaultGear }
→ watchdog.resetOnDefaultGear()
```

**三迴路協作**: 觸發條件嚴格遞增。Inner Loop 每個 tool round 運作；VedanaEmergency 需 5 consecutive ticks；VitakkaWatchdog 需 10 cycles 或 5s。

### 6.5 Model Delta 五層閾值公式

五蘊互動的量化匯聚——所有跨蘊信號匯為 ManoAggregator 的 effectiveBaseThreshold：

```
theta_final = clamp(
  theta_base                    // Layer 0: 0.6 (配置常數)
  + Delta_klesha                // Layer 1: w_sneha*mu_sneha + w_mana*mu_mana (識蘊)
  + Delta_audit                 // Layer 2: IConfidenceAuditor LLM +/- 0.05 (未實作, Plan29+)
  + Delta_loopQuality            // Layer 3: LoopQualityVector (未實作, Plan29+)
  + Delta_vedana_emergency      // Layer 4: 0.15 sustained dukkha (受蘊→識蘊)
  + Delta_risk,                 // Layer 5: riskDelta[category] (想蘊→識蘊)
  0.30,                         // floor
  0.90                          // ceiling
)
```

| Layer | 來源蘊 | v0.28.0-alpha 實作狀態 |
|-------|--------|----------------------|
| 0. Base (0.6) | 配置 | `ManoAggregatorConfig.baseThreshold` |
| 1. Klesha | 識蘊 | `KleshaModulatedDispatcher.computeThreshold()` |
| 2. ConfidenceAuditor | 識蘊 | 未實作 (Plan29+, FC-31) |
| 3. LoopQuality | 識蘊 | 未實作 (Plan29+, FC-32) |
| 4. VedanaEmergency | 受蘊→識蘊 | `checkVedanaEmergency()` |
| 5. Risk | 想蘊→識蘊 | `computeAdjustedThreshold()` |

---

## 七、架構完備性評估 (Architecture Assessment)

### 7.1 五根介面充分性

**D3-R1 (20/20 全票)**: 五個根介面足夠。

驗證方法：

1. **型別代數完備性** (BABBAGE): PluginHooks 中每個陣列欄位的元素型別，都 extends 某個根介面或在 PluginManifest 中可宣告對應 skandha。Commands 是唯一不歸屬特定蘊的 hook，但它是 bypass ExecutionLoop 的基礎設施。

2. **微核心映射完備性** (KERNEL): I/O 管理 (色蘊) + 感測 (受蘊) + 計算引擎 (想蘊) + 動作執行 (行蘊) + 排程策略 (識蘊) = 微核心五子系統。

3. **功能覆蓋率** (LINNAEUS): 所有 plugin hook 均已歸屬至少一個蘊或標記為跨蘊基礎設施，不存在無法歸類的 hook。

### 7.2 已知缺口

#### 缺口 1: 三個弱繼承介面

IVedanaSensor、IGearArbiter、IKlesha 未顯式 extends 對應根介面。`isSkandha()` 對這些介面的實例不生效。

**評估**: 有意識的設計 trade-off。蘊歸屬透過 PluginManifest 的 `skandha` 欄位在 plugin 層級宣告。Cycle 02-4 D1 已裁定 IGearArbiter 採方案 (B)，DC-12 已裁定 IKlesha 屬識蘊。

#### 缺口 2: VedanaAssessment 佈線不完整

IVolition 的 vedana getter 目前回傳靜態 neutral 值 (全零/upekkha)。VedanaSensor plugin 可註冊，但 ExecutionLoop 尚未完整消費 VedanaRegistry 的即時資料。

**評估**: Plan26 已完成 registry 佈線。完整的 sensor→assessment→klesha→threshold 迴路待後續 Plan 實作。

#### 缺口 3: IConfidenceAuditor / ILoopQualityMonitor 未實作

Model Delta 五層公式中的 `Delta_audit` 和 `Delta_loopQuality` 在 v0.28.0-alpha 中均為 0。

**評估**: 已在路線圖中 (Plan29+)。FC-31 ~ FC-33 前置條件部分已滿足。

### 7.3 演化路線

| Plan | 新增能力 | PluginHooks slot |
|------|---------|-----------------|
| Plan29+ | ILoopQualityMonitor (monitors) | `hooks.monitors?: IMonitor[]` |
| Plan29+ | IConfidenceAuditor interface | 待設計 |
| Cycle 02-5 | Lyapunov 參數校準 (FC-33) | N/A (理論工作) |
| Cycle 02-5 | Moha/Sneha 耦合模擬 (FC-34) | N/A (模擬工作) |

D3-R4b: DC-6「行蘊開放不鎖定」繼續有效。未來有工程需求時可在 ISamskara 下新增子介面。

**識蘊膨脹監控** (PENROSE 建議，D3-R1 附帶): 目前識蘊有 4 個子介面 (IGuide, IVolition, IGearArbiter, IKlesha)，加上 IConfidenceAuditor 和 ILoopQualityMonitor 將達 6 個。維持在合理範圍內，但需持續監控。

---

## 八、附錄

### 附錄 A: EventBus 完整事件目錄

| # | 事件 | 來源蘊 | 目標蘊 | 觸發條件 |
|---|------|--------|--------|---------|
| 1 | `input:received` | 色 | 識 (queue) | 外部輸入到達 |
| 2 | `sparsha:contact` | 色+識 | 全部 | 每次 processEvent |
| 3 | `loop:assembling_context` | 識 | - (observability) | Context assembly 開始 |
| 4 | `loop:awaiting_llm` | 想 | - (observability) | LLM 呼叫開始 |
| 5 | `stream:text_delta` | 想 | 色 (IUI) | LLM 產生文字 |
| 6 | `stream:tool_call_*` | 想 | 行 (pending) | LLM 提出工具呼叫 |
| 7 | `stream:finish` | 想 | - | LLM 串流結束 |
| 8 | `message:assistant` | 想 | - | 完整 assistant 訊息 |
| 9 | `gear:arbiter_evaluated` | 想+識 | - (observability) | 每個 arbiter 評估完成 |
| 10 | `gear:switch` | 識 | 識 (watchdog) | Gear routing 決定 |
| 11 | `volition:deliberation` | 識 | - (observability) | Plan-level 審議完成 |
| 12 | `volition:veto` | 識 | 行 (skip) | Per-action 否決 |
| 13 | `action:proposed` | 行 | - (observability) | 工具即將執行 |
| 14 | `tool:executing` | 行 | - (observability) | 工具開始執行 |
| 15 | `tool:result` | 行 | 受 (sensor) + 色 (IUI) | 工具成功 |
| 16 | `tool:error` | 行 | 受 (sensor) + 色 (IUI) | 工具失敗 |
| 17 | `action:executed` | 行 | - (observability) | 工具執行完成 |
| 18 | `vedana:assessment` | 受 | 識 (klesha) | Vedana 評估完成 |
| 19 | `vedana:channel_update` | 受 | 識 | 單通道更新 |
| 20 | `klesha:update` | 識 | 識 (self) | Klesha 信號更新 |
| 21 | `coarising:bundle` | 跨蘊 | 全部 | 五遍行 bundle 組裝完成 |
| 22 | `vitakka:stall` | 識 | 識 (ManoAggregator) | Watchdog 觸發 |
| 23 | `safety:lockout` | 識 (safety) | 色 (IUI) | 安全斷路器觸發 |
| 24 | `safety:warning` | 識 (safety) | 想 (inject prompt) | 行為警告 |

### 附錄 B: 蘊介面繼承完整圖

```
┌─────────────────────────────────────────────────────────────────────────┐
│                          aggregates.ts                                  │
│  Skandha = 'rupa' | 'vedana' | 'samjna' | 'samskara' | 'vijnana'       │
│                                                                         │
│  IRupa ──── IVedana ──── ISamjna ──── ISamskara ──── IVijnana           │
└──┬─────────┬───────────┬──────────┬──────────────┬──────────────────────┘
   │         │           │          │              │
   │ extends │           │ extends  │ extends      │ extends
   v         │           v          v              v
┌──────┐     │      ┌─────────┐  ┌───────┐   ┌─────────┐
│IList-│     │      │IProvider│  │ITool  │   │IGuide   │
│ener  │     │      │  .ts    │  │<T>.ts │   │  .ts    │
│ .ts  │     │      └─────────┘  └───────┘   └─────────┘
└──────┘     │                                    │ extends
             │                               ┌────┴────┐
┌──────┐     │                               │IVolition│
│ IUI  │     │                               │  .ts    │
│ .ts  │     │                               └─────────┘
└──────┘     │
  extends    │
             │ [弱繼承: 不 extends IVedana]
             v
        ┌───────────┐
        │IVedana    │
        │Sensor .ts │
        └───────────┘

[弱繼承: 獨立介面, manifest skandha: ['samjna','vijnana']]
┌──────────────┐
│IGearArbiter  │
│gear-arbiter  │
│  .ts         │
└──────────────┘

[弱繼承: 獨立介面, DC-12 裁定屬識蘊]
┌──────────────┐
│IKlesha       │
│klesha.ts     │
├──────────────┤
│Moha  │Drishti│
│Mana  │Sneha  │
└──────────────┘
```

### 附錄 C: 決議索引

| 決議編號 | 內容 | 票數 | 影響 |
|---------|------|------|------|
| D3-R1 | 五個根介面足夠 | 20/20 | 型別階層穩定 |
| D3-R2 | PluginHooks 到蘊映射正確 | 20/20 | 映射表定案 |
| D3-R2a | SlashCommand 不歸入蘊的理由記載 | 附帶 | bypass ExecutionLoop |
| D3-R2b | SlashCommand bypass SafetyMonitor 安全觀察 | 附帶 | 安全文件 |
| D3-R3 | skandha 是 metadata，非路由依據 | 18/20 | PluginLoader 不變 |
| D3-R4 | ISamskara 維持 ITool 為唯一子介面 | 20/20 | 行蘊穩定 |
| D3-R4a | 行蘊窄化與佛學差異分析記錄 | 附帶 | 見下 |
| D3-R4b | DC-6「行蘊開放不鎖定」繼續有效 | 附帶 | 演化保留 |
| D3-R5 | 十二因緣選擇性映射 (附錄, 功能性類比) | 13/20 | 見附錄 D |
| D3-R6 | 認知序列選擇性映射 (附錄, 結構平行) | 17/20 | 見附錄 E |

**少數意見存檔**:
- D3-R3: GUARDIAN (#11), LINNAEUS (#13) 支持提案 B（一致性驗證 warning）
- D3-R5: VITRUVIUS, MESH, BABBAGE, KERNEL, GUARDIAN, ARCHIMEDES, TURING 支持不映射
- D3-R6: MESH, GUARDIAN, TURING 支持不映射

### 附錄 D: 十二因緣功能性類比 (D3-R5)

> **注意**: 以下為功能性類比 (functional analogy)，非正式映射 (formal mapping)。十二因緣描述三世因果（宏觀存有論），ExecutionLoop 描述單一認知迴圈（微觀運算結構），兩者尺度不同。

十二因緣「觸→受→愛→取」鏈與 ExecutionLoop 的部分功能性平行：

| 十二因緣 | ExecutionLoop | 功能平行 |
|---------|---------------|---------|
| 觸 (sparsha) | SparshEvent | 根+境+識三條件交會 |
| 受 (vedana) | ChannelVedana | 苦/樂/捨感受信號 |
| 愛 (tanha) | Sneha (IKlesha) | 對愉悅結果的依附 (integrator) |
| 取 (upadana) | 重複模式 | 模式固著 (Drishti band-pass) |

僅上述四環有實質平行。其餘八環（無明、行、識、名色、六入、有、生、老死）在 ExecutionLoop 中無精確對應，不強行映射。

### 附錄 E: 認知序列結構平行 (D3-R6)

> **注意**: 以下為結構平行 (structural parallel)，非正式等價。認知序列 (citta-vithi) 與 ExecutionLoop 描述相同尺度的現象——一次認知活動的內部階段——因此平行精確度較高。PENROSE 觀察：此結構收斂是「描述認知迴圈所需的最小結構」的功能需求必然結果，非刻意模仿。

| 認知序列 | 功能 | ExecutionLoop 狀態 | 平行程度 |
|---------|------|-------------------|---------|
| 有分 (bhavanga) | 心識休息態 | WAITING_FOR_EVENT | 高 |
| 引轉 (avajjana) | 注意力轉向刺激 | ASSEMBLING_CONTEXT | 高 |
| 五識 (panca-vijnana) | 初步感知 | SparshEvent | 中高 |
| 領受 (sampaticchana) | 接收感知結果 | LLM stream 接收 | 中 |
| 推度 (santirana) | 初步分析 | PROCESSING_RESPONSE | 中 |
| 確定 (votthapana) | 確定性質 | Gear Routing | 中高 |
| 速行 (javana) | 行動/造業 | EXECUTING_TOOLS | 高 |
| 彼所緣 (tadarammana) | 結果登錄 | FEEDBACK + Klesha 更新 | 中高 |

### 附錄 F: 行蘊窄化設計決策 (D3-R4a)

OpenStarry 的行蘊 (ISamskara) 被窄化為「外在可觀察的行動」(ITool)，而傳統佛學中行蘊覆蓋 51 心所中除受、想外的 49 個心所，包含：

- **思心所 (cetana)**: OpenStarry 中對應 IVolition → 歸入識蘊 (extends IVijnana)
- **煩惱心所**: OpenStarry 中對應 IKlesha → 歸入識蘊 (DC-12 裁定)

此窄化是有意識的架構決策：

1. **IVolition 的審議功能**在 ExecutionLoop 的 Phase 5 (Deliberation) 執行——在工具執行 (Phase 6) 之前，屬於「決策」而非「行動」。歸入識蘊使 Phase 4→5→6 的蘊流轉為 識→識→行，概念上更自然。

2. **IKlesha 的感知功能** (`perceive()`) 更接近意識的自我監察而非外在行動。

3. **工程清晰度**: 在 OOP 中，ITool 的 `execute()` 和 IVolition 的 `deliberatePlan()` 有截然不同的語義和時序特性（30s vs 3ms）。強制置於同一蘊會模糊邊界。

DC-6 裁定行蘊保持開放，未來若有工程需求可新增子介面。

---

*[文件結束]*

---

## Cycle 02-6: ISamskara Deep Dive Results

本節記錄 Cycle 02-6 對行蘊 (ISamskara) 的深度研究成果，以及 AuditContext 整合設計。

### 行蘊核心定義 [D1-R1, 20/20]

Cycle 02-6 D1 從原始經典重新定義行蘊：

- **cetanā 中心性**: 六種思身（SN 22.56）= 行蘊定義，非唯識「受想以外的心所集合」
- **造作一切有為法**: 行蘊造作所有五蘊的被條件化狀態 (SN 22.79)
- **無核心的動態過程**: 芭蕉喻，層層剝開無堅實核心 (SN 22.95)

### 行蘊屬性判定三準則 [D1-R5, 20/20]

永久工具，判定某功能是否屬行蘊：

1. **造作性**: 是否創造/改變/產生新狀態？
2. **意圖驅動**: 是否由 cetanā 驅動？
3. **環境改變**: 是否改變後續認知條件？

核心區分：**行蘊 = WRITE（主動改變）; 識蘊 = READ（被動評估）**

此區分直接對應 §二 型別階層中 ISamskara（ITool, ISlashCommand — 執行/造作）與 IVijnana（IGuide, IVolition — 評估/引導）的分工。

### ISamskara 拓展方向 [D1-R3, 20/20]

目前不新增子介面（延續 Cycle 02-5 D3-R4）。四個方向存檔為長期候選：

| 方向 | 內容 | 時程 |
|------|------|------|
| A | 意圖規劃 (cetanā-formation) | Cycle 03+ |
| B | 習氣形成 (vāsanā-imprinting) | 長期 |
| C | 環境轉換 (kāya extension) | 無需排程 |
| D | 溝通形成 (vacī) | 無需排程 |

### 蘊歸屬永久原則 [D1-R6, 20/20]

五項永久原則適用於所有未來蘊歸屬判定：

1. 功能分析為唯一依據
2. 心所系統作參考，不作歸屬依據
3. 梵文命名限原始經典
4. plugin ≠ 心所，可跨多蘊
5. 既有歸屬決議有效

附帶 **Master's 8-point cetasika naming rules**（詳見 Doc 41 Cycle 02-6 New Decisions 節）。

### AuditContext 整合設計 [D2-R1, 20/20]

Cycle 02-6 D2 設計了 AuditContext type，使 IConfidenceAuditor (IVijnana) 接收豐富的決策上下文：

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

extras bag 協議（D2-R2, 19/20）建立了 Plugin 間資料共享的通用管道，使 §五 執行流中的 LoopQualityMonitor（vedana+samjna+vijnana）能將品質資訊傳遞給 IConfidenceAuditor（vijnana）——跨蘊的資料流完全透過 EventBus + extras 實現，不破壞微核心的解耦原則。

完整 AuditContext 規格見 **Doc 46**。

*本文件基於 Cycle 02-5 R1 獨立研究和 D3 辯論撰寫，Cycle 02-6 新增 ISamskara 深度研究與 AuditContext 整合成果。所有決議經 20 位學者投票確認。*
