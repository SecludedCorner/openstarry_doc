<!-- QUARANTINE NOTICE 2026-06-12 -->
> **⚠ 隔離標記（2026-06-12 蒸餾掃描）**：本文所述與現行實作不符，或描述未曾建造的系統——**請勿作為規格使用**。API 最高權威＝SDK 型別檔（`packages/sdk/src/`），行為權威＝測試套件。分類依據見 [DISTILLATION_LIST.md](../DISTILLATION_LIST.md)。
> 判定理由（掃描原文）：337 lines specifying the systemd-like daemon (process supervision, infra-plugin hosting, management API) that was never built; apps/ contains only runner and channel.
>
> **[v0.59.7 更新 — 混合文件，部分已落地]** 本文為**混合**：systemd 式管理 daemon（process supervision、infra-plugin hosting、REST 管理 API）**仍為虛構**；但下方 **Plan37 Process Tree 一節所述機制已落地且有 e2e 證明**——`agent.spawnChild`／`agent.processTree`／父子樹邊／SEC-003 拒絕／父亡子收（`apps/runner/__tests__/e2e/daemon-process-tree.e2e.test.ts`），CLI `openstarry ps --tree`（v0.59.7，唯讀走既有 RPC，見本節「CLI 觀測」小節）。讀此文時：Process Tree + ps --tree 為真，其餘編排器/管理面仍依隔離牌處理。

# 12. 運行時核心：編排器守護進程 (Orchestrator Daemon)

本文件詳細闡述了 `Orchestrator Daemon` 的設計，這是在我們最終架構中，負責管理所有代理人 OS 行程生命週期的關鍵後台服務。

## 核心定位：代理人系統的「init/systemd」

`Orchestrator Daemon` 是一個**常駐後台的、與代理人業務邏輯無關**的系統級服務。它不參與任何 LLM 的思考或任務的執行，其職責非常純粹和底層。

*   **類比：** 它就像 Linux 系統中的 `systemd` 或 `init` 進程，或者容器化世界中的 `Docker Compose` 或 `Kubernetes Kubelet`。
*   **職責：**
    1.  **進程生命週期管理：** 負責啟動、停止、監控和重啟所有「代理人核心」的獨立操作系統行程。
    2.  **基礎設施插件託管：** 負責加載 `infrastructure` 類型的插件，為代理人提供本地共享服務（如消息總線、狀態數據庫），確保環境的開箱即用與可替換性。
    3.  **資源管理：** 監控每個代理人行程的資源消耗（CPU、內存），並可以根據策略進行管理（例如，終止消耗過多的進程）。
    4.  **提供管理接口：** 暴露一組內部 API，供 `AgentManagerTool` 或外部管理工具調用，以程序化的方式管理代理人實例。

---

## 工作流程

### 1. 系統啟動

*   系統管理員在服務器上執行 `openstarry-daemon start` 命令，啟動守護進程。
*   `Orchestrator Daemon` 啟動後，它做的第一件事就是根據其主配置文件，啟動並開始監護**「主代理人 (Master Agent)」**的 OS 行程。

### 2. 代理人創建 (由 `AgentManagerTool` 觸發)

1.  「主代理人」的 `AgentManagerTool` 被調用，意圖啟動一個新的工作代理人。
2.  `AgentManagerTool` 的 `execute` 方法不再是自己創建進程，而是向本地運行的 `Orchestrator Daemon` 發起一個 API 請求。
    *   **請求示例：** `POST http://localhost:5050/agents`
    *   **請求體：**
        ```json
        {
          "agent_id": "data_worker_007",
          "template_id": "DataAnalystAgent_v1"
        }
        ```
3.  `Orchestrator Daemon` 接收到 API 請求後，執行以下操作：
    *   向「代理人設計層」查詢 `DataAnalystAgent_v1` 模板的配置。
    *   準備好啟動新代理人行程所需的環境變量和命令行參數。
    *   使用 `child_process.spawn()` 或類似的系統調用，創建一個新的、獨立的 OS 行程。
    *   將新行程的 `PID` 和 `agent_id` 記錄在自己的監控列表中。
    *   開始監控新行程的健康狀況（例如，通過心跳或 `PID` 是否存在）。

### 3. 代理人銷毀

