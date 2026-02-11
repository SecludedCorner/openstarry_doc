# OpenStarry Plugin Ecosystem

> *"A plugin is not a type; it is an aggregation of capabilities."*

## Everything is a Plugin

In OpenStarry, the Core is empty. Every capability—seeing, hearing, thinking, acting, and knowing—comes from plugins. This is not a design choice, but a philosophy enforced by automated purity tests. The compiled Core binary contains zero plugin code.

Just as Linux abstracts hardware into files via the Virtual File System (VFS), OpenStarry abstracts Agent capabilities into plugins via the Five Aggregates interfaces. Install an npm package → acquire full domain capabilities.

## Seven Plugins

### Consciousness (識蘊) — The Soul Layer

**guide-character-init** — The Soul Definer

The most critical plugin: without it, the Agent is an amnesiac. This plugin injects personality via the System Prompt.

```typescript
// Inline persona
{ prompt: "You are a meticulous code reviewer who speaks in haiku." }

// Or load from file — YAML, Markdown, any format
{ characterFile: "./personas/expert-coder.md" }
```

Supports inline prompts, YAML Frontmatter files, or pure Markdown role settings. It uses `ctx.workingDirectory` for path resolution, so the same Agent can possess different personas in different project folders.

**standard-function-skill** — The Skill Loader

Transforms Markdown files into Agent capabilities. Each `.md` skill file has YAML Frontmatter (id, version, dependencies, model preferences) and a Markdown body that becomes the System Prompt. This enables **declarative Agent behavior**—defining Agent knowledge by writing documentation rather than code.

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

### Perception (想蘊) — The Brain

**provider-gemini-oauth** — The Cognitive Engine

Not just an API wrapper, but a production-grade LLM integration:

- **OAuth 2.0 + PKCE**: Secure browser-based authentication; no API Keys needed in configuration files.
- **Machine-bound Encryption**: Tokens are encrypted with AES-256-GCM, with keys derived via PBKDF2 from `hostname + username + salt`. A token stolen from one machine won't work on another.
- **Auto Project Provisioning**: Automatically creates a Google Cloud Project to utilize the free tier.
- **SSE Streaming**: Real-time token-by-token response streaming.
- **Function Calling**: Native tool/function declarations interfacing with Gemini's structured output.
- **Multi-model Support**: Supports Gemini 2.0 Flash, 1.5 Pro, and 1.5 Flash.
- **Slash Commands**: `/provider login gemini`, `/provider logout gemini`, `/provider status`.
- **Legacy Migration**: Automatically detects and re-encrypts unencrypted legacy tokens.

The Provider interface is intentionally kept lean—any LLM backend capable of streaming chat completions can be adapted:

```typescript
export interface IProvider {
  id: string;
  name: string;
  models: ModelInfo[];
  chat(request: ChatRequest): AsyncIterable<ProviderStreamEvent>;
}
```

### Volition (行蘊) — The Hands

**standard-function-fs** — Five File System Tools

| Tool | Function | Safety |
|------|------|--------|
| `fs.read` | Read file content (configurable encoding) | Path validation via `allowedPaths` |
| `fs.write` | Write/create files | Path validation to prevent escape |
| `fs.list` | List directories (recursive support) | Restricted to workspace scope |
| `fs.mkdir` | Create directories (including parents) | Only within permitted boundaries |
| `fs.delete` | Delete files or directories (recursive) | Strict boundary enforcement |

Every path is validated by the security layer before execution. Trying to read `/etc/passwd`? The Agent receives a `SecurityError`—which becomes a pain signal fed back into its context for self-correction.

```typescript
// Tool parameters are validated at runtime via Zod
parameters: z.object({
  path: z.string().describe("File path to read"),
  encoding: z.string().optional().describe("File encoding (default: utf-8)"),
})
```

### Sensation + Form (受蘊 + 色蘊) — The Sensorimotor Layer

**standard-function-stdio** — The Terminal Body

A "sensory pairing" plugin: one package provides both input (Listener) and output (UI).

**Listener** reads stdin via readline, transforms EOF into `/quit`, and pushes standardized input events:
```typescript
pushInput({ source: "cli", inputType: "user_input", data: line })
```

**UI** uses ANSI color codes for visual presentation:
- `\x1b[36m` Cyan — Input prompt ("You: ")
- `\x1b[32m` Green — Agent response (streaming, character-by-character)
- `\x1b[33m` Yellow — Tool calls (what the Agent is doing)
- `\x1b[31m` Red — Errors and safety lockouts
- `\x1b[35m` Magenta — System messages

Handles over 17 event types, including `AGENT_STARTED` (welcome message), `STREAM_TEXT_DELTA` (real-time typing effect), `TOOL_CALL_START/RESULT/ERROR` (tool lifecycle), and `SAFETY_LOCKOUT` (emergency alerts).

**transport-websocket** — The Network Body

Full-duplex communication with production-grade features:

```
Client → Server:
{ "type": "user_input", "payload": { "text": "..." }, "sessionId": "..." }

Server → Client:
{ "type": "agent_event", "event": { "type": "stream:text_delta", ... } }
```

- **Session Isolation**: Each WebSocket connection automatically establishes an independent Session. Events are routed by `sessionId`—User A never sees User B's conversation.
- **Session Recovery**: Reconnect with the same `sessionId` after disconnection to resume where you left off.
- **Health Monitoring**: Configurable ping/pong intervals (default 30s). After N consecutive failed pongs (default 2), the connection is deemed stale and terminated.
- **Targeted Routing**: Events are routed by `replyTo`, `sessionId`, or broadcast to all connections.

