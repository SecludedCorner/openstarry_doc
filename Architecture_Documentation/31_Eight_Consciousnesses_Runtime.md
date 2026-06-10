<!-- Status: CURRENT -->
<!-- Layer: 3-Research -->
<!-- Applies to: v0.34.0-alpha -->
<!-- Last verified: 2026-03-16 -->

# 31. 八識俱轉運行機制 (Eight Consciousnesses Runtime Model)

> Master 文件需求 #3: 八識俱轉的工程映射
> Cycle 02-3 整合版

---

## 1. 經典依據

### 《解深密經》

> 「廣慧！阿陀那識為依止、為建立故，六識身轉，謂眼識，耳、鼻、舌、身、意識。
> 在此中，或有一識暫時而轉，或二、或三、或四、或五，或六識身一時俱轉。
> 譬如大河，若有一浪生緣現前，唯一浪轉；若二、若多生緣現前，多浪隨轉。
> 而此大河，自體恆流，無有斷絕。」

### 大河比喻的工程解讀

| 佛學概念 | 工程映射 | 說明 |
|----------|---------|------|
| 大河自體恆流 | AgentCore EventBus + ExecutionLoop | 基礎設施持續運行，不因個別識停轉而中斷 |
| 一浪轉 | 單一 IListener 通道活躍 | 單一感官輸入（如僅 HTTP） |
| 多浪俱轉 | 多 IListener 同時觸發 | WebSocket + HTTP + MCP 同時有事件 |
| 浪生緣現前 | InputEvent 到達 | 因緣具足（事件到達）時識轉起 |
| 阿陀那識為依止 | AgentCore 作為基底 | 所有識的轉起都依託 Core |

---

## 2. 八識工程映射

### 2.1 八識對照表

| 八識 | 梵文 | 五蘊歸屬 | 工程映射 | 運行特性 |
|------|------|---------|---------|---------|
| 眼識 | Caksur-vijnana | 色蘊 | Visual IListener (Web UI, Dashboard) | 事件驅動 |
| 耳識 | Srotra-vijnana | 色蘊 | CLI/WebSocket IListener | 事件驅動 |
| 鼻識 | Ghrana-vijnana | 色蘊 | Environmental IListener (Monitoring) | 輪詢/事件 |
| 舌識 | Jihva-vijnana | 色蘊 | Data Quality IListener (Validation) | 事件驅動 |
| 身識 | Kaya-vijnana | 色蘊 | FS/Haptic IListener (File Watcher) | 事件驅動 |
| 意識 | Mano-vijnana | 想蘊 | ExecutionLoop + IProvider.chat() | 按需（最慢） |
| 末那識 | Manas | 識蘊 | IGuide + EgoFramework | 恒審思量（持續） |
| 阿賴耶識 | Alaya-vijnana | 跨層 | AgentCore + 協調層 (纖維叢投影) | 恆流不斷 |

### 2.2 俱轉機制

```
阿賴耶識 (AgentCore)
  │ 恆流不斷 — EventBus + ServiceRegistry 持續運行
  │
  ├─ 末那識 (EgoFramework)
  │   │ 恒審思量 — 每個 tick 檢查行為約束
  │   │ 四煩惱常俱 — Moha/Drishti/Mana/Sneha 同時運作
  │   │
  │   └─ constrains all actions
  │
  ├─ 意識 (ExecutionLoop + Provider)
  │   │ 按需轉起 — LLM 調用時啟動
  │   │ 最慢的識 — 推理需要時間
  │   │
  │   └─ 歸依意：路由所有前五識的結果至齒輪選擇
  │
  └─ 前五識 (IListener channels)
      │ 事件驅動 — 有輸入時轉起，無輸入時不轉
      │
      ├─ 眼識 channel: Web UI events
      ├─ 耳識 channel: CLI/WebSocket messages
      ├─ 鼻識 channel: Environment monitoring
      ├─ 舌識 channel: Data quality checks
      └─ 身識 channel: File system events
```

---

## 3. 俱轉的工程實現

### 3.1 並行處理模型

