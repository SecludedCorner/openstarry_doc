# 12. 能力發現與依賴注入機制 (Capabilities Discovery & Injection)

> ⚠️ **[漂移更正 — v0.59.6 / 陳舊 API]** 本文件的 §2「核心介面定義」與 §3/§4 的「register* 注入」描述為**早期設計稿，與已交付 SDK 契約不符**。實際真相（`packages/sdk/src/types/plugin.ts:376-379`）：
> - **`IPlugin = { manifest, factory }`**，採**工廠模式**回傳能力，**不是** `{ id, version, initialize, shutdown }` 的「實作類別」。對照根目錄 `openstarry_eco/CLAUDE.md`「Factory Pattern — Plugins export `createXxxPlugin()` → `IPlugin` with `manifest` + `factory(ctx)`」。
> - **`IPluginContext`（`plugin.ts:234-273`）沒有 `registerTool` / `registerListener` / `setLLMProvider`** 等「主動推送」方法。能力是由 `factory(ctx)` **回傳** `PluginHooks`（`plugin.ts:276-308`，含 `tools` / `listeners` / `providers` / `vedanaSensors` …）給 Loader，而非插件反向呼叫 context 註冊。context 只提供 `bus` / `pushInput` / `sessions` 與惰性唯讀存取器（`tools` / `guides` / `providers` / `services`）。
> - **`IListener` 屬「色蘊（rupa）」非「受蘊（vedana）」**。`listener.ts:1-13`：`IListener extends IRupa`，JSDoc 標 `@skandha rupa (色蘊 — 感官根·輸入)`；`aggregates.ts:12-25` 確認色蘊涵蓋 IListener+IUI。受蘊（vedana）是獨立蘊（`aggregates.ts:41-44`），由 `PluginHooks.vedanaSensors`（`plugin.ts:283`，Plan26 `IVedanaSensor`）承載。本文 §2.2 把 `registerListener` 標成「[受]」是雙重錯誤（既無此方法，且 listener 對應色而非受）。
>
> 下方原始 §2/§3/§4 散文保留作**歷史設計稿**；正確契約見本檔末新增的「附錄 A：實際交付的 SDK 契約（v0.59.x）」。`@openstarry/sdk` 為 API 最高權威——任何衝突以 SDK 為準。

本文件定義了 OpenStarry 協調層 (Coordination Layer) 如何加載插件，並將其提供的功能注入到 Agent Core 的各個子系統中。這是實現「微內核架構」與「動態擴展」的關鍵機制。

## 1. 機制概述 (Overview)

我們採用 **控制反轉 (IoC)** 與 **依賴注入 (DI)** 模式。

*   **Host (The Driver):** 這是啟動 Agent 的宿主進程（CLI 或 Daemon）。**唯有 Host 具備物理 I/O 權限**，能讀取磁碟上的插件檔案。
*   **PluginLoader (The Injector):** 運行於 Host 語境中，負責將物理檔案轉化為內核對象。
*   **Core (The Recipient):** 純淨的接收者，坐享其成地獲取注入的能力。

> **💡 設計哲學 (Design Philosophy):**
> 這對應了「五蘊」中的「色不異空」。Plugin (色) 是具體功能的聚合，Core (空) 是純粹的執行容器。Loader 的職責就是打破聚合，將能力填入容器。

---

## 2. 核心介面定義 (Core Interfaces)

