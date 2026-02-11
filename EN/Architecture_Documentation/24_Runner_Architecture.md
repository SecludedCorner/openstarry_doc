# 24. Runner Architecture

This document explains the design philosophy and responsibility boundaries of `apps/runner`.

---

## 1. What is the Runner?

The Runner is the **Host Bootstrap Program** for OpenStarry. It is the entry point for an Agent but does not contain any business logic itself.

```
apps/runner/src/bin.ts
```

The Runner's responsibility consists of only four steps:
1. Read the `agent.json` configuration (or use built-in defaults).
2. Create an `AgentCore` instance.
3. Dynamically resolve and load all plugins.
4. Start the Agent.

---

## 2. Why is it not called CLI?

The Runner was formerly named `apps/cli` but was renamed during the Plan03 Phase C refactoring. The reasons are as follows:

| Issue | Description |
|------|------|
| Misleading Name | The CLI interactive experience (readline, colored output) is provided by the `standard-function-stdio` plugin, not the Runner. |
| Implied Coupling | The name "CLI" suggests it is limited to the command line, but the Runner is unaware of and unconcerned with what the UI is. |
| Microkernel Philosophy | The Runner should be a pure launcher that is UI-agnostic. |

---

## 3. Runner Dependencies

The Runner depends only on three core packages:

```json
{
  "dependencies": {
    "@openstarry/core": "workspace:*",
    "@openstarry/sdk": "workspace:*",
    "@openstarry/shared": "workspace:*"
  }
}
```

**Zero plugin dependencies.** All `@openstarry-plugin/*` packages are dynamically loaded via `import()` at runtime.

---

## 4. How do plugins push input?

Before the Runner was decoupled, the stdio plugin relied on an `onInput` callback injected by the host. This forced the Runner to recognize the stdio plugin and perform special handling.

Current architecture:

```
SDK Layer: IPluginContext.pushInput(event: InputEvent)
         ↓
Core Layer: getPluginContext() automatically injects pushInput → core.pushInput
         ↓
Plugin Layer: stdio listener receives user input → ctx.pushInput({ source: "cli", data: text })
         ↓
Core Layer: pushInput → Slash command fast path / EventQueue → ExecutionLoop
```

**All Listener plugins** can push input via `ctx.pushInput`. The Runner does not participate in this workflow.

---

## 5. Default Configuration

When no `agent.json` is provided, the Runner uses the built-in `defaultConfig()`:

```typescript
function defaultConfig(): IAgentConfig {
  return {
    identity: { id: "openstarry-agent", name: "OpenStarry Agent", ... },
    cognition: { provider: "gemini-oauth", model: "gemini-2.0-flash", ... },
    plugins: [
      { name: "@openstarry-plugin/provider-gemini-oauth" },
      { name: "@openstarry-plugin/standard-function-fs" },
      { name: "@openstarry-plugin/standard-function-stdio" },
    ],
    guide: "default-guide",
  };
}
```

Note: Even in the default configuration, plugins are loaded using their full package names via dynamic import.

---

## 6. Process-Level Responsibilities

The Runner retains a few process-level concerns that do not fall under the responsibility of any plugin:

| Responsibility | Description |
|------|------|
| `process.exit()` | Listens for the `__QUIT__` event and executes process exit. |
| `SIGINT / SIGTERM` | Gracefully shuts down the Agent upon receiving termination signals. |
| Config Loading | Reads `agent.json`, performs Zod validation, and falls back to defaults. |

---

## 7. Future Evolution

The Runner can be replaced by different hosts without modifying any plugins:

| Host | Description |
|------|------|
| `apps/runner` | The current Node.js process host. |
| `apps/daemon` | A future daemon managing multiple Agent instances. |
| `apps/web-server` | A future Web service host receiving input via HTTP. |

Every host performs the same tasks: read config → create core → load plugins → start. The differences lie only in process management and external interfaces.
