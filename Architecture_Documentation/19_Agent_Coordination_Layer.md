# 19. Agent Coordination Layer v2 (Multi-Agent)

**Version**: v2.0 (Cycle 03-1 Research Draft)
**Status**: Research Team Draft -- pending engineering team formal spec
**Basis**: D2-R4, D2-R5, D2-R7, D2-R9
**Previous Version**: Doc 19 v1 (single-agent coordination)

This document defines the Coordination Layer within the Daemon, extended for multi-agent operation. The Coordination Layer is the "central nervous system" of the OpenStarry system, managing plugin registration, agent communication routing, cross-agent event forwarding, and lifecycle management.

> **Convention**: Sections marked `[EXISTING]` are unchanged from Doc 19 v1. Sections marked `[NEW]` are multi-agent additions. Sections marked `[MODIFIED]` update existing content.

---

## 1. Core Responsibilities [MODIFIED]

The Coordination Layer is a logical module within the **Daemon** process. It bears the following responsibilities:

1.  **Plugin Registry Authority** [EXISTING]:
    *   Maintains the catalog of all available plugins (scanned from `~/.openstarry/plugins`).
    *   Responds to Agent load requests, providing plugin physical paths.
    *   Handles `openstarry plugin sync` commands.

2.  **Agent Registry** [MODIFIED -- expanded for multi-agent]:
    *   Records all active Agents' PID, communication ports (Socket/Pipe), status, **capabilities, and parent-child relationships**.
    *   Provides `getAgent(id)` for CLI (`attach`) or other Agents.
    *   **[NEW] Cross-agent queries: `findByCapability(cap)`, `findByRole(role)`.**
    *   **[NEW] Process Tree tracking: `parentAgentId`, child agent list, orphan reparenting.**

3.  **Message Routing** [MODIFIED -- capability-enforced]:
    *   When an Agent sends a message to another Agent, the Coordination Layer resolves the target and **verifies communication capabilities** before forwarding.
    *   **[NEW] MessageRouter component enforces `canSendTo`/`canReceiveFrom` on every message forward.**

4.  **[NEW] EventBridge** (cross-agent event forwarding):
    *   Daemon-level service for selective cross-agent event routing.
    *   Agents opt-in via event type whitelist in `agent.json`.

5.  **[NEW] L2 Global ServiceRegistry** (cross-agent service discovery):
    *   Daemon-level service enabling Agents to discover services provided by other Agents.
    *   Complements the existing L1 Local ServiceRegistry (per-Agent, unchanged).

---

## 2. Plugin Registration Flow [EXISTING]

When a developer installs a new plugin, or on system startup:

1.  **Scan**: Coordination Layer scans the plugin directory.
2.  **Validate**: Checks `plugin.json` legality.
3.  **Index**: Stores plugin `id`, `version`, `capabilities` in in-memory database.
    *   Index example: `Capability("weather") -> Plugin("openstarry-weather-v1")`.

---

## 3. Agent Startup Interaction [MODIFIED]

1.  **Request**: New Agent Core sends to Coordination Layer: "I need `standard/fs` and `gemini`."
2.  **Resolve**: Coordination Layer queries the index, confirms plugins exist and are version-compatible.
3.  **Authorize**: Checks `agent.json` permission declarations.
4.  **Return**: Returns plugin entry file paths.
5.  **Register**: Agent reports online status: "I am `data-bot-01`, I have `analyze-data` capability."
6.  **[NEW] Capability Declaration**: Agent registers its communication capabilities (`canSendTo`, `canReceiveFrom`, `exposedTools`) with the AgentRegistry.
7.  **[NEW] Discovery Broadcast**: Daemon broadcasts `agent:joining` CoordinationMessage to subscribed Agents (via EventBridge).
8.  **[NEW] Service Registration**: Agent's comm plugin reports local services to L2 Global ServiceRegistry via `service:register` IPC call.

---

## 4. [NEW] Multi-Agent AgentRegistry

### 4.1 AgentRegistryEntry (Extended)

```typescript
interface AgentRegistryEntry {
  // --- EXISTING fields ---
  agentId: string;
  pid: number;
  socketPath: string;
  status: AgentStatus;

  // --- NEW fields for multi-agent ---
  /** Parent Agent ID. null = spawned directly by Daemon (root Agent). */
  parentAgentId: string | null;

  /** Child Agent IDs managed by this Agent. */
  childAgentIds: string[];

  /** Communication capabilities declared in agent.json. */
  communication: {
    canSendTo: string[];      // Agent IDs or patterns (e.g., "*", "orchestrator")
    canReceiveFrom: string[]; // Agent IDs or patterns
    exposedTools: string[];   // Tool names callable by other Agents
    maxMessageSize?: number;  // bytes, default 4096
    eventSubscriptions?: string[]; // event types to receive via EventBridge
  };

  /** Capabilities this Agent provides (for cross-agent discovery). */
  declaredCapabilities: string[];

  /** Agent role (from agent.json, e.g., "researcher", "coder", "reviewer"). */
  role?: string;
}
```

