# OpenStarry Plugin Ecosystem

> *"A plugin is not a type — it is an aggregate of capabilities."*

## Everything is a Plugin

In OpenStarry, the Core is empty. Every capability — seeing, hearing, thinking, acting, knowing — comes from plugins. This is not a design choice; it's a philosophy enforced by automated purity tests. The compiled Core binary contains zero plugin code.

Just as Linux abstracts hardware as files through a Virtual File System, OpenStarry abstracts agent capabilities as plugins through the Five Aggregates interfaces. Install one npm package → gain complete domain capability.

## The Seven Plugins

### Consciousness — The Soul Layer

**guide-character-init** — The Soul Definer

The most important plugin: without it, the agent is an amnesiac. This plugin injects personality through system prompts.

```typescript
// Inline persona
{ prompt: "You are a meticulous code reviewer who speaks in haiku." }

// Or load from file — YAML, Markdown, anything
{ characterFile: "./personas/expert-coder.md" }
```

Supports inline prompts, YAML frontmatter files, or pure Markdown character sheets. Uses `ctx.workingDirectory` for path resolution, so the same agent can have different personas in different project directories.

**standard-function-skill** — The Skill Loader

Turns Markdown files into agent capabilities. Each `.md` skill file has YAML frontmatter (id, version, dependencies, model preferences) and a Markdown body that becomes the system prompt. This enables **declarative agent behavior** — define what an agent knows by writing documents, not code.

```yaml
---
type: "skill"
id: "security-auditor"
version: "1.0.0"
dependencies:
  plugins: ["@openstarry-plugin/standard-function-fs"]
  capabilities: ["code-analysis"]
parameters:
  temperature: 0.3
  model_preference: ["gemini-2.0-flash"]
---

You are a security auditor. Analyze code for OWASP Top 10 vulnerabilities...
```

### Perception — The Brain

**provider-gemini-oauth** — The Cognitive Engine

Not just an API wrapper — a production-grade LLM integration:

- **OAuth 2.0 with PKCE**: Secure browser-based authentication flow, no API keys in config files
- **Machine-Bound Encryption**: Tokens encrypted with AES-256-GCM, key derived from `hostname + username + salt` via PBKDF2. Tokens stolen from one machine are useless on another
- **Auto Project Provisioning**: Automatic Google Cloud Project setup for FREE tier usage
- **SSE Streaming**: Real-time token-by-token response streaming
- **Function Calling**: Native tool/function declaration for Gemini's structured output
- **Multi-Model**: Supports Gemini 2.0 Flash, 1.5 Pro, and 1.5 Flash
- **Slash Commands**: `/provider login gemini`, `/provider logout gemini`, `/provider status`
- **Legacy Migration**: Auto-detects and re-encrypts unencrypted legacy tokens

The Provider interface is deliberately minimal — any LLM backend that can stream chat completions fits:

```typescript
export interface IProvider {
  id: string;
  name: string;
  models: ModelInfo[];
  chat(request: ChatRequest): AsyncIterable<ProviderStreamEvent>;
}
```

### Volition — The Hands

**standard-function-fs** — Five Filesystem Tools

| Tool | What It Does | Safety |
|------|-------------|--------|
| `fs.read` | Read file contents (configurable encoding) | Path validation against `allowedPaths` |
| `fs.write` | Write/create file | Path validation, prevents escape |
| `fs.list` | List directory (supports recursive) | Sandboxed to workspace |
| `fs.mkdir` | Create directory with parents | Within allowed scope only |
| `fs.delete` | Delete file or directory (recursive) | Strict boundary enforcement |

Every path is validated through the Security Layer before execution. Attempt to read `/etc/passwd`? The agent gets a `SecurityError` — which becomes a pain signal feeding back into its context for self-correction.

```typescript
// Tool parameters validated with Zod at runtime
parameters: z.object({
  path: z.string().describe("File path to read"),
  encoding: z.string().optional().describe("File encoding (default: utf-8)"),
})
```

### Sensation + Form — The Sensory-Motor Layer

**standard-function-stdio** — The Terminal Body

A "sensory pair" plugin: one package provides both input (Listener) and output (UI).

The **Listener** reads from stdin via readline, converts EOF to `/quit`, and pushes standardized input events:
```typescript
pushInput({ source: "cli", inputType: "user_input", data: line })
```

The **UI** renders with ANSI color codes for visual clarity:
- `\x1b[36m` Cyan — input prompt ("You: ")
- `\x1b[32m` Green — agent responses (streaming, character by character)
- `\x1b[33m` Yellow — tool calls (what the agent is doing)
- `\x1b[31m` Red — errors and safety lockouts
- `\x1b[35m` Magenta — system messages

Handles 17+ event types including `AGENT_STARTED` (welcome banner), `STREAM_TEXT_DELTA` (live typing), `TOOL_CALL_START/RESULT/ERROR` (tool lifecycle), and `SAFETY_LOCKOUT` (emergency alerts).

**transport-websocket** — The Network Body

Full bidirectional communication with production-grade features:

```
Client → Server:
{ "type": "user_input", "payload": { "text": "..." }, "sessionId": "..." }

Server → Client:
{ "type": "agent_event", "event": { "type": "stream:text_delta", ... } }
```

