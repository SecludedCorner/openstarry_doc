# 16. 插件類型哲學總結圖表 (Plugin Types Philosophical Mapping)

本文件匯總了 OpenStarry 架構中插件類型與「五蘊 (Pañcaskandha)」的映射關係。

> **Cycle 02-3 整合版**：整合 M-1 梵文命名裁定、A-2 IVijnana 子介面擴展、A-3 觀察者組合修正、M-7 多值 skandha 支援。

## 1. 核心觀點：緣起性空

*   **Agent Core (空):** 體現「緣起性空」——無自性，但具備使五蘊聚合運作的潛能。
*   **Plugins (有):** 五蘊聚合賦予 Agent 生命的五個維度。蘊與 Core 相互依存。

## 2. 五蘊映射圖表 (The Mapping Chart)

### 根介面與子介面 (M-1 梵文命名)

| 根介面 | 子介面 | 蘊 | 梵文 | 定義 | 系統職責 | 代表性組件 |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **IRupa** | **IUI** | **色蘊** (輸出) | Rūpa | 物質顯相——向外呈現 | 定義 Agent 的呈現方式 | Dashboard, CLI, Web UI |
| **IRupa** | **IListener** | **色蘊** (輸入) | Rūpa | 物質感官——向內接收 | 定義 Agent 接收資訊的通道 | HTTP, WebSocket, MCP |
| **IVedana** | *(三受感測器)* | **受蘊** | Vedanā | 三受反饋：苦·樂·捨 | 情感性評價機制 | VedanaPlugin (PID 控制) |
| **ISamjna** | **IProvider** | **想蘊** | Saṃjñā | 認知處理全光譜 | 認知引擎 | Gemini, Claude, ChatGPT |
| **ISamjna** | **IInferenceProvider** | **想蘊** | Saṃjñā | 推理能力 | 非互動式推理 | LM Studio, Ollama |
| **ISamskara** | **ITool** | **行蘊** (身行) | Saṃskāra | 外部工具操作 | 對外部世界產生影響 | FS, API, Code Exec |
| **ISamskara** | **ISlashCommand** | **行蘊** (語行) | Saṃskāra | 指令 | 語言行為 | /help, /provider |
| **ISamskara** | *(意行)* | **行蘊** (意行) | Saṃskāra | 內部狀態變化 | 造作引擎 | State updates, Workflow |
| **IVijnana** | **IIdentity** | **識蘊** | Vijñāna | 自我認同 | 「我是誰」 | Agent ID, Name |
| **IVijnana** | **IGuide** | **識蘊** | Vijñāna | 行為引導 | 「我應該怎麼做」 | Markdown Skills, MCP Logic |
| **IVijnana** | **IVolition** | **識蘊** | Vijñāna | 意志/動機 (我執) | 煩惱驅動行為引導 | EgoFramework |
| **IVijnana** | **IReflection** | **識蘊** | Vijñāna | 自省 | 深度自我評估 | Pattern C Observer (future) |

### 命名對照 (M-1 裁定)

| 舊名 (v0.24.0-beta) | 新名 (v0.25.0+) | 蘊 |
|---------------------|----------------|---|
| ISensory | **IRupa** | 色蘊 |
| ISensation | **IVedana** | 受蘊 |
| ICognition | **ISamjna** | 想蘊 |
| IAction | **ISamskara** | 行蘊 |
| IIdentity | **IVijnana** | 識蘊 |

## 3. 多值 Skandha (M-7 裁定)

Plugin 可以同時具備多種 skandha：

```typescript
interface PluginManifest {
  name: string;
  skandha: Skandha | Skandha[];  // 單值或多值
}
```

### 22 個 Plugin 的五蘊歸屬

