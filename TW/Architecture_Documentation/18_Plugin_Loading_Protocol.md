# 18. 插件加載與註冊協議 (Plugin Loading Protocol)

本文件定義了 OpenStarry 系統如何從磁碟加載插件代碼，並將其與核心系統進行對接的詳細協議。這是 Phase 3 的核心技術規範。

## 1. 核心挑戰

由於插件是「五蘊聚合體」，而 Core 內部的運作是分模組的（ToolManager, ListenerManager 等），加載器必須解決以下問題：
1.  **動態性：** 插件代碼在編譯時未知。
2.  **解構 (Destructuring)：** 將一個插件包拆解為多個功能單元。
3.  **依賴注入 (Dependency Injection)：** 讓插件獲得 Core 的能力（如 Logger, Config），而不讓插件依賴 Core 的實作。

## 2. 加載協議：工廠模式 (Factory Pattern)

為了最大化靈活性，我們放棄「類別掃描」模式，採用**工廠函數模式**。

> **重要更新：** 原有的 `BUILTIN_FACTORIES` 硬編碼映射表已被移除。所有插件（包括內建插件）現在統一透過動態加載機制載入：優先使用 `ref.path`（本地路徑），否則使用 `import(ref.name)`（NPM 包名）。這確保了插件加載邏輯的一致性，消除了內建與外部插件之間的區別。

### 插件入口規範 (`index.ts`)

每個插件必須導出一個名為 `initialize` 的異步函數，或者是實現了 `IPluginFactory` 介面的默認導出。

```typescript
import { IPluginContext, IPlugin } from '@openstarry/sdk';
import { MyTool } from './tools/my-tool';
import { MyListener } from './listeners/my-listener';

// 插件工廠函數
export default async function initialize(context: IPluginContext): Promise<void> {
  // 1. 獲取 Core 注入的依賴
  const logger = context.logger;
  logger.info('Initializing My Plugin...');

  // 2. 註冊行蘊 (Tools)
  // 這裡插件主動將自己的 Tool 實例交給 Core
  context.registerTool(new MyTool());

  // 3. 註冊受蘊 (Listeners)
  context.registerListener(new MyListener(context.config));
  
  // 4. 註冊識蘊 (Guide)
  context.registerGuide({
    systemPrompt: "...",
    memoryPolicy: "sliding-window"
  });
}
```

## 3. 宿主端加載流程 (`PluginLoader` Logic)

`PluginLoader` 運行在 **協調層 (Coordination Layer / Daemon)** 或 **Host Process** 中。

### 步驟 A: 物理讀取
1.  讀取 `~/.openstarry/plugins/<plugin-id>/package.json`。
2.  解析 `main` 欄位找到入口文件 (e.g. `dist/index.js`)。
3.  使用 `import()` (ESM) 或 `require()` (CJS) 動態載入該模組。

### 步驟 B: 上下文構建 (Context Construction)
Loader 為該插件創建一個專屬的 `PluginContext` 實例。這個 Context 是插件與外界溝通的唯一管道。

```typescript
const context: IPluginContext = {
  logger: new ScopedLogger(`Plugin:${pluginId}`),
  config: agentConfig.plugins[pluginId] || {}, // 從 agent.json 讀取配置
  
  // 註冊回調 (Callback)
  registerTool: (tool) => core.toolRegistry.add(tool),
  registerListener: (listener) => core.listenerRegistry.add(listener),
  registerProvider: (provider) => core.providerRegistry.add(provider),
  registerGuide: (guide) => core.guideRegistry.set(guide),

  // 輸入注入 — 插件可透過 pushInput 向 Agent 主動推送訊息
  pushInput: (input) => core.inputQueue.push(input),
};
```

### 步驟 C: 初始化與解構
呼叫插件的 `initialize(context)`。
*   此時，插件內部的邏輯開始執行，並透過 `register*` 方法將其五蘊成分「交出」。
*   Core 接收到這些成分後，將其分別歸檔到對應的管理器中。

## 4. 錯誤隔離與沙箱

*   **加載失敗：** 如果 `initialize` 拋出異常，Loader 應捕獲並記錄「痛覺」，但不能讓整個 Agent 崩潰。該插件將被標記為 `FAILED`。
*   **依賴注入安全：** `IPluginContext` 只暴露安全的 API。插件無法直接訪問 Core 的私有屬性或宿主的 `process` 對象（除非在 Level 0 信任模式）。

## 5. 協調層註冊

所有成功加載的插件，其元數據（ID, Version, Capabilities）會被回報給 **Agent Coordination Layer**。這使得協調層能夠回答「誰有天氣查詢功能？」這類問題，實現跨 Agent 的動態路由。

---

## 6. 依賴解析與拓撲加載 (Dependency Resolution & Topological Loading)

為了防止因加載順序錯誤導致的系統崩潰（例如：Workflow 插件在 Skill 插件之前加載，導致無法獲取 Markdown Parser），Loader **必須** 實作依賴圖解析算法。

### 兩階段加載流程 (Two-Phase Loading)

#### Phase A: 掃描與建圖 (Scan & Graph)
1.  Loader 掃描所有目標插件目錄。
2.  讀取每個插件的 `package.json` 中的 `openstarry.dependencies` 字段。
    ```json
    "dependencies": {
      "plugins": ["@openstarry-plugin/skill"] // 聲明對其他插件的硬依賴
    }
    ```
3.  在內存中構建 **依賴有向圖 (Dependency Directed Graph)**。
    *   節點：Plugin ID
    *   邊：Dependency -> Dependent

#### Phase B: 排序與執行 (Sort & Execute)
1.  **循環偵測 (Cycle Detection):** 檢查圖中是否存在循環依賴 (A -> B -> A)。若發現，立即拋出 `FatalDependencyError` 並終止啟動。
2.  **拓撲排序 (Topological Sort):** 使用 Kahn 演算法或 DFS 計算出線性的加載順序。
    *   *Result:* `['@openstarry-plugin/skill', '@openstarry-plugin/gemini', '@openstarry-plugin/workflow']`
3.  **依序初始化:** 按照排序後的清單，依次呼叫 `initialize(context)`。
    *   這確保了當 `workflow` 初始化時，`skill` 已經準備好並將 Parser 注入到了系統中。

### 依賴注入的時機
跨插件的服務（如 `markdownParser`）必須在 **Phase B** 的排序過程中被動態注入。
*   當 `@openstarry-plugin/skill` 初始化完成後，它會返回（或註冊）其服務實例。
*   Loader 暫存這些實例。
*   當輪到 `@openstarry-plugin/workflow` 初始化時，Loader 從暫存區取出 `markdownParser` 並放入 `context.dependencies`。

