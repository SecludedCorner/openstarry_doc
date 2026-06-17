# 實作範例：UI 插件

> ⚠️ **[漂移更正 — v0.59.6 / 虛構核心 API；下方原文為早期概念草圖，已被實際 SDK 取代]** 下方原文的 `class Terminal_UI_Plugin(coreApi)` 與其使用的 `core.on('onNewMessage'/'onToolCallRequest'/'onReadyForInput')`、`core.submitUserInput()`、`core.provideConfirmation()`、`module.exports = Terminal_UI_Plugin` 這 5 個識別符**在當前代碼樹中皆 0 命中（`packages/`＋`openstarry_plugin/` 全 source 搜尋為空）**，屬虛構的雙向通信契約。真實 IUI 契約（`packages/sdk/src/types/ui.ts:9-15`）為：
>
> ```ts
> export interface IUI extends IRupa {          // IRupa: readonly skandha: 'rupa'
>   id: string;
>   name: string;
>   onEvent(event: AgentEvent): void | Promise<void>;   // 核心 → UI 單一入口（取代 onNewMessage/onToolCallRequest/onReadyForInput）
>   start?(): Promise<void>;
>   stop?(): Promise<void>;
> }
> ```
>
> 關鍵差異（以代碼為準）：
> - **不是 class + `constructor(coreApi)`**，而是工廠函式 `createXxxPlugin(): IPlugin`，其 `factory(ctx)` 回傳 `PluginHooks` 並以 `ui: [myUI]` 註冊（`packages/sdk/src/types/plugin.ts:280` `ui?: IUI[]`）。
> - **核心 → UI 只有一個入口 `onEvent(event: AgentEvent)`**；訊息/工具呼叫/就緒等都是同一串 `AgentEvent`，靠 `event.type`（如 `message:assistant`、`tool:executing`、`confirmation:request`，見 `packages/sdk/src/types/events.ts:58-244`）分流，沒有 `onNewMessage`/`onToolCallRequest`/`onReadyForInput` 三個獨立回呼。
> - **UI → 核心不呼叫 `submitUserInput()`**，而是透過 `ctx.pushInput({ source, inputType, data, sessionId })`（pushInput 模式；參考 `openstarry_plugin/tui-dashboard/src/index.ts:122-127`）。
> - **沒有 `provideConfirmation(id, approved)` 這個核心方法**；確認流程走事件：核心發 `confirmation:request`、UI 回 `confirmation:response`（`packages/sdk/src/types/events.ts:240-243`）。
> - **沒有 `module.exports = ClassName`**；入口是 `export default createXxxPlugin`（見 tui-dashboard `index.ts:194`）。
>
> 真實可運行的 UI 插件範例請直接看 `openstarry_plugin/tui-dashboard/src/index.ts`（`createTuiDashboardPlugin()` → `{ ui: [tuiUI], listeners: [tuiListener] }`）。下方原文僅保留作歷史設計草稿。

本文件提供一個 UI 插件如何實現其接口的具體程式碼範例。

---

```javascript
// ⚠️ 虛構草稿 — 下列 coreApi.on(...)/submitUserInput/provideConfirmation/module.exports 在真實代碼中不存在；真實契約見頂端更正牌與 tui-dashboard/src/index.ts
// 示例：一個實現了核心雙向通信接口的 TUI (Text-based UI) 插件
class Terminal_UI_Plugin {

  /**
   * 構造函數在插件被加載時調用
   * @param {object} coreApi - 核心傳入的 API 對象，用於與核心通信
   */
  constructor(coreApi) {
    this.core = coreApi;
    this.initListeners();
    this.render(); // 渲染初始界面
  }

  // 監聽來自核心的事件
  initListeners() {
    this.core.on('onNewMessage', (message) => {
      this.displayMessageInChatWindow(message);
    });

    this.core.on('onToolCallRequest', (request) => {
      this.showConfirmationDialog(request.toolName, request.args, (approved) => {
        // 將用戶的決定發送回核心
        this.core.provideConfirmation(request.confirmationId, approved);
      });
    });

    this.core.on('onReadyForInput', () => {
      this.enableUserInput();
    });
  }

  // 處理用戶輸入
  handleUserInput(text) {
    // 將用戶的輸入提交給核心
    this.core.submitUserInput(text);
  }

  // ... 此處省略了所有具體的界面繪製代碼 (displayMessage, showConfirmationDialog, etc.)
  render() { /* ... */ }
  displayMessageInChatWindow(message) { /* ... */ }
  showConfirmationDialog(tool, args, callback) { /* ... */ }
  enableUserInput() { /* ... */ }
}

// 插件入口點，導出插件類
module.exports = Terminal_UI_Plugin;
```
