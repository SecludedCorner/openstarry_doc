# 16. OpenStarry 标准协议 (OpenStarry Standard Protocol)

本文档定义了 OpenStarry 生态系中，Agent Core 与外部插件（特别是 Provider 与 Tools）沟通的技术标准。所有官方与第三方插件都必须遵守此协议，以确保跨组件的互操作性。

---

## 1. 指令路径对比：斜杠指令 vs. 自主行为

在 OpenStarry 中，同样的一个工具（如 `fs.read`）可以通过两条截然不同的路径被触发：

| 特性 | 斜杠指令 (Slash Commands) | 自主行为 (Autonomous Actions) |
| :--- | :--- | :--- |
| **触发源** | 用户 (User) | 代理人大脑 (LLM/Provider) |
| **触发方式** | 输入 `/read filename.txt` | LLM 决定调用工具并回传 `tool_call` |
| **绕过思考** | 是 (直接执行) | 否 (经过 OODA 循环决策) |
| **权限模型** | 用户授权 (User Authorized) | 代理人自律 (Agent Autonomy) |
| **处理逻辑** | 由 `ExecutionLoop` 前置解析 | 由 `ExecutionLoop` 解析 `ProviderResponse` |

---

## 2. 转译指南：从异质 API 到标准格式

Provider 的核心价值在于「消除差异」。以下是以 Google Gemini API 为例的转译虚拟代码：

### A. 上行转译 (将 Core 工具转为 API 格式)
```typescript
// Core 的工具定义
const coreTool: ITool = { name: "read", description: "...", parameters: { ... } };

// 转译为 Gemini 格式
const geminiFunction = {
  name: coreTool.name,
  description: coreTool.description,
  parameters: coreTool.parameters // Gemini 接受 JSON Schema
};
```

### B. 下行转译 (将 API 回应转为 Core 协议)
```typescript
// LLM API 的原始回传 (Raw Content)
const geminiPart = { functionCall: { name: "read", args: { path: "a.txt" } } };

// 转译为 OpenStarry 标准协议
const response: ProviderResponse = {
  segments: [{
    type: 'tool_call',
    toolCall: {
      name: geminiPart.functionCall.name,
      args: geminiPart.functionCall.args
    }
  }]
};
```

---

## 3. 类型定义 (Type Definitions)

这些定义位于 `@openstarry/sdk` 的 `interfaces.ts` 中。

### A. 工具调用结构 (ToolCall)
这是 Agent 意图执行某个动作的标准载体。

```typescript
export interface ToolCall {
  /**
   * 调用的唯一识别码 (用于追踪与回调匹配)
   * 某些 LLM (如 OpenAI) 会提供此 ID，若 LLM 未提供，Provider 应自动生成一个 UUID。
   */
  id?: string;

  /**
   * 要调用的工具名称 (必须与注册的 tool.name 完全匹配)
   */
  name: string;

  /**
   * 传递给工具的参数对象
   */
  args: any;
}
```

### B. Provider 回应结构 (ProviderResponse)
这是 `IProvider.generate()` 方法必须回传的标准结果。为了支持多模态 (VLLM, Audio, UI)，我们采用 **分段式内容 (Segmented Content)** 设计。

```typescript
export type ContentType = 'text' | 'image' | 'audio' | 'video' | 'ui' | 'tool_call';

export interface ContentSegment {
  type: ContentType;
  
  /**
   * 用于 type='text'
   */
  text?: string;
  
  /**
   * 用于 type='image' | 'audio' | 'video'
   * 通常是 Base64 编码的数据或 URL
   */
  data?: string;
  
  /**
   * 媒体类型 (MIME Type), e.g., "image/png", "audio/mp3"
   */
  mimeType?: string;

  /**
   * 用于 type='tool_call'
   */
  toolCall?: ToolCall;

  /**
   * 用于 type='ui'
   * 描述前端应渲染的组件 (JSON)
   */
  ui?: any;
}

export interface ProviderResponse {
  /**
   * 有序的内容段落列表
   * Agent 可以同时说话、展示图片并执行工具。
   */
  segments: ContentSegment[];

  /**
   * 额外的元数据 (如 Token 使用量、模型延迟等)
   */
  metadata?: any;
}
```