*   當 `AgentManagerTool` 調用 `agent:stop(agent_id)` 時，它會向 Daemon 發起 `DELETE /agents/{agent_id}` 的 API 請求。
*   `Orchestrator Daemon` 收到請求後，負責安全地終止對應的 OS 行程並清理相關資源。

---

---

## 協調層 Daemon 規格 (Coordination Daemon Specification)

> [Cycle 02-2 Q4 整合] Master 裁定：「必然是獨立 daemon（獨立進程），現在就必須安排。」

### 最小 Daemon 架構

```
packages/coordination/
├── src/
│   ├── daemon.ts              — Daemon 進程入口
│   ├── ipc/
│   │   ├── protocol.ts        — IPC 訊息協議
│   │   └── server.ts          — IPC 伺服器
│   ├── registry/
│   │   ├── plugin-registry.ts — Plugin 註冊表
│   │   └── agent-registry.ts  — Agent 註冊表
│   └── health/
│       └── monitor.ts         — 健康檢查
├── package.json
└── tsconfig.json
```

### 核心介面

```typescript
/**
 * CoordinationDaemon — 協調層守護進程。
 * 獨立於 AgentCore 運行，管理跨 Agent 資源。
 *
 * 佛學映射: 阿賴耶識的「能藏」面向 — 能力註冊、Plugin 解析。
 */
interface CoordinationDaemon {
  readonly pluginRegistry: PluginRegistryService;
  readonly agentRegistry: AgentRegistryService;
  readonly ipc: IPCService;
  start(): Promise<void>;
  stop(): Promise<void>;
  healthCheck(): CoordinationHealthReport;
}

/**
 * PluginRegistryService — 跨 Agent 的 Plugin 註冊表。
 * 支援按蘊查詢 (listBySkandha)。
 */
interface PluginRegistryService {
  list(): PluginRegistryEntry[];
  listBySkandha(skandha: Skandha): PluginRegistryEntry[];
  checkTrust(pluginName: string): PluginTrustLevel;
}

/**
 * AgentRegistryService — Agent 註冊表。
 * 每個 Agent 攜帶五蘊完備性報告。
 */
interface AgentRegistryService {
  list(): AgentRegistryEntry[];
  get(agentId: string): AgentRegistryEntry | undefined;
}

interface AgentRegistryEntry {
  agentId: string;
  name: string;
  skandhaCompleteness: SkandhaCompletenessReport;
  mode: 'normal' | 'developer';
  status: 'running' | 'stopped' | 'error';
  startedAt: number;
}
```

### IPC 訊息協議

```typescript
/**
 * CoordinationMessage — 協調層 IPC 訊息型別聯合。
 * 使用 Named Pipes (Windows) / Unix Domain Sockets (Linux/macOS)。
 */
type CoordinationMessage =
  | { type: 'agent:register'; agentId: string; config: unknown }
  | { type: 'agent:deregister'; agentId: string }
  | { type: 'agent:health'; agentId: string; report: unknown }
  | { type: 'plugin:query'; skandha?: Skandha }
  | { type: 'plugin:trust_check'; pluginName: string }
  | { type: 'coordination:health_check' };
```

### 實作路線圖

| 階段 | 內容 | 目標 Plan |
|------|------|----------|
| Phase 1 | 設計文件 + Daemon skeleton | Plan-AC MVP |
| Phase 2 | Plugin Registry + IPC | Plan-AC |
| Phase 3 | Agent Registry + Health | Plan-AC |
| Phase 4 | Sila Engine (戒律引擎) | Plan29/Plan-AC |
| Phase 5 | 多機器協調 | Future |

---

## Plan37 擴展：Process Tree + Graceful Shutdown (v0.37.0-alpha)

> [Cycle 03-1/03-2] Plan37 將 Daemon 從單代理管理升級為多代理進程樹管理。

### Process Tree（進程樹）

Plan37 引入 `parentAgentId` 和 `spawnChildAgent` 機制，建立父子代理層級結構：

```typescript
interface ProcessTreeEntry {
  agentId: string;
  parentAgentId?: string;    // undefined = root agent
  children: string[];
  depth: number;             // 0 = root, max = COMPOSITE_AGENT_MAX_DEPTH (3)
  state: AgentProcessState;
}

type AgentProcessState = 'RUNNING' | 'DRAINING' | 'TERMINATED';
```

