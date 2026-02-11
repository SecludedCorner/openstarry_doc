# 16. 插件类型哲学总结图表 (Plugin Types Philosophical Mapping)

本文档汇总了 OpenStarry 架构中 **「五种插件类型」** 与东方哲学 **「五蕴 (Five Aggregates)」** 的最终映射关系。这是理解系统如何构成「数字生命」的核心索引。

## 1. 核心观点：空与有的结合

*   **Agent Core (空):** 是一个纯粹的容器，具备运作的潜能，但没有具体的特质。
*   **Plugins (有):** 是填充这个容器的内容。五种插件分别赋予了 Agent 生命的五个维度。

## 2. 五蕴映射图表 (The Mapping Chart)

| 插件类型 (Type) | 哲学对应 (Aggregate) | 梵文 (Sanskrit) | 定义 (Definition) | 系统职责 (Responsibility) | 代表性组件 (Examples) |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **UI** | **色蕴** | **Rupa** | 物质、形体、显相 | 定义 Agent 在物理世界或数字界面中的呈现方式。 | Dashboard, CLI, React Components |
| **Listener** | **受蕴** | **Vedana** | 感受、输入、刺激 | 定义 Agent 接收外部信息的通道与协议。 | HTTP Server, WebSocket, Cron Trigger |
| **Provider** | **想蕴** | **Samjna** | 认知、概念、处理 | 定义 Agent 的大脑处理引擎与推理模型。 | Gemini Adapter, OpenAI Adapter |
| **Tool** | **行蕴** | **Samskara** | 造作、意志、行动 | 定义 Agent 对外部世界产生影响的能力。 | File System, API Client, Code Exec |
| **Guide** | **识蕴** | **Vijnana** | 识别、灵魂、主体 | 定义 Agent 的自我意识、记忆结构与行为准则。 | Markdown Skills, MCP Logic, Workflows |

## 3. 详细解读示例

### 色蕴 (Rupa) - UI
这是 Agent 的「皮囊」。同样一个 Agent Core，换了 UI 插件，可以从一个 CLI 工具变成一个 Web 聊天机器人。

举例说明
UI 插件实现 `IUI` 接口，负责接收 `AgentEvent` 并呈现输出。它与 Listener 是完全独立的：
- **UI (色蕴)** — 输出渲染，接收事件并呈现
- **Listener (受蕴)** — 输入接收，推送事件到队列

### 受蕴 (Vedana) - Listener
这是 Agent 的「感官」。它决定了 Agent 能「听到」什么。是 HTTP 请求？还是时间流逝？

举例说明
Listener 插件实现 `IListener` 接口，专注于接收外部输入并通过 `ctx.pushInput()` 推送到事件队列。

### 想蕴 (Samjna) - Provider
这是 Agent 的「智商」。它决定了 Agent 如何理解输入的信息。

### 行蕴 (Samskara) - Tool
这是 Agent 的「手脚」。它决定了 Agent 能「做」什么。没有 Tool，Agent 就只是个只会说话的哲学家。

### 识蕴 (Vijnana) - Guide
这是 Agent 的「人设」。它整合了上述四蕴，并赋予其方向。
*   没有 Guide，Core 就像失忆的人，有能力但不知道我是谁。
*   加载了 `expert-coder.md` (Guide)，Core 就「识别」出自己是个工程师。
