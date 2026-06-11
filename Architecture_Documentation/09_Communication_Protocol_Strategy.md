# 09. Communication Protocol Strategy v2 (Multi-Agent)

**Version**: v2.0 (Cycle 03-1 Research Draft)
**Status**: Research Team Draft -- pending engineering team formal spec
**Basis**: D2-R1, D2-R2, D2-R3, D2-R6
**Previous Version**: Doc 09 v1 (single-agent "Communication as Plugin")

This document defines the core architecture philosophy for communication in the OpenStarry system, extended for multi-agent operation. The original "Communication as Plugin" principle is preserved and strengthened: Agent Core remains protocol-agnostic, and all communication -- including inter-agent messaging -- flows through plugins implementing the new `ICommChannel` unified interface.

> **Convention**: Sections marked `[EXISTING]` are unchanged from Doc 09 v1. Sections marked `[NEW]` are multi-agent additions. Sections marked `[MODIFIED]` update existing content.

---

## 1. Core Philosophy: Communication as Plugin [EXISTING, EXTENDED]

In OpenStarry, **Agent Core is "Protocol Agnostic"**. Core understands only internal "Events" and "Intents" and has no knowledge of how the external world communicates with it.

All external communication -- whether chatting with humans, collaborating with other machines, **or coordinating with other Agents** -- must go through **Communication Plugins**.

**[NEW] Multi-Agent Extension**: The "Communication as Plugin" principle extends naturally to multi-agent. Inter-agent communication is NOT a Core capability. It is provided by communication channel plugins (implementing `ICommChannel`) registered via the `commChannels` hook slot. An Agent with no communication plugins is a valid headless entity -- it simply cannot participate in multi-agent workflows.

This design enables an Agent to be a "Multi-headed" entity: it can interact simultaneously through multiple channels and protocols with different targets, including other Agents.

---

## 2. [NEW] Four Communication Modes

Multi-agent communication is organized into four modes, implemented in priority order [D2-R1]:

### 2.1 Implementation Priority

| Priority | Mode | Plan Target | LOC Estimate | Core Change |
|----------|------|-------------|-------------|-------------|
| **1st** | Pipeline | Plan37 (MVP) | ~150-200 | Zero |
| **2nd** | Hub-and-Spoke | Plan38 | ~400-500 | Zero |
| **3rd** | API Runtime | Plan39+ | ~800+ | Zero |
| **4th** | Mesh | Phase 6 late | ~1200+ | Zero |

### 2.2 Pipeline

Sequential orchestration: Agent A produces output, passed as input to Agent B, then to Agent C.

```
[Agent A] --output--> [Agent B] --output--> [Agent C] --output--> [Result]
```

- **Characteristics**: Simplest mode. Zero Core modification. Pure external orchestration via Daemon `spawnAgent()`.
- **CAP Trade-off**: CP (strong consistency, partition tolerant, sacrifices availability) [MESH F1].
- **Tenet #7 Purity**: Highest -- no Core awareness of Pipeline. Orchestration logic entirely in Plugin/Daemon.
- **Industry Precedent**: Unix pipes, LangGraph sequential chains, CrewAI sequential process.
- **Implementation**: `PipelineChannel` plugin implementing `ICommChannel(capabilities: ['messaging'], topology: 'pipeline')`.

### 2.3 Hub-and-Spoke

Central coordinator (Hub = Daemon/Orchestrator Agent) routes messages between worker Agents (Spokes).

```
        [Agent A]
            |
[Agent B] --Hub-- [Agent C]
            |
        [Agent D]
```

- **Characteristics**: Natural second step -- Daemon is already a natural Hub (QNX model). Adds MessageRouter + structured toolset.
- **CAP Trade-off**: CP with Hub as single point (MESH analysis).
- **Tenet #7 Purity**: Low intrusion -- MessageRouter is Daemon logic, not Core.
- **Industry Precedent**: AutoGen GroupChat, LangGraph routing, QNX message passing.
- **Implementation**: `HubSpokeChannel` plugin implementing `ICommChannel(capabilities: ['messaging', 'streaming'])`.

### 2.4 API Runtime

Full lifecycle control via programmatic API. Agents managed as service instances.

```
[Runtime API] --> spawn/stop/configure Agents programmatically
                  Full lifecycle control (create, configure, start, stop, restart)
                  IAgentDriver abstraction
```

- **Characteristics**: Most complete lifecycle control. Prerequisite for true Mesh. Requires `IAgentDriver` interface.
- **Tenet #7 Purity**: Medium intrusion -- `IAgentDriver` defined in SDK, runtime logic in Daemon.
- **Prerequisite for**: Mesh mode (independent network endpoints require API Runtime) [MESH R2 challenge].

