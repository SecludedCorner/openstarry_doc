# 实作示例：UI 插件

本文档提供一个 UI 插件如何实现其接口的具体代码示例。

---

```javascript
// 示例：一个实现了核心双向通信接口的 TUI (Text-based UI) 插件
class Terminal_UI_Plugin {

  /**
   * 构造函数在插件被加载时调用
   * @param {object} coreApi - 核心传入的 API 对象，用于与核心通信
   */
  constructor(coreApi) {
    this.core = coreApi;
    this.initListeners();
    this.render(); // 渲染初始界面
  }

  // 监听来自核心的事件
  initListeners() {
    this.core.on('onNewMessage', (message) => {
      this.displayMessageInChatWindow(message);
    });

    this.core.on('onToolCallRequest', (request) => {
      this.showConfirmationDialog(request.toolName, request.args, (approved) => {
        // 将用户的决定发送回核心
        this.core.provideConfirmation(request.confirmationId, approved);
      });
    });

    this.core.on('onReadyForInput', () => {
      this.enableUserInput();
    });
  }

  // 处理用户输入
  handleUserInput(text) {
    // 将用户的输入提交给核心
    this.core.submitUserInput(text);
  }

  // ... 此处省略了所有具体的界面绘制代码 (displayMessage, showConfirmationDialog, etc.)
  render() { /* ... */ }
  displayMessageInChatWindow(message) { /* ... */ }
  showConfirmationDialog(tool, args, callback) { /* ... */ }
  enableUserInput() { /* ... */ }
}

// 插件入口点，导出插件类
module.exports = Terminal_UI_Plugin;
```
