# 12. 能力發現與依賴注入機制 (Capabilities Discovery & Injection)

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