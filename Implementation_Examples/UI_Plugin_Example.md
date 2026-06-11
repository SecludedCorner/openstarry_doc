# 實作範例：UI 插件

本文件提供一個 UI 插件如何實現其接口的具體程式碼範例。

---

```javascript
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
