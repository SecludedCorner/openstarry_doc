# 06. Plugin Directory Conventions

This document implements the core design philosophy of **"Directory as Protocol."** It defines how the OpenStarry system identifies, loads, and validates a plugin.

## 1. Physical Form of a Plugin

A standard OpenStarry plugin is an **NPM Package**. It can be published to an npm registry or exist directly as a folder within a `plugins/` directory.

### Standard Directory Layout

```text
my-awesome-plugin/
├── src/
│   ├── index.ts            # Entry file (must export the plugin class)
│   ├── tools/              # Tool logic (if any)
│   └── listeners/          # Listener logic (if any)
├── tests/                  # Unit tests
├── package.json            # NPM configuration (defines dependencies and metadata)
├── plugin.json             # [Optional] OpenStarry-specific metadata (or merged into package.json)
├── README.md               # Documentation
└── tsconfig.json           # TypeScript configuration
```

## 2. Plugin Entry Point

The system loader looks for the file pointed to by the `main` field in `package.json`. This file **must** export a class implementing the `IPlugin` interface as either a default export or a named export.

```typescript
// src/index.ts
import { IPlugin, IPluginContext } from '@openstarry/sdk';

export default class MyAwesomePlugin implements IPlugin {
  id = 'my-awesome-plugin';
  name = 'My Awesome Plugin';
  
  async initialize(context: IPluginContext) {
    // 1. Retrieve dependencies injected by the Core
    const apiKey = context.config.apiKey; // Configuration from agent.json
    const logger = context.logger;
    
    // 2. Retrieve cross-plugin services (if any)
    const mdParser = context.dependencies.markdownParser;

    logger.info('Initializing MyAwesomePlugin...');

    // 3. Register aggregate components
    context.registerTool(new MyTool(apiKey));
    context.registerListener(new MyListener());
  }
}
```

## 3. Metadata Specification (Manifest)

To enable the `Agent Design Service` to discover and manage plugins, we add an `openstarry` field to `package.json` or provide a standalone `plugin.json`.

**Recommended Method: Extending `package.json` directly**

```json
{
  "name": "@openstarry-plugin/weather",
  "version": "1.0.0",
  "main": "dist/index.js",
  "dependencies": {
    "@openstarry/sdk": "^0.1.0"
  },
  "openstarry": {
    "type": "tool", 
    "capabilities": ["weather-lookup", "forecast"],
    "permissions": {
      "network": ["api.weather.com"]
    }
  }
}
```

*   **type**: Plugin type (`tool` | `provider` | `listener` | `bundle`).
*   **capabilities**: Capability tags for the plugin (used for AI-based search).
*   **permissions**: Permissions requested by the plugin (the Core uses this to generate safety policies).

## 4. Plugin Categorization Directories

In the **Ecosystem Repo (`openstarry_plugin`)**, we adopt a **minimalist flat structure**, with all functional aggregate packages stored directly in the repository root:

```text
openstarry_plugin/
├── @openstarry-plugin/standard-function-fs/      # System tools
├── @openstarry-plugin/standard-function-stdio/   # Core interaction
├── @openstarry-plugin/provider-gemini-oauth/     # Perception adaptation
├── @openstarry-plugin/guide-mcp/                 # MCP protocol
├── @openstarry-plugin/standard-function-skill/   # Skill loading (Markdown parsing)
└── @openstarry-plugin/experimental-weather-tool/ # Experimental plugins
```

> **Naming Convention:** All official plugins are published using the `@openstarry-plugin/` namespace. When referenced in the `plugins` field of `agent.json`, the full package name (e.g., `@openstarry-plugin/standard-function-stdio`) must be used; short names are no longer supported.

This structure ensures the absolute purity of the core codebase (`openstarry`), with all concrete capabilities existing as external modules.