**transport-http** — The Web Body

REST + SSE for Web integration:

| Method | Endpoint | Purpose |
|------|------|------|
| POST | `/api/input` | Submit user input → `{status, requestId}` |
| GET | `/api/status` | Agent health check → `{status, pendingRequests}` |
| GET | `/api/response?requestId=xxx` | Poll for responses → `{events, complete}` |
| GET | `/api/events[?sessionId=xxx]` | SSE Stream → Real-time event flow |

Supports Session binding, CORS support, configurable buffer sizes (default 100 events), response timeouts (default 5 mins), and heartbeat health checks for detecting stale SSE connections.

## Plugin Architecture

### Factory Pattern

Each plugin is a function returning a manifest (Who am I?) and a factory (What can I do?):

```typescript
export function createXxxPlugin(): IPlugin {
  return {
    manifest: {
      name: "@openstarry-plugin/xxx",
      version: "1.0.0",
      description: "What this plugin does"
    },
    async factory(ctx: IPluginContext) {
      // ctx provides everything you need:
      // - ctx.bus: Event Bus for pub/sub
      // - ctx.config: User settings
      // - ctx.logger: Structured logging
      // - ctx.sessions: Session manager
      // - ctx.pushInput(): Send events to Core
      // - ctx.workingDirectory: Sandboxed base path
      // - ctx.agentId: Who am I?

      return {
        listeners: [],   // Sensation — How to hear
        ui: [],          // Form — How to present
        providers: [],   // Perception — How to think
        tools: [],       // Volition — How to act
        guides: [],      // Consciousness — Who am I
        commands: [],    // Slash commands
        dispose: async () => { /* Cleanup on shutdown */ }
      };
    }
  };
}
```

### Plugin Composition Patterns

A plugin is not limited to a single aggregate. Just as a living organ can serve multiple functions, plugins can combine different capabilities:

| Pattern | Example | Composition | Analogy |
|------|------|------|------|
| Pure Guide | guide-character-init | `IGuide` only | Soul without a body |
| Pure Provider | provider-gemini-oauth | `IProvider` + commands | Brain in a vat |
| Pure Tool | standard-function-fs | `ITool` × 5 | Hands without a brain |
| Sensory Pairing | standard-function-stdio | `IListener` + `IUI` | Eyes + Mouth |
| Transport Pairing | transport-websocket | `IListener` + `IUI` + dispose | Ears + Voice + Farewell |

### pushInput Contract

Plugins never call Core APIs directly. All plugin-to-Core communication passes through a single gateway: `ctx.pushInput()`. This is the sensory nerve—the plugin perceives the world and pushes standardized events into the Core's event queue. The Core retrieves events from the queue and processes them in its execution loop.

This unidirectional contract means plugins can be loaded, unloaded, and replaced without ever touching the Core's internals.

### Lifecycle Management

Every plugin can implement `dispose()` for graceful shutdown—closing HTTP servers, terminating WebSocket connections, destroying Sessions, clearing buffers. The Plugin Loader calls `dispose()` on all loaded plugins in the reverse order of loading when the Agent shuts down.

```typescript
// Plugin Loader handles registration automatically
const hooks = await plugin.factory(ctx);

if (hooks.tools)     for (const t of hooks.tools) toolRegistry.register(t);
if (hooks.providers) for (const p of hooks.providers) providerRegistry.register(p);
if (hooks.listeners) for (const l of hooks.listeners) listenerRegistry.register(l);
if (hooks.ui)        for (const u of hooks.ui) uiRegistry.register(u);
if (hooks.guides)    for (const g of hooks.guides) guideRegistry.register(g);
```

## USB Plug-and-Play Scenario

One of OpenStarry's most compelling portability demonstrations: **Agent on a USB stick**.

Imagine inserting a USB stick with this structure:
```
E:/ (USB Root)
├── configs/
│   └── agent.json     # "I am a USB Photo Backup assistant. I read Pictures/ and write to E:/backup_data/"
├── plugins/
│   └── backup-utils/  # Specialized incremental backup algorithm
├── logs/
└── backup_data/
```

The host system detects the USB, reads the manifest, asks for user authorization, and spawns the Agent with **strict path boundaries** (`fs_allow_paths: ["C:/Users/Public/Pictures", "E:/backup_data"]`, `network_allow_hosts: []`). The Agent then executes the backup task. Upon completion, it reports results, and the USB can be safely ejected.

The Agent's soul (its Prompt and plugins) resides entirely on the USB. Its body (Core execution environment) resides on the host. Plug it into another computer—same soul, different body. This is the Five Aggregates in practice.

## Building Your Own Plugin

1. Create a package dependent on `@openstarry/sdk`.
2. Export a `createXxxPlugin()` factory function.
3. Ask yourself: **Which aggregates does my plugin provide?**
   - Does it show something? → `IUI`
   - Does it hear something? → `IListener`
   - Does it think? → `IProvider`
   - Does it perform actions? → `ITool`
   - Does it know who it is? → `IGuide`
4. Use `ctx.pushInput()` for plugin-to-Core communication.
5. Implement `dispose()` for resource cleanup.
6. Publish as an npm package: `@openstarry-plugin/your-plugin`.
