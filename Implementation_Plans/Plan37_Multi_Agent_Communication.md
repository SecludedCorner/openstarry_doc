# Plan37 規格書 — Phase 6 Multi-Agent Foundation

**版本目標**: v0.37.0-alpha
**估計 LOC**: ~630-830 production + ~400 test
**基礎版本**: v0.36.1-alpha (v0.36.0-alpha + hotfixes)
**研究來源**: Cycle 03-1 R3 decisions (D1-R1~R5, D2-R1~R10, D3-R1~R3, D4-R1~R4)

---

## 總覽

Plan37 是 Phase 6 (Multi-Agent) 的第一個工程 Plan，承擔三重任務：
1. **清除 Cycle 02 技術債** — BUG-3 修復 (W1, P0)
2. **建立多代理基礎設施** — ICommChannel + IMcpTransport + Process Tree (W2, P1)
3. **推進宣言合規** — AC-6 前五識類型化 + 通訊能力模型 (W3, P1)

附帶條件性交付：EventBridge + L2 Global ServiceRegistry (W4, P2)。

### Master 補充指示（2026-03-24）

Master 確認多代理通訊架構方向：**所有節點皆 MCP Server + MCP Client**。

```
CLI #1 (Server+Client) ←──→ openstarry-channel (Server+Client) ←──→ CLI #2 (Server+Client)
                                      ↕
                              CLI #3 (Server+Client)
```

- **openstarry-channel** = OpenStarry 自建的 MCP Hub 進程（核心基礎設施，Required 級別）
- ≠ Claude Code Channels（Anthropic Research Preview feature，仍為 optional transport adapter）
- Plan37 定義 IMcpTransport 介面作為預備；Plan38 實作 openstarry-channel
- D2-R3 修訂：openstarry-channel = Required 核心基礎設施；Claude Code Channels = optional
- 評估詳見：`discussions/master_directive_multiagent_comm_evaluation.md`

---

## Wave 1 (P0): BUG-3 修復 + 累積 Clamp 驗證 (~50 LOC)

### BUG-3 概要

