# OpenStarry: System Overview & Architectural Evolution

> *"We don't just build Chatbots; we build an operating system for digital species."*

## What OpenStarry Includes

A complete, Headless AI Agent operating system:

| Layer | Component |
|------|------|
| **Five Aggregates SDK** | TypeScript interfaces: IUI, IListener, IProvider, ITool, IGuide |
| **Agent Core** | 6-state execution loop, EventBus, context management, session isolation |
| **Security System** | 3-layer circuit breakers (Resource level → Behavioral level → Human override) |
| **Plugin Ecosystem** | Transport (stdio, WebSocket, HTTP+SSE), Provider (Gemini OAuth), Tools (File system), Guides (Persona, Skill) |
| **Session Management** | Per-connection isolation, session recovery, unified TraceId |
| **Orchestrator Daemon** | `openstarryd` — lifecycle management, process isolation (Docker, WASM), state persistence |
| **MCP Protocol** | Inter-Agent collaboration, based on JSON-RPC 2.0, tool exposure, recursion protection |
| **TUI Dashboard** | Real-time Agent monitoring, interactive designer, plugin orchestration |
| **Observability** | Structured JSON logging, Trace context propagation, health checks |

## How It Was Built: Eight Evolutionary Stages

OpenStarry was not built all at once. It evolved through eight carefully planned architectural stages, each adding a new layer of capability—much like a digital organism developing from embryo to maturity.

### Stage 1: Genesis — The Skeleton

Monorepo scaffolding, strict TypeScript settings, pnpm workspace protocol, and the Five Aggregates SDK interfaces. At this stage, the Agent was just a set of type definitions—pure potential, like DNA before an organism takes shape.

**Key Outputs:** 11 workspace packages with clear dependency boundaries.

### Stage 2: Core Consciousness — The Heartbeat

Execution loop state machine, EventBus, context management, and a safety monitor with circuit breakers. The Agent gained a "heartbeat"—a continuous cycle of sense → think → act → learn.

**Key Outputs:**
- 6-state execution loop (WAITING → ASSEMBLING → AWAITING_LLM → PROCESSING → EXECUTING_TOOLS → SAFETY_LOCKOUT)
- Token budget (100k), loop cap (50 iterations), repetitive failure detection (SHA-256 fingerprinting)
- Asynchronous FIFO event queue, decoupling producers and consumers
- Sliding window context management with System Prompt anchoring

### Stage 3: Body & Senses — The First Organs

Plugin infrastructure: Plugin Loader with auto-hook registration, Tool Registry with Zod-to-JSON Schema conversion, Provider Registry, UI Registry, Listener Registry, and Guide Registry. The empty Core could now receive organs.

**Key Outputs:** A universal plugin loading system: "install one npm package = acquire full domain capabilities."

### Stage 4: The First Breath — It's Alive

CLI Runner (`apps/runner`), stdio plugin (terminal I/O), Gemini OAuth Provider (the brain), filesystem tools (the hands), and persona initialization Guide (the soul). For the first time, an OpenStarry Agent could perceive, think, act, and speak.

**Key Outputs:**
- End-to-end Agent lifecycle: startup → load plugins → listen → respond → shutdown
- OAuth 2.0 + PKCE authentication with AES-256-GCM machine-bound token encryption
- Path-sandboxed filesystem tools with Zod validation
- Pain mechanism: errors as feedback, not crashes

### Stage 5: Multi-channel — Multiple Bodies

Transport plugins for WebSocket and HTTP. The same Agent can now be accessed simultaneously via terminal, WebSocket, or HTTP API. Session isolation provides independent conversation states for every connection.

**Key Outputs:**
- WebSocket: Bi-directional communication, ping/pong health checks, session recovery
- HTTP: REST endpoints + Server-Sent Events for real-time streaming
- Transport Bridge: Routes events from the Core to all registered UIs with error isolation
- Per-connection session management, backward-compatible with default sessions

### Stage 6: Fractal Society — Agent Collaboration

MCP (Model Context Protocol) integration. Agents can expose tools to other Agents, call tools of other Agents, and form dynamic teams.

**Key Outputs:**
- MCP Server: Any Agent can expose its tools via JSON-RPC 2.0
- MCP Client: Any Agent can invoke tools from other Agents
- Tool Whitelisting: `expose_tools` / `private_tools` configuration
- Recursion Protection: TraceId + depth counter (max 5) to prevent infinite loops
- DevTools: Debugging interface for inspecting Agent internal state
- Fractal Composition: Agent teams expose the same interface as individual Agents