### 2.5 Mesh

Peer-to-peer communication with service discovery. Any Agent can communicate directly with any other.

```
[Agent A] <---> [Agent B]
    |    \     /    |
    |     \   /     |
[Agent C] <---> [Agent D]
```

- **Characteristics**: Highest complexity (O(n^2) connections). Highest Tenet #7 risk. Requires API Runtime as prerequisite.
- **CAP Trade-off**: AP (availability + partition tolerance, eventual consistency) [MESH F6].
- **Deferred until**: Pipeline + Hub-and-Spoke proven stable.

### 2.6 Classification

- **Implementation order (Pipeline -> Hub-and-Spoke -> API Runtime -> Mesh)**: Policy -- sequence can be adjusted based on engineering velocity and Master directives.
- **Pipeline-first for MVP**: Mechanism-adjacent -- Pipeline is the only mode requiring zero Core change; starting with anything else would violate Tenet #7 minimalism.

---

## 3. [NEW] ICommChannel Unified Interface

All communication modes are implemented through the `ICommChannel` interface, defined in SDK [D2-R2].

### 3.1 Design Principles

1. **Defined in SDK, not Core**: `packages/sdk/src/types/comm-channel.ts`. Zero Core modification. Tenet #7 preserved.
2. **Capability-based**: Base interface defines minimum (metadata + lifecycle). Optional capabilities declared per channel.
3. **Extends IPluginService**: Channels are discoverable via ServiceRegistry.
4. **Array hook slot**: `PluginHooks.commChannels?: ICommChannel[]` -- consistent with `vedanaSensors[]`, `tools[]`, `listeners[]`.

### 3.2 Interface Definition

```typescript
interface ICommChannel extends IPluginService {
  /** Supported capability set for this channel. */
  capabilities: CommCapability[];

  /** Channel topology type. */
  topology: 'point-to-point' | 'broadcast' | 'request-response' | 'pipeline';

  /** Current connection status. */
  getStatus(): CommChannelStatus;

  /** Lifecycle */
  connect(target?: string): Promise<void>;
  disconnect(): Promise<void>;

  /** Message-level (requires 'messaging' capability) */
  send?(target: string, message: CommMessage): Promise<void>;
  onMessage?(handler: (msg: CommMessage, from: string) => void): () => void;
  reply?(msgId: string, response: CommMessage): Promise<void>;

  /** Stream-level (requires 'streaming' capability) */
  subscribe?(topic: string): AsyncIterable<CommMessage>;
  publish?(topic: string, message: CommMessage): Promise<void>;

  /** RPC-level (requires 'rpc' capability) */
  call?(method: string, params: unknown): Promise<unknown>;
  expose?(method: string, handler: (params: unknown) => Promise<unknown>): void;
}
```

### 3.3 CommCapability

```typescript
type CommCapability = 'messaging' | 'streaming' | 'rpc' | 'composable';
```

| Capability | Methods Required | Use Case |
|-----------|-----------------|----------|
| `messaging` | `send`, `onMessage`, `reply` | Point-to-point messaging (Pipeline, Hub-and-Spoke) |
| `streaming` | `subscribe`, `publish` | Pub/sub event streams (EventBridge integration) |
| `rpc` | `call`, `expose` | Remote procedure calls (MCP, REST API) |
| `composable` | (none additional) | Declares channel supports composition via CompositeChannel |

### 3.4 CommMessage

```typescript
interface CommMessage {
  id: string;
  timestamp: number;
  source: string;       // Source Agent ID
  target?: string;      // Target Agent ID (optional for broadcast)
  payload: unknown;
  performative?: CommPerformative; // FIPA ACL performative
  /** Trace context for recursion guard (extends Doc 06 X-OpenStarry-Trace-ID) */
  traceId?: string;
  traceDepth?: number;
  /** Message timeout in ms. Outer timeout > sum of inner (Timeout Hierarchy, D2-R8). */
  timeoutMs?: number;
}

/** FIPA ACL performatives (subset for MVP). */
type CommPerformative =
  | 'inform'       // Share information
  | 'request'      // Request action
  | 'agree'        // Agree to request
  | 'refuse'       // Refuse request
  | 'propose'      // Propose action
  | 'query-ref'    // Query for information
  | 'cfp';         // Call for proposals
```

### 3.5 CommChannelStatus

```typescript
type CommChannelStatus = 'disconnected' | 'connecting' | 'connected' | 'error';
```

---

## 4. [NEW] IPC Primitives

