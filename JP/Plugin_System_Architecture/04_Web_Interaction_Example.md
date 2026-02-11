# プラグイン例： Web 対話 (Web Interaction)

このドキュメントでは、 Webhook やブラウザ制御など、 Web と対話するためのプラグイン設計について説明します。

---

### 1. Webhook

Webhook の機能は `Listener` プラグインを通じて実装され、エージェントが Web からのイベントを受動的に受信できるようにします。

*   **プラグインタイプ：** `listener`
*   **責務：** HTTP サービスを起動し、特定の URL エンドポイントを監視して、受信したリクエストをコアのイベントキューに転送します。
*   **`plugin.json` の例：**
    ```json
    {
      "name": "Webhook-Listener",
      "type": "listener",
      "entryPoint": "./WebhookListener.js",
      "config": {
        "port": 8080,
        "endpoint": "/hooks/github"
      }
    }
    ```
*   **実装の考え方：**
    1.  `WebhookListener.js` の `start(eventQueue, config)` メソッドがコアから呼び出されます。
    2.  `start` メソッド内で、 `Express` や `Fastify` などのフレームワークを使用して、指定された `port` で HTTP サービスを起動します。
    3.  サービスは設定された `endpoint` で `POST` または `GET` リクエストを監視します。
    4.  リクエストを受信すると、リクエストの `headers`, `body`, `query_params` などの情報を標準イベントオブジェクトとしてパッケージ化します。
    5.  `eventQueue.push({ source: 'webhook', type: 'http_request', payload: ... })` を呼び出して、イベントをキューにプッシュします。

---

### 2. ブラウザ制御 (Browser Control)

ブラウザ制御は強力な `Tool` プラグインセットであり、 LLM が人間のようにウェブページを「見て」「操作する」ことを可能にします。

*   **プラグインタイプ：** `tool` （通常、複数のツールを含むスイート）
*   **コアの依存関係：** バックエンドの **ブラウザマネージャー** です。このマネージャーは `Puppeteer` または `Playwright` を通じて、1つ以上のヘッドレスブラウザインスタンスを制御します。安全上の理由から、ブラウザインスタンス自体もサンドボックス環境で実行されるべきです。
*   **`plugin.json` の例：**
    ```json
    {
      "name": "Browser-Control-Suite",
      "type": "tool",
      "entryPoint": "./BrowserTools.js"
    }
    ```
*   **ツールの例（ `BrowserTools.js` が複数のツールをエクスポート）：**
    *   `browser:new_page(url)`：新しいページを開き、指定された URL に遷移して、 `page_id` を返します。
    *   `browser:scrape(page_id, selectors, format)`：指定されたページで CSS セレクター `selectors` に基づいて内容を抽出し、 `text` または `markdown` 形式での返却を要求できます。
    *   `browser:click(page_id, selector)`：指定されたページで要素をクリックします。
    *   `browser:type(page_id, selector, text)`：指定されたページの入力ボックスにテキストを入力します。
    *   `browser:screenshot(page_id)`：ページのスクリーンショットを撮り、画像の Base64 エンコードを返します。
*   **実装の考え方：**
    *   各ツールの `execute` メソッドは、バックエンドの「ブラウザマネージャー」を介して対応する `Playwright` 操作を実行します。
    *   マネージャーは `page_id` と実際のページオブジェクトのマッピングを維持する責任を負います。
    *   `scrape` の結果や `screenshot` の画像データが LLM に返され、 LLM がウェブページの内容を「見て」次のアクションを決定できるようになります。