### Stage 7: Daemon — Persistent Life

Orchestrator Daemon (`openstarryd`). Agents become true OS-level processes with persistent lifecycle management.

**Key Outputs:**
- Agent lifecycle management: spawn, monitor, restart, terminate
- State persistence and recovery: Agents survive restarts, memory is passed forward
- Process isolation: Docker containers, WASM sandboxes
- Hardware Abstraction Layer (HAL): Cameras, sensors, actuators—Agents enter the physical world
- Auto-boot registration: Agents start on system boot

### Stage 8: OS Evolution — The Operating System

TUI dashboard and interactive Agent designer. The realization of the full vision: an operating system for digital life.

**Key Outputs:**
- **TUI Dashboard** (`openstarry`): Real-time monitoring of all running Agents—CPU, RAM, thoughts/sec, status
- **Interactive Designer** (`openstarry design`): Visually assemble the Agent's Five Aggregates—choose brain, senses, tools, soul
- **Workflow Engine** (`openstarry run workflow.yaml`): Chain Agents into multi-step workflows with dynamic handoffs
- **Plugin Sync** (`openstarry plugin sync`): Manage and update the capability library
- **Agent Registration** (`openstarry register`): Register Agents with the Daemon for management

## Technical Specifications

Seven technical specifications define the system's contracts:

| Spec | Scope | Key Details |
|------|------|---------|
| **01: Command Registry** | Plugin CLI command registration | Dynamic discovery, `registry.json` as the single source of truth |
| **02: Event Bus Protocol** | Standard event packets | UUID, timestamp, traceId, source, sessionId, priority |
| **03: Plugin Interfaces** | Five Aggregates SDK | IUI, IListener, IProvider, ITool, IGuide + IPluginContext |
| **04: Context Management** | Hierarchical memory | Immediate (5-10 turns, locked) → Short-term (sliding window) → Long-term (RAG) |
| **05: Security Protocol** | Multi-layer defense | Filesystem sandbox, command whitelisting, resource quotas (50 ticks, 100k tokens, 30s timeout) |
| **06: MCP Protocol** | Inter-Agent communication | JSON-RPC 2.0, tool exposure whitelist, recursion protection (max 5 levels) |
| **07: Management Zone** | Daemon architecture | Process/Docker/WASM isolation, HAL for IoT, YAML orchestration rules |

## Pluggable Memory Strategies

Different Agents require different memory styles. OpenStarry context management is a plugin, not hardcoded:

| Strategy | Operation | Best Use Case |
|------|---------|-----------|
| **Sliding Window** (Default) | FIFO — Keep last N turns, discard oldest | Simple Q&A, short-term tasks |
| **Dynamic Summary** | Uses lightweight LLM to compress old history into natural language summaries | Long-term companionship, complex projects |
| **State Extraction** | Extracts structured JSON state from dialogue, discards narrative text | Form-filling bots, booking Agents |

```json
{
  "contextStrategy": {
    "type": "plugin",
    "pluginId": "std-summarization-strategy",
    "config": { "compressionModel": "gemini-1.5-flash", "threshold": 20 }
  }
}
```

> *"The Core remains lightweight; complex memory management logic is offloaded to specialized strategy modules."*

## Agent Configuration

A complete Agent is defined declaratively via its Five Aggregates:

```jsonc
{
  "identity": { "id": "dev-bot-01", "name": "Resilient Developer" },
  "plugins": [
    // [Perception] Brain: Cognitive engine
    { "name": "@openstarry-plugin/provider-gemini" },
    // [Volition] Hands: Filesystem operations
    { "name": "@openstarry-plugin/standard-function-fs" },
    // [Sensation] Senses: Terminal input
    { "name": "@openstarry-plugin/standard-function-stdio" },
    // [Sensation+Form] Network Body: WebSocket transport
    { "name": "@openstarry-plugin/transport-websocket" },
    // [Consciousness] Soul: Persona & pain mechanism
    { "name": "@openstarry-plugin/guide-character-init",
      "config": { "characterFile": "./personas/developer.md" } }
  ],
  "policy": {
    "safety": { "max_consecutive_errors": 3 }
  }
}
```

## Data at a Glance

| Metric | Value |
|------|------|
| Workspace Packages | 11 |
| Plugin Packages | 7+ |
| Architecture Docs | 27 |
| Deep Dive Articles | 14 |
| Technical Specs | 7 |
| Implementation Plans | 9+ |
| Execution Loop States | 6 |
| Event Types | 25+ |
| Lines of Documentation | 10,000+ |
