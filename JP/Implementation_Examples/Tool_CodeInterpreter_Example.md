# 実装例：ツール (Tool) - コードインタープリタ (レベル 3 隔離)

このドキュメントでは、高リスクな Tool プラグインの実装方法を示します。「厳格なアーキテクチャ」のセキュリティ要件に適合するため、このツールはエージェントコアのプロセス内でコードを直接実行 **しません** 。代わりに、デーモンが管理する **サンドボックス・インフラストラクチャ (Sandbox Infrastructure)** に対して実行をリクエストします。

## アーキテクチャの原理

1.  **エージェントコア**：「コードを実行する」という意思決定を担当します。
2.  **ツールプラグイン ( `python:execute` )**：リクエストをカプセル化し、サンドボックスサービスに送信する役割を担います。
3.  **インフラストラクチャ (Daemon)**：実際に Docker コンテナや VM を維持し、コードを実行する役割を担います。

---

## 実装コード (PythonTool.ts)

```typescript
import { Tool, PluginContext, AgentContext, ToolExecutionResult } from '@openstarry/core/interfaces';

export default class PythonInterpreterTool implements Tool {
  id = 'python-interpreter';
  name = 'python:execute';
  description = 'Executes Python code in a secure sandbox.';
  
  // 引数スキーマの定義 (JSON Schema)
  parameters = {
    type: 'object',
    properties: {
      code: { type: 'string', description: 'The Python code to execute' },
      requirements: { type: 'array', items: { type: 'string' } }
    },
    required: ['code']
  };

  private sandboxClient: any; // インフラストラクチャと通信するためのクライアント

  async init(context: PluginContext): Promise<void> {
    // 重要：プラグインコンテキストから "Sandbox Infrastructure" のクライアントを取得します
    // これにより、ツールは下層が Docker なのか gVisor なのかを知る必要がなくなります
    this.sandboxClient = context.infrastructure.getService('sandbox_manager');
    
    if (!this.sandboxClient) {
      throw new Error("Sandbox Infrastructure not available. Cannot initialize Python Tool.");
    }
  }

  async execute(
    agentContext: AgentContext, 
    args: any
  ): Promise<ToolExecutionResult> {
    const { code, requirements } = args;

    // 1. 実行リクエストの準備
    const executionRequest = {
      sessionId: agentContext.sessionId, // セッション状態（変数のコンテキストなど）を維持
      traceId: agentContext.traceId,     // トレース ID を透過
      runtime: 'python:3.10',
      code: code,
      packages: requirements || []
    };

    try {
      // 2. インフラストラクチャの呼び出し（ gRPC または IPC を介する可能性がある）
      // このステップはプロセス間、あるいはマシン間で行われます
      const result = await this.sandboxClient.runCode(executionRequest);

      // 3. 結果の処理
      if (result.exitCode !== 0) {
        return {
          success: false,
          output: result.stderr,
          error: "Execution failed with non-zero exit code"
        };
      }

      return {
        success: true,
        output: result.stdout,
        metadata: {
          executionTime: result.durationMs,
          memoryUsage: result.memoryUsage
        }
      };

    } catch (error) {
      // インフラストラクチャレベルのエラー（コンテナの起動失敗など）
      return {
        success: false,
        output: null,
        error: `Sandbox Error: ${error.message}`
      };
    }
  }
}
```

## なぜこのような設計なのか？

1.  **シンクライアント (Thin Plugin)**：ツールプラグイン自体は数十行のコードのみで構成され、非常に軽量です。
2.  **セキュリティ (Security)**：たとえ悪意のあるコードがサンドボックスを突破したとしても、それはデーモンが管理する一時的なコンテナを突破したにすぎず、エージェントコア自体ではありません。コアプロセスは一切影響を受けません。
3.  **リソース隔離**： Docker コンテナの CPU/RAM 制限は、インフラストラクチャの起動時にデーモンによって設定されます。ツールプラグインがこれを回避することはできません。