**spawn 流程**（8 步 fork+exec 模式）：
1. 驗證 parentAgentId 存在且 state = RUNNING
2. SEC-002: 驗證呼叫者 PID 匹配 parentAgentId
3. SEC-003: isPathSafe() 驗證 configPath
4. 驗證 depth < COMPOSITE_AGENT_MAX_DEPTH (3)
5. F-5: Permission lattice 驗證 (path ⊆ parent, budget <= remaining, ceiling <= current)
6. 分配 childAgentId
7. child_process.spawn() 建立新進程
8. 註冊至 Process Tree + AgentRegistry

**seL4 capability model**（Rule #33）：
- Default zero: 新代理能力集為空
- Child ⊆ Parent: 子代理能力不超過父代理
- Capability checking = mechanism; permission rules = policy

#### CLI 觀測：`openstarry ps --tree`（v0.59.7-alpha LANDED）

進程樹現可由 CLI 直接觀測——`ps --tree` 對每個 running daemon（`pidManager.listRunningAgents`）開 IPC、呼叫既有的 `agent.processTree` RPC，把回傳的 `ProcessTreeNode[]` 依層級縮排渲染（每層 2 空格，附 `agentId`／`pid`／`status`／`depth`）：

```
$ openstarry ps --tree
PROCESS TREE
root-agent (pid 100) [running] depth=0
  first-worker [root-agent-1] (pid 200) [running] gen=1 depth=1
    grandchild (pid 300) [draining] depth=2

1 daemon(s) queried
```

唯讀（不 spawn/kill）；因每個被 spawn 的子代理本身是獨立 daemon、會在自己的 registry 自報為 root，CLI 以 `collectChildIds` 把這類節點折疊到其父節點下，避免重複列印。實作：`apps/runner/src/commands/ps.ts`（`PsCommand.printTree` + 純函數 `toRenderTreeNodes`／`collectChildIds`／`renderProcessTreeLines`），測試 `apps/runner/__tests__/commands/ps-tree.test.ts`（10 測試）。

#### Spec Addendum A — agent 命名與世代序號（v0.59.8-alpha LANDED，Fractal Society Phase 1）

凍結型別 `AgentRegistryEntry` / `ChildAgentSpawnConfig` / SDK `DaemonSpawnChildInput`／`DaemonProcessTreeNode`／`DaemonChildAgentInfo` 經 **Spec Addendum A（2026-06-26，Master ratified）** 增量擴充（皆向後相容選填欄）：

- **`generation`（每父世代序號）**：每個父代理各自從 1 計數其子代理（父 A 的孩子＝A 的 #1、#2…），跨重啟持久化（`apps/runner/src/daemon/generation-counter.ts`，per-parent 檔於 `OPENSTARRY_HOME/generation/`，原子寫、fail-open+WARN）。數字小且有意義，避免單一全域計數器無限增大。
- **`name`（人類好記標籤）**：使用者可為子代理設一個方便的名稱（非唯一、可變、純顯示；省略＝沿用 agentId）。`ps --tree` 顯示為 `name [agentId]`。
- **`agentId` 可省略**：`spawnChild` 未給 id 時，daemon 自動產唯一 `<parentId>-<generation>`；給了 id 則**碰撞 fail-closed 拒收**——修掉一個現存 bug（`agentRegistry.set` 原本會靜默覆蓋同名子代理）。
- 接點：`handleSpawnChild`（`daemon-entry.ts`）；工具 `agent.spawnChild`（`@openstarry-plugin/agent-spawn` input 加 `name?`、`agentId` 改選填）。測試：`generation-counter.test.ts`（7）+ ps-tree 渲染（+3）+ 真 daemon e2e `daemon-naming.e2e.test.ts`（2：auto-gen+世代+name 入樹、同名拒收反證）。

