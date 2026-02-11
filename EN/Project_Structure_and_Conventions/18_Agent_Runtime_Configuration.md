# 18. Agent Runtime Configuration

This document supplements `05_Agent_Manifest_Specification.md`, focusing on how the **Runtime** parses, validates, and applies the configuration in `agent.json`, with particular emphasis on plugin activation and parameter injection.

## 1. Configuration Loading Flow

When `openstarry start` or the `Daemon` launches an Agent:

1.  **Read:** Load the raw JSON from `configs/agent.json`.
2.  **Merge (Merger):**
    *   If `agent.json` defines `extends: "generic-assistant-v1"`, the system first loads the template's configuration, then overlays the current project's configuration on top (Deep Merge).
    *   This allows projects to define only the differences (e.g., modifying only the Prompt or adding a single Tool).
3.  **Environment Variable Injection:** Parse `${ENV_VAR}` syntax and inject sensitive information such as API Keys.

## 2. Plugin Activation and Parameter Injection

This is the most critical part of the configuration: how the Agent becomes aware of its available capabilities.

### Configuration Structure

```json
"capabilities": {
  "plugins": {
    "@openstarry-plugin/fs": {
      "enabled": true,
      "config": { "root": "./workspace" } // Injected into initialize(context)
    },
    "@openstarry-plugin/weather": {
      "enabled": false // Explicitly disabled
    }
  }
}
```

### Runtime Logic

1.  **Filter:** The Loader iterates through the `plugins` list, filtering out entries with `enabled: false`.
2.  **Context Construction:**
    *   For each enabled plugin (e.g., `@openstarry-plugin/fs`), the Loader extracts its `config` object (e.g., `{ "root": "./workspace" }`).
    *   This object is placed into `IPluginContext.config`.
3.  **Initialization:** The plugin's `initialize(context)` method is called.
    *   Internally, the plugin retrieves parameters via `context.config.root` and uses them to configure path restrictions for the `fs` tool.

## 3. Five Aggregates Configuration Mapping

The `cognition` and `identity` sections of `agent.json` directly configure **Guide (Consciousness)** type plugins.

*   **System Prompt:** If `agent.json` defines a `system_prompt`, the Core wraps it as a temporary `AnonymousGuidePlugin` with the highest priority.
*   **Provider:** The `cognition.provider` configuration is passed directly to the corresponding Provider plugin (e.g., `gemini`).

## 4. Error Handling and Default Behavior

*   **Zero Plugin State:**
    *   If the system detects no plugins at startup (or `~/.openstarry/plugins` is empty), the Agent Core **must not crash**.
    *   The system should enter **"Idle Mode"** and display a prominent notice in the terminal or Dashboard:
        > "⚠️ No plugins found. Please run `openstarry plugin sync` to install standard capabilities."
*   **Missing Dependencies:** If the configuration enables `plugin-x` but the system cannot locate it, the Agent should fail to start with an error (Fail Fast), unless the plugin is marked as `optional: true`.
*   **Invalid Configuration:** Plugins should validate the incoming `config` during the `initialize` phase and throw a `ConfigurationError` if it is invalid.
