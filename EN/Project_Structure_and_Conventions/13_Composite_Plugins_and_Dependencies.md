# 13. Composites & Dependencies

This document defines how to build **"Composites"**—hardcoded modules that do not produce low-level capabilities but instead combine existing Providers (Perception) and Tools (Volition), adding custom Listeners (Sensation) to create new application scenarios.

## 1. Core Concept: Capability Dependency

To maintain decoupling, a Composite **should not** depend directly on the code of Plugin A. Instead, it should depend on the **"Capability Tag"** provided by Plugin A.

*   ❌ **Incorrect Dependency:** `import { GoogleSearch } from 'openstarry-plugin-google'`
*   ✅ **Correct Dependency:** `dependencies: { "tools": ["search"] }`

In this way, whether the user installs Google Search or Bing Search, as long as it provides the `search` capability, Plugin C can operate.

## 2. Manifest Declaration

In `plugin.json` or `package.json`, add a `dependencies` field pointing to other **plugin IDs**:

```json
{
  "id": "workflow-engine",
  "version": "1.0.0",
  "openstarry": {
    "type": "composite",
    "dependencies": {
      "plugins": ["@openstarry-plugin/standard-function-skill"] // Hard dependency
    }
  }
}
```

## 3. Infrastructure Dependency

Certain high-level plugins (such as workflow engines) depend on the system's foundational parsing capabilities.

*   **Scenario:** Requires parsing of Markdown skill files.
*   **Implementation:**
    ```typescript
    initialize(context: IPluginContext) {
      // Obtain the Parser from the dependencies injected by the coordination layer
      const parser = context.dependencies['@openstarry-plugin/standard-function-skill'].parser;
      
      if (!parser) throw new Error("Missing Skill Parser dependency");
    }
    ```

## 4. Responsibility of the Coordination Layer (Validation)

When the `PluginLoader` loads Plugin C, it performs a **Dependency Check**:

1.  Checks if an LLM is registered in the Core's `ProviderRegistry`.
2.  Checks if tools with `web-search` and `fs-write` tags exist in the Core's `ToolRegistry`.
3.  **Outcome:** If the check fails, Plugin C will **refuse to start** and return an error prompting the user to install the missing plugins.

## 5. Implementation: How to Invoke Capabilities?

Within the plugin code, capabilities are accessed via the `AgentContext` (Note: not the `PluginContext` from initialization, but the `AgentContext` provided by runtime events).

```typescript
// plugins/experimental/research-assistant/index.ts

export default class ResearchPlugin implements IPlugin {
  async initialize(context: IPluginContext) {
    // Register a listener to wait for tasks
    context.registerListener({
      id: 'research-trigger',
      onEvent: async (event, agentOps: IAgentOperations) => {
        
        // 1. Obtain the brain (Perception)
        const llm = agentOps.getLLM();
        
        // 2. Obtain the tool (Volition)
        // The system automatically searches for tools tagged with 'web-search'
        const searchTool = agentOps.findToolByTag('web-search');
        
        if (!searchTool) {
          throw new Error("Missing required capability: web-search");
        }

        // 3. Execute logic
        const query = await llm.generate(`Extract keywords from: ${event.content}`);
        const results = await searchTool.call({ query });
        
        // 4. Feedback
        agentOps.reply(`Search results: ${results}`);
      }
    });
  }
}
```

## 6. Summary: Hierarchy of the Ecosystem

This mechanism naturally forms a hierarchy within the ecosystem:

*   **Level 1 (Foundation):** Plugins providing atomic capabilities (`fs`, `http`, `gemini`).
*   **Level 2 (Composites):** Composite plugins combining L1 capabilities (`researcher`, `coder`, `writer`).
*   **Level 3 (Agents):** Complete digital species combining L2 Composites.

This allows developers to build complex applications like building blocks.
