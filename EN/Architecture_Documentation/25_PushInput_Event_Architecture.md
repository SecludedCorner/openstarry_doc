# 25. pushInput Event Architecture

This document explains the design decisions and operational mechanisms of `IPluginContext.pushInput`.

---

## 1. Problem Background

Before Plan02, the stdio plugin pushed user input through an `onInput` callback injected by the host:

```typescript
// Old Pattern (Plan01-02)
export function createStdioPlugin(opts: { onInput: (text: string) => void }): IPlugin {
  // opts.onInput is injected by the host (CLI) during construction
}
```

This introduced three problems:

| Issue | Impact |
|------|------|
| **Host must recognize the plugin** | The CLI needed to detect `ref.name === "standard-function-stdio"` and inject the callback. |
| **Special Handling** | The loading path for stdio was different from other plugins, breaking architectural uniformity. |
| **Lack of full dynamism** | Every time a new Listener requiring input capabilities was added, the host needed additional special handling. |

---

## 2. Solution: IPluginContext.pushInput

A standardized input pushing method was added to the SDK's `IPluginContext`:

```typescript
export interface IPluginContext {
  bus: EventBus;
  workingDirectory: string;
  agentId: string;
  config: Record<string, unknown>;
  /** Push an input event into the agent's processing queue. */
  pushInput: (event: InputEvent) => void;  // ← New
}
```

The Core automatically injects this when creating the plugin context:

```typescript
function getPluginContext(pluginConfig?: Record<string, unknown>): IPluginContext {
  return {
    bus,
    workingDirectory: process.cwd(),
    agentId: config.identity.id,
    config: pluginConfig ?? {},
    pushInput: (event) => core.pushInput(event),  // ← Bound to core
  };
}
```

---

## 3. Input Event Flow

```
User enters "hello" in the terminal
    ↓
stdio listener (readline "line" event)
    ↓
ctx.pushInput({ source: "cli", inputType: "user_input", data: "hello" })
    ↓
core.pushInput(inputEvent)
    ↓
┌─ Is it a slash command? ──→ handleSlashCommand() ──→ Fast Path (bypass LLM)
│
└─ Standard Input ──→ queue.push(INPUT_RECEIVED) ──→ ExecutionLoop.processEvent()
```

### InputEvent Structure

```typescript
export interface InputEvent {
  source: string;                          // Source identifier ("cli", "web", "api")
  inputType: string;                       // Type ("user_input", "system_event")
  data: string | Record<string, unknown>;  // Content
  replyTo?: string;                        // Reply target (optional)
}
```

---

## 4. Impact on Plugin Developers

### Before (Host cooperation required)

```typescript
// Plugin needs to receive callback in constructor
export function createMyListener(opts: { onInput: Function }): IPlugin { ... }

// Host requires special handling
const isMyPlugin = ref.name === "my-listener";
const opts = isMyPlugin ? { onInput: handleInput } : undefined;
```

### After (Standardized)

```typescript
// Plugin uses ctx.pushInput directly in factory
export function createMyListener(): IPlugin {
  return {
    manifest: { name: "my-listener", version: "1.0.0" },
    async factory(ctx: IPluginContext): Promise<PluginHooks> {
      const listener: IListener = {
        id: "my-listener",
        name: "My Custom Listener",
        onEvent(event) { /* handle output events */ },
        async start() {
          // Any input source can push
          someInputSource.on("data", (text) => {
            ctx.pushInput({
              source: "my-source",
              inputType: "user_input",
              data: text,
            });
          });
        },
      };
      return { listeners: [listener] };
    },
  };
}

// Host requires no special handling
```

---

## 5. Applicable Scenarios

`pushInput` is not limited to the CLI. Any plugin that needs to push input to the Agent can use it:

| Plugin | Input Source | source Value |
|------|---------|-----------|
| `standard-function-stdio` | Terminal readline | `"cli"` |
| Future: Web Listener | HTTP Request | `"web"` |
| Future: Discord Bot | Discord Message | `"discord"` |
| Future: File Watcher | File Change Event | `"file-watcher"` |
| Future: Cron Scheduler | Scheduled Trigger | `"scheduler"` |

---

## 6. Design Decisions

### Why not push directly via EventBus?

Since plugins already have `ctx.bus`, why not just `bus.emit(INPUT_RECEIVED, payload)`?

Because `pushInput` encapsulates additional logic:
1. **Slash Command Fast Path**: Commands like `/help` or `/quit` do not enter the EventQueue and are processed directly.
2. **Event Format Standardization**: Automatically wraps events as `{ type: INPUT_RECEIVED, payload: inputEvent }`.
3. **Future Extensibility**: Logic such as rate limiting, filtering, or authentication can be added within `pushInput`.

Directly using the EventBus would bypass these mechanisms.
