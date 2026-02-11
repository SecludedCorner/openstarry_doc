# 実装例：プロバイダー (Provider) - Gemini API

このドキュメントでは、「すべてはプラグインである」という原則に従った Gemini API プロバイダーの実装例を提供します。

## コア・インターフェース契約

プロバイダープラグインは、コアで定義された `LLMProvider` インターフェースを実装する必要があります。

```typescript
interface LLMProvider {
  id: string;
  // 初期化：プラグインコンテキスト（設定、ロガーなどを含む）を受け取る
  init(context: PluginContext): Promise<void>;
  // 生成：コアが現在のコンテキスト、履歴、およびツール定義を渡す
  generate(
    context: AgentContext, 
    history: ChatMessage[], 
    tools: ToolDefinition[]
  ): Promise<LLMResponse>;
}
```

---

## 実装コード (GeminiProvider.ts)

```typescript
import { LLMProvider, PluginContext, AgentContext, ChatMessage, ToolDefinition, LLMResponse } from '@openstarry/core/interfaces';
import { GoogleGenerativeAI } from '@google/generative-ai';

export default class GeminiProviderPlugin implements LLMProvider {
  id = 'provider-gemini';
  private client: GoogleGenerativeAI;
  private logger: any;

  /**
   * 初期化フェーズ
   * プラグインはここで設定を読み取り、接続を確立します。コアの起動をブロックしません。
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
   * 生成フェーズ
   * traceId を厳格に透過させ、ツール定義の変換を処理します。
   */
  async generate(
    agentContext: AgentContext, 
    history: ChatMessage[], 
    tools: ToolDefinition[]
  ): Promise<LLMResponse> {
    const model = this.client.getGenerativeModel({ model: "gemini-pro" });
    
    // 1. 形式変換：コアの標準形式を Gemini SDK 形式に変換
    const chatHistory = this.convertToGeminiHistory(history);
    const toolConfig = this.convertToGeminiTools(tools);

    this.logger.debug("Calling Gemini API", { 
      traceId: agentContext.traceId, // 重要：ログには必ず TraceID を含める
      agentId: agentContext.agentId,
      messageCount: history.length 
    });

    try {
      // 2. API 呼び出しの実行
      const chat = model.startChat({
        history: chatHistory,
        tools: toolConfig
      });
      
      // 注意：ここでは最後のメッセージが新しい入力であると仮定しています
      const lastMsg = history[history.length - 1].content;
      const result = await chat.sendMessage(lastMsg);
      const response = await result.response;

      // 3. 応答の標準化： SDK の応答をコアの標準形式に戻す
      return this.parseGeminiResponse(response);

    } catch (error) {
      this.logger.error("Gemini API Call Failed", {
        traceId: agentContext.traceId,
        error: error.message
      });
      throw error; // エラーをスローしてコアの実行ループでキャッチさせる
    }
  }

  // --- ヘルパーメソッド (private) ---

  private convertToGeminiHistory(history: ChatMessage[]) {
    // 標準形式から Gemini Content 形式へのマッピングを実装
    // ...
    return []; 
  }

  private convertToGeminiTools(tools: ToolDefinition[]) {
    // JSON Schema を Gemini Function Declaration に変換
    // ...
    return [];
  }

  private parseGeminiResponse(response: any): LLMResponse {
    // text または functionCall を解析
    // ...
    return {
      content: "",
      toolCalls: []
    };
  }
}
```

## 主要な改善ポイント

1.  **`init(context)`**： `constructor` に代わるもので、非同期初期化と依存性の注入（ロガーなど）のニーズに対応します。
2.  **`traceId` の透過**：ログに `agentContext.traceId` を厳格に記録し、可観測性を確保します。
3.  **エラー処理**：エラーを明示的にスローし、コアの `Error Handling` メカニズム（再試行や遮断をトリガーする可能性がある）に処理を任せます。
