# Architecture Overview

## Five Aggregates

All plugin capabilities in OpenStarry map to five core interfaces:

| Aggregate | Interface | Role |
|-----------|-----------|------|
| Form (色) | IUI | User interface (output) |
| Sensation (受) | IListener | Event listeners (input) |
| Perception (想) | IProvider | LLM service providers |
| Formation (行) | ITool | Executable tools |
| Consciousness (識) | IGuide | System prompts / guidance |

## Microkernel Design

- **Core has zero plugin dependencies** — all features are provided via plugins
- Plugins communicate with the core through `ctx.pushInput()`, never calling APIs directly
- Plugins use the factory pattern: `createXxxPlugin()` → `IPlugin { manifest, factory(ctx) }`

### Core Packages

| Package | Description |
|---------|-------------|
| `packages/sdk` | Type contracts (interfaces, events, errors) |
| `packages/core` | Agent core (EventBus, ExecutionLoop, safety layer) |
| `packages/shared` | Shared utilities (logger, UUID, validation) |
| `packages/plugin-signer` | Plugin signature verification |
| `apps/runner` | CLI launcher (entry point for all commands) |

## Event-Driven Flow

```
User Input → IListener → pushInput() → EventBus → ExecutionLoop → IProvider (LLM)
                                                                      ↓
UI Output ← IUI ← AgentEvent ← EventBus ← Tool Results ← ITool ← LLM Response
```

1. **Input**: IListener receives user input and injects it into the event bus via `pushInput()`
2. **Processing**: ExecutionLoop takes events from the EventBus and passes them to IProvider (LLM)
3. **Tool Calls**: Tool requests in the LLM response are executed by ITool; results flow back to EventBus
4. **Output**: IUI listens for AgentEvents and renders responses to the user

## Further Reading

- `packages/sdk/README.md` — All interface definitions
- `packages/core/README.md` — Core architecture details