> ⚠️ **[漂移更正 — v0.59.6]** 以下 §2.1 / §2.2 兩段介面與 `register*` 描述為**陳舊設計稿，與已交付 SDK 不符**。正確契約見[附錄 A](#附錄-a實際交付的-sdk-契約-v059x)。保留原文供歷史對照。

所有的交互都基於 `@openstarry/sdk` 中定義的契約。

### 2.1 插件介面 (The Plugin Contract)

每個插件必須導出一個實現此介面的類別。

```typescript
export interface IPlugin {
  readonly id: string;
  readonly version: string;
  
  /**
   * 初始化插件。
   * @param context - 協調層提供的註冊上下文，用於回傳能力。
   */
  initialize(context: IPluginContext): Promise<void>;
  
  /**
   * 資源清理 (如關閉 WebSocket 連線)。
   */
  shutdown(): Promise<void>;
}
```

### 2.2 註冊上下文 (The Registration Context)

這是協調層暴露給插件的 API，用於收集能力。

```typescript
export interface IPluginContext {
  // 基礎設施
  readonly logger: ILogger;
  readonly config: Record<string, any>; // 來自 agent.json 的該插件配置段落

  // [行] 註冊工具：供 LLM 調用的函數
  registerTool(tool: ITool): void;
  
  // [受] 註冊監聽器：觸發 Agent 運行的外部事件源
  registerListener(listener: IListener): void;
  
  // [想] 設定模型：Agent 的大腦 (通常互斥，後註冊者覆蓋或報錯)
  setLLMProvider(provider: ILLMProvider): void;
}
```

---

## 3. 加載與注入流程 (Loading Sequence)

以下是 `Agent Core` 啟動時，`PluginLoader` 的標準作業程序：

1.  **解析配置與發現 (Resolve & Discovery):** 
    *   讀取 `agent.json` 中的 `plugins` ID 列表。
    *   **查詢全域註冊表:** 調用 `PluginRegistryService.resolve(id)`。
    *   **獲取路徑:** 註冊表返回插件的準確物理路徑（已處理好系統/專案優先級）。

2.  **動態載入 (Dynamic Import):**
    使用 Node.js 的 `import()` 或 `require()` 加載該路徑下的入口文件。

3.  **實例化 (Instantiate):**
    創建 `IPlugin` 的實例。

4.  **初始化與注入 (Initialize & Inject):**
    *   Loader 創建一個 `PluginContext` 實例，綁定到當前的 Core 實例（ToolRegistry, EventBus）。
    *   調用 `plugin.initialize(context)`。
    *   **插件代碼執行:** 插件內部調用 `context.registerTool(...)`。
    *   **實際綁定:** `PluginContext` 接收到 Tool 後，將其存入 Core 的 `ToolRegistry`。

5.  **生命週期管理:**
    Loader 將插件實例保存在內部 Map 中，以便在 Agent 關閉時調用 `shutdown()`。

---

## 4. 代碼範例 (Implementation Example)

### 插件端寫法 (`plugins/standard/fs/index.ts`)

```typescript
import { IPlugin, IPluginContext } from '@openstarry/sdk';
import { ReadFileTool } from './tools/ReadFile';

export default class FileSystemPlugin implements IPlugin {
  id = 'openstarry-fs';
  version = '1.0.0';

  async initialize(context: IPluginContext) {
    context.logger.info('正在掛載文件系統能力...');
    
    // 注入工具
    context.registerTool(new ReadFileTool());
    
    // 如果配置允許，還可以注入寫入工具
    if (context.config.enableWrite) {
       // ... register WriteFileTool
    }
  }
  
  async shutdown() {}
}
```

### 核心端寫法 (`packages/core/infrastructure/PluginLoader.ts`)

```typescript
class PluginContextImpl implements IPluginContext {
  constructor(private core: AgentCore, private pluginConfig: any) {}

  registerTool(tool: ITool) {
    // 直接操作核心的註冊表
    this.core.toolRegistry.register(tool);
  }
  
  // ... 其他方法
}
```

---

## 5. 錯誤處理

*   **加載失敗:** 若 `require` 失敗，Loader 應記錄錯誤但不應導致 Core 崩潰（除非是關鍵插件）。
*   **初始化超時:** `initialize` 方法應設有超時限制，防止插件卡死啟動流程。
*   **衝突處理:** 若不同插件註冊了同名的 Tool（如兩個插件都有 `read_file`），Loader 應根據配置策略（覆蓋、報錯、或命名空間隔離）進行處理。

---

## 附錄 A：實際交付的 SDK 契約 (v0.59.x)

> ✅ **[實作狀態 — v0.59.6]** 以下為已交付、有測試覆蓋的真實契約，取代上方 §2/§3/§4 的設計稿。引用 `packages/sdk/src/types/`（API 最高權威）。

### A.1 插件契約：工廠模式（非實作類別）

插件**不導出實作 `initialize`/`shutdown` 的類別**；而是導出一個 `IPlugin` 物件（慣例上由 `createXxxPlugin()` 工廠函數產生），帶 `manifest` 中介資料與一個回傳 `PluginHooks` 的 `factory`。

```typescript
// packages/sdk/src/types/plugin.ts:376-379
export interface IPlugin {
  manifest: PluginManifest;
  factory: (ctx: IPluginContext) => Promise<PluginHooks>;
}
```

`PluginManifest`（`plugin.ts:24-104`）含 `name` / `version` / `description?` / `skandha?` / `dependencies?` / `criticality?` / `capabilities?` 等，**沒有** `id`。生命週期清理由 `PluginHooks.dispose?()`（`plugin.ts:307`）負責，而非 `IPlugin.shutdown()`。

### A.2 能力以「回傳 hooks」交付，而非「呼叫 context 註冊」

`factory(ctx)` 回傳 `PluginHooks`；Loader 收下後注入 Core。插件**不**呼叫 `ctx.registerTool()` 之類方法。

```typescript
// packages/sdk/src/types/plugin.ts:276-308（節錄）
export interface PluginHooks {
  providers?: IProvider[];                       // 想蘊 — LLM 後端
  tools?: ITool[];                               // 行蘊 — 可執行動作
  listeners?: (ITypedListener | IListener)[];    // 色蘊 — 感官輸入
  ui?: IUI[];                                    // 色蘊 — 輸出渲染
  guides?: IGuide[];                             // 識蘊 — 我執框架
  vedanaSensors?: IVedanaSensor[];               // 受蘊 — 三受感測器 (Plan26)
  // … commands / gearArbiters / volition / monitors / auditor /
  //    contextManager / confirmationGate / commChannels / onCheckpoint /
  //    onRestore / dispose
}
```

`IPluginContext`（`plugin.ts:234-273`）提供的是**基礎設施 + 惰性唯讀存取器**，沒有任何 `register*` / `setLLMProvider`：

```typescript
// packages/sdk/src/types/plugin.ts:234-273（節錄）
export interface IPluginContext {
  bus: EventBus;
  workingDirectory: string;
  agentId: string;
  config: Record<string, unknown>;
  pushInput: (event: InputEvent) => void;        // Plugin→Core 的標準輸入路徑
  sessions: ISessionManager;
  tools?: { list(): ITool[]; get(id: string): ITool | undefined };  // 唯讀
  guides?: { list(): IGuide[] };                 // 唯讀
  providers?: { list(): IProvider[]; get(id: string): IProvider | undefined };  // 唯讀
  services?: IServiceRegistry;                   // 跨插件服務注入 (Plan19)
  commands?: { list(): SlashCommand[] };
  metrics?: { getSnapshot(): Record<string, unknown> };
}
```

### A.3 五蘊對應更正：IListener = 色蘊（rupa），非受蘊（vedana）

```typescript
// packages/sdk/src/types/listener.ts:1-13
/**
 * Listener interface — sensory input channels.
 * @skandha rupa (色蘊 — 感官根·輸入)
 */
export interface IListener extends IRupa {
  id: string;
  name: string;
  start?(): Promise<void>;
  stop?(): Promise<void>;
}
```

| Hook | 蘊（aggregate） | SDK 根介面 / 來源 |
|---|---|---|
| `listeners` / `ui` | **色蘊 rupa** | `IRupa`（`aggregates.ts:22-25`）；IListener `listener.ts:8` |
| `vedanaSensors` | **受蘊 vedana** | `IVedana`（`aggregates.ts:41-44`）；`IVedanaSensor`（Plan26） |
| `providers` | 想蘊 samjna | `ISamjna`（`aggregates.ts:56-59`） |
| `tools` | 行蘊 samskara | `ISamskara`（`aggregates.ts:70-73`） |
| `guides` | 識蘊 vijnana | `IVijnana`（`aggregates.ts:85-88`） |

因此本文 §2.2 將 `registerListener` 註記為「[受]」屬雙重錯誤：(1) 該方法不存在；(2) listener 對應「色」而非「受」。「受」由獨立的 `vedanaSensors` hook 承載。

### A.4 真實插件寫法（參考）

```typescript
// 對照 openstarry_plugin/auditor-passthrough/src/index.ts:14,20 等 46 個可載入插件
import type { IPlugin, IPluginContext, PluginHooks } from "@openstarry/sdk";

export function createExamplePlugin(): IPlugin {
  return {
    manifest: { name: "@openstarry-plugin/example", version: "1.0.0", skandha: "samskara" },
    async factory(ctx: IPluginContext): Promise<PluginHooks> {
      // 透過「回傳 hooks」交付能力，而非呼叫 ctx.register*
      return {
        tools: [/* new ReadFileTool() */],
        dispose: async () => { /* 資源清理，取代 shutdown() */ },
      };
    },
  };
}
```