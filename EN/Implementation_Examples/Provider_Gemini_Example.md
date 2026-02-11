# Implementation Example: Provider - Gemini API

This document provides an implementation example of a Gemini API Provider adhering to the "Everything is a Plugin" principle.

## Core Interface Contract

Provider plugins must implement the `LLMProvider` interface defined by the Core.

```typescript
interface LLMProvider {
  id: string;
  // Initialization: Receives plugin context (config, logger, etc.)
  init(context: PluginContext): Promise<void>;
  // Generation: Core passes current context, history, and tool definitions
  generate(
    context: AgentContext, 
    history: ChatMessage[], 
    tools: ToolDefinition[]
  ): Promise<LLMResponse>;
}
```

---

## Implementation Code (GeminiProvider.ts)

```typescript
import { LLMProvider, PluginContext, AgentContext, ChatMessage, ToolDefinition, LLMResponse } from '@openstarry/core/interfaces';
import { GoogleGenerativeAI } from '@google/generative-ai';

export default class GeminiProviderPlugin implements LLMProvider {
  id = 'provider-gemini';
  private client: GoogleGenerativeAI;
  private logger: any;

  /**
   * Initialization Phase
   * The plugin reads configuration and establishes connections here without blocking Core startup.
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
   * Generation Phase
   * Strictly propagates traceId and handles tool definition conversion.
   */
  async generate(
    agentContext: AgentContext, 
    history: ChatMessage[], 
    tools: ToolDefinition[]
  ): Promise<LLMResponse> {
    const model = this.client.getGenerativeModel({ model: "gemini-pro" });
    
    // 1. Format Conversion: Convert Core standard format to Gemini SDK format
    const chatHistory = this.convertToGeminiHistory(history);
    const toolConfig = this.convertToGeminiTools(tools);

    this.logger.debug("Calling Gemini API", { 
      traceId: agentContext.traceId, // Critical: TraceID must be included in logs
      agentId: agentContext.agentId,
      messageCount: history.length 
    });

    try {
      // 2. Execute API call
      const chat = model.startChat({
        history: chatHistory,
        tools: toolConfig
      });
      
      // Note: Assuming the last item is the new message
      const lastMsg = history[history.length - 1].content;
      const result = await chat.sendMessage(lastMsg);
      const response = await result.response;

      // 3. Response Normalization: Convert SDK response back to Core standard format
      return this.parseGeminiResponse(response);

    } catch (error) {
      this.logger.error("Gemini API Call Failed", {
        traceId: agentContext.traceId,
        error: error.message
      });
      throw error; // Rethrow error to be caught by the Core's execution loop
    }
  }

  // --- Helper Methods (Private) ---

  private convertToGeminiHistory(history: ChatMessage[]) {
    // Implementation of mapping standard format to Gemini Content format
    // ...
    return []; 
  }

  private convertToGeminiTools(tools: ToolDefinition[]) {
    // Convert JSON Schema to Gemini Function Declaration
    // ...
    return [];
  }

  private parseGeminiResponse(response: any): LLMResponse {
    // Parse text or functionCall
    // ...
    return {
      content: "",
      toolCalls: []
    };
  }
}
```

## Key Improvements

1.  **`init(context)`**: Replaces the `constructor` to satisfy asynchronous initialization and dependency injection (e.g., logger) requirements.
2.  **`traceId` Propagation**: Strictly logs `agentContext.traceId` to ensure observability.
3.  **Error Handling**: Explicitly throws errors to be managed by the Core's `Error Handling` mechanism (potentially triggering retries or circuit breaking).
