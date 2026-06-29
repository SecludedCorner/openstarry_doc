# 57. Multi-Agent Communication Interface Specification

**Version**: v1.0 (Cycle 03-1 Research Draft)
**Status**: IMPLEMENTED + FROZEN (v0.59.6 honest re-verification). `ICommChannel` and its companion types are FROZEN in `packages/sdk/src/types/comm-channel.ts`; reference implementations exist and are tested: `PipelineChannel` (`apps/runner/src/daemon/pipeline-channel.ts`), the `comm-pipeline` plugin, and `CompositeChannel` (§11, `apps/runner/src/daemon/composite-channel.ts`, 16 tests). The `commChannels[]` hook is wired in `plugin-loader.ts`.

> **[v0.59.8-alpha — registry now has a consumer]** Honest correction: until v0.59.8 the `commChannelRegistry` was **populated by the loader but never consumed** in production — channels' `send()`/`onMessage()` did real work nowhere (PipelineChannel / CompositeChannel / the `comm-pipeline` plugin were verify-only reference impls; the real cross-daemon transport was the separate daemon `CommTransport` of Fractal Society C/T1). v0.59.8 adds the **first live consumer**: the new `@openstarry-plugin/comm-channel-p2p` plugin provides a real point-to-point `ICommChannel` ('messaging') whose `send()` routes through the daemon `DAEMON_COMM` transport, and the daemon now `connect()`s registered channels at startup and **dispatches every inbound `CommMessage` to them** (`commChannelRegistry.list()` → channel `deliverInbound` → `onMessage`). Two-process e2e: `daemon-channel.e2e.test.ts`. **§8.1 clarification**: an `ICommChannel.send()` MUST route through an authenticated transport (e.g. `DAEMON_COMM`, which enforces `validateOutbound` + HMAC + replay via the daemon `MessageRouter`); the interface itself does not enforce this — it is the channel implementation's responsibility. PipelineChannel / CompositeChannel / comm-pipeline remain **in-process composition reference impls** (honestly marked, not cross-process).

**Remaining sub-gap**: §10 transport-plugin dual-registration (see §10 note).
**Basis**: D2-R2, D2-R5, D2-R6
**Category**: NEW document (Technical Specification)

This document provides the complete specification for `ICommChannel`, the unified communication interface that all multi-agent communication channels must implement. This is the foundational type contract for Phase 6 multi-agent communication.

---

## 1. Overview

### 1.1 Purpose

`ICommChannel` is the **single abstraction** through which all communication modes (Pipeline, Hub-and-Spoke, API Runtime, Mesh) are exposed to the OpenStarry system. It ensures:

1. **Uniform discovery**: All channels are registered via `commChannels` hook and discoverable via `CommChannelRegistry`.
2. **Capability negotiation**: Channels declare what they support (`messaging`, `streaming`, `rpc`, `composable`), avoiding forced implementation of unused features.
3. **Transport independence**: The interface defines semantics (what), not transport (how). A channel may use Unix Domain Socket, Named Pipe, HTTP, WebSocket, or any other transport.

### 1.2 Location

```
packages/sdk/src/types/comm-channel.ts   -- interface + type definitions
packages/sdk/src/index.ts                -- re-export
```

### 1.3 Design Lineage

- Extends `IPluginService` (existing service discovery mechanism)
- Follows array hook slot pattern (like `vedanaSensors[]`, `tools[]`, `listeners[]`)
- Follows three-layer config (Core zero-default, SDK defaults, IAgentConfig override)
- Capability model inspired by seL4 capability-based security [D2-R5]

---

## 2. ICommChannel Interface

### 2.1 Full Definition

