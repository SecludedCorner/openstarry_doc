# Implementation Example: Tool - Code Interpreter (Level 3 Isolation)

This document demonstrates how to implement a high-risk Tool plugin. To comply with "Strict Architecture" security requirements, this tool **does not** execute code directly within the Agent Core's process. Instead, it issues requests to the **Sandbox Infrastructure** managed by the Daemon.

## Architectural Principles

1.  **Agent Core**: Responsible for the decision to "execute code."
2.  **Tool Plugin (`python:execute`)**: Responsible for encapsulating the request and sending it to the Sandbox Service.
3.  **Infrastructure (Daemon)**: Responsible for the actual maintenance of Docker containers or VMs and executing the code.

---

## Implementation Code (PythonTool.ts)

```typescript
import { Tool, PluginContext, AgentContext, ToolExecutionResult } from '@openstarry/core/interfaces';

export default class PythonInterpreterTool implements Tool {
  id = 'python-interpreter';
  name = 'python:execute';
  description = 'Executes Python code in a secure sandbox.';
  
  // Define parameter Schema (JSON Schema)
  parameters = {
    type: 'object',
    properties: {
      code: { type: 'string', description: 'The Python code to execute' },
      requirements: { type: 'array', items: { type: 'string' } }
    },
    required: ['code']
  };

  private sandboxClient: any; // Client for communicating with infrastructure

  async init(context: PluginContext): Promise<void> {
    // Critical: Retrieve the client for "Sandbox Infrastructure" from the plugin context.
    // This ensures the Tool remains oblivious to whether the underlying layer is Docker or gVisor.
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

    // 1. Prepare execution request
    const executionRequest = {
      sessionId: agentContext.sessionId, // Maintain session state (e.g., variable context)
      traceId: agentContext.traceId,     // Propagate trace ID
      runtime: 'python:3.10',
      code: code,
      packages: requirements || []
    };

    try {
      // 2. Invoke infrastructure (potentially via gRPC or IPC)
      // This step is cross-process or even cross-machine.
      const result = await this.sandboxClient.runCode(executionRequest);

      // 3. Process results
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
      // Infrastructure-level errors (e.g., container failed to start)
      return {
        success: false,
        output: null,
        error: `Sandbox Error: ${error.message}`
      };
    }
  }
}
```

## Why this design?

1.  **Thin Plugin**: The Tool plugin itself consists of only a few dozen lines of code, making it very lightweight.
2.  **Security**: Even if malicious code breaches the Sandbox, it only compromises a temporary container managed by the Daemon, not the Agent Core itself. The Core process remains completely unaffected.
3.  **Resource Isolation**: CPU/RAM limits for Docker containers are configured by the Daemon when starting the Infrastructure; the Tool plugin cannot bypass them.