### 4.2 Cross-Agent Queries

The AgentRegistry exposes new query methods for multi-agent scenarios:

```typescript
interface IAgentRegistry {
  // --- EXISTING ---
  getAgent(id: string): AgentRegistryEntry | undefined;
  listAgents(): AgentRegistryEntry[];

  // --- NEW ---
  /** Find Agents by declared capability. */
  findByCapability(capability: string): AgentRegistryEntry[];

  /** Find Agents by role. */
  findByRole(role: string): AgentRegistryEntry[];

  /** Get the full process tree from a given root. */
  getProcessTree(rootId?: string): IAgentTreeNode;

  /** Get all children of an Agent. */
  getChildren(parentId: string): AgentRegistryEntry[];

  /** Check if source Agent is allowed to send to target Agent. */
  canCommunicate(sourceId: string, targetId: string): boolean;
}

interface IAgentTreeNode {
  agent: AgentRegistryEntry;
  children: IAgentTreeNode[];
}
```

### 4.3 Process Tree [NEW]

The Process Tree tracks parent-child relationships between Agents, modeled after Unix process hierarchy:

- **Root Agents**: Spawned directly by Daemon (`parentAgentId = null`). Analogous to init children.
- **Child Agents**: Spawned by a parent Agent via `spawnChildAgent()`. Inherit a subset of parent capabilities.
- **Orphan Reparenting**: If a parent Agent terminates, its children are reparented to Daemon (PID 1 analogy). [Source: D2-R8, KERNEL F3]
- **Signal Propagation**: Parent termination sends `AGENT_SIGHUP` to all children. Children may choose to terminate (default) or continue under Daemon supervision.

```
Daemon (PID 1)
├── orchestrator (root, parentAgentId=null)
│   ├── researcher-01 (child, parentAgentId=orchestrator)
│   ├── researcher-02 (child, parentAgentId=orchestrator)
│   └── coder-01 (child, parentAgentId=orchestrator)
│       └── test-runner (grandchild, parentAgentId=coder-01)
└── monitor-agent (root, parentAgentId=null)
```

---

## 5. [NEW] MessageRouter (Capability-Enforced Routing)

The MessageRouter is a Daemon-level component responsible for all inter-agent message forwarding. It enforces communication capabilities as a **mechanism** constraint (non-bypassable).

### 5.1 Routing Flow

```
Agent A sends message to Agent B:
  1. Agent A's comm plugin sends message to Daemon via IPC
  2. MessageRouter receives the message
  3. CAPABILITY CHECK (mechanism):
     a. Verify A.canSendTo includes B (or "*")
     b. Verify B.canReceiveFrom includes A (or "*")
     c. If either fails -> DENY with error response (fail-closed, per Rule #29)
  4. TARGET RESOLUTION:
     a. Look up B in AgentRegistry
     b. If B not found -> DENY (agent not registered)
     c. If B.status == DRAINING or TERMINATED -> DENY
  5. FORWARD: Send message to B's IPC socket
  6. AUDIT: Log routing decision (source, target, allowed/denied, timestamp)
```

### 5.2 Routing Rules

| Rule | Type | Description |
|------|------|-------------|
| Capability check on every message | Mechanism | Cannot be disabled. Daemon MUST verify canSendTo/canReceiveFrom. |
| Default: zero capability | Mechanism | Agent with no `communication` section cannot send/receive. |
| Child capability subset of parent | Mechanism | `C_child.canSendTo` must be subset of `C_parent.canSendTo`. Enforced at `spawnChildAgent()`. |
| Capability revocation | Mechanism | Daemon can immediately revoke any Agent's communication capabilities. |
| Pattern matching (e.g., `"*"`) | Policy | Wildcard patterns are configurable per agent.json. |
| `maxMessageSize` enforcement | Policy | Default 4096 bytes, configurable. |

### 5.3 Security Model

Based on seL4-inspired capability model [D2-R5]:

- **Capability checking = mechanism** (Daemon-enforced, non-bypassable)
- **Permission rules = policy** (configurable in agent.json)
- **Default zero capability** (no communication section = no communication)
- **Hub-mediated routing regardless of transport** -- even when using alternative transport adapters (e.g., future Channels), all messages MUST route through Hub for capability verification [D2-R3, GUARDIAN]

