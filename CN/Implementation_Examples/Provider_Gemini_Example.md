# 实作示例：Provider - Gemini API

本文档提供一个遵循「一切皆插件」原则的 Gemini API Provider 实作示例。

## 核心接口契约

Provider 插件必须实现核心定义的 `LLMProvider` 接口。

```typescript
interface LLMProvider {
  id: string;
  // 初始化：接收插件上下文（包含配置、日志器等）
  init(context: PluginContext): Promise<void>;
  // 生成：核心传入当前上下文、历史与工具定义
  generate(
    context: AgentContext, 
    history: ChatMessage[], 
    tools: ToolDefinition[]
  ): Promise<LLMResponse>;
}
```

---

## 实作代码 (GeminiProvider.ts)

```typescript
import { LLMProvider, PluginContext, AgentContext, ChatMessage, ToolDefinition, LLMResponse } from '@openstarry/core/interfaces';
import { GoogleGenerativeAI } from '@google/generative-ai';

export default class GeminiProviderPlugin implements LLMProvider {
  id = 'provider-gemini';
  private client: GoogleGenerativeAI;
  private logger: any;

  /**
   * 初始化阶段
   * 插件在此处读取配置并建立连线，不阻塞 Core 启动。
   */
  async init(pluginContext: PluginContext): Promise<void> {
    this.logger = pluginContext.logger;
    const apiKey = pluginContext.config.apiKey;
    
    if (!apiKey) {
      throw new Error("Gemini Provider requires 'apiKey' in plugin config.");
    }

    this.client = new GoogleGenerativeAI(apiKey);
    this.logger.info("Gemini Provider initialized", { module: 'GeminiProvider' });
  }

  /**
   * 生成阶段
   * 严格透传 traceId，并处理工具定义转换
   */
  async generate(
    agentContext: AgentContext, 
    history: ChatMessage[], 
    tools: ToolDefinition[]
  ): Promise<LLMResponse> {
    const model = this.client.getGenerativeModel({ model: "gemini-pro" });
    
    // 1. 格式转换：将 Core 的标准格式转为 Gemini SDK 格式
    const chatHistory = this.convertToGeminiHistory(history);
    const toolConfig = this.convertToGeminiTools(tools);

    this.logger.debug("Calling Gemini API", { 
      traceId: agentContext.traceId, // 关键：Log 中必须包含 TraceID
      agentId: agentContext.agentId,
      messageCount: history.length 
    });

    try {
      // 2. 执行 API 调用
      const chat = model.startChat({
        history: chatHistory,
        tools: toolConfig
      });
      
      // 注意：这里假设最后一条是新消息
      const lastMsg = history[history.length - 1].content;
      const result = await chat.sendMessage(lastMsg);
      const response = await result.response;

      // 3. 响应标准化：将 SDK 响应转回 Core 的标准格式
      return this.parseGeminiResponse(response);

    } catch (error) {
      this.logger.error("Gemini API Call Failed", {
        traceId: agentContext.traceId,
        error: error.message
      });
      throw error; // 抛出错误由 Core 的执行循环捕获
    }
  }

  // --- 辅助方法 (私有) ---

  private convertToGeminiHistory(history: ChatMessage[]) {
    // 实作标准格式到 Gemini Content 格式的映射
    // ...
    return []; 
  }

  private convertToGeminiTools(tools: ToolDefinition[]) {
    // 将 JSON Schema 转换为 Gemini Function Declaration
    // ...
    return [];
  }

  private parseGeminiResponse(response: any): LLMResponse {
    // 解析 text 或 functionCall
    // ...
    return {
      content: "",
      toolCalls: []
    };
  }
}
```

## 关键改进点

1.  **`init(context)`**: 取代 `constructor`，符合异步初始化与依赖注入（如 logger）的需求。
2.  **`traceId` 透传**: 在 Log 中严格记录 `agentContext.traceId`，确保可观测性。
3.  **错误处理**: 明确将错误抛出，交由 Core 的 `Error Handling` 机制处理（可能触发重试或熔断）。
