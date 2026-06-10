# 實作範例：Tool - 代碼解釋器 (Level 3 Isolation)

本文件展示如何實作一個高風險的 Tool 插件。為了符合「嚴格架構」的安全性要求，本工具**不會**在 Agent Core 的進程中直接執行代碼，而是透過請求 Daemon 管理的 **Sandbox Infrastructure** 來執行。

## 架構原理

1.  **Agent Core**: 負責決策「要執行代碼」。
2.  **Tool Plugin (`python:execute`)**: 負責將請求封裝，並發送給 Sandbox Service。
3.  **Infrastructure (Daemon)**: 負責實際維護 Docker 容器或 VM，並執行代碼。

---

## 實作代碼 (PythonTool.ts)

```typescript
import { Tool, PluginContext, AgentContext, ToolExecutionResult } from '@openstarry/core/interfaces';

export default class PythonInterpreterTool implements Tool {
  id = 'python-interpreter';
  name = 'python:execute';
  description = 'Executes Python code in a secure sandbox.';
  
  // 定義參數 Schema (JSON Schema)
  parameters = {
    type: 'object',
    properties: {
      code: { type: 'string', description: 'The Python code to execute' },
      requirements: { type: 'array', items: { type: 'string' } }
    },
    required: ['code']
  };

  private sandboxClient: any; // 與基礎設施通訊的客戶端

  async init(context: PluginContext): Promise<void> {
    // 關鍵：從插件上下文獲取 "Sandbox Infrastructure" 的客戶端
    // 這確保了 Tool 不需要知道底層是 Docker 還是 gVisor
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

    // 1. 準備執行請求
    const executionRequest = {
      sessionId: agentContext.sessionId, // 保持會話狀態 (如變量上下文)
      traceId: agentContext.traceId,     // 透傳追蹤 ID
      runtime: 'python:3.10',
      code: code,
      packages: requirements || []
    };

    try {
      // 2. 調用基礎設施 (可能透過 gRPC 或 IPC)
      // 這一步是跨進程甚至跨機器的
      const result = await this.sandboxClient.runCode(executionRequest);

      // 3. 處理結果
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
      // 基礎設施級別的錯誤 (如容器啟動失敗)
      return {
        success: false,
        output: null,
        error: `Sandbox Error: ${error.message}`
      };
    }
  }
}
```

## 為什麼這樣設計？

1.  **瘦客戶端 (Thin Plugin)**: Tool 插件本身只有幾十行代碼，非常輕量。
2.  **安全性 (Security)**: 即使惡意代碼攻破了 Sandbox，它攻破的也只是 Daemon 管理的一個臨時容器，而不是 Agent Core 本身。Core 進程完全不受影響。
3.  **資源隔離**: Docker 容器的 CPU/RAM 限制由 Daemon 在啟動 Infrastructure 時配置，Tool 插件無法繞過。