| Plugin | skandha | 說明 |
|--------|---------|------|
| devtools | `['samskara']` | 工具 |
| guide-character-init | `['vijnana']` | 引導 |
| http-static | `['rupa']` | 靜態服務 |
| mcp-client | `['rupa']` | MCP 客戶端 |
| mcp-server | `['rupa']` | MCP 伺服器 |
| provider-chatgpt | `['samjna']` | ChatGPT 認知 |
| provider-claude | `['samjna']` | Claude 認知 |
| provider-gemini | `['samjna']` | Gemini 認知 |
| provider-gemini-oauth | `['samjna']` | Gemini OAuth 認知 |
| provider-lmstudio | `['samjna']` | LM Studio 認知 |
| provider-local-llama | `['samjna']` | Ollama 認知 |
| standard-core-commands | `['samskara']` | 核心指令 |
| standard-function-fs | `['samskara']` | 檔案系統 |
| **standard-function-skill** | **`['samskara', 'vijnana']`** | **工具+引導 (多值)** |
| standard-function-stdio | `['samskara']` | 標準 I/O |
| standard-model-selector | `['samskara']` | 模型選擇 |
| transport-http | `['rupa']` | HTTP 傳輸 |
| transport-websocket | `['rupa']` | WebSocket 傳輸 |
| tui-dashboard | `['rupa']` | TUI 儀表板 |
| web-ui | `['rupa']` | Web UI |
| workflow-engine | `['samskara']` | 工作流引擎 |

## 4. Klesha 與心所映射 (M-3)

### 四根本煩惱 (注入 IVijnana)

| 梵文 | 中文 | 英語 | 工程含義 |
|------|------|------|---------|
| Moha | 我癡 | Delusion | 自我盲點，無法看到系統缺陷 |
| Drishti | 我見 | Self-View | 自我認同的執取，IIdentity 注入其中 |
| Mana | 我慢 | Pride | 過度自信傾向 |
| Sneha | 我愛 | Self-Attachment | 自我保護傾向 |

### 心所映射 (M-4)

| 子介面 | 心所 | 類型 | 作用 |
|--------|------|------|------|
| IGuide | 作意 (Manasikara) | 遍行 | 注意力導向 |
| IVolition | 思 (Cetana) | 遍行 | 造作決定 |
| IReflection | 尋伺 (Vitakka/Vicara) | 不定 | 邏輯解析 |

> (Master): 心所是動態開放的，不同的樣態都可以是存在的，只要跑的動。

## 5. 觀察者組合模式 (A-3 修正)

觀察者**不屬於任何蘊**，而是五蘊子類別的 Composition：

| 模式 | 名稱 | 組合 | 蘊分類 | 層次 |
|------|------|------|--------|------|
| Pattern A | SimpleObserver | IVedana | 受蘊 | 前末那感知 |
| Pattern B | AnalyticalObserver | IVedana + ISamjna | 受蘊+想蘊 | 第六識分析 |
| Pattern C | ReflectiveObserver | IVedana + ISamjna + IVijnana | 全蘊 | 第七識(末那) |

### 5.1 IObserver 正式介面

> [Cycle 02-2 C-2 整合] 觀察者不繼承任何蘊，透過 Composition 組合五蘊子類別。

```typescript
/**
 * IObserver — 觀察者介面
 * 不繼承任何蘊。透過 Composition 組合五蘊子類別。
 *
 * A-3 修正: 觀察者 ≠ 受蘊。
 * 觀察者是上層模組，受蘊是其組件之一。
 */
export interface IObserver {
  /** 語義化名稱 (如 ResourceHealthObserver) */
  readonly name: string;
  /** simple = 純受蘊 | analytical = 受蘊+想蘊 | reflective = 受蘊+想蘊+識蘊 */
  readonly pattern: ObserverPattern;
  /** 執行觀察 */
  observe(): Promise<ObservationReport>;
  /** 取得組合的蘊列表 */
  getSkandhaComposition(): Skandha[];
}

type ObserverPattern = 'simple' | 'analytical' | 'reflective';

interface ObservationReport {
  readonly timestamp: number;
  readonly observerName: string;
  readonly pattern: ObserverPattern;
  readonly vedana?: VedanaAssessment;
  readonly cognitionAnalysis?: {
    patterns: string[];
    anomalies: string[];
    classification: string;
  };
  readonly reflection?: {
    selfAssessment: string;
    confidence: number;
    suggestedChanges: string[];
  };
  readonly recommendation: ObserverRecommendation;
}

type ObserverRecommendation =
  | { action: 'continue'; confidence: number }
  | { action: 'caution'; reason: string }
  | { action: 'intervene'; details: string }
  | { action: 'escalate'; reason: string };
```