```typescript
import { IPluginService } from './service';

/**
 * Unified communication channel interface.
 * All multi-agent communication modes implement this interface.
 *
 * Design: capability-based -- base interface defines minimum (metadata + lifecycle),
 * optional methods activated by capability declaration.
 */
export interface ICommChannel extends IPluginService {
  // --- Inherited from IPluginService ---
  // name: string;     // Channel unique name (e.g., "pipeline", "hub-spoke", "mcp-rpc")
  // version: string;  // Semantic version

  // --- Metadata ---

  /** Capabilities supported by this channel. */
  readonly capabilities: readonly CommCapability[];

  /** Channel topology type. */
  readonly topology: CommTopology;

  // --- Lifecycle ---

  /** Get current connection status. */
  getStatus(): CommChannelStatus;

  /**
   * Establish connection.
   * @param target - Optional target identifier (e.g., Agent ID, URL).
   *                 For pipeline: next stage Agent ID.
   *                 For hub-spoke: hub Agent ID or Daemon.
   *                 For mesh: peer Agent ID.
   */
  connect(target?: string): Promise<void>;

  /** Gracefully disconnect. Complete in-flight messages before closing. */
  disconnect(): Promise<void>;

  // --- Message-level (requires 'messaging' capability) ---

  /**
   * Send a message to a target Agent.
   * Routes through Daemon's MessageRouter for capability verification.
   * @throws CommCapabilityError if channel lacks 'messaging' capability.
   */
  send?(target: string, message: CommMessage): Promise<void>;

  /**
   * Register a handler for incoming messages.
   * @returns Unsubscribe function.
   */
  onMessage?(handler: CommMessageHandler): () => void;

  /**
   * Reply to a specific message by ID.
   * Enables request-response patterns over messaging.
   */
  reply?(msgId: string, response: CommMessage): Promise<void>;

  // --- Stream-level (requires 'streaming' capability) ---

  /**
   * Subscribe to a topic. Returns async iterable of messages.
   * Used for EventBridge integration and pub/sub patterns.
   */
  subscribe?(topic: string): AsyncIterable<CommMessage>;

  /**
   * Publish a message to a topic.
   * Hub internally implements as: for each subscriber: channel.send(sub, msg).
   */
  publish?(topic: string, message: CommMessage): Promise<void>;

  // --- RPC-level (requires 'rpc' capability) ---

  /**
   * Call a remote method on another Agent.
   * @returns Method result.
   */
  call?(method: string, params: unknown): Promise<unknown>;

  /**
   * Expose a method for remote invocation by other Agents.
   * Only methods listed in agent.json `exposedTools` are callable.
   */
  expose?(method: string, handler: (params: unknown) => Promise<unknown>): void;
}

/** Handler type for incoming messages. */
export type CommMessageHandler = (msg: CommMessage, from: string) => void;
```

### 2.2 Capability Enforcement

Methods marked with `?` (optional) are only available when the channel declares the corresponding capability. Calling an unsupported method should throw `CommCapabilityError`:

```typescript
class CommCapabilityError extends Error {
  constructor(
    public readonly channel: string,
    public readonly requiredCapability: CommCapability,
    public readonly availableCapabilities: readonly CommCapability[]
  ) {
    super(
      `Channel "${channel}" does not support "${requiredCapability}". ` +
      `Available: [${availableCapabilities.join(', ')}]`
    );
  }
}
```

---

## 3. CommCapability Enum

```typescript
/**
 * Communication capabilities that a channel may declare.
 * Channels only implement methods for declared capabilities.
 */
export type CommCapability =
  | 'messaging'    // send, onMessage, reply
  | 'streaming'    // subscribe, publish
  | 'rpc'          // call, expose
  | 'composable';  // supports CompositeChannel composition
```

### 3.1 Capability-Method Mapping

| Capability | Required Methods | Optional Methods | Use Case |
|-----------|-----------------|-----------------|----------|
| `messaging` | `send`, `onMessage` | `reply` | Point-to-point messaging |
| `streaming` | `subscribe`, `publish` | -- | Pub/sub event streams |
| `rpc` | `call`, `expose` | -- | Remote procedure calls |
| `composable` | (none additional) | -- | Marker: channel can participate in CompositeChannel |

