# 12. Capabilities Discovery & Injection Mechanism

This document defines how the OpenStarry Coordination Layer loads plugins and injects the functionalities they provide into the various subsystems of the Agent Core. This mechanism is key to achieving a "microkernel architecture" and "dynamic extensibility."

## 1. Mechanism Overview

We adopt the **Inversion of Control (IoC)** and **Dependency Injection (DI)** patterns.

*   **Host (The Driver):** The host process (CLI or Daemon) that starts the Agent. **Only the Host possesses physical I/O privileges**, enabling it to read plugin files from the disk.
*   **PluginLoader (The Injector):** Runs within the context of the Host, responsible for transforming physical files into kernel objects.
*   **Core (The Recipient):** A pure recipient that acquires injected capabilities passively.

> **💡 Design Philosophy:**
> This corresponds to "Form is not different from Emptiness" in the Five Aggregates. The Plugin (Form) is an aggregation of concrete functionalities, while the Core (Emptiness) is a pure execution container. The Loader's responsibility is to disassemble the aggregation and fill the container with capabilities.

---

## 2. Core Interface Definitions

All interactions are based on contracts defined in `@openstarry/sdk`.

### 2.1 The Plugin Contract

Every plugin must export a class that implements this interface.

```typescript
export interface IPlugin {
  readonly id: string;
  readonly version: string;
  
  /**
   * Initializes the plugin.
   * @param context - The registration context provided by the coordination layer, used to return capabilities.
   */
  initialize(context: IPluginContext): Promise<void>;
  
  /**
   * Resource cleanup (e.g., closing WebSocket connections).
   */
  shutdown(): Promise<void>;
}
```

### 2.2 The Registration Context

This is the API exposed by the coordination layer to plugins for collecting capabilities.

```typescript
export interface IPluginContext {
  // Infrastructure
  readonly logger: ILogger;
  readonly config: Record<string, any>; // The configuration segment for this plugin from agent.json

  // [Volition] Register Tool: Functions callable by the LLM
  registerTool(tool: ITool): void;
  
  // [Sensation] Register Listener: External event sources that trigger Agent operation
  registerListener(listener: IListener): void;
  
  // [Perception] Set Model: The Agent's brain (usually mutually exclusive; latecomers override or throw errors)
  setLLMProvider(provider: ILLMProvider): void;
}
```

---

## 3. Loading and Injection Sequence

The following is the standard operating procedure for the `PluginLoader` when the `Agent Core` starts:

1.  **Resolve & Discovery:** 
    *   Reads the `plugins` ID list from `agent.json`.
    *   **Query Global Registry:** Calls `PluginRegistryService.resolve(id)`.
    *   **Obtain Path:** The registry returns the precise physical path of the plugin (having resolved system/project priorities).

2.  **Dynamic Import:**
    Loads the entry file at that path using Node.js's `import()` or `require()`.

3.  **Instantiate:**
    Creates an instance of `IPlugin`.

4.  **Initialize & Inject:**
    *   The Loader creates a `PluginContext` instance bound to the current Core instance (ToolRegistry, EventBus).
    *   Invokes `plugin.initialize(context)`.
    *   **Plugin Code Execution:** The plugin internally calls `context.registerTool(...)`.
    *   **Actual Binding:** Upon receiving a Tool, the `PluginContext` stores it in the Core's `ToolRegistry`.

5.  **Lifecycle Management:**
    The Loader maintains the plugin instance in an internal Map to invoke `shutdown()` when the Agent terminates.

---

## 4. Implementation Example

### Plugin Side (`plugins/standard/fs/index.ts`)

```typescript
import { IPlugin, IPluginContext } from '@openstarry/sdk';
import { ReadFileTool } from './tools/ReadFile';

export default class FileSystemPlugin implements IPlugin {
  id = 'openstarry-fs';
  version = '1.0.0';

  async initialize(context: IPluginContext) {
    context.logger.info('Mounting file system capabilities...');
    
    // Inject Tool
    context.registerTool(new ReadFileTool());
    
    // Inject write tool if permitted by configuration
    if (context.config.enableWrite) {
       // ... register WriteFileTool
    }
  }
  
  async shutdown() {}
}
```

### Core Side (`packages/core/infrastructure/PluginLoader.ts`)

```typescript
class PluginContextImpl implements IPluginContext {
  constructor(private core: AgentCore, private pluginConfig: any) {}

  registerTool(tool: ITool) {
    // Directly manipulate the Core's registry
    this.core.toolRegistry.register(tool);
  }
  
  // ... other methods
}
```

---

## 5. Error Handling

*   **Loading Failure:** If `require` fails, the Loader should log the error but must not cause the Core to crash (unless it is a critical plugin).
*   **Initialization Timeout:** The `initialize` method should have a timeout limit to prevent a plugin from stalling the startup process.
*   **Conflict Resolution:** If different plugins register Tools with the same name (e.g., two plugins both have `read_file`), the Loader should handle this according to configuration policy (override, error, or namespace isolation).
