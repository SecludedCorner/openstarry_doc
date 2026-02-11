# 10. 上下文管理策略 (Context Management Strategy)

本文档定义了 OpenStarry 架构中关于 LLM Context Window（上下文窗口）管理的设计哲学与实作接口。

## 核心哲学

基于 OpenStarry 的「去中心化」与「模块化」原则，Context 管理不应被硬编码在代理人核心的执行循环中。相反，它被视为一种**「认知策略 (Cognitive Strategy)」**，并且必须是**可插拔 (Pluggable)** 的。

### 为什么需要策略化？

不同的代理人角色对记忆的需求截然不同：
*   **客服 Agent:** 需要精确记住用户刚说的这句话，但可以遗忘 10 分钟前的闲聊。（适合滑动窗口）
*   **小说家 Agent:** 需要记住每一章的剧情摘要，但不需要记住逐字稿。（适合动态摘要）
*   **订票 Agent:** 只需要记住「目的地」、「时间」等关键变量。（适合状态提取）

因此，`ContextManager` 必须是一个接口，允许开发者根据 Agent 的角色挂载不同的策略插件。

---

## 架构设计

### 1. 接口定义 (`IContextManager`)

所有的 Context 策略都必须实现以下标准接口：

```typescript
interface IContextManager {
  /**
   * 核心方法：组装发送给 LLM 的最终 Payload
   * @param systemPrompt - 不可变的系统指令
   * @param history - 原始的对话历史队列
   * @param tools - 当前可用的工具定义
   * @param ragContext - (可选) 检索到的相关知识
   * @returns 经修剪/压缩后，符合 Token 限制的 Messages 数组
   */
  composePayload(
    systemPrompt: string,
    history: ChatMessage[],
    tools: ToolDefinition[],
    ragContext?: string
  ): Promise<LLMPayload>;

  /**
   * 钩子：在每次对话结束后调用，用于后台处理（如更新摘要）
   */
  onTurnComplete(newHistory: ChatMessage[]): Promise<void>;
}
```

### 2. Context 的解剖学 (Anatomy of Context)

一个典型的 Context Payload 由以下优先级组成（由高到低）：

1.  **System Block:** 身份定义、核心指令。(权重：最高，不可丢弃)
2.  **Tool Definitions:** 工具的 JSON Schema。(权重：极高，必须完整)
3.  **Dynamic Context:** 当前任务目标、RAG 检索结果。(权重：高)
4.  **Memory/Summary Block:** 对过去对话的压缩摘要。(权重：中)
5.  **Recent History:** 最近的 N 轮对话。(权重：低，最早的可以被牺牲)

---

## 标准策略实现

我们预定义了三种标准策略，开发者也可以编写自己的策略。

### 策略 A: 滑动窗口 (Sliding Window) - [默认]

*   **逻辑：** 纯粹的 FIFO (First-In-First-Out)。
*   **算法：**
    1.  计算 `System` + `Tools` + `Dynamic` 的 Token 消耗。
    2.  剩余的空间 `Remaining_Tokens` 全部留给 `History`。
    3.  从 `History` 队列的**末尾 (最新)** 开始往前选取消息，直到总 Token 数接近 `Remaining_Tokens`。
    4.  舍弃更早的消息。
*   **适用场景：** 简单指令执行、短期问答。

### 策略 B: 动态摘要 (Dynamic Summarization) - [推荐]

*   **逻辑：** 定期将旧的对话历史压缩成自然语言摘要。
*   **算法：**
    1.  维护一个 `Summary_Buffer` 字符串。
    2.  当 `History` 长度超过阈值 (如 10 轮) 时，触发后台任务。
    3.  使用一个轻量级 LLM (如 Gemini Flash) 读取最早的 5 轮对话与当前的 `Summary_Buffer`。
    4.  生成新的摘要，并清空那 5 轮对话。
    5.  组装 Payload 时，将 `Summary_Buffer` 插入 System Prompt 之后。
*   **适用场景：** 长期陪伴型助手、复杂任务协作。

### 策略 C: 关键状态提取 (Entity Extraction)

*   **逻辑：** 不保留对话流，只保留结构化状态。
*   **算法：**
    1.  定义一个状态 Schema (例如 `{ destination: string, date: string }`)。
    2.  每次用户输入后，先调用一个专门的 Extraction Chain 更新这个 JSON 对象。
    3.  组装 Payload 时，直接将这个 JSON 对象注入 Context。
*   **适用场景：** 填表单机器人、特定流程导引。

---

## 插件化实现

在 `plugin.json` 中，开发者可以指定使用哪种策略：

```json
{
  "id": "my-complex-agent",
  "contextStrategy": {
    "type": "plugin",
    "pluginId": "std-summarization-strategy",
    "config": {
      "compressionModel": "gemini-1.5-flash",
      "threshold": 20
    }
  }
}
```

这种设计确保了 Agent Core 始终保持轻量，而将复杂的记忆管理逻辑下放给了专门的策略模块。