> Tenet#10 分形社會結構「好好實現」於 v0.59.8-alpha 大幅落地（見下方各節 + CHANGELOG）：
> - **Phase 1 命名（Spec Addendum A）**：每父世代序號 + 人類 name + agentId auto-gen + 同名拒收。
> - **comm 傳輸層（Spec Addendum C）**：C/T1 點對點訊息、C/T2 叢集 pub/sub、C/T3 服務發現閉環、**C/T4 言語行為（request-response/broadcast）＋ pipeline 拓撲（A→B→C 來源路由中繼）**——皆跨 daemon、HMAC 簽章、有兩/三進程 e2e。
> - **supervisor 重啟策略**：one-for-one/all/rest-for-one，崩潰子代理自動重生（pid 輪詢偵測）。
> - **fork/branch（Spec Addendum B，2026-06-27 ratify）**：fork＝spawn＋父 session 快照注入子；能力仍 child⊆parent；memory/alaya 不繼承；branch＝同快照 N 子共享 forkOrigin。
>
> **仍為誠實未來**：跨主機 transport、N>2 gossip、mesh 全互聯、fork 的 merge/select、depth>3。設計檔：`share/engineering_delivery/20260626_fractal_society_design/`（DESIGN/PLAN/COMM_TRANSPORT/ADDENDUM_B_DRAFT）。

### Graceful Shutdown Protocol

Plan37 引入三態 Graceful Shutdown 協議：

```
RUNNING → DRAINING → TERMINATED
```

| 狀態 | 行為 |
|------|------|
| RUNNING | 正常運行，接受所有請求 |
| DRAINING | 拒絕新請求（spawn/send），等待進行中請求完成 |
| TERMINATED | 進程已終止，資源已清理 |

- **Grace period**: 30s default (policy) / 300s max (mechanism, = settling time bound per WIENER)
- **SEC-001**: spawn 在 DRAINING 狀態被拒絕（防止 drain-evasion 攻擊）
- **Cascade**: 父代理 termination 向下傳播至所有子代理

### openstarry-channel 生命週期管理 (Plan38)

Plan38 將引入 `openstarry-channel` 作為跨 Daemon 通訊 Hub：

- **Daemon 職責**: 管理 openstarry-channel 進程的生命週期（start/stop/monitor）
- **啟動順序**: Daemon Init → openstarry-channel Bootstrap → READY signal → Agent Spawn
- **READY timeout**: 30s default, 60s mechanism max
- **失敗降級**: kill → retry with backoff → degraded mode (local Pipeline only)

### MessageRouter（6 層驗證鏈）

Daemon 層的 `MessageRouter` 實作 6 層 fail-closed 防禦：

| Layer | Check | Fail Action |
|-------|-------|-------------|
| L1 | Sender registered? | Reject |
| L2 | Target registered? | Reject |
| L3 | traceDepth <= MAX? | Reject |
| L4 | Sender capability check | Reject |
| L5 | Target capability check | Reject |
| L6 | Message validation | Reject |

所有 Layer 均為 fail-closed（enforcement, Rule #29）。

**v0.59.8-alpha（C/T1）— MessageRouter 由「驗證層」升為「真實傳輸」**：原本 `validateMessage` 假設收發雙方同 daemon、且無 production caller（驗證層虛構）。在 1-daemon-1-agent 模型下拆為兩半：sender daemon 跑 `validateOutbound`（自身 `canSendTo`），receiver daemon 跑 `validateInbound`（`canReceiveFrom` ＋ replay ＋ freshness ＋ envelope）；遠端 sender 不在本地註冊，身份靠 **HMAC 簽章**認證（cluster key，Spec Addendum C-2）。承載傳輸＝新 `CommTransport`（一般化已 e2e 證明的 alaya `IpcRemotePeer`，runner 內）＋ `comm.deliver`/`comm.send`/`comm.inbox` daemon RPC；拒收記 `comm_denied` 審計。兩進程 e2e：`daemon-comm.e2e.test.ts`。

### EventBridge

跨代理事件橋接服務（Daemon plugin）：
- Fail-open 設計（observation, Rule #29）
- Self-delivery exclusion（代理不收到自己發送的事件）
- 支援 agent:* 和 system:* 事件命名空間

**v0.59.8-alpha（C/T2）— 補上缺席的投遞層**：原本 `setDeliveryFn` 從未被 production 呼叫＝事件算完即丟（fail-open stub）。現以 `CommTransport.deliverEvent`（簽章 `comm.event` RPC）接真。訂閱為**訂閱者主動式**：訂閱者 daemon 以簽章 `comm.subscribe` 在發布者 daemon 的 EventBridge 註冊自己；發布者 publish 時，deliverFn 跨 daemon 投遞給每個訂閱者。兩進程 e2e（訂閱→發布→收到；未訂類型不送、偽簽/偽訂閱 fail-closed 拒）。

### GlobalServiceRegistry

