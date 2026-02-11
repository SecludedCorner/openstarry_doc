# 実装例： UI プラグイン

このドキュメントでは、 UI プラグインがそのインターフェースをどのように実装するかについて、具体的なコード例を提供します。

---

```javascript
// 例：コアの双方向通信インターフェースを実装した TUI (Text-based UI) プラグイン
class Terminal_UI_Plugin {

  /**
   * プラグインがロードされるときに呼び出されるコンストラクタ
   * @param {object} coreApi - コアから渡される、通信用の API オブジェクト
   */
  constructor(coreApi) {
    this.core = coreApi;
    this.initListeners();
    this.render(); // 初期画面をレンダリング
  }

  // コアからのイベントを監視
  initListeners() {
    this.core.on('onNewMessage', (message) => {
      this.displayMessageInChatWindow(message);
    });

    this.core.on('onToolCallRequest', (request) => {
      this.showConfirmationDialog(request.toolName, request.args, (approved) => {
        // ユーザーの決定をコアに送り返す
        this.core.provideConfirmation(request.confirmationId, approved);
      });
    });

    this.core.on('onReadyForInput', () => {
      this.enableUserInput();
    });
  }

  // ユーザー入力の処理
  handleUserInput(text) {
    // ユーザーの入力をコアに送信
    this.core.submitUserInput(text);
  }

  // ... ここでは具体的な画面描画コード (displayMessage, showConfirmationDialog など) は省略
  render() { /* ... */ }
  displayMessageInChatWindow(message) { /* ... */ }
  showConfirmationDialog(tool, args, callback) { /* ... */ }
  enableUserInput() { /* ... */ }
}

// プラグインのエントリポイント。プラグインクラスをエクスポートする
module.exports = Terminal_UI_Plugin;
```
