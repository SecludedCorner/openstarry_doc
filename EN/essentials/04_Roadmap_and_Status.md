# OpenStarry: System Overview & Architecture Evolution

> *"We don't just build chatbots — we build an operating system for digital species."*

## What OpenStarry Includes

A complete, headless AI agent operating system:

| Layer | Components |
|-------|-----------|
| **Five Aggregates SDK** | TypeScript interfaces for IUI, IListener, IProvider, ITool, IGuide |
| **Agent Core** | 6-state execution loop, EventBus, context management, session isolation |
| **Safety System** | 3-level circuit breaker (resource → behavioral → human override) |
| **Plugin Ecosystem** | Transport (stdio, WebSocket, HTTP+SSE), Provider (Gemini OAuth), Tools (filesystem), Guides (character, skill) |
| **Session Management** | Per-connection isolation, session resumption, unified TraceId |
| **Orchestrator Daemon** | `openstarryd` — lifecycle management, process isolation (Docker, WASM), state persistence |
| **MCP Protocol** | Inter-agent cooperation via JSON-RPC 2.0, tool exposure, recursion guard |
| **TUI Dashboard** | Real-time agent monitoring, interactive designer, plugin orchestration |
| **Observability** | Structured JSON logging, trace context propagation, health checks |

## How It Was Built: Eight Phases of Evolution

OpenStarry was not built all at once. It evolved through eight deliberate architectural phases, each adding a new layer of capability — like a digital organism developing from embryo to maturity.

### Phase 1: Genesis — The Skeleton

The monorepo scaffold, TypeScript strict configuration, pnpm workspace protocol, and the Five Aggregates SDK interfaces. At this stage, the agent was just a set of type definitions — pure potential, like DNA before the organism forms.

**Key deliverable:** 11 workspace packages with clean dependency boundaries.

### Phase 2: Conscious Kernel — The Heartbeat

The execution loop state machine, EventBus, context management, and the safety monitor with circuit breakers. The agent gained its "heartbeat" — a continuous cycle of sense → think → act → learn.

**Key deliverables:**
- 6-state execution loop (WAITING → ASSEMBLING → AWAITING_LLM → PROCESSING → EXECUTING_TOOLS → SAFETY_LOCKOUT)
- Token budget (100k), loop cap (50 iterations), repetitive failure detection (SHA-256 fingerprinting)
- Async FIFO event queue decoupling producers from consumer
- Sliding window context management with system prompt anchoring

### Phase 3: Body & Senses — The First Organs

The plugin infrastructure: Plugin Loader with automatic hook registration, Tool Registry with Zod→JSON Schema conversion, Provider Registry, UI Registry, Listener Registry, Guide Registry. The empty Core could now receive organs.

**Key deliverable:** A universal plugin loading system where `install one npm package = gain complete domain capability`.

### Phase 4: First Breath — Alive

The CLI runner (`apps/runner`), the stdio plugin (terminal I/O), the Gemini OAuth provider (brain), filesystem tools (hands), and character init guide (soul). For the first time, an OpenStarry agent could perceive, think, act, and speak.

**Key deliverables:**
- End-to-end agent lifecycle: boot → load plugins → listen → respond → shutdown
- OAuth 2.0 + PKCE authentication with AES-256-GCM machine-bound token encryption
- Path-sandboxed filesystem tools with Zod validation
- Pain mechanism: errors as feedback, not crashes

### Phase 5: Multi-Channel — Many Bodies

Transport plugins for WebSocket and HTTP. The same agent could now be reached via terminal, WebSocket, or HTTP API simultaneously. Session isolation gave each connection its own conversation state.

**Key deliverables:**
- WebSocket: bidirectional communication, ping/pong health checks, session resumption
- HTTP: REST endpoints + Server-Sent Events for real-time streaming
- Transport Bridge: routes events from Core to all registered UIs with error isolation
- Per-connection session management with backward-compatible default session

### Phase 6: Fractal Society — Agent Cooperation

MCP (Model Context Protocol) integration. Agents can expose tools to other agents, call other agents' tools, and form dynamic teams.

**Key deliverables:**
- MCP Server: any agent can expose its tools via JSON-RPC 2.0
- MCP Client: any agent can call other agents' tools
- Tool whitelisting: `expose_tools` / `private_tools` configuration
- Recursion guard: TraceId + depth counter (max 5 layers) prevents infinite loops
- DevTools: debugging interface for inspecting agent internals
- Fractal composition: teams of agents expose the same interface as individual agents

