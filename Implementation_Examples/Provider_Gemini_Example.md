# 實作範例：Provider - Gemini API

> ⚠️ **[漂移更正 — v0.59.6 — 虛構介面契約]** 本範例描述的介面契約是**虛構的，已被取代**。下方所有 `LLMProvider` / `init()` / `generate()` / `@openstarry/core/interfaces` 內容**與真實代碼不符**，僅保留作歷史草稿：
>
> - **真實介面**：`IProvider extends ISamjna`，唯一方法是串流式 `chat(request: ChatRequest): AsyncIterable<ProviderStreamEvent>`（`packages/sdk/src/types/provider.ts:40`，`chat()` 簽章在 `:44`）。**沒有** `init()`、**沒有** `generate()`、**沒有** `Promise<LLMResponse>` 一次性回傳。
> - **匯入來源虛構**：`@openstarry/core/interfaces` 這個模組**不存在**。真實型別（`IProvider`、`IPlugin`、`ChatRequest`、`ProviderStreamEvent`、`Message`、`ModelInfo` 等）一律從 `@openstarry/sdk` 匯入。
> - **真實的 provider-gemini 實作**：`openstarry_plugin/provider-gemini/src/index.ts`。它走「工廠模式」匯出 `createGeminiPlugin(): IPlugin`（`:447`），由 `factory(ctx)` 在 `PluginHooks.providers` 內回傳 `createGeminiAdapter(keyManager): IProvider`（`:382`）；adapter 以 `async *chat(request: ChatRequest): AsyncGenerator<ProviderStreamEvent>`（`:394`）串流產出 `text_delta` / `tool_call_start|delta|end` / `finish` / `error` 事件，呼叫 `streamGenerateContent?alt=sse` 端點（`callGeminiStream`，`:122`）。API key 經 `SecureStore`（AES-256-GCM）加密落地，登入由 `/provider login gemini <API_KEY>` slash command 管理。
> - **沒有 `init(context)`/依賴注入 logger 的接口**：上下文是透過 `factory(ctx: IPluginContext)` 取得（`:456`），logger 由 `createLogger("gemini")`（`@openstarry/shared`）建立，而非 `pluginContext.logger`。
>
> 真實參考：介面 `packages/sdk/src/types/provider.ts`；實作 `openstarry_plugin/provider-gemini/src/index.ts`。以下原文為早期設計草稿，**請勿照抄**。

---

本文件提供一個遵循「一切皆插件」原則的 Gemini API Provider 實作範例。

## 核心介面契約

Provider 插件必須實現核心定義的 `LLMProvider` 接口。

```typescript
interface LLMProvider {
  id: string;
  // 初始化：接收插件上下文（包含配置、日誌器等）
  init(context: PluginContext): Promise<void>;
  // 生成：核心傳入當前上下文、歷史與工具定義
  generate(
    context: AgentContext, 
    history: ChatMessage[], 
    tools: ToolDefinition[]
  ): Promise<LLMResponse>;
}
```

---

## 實作代碼 (GeminiProvider.ts)

```typescript
import { LLMProvider, PluginContext, AgentContext, ChatMessage, ToolDefinition, LLMResponse } from '@openstarry/core/interfaces';
import { GoogleGenerativeAI } from '@google/generative-ai';

export default class GeminiProviderPlugin implements LLMProvider {
  id = 'provider-gemini';
  private client: GoogleGenerativeAI;
  private logger: any;

  /**
   * 初始化階段
   * 插件在此處讀取配置並建立連線，不阻塞 Core 啟動。
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
   * 生成階段
   * 嚴格透傳 traceId，並處理工具定義轉換
   */
  async generate(
    agentContext: AgentContext, 
    history: ChatMessage[], 
    tools: ToolDefinition[]
  ): Promise<LLMResponse> {
    const model = this.client.getGenerativeModel({ model: "gemini-pro" });
    
    // 1. 格式轉換：將 Core 的標準格式轉為 Gemini SDK 格式
    const chatHistory = this.convertToGeminiHistory(history);
    const toolConfig = this.convertToGeminiTools(tools);

    this.logger.debug("Calling Gemini API", { 
      traceId: agentContext.traceId, // 關鍵：Log 中必須包含 TraceID
      agentId: agentContext.agentId,
      messageCount: history.length 
    });

    try {
      // 2. 執行 API 調用
      const chat = model.startChat({
        history: chatHistory,
        tools: toolConfig
      });
      
      // 注意：這裡假設最後一條是新消息
      const lastMsg = history[history.length - 1].content;
      const result = await chat.sendMessage(lastMsg);
      const response = await result.response;

      // 3. 響應標準化：將 SDK 響應轉回 Core 的標準格式
      return this.parseGeminiResponse(response);

    } catch (error) {
      this.logger.error("Gemini API Call Failed", {
        traceId: agentContext.traceId,
        error: error.message
      });
      throw error; // 拋出錯誤由 Core 的執行循環捕獲
    }
  }

  // --- 輔助方法 (私有) ---

  private convertToGeminiHistory(history: ChatMessage[]) {
    // 實作標準格式到 Gemini Content 格式的映射
    // ...
    return []; 
  }

  private convertToGeminiTools(tools: ToolDefinition[]) {
    // 將 JSON Schema 轉換為 Gemini Function Declaration
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

## 關鍵改進點

1.  **`init(context)`**: 取代 `constructor`，符合異步初始化與依賴注入（如 logger）的需求。
2.  **`traceId` 透傳**: 在 Log 中嚴格記錄 `agentContext.traceId`，確保可觀測性。
3.  **錯誤處理**: 明確將錯誤拋出，交由 Core 的 `Error Handling` 機制處理（可能觸發重試或熔斷）。