### 3.2 Mechanism vs Policy Classification [D2-R2, D2-R6]

| Aspect | Classification | Rationale |
|--------|---------------|-----------|
| ICommChannel interface contract | **Mechanism** | All communication modes MUST implement this interface |
| Three messaging primitives (send/onMessage/reply) as minimum | **Mechanism** | ICommChannel contract for 'messaging' capability |
| Publish as 'streaming' capability | **Policy** | Channel implementation choice |
| commChannels as array slot | **Mechanism** | Consistent with existing hook precedents |
| PipelineChannel as sole Plan37 implementation | **Policy** | Plan37 scope decision |

---

## 4. CommMessage Type

```typescript
export interface CommMessage {
  id: string;              // UUID v4
  timestamp: number;       // Unix ms
  source: string;          // Source Agent ID (verified by MessageRouter)
  target?: string;         // Target Agent ID (optional for broadcast)
  payload: unknown;        // JSON-serializable
  performative?: CommPerformative; // FIPA ACL intent (default: 'inform')
  traceId?: string;        // Distributed tracing (extends Doc 06 X-OpenStarry-Trace-ID)
  traceDepth?: number;     // Incremented per hop; rejected if > MAX_TRACE_DEPTH (5)
  timeoutMs?: number;      // Timeout Hierarchy: outer > sum of inner (D2-R8)
  correlationId?: string;  // For request-response matching (reply sets this = original id)
}

type CommPerformative =
  | 'inform' | 'request' | 'agree' | 'refuse'
  | 'propose' | 'query-ref' | 'cfp' | 'failure';
```

---

## 5. CommChannelStatus

```typescript
/**
 * Channel connection lifecycle states.
 */
export type CommChannelStatus =
  | 'disconnected'  // Not connected. Initial state.
  | 'connecting'    // Connection in progress.
  | 'connected'     // Active and ready for communication.
  | 'draining'      // Completing in-flight messages, not accepting new ones.
  | 'error';        // Connection error. May attempt reconnection.
```

State transitions:

```
disconnected --> connecting --> connected --> draining --> disconnected
                     |              |
                     v              v
                   error          error
                     |
                     v
                disconnected (after recovery timeout)
```

---

## 6. CommTopology

```typescript
/**
 * Topology declared by the channel, informing the system about
 * the communication pattern this channel supports.
 */
export type CommTopology =
  | 'point-to-point'     // Direct A->B messaging
  | 'broadcast'          // One-to-many (pub/sub)
  | 'request-response'   // Synchronous call/return (RPC)
  | 'pipeline';          // Sequential A->B->C handoff
```

---

## 7. PipelineChannel Reference Implementation

Plan37 MVP implementation. Implements `messaging` capability only, uses existing IPC infrastructure.

```typescript
class PipelineChannel implements ICommChannel {
  readonly name = 'pipeline';
  readonly version = '1.0.0';
  readonly capabilities: readonly CommCapability[] = ['messaging'];
  readonly topology: CommTopology = 'pipeline';
  private status: CommChannelStatus = 'disconnected';
  private ipcClient: IIPCClient; // Injected, reuses existing Daemon IPC

  getStatus(): CommChannelStatus { return this.status; }

  async connect(_target?: string): Promise<void> {
    this.status = 'connected'; // Reuses existing IPC -- no new transport
  }

  async disconnect(): Promise<void> {
    this.status = 'draining';
    // Wait for in-flight messages (bounded by grace period), then:
    this.status = 'disconnected';
  }

  async send(target: string, message: CommMessage): Promise<void> {
    // Routes through Daemon's MessageRouter for capability verification
    await this.ipcClient.send('agent:send_message', { target, message });
  }

  onMessage(handler: CommMessageHandler): () => void {
    const unsub = this.ipcClient.on('agent:message_received', (data) => {
      handler(data.message, data.from);
    });
    return unsub;
  }

  async reply(msgId: string, response: CommMessage): Promise<void> {
    await this.ipcClient.send('agent:reply_message', {
      msgId, response: { ...response, correlationId: msgId }
    });
  }
}
```