Three primitives form the minimum for any `messaging` channel [D2-R6]:

1. **`send(target, message)`** -- point-to-point message delivery
2. **`onMessage(handler)`** -- receive messages (returns unsubscribe function)
3. **`reply(msgId, response)`** -- respond to a specific message

`publish/subscribe` is a composition over send, available in channels declaring the `streaming` capability. It is NOT a Core primitive. The Hub internally implements `publish` as `for each subscriber: channel.send(sub, msg)`.

This follows MINIX minimalism: kernel provides only minimum IPC, everything else in userspace [D2-R6, TANENBAUM].

---

## 5. [NEW] PipelineChannel (Plan37 Reference Implementation)

The first `ICommChannel` implementation. Minimal, leverages existing IPC infrastructure.

```typescript
class PipelineChannel implements ICommChannel {
  name = 'pipeline';
  version = '1.0.0';
  capabilities: CommCapability[] = ['messaging'];
  topology = 'pipeline' as const;

  private status: CommChannelStatus = 'disconnected';
  private handlers: Array<(msg: CommMessage, from: string) => void> = [];

  getStatus(): CommChannelStatus { return this.status; }

  async connect(target?: string): Promise<void> {
    // Uses existing IPC Server (Unix Domain Socket / Named Pipe)
    // No new transport dependency
    this.status = 'connected';
  }

  async disconnect(): Promise<void> {
    this.status = 'disconnected';
  }

  async send(target: string, message: CommMessage): Promise<void> {
    // Route through Daemon's MessageRouter via IPC
    // MessageRouter enforces capability check before forwarding
    await this.ipcClient.send('agent:send_message', { target, message });
  }

  onMessage(handler: (msg: CommMessage, from: string) => void): () => void {
    this.handlers.push(handler);
    return () => {
      this.handlers = this.handlers.filter(h => h !== handler);
    };
  }

  async reply(msgId: string, response: CommMessage): Promise<void> {
    await this.ipcClient.send('agent:reply_message', { msgId, response });
  }
}
```

### 5.1 Transport

Plan37 uses existing IPC infrastructure (Unix Domain Socket / Named Pipe). No new transport dependency required. All messages route through Daemon's MessageRouter for capability verification.

---

## 6. Communication Plugin Classification [MODIFIED]

### 6.1 Channel Plugins (Human-facing) [EXISTING]

Plugins handling non-structured or semi-structured interaction, simulating "conversation":

*   **Target**: Human users, chatbot platforms, notification systems.
*   **Examples**: WebSocket, WhatsApp/Telegram/Discord adapter, Email listener, Voice interface.

### 6.2 Protocol Plugins (Machine-facing) [EXISTING]

Plugins handling structured data exchange, simulating RPC or resource access:

*   **Target**: Other Agents, legacy systems, microservices, IDEs, databases.
*   **Examples**: MCP (recommended standard), REST API adapter, ActivityPub, custom binary protocol.

### 6.3 [NEW] Inter-Agent Channel Plugins

Plugins specifically for Agent-to-Agent communication within OpenStarry:

*   **Target**: Other OpenStarry Agents (same Daemon or remote).
*   **Examples**:
    *   `PipelineChannel` -- sequential handoff (Plan37)
    *   `HubSpokeChannel` -- hub-mediated routing (Plan38)
    *   `MeshChannel` -- peer-to-peer discovery (Phase 6 late)
    *   `ChannelTransportAdapter` -- Claude Code Channels as optional transport (future, after stability assessment)
*   **All must implement `ICommChannel`**.
*   **All route through Daemon's MessageRouter for capability checking** (even if transport is direct).

---

## 7. Multi-Headed Agent Model [MODIFIED]

```
                Agent Core (Headless)
               /         |          \
     [WebSocket]   [PipelineChannel]   [MCP Server]
          |              |                  |
      Human User    Other Agent         External Tool
```

An Agent can simultaneously:
1. Chat with a human via WebSocket plugin
2. Receive Pipeline input from an orchestrator Agent via PipelineChannel
3. Expose tools to external systems via MCP Server plugin

Core does not know it is handling three different protocols. It receives `Input`, thinks, produces `Output`. Each plugin distributes Output to the correct channel.

---

## 8. [NEW] Claude Code Channels Positioning

Claude Code Channels = **optional transport adapter plugin**. Not a dependency. [D2-R3]

