# 實現思路：openclaw - 多渠道 UI 適配器

本文件詳細描述了為實現 `openclaw` 的多渠道接入能力而設計的 `UI` 插件——UI 適配器的實現思路。

## 核心設計：作為橋樑的 UI 插件

每個外部消息平台（如 WhatsApp, Telegram, Slack）都通過一個專門的 `UI` 插件接入到我們的代理人系統中。這個插件扮演著**「協議翻譯官」**和**「雙向橋樑」**的角色。

以 `WhatsApp_UI_Plugin` 為例：

---

### 1. 插件的職責

*   實現核心的「雙向通信接口」。
*   處理與特定平台（WhatsApp Business API）的所有網絡通信和認證。
*   將平台的事件模型翻譯成核心的命令模型，反之亦然。

### 2. 工作流程

#### 入向流程 (Inbound: 用戶 -> 代理人)

1.  **監聽 Webhook:**
    *   插件需要啟動一個輕量的 HTTP 服務器，並暴露一個 Webhook 端點（例如 `/webhooks/whatsapp`）。
    *   這個端點需要被配置在 WhatsApp Business API 的後台，用於接收新消息事件。

2.  **處理請求:**
    *   當用戶向 WhatsApp 號碼發送消息時，WhatsApp 會向我們的 Webhook 端點發送一個 POST 請求，其中包含消息內容、用戶信息等。

3.  **解析與轉換:**
    *   插件的 HTTP 服務器接收到請求後，會解析其 JSON 內容，提取出 `message.content` 和 `user.id` 等關鍵信息。

4.  **調用核心:**
    *   插件從請求中識別出對應的會話，然後調用核心的 `coreApi.submitUserInput(message.content)` 方法。
    *   **關鍵點：** 插件需要維護一個 `用戶 ID` 到 `會話 ID` 的映射表，以確保來自同一用戶的消息能進入同一個對話會話。

#### 出向流程 (Outbound: 代理人 -> 用戶)

1.  **監聽核心事件:**
    *   插件在其 `constructor` 中，會監聽核心的事件，特別是 `core.on('onNewMessage', ...)`。

2.  **格式化消息:**
    *   當 `onNewMessage` 事件觸發時，插件會獲取到代理人生成的最終答覆 `{ content: "...", ... }`。
    *   插件需要將這個標準格式的答覆，轉換成 WhatsApp API 所要求的 JSON 格式（例如，指明接收方號碼、消息類型為文本等）。

3.  **調用平台 API:**
    *   插件使用 `fetch` 或其他 HTTP 客戶端，攜帶必要的認證信息（API Token），向 WhatsApp Business API 的消息發送端點發起 POST 請求。

### 結論

通過為每個平台（Telegram, Slack, Discord 等）都創建一個類似的、專門的 `UI` 插件，我們的系統就能實現 `openclaw` 那樣強大的多渠道接入能力。所有特定於平台的複雜邏輯都被封裝在各自的插件中，而代理人核心和協調層則保持通用和平台無關。
