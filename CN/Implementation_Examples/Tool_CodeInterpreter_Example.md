# 实作示例：Tool - 代码解释器 (Level 3 Isolation)

本文档展示如何实作一个高风险的 Tool 插件。为了符合「严格架构」的安全性要求，本工具**不会**在 Agent Core 的进程中直接执行代码，而是通过请求 Daemon 管理的 **Sandbox Infrastructure** 来执行。

## 架构原理

1.  **Agent Core**: 负责决策「要执行代码」。
2.  **Tool Plugin (`python:execute`)**: 负责将请求封装，并发送给 Sandbox Service。
3.  **Infrastructure (Daemon)**: 负责实际维护 Docker 容器或 VM，并执行代码。

---

## 实作代码 (PythonTool.ts)

```typescript
import { Tool, PluginContext, AgentContext, ToolExecutionResult } from '@openstarry/core/interfaces';

export default class PythonInterpreterTool implements Tool {
  id = 'python-interpreter';
  name = 'python:execute';
  description = 'Executes Python code in a secure sandbox.';
  
  // 定义参数 Schema (JSON Schema)
  parameters = {
    type: 'object',
    properties: {
      code: { type: 'string', description: 'The Python code to execute' },
      requirements: { type: 'array', items: { type: 'string' } }
    },
    required: ['code']
  };

  private sandboxClient: any; // 与基础设施通讯的客户端

  async init(context: PluginContext): Promise<void> {
    // 关键：从插件上下文获取 "Sandbox Infrastructure" 的客户端
    // 这确保了 Tool 不需要知道底层是 Docker 还是 gVisor
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

    // 1. 准备执行请求
    const executionRequest = {
      sessionId: agentContext.sessionId, // 保持会话状态 (如变量上下文)
      traceId: agentContext.traceId,     // 透传追踪 ID
      runtime: 'python:3.10',
      code: code,
      packages: requirements || []
    };

    try {
      // 2. 调用基础设施 (可能通过 gRPC 或 IPC)
      // 这一步是跨进程甚至跨机器的
      const result = await this.sandboxClient.runCode(executionRequest);

      // 3. 处理结果
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
      // 基础设施级别的错误 (如容器启动失败)
      return {
        success: false,
        output: null,
        error: `Sandbox Error: ${error.message}`
      };
    }
  }
}
```

## 为什么这样设计？

1.  **瘦客户端 (Thin Plugin)**: Tool 插件本身只有几十行代码，非常轻量。
2.  **安全性 (Security)**: 即使恶意代码攻破了 Sandbox，它攻破的也只是 Daemon 管理的一个临时容器，而不是 Agent Core 本身。Core 进程完全不受影响。
3.  **资源隔离**: Docker 容器的 CPU/RAM 限制由 Daemon 在启动 Infrastructure 时配置，Tool 插件无法绕过。