| Aspect | Assessment |
|--------|-----------|
| Plan37 dependency | **None** -- all multi-agent uses existing IPC |
| Session isolation | Hard limitation: no session-to-session communication |
| Research Preview status | Stability risk for production dependency |
| Future potential | May be contributed as `ICommChannel` implementation after stability (v2.2+, GitHub #32631 cross-session support) |
| Security constraint | Even with Channels transport, all messages MUST route through Hub for capability verification [GUARDIAN] |

---

## 9. MCP's Special Position [MODIFIED]

MCP (Model Context Protocol) remains a **first-class citizen** for tool interoperability and resource sharing.

**[NEW]** However, MCP is now positioned as **one `ICommChannel` implementation** among several, not the sole inter-agent protocol. MCP excels at structured tool invocation (`rpc` capability) but is not optimized for lightweight inter-agent messaging (`messaging` capability).

Recommended usage:
- **Agent-to-Tool**: MCP (primary)
- **Agent-to-Agent Pipeline**: PipelineChannel (Plan37)
- **Agent-to-Agent Hub-and-Spoke**: HubSpokeChannel (Plan38)
- **Agent-to-External**: MCP, REST, WebSocket (per use case)

---

## 10. [NEW] Communication Configuration

Following the three-layer config pattern (Baseline Rule #10):

### Layer 1: Core Zero-Default

Core assumes no communication exists. An Agent with no communication plugins is a valid headless entity.

### Layer 2: SDK Defaults

```typescript
// packages/sdk/src/constants.ts
export const DEFAULT_COMM_TIMEOUT_MS = 30000;
export const DEFAULT_COMM_MAX_RETRIES = 3;
export const DEFAULT_COMM_MAX_MESSAGE_SIZE = 4096;
export const DEFAULT_COMM_CAN_SEND_TO = ['orchestrator'];
export const DEFAULT_COMM_CAN_RECEIVE_FROM = ['orchestrator'];
```

### Layer 3: IAgentConfig Override

```json
{
  "communication": {
    "canSendTo": ["orchestrator", "researcher-*"],
    "canReceiveFrom": ["orchestrator", "*"],
    "exposedTools": ["analyze_data", "summarize"],
    "maxMessageSize": 8192,
    "eventSubscriptions": ["tool:result", "agent:status_changed"]
  }
}
```

### Zod Schema Alignment (PROC-SPEC-3)

Per Baseline Rule #21, `IAgentConfig.communication` must be reflected in the Zod schema (`config-schema.ts`) when implemented. Zod `safeParse()` silently strips unknown keys -- any new field MUST be added to the schema.

---

## 11. [NEW] CommChannelRegistry

Parallels the VedanaRegistry pattern [SUSSMAN F7]:

```typescript
interface ICommChannelRegistry {
  register(channel: ICommChannel): void;
  unregister(name: string): void;
  get(name: string): ICommChannel | undefined;
  list(): ICommChannel[];
  findByCapability(cap: CommCapability): ICommChannel[];
}
```

- Created in PluginLoader alongside VedanaRegistry, GearArbiterRegistry, etc.
- `PluginLoaderDeps` extended with `commChannelRegistry?: ICommChannelRegistry`.
- PluginLoader processes `hooks.commChannels` array and registers each channel.

---

## Tenet Compliance

| Tenet | Assessment |
|-------|-----------|
| #2 (Everything Plugin) | COMPLIANT -- all communication modes are plugins implementing ICommChannel |
| #7 (Microkernel Purity) | COMPLIANT -- ICommChannel in SDK, channels in Plugin, routing in Daemon. Core unchanged. |
| #8 (Closed-loop Control) | COMPLIANT -- each Agent maintains independent control loop regardless of communication mode |
| #9 (Extensibility) | COMPLIANT -- ICommChannel array slot enables unlimited channel implementations |
| #10 (Fractal) | SUPPORTED -- Hub-and-Spoke + Process Tree enables recursive agent composition |

---

## References

- [D2-R1] Implementation order: Pipeline -> Hub-and-Spoke -> API Runtime -> Mesh
- [D2-R2] ICommChannel in Plan37 SDK, only PipelineChannel implemented
- [D2-R3] Claude Code Channels = optional transport adapter
- [D2-R6] Three primitives (send/receive/reply) + publish as streaming capability
- [R1_SUSSMAN F6] ICommChannel interface design
- [R1_MESH F1, F6] CAP theorem analysis of communication modes
- [R1_ATHENA F2, F3] Hub-and-Spoke evaluation, industry comparison
- [R1_TANENBAUM F5] Tenet #7 purity ranking, MINIX IPC model
- [R1_RUSSELL F5, F6] Evaluation matrix, Blackboard/Stigmergy analysis

---

*Research Team Draft -- Cycle 03-1, 2026-03-24*
*To be converted to formal engineering doc by engineering team.*
