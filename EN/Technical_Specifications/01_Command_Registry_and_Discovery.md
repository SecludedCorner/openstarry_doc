# Technical Specification 01: Command Registration, Discovery, and Dependency Injection

> **Status**: Draft
> **Applicable Version**: OpenStarry v0.1.0+
> **Related Architecture Documents**: 08_Command_And_Tool_Design, 19_Agent_Coordination_Layer

This specification precisely defines the registration workflow for Plugin commands (`/command`), the file structure, and the implementation details of dependency injection during Agent startup, in order to eliminate discrepancies between architectural design and code implementation.

## 1. File System Structure (The System Folder)

The "system folder" and "Registry" maintained by the Coordination Layer (Coordinator) must adhere to the following physical structure:

```text
~/.openstarry/
├── config.toml                 # Daemon main configuration
├── registry.json               # [Auto-generated] Global command and plugin index file (The Source of Truth)
└── plugins/                    # Plugin installation directory
    ├── @openstarry-plugin/provider-gemini/
    │   ├── package.json
    │   ├── plugin.json         # [Critical] Defines which commands this plugin provides
    │   └── dist/index.js
    └── @openstarry-plugin/provider-claude/
        ├── ...
```

### 1.1 Registry Index File (`registry.json`)
This is the cache file used by the CLI and the Coordination Layer to query "which commands are available." When a Plugin changes, the Coordination Layer updates this file.

```json
{
  "last_updated": "2026-02-04T12:00:00Z",
  "commands": {
    "provider": {
      "description": "Manage AI Providers",
      "subcommands": {
        "gemini": {
          "plugin_id": "@openstarry-plugin/provider-gemini",
          "handler": "handleCliCommand",
          "subcommands": {
             "login": { "description": "Authenticate Gemini" }
          }
        },
        "claude": {
          "plugin_id": "@openstarry-plugin/provider-claude",
          "handler": "handleCliCommand",
          "subcommands": {
             "login": { "description": "Authenticate Claude" }
          }
        }
      }
    }
  },
  "capabilities": {
    "llm:gemini": "@openstarry-plugin/provider-gemini",
    "llm:claude": "@openstarry-plugin/provider-claude"
  }
}
```

---

## 2. Plugin Definition Specification (`plugin.json`)

Plugin developers must explicitly declare the CLI commands they provide in `plugin.json`. This fulfills the requirement for "dynamic Plugin registration."

**Example: Gemini Provider's `plugin.json`**

```json
{
  "id": "@openstarry-plugin/provider-gemini",
  "version": "1.0.0",
  "type": "provider",
  "capabilities": ["llm:gemini"],
  "cli_commands": [
    {
      "trigger": "provider gemini",
      "description": "Gemini Provider Management",
      "method": "handleCliCommand",
      "help_text": "Use 'login' to authenticate or 'config' to set options."
    }
  ]
}
```

*   **trigger**: Defines the command path, where spaces represent subcommand hierarchy levels.
*   **method**: Corresponds to the specific function name in the Plugin entry point class.

---

## 3. Implementation Workflow: Dynamic Scanning and Registration

The Coordination Layer (`Coordinator`) must implement the following logic to achieve "updating the system folder's command-related files":

### 3.1 Pseudocode Logic (Coordinator Daemon)

```typescript
class PluginRegistry {
  private registryPath = "~/.openstarry/registry.json";

  // 1. Scan the plugin directory
  async refreshRegistry() {
    const plugins = await scanPluginDir("~/.openstarry/plugins");
    const newRegistry = { commands: {}, capabilities: {} };

    for (const plugin of plugins) {
      const manifest = await readJson(plugin.path + "/plugin.json");

      // 2. Register CLI commands
      if (manifest.cli_commands) {
        for (const cmd of manifest.cli_commands) {
          // Parse "provider gemini" into a nested object structure
          this.mergeCommandTree(newRegistry.commands, cmd.trigger, {
            plugin_id: manifest.id,
            handler: cmd.method,
            description: cmd.description
          });
        }
      }

      // 3. Register capabilities (used for Agent injection)
      if (manifest.capabilities) {
        for (const cap of manifest.capabilities) {
          newRegistry.capabilities[cap] = manifest.id;
        }
      }
    }

    // 4. Update the system file
    await writeJson(this.registryPath, newRegistry);
  }
}
```

---

## 4. Agent Spawning and Dependency Injection

This section addresses the requirement to "configure different Providers for different Agents."

### 4.1 Request Structure
When the CLI or API requests the creation of an Agent:

```json
// POST /agents/spawn
{
  "name": "my-coding-bot",
  "config": {
    "provider": "gemini",  // Specifies using gemini
    "model": "gemini-1.5-pro",
    "tools": ["fs", "terminal"]
  }
}
```

### 4.2 Coordination Layer Processing Logic (Resolution)

Upon receiving a request, the Coordination Layer must perform "Resolution":

1.  Read the `config.provider` value as `"gemini"`.
2.  Query `registry.json` or the in-memory index to find the Plugin ID that provides the `llm:gemini` capability -> `@openstarry-plugin/provider-gemini`.
3.  Load the Plugin.
4.  Instantiate the Agent Runtime and inject the Provider instance.

```typescript
// Coordinator Code
async function spawnAgent(request) {
  // 1. Resolve the Provider Plugin
  const providerPluginId = registry.capabilities[`llm:${request.config.provider}`];
  if (!providerPluginId) throw new Error("Unknown provider");

  const plugin = await loadPlugin(providerPluginId);

  // 2. Obtain the Provider instance (Factory Pattern)
  const providerInstance = await plugin.createProvider({
    model: request.config.model
    // This automatically reads the token previously saved via /provider gemini login
  });

  // 3. Inject into the Agent
  const agent = new AgentCore({
    name: request.name,
    provider: providerInstance, // <--- Injection point
    tools: request.config.tools
  });

  await agent.start();
}
```

## 5. Summary: Developer Implementation Checklist

To comply with this specification, you need to implement:

1.  **Registry Manager**: Implement the logic to scan `plugins/` and generate `registry.json` in `packages/core` or `daemon`.
2.  **CLI Router**: Upon startup, the CLI (`apps/runner`) reads `registry.json` and dynamically registers `commander` commands. When a user enters `/provider gemini login`, the CLI calls the Daemon via IPC, and the Daemon forwards the request to the Plugin's `handleCliCommand`.
3.  **Plugin Interface**: Ensure all Provider Plugins implement the standard interface (e.g., `IProviderPlugin`).