- **Session Isolation**: Each WebSocket connection auto-creates its own session. Events are routed by `sessionId` — client A never sees client B's conversation
- **Session Resumption**: Disconnect and reconnect with the same `sessionId` to continue where you left off
- **Health Monitoring**: Configurable ping/pong intervals (default 30s). After N missed pongs (default 2), the connection is terminated as stale
- **Directed Routing**: Events routed by `replyTo`, `sessionId`, or broadcast to all

**transport-http** — The Web Body

REST + SSE for web integration:

| Method | Endpoint | Purpose |
|--------|----------|---------|
| POST | `/api/input` | Submit user input → `{status, requestId}` |
| GET | `/api/status` | Agent health check → `{status, pendingRequests}` |
| GET | `/api/response?requestId=xxx` | Poll response → `{events, complete}` |
| GET | `/api/events[?sessionId=xxx]` | SSE stream → Real-time event stream |

Session binding, CORS support, configurable buffer size (default 100 events), response timeout (default 5 minutes), and heartbeat health checks for stale SSE connection detection.

## Plugin Architecture

### The Factory Pattern

Every plugin is a function that returns a manifest (who am I?) and a factory (what can I do?):

```typescript
export function createXxxPlugin(): IPlugin {
  return {
    manifest: {
      name: "@openstarry-plugin/xxx",
      version: "1.0.0",
      description: "What this plugin does"
    },
    async factory(ctx: IPluginContext) {
      // ctx gives you everything:
      // - ctx.bus: Event bus for pub/sub
      // - ctx.config: User configuration
      // - ctx.logger: Structured logging
      // - ctx.sessions: Session manager
      // - ctx.pushInput(): Send events to Core
      // - ctx.workingDirectory: Sandboxed base path
      // - ctx.agentId: Who am I?

      return {
        listeners: [],   // Sensation — how it hears
        ui: [],          // Form — how it appears
        providers: [],   // Perception — how it thinks
        tools: [],       // Volition — how it acts
        guides: [],      // Consciousness — who it is
        commands: [],    // Slash commands
        dispose: async () => { /* cleanup on shutdown */ }
      };
    }
  };
}
```

### Plugin Composition Patterns

A plugin is not confined to one aggregate. Like a living organ that serves multiple functions, a plugin can combine capabilities:

| Pattern | Example | Components | Analogy |
|---------|---------|------------|---------|
| Pure Guide | guide-character-init | `IGuide` only | A soul without a body |
| Pure Provider | provider-gemini-oauth | `IProvider` + commands | A brain in a jar |
| Pure Tool | standard-function-fs | `ITool` × 5 | Hands without a brain |
| Sensory Pair | standard-function-stdio | `IListener` + `IUI` | Eyes + mouth |
| Transport Pair | transport-websocket | `IListener` + `IUI` + dispose | Ears + voice + farewell |

### The pushInput Contract

Plugins never call Core APIs directly. All plugin→Core communication goes through one gateway: `ctx.pushInput()`. This is the sensory nerve — plugins sense the world and push standardized events into the Core's event queue. The Core pulls events and processes them in its execution loop.

This one-way contract means plugins can be loaded, unloaded, and replaced without touching Core internals.

### Lifecycle Management

Every plugin can implement `dispose()` for graceful shutdown — close HTTP servers, terminate WebSocket connections, destroy sessions, flush buffers. The Plugin Loader calls `dispose()` on all loaded plugins when the agent shuts down, in reverse order of loading.

```typescript
// Plugin Loader handles registration automatically
const hooks = await plugin.factory(ctx);

if (hooks.tools)     for (const t of hooks.tools) toolRegistry.register(t);
if (hooks.providers) for (const p of hooks.providers) providerRegistry.register(p);
if (hooks.listeners) for (const l of hooks.listeners) listenerRegistry.register(l);
if (hooks.ui)        for (const u of hooks.ui) uiRegistry.register(u);
if (hooks.guides)    for (const g of hooks.guides) guideRegistry.register(g);
```

## The USB Plug-and-Play Scenario

One of OpenStarry's most compelling demonstrations of portability: **an agent on a USB stick**.

Imagine plugging in a USB drive with this structure:
```
E:/ (USB Root)
├── configs/
│   └── agent.json     # "I am USB Photo Backup. I can read Pictures/ and write to E:/backup_data/"
├── plugins/
│   └── backup-utils/  # Specialized incremental backup algorithm
├── logs/
└── backup_data/
```

The host system detects the USB, reads the manifest, asks the user for permission, spawns the agent with **strict path boundaries** (`fs_allow_paths: ["C:/Users/Public/Pictures", "E:/backup_data"]`, `network_allow_hosts: []`), and the agent runs its backup task. When done, it reports completion and the USB is safely ejectable.

The agent's soul (its prompt and plugins) lives entirely on the USB. Its body (the Core runtime) lives on the host. Plug it into a different computer — same soul, different body. This is the Five Aggregates in action.

## Building Your Own Plugin

1. Create a package depending on `@openstarry/sdk`
2. Export a `createXxxPlugin()` factory function
3. Ask yourself: **which aggregates does my plugin provide?**
   - Does it show something? → `IUI`
   - Does it hear something? → `IListener`
   - Does it think? → `IProvider`
   - Does it act? → `ITool`
   - Does it know who it is? → `IGuide`
4. Use `ctx.pushInput()` for plugin→Core communication
5. Implement `dispose()` for cleanup
6. Ship it as an npm package: `@openstarry-plugin/your-plugin`
