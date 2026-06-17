<!-- Status: CURRENT -->
<!-- Layer: 1-Engineering -->
<!-- Applies to: v0.59.4-alpha -->
<!-- Last verified: 2026-06-16 (re-stamped for v0.59.4; added --resume + agent-spawn; corrected init file list and plugin count. Originally rewritten 2026-06-11 against actual CLI + resolver source, replacing a v0.35-era guide with an unfollowable quickstart.) -->

> **Audience**: External developers, first-time plugin authors
> **Prerequisite**: Node.js basics, TypeScript familiarity

# Getting Started with OpenStarry

> OpenStarry is a high-discipline, plugin-native, local-first agent runtime.
> Build AI agents with structured control loops, plugin isolation, and auditable decisions.

---

## Prerequisites

- **Node.js** >= 20.0.0
- **pnpm** >= 9.0.0

## Install (from source — packages are NOT published to npm yet)

OpenStarry is a monorepo with a sibling plugin workspace. Clone both side by side:

```bash
git clone https://github.com/SecludedCorner/openstarry
git clone https://github.com/SecludedCorner/openstarry_plugin
cd openstarry
pnpm install
pnpm build        # builds the core monorepo AND the sibling plugin workspace
```

There is no global `openstarry` binary. The runner CLI is:

```bash
node apps/runner/dist/bin.js <command>
# tip: alias it —  alias openstarry="node /path/to/openstarry/apps/runner/dist/bin.js"
```

---

## Step 1: Create Your Agent Config

A working minimal config needs **four things** the validator/core will refuse to run without:

1. a **context manager** plugin (core throws without one — Tenet #7: even memory strategy is a plugin)
2. a **listener/UI** plugin so you can actually talk to it (`standard-function-stdio` = terminal REPL)
3. at least **one tool** in `capabilities.tools` (the validator rejects an empty tool list)
4. a **provider** plugin (the LLM brain)

The fastest zero-API-key path: if you have the `claude` CLI installed and authenticated,
use `provider-claude-cli` (drives `claude -p` as a subprocess; text-only).

Create `my-agent.json`:

```json
{
  "identity": {
    "id": "my-first-agent",
    "name": "My First Agent",
    "description": "A minimal OpenStarry agent",
    "version": "0.1.0"
  },
  "cognition": {
    "provider": "claude-cli",
    "model": "haiku",
    "temperature": 0.7,
    "maxTokens": 4096,
    "maxToolRounds": 5
  },
  "capabilities": {
    "tools": ["fs.read", "fs.list"],
    "allowedPaths": ["."]
  },
  "plugins": [
    { "name": "@openstarry-plugin/context-sliding-window" },
    { "name": "@openstarry-plugin/standard-function-stdio" },
    { "name": "@openstarry-plugin/standard-function-fs" },
    { "name": "@openstarry-plugin/provider-claude-cli" },
    { "name": "@openstarry-plugin/guide-character-init" }
  ],
  "guide": "default-guide"
}
```

Prefer a direct API instead? Swap the provider plugin + cognition block:
`provider-claude` (Anthropic API key), `provider-chatgpt` (OpenAI key),
`provider-gemini` (Google key), `provider-lmstudio` / `provider-local-llama` (local).
See `configs/` in the repo for eight ready-made examples (`configs/README.md`).

**Key fields:**

| Field | Purpose |
|-------|---------|
| `identity` | Agent metadata — id must be unique |
| `cognition` | LLM provider selection and parameters |
| `capabilities.tools` | Whitelist of tool IDs the agent may invoke (must be non-empty) |
| `capabilities.allowedPaths` | Filesystem sandbox boundaries |
| `plugins` | Plugins to load — see resolution order below |
| `guide` | System prompt provider (guide plugin id) |

---

## Step 2: Start the Agent

```bash
node apps/runner/dist/bin.js start --config ./my-agent.json
```

> The flag is **`--config`** (NOT `--agent` — older drafts of this guide were wrong).
> If a parent directory contains a `.openstarry/` project config you don't want
> applied, add `--no-project-dir`.

What happens on startup:

1. Runner reads `my-agent.json` (`--config`, else `./agent.json`, else the default path)
2. Plugins are resolved and loaded (see resolution order below)
3. Plugin factories receive `IPluginContext` and return their hooks
4. Core validates declared tools against `capabilities.tools` (runtime-enforced per plugin manifest)
5. Control loop begins — you get the REPL prompt

Type a message and press Enter. `/help` lists commands; `/quit` exits.

> **Heads-up on partial failures**: a plugin that fails to load does NOT abort
> startup (three-tier criticality model). If the provider failed to load you
> will still get a REPL — the error surfaces on your first message. Watch the
> `[plugin]` startup lines.

### Resuming a previous conversation (`--resume`)

```bash
node apps/runner/dist/bin.js start --config ./my-agent.json --resume
```

`--resume` restores the previous CLI session's conversation history. Non-empty
sessions are saved automatically at graceful shutdown (`/quit` or SIGINT) into the
same `FileSessionPersistence` store the daemon uses; `--resume` reloads the default
CLI session before the control loop starts. (Added v0.59.4-alpha; wired in
`apps/runner/src/commands/start.ts`, backed by `utils/cli-session-persistence.ts`;
listed in `--help` since v0.59.5-alpha.)

### Listing running agents (`ps`, `ps --tree`)

When agents run in background daemon mode (`daemon start`), `ps` lists them:

```bash
node apps/runner/dist/bin.js ps            # AGENT ID / PID / STATUS / UPTIME / LOG
node apps/runner/dist/bin.js ps --verbose  # per-agent detail (config / socket)
node apps/runner/dist/bin.js ps --tree     # parent → child process hierarchy
```

`--tree` (Doc 13) queries each daemon's `agent.processTree` RPC and renders the
spawn hierarchy indented by depth (`agentId (pid N) [status] depth=D`). Children
spawned via `agent.spawnChild` are folded under their parent so each agent prints
once. Read-only. (Added v0.59.7-alpha; `apps/runner/src/commands/ps.ts`.)

### Plugin Resolution Order (actual resolver behavior)

For each `plugins[]` entry the runner tries, in order:

1. **Explicit `path` field** (per-plugin): `{ "name": "x", "path": "../openstarry_plugin/x/dist/index.js" }`
   — path is validated against the project root when project config is active
2. **Package name** via Node module resolution (workspace / node_modules)
3. **System directory** scan: paths from `~/.openstarry/config.json` `pluginSearchPaths`,
   **plus (v0.58.0-alpha) the built-in monorepo sibling `../openstarry_plugin/`** —
   so in the documented two-repo layout, by-name loading works with zero configuration

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
- Tools use [zod](https://zod.dev/) schemas for parameter validation; `execute` returns a string
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

Compile your plugin (any tsc setup that emits ESM), then add it to the agent config:

```json
{
  "plugins": [
    { "name": "my-greet-plugin", "path": "./dist/my-tool-plugin.js" }
  ],
  "capabilities": {
    "tools": ["fs.read", "fs.list", "greet"]
  }
}
```

**Important:** Add `"greet"` to `capabilities.tools` — tools not listed there are blocked at runtime.

Restart the agent and ask: *"Greet Alice"* — the agent should invoke your `greet` tool.

> Note: `provider-claude-cli` is **text-only** (function calling deliberately
> disabled for isolation). To exercise tool calls end-to-end, use a direct API
> provider such as `provider-claude` or `provider-chatgpt`.

---

## Step 4.5: Project-Level Configuration (Optional)

For per-project agent customization, create a `.openstarry/` directory:

```bash
node apps/runner/dist/bin.js init --project
```

This generates three files in `.openstarry/`:

| File | Purpose |
|------|---------|
| `config.json` | Identity, cognition, memory overrides |
| `permissions.json` | Security restrictions (paths, tools, limits) |
| `README.md` | Generated notes describing the directory |

> **Note**: `init --project` does **not** generate `plugins.json` — an empty
> plugins array would fail `ProjectPluginsSchema.min(1)` validation, so absence
> means "no override" (`init.ts`). Create `plugins.json` manually only when you
> want to replace the agent's plugin list entirely.

Project config uses a **restrict-only model** — it can narrow capabilities but never expand them. Skip with `--no-project-dir`.

---

## Step 4.6: Advanced — Build a Custom Context Manager (Optional)

Context management is a plugin (Tenet #9). The REAL contract is small:

```typescript
import type {
  IPlugin,
  IPluginContext,
  PluginHooks,
  IContextManager,
  Message,
} from '@openstarry/sdk';

// 1. Implement IContextManager — one method: assemble the context window
const lastNContextManager: IContextManager = {
  assembleContext(messages: Message[], maxTurns: number): Message[] {
    // keep system messages + the last N conversational turns
    const system = messages.filter((m) => m.role === 'system');
    const rest = messages.filter((m) => m.role !== 'system');
    return [...system, ...rest.slice(-maxTurns)];
  },
};

// 2. Export the factory
export function createMyContextManagerPlugin(): IPlugin {
  return {
    manifest: {
      name: 'my-context-manager',
      version: '0.1.0',
      description: 'Custom context window strategy',
      skandha: 'samjna' as const,
    },
    async factory(_ctx: IPluginContext): Promise<PluginHooks> {
      return {
        contextManager: lastNContextManager,
      };
    },
  };
}

export default createMyContextManagerPlugin;
```

Replace `@openstarry-plugin/context-sliding-window` with yours in the config.
Reference implementations: `context-sliding-window` (turn window) and
`context-summary` in the plugin workspace.

---

## Step 4.7: Observability (Optional, v0.58.0-alpha)

Opt-in, env-driven — zero behavior change when unset:

| Env | Effect |
|-----|--------|
| `OPENSTARRY_LOG_PATH=./runner.jsonl` | JSONL lifecycle records (runner:started / plugin:loaded / runner:shutdown); level via `LOG_LEVEL` |
| `OPENSTARRY_AUDIT=1` | Journals `capability_denied` tool-filter events to `~/.openstarry/audit-trail.jsonl` (dedup + buffered) |
| `OPENSTARRY_WORKFLOW_STATE_DIR=./wf-state` | workflow-engine persists every execution result JSON; status survives the process |

---

## Step 5: Next Steps

| Goal | Resource |
|------|----------|
| Build more plugins | [Plugin Philosophy: Five Aggregates](./Plugin_System_Architecture/00_Plugin_Philosophy_Five_Aggregates.md) |
| Working example configs | `configs/README.md` in the openstarry repo (8 examples) |
| Advanced plugin examples | [Implementation Examples](./Implementation_Examples/) |
| Understand the system | [Ten Tenets (README §十大核心宣言)](./README.md#-十大核心宣言-the-ten-tenets) |
| Authoritative API contracts | `packages/sdk/src/` type files — **the SDK types are the spec**; where any doc disagrees with them, the SDK wins |

### Pre-built Plugins (selection — 46 loadable plugins in the plugin workspace, +1 `mcp-common` shared types lib = 47 packages)

| Plugin | Type | Description |
|--------|------|-------------|
| `@openstarry-plugin/provider-claude-cli` | Model | Drives the `claude` CLI as backend (zero API key; text-only) |
| `@openstarry-plugin/provider-claude` | Model | Direct Anthropic Messages API (streaming + tool use) |
| `@openstarry-plugin/provider-chatgpt` | Model | Direct OpenAI Chat Completions (streaming + tool calls) |
| `@openstarry-plugin/provider-gemini` / `-oauth` | Model | Google Gemini (API key / OAuth) |
| `@openstarry-plugin/provider-lmstudio` / `provider-local-llama` | Model | Local servers (LM Studio / Ollama) |
| `@openstarry-plugin/standard-function-fs` | Tool | File read/write/list/mkdir/delete (path-jailed) |
| `@openstarry-plugin/standard-function-stdio` | I/O | Terminal REPL (listener + UI) |
| `@openstarry-plugin/context-sliding-window` | Memory | Turn-window context manager (mandatory category) |
| `@openstarry-plugin/mcp-client` | Bridge | Connect external MCP servers (stdio + HTTP/OAuth) |
| `@openstarry-plugin/mcp-server` | Bridge | Expose the agent AS an MCP server |
| `@openstarry-plugin/workflow-engine` | Tool | Declarative YAML workflows (loop steps + persistence since v0.58) |
| `@openstarry-plugin/agent-spawn` | Tool | Lets an agent spawn child agents at runtime via the daemon's `agent.spawnChild` tool (Tenet #10; daemon mode required) |
| `@openstarry-plugin/web-ui` / `tui-dashboard` | UI | Browser / terminal dashboards |

> There is **no built-in shell-exec tool** — by design. Wire one via an MCP
> server or write a tool plugin with an explicit allowlist.

---

*OpenStarry Getting Started Guide — Layer 1 Engineering*
*Rewritten 2026-06-11 (v0.58.0-alpha repair sprint) — every command and config in this guide was executed against the real runner before publishing.*