```typescript
/**
 * 八識俱轉的核心特性：
 * 1. 前五識可以 1~5 個同時轉起（大河多浪）
 * 2. 意識在需要時轉起（歸依意路由）
 * 3. 末那識恆常運作（恒審思量）
 * 4. 阿賴耶識是基底（恆流不斷）
 */

// 前五識的並行事件處理
interface ConcurrentSenseProcessing {
  /** 當前活躍的感官通道 */
  readonly activeChannels: readonly string[];

  /** 各通道的 CoarisingBundle（觸→俱生結果） */
  readonly channelResults: ReadonlyMap<string, CoarisingBundle>;

  /** 是否需要意識介入（歸依意） */
  readonly requiresManoProcessing: boolean;
}
```

### 3.2 大河模型：EventBus 作為恆流

```typescript
/**
 * EventBus = 阿賴耶識的「恆流」
 *
 * 不管有多少浪（事件），河（EventBus）本身不斷。
 * 單一事件 = 一浪轉
 * 多事件同時 = 多浪俱轉
 * 無事件 = 河仍流（idle loop）
 */
interface AlayaFlow {
  /** 恆流 — EventBus 持續監聽，永不停止 */
  readonly eventBus: EventBus;

  /** 種子 — 潛在能力（已載入的 plugins），未啟用不消耗 */
  readonly seeds: ReadonlyMap<string, IPlugin>;

  /** 現行 — 當前活躍的處理流 */
  readonly manifestations: readonly ActiveProcess[];
}
```

### 3.3 末那識的恒審思量

```typescript
/**
 * 末那識 = EgoFramework 的持續運作
 *
 * 「第二能變識，是識名末那；依彼轉緣彼，思量為性相」
 * 末那識的特性是「恒審思量」——不間斷地自我參照。
 *
 * 四煩惱常俱：
 * - 我癡 (Moha): 系統盲點，無法自覺的限制
 * - 我見 (Drishti): 自我認同的執取
 * - 我慢 (Mana): 能力過度自信
 * - 我愛 (Sneha): 自我保護傾向
 *
 * 這四者「常俱」——在識運行時始終同時運作。
 */
interface ManasRuntime {
  /** 恒審思量 — 每個 ExecutionLoop tick 都檢查 */
  checkAction(proposed: ProposedAction): ActionDecision;

  /** 四煩惱常俱 — DI 注入的煩惱同時運作 */
  readonly kleshas: {
    moha: Moha;      // 我癡
    drishti: Drishti; // 我見 (contains IIdentity)
    mana: Mana;      // 我慢
    sneha: Sneha;    // 我愛
  };
}
```

---

## 4. 俱轉的時間性

### 4.1 多時鐘架構 (M-10)

> (Master): 「OpenStarry 的微核需要一套自己的『時間感』——不同層級的運作有不同的速度，不是所有東西都跑在同一個時鐘上。」

| 層級 | 對應識 | 時鐘速度 | 工程實現 |
|------|--------|---------|---------|
| 最快 | 阿賴耶 (基底) | 持續 | EventBus heartbeat |
| 快 | 末那 (我執) | 每 tick | ExecutionLoop tick callback |
| 中 | 前五識 (感官) | 事件驅動 | IListener event handlers |
| 中 | 行蘊 (執行) | 按需 | ITool.execute() |
| 最慢 | 意識 (認知) | 按需 | IProvider.chat() (LLM call) |

### 4.2 諸受俱起

> 《大毘婆沙論》「有時諸受，或是俱起，或不俱起。若俱起者，一剎那頃，多受現前。」
> 《雜阿含經》「受苦痛受，生憂悲稱怨……是名凡夫二受俱起，謂身受、心受。」

多個感官通道的受可以同時存在：
- HTTP 通道收到錯誤 → 苦受
- WebSocket 通道收到成功 → 樂受
- 兩者同時存在，歸依意路由後產生「意受」

```
channel_A: vedana = dukkha (error response)  ─┐
channel_B: vedana = sukha (success msg)      ─┤→ ManoAggregator (純路由) → 意受
channel_C: vedana = upekkha (idle)           ─┘
```