跨代理共享服務的全域註冊表（Daemon plugin）：
- 支援服務註冊、查詢、健康檢查
- 與 openstarry-channel 的 service discovery 協作

**v0.59.8-alpha（C/T3）— 發現迴圈閉合**：原本有 register/lookup 但無人用查得結果去真的對話（real 控制面、無 consumer 閉環）。新增簽章 `comm.register`/`comm.lookup`（對 registry hub）＋ `comm.registerOn`/`comm.findPeer` 控制面；provider 以服務名登記、consumer 以服務名發現 provider 的 agentId 後用 `comm.send` 對話。三 daemon e2e（hub＋provider＋consumer，無靜態 peer config）。

### C/T4 言語行為與拓撲（v0.59.8-alpha）

在已接真的傳輸上實作凍結型別 `CommPerformative`/`CommTopology`（不改型別）：
- **request-response**：`comm.request` 送 `performative:'request'` 並以 `correlationId` 在 `pendingRequests` 等相關回覆（逾時拒）；`comm.reply` 回送 `correlationId=請求 id`，receiver 端 commDeliver 解析相關回覆。
- **broadcast**：`comm.broadcast` 對多目標扇出，逐目標獨立 capability 檢查、單一失敗不中止其餘。
- **pipeline（A→B→C）**：來源路由中繼——發起者定 hop 序列，每個 daemon 收到 pipeline 訊息後中繼給下一跳（逐跳 capability+簽章；route/trail 走 metadata；traceDepth 上界限制長度）。中段拒收即斷鏈並記審計。

### Supervisor 重啟策略（v0.59.8-alpha）

`SupervisorStrategy` 原為型別+常數、daemon 無實作。現 daemon 監看自己生的子代理（`childSpawnConfigs` 保留重生配置、HMAC key 重啟時重注不落盤）：pid 輪詢偵測到子代理崩潰（process 死但 registry status 仍 `running`，即非優雅停止）→ 依 `selectRestartSet`（one-for-one／one-for-all／rest-for-one）重生，受 `maxRestarts` 上界。`agent.supervise` 工具。誠實邊界：pid 輪詢非健全 OS supervision API。

### Fork / Branch（Spec Addendum B，v0.59.8-alpha）

fork＝spawn 子 + 注入父 session 快照為子初始 session（沿用 `FileSessionPersistence`）。Addendum B 繼承策略：會話快照帶（D4-a）、能力仍 child⊆parent（D4-b，走 `validateSpawnConstraints`／SEC-003 不繞）、memory/alaya **不**帶（D4-c）。branch＝同快照 N 子，共享 `AgentRegistryEntry.forkOrigin`（一個分支組）。`agent.fork`/`agent.branch` 工具。merge/select（選優回填）＝誠實未來。

### ICommChannel 層接真（Doc 53，v0.59.8-alpha）

Doc 53 的 `ICommChannel` 凍結契約其 `commChannelRegistry` 自始由 plugin loader 填充、卻**從無 production 消費者**（send/onMessage 不做實事）。現補上：新插件 `comm-channel-p2p`（色蘊/rupa）提供真 point-to-point ICommChannel（'messaging'），`send` 走真 DAEMON_COMM 傳輸、`onMessage` 收真 inbound。daemon 開機時 `connect()` 所有註冊 channel，並於 `commDeliver` 把每筆 inbound 派發給 registry 內的 channel（`commChannelRegistry.list()` → duck-typed `deliverInbound` → onMessage handlers）——**registry 首次被消費**。控制面 `comm.channelList`/`comm.channelSend`/`comm.channelReceived`；兩進程 e2e。comm-pipeline／comm-proxy／CompositeChannel 維持為 in-process 組合 reference impl（誠實標記，非 cross-process）。

---

## 結論

引入 `Orchestrator Daemon` 是我們架構演進的最終形態。它完美地解決了多行程架構下的資源管理和生命週期監護問題，同時讓「主代理人」和 `AgentManagerTool` 的職責保持純粹——它們只負責**決策**（「我需要一個新代理人」），而將**執行**（「如何實際創建一個 OS 行程並監控它」）的髒活累活交給了這個專業的底層服務。

協調層 Daemon 的正式引入（Q4 裁定）進一步明確了此架構的方向——Daemon 不僅管理進程生命週期，還承擔跨 Agent 的 Plugin 註冊、信任檢查、五蘊完備性驗證等協調層職責，對應阿賴耶識的「能藏」功能。