**確認狀態**: CONFIRMED (D1-R1)
**雙根因**:
- RC-1: loop.ts 每輪 tool_use batch 僅呼叫 route() 一次（非每工具）[程式碼: packages/core/src/execution/loop.ts#processEvent, L475-486]
- RC-2: gear-arbiter-static 使用 Array.some() first-match，遮蔽高風險工具 [程式碼: openstarry_plugin/gear-arbiter-static/src/index.ts#createStaticRuleEvaluator, L50-67]

### C1: Worst-Risk riskCategory Selection (~15 LOC)

- **位置**: `openstarry_plugin/gear-arbiter-static/src/index.ts`, createStaticRuleEvaluator (L50-67)
- **現行行為**: First-match — 迭代 rules，首個 `some()` 匹配即 return
- **目標行為**: Worst-risk — 掃描所有 rules × 所有 tools，回傳最高風險 riskCategory
- **風險排序**: `destructive(3) > state_modifying(2) > read_only(1) > informational(0)`
- **分類**: **Mechanism** — 決定哪個安全約束適用於 route() 決策
- **實作草案**:
```typescript
const RISK_ORDER: Record<string, number> = {
  destructive: 3, state_modifying: 2, read_only: 1, informational: 0
};
let worstRisk: MatchedRule | null = null;
for (const rule of config.rules) {
  const matched = context.proposedToolCalls.some(tc => matchPattern(rule.pattern, tc.name));
  if (matched) {
    if (!worstRisk || RISK_ORDER[rule.riskCategory] > RISK_ORDER[worstRisk.riskCategory]) {
      worstRisk = rule;
    }
  }
}
return worstRisk
  ? { action: worstRisk.gear, confidence: worstRisk.confidence, riskCategory: worstRisk.riskCategory, ... }
  : { action: 'abstain', ... };
```
- **BABBAGE BCT**: route() 介面簽名不變，行為修正恢復原有語義（非機制/策略轉變）[D1-R1]
- **測試**: 混合風險 batch {fs.list, fs.write, fs.delete} 須回傳 riskCategory='destructive'
- **來源**: D1-R1 Component 1

### C2: Per-Tool Audit Events (~25 LOC)

- **位置**: `packages/core/src/execution/loop.ts`, tool execution loop (~L533-613)
- **新增事件**: `audit:tool_audited` per tool execution
- **事件酬載**:
```typescript
interface AuditToolEvent {
  toolName: string;
  inferredRiskCategory: RiskCategory;  // via SDK inferRiskCategory()
  executionResult: 'success' | 'error';
  batchIndex: number;
  batchSize: number;
  routeRiskCategory: RiskCategory;     // batch-level worst-risk from route()
  timestamp: number;
}
```
- **行為**: Non-blocking，在 `for (const tc of pendingToolCalls)` 迴圈中每次工具執行後透過 EventBus emit
- **AuditTrailWriter**: 須訂閱 `audit:tool_audited` 事件類型（~5 LOC 新增）
- **分類**: **Policy** — 可觀測性增強，不影響控制行為
- **安全保證**: fail-open (Rule #29) — 事件發送失敗不阻斷工具執行
- **must-invoke 驗證**: 每輪結束後，audit:tool_audited 事件計數 === 已執行工具計數（D1-R4）。不符時發出 warning log（觀測型，fail-open）
- **測試**: 3 工具 batch 須產生 3 個 `audit:tool_audited` 事件，各含正確 per-tool riskCategory
- **來源**: D1-R1 Component 2, D1-R4

### C3: Cumulative Clamp 驗證 + 對稱地板 (~5 LOC)

- **位置**: `openstarry_plugin/auditor-threshold/src/threshold-auditor.ts`, L36-37
- **驗證**: `MAX_CUMULATIVE_POSITIVE = 0.05` (Rule #26, Master directive 02-12_final D1-R2)
  - 若現行程式碼為 0.1，修正為 0.05
- **新增**: `MAX_CUMULATIVE_NEGATIVE = -0.05` (對稱地板，provisional per D1-R3)
  - GUARDIAN 非對稱提案 (+0.05/-0.10) 正式登記 Cycle 03-2 評估
  - 所需證據：WIENER 穩定性分析 + KNUTH W2-R1 實證 + BABBAGE 機制/策略分類
- **分類**: 地板值 = **Policy**；Clamp 機制本身 = **Mechanism**
- **來源**: D1-R3

### C4: Must-Invoke 審計計數驗證 (~5 LOC)

- **位置**: loop.ts after tool batch completion
- **驗證**: `audit:tool_audited` event count === executed tool count
- **失敗行為**: Warning log（觀測型，fail-open）
- **分類**: **Mechanism** — must-invoke 是執行迴圈的架構保證
- **形式定義**: CTL `AF invoke(c)` — 所有路徑上，元件 c 必被觸發 [D1-R4]
- **來源**: D1-R4

### Wave 1 約束

- 不得引入新 policy 常量至 Core (Tenet #7, MR-6)
- RISK_ORDER 映射位於 gear-arbiter-static plugin，為 **mechanism**（固定排序，不可配置）
- MAX_CUMULATIVE_POSITIVE = 0.05 為 Master-directed (Rule #26) — 不可協商
- MAX_CUMULATIVE_NEGATIVE = -0.05 為 provisional — 依 Cycle 03-2 審查結果
- W1 須獨立驗證後方可進入 W2（BUG-3 修復為 W2-R1 前提條件）

---

## Wave 2 (P1): ICommChannel + IMcpTransport + Process Tree (~330 LOC)

### C5: ICommChannel SDK Type Definition (~80 LOC)

- **位置**: `packages/sdk/src/types/comm-channel.ts` (NEW)
- **介面**: `ICommChannel extends IPluginService`
- **能力**: `CommCapability = 'messaging' | 'streaming' | 'rpc' | 'composable'`
- **訊息類型**:
```typescript
interface CommMessage {
  id: string;
  from: string;         // sender agentId
  to: string;           // target agentId or '*' for broadcast
  performative: string; // FIPA-style: 'inform' | 'request' | 'confirm' | ...
  payload: unknown;
  traceId: string;      // recursion guard (extending Doc 06)
  traceDepth: number;   // current depth in call chain
  timestamp: number;
}

type CommChannelStatus = 'connecting' | 'connected' | 'disconnected' | 'error';
```
- **方法**: `connect()`, `disconnect()`, `getStatus()`, `send(target, message)`, `onMessage(handler)`, `reply(msgId, response)`
- **最小 messaging 原語** (D2-R6): send / onMessage / reply — 任何 ICommChannel with 'messaging' capability
- **串流原語**: publish / subscribe — 用於 'streaming' capability (Plan38 Hub-and-Spoke)
- **三層配置** (Rule #10): Core zero-default（不假設任何通訊存在）→ SDK DEFAULT_* → IAgentConfig
- **分類**: **Mechanism** — SDK 型別契約，所有通訊模式必須實作
- **來源**: D2-R2, D2-R6

### C6: PluginHooks.commChannels + CommChannelRegistry (~40 LOC)

- **位置**: `packages/sdk/src/types/plugin.ts` (modify), `packages/core/src/infrastructure/` (NEW or modify)
- **新增**: `commChannels?: ICommChannel[]` in PluginHooks (array slot, consistent with vedanaSensors[], tools[], listeners[])
- **Registry**: CommChannelRegistry（平行 VedanaRegistry pattern）
- **分類**: commChannels 為 array slot = **Mechanism**（一致既有 array slot 先例）
- **來源**: D2-R2

### C7: PipelineChannel Plugin (~80 LOC)

- **位置**: `packages/core/src/plugins/pipeline-channel/` (NEW)
- **實作**: ICommChannel with `capabilities: ['messaging']`, `topology: 'pipeline'`
- **傳輸**: 既有 IPC (Unix Domain Socket / Named Pipe) — 無新傳輸依賴
- **行為**: Sequential message passing A->B->C, blocking send with timeout
- **Plan37 唯一通訊實作**: Hub-and-Spoke / Mesh channels 於後續 Plans
- **MCP 演進路徑**: Plan37 PipelineChannel 使用 IPC 傳輸（最簡 MVP）；Plan38 McpHubChannel 使用 IMcpTransport 實作 openstarry-channel 雙角色架構。ICommChannel 抽象確保遷移無痛。
- **分類**: PipelineChannel 為 Plan37 唯一實作 = **Policy**（scope 決策）
- **來源**: D2-R1, D2-R2, Master 補充指示

### C15: IMcpTransport Interface Definition (~30 LOC) [NEW — Master Directive]

- **位置**: `packages/sdk/src/types/mcp-transport.ts` (NEW)
- **背景**: Master 確認「所有節點皆 MCP Server + Client」為 Phase 6 通訊基礎。Plan37 定義介面，Plan38 實作 openstarry-channel。
- **介面**:
```typescript
/**
 * IMcpTransport — MCP 雙角色傳輸介面
 *
 * 每個 OpenStarry 節點（Agent CLI session / openstarry-channel）
 * 同時作為 MCP Server 和 MCP Client。
 * 此介面定義雙角色的連接管理契約。
 *
 * Plan37: 介面定義 only
 * Plan38: McpHubTransport 實作 (openstarry-channel)
 */
interface IMcpTransport {
  /** 本節點的 MCP Server 資訊（供其他節點連接） */
  readonly serverInfo: McpServerEndpoint;

  /** 本節點作為 Client 連接到的 MCP Server 列表 */
  readonly connectedServers: ReadonlyArray<McpClientConnection>;

  /** 向 Hub (openstarry-channel) 註冊自己的 Server 端點 */
  registerWithHub(hubEndpoint: McpServerEndpoint): Promise<void>;

  /** 從 Hub 取消註冊 */
  deregisterFromHub(): Promise<void>;

  /** 連接到另一個節點的 MCP Server (Hub→CLI 或 CLI→Hub) */
  connectTo(endpoint: McpServerEndpoint): Promise<McpClientConnection>;

  /** 斷開與指定節點的連接 */
  disconnectFrom(endpointId: string): Promise<void>;
}

interface McpServerEndpoint {
  agentId: string;
  transport: 'stdio' | 'http' | 'sse';
  url?: string;          // http/sse transport
  command?: string;       // stdio transport
  args?: string[];
  exposedTools: string[]; // D2-R5 capability model
}

interface McpClientConnection {
  endpointId: string;
  status: 'connecting' | 'connected' | 'disconnected' | 'error';
  capabilities: string[];  // server's declared capabilities
}
```
- **與 ICommChannel 的關係**: `McpHubChannel implements ICommChannel` 內部使用 `IMcpTransport` 管理連接。ICommChannel 是上層抽象（message semantics），IMcpTransport 是下層傳輸（connection management）。
- **與現有 mcp-server / mcp-client 的關係**: IMcpTransport 統一管理雙角色。現有 mcp-server 插件 = Server 端，mcp-client 插件 = Client 端。IMcpTransport 將兩者整合為單一介面。
- **分類**: **Mechanism** — SDK 型別契約，Plan38 openstarry-channel 必須實作
- **來源**: Master 補充指示 (2026-03-24), D2-R3 修訂

### C8: Process Tree + IDaemonControlPlane Extension (~100 LOC)

- **位置**: `apps/runner/src/daemon/` (modify)
- **新增 AgentRegistryEntry 欄位**: `parentAgentId?: string`
- **新增方法**:
  - `spawnChildAgent(parentId, childConfig)`: 生成子 Agent
  - `getProcessTree()`: 取得完整代理樹
  - `getChildAgents(parentId)`: 取得指定 parent 的子 Agents
- **Permission lattice** (MECHANISM, spawn 時強制驗證):
  1. `child.allowedPaths ⊆ parent.allowedPaths` (Tenet #5 延伸, D3-R3)
  2. `child.maxTokenBudget ≤ parent.remainingBudget` (D3-R3)
  3. `child.cumulativeDeltaCeiling ≤ parent.remainingCeiling` (D3-R3)
- **Token budget**: child.maxTokenBudget <= parent.remainingBudget (MECHANISM)
- **BABBAGE BCT**: spawnAgent() 仍可正常運作（parentAgentId defaults to undefined = root agent）。零子代 CompositeAgent 弱雙模擬獨立 Agent [D3-R3]
- **Supervisor Strategy**: one-for-one default (POLICY, configurable per parent Agent) [D2-R8]
- **Graceful Shutdown Protocol**:
```
RUNNING -> (terminate signal) -> DRAINING
  - 停止接受新請求
  - 完成進行中請求 (best effort)
  - 通知通訊夥伴 via agent:leaving
  - DRAINING Agent 不可生成新子 Agent (GUARDIAN, 防止 drain evasion)
  -> grace_period 到期 OR 所有進行中完成 -> TERMINATED
  -> grace_period 到期 AND 仍有進行中 -> FORCE_KILL
```
- **Grace period**: SDK `DEFAULT_AGENT_GRACE_PERIOD_MS = 30000` (POLICY), hard maximum 300000ms (MECHANISM ceiling) [D2-R8]
- **來源**: D2-R5, D2-R8, D3-R3, D4-R1

### Wave 2 架構邊界 (D2-R4: Core adds ZERO new IPC)

| Layer | Multi-Agent 職責 | 修改 |
|-------|-----------------|------|
| **Core** | 無。Per-Agent EventBus, per-Agent StateManager, per-Agent ExecutionLoop | ZERO change |
| **SDK Types** | ICommChannel, CommMessage, CommCapability, **IMcpTransport** 定義。PluginHooks.commChannels? | 新增型別定義 |
| **Plugin** | PipelineChannel (Plan37), 未來 HubSpokeChannel / MeshChannel | 新增 plugins |
| **Daemon** | MessageRouter, ProcessTree, TokenPool, IDaemonControlPlane extensions | Daemon 擴展 |

---

## Wave 3 (P1): AC-6 + Capability Model (~200 LOC)

### C9: Front-Five Typed IListener Subinterfaces (~80 LOC)

- **位置**: `packages/sdk/src/types/listener.ts` (modify)
- **新增**:
```typescript
type SenseType = 'caksur' | 'srotra' | 'ghana' | 'jihva' | 'kaya' | 'mano';

interface ITypedListener extends IListener {
  readonly senseType: SenseType;
}

// Backward-compatible union
type AnyListener = ITypedListener | IListener;
```
- **五個具名子介面**: IVisualListener (caksur), IAuditoryListener (srotra), IOlfactoryListener (ghana), IGustatoryListener (jihva), ITactileListener (kaya)
- **更新**: `PluginHooks.listeners` 型別從 `IListener[]` 改為 `(ITypedListener | IListener)[]`（backward-compatible union）
- **分類**: senseType discriminant = **Mechanism** (型別系統)
- **BABBAGE BCT**: 既有 IListener 實作增加 subtype marker 但 runtime 行為不變 — trivially satisfied [D3-R2]
- **Tenet #6 AC-6**: 直接解決前五識缺乏類型區分的偏差
- **來源**: D3-R2

### C10: Communication Capability in agent.json (~60 LOC)

- **位置**: `packages/shared/src/utils/config-schema.ts` (modify), `packages/sdk/src/types/agent.ts` (modify)
- **新增 IAgentConfig.communication**:
```typescript
interface CommunicationConfig {
  canSendTo?: string[];        // 可發送目標 agentId 列表
  canReceiveFrom?: string[];   // 可接收來源 agentId 列表
  exposedTools?: string[];     // 向其他 Agent 暴露的工具白名單
  maxMessageSize?: number;     // 訊息大小上限 (bytes)
  eventSubscriptions?: string[]; // EventBridge 事件訂閱白名單
}
```
- **Zod schema**: 必須包含所有欄位 (PROC-SPEC-3, Rule #21) — 避免重蹈 BUG-2 覆轍
- **Default**: 零能力（無 communication section = 無多代理通訊）— 最小權限原則 (Tenet #5 延伸)
- **SDK safe defaults**: `canSendTo: ['orchestrator']`, `canReceiveFrom: ['orchestrator']`
- **來源**: D2-R5

### C11: Daemon Capability Enforcement (~60 LOC)

- **位置**: `apps/runner/src/daemon/message-router.ts` (NEW)
- **行為**: 每則訊息經由 Daemon MessageRouter 路由時，驗證：
  1. sender 的 `canSendTo` 是否包含 receiver
  2. receiver 的 `canReceiveFrom` 是否包含 sender
- **失敗**: Reject with error (**fail-closed**, Rule #29) — 能力檢查為 enforcement 元件
- **子代能力子集** (MECHANISM, spawn 時驗證):
  - `C_child.canSendTo ⊆ C_parent.canSendTo`
  - `C_child.exposedTools ⊆ C_parent.exposedTools`
- **能力撤銷**: Daemon 可立即撤銷任何 Agent 的通訊能力（進行中訊息可能仍被送達，但不接受新訊息）
- **粒度**: Agent-level for MVP — tool-level capability filtering deferred to Plan39+ (GUARDIAN recommendation)
- **分類**: Capability checking = **Mechanism**（不可繞過的安全執行點）；Permission lists = **Policy**（可配置）
- **來源**: D2-R5

---

## Wave 4 (P2, Conditional): EventBridge + L2 Registry (~150 LOC)

> W4 為條件性交付。若 Plan37 時程無法以充足品質完成，延後至 Plan38 可接受 [D4-R1]。

### C12: EventBridge Daemon Service (~80 LOC)

- **位置**: `apps/runner/src/daemon/event-bridge.ts` (NEW)
- **訂閱表**: `Map<eventType, Set<agentId>>`
- **過濾**: Per-agent `eventSubscriptions` whitelist (from agent.json communication)
- **新增 CoordinationMessage 類型**: `agent:joining` (with capabilities), `agent:leaving` (with grace period), `agent:status_changed` [D2-R7]
- **行為**: Agent 的 comm plugin 選擇性轉發事件至 EventBridge via IPC
- **Fail-open**: EventBridge 故障不影響 per-Agent EventBus（觀測型元件, Rule #29）
- **分類**: EventBridge 位於 Daemon = **Mechanism** (Tenet #7 boundary)；事件類型白名單過濾 = **Policy**
- **來源**: D2-R7

### C13: L2 Global ServiceRegistry (~50 LOC)

- **位置**: `apps/runner/src/daemon/global-service-registry.ts` (NEW)
- **查詢**: L1 local first -> L2 global via IPC (DNS analogy: L1 = /etc/hosts, L2 = DNS resolver)
- **註冊**: Agent 啟動時 comm plugin 透過 `service:register` IPC 呼叫向 Daemon 報告本地服務
- **自動清理**: Service deregistration on agent TERMINATED (automatic via DRAINING -> TERMINATED lifecycle)
- **與 Blackboard 分離**: 獨立於未來 Blackboard/Alaya 實作 — 統一評估延至 Plan39+ [D2-R7]
- **來源**: D2-R7

### C14: Graceful Shutdown Protocol (Daemon 層) (~20 LOC)

- **位置**: `apps/runner/src/daemon/` (modify)
- **狀態機**: `RUNNING -> DRAINING -> TERMINATED` (with FORCE_KILL after grace period)
- **Default grace period**: 30000ms (SDK), max 300000ms (mechanism ceiling)
- **Orphan reparenting**: 子 Agent 的 parent 終止時，reparent 至 Daemon (PID 1 analogy) [D2-R8]
- **來源**: D2-R8

---

## 依賴與約束

| 依賴 | 類型 | 細節 |
|------|------|------|
| SDK inferRiskCategory() | Existing | C2 用於 per-tool riskCategory 推斷。已存在於 packages/sdk/src/types/gear-arbiter.ts L258-270 |
| EventBus | Existing | C2 用於事件發送。無須修改 |
| AuditTrailWriter | Modification | 須訂閱 `audit:tool_audited` 事件類型。~5 LOC |
| SIGNATURE.json schema | Consideration | Per-tool 事件產生額外 audit entries。Hash chain (Rule #30) 持續運作 — 新 entries 附加至 chain |
| Zod schema (config-schema.ts) | Modification | C10 communication 欄位必須加入 Zod schema (PROC-SPEC-3, Rule #21)。避免 BUG-2 重演 |
| W2-R1 rerun | Post-fix | BUG-3 修復使 W2 v2 結果失效。W2-R1 須在 Plan37 交付後以 >= 70B LLM 重新執行 |
| mcp-server / mcp-client plugins | Reference | C15 IMcpTransport 定義參考現有 MCP 雙角色插件架構。Plan37 不修改現有 MCP 插件 |

**Hard constraints**:
- Fix 不得引入新 policy 常量至 Core (Tenet #7, MR-6)
- RISK_ORDER 映射位於 gear-arbiter-static 為 **mechanism**（固定排序，不可配置）
- Per-tool events 為 fail-open (Rule #29) — 發送失敗不阻斷工具執行
- MAX_CUMULATIVE_POSITIVE = 0.05 為 Master-directed (Rule #26) — 不可協商
- MAX_CUMULATIVE_NEGATIVE = -0.05 為 provisional — 依 Cycle 03-2 審查
- Core adds ZERO new IPC primitives for multi-agent (D2-R4, Tenet #7)
- Default zero capability for communication (Tenet #5 extension, D2-R5)
- Child capability subset_of parent capability (MECHANISM, D2-R5, D3-R3)
- openstarry-channel = Required 核心基礎設施 (Master directive); Claude Code Channels = optional (D2-R3 修訂)
- IMcpTransport 為 Plan38 openstarry-channel 的介面預備；Plan37 僅定義型別，不實作

---

## 驗證要求

| 驗證者 | 範圍 | 方法 |
|--------|------|------|
| TURING (#17) | 每 Wave 三欄驗證 | Three-column verification format (02-11 standard) |
| BABBAGE (#9) | C8 Process Tree backward compat | BCT: zero-child CompositeAgent 弱雙模擬獨立 Agent |
| GUARDIAN (#11) | C10/C11 capability model | Security review: capability enforcement, zero-capability default, child lattice |
| WIENER (#12) | C1/C2 BUG-3 修復後 delta 行為 | Control analysis: worst-risk delta correctness, post-fix sigma projection |
| SUSSMAN (#22) | C5/C6 ICommChannel 設計 | Architecture review: 三層配置合規, capability model extensibility |
| KNUTH (#21) | C2 audit 覆蓋率 | Algorithm analysis: O(k) per-tool audit growth, SC-3 achievability post-fix |

---

## 十大宣言合規檢查

| # | 宣言 | Plan37 影響 | 評估 |
|---|------|-----------|------|
| 1 | 代理人即操作系統進程 (Agent as OS Process) | Process Tree 擴展 — 子 Agent 為獨立 OS process | COMPLIANT |
| 2 | 一切皆插件 (Everything is a Plugin) | PipelineChannel, EventBridge 均為 plugins; 通訊通道經 commChannels hook 註冊 | COMPLIANT |
| 3 | 五蘊聚合架構 (Five Aggregates Architecture) | Per-agent 五蘊保留；每個 Agent 為完整認知單元 [D3-R1] | COMPLIANT |
| 4 | 目錄結構即協議 (Directory as Protocol) | 無直接變更 | COMPLIANT |
| 5 | 目錄結構即權限 (Directory as Permission) | Permission lattice 延伸至階層式 Agents: child.allowedPaths ⊆ parent.allowedPaths | COMPLIANT (strengthened) |
| 6 | 八識俱轉與功能性代理 (Concurrent Consciousness and Functional Agency) | AC-6 前五識類型化直接推進合規；D3-R1 確立多代理八識分配 | Advancing (AC-6) |
| 7 | 微內核與絕對純淨 (Microkernel & Absolute Purity) | Core adds ZERO new IPC (D2-R4)。所有通訊邏輯在 Daemon + Plugin | COMPLIANT |
| 8 | 控制理論閉環模型 (Control-Theoretic Loop Model) | BUG-3 修復恢復 4/4 riskCategory 閉環。Per-agent 控制迴圈獨立 | COMPLIANT (with remediation via W1) |
| 9 | 可插拔的記憶策略 (Pluggable Context Strategy) | ICommChannel array slot 啟用無限通訊模式擴展 | COMPLIANT |
| 10 | 碎形社群結構 (Fractal Social Structure) | ICompositeAgent + Process Tree 開始解決 Tenet #10 | Advancing toward COMPLIANT |

**合規計數 (projected)**: 8 COMPLIANT / 0 CONDITIONAL / 0 NON-COMPLIANT / 2 Advancing
- Tenet #6: CONDITIONAL -> Advancing (AC-6 implementation)
- Tenet #10: NON-COMPLIANT -> Advancing (Process Tree + ICompositeAgent pattern)

---

## Baseline Rules 檢查

| Rule # | 規則 | Plan37 合規狀態 |
|--------|------|----------------|
| 1 | Cognitive sequence 6-state | Per-agent 保留 |
| 2 | Cetana layered treatment A/B/C | 無影響 |
| 3 | Conditionalized temporal logic | 無影響 |
| 4 | Twelve Nidanas three-tier taxonomy | 無影響 |
| 5 | Vedana/trsna strict distinction | 無影響 |
| 6 | Skandha soft constraint default L2 | 無影響 |
| 7 | Calibration sequence C1->C2 four-phase | 無影響 |
| 8 | TOST primary stability test epsilon=0.03 | W2-R1 post-fix 適用 |
| 9 | BABBAGE continuity test | C8 Process Tree BCT required; C9 AC-6 trivially satisfied |
| 10 | SUSSMAN three-layer config | ICommChannel: IAgentConfig -> SDK DEFAULT_* -> Core zero. Communication config follows same pattern |
| 11 | Three-tier criticality | PipelineChannel = optional-no-effect; Capability enforcement = Required; EventBridge = optional-degraded |
| 12 | Plan34 三文件職責分離 | 無影響 |
| 13 | vedanaFn wiring BABBAGE test | 無影響 |
| 14 | guide-persistent 哲學文件要求 | 無影響 |
| 15 | 三層文件架構 | Doc 57 = Engineering; Doc 19/09 updates span Engineering + Philosophy |
| 16 | PROC 規則 | PROC-SPEC-3 (Rule #21) enforced on communication schema |
| 17 | Context-summary SDK defaults | 無影響 |
| 18 | vedanaFn uses SDK classifyVedana() | 無影響 |
| 19 | Guide-persistent multi-agent isolation | Per-agent guide isolation maintained |
| 20 | Tenet #9 infinite extensibility | Array slot enables unlimited communication modes |
| 21 | PROC-SPEC-3 schema-config alignment | C10 communication 欄位必須同步 Zod schema |
| 22 | Micro-positive delta values | 不變；worst-risk 修復僅更正 riskCategory 選擇邏輯 |
| 23 | Three-tier Go/No-Go precondition | W2-R1 適用: sigma>0 AND n_eff>=10 |
| 24 | VedanaSensor fail-open error model | 無影響 |
| 25 | VedanaSensor aggregation | 無影響 |
| 26 | Cumulative positive delta clamp = +0.05 | C3 驗證並修正 |
| 27 | T3 gate singular slot | 無影響 |
| 28 | timeoutAction='deny' is mechanism | 一致模式：capability checking 為 mechanism (fail-closed) |
| 29 | Observation fail-open; enforcement fail-closed | C2 audit events fail-open; C11 capability enforcement fail-closed; must-invoke added as third category (D1-R4) |
| 30 | Audit trail SHA-256 hash chain | Per-tool events 附加至 chain，無 schema 變更 |
| 31 | Eight consciousnesses multi-agent allocation (NEW) | D3-R1: Front-five + Mano per-agent; Manas per-agent with hierarchical override; Alaya shared as samvriti-satya |
| 32 | AC-6 before AC-7 ordering (NEW) | D3-R2: AC-6 (~150-200 LOC, Plan37) BEFORE AC-7 (~800+ LOC, Plan38+) |
| 33 | ICompositeAgent permission lattice (NEW) | D3-R3: C8 Process Tree enforces child ⊆ parent; reserve_ratio SDK default 0.3; depth limit 3 |
| 34 | W2-R1 enhanced success criteria (NEW) | D4-R4: SC-1~SC-6, fresh recalibration, BUG-3 fix precondition |
| 35 | ICommChannel unified interface (NEW from D2-R2) | All multi-agent channels implement ICommChannel with capability declaration |
| 36 | Zero Core modification for multi-agent (NEW from D2-R4) | All communication in Daemon + Plugin; Core unchanged |
| 37 | seL4-inspired capability model (NEW from D2-R5) | Capability checking = mechanism; permission rules = policy; default zero capability; child ⊆ parent |
| 38 | Five-layer failure isolation (NEW from D2-R8) | L1-L5 model; L4 Supervisor in Plan37; L2/L3/L5 in Plan38 |
| 39 | Graceful shutdown protocol (NEW from D2-R8) | RUNNING -> DRAINING -> TERMINATED; force-kill mechanism; grace period 30s default / 300s max |

---

## DOC SOP 同步要求 (PROC-DOC-1)

Plan37 交付時必須同步更新以下四文件：

1. **README.md + README_TW.md**: 新增 Multi-agent quick start section (ICommChannel, PipelineChannel, agent.json communication)
2. **Iteration_Log_Cycle03.md**: Plan37 entry (W1-W4 scope, LOC, key decisions)
3. **Roadmap.md**: Plan37 complete + v0.37.0-alpha milestone
4. **openstarry_doc/README.md**: Doc 57 (Multi-Agent Communication Interface Specification) 新增 + Doc 19/09 更新標記

### 文件更新計畫 (D4-R2)

| Priority | 文件 | Action | Plan Target |
|----------|------|--------|-------------|
| **P0** | Doc 09 (Communication Protocol Strategy) | 新增 ICommChannel 介面定義, capability model, pipeline/hub-spoke/mesh taxonomy | Plan37 |
| **P0** | Doc 19 (Agent Coordination Layer) | 60% 重寫: multi-agent Registry, MessageRouter, capability discovery, referencing Doc 09 | Plan37 |
| **P0** | Doc 57 (NEW: Multi-Agent Communication Interface Spec) | ICommChannel type, PipelineChannel plugin spec, capability model, Process Tree interface | Plan37 |
| P1 | Doc 13 (Orchestrator Daemon) | Multi-agent lifecycle management | Plan38 |
| P1 | NEW: Multi-Agent Security Model | Capability model, trust boundaries, attack surface analysis | Plan38 |
| P2 | Doc 06, 22, NEW Unified Comm, NEW State Consistency | 後續 Plans | Plan39+ |
| P3 | Doc 07 (Management Zone) | Cross-agent event chain extensions | Plan40+ |

---

## 新增 / 修改 Baseline Rules 匯總

### 修改

| Rule # | 修改內容 | 來源 |
|--------|---------|------|
| 29 | 擴展為三類安全保證: fail-open + fail-closed + **must-invoke** (CTL AF phi) | D1-R4 |

### 新增

| Rule # | 規則 | 來源 |
|--------|------|------|
| 31 | 八識多代理分配: Front-five + Mano per-agent; Manas per-agent w/ hierarchical override; Alaya shared (samvriti-satya) | D3-R1 |
| 32 | AC-6 BEFORE AC-7 ordering | D3-R2 |
| 33 | ICompositeAgent permission lattice + reserve_ratio=0.3 + depth=3 + bisimulation | D3-R3 |
| 34 | W2-R1 enhanced SC-1~SC-6 + fresh recalibration + BUG-3 precondition | D4-R4 |
| 35 | ICommChannel unified communication interface with capability declaration | D2-R2 |
| 36 | Zero Core modification for multi-agent (Tenet #7 Phase 6 guarantee) | D2-R4 |
| 37 | seL4-inspired capability model: mechanism checking, policy rules, zero default, child ⊆ parent | D2-R5 |
| 38 | Five-layer failure isolation: L1 Process -> L2 Circuit Breaker -> L3 Bulkhead -> L4 Supervisor -> L5 Timeout | D2-R8 |
| 39 | Graceful shutdown: DRAINING -> TERMINATED + force-kill; 30s default / 300s max | D2-R8 |

### Provisional

| 規則 | 來源 |
|------|------|
| Cumulative negative floor = -0.05 (symmetric with Rule #26) | D1-R3, subject to 03-2 review |

---

## Decision 溯源表

| Plan37 Component | R3 Decision(s) | Key Agents |
|-----------------|----------------|------------|
| C1 Worst-Risk | D1-R1 | TURING, KNUTH, WIENER, GUARDIAN, BABBAGE |
| C2 Per-Tool Audit | D1-R1, D1-R4 | TURING, KNUTH, GUARDIAN, KERNEL |
| C3 Cumulative Clamp | D1-R3 | WIENER, GUARDIAN, BABBAGE, KNUTH |
| C4 Must-Invoke Verify | D1-R4 | GUARDIAN, BABBAGE, WIENER, KERNEL |
| C5 ICommChannel | D2-R2, D2-R6 | SUSSMAN, TANENBAUM, ATHENA, BABBAGE |
| C6 CommChannelRegistry | D2-R2 | SUSSMAN, DARWIN |
| C7 PipelineChannel | D2-R1, D2-R2 | MESH, TANENBAUM, ATHENA, ARCHIMEDES |
| C15 IMcpTransport | Master directive, D2-R3 rev | ATHENA, MESH, TANENBAUM, KERNEL |
| C8 Process Tree | D2-R5, D2-R8, D3-R3, D4-R1 | KERNEL, LEIBNIZ, GUARDIAN, BABBAGE, PASCAL |
| C9 AC-6 Front-Five | D3-R2 | ASANGA, LINNAEUS, BABBAGE, ARCHIMEDES |
| C10 Communication Config | D2-R5 | TANENBAUM, GUARDIAN, BABBAGE, LEIBNIZ |
| C11 Capability Enforcement | D2-R5 | GUARDIAN, BABBAGE, TANENBAUM |
| C12 EventBridge | D2-R7 | DARWIN, HERACLITUS, KERNEL, RUSSELL |
| C13 L2 ServiceRegistry | D2-R7 | DARWIN, KERNEL, RUSSELL |
| C14 Graceful Shutdown | D2-R8 | HERACLITUS, PENROSE, KERNEL, GUARDIAN, WIENER |

---

*Plan37 規格書 — Phase 6 Multi-Agent Foundation*
*Cycle 03-1 R4 Engineering Deliverable*
*ARCHIMEDES (#16), with research team input*
*2026-03-24*