---

## 2. 接口行为规范 (Interface Behavior Specifications)

### IProvider (想蕴)

Provider 插件必须实作 `generate` 方法，并遵循以下行为：

1.  **能力感知:** Provider 应从 `context.tools` 读取当前可用的工具列表。
2.  **协议转换:** 将 `context.tools` 转换为目标 LLM API 所需的 Schema 格式。
3.  **结果解析:** 接收 LLM API 的回应，并将其标准化为 `ProviderResponse`。
    *   **纯文字:** 回传 `[{ type: 'text', text: '...' }]`
    *   **工具调用:** 回传 `[{ type: 'tool_call', toolCall: { ... } }]`
    *   **混合模式:** 回传多个 Segment。例如 Gemini 可能先回传一段解释 (text)，再回传一个工具调用 (tool_call)。

```typescript
export interface IProvider {
  name: string;
  generate(prompt: string, context: IAgentContext): Promise<ProviderResponse>;
}
```

### IAgentContext (环境上下文)

Core 必须将其自身的状态与能力，通过 Context 暴露给 Provider。

```typescript
export interface IAgentContext {
  // ... 其他属性
  
  /**
   * 能力反射 (Capability Reflection)
   * 让 Provider 知道 Agent 当前拥有什么工具 (行蕴)。
   * Map key 为工具名称。
   */
  tools: Map<string, ITool>;
}
```

---

## 3. 交互时序示例 (Sequence Example)

以下是一个标准的「思考-行动」循环的协议流：

1.  **Input:** 用户输入 "查询台北天气"。
2.  **Core -> Provider:** 调用 `generate("查询台北天气", context)`。
    *   *Context 包含 `get_weather` 工具定义。*
3.  **Provider -> LLM API:** 发送 Prompt + Function Definition。
4.  **LLM API -> Provider:** 回传原始 JSON (例如 Gemini 的 `functionCall` 结构)。
5.  **Provider (适配):** 将原始 JSON 转换为 `ProviderResponse`:
    ```json
    {
      "content": "好的，我来查询。",
      "toolCalls": [{ "name": "get_weather", "args": { "location": "Taipei" } }]
    }
    ```
6.  **Provider -> Core:** 回传上述对象。
7.  **Core (执行):** 检测到 `toolCalls`，根据 `name` 查找 `context.tools` 并执行。
8.  **Core (反馈):** 将工具执行结果 (如 "25°C, Sunny") 存入记忆，并再次调用 Provider (进入下一轮)。

---

## 4. 进阶模式：大规模工具管理 (Advanced Pattern)

目前的标准协议采用「即时注入 (Just-in-Time Injection)」模式，适合中小型 Agent。对于拥有数百个工具的大型系统，建议采用以下优化策略：

### A. 静态注册与缓存 (Registry & Caching)
针对支持 Context Caching 的 LLM (如 OpenAI Assistants 或 Gemini 1.5)，Provider 应在初始化阶段一次性上传工具 Schema，获取 `tools_id`。后续对话仅需传递 ID，大幅降低 Token 消耗与延迟。

### B. 动态上下文过滤 (Dynamic Context Filtering)
为了避免 Context Window 被大量工具定义塞满，Core 可引入「工具检索层 (Tool Retrieval Layer)」：
1.  **意图识别:** 分析用户当前输入的意图 (Intent)。
2.  **向量检索:** 从工具库中检索出 Top-N 最相关的工具。
3.  **精准注入:** 仅将这 N 个工具注入给 Provider，而非全部。

这确保了 Agent 即使拥有成千上万的技能，也能保持轻量与专注。

---

## 5. 错误处理规范

*   **解析失败:** 如果 Provider 无法解析 LLM 的输出 (例如无效的 JSON)，应捕捉错误并在 `content` 中回传错误描述，或抛出标准 `AgentPainException`。
*   **工具不存在:** 如果 Provider 回传了一个不存在的 `toolCall.name`，Core 应捕捉此错误并将错误信息作为「系统观察」反馈给下一轮对话，让 LLM 有机会自我修正。