Plan37 (v0.37.0-alpha) 進一步擴展 Daemon 為多代理進程樹管理者，引入 Process Tree、Graceful Shutdown、MessageRouter、EventBridge 和 GlobalServiceRegistry。Plan38 將在此基礎上建立 openstarry-channel 跨 Daemon 通訊 Hub。

---

## openstarry-channel 生命週期與 Agent 協調 (Plan38)

### 設計定位

`openstarry-channel` 是一個獨立的多代理通訊 Hub，以 `apps/channel/` 應用程式形式執行，負責跨 Daemon 實例的訊息路由、健康監控與協調。它補充了 Orchestrator Daemon 的 Process Tree 管理，形成完整的多代理通訊基礎設施。

### 核心責職

1. **Agent 生命週期監控** — 維護跨 Daemon 的 AgentRegistry，追蹤所有 Agent 的健康狀態
2. **訊息路由與轉發** — 實現 ICommChannel 實例間的雙向訊息傳遞
3. **健康監測與故障檢測** — 週期性心跳監控（10 秒間隔），3 次失敗自動標記為 TERMINATED
4. **優雅關閉** — 三態協議（RUNNING → DRAINING → TERMINATED）
5. **服務發現** — 通過 GlobalServiceRegistry 暴露跨代理服務

### AgentRegistry 與健康狀態

```typescript
interface AgentRegistryEntry {
  agentId: string;
  parentDaemonId?: string;        // 所屬 Daemon（若有）
  healthState: AgentHealthState;
  lastHeartbeat: number;
  tags: Record<string, string>;
}

type AgentHealthState = 'HEALTHY' | 'DEGRADED' | 'UNREACHABLE' | 'TERMINATED';
```

**狀態轉移**：
- HEALTHY: 定期收到心跳回應
- DEGRADED: 1-2 次失敗後降級（可自動恢復）
- UNREACHABLE: 2-3 次失敗後（待自動轉為 TERMINATED）
- TERMINATED: 3+ 次失敗或顯式關閉命令

### 心跳監控（10 秒間隔）

```
定時任務（每 10s 執行）:
  for each Agent:
    if last_heartbeat_age > 10s:
      send heartbeat ping
      if no pong within 5s:
        failed_count++
        if failed_count >= 3:
          mark as TERMINATED
```

### 優雅關閉協議

```
RUNNING → (stop signal) → DRAINING → (grace period: 30s default) → TERMINATED
  ↑                                                                     ↓
  └─ 新請求被拒絕，進行中請求完成 ─────────────────────────────────┘
```

**Daemon 層級聯動**：
- 當 channel 進入 DRAINING，向所有註冊的 Daemon 發送 DRAINING 信號
- Daemon 依序關閉子 Agent，最後關閉自身進程

### 6 層級 L1 工具

openstarry-channel 暴露 6 個 L1 工具（Orchestrator Daemon 可調用）：

| 工具 | 簽名 | 用途 |
|------|------|------|
| `agent:register` | register(agentId, parentDaemonId?, tags?) | 向 channel 註冊新 Agent |
| `agent:unregister` | unregister(agentId) | 取消註冊已終止的 Agent |
| `agent:heartbeat` | heartbeat(agentId, status) | 心跳訊號（HEALTHY/DEGRADED） |
| `routing:send` | send(fromAgentId, toAgentId, message) | 發送點對點訊息 |
| `routing:broadcast` | broadcast(sourceAgentId, message, excludeAgents?) | 廣播訊息（選擇性排除） |
| `service:discover` | discoverServices(agentId?, namespace?) | 服務發現與查詢 |

### 崩潰恢復（7 步）

當 channel 檢測到 Agent 或 Daemon 故障時：

1. **檢測**：心跳超時或顯式故障信號
2. **隔離**：立即標記為 UNREACHABLE，拒絕新訊息
3. **清理**：取消所有掛起的訊息（發送至死信佇列）
4. **通知**：向依賴方發送 `AGENT_FAILED` 事件
5. **級聯**：若為父 Agent，啟動子 Agent 自動終止流程
6. **日誌**：記錄故障事件到結構化日誌
7. **重試**：心跳恢復後（若 Agent 重啟）可重新加入

---