---

## 6. [NEW] EventBridge (Cross-Agent Event Forwarding)

### 6.1 Architecture

EventBridge is a Daemon-level service that enables selective cross-agent event routing **without modifying** the per-Agent EventBus.

Per-Agent EventBus remains unchanged (single-process, in-memory, fire-and-forget). EventBridge supplements it by:

1. Agent's comm plugin (or comm-proxy sidecar) selectively forwards events to EventBridge via IPC.
2. EventBridge maintains subscription table: `Map<eventType, Set<agentId>>`.
3. Events forwarded only to subscribed Agents -- no global broadcast by default.

### 6.2 Subscription Model

```typescript
interface IEventBridge {
  /** Register an Agent's interest in specific event types. */
  subscribe(agentId: string, eventTypes: string[]): void;

  /** Unsubscribe an Agent from event types. */
  unsubscribe(agentId: string, eventTypes?: string[]): void;

  /** Forward an event from source Agent to all subscribers of that event type. */
  forward(sourceAgentId: string, event: AgentEvent): void;

  /** Get all subscribers for an event type. */
  getSubscribers(eventType: string): string[];
}
```

### 6.3 Configuration

Per-agent event subscriptions are configured in agent.json:

```json
{
  "communication": {
    "eventSubscriptions": ["tool:result", "agent:status_changed", "cycle:complete"]
  }
}
```

### 6.4 Failure Model

EventBridge is **observational** infrastructure. Per Rule #29:
- **EventBridge failure = fail-open**: If EventBridge is unavailable, Agents continue operating with per-Agent EventBus only. No cross-agent events are delivered, but Agent functionality is preserved.
- EventBridge does NOT affect Agent's internal control loop (Tenet #8 preserved).

---

## 7. [NEW] CoordinationMessage Extensions

Existing CoordinationMessage types: `agent:register`, `agent:deregister`, `agent:health`.

New types for multi-agent lifecycle [D2-R7, HERACLITUS F3]:

| Message Type | Direction | Payload | Trigger |
|-------------|-----------|---------|---------|
| `agent:joining` | Daemon -> subscribed Agents | `{ agentId, capabilities, role, parentAgentId }` | New Agent completes registration |
| `agent:leaving` | Daemon -> subscribed Agents | `{ agentId, gracePeriodMs, reason }` | Agent enters DRAINING state |
| `agent:status_changed` | Daemon -> subscribed Agents | `{ agentId, oldStatus, newStatus }` | Any Agent lifecycle state change |

These messages are delivered via EventBridge to Agents that have subscribed to the corresponding event types.

---

## 8. [NEW] L2 Global ServiceRegistry

### 8.1 Two-Layer Registry Model

| Layer | Scope | Location | Query Speed | Use Case |
|-------|-------|----------|-------------|----------|
| **L1: Local** | Per-Agent plugins | Core (existing ServiceRegistry) | Fast (in-process Map) | Plugin-to-plugin within same Agent |
| **L2: Global** | All Agents | Daemon CoordinationLayer | Slower (IPC round-trip) | Cross-agent service discovery |

### 8.2 Query Flow (DNS Analogy)

```
Agent A needs service "data-analysis":
  1. Query L1 Local ServiceRegistry (fast path)
     -> Found locally? Use it.
  2. Query L2 Global ServiceRegistry via IPC to Daemon (remote path)
     -> Found on Agent B? Route through MessageRouter.
     -> Not found? Service unavailable error.
```

### 8.3 Service Lifecycle

- **Registration**: Agent's comm plugin reports local services to Daemon via `service:register` IPC.
- **Deregistration**: Automatic on Agent DRAINING -> TERMINATED lifecycle transition.
- **Health**: Services inherit their Agent's health status. If Agent is unhealthy, its services are marked unavailable.

### 8.4 Separation from Blackboard

L2 Global ServiceRegistry is **separate** from the future Blackboard/Alaya implementation [D2-R7]:
- ServiceRegistry: structured metadata (name, version, capability). Typed schema.
- Blackboard: arbitrary shared content. Schema-flexible.
- Unification will be evaluated in Plan39+ when Blackboard design matures.

---

## 9. Implementation Technology [MODIFIED]

*   **Communication Protocol** [EXISTING]: gRPC or Named Pipes (IPC) for Core-Daemon communication.
*   **Storage** [EXISTING]: Lightweight embedded database for registry persistence.
*   **[NEW] MessageRouter**: Daemon in-process module, invoked on every inter-agent message.
*   **[NEW] EventBridge**: Daemon in-process service, started with Daemon lifecycle.
*   **[NEW] L2 ServiceRegistry**: Daemon in-process service, persistent across Agent restarts.