### Phase 7: Daemon — Persistent Life

The Orchestrator Daemon (`openstarryd`). Agents become true OS-level processes with persistent lifecycle management.

**Key deliverables:**
- Agent lifecycle management: spawn, monitor, restart, terminate
- State persistence and recovery: agents survive restarts, memory carries forward
- Process isolation: Docker containers, WASM sandboxes
- Hardware Abstraction Layer (HAL): camera, sensors, actuators — agents in the physical world
- Auto-start registry: agents launch at system boot

### Phase 8: OS Evolution — The Operating System

The TUI Dashboard and interactive agent designer. The full vision realized: an operating system for digital life.

**Key deliverables:**
- **TUI Dashboard** (`openstarry`): real-time monitoring of all running agents — CPU, memory, thoughts/sec, status
- **Interactive Designer** (`openstarry design`): assemble an agent's Five Aggregates visually — choose brain, senses, tools, soul
- **Workflow Engine** (`openstarry run workflow.yaml`): chain agents into multi-step workflows with dynamic handoff
- **Plugin Sync** (`openstarry plugin sync`): manage and update the capability library
- **Agent Registration** (`openstarry register`): register agents for daemon management

## Technical Specifications

Seven technical specs define the system's contracts:

| Spec | Scope | Key Detail |
|------|-------|------------|
| **01: Command Registry** | Plugin CLI command registration | Dynamic discovery, `registry.json` as source of truth |
| **02: Event Bus Protocol** | Standard event envelope | UUID, timestamp, traceId, source, sessionId, priority |
| **03: Plugin Interfaces** | Five Aggregates SDK | IUI, IListener, IProvider, ITool, IGuide + IPluginContext |
| **04: Context Management** | Hierarchical memory | Immediate (5-10 turns, locked) → Short-term (sliding window) → Long-term (RAG) |
| **05: Security Protocol** | Multi-layer defense | Filesystem sandbox, command whitelist, resource quotas (50 ticks, 100k tokens, 30s timeout) |
| **06: MCP Protocol** | Inter-agent communication | JSON-RPC 2.0, tool exposure whitelist, recursion guard (max 5 layers) |
| **07: Management Zone** | Daemon architecture | Process/Docker/WASM isolation, HAL for IoT, YAML orchestration rules |

## Pluggable Memory Strategies

Different agents need different memory. OpenStarry's context management is a plugin, not hardcoded:

| Strategy | How It Works | Best For |
|----------|-------------|----------|
| **Sliding Window** (default) | FIFO — keep N most recent turns, discard oldest | Simple Q&A, short tasks |
| **Dynamic Summarization** | Compress old turns into natural language summary using lightweight LLM | Long-term companions, complex projects |
| **Key State Extraction** | Extract structured JSON state from dialogue, discard prose | Form-filling bots, booking agents |

```json
{
  "contextStrategy": {
    "type": "plugin",
    "pluginId": "std-summarization-strategy",
    "config": { "compressionModel": "gemini-1.5-flash", "threshold": 20 }
  }
}
```

> *"Core stays lightweight. Complex memory logic is delegated to specialist strategy modules."*

## Agent Configuration

A complete agent is defined declaratively through its Five Aggregates:

```jsonc
{
  "identity": { "id": "dev-bot-01", "name": "Resilient Developer" },
  "plugins": [
    // Brain: cognitive engine
    { "name": "@openstarry-plugin/provider-gemini" },
    // Hands: filesystem operations
    { "name": "@openstarry-plugin/standard-function-fs" },
    // Senses: terminal input
    { "name": "@openstarry-plugin/standard-function-stdio" },
    // Network body: WebSocket transport
    { "name": "@openstarry-plugin/transport-websocket" },
    // Soul: persona and pain mechanism
    { "name": "@openstarry-plugin/guide-character-init",
      "config": { "characterFile": "./personas/developer.md" } }
  ],
  "policy": {
    "safety": { "max_consecutive_errors": 3 }
  }
}
```

## By The Numbers

| Metric | Value |
|--------|-------|
| Workspace packages | 11 |
| Plugin packages | 7+ |
| Architecture docs | 27 |
| Deep-dive articles | 14 |
| Technical specs | 7 |
| Implementation plans | 9+ |
| Execution loop states | 6 |
| Event types | 25+ |
| Lines of documentation | 10,000+ |
