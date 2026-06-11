<!-- Status: CURRENT -->
<!-- Layer: 1-Engineering -->
<!-- Applies to: v0.35.0-alpha -->
<!-- Last verified: 2026-03-20 -->

> **Audience**: External developers, first-time plugin authors
> **Prerequisite**: Node.js basics, TypeScript familiarity

# Getting Started with OpenStarry

> OpenStarry is a high-discipline, plugin-native, local-first agent runtime.
> Build AI agents with structured control loops, plugin isolation, and auditable decisions.

---

## Prerequisites

- **Node.js** >= 20.0.0
- **pnpm** >= 9.0.0
- **TypeScript** >= 5.5.0

## Quick Install

```bash
pnpm add @openstarry/sdk @openstarry/runner
```

---

## Step 1: Create Your Agent Config (~15 lines)

Create a file `my-agent.json`:

```json
{
  "identity": {
    "id": "my-first-agent",
    "name": "My First Agent",
    "description": "A minimal OpenStarry agent",
    "version": "0.1.0"
  },
  "cognition": {
    "provider": "chatgpt",
    "model": "gpt-4o-mini",
    "temperature": 0.7,
    "maxTokens": 4096,
    "maxToolRounds": 5
  },
  "capabilities": {
    "tools": ["fs.read", "fs.list"],
    "allowedPaths": ["."]
  },
  "plugins": [
    { "name": "@openstarry-plugin/provider-chatgpt" },
    { "name": "@openstarry-plugin/standard-function-fs" }
  ],
  "guide": "default-guide"
}
```

**Key fields:**

| Field | Purpose |
|-------|---------|
| `identity` | Agent metadata — id must be unique |
| `cognition` | LLM provider selection and parameters |
| `capabilities.tools` | Whitelist of tool IDs the agent may invoke |
| `capabilities.allowedPaths` | Filesystem sandbox boundaries |
| `plugins` | Plugins to load (resolved by package name or file path) |
| `guide` | System prompt provider (guide plugin id) |

---

## Step 2: Start the Agent

```bash
openstarry start --agent ./my-agent.json
```

What happens on startup:

1. Runner reads `my-agent.json`
2. Plugins are resolved and loaded (package name → node_modules, or file path)
3. Plugin factories receive `IPluginContext` and return their hooks
4. Core validates all declared tools against `capabilities.tools`
5. Control loop begins — the agent is ready for input

You should see the interactive REPL. Type a message and press Enter.

---

## Step 3: Write Your First Plugin (~40 lines)

Create `my-tool-plugin.ts`:

```typescript
import type {
  IPlugin,
  IPluginContext,
  PluginHooks,
  ITool,
} from '@openstarry/sdk';
import { z } from 'zod';

// 1. Define a tool
const greetTool: ITool<{ name: string }> = {
  skandha: 'samskara' as const,   // code category — action/tool type
  id: 'greet',
  description: 'Returns a greeting message',
  parameters: z.object({
    name: z.string().describe('Name to greet'),
  }),
  async execute(input) {
    return `Hello, ${input.name}! Welcome to OpenStarry.`;
  },
};

// 2. Export a factory function
export function createGreetPlugin(): IPlugin {
  return {
    manifest: {
      name: 'my-greet-plugin',
      version: '0.1.0',
      description: 'A simple greeting tool',
      skandha: 'samskara' as const,
    },
    async factory(_ctx: IPluginContext): Promise<PluginHooks> {
      return {
        tools: [greetTool],
      };
    },
  };
}

export default createGreetPlugin;
```

**Pattern summary:**

