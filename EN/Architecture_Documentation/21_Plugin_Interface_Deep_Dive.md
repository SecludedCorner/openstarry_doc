# 21. Plugin Interface & Aggregates Deep Dive

> **Technical Specification Note:**
> This document focuses on architectural philosophy and conceptual explanations. For strict TypeScript definitions and the latest API specifications for interfaces such as `IPlugin`, `ITool`, and `IAgentGuide`, please refer to **[03_Plugin_Interface_Definitions.md](../Technical_Specifications/03_Plugin_Interface_Definitions.md)**.

This document aims to clarify the architectural relationship between `IPlugin` (plugin container), `IPluginContext` (interaction environment), and `ITool/IListener` (the five aggregates components).

## 1. Why `IPlugin` exists alongside the five aggregates interfaces?

This is a distinction between "Container" and "Content."

*   **The Five Aggregates (ITool, IListener...):** These are the **"parts."** For example, a weather query function is an `ITool`, and a Discord listener is an `IListener`.
*   **Plugin (IPlugin):** This is the **"package/delivery box."**

The Core cannot know out of thin air which `ITool`s exist on the disk. It needs a standardized entry point to "open the package" and extract the parts. `IPlugin` serves as this standardized entry point.

### Analogy
*   **Core:** The computer case/motherboard.
*   **ITool (Volition):** The mouse.
*   **IListener (Sensation):** The keyboard input.
*   **IUI (Form):** The monitor output.
*   **IPlugin:** The **USB port and driver**. It is responsible for telling the host: "Hey, I have a mouse, a keyboard input, and a monitor output here; please drive these."

**Crucial Distinction:** `IListener` and `IUI` are separate interfaces:
- `IListener` (Sensation) focuses on receiving external input and pushing events via `ctx.pushInput()`.
- `IUI` (Form) focuses on output rendering, receiving and presenting events via `onEvent()`.

### Code Relationship
```typescript
// Developers follow the IPlugin interface to define the "package"
export default class MyPlugin implements IPlugin {
  // When the Core calls this method, it's equivalent to "plugging in the USB"
  async initialize(context: IPluginContext) {
    // Here, the developer takes out the "parts (aggregates)" and hands them to the Core
    context.registerTool(new WeatherTool()); // WeatherTool implements ITool
    context.registerListener(new DiscordListener()); // DiscordListener implements IListener
  }
}
```

---

## 2. Why is `IPluginContext` necessary?

`IPluginContext` acts as the **"transaction counter"** or **"exchange protocol"** between the Core and the Plugin. It carries information flow in two directions:

### Direction A: Core -> Plugin (Granting Capabilities)
A Plugin requires infrastructure provided by the Core to function.
*   **Logger:** Allows the Plugin to write logs.
*   **Config:** Allows the Plugin to read settings from `agent.json`.
*   **Dependencies:** (Critical) Allows the Plugin to access services provided by other Plugins.

### Direction B: Plugin -> Core (Delivering Components)
A Plugin needs a way to "register" its aggregate components into the Core system.
*   `registerTool()`
*   `registerListener()`
*   `pushInput(event: InputEvent)` — Allows the plugin to actively push input events to the Core (e.g., after receiving user input, a Stdio Listener injects the event into the execution loop via `context.pushInput()`, rather than through a constructor callback).

Without `IPluginContext`, a Plugin becomes an island, unable to acquire resources from the Core or deliver its functionality to the system.

> **Design Note: `pushInput` replaces Constructor Callbacks**
> In earlier implementations, Listener plugins relied on callbacks injected via the constructor to return user input to the Core. This created tight coupling between the Plugin and the Core. In the current architecture, all plugins push input events via `context.pushInput(event)`, achieving complete Inversion of Control (IoC).

> **Design Note: Dynamic Loading replaces BUILTIN_FACTORIES**
> The old architecture included a `BUILTIN_FACTORIES` map that mapped short names to built-in plugin factories. This mechanism has been entirely removed. All plugins (including official standard ones) are dynamically loaded via `import()`. The Core no longer hardcodes any plugin references, ensuring absolute kernel purity.

---

## 3. The Coordination Layer and the `dependencies` field

The "Coordination Layer having a dependencies field" refers to the **runtime dependency injection mechanism**.

### Scenario: A Workflow plugin needs to understand MD files
1.  **Skill Plugin (Provider):** Implements a `MarkdownParser` service.
2.  **Workflow Plugin (Consumer):** Requires this Parser to function.

### The Work of the Coordination Layer (Daemon)
Before starting the Workflow Plugin, the Coordination Layer performs the following:

```typescript
// 1. First, get the Parser instance from the Skill Plugin
const mdParser = skillPlugin.getService();

// 2. Prepare the Context for the Workflow Plugin
const contextForWorkflow = {
  logger: ...,
  // 3. Stuff the Parser into the dependencies field
  dependencies: {
    markdownParser: mdParser 
  }
};

// 4. Initialize the Workflow Plugin
workflowPlugin.initialize(contextForWorkflow);
```

### Developer's Perspective
Within the Workflow Plugin code:

```typescript
initialize(context: IPluginContext) {
  // Retrieve the dependencies prepared by the Coordination Layer from the Context
  const parser = context.dependencies.markdownParser;
  
  // Begin using it
  parser.parse("workflow.md");
}
```

## 4. Summary

*   **Aggregates Interfaces (`ITool`, etc.)**: Define **"what it is"** (it's a tool).
*   **`IPlugin`**: Defines **"how to load it"** (the initialization entry point).
*   **`IPluginContext`**: Defines **"how to communicate"** (the channel for exchanging resources and capabilities).

All three are indispensable and together constitute a pluggable, collaborative ecosystem.
