# 深度解析：状态管理器

本文档深入探讨「无头代理人核心」的记忆中枢——状态管理器的数据结构与职责。

## 核心职责

状态管理器负责维护**单次对话会话**的完整上下文，确保对话的连贯性。它主要管理「短期记忆」（当前正在处理的思考链）和「中期记忆」（本次会话的完整历史）。它**不负责**「长期记忆」。

---

## 数据结构深度解析

状态管理器的核心是一个线性的消息列表 (Message List)。为了准确地追踪工具调用与其结果之间的关系，数据结构需要更为精细。

```typescript
// 定义消息的类型
type MessageRole = 'user' | 'assistant' | 'tool';

// 助理发起的工具调用请求
interface ToolCallRequest {
  id: string;      // 每次工具调用的唯一 ID，由核心生成
  name: string;    // 工具名称
  args: object;    // 工具参数
}

// 工具执行的结果
interface ToolCallResult {
  tool_call_id: string; // 对应 ToolCallRequest 的 id
  result: any;          // 工具执行的返回结果
  is_error: boolean;    // 标记结果是否为一个错误
}

// 统一的消息对象
interface Message {
  id: string;           // 消息的唯一 ID
  timestamp: string;    // 消息创建时间
  role: MessageRole;

  // 当 role 为 'user' 或 'assistant' (最终答复) 时
  content?: string | null;

  // 当 role 为 'assistant' 且发起工具调用时
  tool_calls?: ToolCallRequest[];

  // 当 role 为 'tool' 时
  tool_results?: ToolCallResult[]; 
}
```
**关键设计：** 通过 `tool_calls` 列表中的 `id` 和 `tool_results` 列表中的 `tool_call_id`，我们可以清晰地将一个或多个工具调用请求与其对应的结果关联起来，即使在多工具并行调用的情况下也能保持上下文的准确性。

---

## 关键实现细节

### 1. 记忆截断策略 (Truncation Strategy)
为了防止上下文超过 LLM 的 Token 限制，状态管理器必须实现记忆截断策略。
*   **Token 计数器:** 在添加任何新消息之前，实时计算当前消息列表的总 Token 数。
*   **滑动窗口 (Sliding Window):** 当 Token 超限时，从列表的**开头**（即最旧的消息，但通常会跳过第一条系统提示）移除一条或多条消息。
*   **对话总结 (Summarization):** 一种更高级的策略。当历史记录过长时，可以异步地调用一个工具或专门的 LLM，将较早的几轮对话进行总结。然后，用一条包含 `role: 'system'` 和 `content: '前文摘要：...'` 的总结性消息，替换掉被总结的多条原始消息。

### 2. 与长期记忆引擎的交互
状态管理器不存储长期记忆，但它是长期记忆的**来源**。
*   **数据上报/卸载 (Data Offloading):**
    *   在一次对话会话结束时 (`/reset` 或超时)，状态管理器可以将本次会话的完整历史记录发送给外部的「记忆与 RAG 引擎」。
    *   「记忆与 RAG 引擎」接收到这些原始对话后，会在后台进行异步处理，提取关键实体、用户偏好、生成新的知识，并存储到其知识库中。
*   **引导上下文 (Bootstrapping Context):** 在一次新对话会话开始时，「代理人核心」可以先向「记忆与 RAG 引擎」查询关于当前用户的关键信息摘要，并将其作为初始的系统提示的一部分，预先填入状态管理器。

### 3. 会话管理 (Session Management)
在实际应用中，系统需要同时处理多个用户的多个会话。
*   状态管理器本身应该是**无状态**的，它的所有数据（消息列表）都来自于传入的参数。
*   在「代理人核心」之上，应该有一个**「会话管理器」 (Session Manager)**，负责：
    1.  创建和销毁会话。
    2.  维护一个从 `session_id` 到 `Message[]` (消息列表) 的映射。
    3.  每次调用「执行循环」时，将对应 `session_id` 的消息列表传递给状态管理器进行操作。
    4.  可以将这个映射存储在内存 (如 Redis) 或数据库中，以支持无状态的水平扩展。
    5.  可以将这个映射存储在内存 (如 Redis) 或数据库中，以支持无状态的水平扩展。
