# OpenStarry Implementation Plan 01 — MVP Alpha Foundation

> **狀態**: ✅ 已完成 (2026-02-04)

## Target
Implement Phase 1~4 to achieve **MVP v0.1 Alpha** milestone:
- Local CLI Agent runs
- Gemini as LLM Provider (PKCE + OAuth, ref: `ref/openoctopus/packages/provider-gemini-oauth`)
- Local filesystem operations (fs tools)
- Basic memory (5-turn sliding window)
- Complete "perceive → think → tool call → correct → respond" loop

## Architecture Overview
- **Monorepo**: `openstarry/` with `apps/` and `packages/`
- **Plugin Ecosystem**: `openstarry_plugin/` (flat structure, separate repo)
- **Language**: TypeScript (ESM modules)
- **Package Manager**: pnpm workspaces
- **Reference**: OpenOctopus patterns (factory functions, EventBus, Zod, async generators)

---

## Structure

```
openstarry/
├── package.json
├── pnpm-workspace.yaml
├── tsconfig.base.json
├── tsconfig.json
├── example-agent.json
├── apps/runner/src/
│   ├── bin.ts          # CLI entry + host bootstrap
│   └── index.ts        # Re-export
├── packages/sdk/src/
│   ├── types/          # message, tool, provider, plugin, listener, guide, agent, events
│   ├── errors/         # AgentError hierarchy
│   └── interfaces/     # IContextManager, IStateManager
├── packages/shared/src/
│   ├── logger/         # Structured logging
│   ├── utils/          # UUID, Zod validation helpers
│   └── constants/      # Event constants
└── packages/core/src/
    ├── bus/            # EventBus (pub/sub)
    ├── execution/      # EventQueue, ExecutionLoop (state machine)
    ├── state/          # StateManager (in-memory)
    ├── memory/         # ContextManager (sliding window)
    ├── infrastructure/ # ToolRegistry, ProviderRegistry, ListenerRegistry,
    │                   # GuideRegistry, CommandRegistry, PluginLoader
    ├── security/       # SecurityLayer (path guardrails)
    ├── agents/         # AgentCore (main orchestrator)
    └── transport/      # TransportBridge (event routing to listeners)

openstarry_plugin/
├── provider-gemini-oauth/src/index.ts   # Gemini PKCE+OAuth provider
├── standard-function-fs/src/index.ts    # fs.read/write/list/mkdir/delete
└── standard-function-stdio/src/index.ts # CLI I/O listener + default guide
```

## Implementation Steps (Completed)

1. **Monorepo Scaffolding** — pnpm workspace, tsconfig, package.json
2. **SDK Types** — Message, Tool, Provider, Plugin, Listener, Guide, Agent, Events, Errors
3. **Shared Utils** — Logger, UUID, Zod helpers, event constants
4. **Core: EventBus + EventQueue** — Pub/sub + async FIFO queue
5. **Core: StateManager + ContextManager** — In-memory history + sliding window (5 turns)
6. **Core: Registries** — Tool, Provider, Listener, Guide, Command registries + PluginLoader
7. **Core: SecurityLayer** — Path normalization + scope validation
8. **Core: ExecutionLoop** — IDLE → ASSEMBLING_CONTEXT → AWAITING_LLM → PROCESSING_RESPONSE → (tool_use loop)
9. **Core: AgentCore + Transport** — Main orchestrator + event broadcast bridge
10. **Plugin: provider-gemini-oauth** — PKCE, OAuth callback server, AES-256-GCM token storage, SSE streaming, function calling
11. **Plugin: standard-function-fs** — fs.read/write/list/mkdir/delete with path security
12. **Plugin: standard-function-stdio** — stdin listener + ANSI stdout UI + default guide persona
13. **CLI: bin.ts** — Config loading, plugin resolution, AgentCore bootstrap
14. **Integration** — `pnpm install && pnpm build` passes, CLI starts and responds to commands

## Key Design Decisions

1. **ESM only** — All packages use ES modules
2. **Zod validation** — Tool parameters validated at runtime
3. **Factory pattern** — Plugins expose factory functions
4. **Async generators** — Provider streaming uses async generators
5. **No Daemon** — MVP CLI directly bootstraps AgentCore (standalone)
6. **PKCE + OAuth** — Gemini uses Google OAuth, tokens encrypted with machine-bound AES-256-GCM
7. **In-memory state** — No persistence in MVP, state in process memory
8. **Path security** — FS tools restricted via path normalization + allowedPaths

## Verification

```bash
cd openstarry
pnpm install && pnpm build           # All 7 packages compile
node apps/runner/dist/bin.js             # Starts with default config
node apps/runner/dist/bin.js --config ./example-agent.json  # Starts with custom config

# First use: /provider login gemini   → Opens browser for OAuth
# Test:      /help                    → Lists 4 commands
# Test:      /reset                   → Resets conversation
# Test:      /quit                    → Exits cleanly
# Chat:      "Hello"                  → Gemini responds (after OAuth login)
# Tools:     "List files in ."        → Agent calls fs.list
```