---

## 8. Capability Model: Mechanism vs Policy

### 8.1 Security at Daemon (Mechanism)

All inter-agent messages route through Daemon's MessageRouter, regardless of which `ICommChannel` implementation is used. The MessageRouter performs:

1. **Source verification**: Is `message.source` the actual sending Agent?
2. **canSendTo check**: Does source Agent's `communication.canSendTo` include target?
3. **canReceiveFrom check**: Does target Agent's `communication.canReceiveFrom` include source?
4. **exposedTools check** (for RPC): Is the requested method in target's `exposedTools` whitelist?
5. **Trace depth check**: Is `traceDepth < MAX_TRACE_DEPTH`?

All checks are **mechanism** (non-bypassable). Failure mode is **fail-closed** (deny) per Baseline Rule #29.

### 8.2 Policy and Defaults

- **Permission rules** (`canSendTo`/`canReceiveFrom`/`exposedTools` values) are **policy** -- configurable per agent.json (see Doc 09 v2 Section 10 and Section 9 below).
- **Default zero capability**: Agent with no `communication` section cannot send/receive (Core zero-default, Rule #10).
- **Child subset of parent**: `C_child.canSendTo` must be subset of `C_parent.canSendTo` (and likewise for `canReceiveFrom`, `exposedTools`). Enforced at `spawnChildAgent()`. This is a **mechanism** constraint (lattice `C_child <= C_parent`).

---

## 9. Configuration: IAgentConfig.communication

### 9.1 ICommConfig

```typescript
interface ICommConfig {
  canSendTo?: string[];           // Agent IDs or patterns
  canReceiveFrom?: string[];      // Agent IDs or patterns
  exposedTools?: string[];        // Tool names callable by other Agents
  maxMessageSize?: number;        // bytes
  eventSubscriptions?: string[];  // event types for EventBridge
  timeoutMs?: number;             // communication timeout
  maxRetries?: number;            // retry count for failed delivery
}
```

### 9.2 SDK Defaults and Zod Schema

SDK defaults: `DEFAULT_COMM_CAN_SEND_TO = ['orchestrator']`, `DEFAULT_COMM_CAN_RECEIVE_FROM = ['orchestrator']`, `DEFAULT_COMM_EXPOSED_TOOLS = []`, `DEFAULT_COMM_MAX_MESSAGE_SIZE = 4096`, `DEFAULT_COMM_TIMEOUT_MS = 30000`, `DEFAULT_COMM_MAX_RETRIES = 3`.

Per Baseline Rule #21 (PROC-SPEC-3), `IAgentConfig.communication` MUST be reflected in Zod schema. See Doc 09 v2 Section 10 for full Zod definition.

---

## 10. Migration Path for Existing Transport Plugins

> ⚠️ **[實作狀態 — v0.59.6] 尚未實作（誠實標記）。** `transport-http` / `transport-websocket` 目前**仍只註冊為 `listeners`**（已對 plugin 原始碼 grep 確認：兩者 src 皆無 `commChannels` 參照）。下列「雙註冊」遷移為**設計路徑、非已落地行為**。這是 §11 CompositeChannel 之外、doc 53 唯一未竟的機械性子項；屬可選後續，與本文其餘已落地部分（FROZEN 介面＋PipelineChannel／comm-pipeline／CompositeChannel）分開計。

Existing transport plugins (`transport-http`, `transport-websocket`) currently register as `listeners` in PluginHooks. To participate in multi-agent communication:

Migration steps: (1) Wrap existing transport logic in `ICommChannel`. (2) Declare capabilities (e.g., `transport-websocket` -> `['messaging', 'streaming']`). (3) Register via `hooks.commChannels`. (4) Retain backward compatibility by providing BOTH `listeners` (legacy) and `commChannels` (multi-agent) in PluginHooks.

```typescript
// After migration -- dual registration for backward compatibility:
export const factory = (ctx: IPluginContext): PluginHooks => ({
  listeners: [createWebSocketListener(ctx)],    // legacy single-agent
  commChannels: [new WebSocketChannel(ctx)],     // multi-agent support
});
```

---

## 11. Composition Model: CompositeChannel

Channels declaring `composable` capability can be composed [SUSSMAN F5]. `CompositeChannel` wraps multiple `ICommChannel` instances with a `CompositionStrategy` (fallback, broadcast, or pipeline chaining). Lifecycle methods delegate to all child channels.

Constraints:
- **Maximum composition depth**: 3 levels (prevents unbounded nesting).
- **Capability intersection**: CompositeChannel capabilities = intersection of all child capabilities.
- **Only composable channels**: Channels must declare `composable` to participate.
- **Strategy types**: `fallback` (primary + secondary), `broadcast` (send to all), `pipeline` (chain sequentially).

> ✅ **[實作狀態 — v0.59.6] 已落地。** `apps/runner/src/daemon/composite-channel.ts` 實作 `CompositeChannel implements ICommChannel`：三策略（fallback 首個成功者勝／broadcast best-effort 全發／pipeline 逐段必成）、capabilities＝子通道交集、僅接受 `composable` 子通道、`MAX_COMPOSITION_DEPTH=3` 巢狀上限、lifecycle 委派全子通道、`onMessage` 扇入。測試 `composite-channel.test.ts`（16 例）。與 `PipelineChannel` 同為凍結 `ICommChannel` 的 reference 實作。

---

## 12. CommChannelRegistry

### 12.1 Interface

```typescript
export interface ICommChannelRegistry {
  register(channel: ICommChannel): void;
  unregister(name: string): void;
  get(name: string): ICommChannel | undefined;
  list(): ICommChannel[];
  findByCapability(cap: CommCapability): ICommChannel[];
  findByTopology(topology: CommTopology): ICommChannel[];
}
```

### 12.2 PluginHooks Extension and PluginLoader Integration

`PluginHooks` gains `commChannels?: ICommChannel[]` (array slot, consistent with `vedanaSensors[]`, `tools[]`). PluginLoader iterates `hooks.commChannels` and registers each channel with `deps.commChannelRegistry`. See Doc 09 v2 Section 11 for details.

---

## Tenet Compliance

| Tenet | Assessment |
|-------|-----------|
| #2 (Everything Plugin) | COMPLIANT -- all channels are plugins registered via commChannels hook |
| #5 (Directory = Permission) | COMPLIANT -- extended to communication permissions (default zero capability) |
| #7 (Microkernel Purity) | COMPLIANT -- ICommChannel in SDK types, channels in Plugin, routing in Daemon. Zero Core code. |
| #8 (Closed-loop Control) | COMPLIANT -- per-Agent control loop unaffected; communication is supplementary |
| #9 (Extensibility) | COMPLIANT -- array slot enables unlimited channel implementations |

---

## References

- [D2-R2] ICommChannel defined in Plan37 SDK, only PipelineChannel implemented
- [D2-R5] seL4-inspired capability model, Daemon enforcement
- [D2-R6] Three primitives (send/onMessage/reply), publish as streaming capability
- [R1_SUSSMAN F1-F7] PluginHooks gap analysis, ICommChannel design, commChannels hook, composition
- [R1_TANENBAUM F1, F4] MINIX IPC primitives, seL4 capability model
- [R1_DARWIN F3] Two-layer ServiceRegistry (L1 Local, L2 Global)
- [Baseline Rule #10] Three-layer config: IAgentConfig -> SDK DEFAULT_* -> Core zero-default
- [Baseline Rule #21] PROC-SPEC-3 schema-config alignment
- [Baseline Rule #29] Observation fail-open, enforcement fail-closed

---

*Research Team Draft -- Cycle 03-1, 2026-03-24*
*To be converted to formal engineering doc by engineering team.*