### 5.2 三種觀察者模式實作範例

**Pattern A: SimpleObserver (純受蘊)**

```typescript
class SimpleObserver implements IObserver {
  readonly pattern = 'simple' as const;
  constructor(
    readonly name: string,
    private readonly vedana: IVedana,
  ) {}
  getSkandhaComposition(): Skandha[] { return ['vedana']; }
  async observe(): Promise<ObservationReport> {
    const assessment = this.vedana.assess();
    return {
      timestamp: Date.now(),
      observerName: this.name,
      pattern: 'simple',
      vedana: assessment,
      recommendation: assessment.dukkha > 0.8
        ? { action: 'intervene', details: `High dukkha: ${assessment.dukkha}` }
        : { action: 'continue', confidence: assessment.upekkha },
    };
  }
}
```

**Pattern B: AnalyticalObserver (受蘊+想蘊)**

```typescript
class AnalyticalObserver implements IObserver {
  readonly pattern = 'analytical' as const;
  constructor(
    readonly name: string,
    private readonly vedana: IVedana,
    private readonly cognition: ISamjna,
  ) {}
  getSkandhaComposition(): Skandha[] { return ['vedana', 'samjna']; }
}
```

**Pattern C: ReflectiveObserver (受蘊+想蘊+識蘊)**

```typescript
class ReflectiveObserver implements IObserver {
  readonly pattern = 'reflective' as const;
  constructor(
    readonly name: string,
    private readonly vedana: IVedana,
    private readonly cognition: ISamjna,
    private readonly vijnana: IVijnana,
  ) {}
  getSkandhaComposition(): Skandha[] { return ['vedana', 'samjna', 'vijnana']; }
}
```

### 5.3 觀察者命名建議

| 觀察者 | Pattern | 名稱 | 功能 |
|--------|---------|------|------|
| 資源健康 | A | `ResourceHealthObserver` | CPU/記憶體/延遲 |
| 錯誤率 | A | `ErrorRateObserver` | 工具錯誤率 |
| 行為模式 | B | `BehaviorPatternObserver` | 重複行為偵測 |
| 異常分類 | B | `AnomalyClassifier` | 異常類型分類 |
| 自省 | C | `SelfReflectionObserver` | 深度自我評估 |

## 6. 八識-五蘊 雙框架慣例

- **五蘊** = **設計時分類** (Plugin 類型, 介面層級, @skandha 標記)
- **八識** = **運行時現象學** (事件生命週期, 觸→俱生迴路)

| 八識 | 梵文 | 五蘊映射 | 工程映射 |
|------|------|---------|---------|
| 眼識 | Caksur-vijnana | 色蘊 | Visual IListener |
| 耳識 | Srotra-vijnana | 色蘊 | CLI/WebSocket IListener |
| 鼻識 | Ghrana-vijnana | 色蘊 | Environmental IListener |
| 舌識 | Jihva-vijnana | 色蘊 | Data Quality IListener |
| 身識 | Kaya-vijnana | 色蘊 | Haptic/FS IListener |
| 意識 | Mano-vijnana | 想蘊 | ExecutionLoop + Provider |
| 末那識 | Manas | 識蘊 | IGuide + EgoFramework |
| 阿賴耶識 | Alaya-vijnana | 跨層 | 纖維叢投影 (協調層+AgentCore) |

---

*整合自: Cycle 02 八識映射, Cycle 02-2 五蘊擴展設計, Cycle 02-3 M-1/M-3/M-4/M-7 裁定*
