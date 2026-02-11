# 18. Plugin Loading and Registration Protocol

This document defines the detailed protocol for how the OpenStarry system loads plugin code from disk and interfaces it with the core system. This is a core technical specification for Phase 3.

## 1. Core Challenges

Since plugins are "Five Aggregates aggregates" and the internal operation of the Core is modular (ToolManager, ListenerManager, etc.), the loader must resolve the following issues:
1.  **Dynamism:** Plugin code is unknown at compile time.
2.  **Destructuring:** Disassembling a plugin package into multiple functional units.
3.  **Dependency Injection:** Allowing plugins to acquire Core capabilities (e.g., Logger, Config) without making the plugins dependent on the Core's implementation.

## 2. Loading Protocol: Factory Pattern

To maximize flexibility, we abandon the "Class Scanning" model in favor of the **Factory Function Pattern**.

> **Important Update:** The original `BUILTIN_FACTORIES` hardcoded mapping table has been removed. All plugins (including built-in ones) are now loaded via a unified dynamic loading mechanism: `ref.path` (local path) is prioritized, otherwise `import(ref.name)` (NPM package name) is used. This ensures consistency in plugin loading logic and eliminates the distinction between built-in and external plugins.

### Plugin Entry Specification (`index.ts`)

Every plugin must export an asynchronous function named `initialize`, or a default export that implements the `IPluginFactory` interface.

```typescript
import { IPluginContext, IPlugin } from '@openstarry/sdk';
import { MyTool } from './tools/my-tool';
import { MyListener } from './listeners/my-listener';

// Plugin Factory Function
export default async function initialize(context: IPluginContext): Promise<void> {
  // 1. Get dependencies injected by the Core
  const logger = context.logger;
  logger.info('Initializing My Plugin...');

  // 2. Register Volition (Tools)
  // The plugin actively hands over its Tool instance to the Core
  context.registerTool(new MyTool());

  // 3. Register Sensation (Listeners)
  context.registerListener(new MyListener(context.config));
  
  // 4. Register Consciousness (Guide)
  context.registerGuide({
    systemPrompt: "...",
    memoryPolicy: "sliding-window"
  });
}
```

## 3. Host-side Loading Workflow (`PluginLoader` Logic)

The `PluginLoader` runs within the **Coordination Layer / Daemon** or the **Host Process**.

### Step A: Physical Reading
1.  Read `~/.openstarry/plugins/<plugin-id>/package.json`.
2.  Parse the `main` field to find the entry file (e.g., `dist/index.js`).
3.  Dynamically load the module using `import()` (ESM) or `require()` (CJS).

### Step B: Context Construction
The Loader creates a dedicated `PluginContext` instance for the plugin. This Context is the sole channel for the plugin to communicate with the outside world.

```typescript
const context: IPluginContext = {
  logger: new ScopedLogger(`Plugin:${pluginId}`),
  config: agentConfig.plugins[pluginId] || {}, // Read configuration from agent.json
  
  // Registration Callbacks
  registerTool: (tool) => core.toolRegistry.add(tool),
  registerListener: (listener) => core.listenerRegistry.add(listener),
  registerProvider: (provider) => core.providerRegistry.add(provider),
  registerGuide: (guide) => core.guideRegistry.set(guide),

  // Input Injection — Plugins can actively push messages to the Agent via pushInput
  pushInput: (input) => core.inputQueue.push(input),
};
```

### Step C: Initialization and Destructuring
Invoke the plugin's `initialize(context)`.
*   At this point, the internal logic of the plugin begins execution and "hands over" its five aggregate components via the `register*` methods.
*   The Core receives these components and archives them into their respective managers.

## 4. Error Isolation and Sandboxing

*   **Loading Failure:** If `initialize` throws an exception, the Loader should catch and record it as "pain," but it must not allow the entire Agent to crash. The plugin will be marked as `FAILED`.
*   **Dependency Injection Security:** `IPluginContext` only exposes safe APIs. Plugins cannot directly access the Core's private properties or the host's `process` object (unless in Level 0 trust mode).

## 5. Coordination Layer Registration

The metadata (ID, Version, Capabilities) of all successfully loaded plugins is reported to the **Agent Coordination Layer**. This enables the coordination layer to answer questions like "Who has the weather query capability?", enabling dynamic routing across Agents.

---

## 6. Dependency Resolution and Topological Loading

To prevent system crashes due to incorrect loading sequences (e.g., a Workflow plugin loading before a Skill plugin, resulting in the unavailability of a Markdown Parser), the Loader **must** implement a dependency graph resolution algorithm.

### Two-Phase Loading Workflow

#### Phase A: Scan and Graph Construction
1.  The Loader scans all target plugin directories.
2.  Reads the `openstarry.dependencies` field in each plugin's `package.json`.
    ```json
    "dependencies": {
      "plugins": ["@openstarry-plugin/skill"] // Declare hard dependencies on other plugins
    }
    ```
3.  Builds a **Dependency Directed Graph** in memory.
    *   Nodes: Plugin ID
    *   Edges: Dependency -> Dependent

#### Phase B: Sorting and Execution
1.  **Cycle Detection:** Checks if circular dependencies (A -> B -> A) exist in the graph. If found, throws a `FatalDependencyError` immediately and terminates startup.
2.  **Topological Sort:** Uses Kahn's algorithm or DFS to calculate a linear loading sequence.
    *   *Result:* `['@openstarry-plugin/skill', '@openstarry-plugin/gemini', '@openstarry-plugin/workflow']`
3.  **Sequential Initialization:** Invokes `initialize(context)` sequentially according to the sorted list.
    *   This ensures that when `workflow` is initialized, `skill` is already prepared and has injected the Parser into the system.

### Timing of Dependency Injection
Cross-plugin services (e.g., `markdownParser`) must be dynamically injected during the sorting process of **Phase B**.
*   After `@openstarry-plugin/skill` completes initialization, it returns (or registers) its service instance.
*   The Loader caches these instances.
*   When it's the turn of `@openstarry-plugin/workflow` to initialize, the Loader retrieves the `markdownParser` from the cache and places it into `context.dependencies`.