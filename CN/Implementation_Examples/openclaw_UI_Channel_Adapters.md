# 实现思路：openclaw - 多渠道 UI 适配器

本文档详细描述了为实现 `openclaw` 的多渠道接入能力而设计的 `UI` 插件——UI 适配器的实现思路。

## 核心设计：作为桥梁的 UI 插件

每个外部消息平台（如 WhatsApp, Telegram, Slack）都通过一个专门的 `UI` 插件接入到我们的代理人系统中。这个插件扮演着**「协议翻译官」**和**「双向桥梁」**的角色。

以 `WhatsApp_UI_Plugin` 为例：

---

### 1. 插件的职责

*   实现核心的「双向通信接口」。
*   处理与特定平台（WhatsApp Business API）的所有网络通信和认证。
*   将平台的事件模型翻译成核心的命令模型，反之亦然。

### 2. 工作流程

#### 入向流程 (Inbound: 用户 -> 代理人)

1.  **监听 Webhook:**
    *   插件需要启动一个轻量的 HTTP 服务器，并暴露一个 Webhook 端点（例如 `/webhooks/whatsapp`）。
    *   这个端点需要被配置在 WhatsApp Business API 的后台，用于接收新消息事件。

2.  **处理请求:**
    *   当用户向 WhatsApp 号码发送消息时，WhatsApp 会向我们的 Webhook 端点发送一个 POST 请求，其中包含消息内容、用户信息等。

3.  **解析与转换:**
    *   插件的 HTTP 服务器接收到请求后，会解析其 JSON 内容，提取出 `message.content` 和 `user.id` 等关键信息。

4.  **调用核心:**
    *   插件从请求中识别出对应的会话，然后调用核心的 `coreApi.submitUserInput(message.content)` 方法。
    *   **关键点：** 插件需要维护一个 `用户 ID` 到 `会话 ID` 的映射表，以确保来自同一用户的消息能进入同一个对话会话。

#### 出向流程 (Outbound: 代理人 -> 用户)

1.  **监听核心事件:**
    *   插件在其 `constructor` 中，会监听核心的事件，特别是 `core.on('onNewMessage', ...)`。

2.  **格式化消息:**
    *   当 `onNewMessage` 事件触发时，插件会获取到代理人生成的最终答复 `{ content: "...", ... }`。
    *   插件需要将这个标准格式的答复，转换成 WhatsApp API 所要求的 JSON 格式（例如，指明接收方号码、消息类型为文本等）。

3.  **调用平台 API:**
    *   插件使用 `fetch` 或其他 HTTP 客户端，携带必要的认证信息（API Token），向 WhatsApp Business API 的消息发送端点发起 POST 请求。

### 结论

通过为每个平台（Telegram, Slack, Discord 等）都创建一个类似的、专门的 `UI` 插件，我们的系统就能实现 `openclaw` 那样强大的多渠道接入能力。所有特定于平台的复杂逻辑都被封装在各自的插件中，而代理人核心和协调层则保持通用和平台无关。
