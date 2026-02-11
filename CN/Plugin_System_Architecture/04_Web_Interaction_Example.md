# 插件示例：Web 交互 (Web Interaction)

本文档阐述了与 Web 进行交互的插件设计，包括 Webhook 和浏览器控制。

---

### 1. Webhook

Webhook 的能力通过 `Listener` 插件来实现，让代理人能被动地接收来自 Web 的事件。

*   **插件类型：** `listener`
*   **职责：** 启动 HTTP 服务，监听特定 URL 端点，并将接收到的请求转发到核心的事件队列。
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
*   **实现思路：**
    1.  `WebhookListener.js` 的 `start(eventQueue, config)` 方法被核心调用。
    2.  在 `start` 方法中，使用 `Express` 或 `Fastify` 等框架，在指定的 `port` 上启动一个 HTTP 服务。
    3.  服务监听配置的 `endpoint` 上的 `POST` 或 `GET` 请求。
    4.  当收到请求时，将请求的 `headers`, `body`, `query_params` 等信息打包成一个标准事件对象。
    5.  调用 `eventQueue.push({ source: 'webhook', type: 'http_request', payload: ... })` 将事件推入队列。

---

### 2. 浏览器控制 (Browser Control)

浏览器控制是一套功能强大的 `Tool` 插件，它允许 LLM 像人一样「看到」和「操作」网页。

*   **插件类型：** `tool` (通常是一个包含多个工具的套件)
*   **核心依赖：** 后端的**浏览器管理器**，该管理器通过 `Puppeteer` 或 `Playwright` 控制一个或多个无头浏览器实例。出于安全考虑，浏览器实例本身也应运行在沙盒环境中。
*   **`plugin.json` 示例：**
    ```json
    {
      "name": "Browser-Control-Suite",
      "type": "tool",
      "entryPoint": "./BrowserTools.js"
    }
    ```
*   **工具范例 (`BrowserTools.js` 导出多个工具):**
    *   `browser:new_page(url)`: 打开一个新页面并导航到指定 URL，返回一个 `page_id`。
    *   `browser:scrape(page_id, selectors, format)`: 在指定页面上，根据 CSS 选择器 `selectors` 提取内容，并可以要求以 `text` 或 `markdown` 格式返回。
    *   `browser:click(page_id, selector)`: 在指定页面上点击一个元素。
    *   `browser:type(page_id, selector, text)`: 在指定页面的输入框中输入文字。
    *   `browser:screenshot(page_id)`: 对页面进行截图，并返回图片的 Base64 编码。
*   **实现思路：**
    *   每个工具的 `execute` 方法都通过后端的「浏览器管理器」来执行相应的 `Playwright` 操作。
    *   管理器负责维护 `page_id` 和实际页面对象之间的映射。
    *   `scrape` 的结果或 `screenshot` 的图片数据会被返回给 LLM，使其能够「看到」网页内容并决定下一步操作。
    *   `scrape` 的结果或 `screenshot` 的图片数据会被返回给 LLM，使其能够「看到」网页内容并决定下一步操作。