---

## 5. 與觸→俱生模型的整合

八識俱轉 + 觸→俱生 = 完整的運行時模型：

```
┌─────────────────────────────────────────────┐
│ 阿賴耶識 (AgentCore — 恆流)                    │
│                                             │
│  ┌──────────────────────────────────────┐   │
│  │ 末那識 (EgoFramework — 恒審思量)       │   │
│  │ 四煩惱常俱: Moha·Drishti·Mana·Sneha  │   │
│  └──────────────┬───────────────────────┘   │
│                 │ constrains                │
│  ┌──────────────┼──────────────────────┐    │
│  │ 前五識 (IListener channels — 俱轉)    │    │
│  │                                     │    │
│  │  channel_1: 根+境+識 → 觸 → 俱生     │    │
│  │  channel_2: 根+境+識 → 觸 → 俱生     │    │
│  │  channel_N: ...                     │    │
│  └──────────────┬──────────────────────┘    │
│                 │ 歸依意                     │
│  ┌──────────────┼──────────────────────┐    │
│  │ 意識 (ExecutionLoop + Provider)      │    │
│  │                                     │    │
│  │  意觸 → 意受·意想·意思               │    │
│  │       → 行蘊執行 (身行/語行/意行)     │    │
│  └──────────────┬──────────────────────┘    │
│                 │                           │
│                 ▼                           │
│          色蘊變化 → 新循環                    │
└─────────────────────────────────────────────┘
```

---

## 6. 阿賴耶識纖維叢分佈 (Alaya-vijnana Fiber Bundle Distribution)

> [Cycle 02 Debate 3 整合] BABBAGE 形式化 + NAGARJUNA 哲學註解

### 6.1 世俗分位聲明

> 「阿賴耶識是**一個**識，顯現在兩個架構層級。協調層和 AgentCore 是同一藏識的投影（世俗分位, conventional partitions），IPC 協議是確保連續性的轉移函數。此分佈是工程方便設定 (prajnapti)，不是本體論上的切割。」
> — BABBAGE 形式化, NAGARJUNA 核可

### 6.2 三藏對照

| 面向 | 位置 | 內容 | 工程映射 |
|------|------|------|---------|
| 能藏 (active storage capacity) | 協調層 | 能力註冊表、Plugin 解析 | PluginRegistryService |
| 所藏 (stored content) | 雙層 | 協調層=系統設定；AgentCore=運行時狀態 | ConfigStore + StateManager |
| 執藏 (ego-appropriated) | AgentCore | Guide 綁定、Identity 執取 | IGuide + IIdentity |

### 6.3 Cocycle 條件 (BABBAGE)

IPC 來回必須保存資訊——雙向一致性檢查：

```
coordinationLayer.getCapability(x)
  → agentCore.loadPlugin(x)
    → agentCore.reportCapability(x)
      → coordinationLayer.verifyCapability(x) == x
```

此 cocycle 條件確保「種子」在兩個層級之間的投射不會遺失資訊。

### 6.4 哲學警語 (NAGARJUNA)

1. 纖維叢模型是「方便施設」——不要把數學工具誤認為本體論真理
2. 能/所/執三藏的區分是功能性的，不是實體性的
3. 最強的對應是 Core 的字面空性（無內建能力）——這是真正的緣起性空

---

## 7. 分階段實作

### Phase 1 (Plan26 近期): 單意識模型
- 現有 ExecutionLoop = 意識的簡化版
- 前五識統一為單一 IListener 介面
- 末那識 = EgoFramework 基礎約束

### Phase 2 (中期): 多通道觸
- 每個 IListener 通道獨立產生 CoarisingBundle
- ManoAggregator 路由多通道結果至 IGearArbiter plugin / IProvider plugin
- 末那識四煩惱 DI 機制

### Phase 3 (遠期): 完整八識俱轉
- 大河模型完整實現
- 多時鐘架構
- 諸受俱起 + 歸依意完整迴路
- 阿賴耶識 = 跨 Agent 協調層

---

*Master 文件需求 #3 完成。*