---

## 10. Persistence & Boot Strategy [EXISTING]

### A. Physical Persistence
Plugins installed via `openstarry plugin sync` or `add` are permanently stored in `~/.openstarry/plugins/`. They persist across restarts unless manually deleted.

### B. Registry Indexing
1.  **Fast Scan**: On Daemon startup, scan plugin directory mtime.
2.  **Incremental Refresh**: If unchanged, read from `registry.db` cache; if changed, re-parse `plugin.json`.
3.  **Stale Cleanup**: Remove entries for plugins no longer on disk.

### C. Runtime State
Registry persistence does not equal plugin instance persistence. On each startup, plugins reload and execute `initialize()` per Doc 18 protocol. Plugin runtime data must use the `state` interface (Doc 06).

---

## 11. Health Check & Self-Healing [EXISTING]

### A. Manual Drop-in
When a user copies a folder into `~/.openstarry/plugins/`:
1.  **Detect**: Daemon's file watcher or startup scan detects new directory.
2.  **Validate**: Attempt to parse `plugin.json`. Success -> `READY`. Failure -> `INVALID_MANIFEST` (yellow warning).

### B. Load Failure
If plugin code crashes during `initialize()`:
1.  **Isolate**: Loader catches exception, prevents Daemon crash.
2.  **Mark**: Status set to `QUARANTINED`.
3.  **Report**: Red indicator in Dashboard with error log access.

### C. Dependency Missing
If plugin A declares dependency on plugin B, but B is absent:
1.  **Check**: Dependency graph detects broken link.
2.  **Block**: Refuse to load A. Status: `UNSATISFIED`.
3.  **Guide**: CLI suggests: `openstarry plugin add B` or `openstarry plugin sync`.

---

## 12. Architecture Boundary Summary [NEW]

Per D2-R4 (Zero Core modification for multi-agent):

| Layer | Multi-Agent Responsibility | Modification |
|-------|---------------------------|-------------|
| **Core** | None. Per-Agent EventBus, per-Agent StateManager, per-Agent ExecutionLoop. | ZERO change |
| **SDK Types** | ICommChannel, CommMessage, CommCapability definitions. PluginHooks.commChannels?. | New type definitions |
| **Plugin** | PipelineChannel, HubSpokeChannel (future), MeshChannel (future). comm-proxy sidecar. | New plugins |
| **Daemon** | MessageRouter, EventBridge, Global ServiceRegistry, ProcessTree, TokenPool. IDaemonControlPlane extensions. | Daemon extensions |

This boundary is a **mechanism** constraint derived from Tenet #7 (Microkernel Purity): Core remains absolutely pure. All multi-agent complexity resides in Daemon and Plugin layers.

---

## Tenet Compliance

| Tenet | Assessment |
|-------|-----------|
| #1 (Independent Process) | COMPLIANT -- each Agent remains independent OS process with isolated Core |
| #2 (Everything Plugin) | COMPLIANT -- all communication is plugin-mediated via ICommChannel |
| #5 (Directory = Permission) | COMPLIANT -- extended to communication permissions (default deny) |
| #7 (Microkernel Purity) | COMPLIANT -- zero Core change; all multi-agent in Daemon + Plugin |
| #8 (Closed-loop Control) | COMPLIANT -- per-Agent EventBus and control loop unchanged |
| #9 (Extensibility) | COMPLIANT -- AgentRegistry queries, ICommChannel array slot enable unlimited extension |
| #10 (Fractal) | SUPPORTED -- Process Tree + Hub-and-Spoke enables recursive agent composition |

---

## References

- [D2-R4] Core adds zero new IPC primitives
- [D2-R5] seL4-inspired capability model
- [D2-R7] EventBridge + L2 Global ServiceRegistry design
- [D2-R8] Five-layer failure isolation, graceful shutdown
- [D2-R9] Doc update priority (Doc 19 = P0)
- [R1_KERNEL F3] Process Tree, orphan reparenting
- [R1_DARWIN F2] EventBridge as Daemon service
- [R1_DARWIN F3] Two-layer ServiceRegistry
- [R1_HERACLITUS F3] CoordinationMessage extensions, graceful shutdown
- [R1_TANENBAUM F4] seL4-inspired capability model
- [R1_GUARDIAN] Security boundaries, hub-mediated routing

---

*Research Team Draft -- Cycle 03-1, 2026-03-24*
*To be converted to formal engineering doc by engineering team.*
