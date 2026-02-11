# 21. 深度解析：插件介面與五蘊的關係 (Plugin Interface & Aggregates Deep Dive)

> **技術規格提示 (Technical Specification Note):**
> 本文件側重於架構哲學與概念解釋。關於 `IPlugin`, `ITool`, `IAgentGuide` 等介面的嚴格 TypeScript 定義與最新 API 規範，請以 **[03_Plugin_Interface_Definitions.md](../Technical_Specifications/03_Plugin_Interface_Definitions.md)** 為準。

本文件旨在釐清 `IPlugin` (插件容器)、`IPluginContext` (交互環境) 與 `ITool/IListener` (五蘊成分) 三者之間的架構關係。

## 1. 為什麼有了五蘊介面，還需要 `IPlugin`？

這是一個「容器 (Container)」與「內容物 (Content)」的區別。

*   **五蘊 (ITool, IListener...)：** 這些是**「零件」**。例如，一個天氣查詢函數是一個 `ITool`，一個 Discord 監聽器是一個 `IListener`。
*   **插件 (IPlugin)：** 這是**「包裹/快遞箱」**。

Core 無法憑空知道硬碟裡有哪些 `ITool`。它需要一個標準的入口點來「打開包裹」並取出零件。`IPlugin` 就是這個標準入口。

### 比喻
*   **Core:** 是一台電腦主機。
*   **ITool (行蘊):** 是滑鼠。
*   **IListener (受蘊):** 是鍵盤輸入端。
*   **IUI (色蘊):** 是螢幕輸出端。
*   **IPlugin:** 是 **USB 接頭與驅動程式**。它負責告訴主機：「嘿，我這裡有一個滑鼠、一個鍵盤輸入端和一個螢幕輸出端，請把這些驅動起來。」

**重要區分：** `IListener` 和 `IUI` 是分離的介面：
- `IListener` (受蘊) 專注於接收外部輸入，透過 `ctx.pushInput()` 推送事件
- `IUI` (色蘊) 專注於輸出渲染，透過 `onEvent()` 接收並呈現事件

### 程式碼關係
```typescript
// 開發者遵循 IPlugin 介面來定義「包裹」
export default class MyPlugin implements IPlugin {
  // 當 Core 呼叫這個方法時，等於「插入 USB」
  async initialize(context: IPluginContext) {
    // 這裡開發者把「零件 (五蘊)」拿出來交給 Core
    context.registerTool(new WeatherTool()); // WeatherTool 遵循 ITool
    context.registerListener(new DiscordListener()); // DiscordListener 遵循 IListener
  }
}
```

---

## 2. 為什麼需要 `IPluginContext`？

`IPluginContext` 是 Core 與 Plugin 之間的**「交易櫃檯」**或**「交換協議」**。它承載了兩個方向的資訊流動：

### 方向 A：Core -> Plugin (賦予能力)
Plugin 需要 Core 提供的基礎設施才能運作。
*   **Logger:** 讓 Plugin 寫日誌。
*   **Config:** 讓 Plugin 讀取 `agent.json` 裡的設定。
*   **Dependencies:** (關鍵) 讓 Plugin 拿到其他 Plugin 提供的服務。

### 方向 B：Plugin -> Core (交付成分)
Plugin 需要一個途徑把它擁有的五蘊成分「註冊」到 Core 系統中。
*   `registerTool()`
*   `registerListener()`
*   `pushInput(event: InputEvent)` — 讓插件主動向 Core 推送輸入事件（例如 Stdio Listener 收到用戶輸入後，透過 `context.pushInput()` 將事件注入執行迴圈，而非透過建構子回呼）。

如果沒有 `IPluginContext`，Plugin 就變成了孤島，既拿不到 Core 的資源，也無法把自己的功能交給 Core。

> **設計備註：`pushInput` 取代建構子回呼**
> 在早期實作中，Listener 插件需要透過建構子注入的 callback 來將用戶輸入傳回 Core。這造成了 Plugin 與 Core 之間的緊耦合。現行架構中，所有插件統一透過 `context.pushInput(event)` 推送輸入事件，實現了完全的控制反轉 (IoC)。

> **設計備註：動態載入取代 BUILTIN_FACTORIES**
> 舊版架構中存在 `BUILTIN_FACTORIES` 映射表，將短名稱對應到內建插件工廠。此機制已完全移除。所有插件（包括官方標準插件）均透過 `import()` 動態載入，Core 不再硬編碼任何插件引用，確保內核的絕對純淨。

---

## 3. 協調層與 `dependencies` 欄位

您提到的「協調層有了 dependencies 欄位」，是指在運行時的**依賴注入機制**。

### 場景：Workflow 插件需要理解 MD 檔案
1.  **Skill Plugin (提供者):** 它實作了一個 `MarkdownParser` 服務。
2.  **Workflow Plugin (消費者):** 它需要這個 Parser 才能運作。

### 協調層 (Daemon) 的工作
在啟動 Workflow Plugin 之前，協調層會做這件事：

```typescript
// 1. 先從 Skill Plugin 拿到 Parser 實例
const mdParser = skillPlugin.getService();

// 2. 準備 Workflow Plugin 的 Context
const contextForWorkflow = {
  logger: ...,
  // 3. 把 Parser 塞進 dependencies 欄位
  dependencies: {
    markdownParser: mdParser 
  }
};

// 4. 初始化 Workflow Plugin
workflowPlugin.initialize(contextForWorkflow);
```

### 開發者的視角
在 Workflow Plugin 的代碼中：

```typescript
initialize(context: IPluginContext) {
  // 從 Context 中取出協調層準備好的依賴
  const parser = context.dependencies.markdownParser;
  
  // 開始使用它
  parser.parse("workflow.md");
}
```

## 4. 總結

*   **五蘊介面 (`ITool` 等)**：定義了**「是什麼」**（它是個工具）。
*   **`IPlugin`**：定義了**「怎麼加載」**（初始化的入口）。
*   **`IPluginContext`**：定義了**「怎麼溝通」**（交換資源與能力的管道）。

這三者缺一不可，共同構成了一個可插拔、可協作的生態系統。