- Every plugin exports a **factory function** that returns `IPlugin`
- `IPlugin.manifest` describes the plugin metadata
- `IPlugin.factory(ctx)` receives context, returns **hooks** (tools, providers, listeners, etc.)
- Tools use [zod](https://zod.dev/) schemas for parameter validation
- The `skandha` field is a **code category value** — it tells the runtime which aggregate this plugin belongs to

### Plugin Categories (code values)

| Engineering Term | Code Value | What It Provides |
|-----------------|------------|-----------------|
| **Input/Sensor** | `'rupa'` | `listeners`, `ui` — receives external signals |
| **Feedback/Sensing** | `'vedana'` | `vedanaSensors` — evaluates interaction quality |
| **Model/Cognition** | `'samjna'` | `providers` — LLM backends |
| **Action/Tool** | `'samskara'` | `tools` — executable actions |
| **Control/Governance** | `'vijnana'` | `guides`, `volition`, `auditor` — routes decisions |

---

## Step 4: Register and Test

Add your plugin to the agent config:

```json
{
  "plugins": [
    { "name": "@openstarry-plugin/provider-chatgpt" },
    { "name": "@openstarry-plugin/standard-function-fs" },
    { "name": "./my-tool-plugin.js" }
  ],
  "capabilities": {
    "tools": ["fs.read", "fs.list", "greet"]
  }
}
```

**Important:** Add `"greet"` to `capabilities.tools` — tools not listed there will be blocked at runtime.

Restart the agent:

```bash
openstarry start --agent ./my-agent.json
```

Test by asking: *"Greet Alice"* — the agent should invoke your `greet` tool.

---

## Step 4.5: Project-Level Configuration (Optional)

For per-project agent customization, create a `.openstarry/` directory:

```bash
openstarry init --project
```

This generates three config files in `.openstarry/`:

| File | Purpose |
|------|---------|
| `config.json` | Identity, cognition, memory overrides |
| `permissions.json` | Security restrictions (paths, tools, limits) |
| `plugins.json` | Plugin list override |

Project config uses a **restrict-only model** — it can narrow capabilities but never expand them. Skip with `--no-project-dir`.

---

## Step 4.6: Advanced — Build a Custom Context Manager (Optional)

For agents that need custom memory strategies, implement a custom `IContextManager` plugin:

```typescript
import type {
  IPlugin,
  IPluginContext,
  PluginHooks,
  IContextManager,
} from '@openstarry/sdk';

// 1. Implement IContextManager interface
const customContextManager: IContextManager = {
  skandha: 'samjna' as const,
  async summarize(messages) {
    // Custom summarization logic
    return `Summary of ${messages.length} messages: [custom logic here]`;
  },
  async filter(messages) {
    // Custom filtering logic
    return messages.filter((m) => m.role !== 'system');
  },
  async extend(messages, input) {
    // Custom context extension logic
    return [...messages, { role: 'user' as const, content: input }];
  },
};

// 2. Export the factory
export function createMyContextManagerPlugin(): IPlugin {
  return {
    manifest: {
      name: 'my-context-manager',
      version: '0.1.0',
      description: 'Custom context management strategy',
      skandha: 'samjna' as const,
      criticality: 'optional-degraded',
    },
    async factory(_ctx: IPluginContext): Promise<PluginHooks> {
      return {
        contextManager: customContextManager,
      };
    },
  };
}

export default createMyContextManagerPlugin;
```

**Important**: The `skandha: 'samjna'` (cognition) tells the runtime this plugin provides a cognitive processing hook — context managers belong to the perception/cognition aggregate.

Add it to your config:

```json
{
  "plugins": [
    { "name": "./my-context-manager.js" }
  ]
}
```

Multiple context managers can coexist. The first one found is used (resolution order: config priority list → default sliding-window fallback).

---

## Step 5: Next Steps

| Goal | Resource |
|------|----------|
| Build more plugins | [Plugin Philosophy: Five Aggregates](./Plugin_System_Architecture/00_Plugin_Philosophy_Five_Aggregates.md) |
| Custom context strategies | [Context Strategy: Sliding Window](./Implementation_Examples/Context_Strategy_SlidingWindow.md) |
| Advanced plugin examples | [Implementation Examples](./Implementation_Examples/) |
| See available plugins | [Architecture Overview](./Architecture_Documentation/01_Architecture_Overview.md) |
| Understand the system | [Ten Tenets (README §十大核心宣言)](./README.md#-十大核心宣言-the-ten-tenets) |

### Pre-built Plugins

OpenStarry ships with ready-to-use plugins:

| Plugin | Type | Description |
|--------|------|-------------|
| `@openstarry-plugin/provider-chatgpt` | Model | OpenAI GPT models |
| `@openstarry-plugin/provider-gemini-oauth` | Model | Google Gemini models |
| `@openstarry-plugin/standard-function-fs` | Tool | File read/write/list/mkdir/delete |
| `@openstarry-plugin/standard-function-shell` | Tool | Shell command execution |
| `@openstarry-plugin/transport-cli` | Input | Interactive CLI (REPL) |
| `@openstarry-plugin/web-ui` | Input | Browser-based interface |

### Plugin Resolution Order

When the runner loads a plugin by name, it searches in this order:

1. **Absolute path**: `/path/to/plugin.js`
2. **Relative path**: `./plugins/my-plugin.js` (relative to agent config)
3. **System directory**: `~/.openstarry/plugins/installed/`
4. **Package name**: `@openstarry-plugin/my-plugin` (Node.js module resolution)

---

*OpenStarry Getting Started Guide — Layer 1 Engineering*
*Cycle 02-10 Track A delivery (QW-3) | Cycle 02-11 Enhancement (PROC-DOC-4)*
*2026-03-20*
