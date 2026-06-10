# 插件範例：Web 交互 (Web Interaction)

本文件闡述了與 Web 進行交互的插件設計，包括 Webhook 和瀏覽器控制。

---

### 1. Webhook

Webhook 的能力通過 `Listener` 插件來實現，讓代理人能被動地接收來自 Web 的事件。

*   **插件類型：** `listener`
*   **職責：** 啟動 HTTP 服務，監聽特定 URL 端點，並將接收到的請求轉發到核心的事件隊列。
*   **`plugin.json` 示例：**
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
*   **實現思路：**
    1.  `WebhookListener.js` 的 `start(eventQueue, config)` 方法被核心調用。
    2.  在 `start` 方法中，使用 `Express` 或 `Fastify` 等框架，在指定的 `port` 上啟動一個 HTTP 服務。
    3.  服務監聽配置的 `endpoint` 上的 `POST` 或 `GET` 請求。
    4.  當收到請求時，將請求的 `headers`, `body`, `query_params` 等信息打包成一個標準事件對象。
    5.  調用 `eventQueue.push({ source: 'webhook', type: 'http_request', payload: ... })` 將事件推入隊列。

---

### 2. 瀏覽器控制 (Browser Control)

瀏覽器控制是一套功能強大的 `Tool` 插件，它允許 LLM 像人一樣「看到」和「操作」網頁。

*   **插件類型：** `tool` (通常是一個包含多個工具的套件)
*   **核心依賴：** 後端的**瀏覽器管理器**，該管理器通過 `Puppeteer` 或 `Playwright` 控制一個或多個無頭瀏覽器實例。出於安全考慮，瀏覽器實例本身也應運行在沙盒環境中。
*   **`plugin.json` 示例：**
    ```json
    {
      "name": "Browser-Control-Suite",
      "type": "tool",
      "entryPoint": "./BrowserTools.js"
    }
    ```
*   **工具範例 (`BrowserTools.js` 導出多個工具):**
    *   `browser:new_page(url)`: 打開一個新頁面並導航到指定 URL，返回一個 `page_id`。
    *   `browser:scrape(page_id, selectors, format)`: 在指定頁面上，根據 CSS 選擇器 `selectors` 提取內容，並可以要求以 `text` 或 `markdown` 格式返回。
    *   `browser:click(page_id, selector)`: 在指定頁面上點擊一個元素。
    *   `browser:type(page_id, selector, text)`: 在指定頁面的輸入框中輸入文字。
    *   `browser:screenshot(page_id)`: 對頁面進行截圖，並返回圖片的 Base64 編碼。
*   **實現思路：**
    *   每個工具的 `execute` 方法都通過後端的「瀏覽器管理器」來執行相應的 `Playwright` 操作。
    *   管理器負責維護 `page_id` 和實際頁面對象之間的映射。
    *   `scrape` 的結果或 `screenshot` 的圖片數據會被返回給 LLM，使其能夠「看到」網頁內容並決定下一步操作。
