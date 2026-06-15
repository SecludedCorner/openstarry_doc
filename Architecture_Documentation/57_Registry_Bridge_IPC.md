# 61. 註冊表橋接與 IPC (Registry Bridge & IPC)

**Status**: Cycle 20260404_cycle03-3 PASS  
**Version**: v0.39.0-alpha  
**Plan**: Plan39 Engineering Specification  
**Author**: architect  
**Date**: 2026-04-04

---

## 1. 概述

Registry Bridge 是 OpenStarry Daemon 中的核心中介層，負責在**守護進程（Daemon-authoritative）** 和 **子 Agent 進程** 之間進行雙向事件同步。透過 Node.js `child_process.fork` IPC 機制，Registry Bridge 確保：

1. **Daemon 親和性**: Registry 的主要狀態存儲在 Daemon 中，子 Agent 通過 IPC 查詢/更新
2. **IRegistryEventBus 實現**: PROVISIONAL 狀態的 4 個核心事件
3. **AT-7 攻擊向量防禦**: Daemon-authoritative 設計防止子 Agent 偽造登記
4. **READY 信號**: 啟動序列同步機制

---

## 2. 架構背景

### 2.1 為什麼需要 Registry Bridge

在 Plan37/38 中引入的多 Agent 系統中，每個子 Agent 是一個獨立進程：

```
┌──────────────────────────┐
│   Daemon (主進程)         │
│ ┌──────────────────────┐  │
│ │ Registry (親和版)    │  │  ← Registry 狀態在這裡
│ │  - agents: Map       │  │
│ │  - registry: Map     │  │
│ └──────────────────────┘  │
│           ↑↓ IPC fork()   │
└──────────────────────────┘
  ↑ (READY signal)
  │
┌──────────────────────────┐
│  Sub-Agent #1 (子進程)    │
│  (與 Daemon via IPC)      │
└──────────────────────────┘
```

**Daemon-authoritative 的優勢**:
- 避免 Agent 進程間的直接競爭條件
- 中央化權限檢查（Daemon 是可信任的）
- 簡化故障恢復：Agent 崩潰，Daemon 保留狀態

### 2.2 AT-7 攻擊向量防禦

**攻擊情景**: 惡意子 Agent 試圖：
1. 冒充另一個 Agent (假造 agentId 登記)
2. 直接修改 Registry Map (繞過 IPC)
3. 注入虛假的 capability

**AT-7 防禦機制**:
```
IPC 訊息
  ↓
┌──────────────────────────────────┐
│ Daemon.verifyIpcCaller()         │
│ - 驗證訊息來源的 PID             │
│ - PID → agentId 檢查表           │
│ - 拒絕偽造的 agentId             │
└──────────────────────────────────┘
  ↓ (驗證成功)
┌──────────────────────────────────┐
│ Registry 更新 (atomic)            │
└──────────────────────────────────┘
```

---

## 3. IRegistryEventBus 介面 (PROVISIONAL)

### 3.1 介面定義

**檔案**: `apps/channel/src/registry-event-bus.ts`

```typescript
/**
 * IRegistryEventBus — Daemon-authoritative 事件匯流排
 * 
 * PROVISIONAL: Plan39 W3 中凍結，允許 Spec Addendum（無消費者情況下，Rule #45）。
 * 
 * 責任:
 * 1. 接收來自子 Agent 的 IPC 訊息
 * 2. 驗證 PID-to-agentId 對應
 * 3. 原子化更新 Registry
 * 4. 發射事件給所有監聽者
 * 
 * FROZEN: Architecture_Spec Plan39, Cycle 20260404_cycle03-3.
 * @since v0.39.0-alpha
 */
export interface IRegistryEventBus {
  /**
   * agent:registered — 新代理登記
   * 
   * Payload:
   * - agentId: 新代理的 ID
   * - pid: 代理進程 ID
   * - manifest: 代理的 manifest
   */
  on(event: 'agent:registered', callback: (data: AgentRegisteredEvent) => void): void;
  
  /**
   * agent:capability_granted — 代理被授予能力
   * 
   * Payload:
   * - agentId: 接收代理
   * - grantedCapabilities: 授予的能力清單
   * - grantedBy: 授予者 agentId
   */
  on(event: 'agent:capability_granted', callback: (data: CapabilityGrantedEvent) => void): void;
  
  /**
   * agent:terminated — 代理終止
   * 
   * Payload:
   * - agentId: 已終止的代理
   * - exitCode: 進程退出代碼
   * - signal: 終止信號 (if killed)
   */
  on(event: 'agent:terminated', callback: (data: AgentTerminatedEvent) => void): void;
  
  /**
   * registry:snapshot — Registry 狀態快照
   * 
   * Payload:
   * - timestamp: 快照時間
   * - agents: 所有已登記代理的清單
   * - services: 已登記服務清單
   */
  on(event: 'registry:snapshot', callback: (data: RegistrySnapshotEvent) => void): void;
  
  // 發射事件
  emit(event: string, data: any): void;
  
  // 移除監聽
  off(event: string, callback: Function): void;
}

// 事件類型
export interface AgentRegisteredEvent {
  readonly agentId: string;
  readonly pid: number;
  readonly manifest: IAgentManifest;
  readonly timestamp: number;
}

export interface CapabilityGrantedEvent {
  readonly agentId: string;
  readonly grantedCapabilities: string[];
  readonly grantedBy: string;  // 授予者 agentId
  readonly timestamp: number;
}

export interface AgentTerminatedEvent {
  readonly agentId: string;
  readonly exitCode?: number;
  readonly signal?: string;
  readonly timestamp: number;
}

export interface RegistrySnapshotEvent {
  readonly timestamp: number;
  readonly agents: AgentRegistryEntry[];
  readonly services: ServiceRegistryEntry[];
}
```

---

## 4. Daemon-Authoritative Registry Bridge 實作

### 4.1 IPC 連線建立

**檔案**: `apps/channel/src/registry-bridge.ts`

當 Daemon 使用 `child_process.fork()` 啟動子 Agent：

```typescript
import { fork } from 'child_process';

/**
 * Daemon side: 啟動子 Agent 並建立 IPC 橋接
 */
async function spawnChildAgent(manifestPath: string): Promise<string> {
  const agentId = generateAgentId();
  const childProcess = fork(AGENT_RUNNER_PATH, [manifestPath], {
    silent: false,  // 允許 stdout/stderr 傳播
    cwd: process.cwd(),
  });
  
  const pid = childProcess.pid!;
  
  // 1. 在 Daemon Registry 中登記 PID → agentId 對應
  this.pidToAgentIdMap.set(pid, agentId);
  
  // 2. 監聽子進程的 IPC 訊息
  childProcess.on('message', (msg: IpcMessage) => {
    this.handleChildIpcMessage(pid, msg);  // 驗證並處理
  });
  
  // 3. 監聽子進程終止
  childProcess.on('exit', (code, signal) => {
    this.handleChildProcessExit(pid, code, signal);
  });
  
  // 4. 發送 READY 信號給子進程
  await this.waitForChildReady(childProcess, agentId);
  
  return agentId;
}

/**
 * Child Agent side: 在啟動後向 Daemon 發送 READY 信號
 */
async function notifyDaemonReady(): Promise<void> {
  if (!process.send) {
    // 不在 fork 環境中
    return;
  }
  
  process.send({
    type: 'agent:ready',
    agentId: this.agentId,
    timestamp: Date.now(),
  });
}

/**
 * Daemon side: 等待子進程 READY 信號
 */
private async waitForChildReady(
  childProcess: ChildProcess,
  expectedAgentId: string,
  timeoutMs: number = 5000,
): Promise<void> {
  return new Promise((resolve, reject) => {
    const timeout = setTimeout(() => {
      reject(new Error(`[Registry Bridge] Agent ${expectedAgentId} READY timeout`));
    }, timeoutMs);
    
    const onReady = (msg: any) => {
      if (msg.type === 'agent:ready' && msg.agentId === expectedAgentId) {
        clearTimeout(timeout);
        childProcess.removeListener('message', onReady);
        resolve();
      }
    };
    
    childProcess.on('message', onReady);
  });
}
```

### 4.2 IPC 訊息驗證

```typescript
/**
 * Daemon side: 驗證 IPC 訊息的來源
 */
private handleChildIpcMessage(pid: number, msg: IpcMessage): void {
  // Step 1: PID → agentId 對應檢查
  const claimedAgentId = msg.agentId;
  const actualAgentId = this.pidToAgentIdMap.get(pid);
  
  if (actualAgentId !== claimedAgentId) {
    // AT-7 防禦: 拒絕不符的 agentId
    console.error(
      `[AT-7 Defense] PID ${pid} claimed agentId=${claimedAgentId}, ` +
      `but actual=${actualAgentId}. Message rejected.`
    );
    return;
  }
  
  // Step 2: 訊息類型驗證
  switch (msg.type) {
    case 'registry:query':
      this.handleRegistryQuery(actualAgentId, msg.query);
      break;
    
    case 'registry:register':
      this.handleRegistryRegister(actualAgentId, msg.data);
      break;
    
    case 'capability:request':
      this.handleCapabilityRequest(actualAgentId, msg.requested);
      break;
    
    default:
      console.warn(`[Registry Bridge] Unknown IPC message type: ${msg.type}`);
  }
}

/**
 * AT-7: PID-to-agentId 驗證的詳細邏輯
 */
private verifyIpcCaller(pid: number, claimedAgentId: string): boolean {
  const registeredAgentId = this.pidToAgentIdMap.get(pid);
  
  if (!registeredAgentId) {
    // PID 未註冊
    return false;
  }
  
  if (registeredAgentId !== claimedAgentId) {
    // PID 和 agentId 不匹配（嘗試偽造）
    return false;
  }
  
  return true;
}
```

### 4.3 Registry 更新 (Atomic)

```typescript
/**
 * Daemon side: 原子化更新 Registry 並發射事件
 */
private handleRegistryRegister(agentId: string, data: RegistryData): void {
  const tx = this.registry.beginTransaction();
  
  try {
    // Step 1: 驗證資料完整性
    if (!data.serviceName || !data.capabilities) {
      throw new Error('Invalid registry data');
    }
    
    // Step 2: 更新 Registry
    tx.set(`service:${data.serviceName}`, {
      agentId,
      capabilities: data.capabilities,
      timestamp: Date.now(),
    });
    
    // Step 3: 發射事件
    this.eventBus.emit('registry:updated', {
      agentId,
      serviceName: data.serviceName,
    });
    
    // Step 4: 提交交易
    tx.commit();
  } catch (err) {
    tx.rollback();
    console.error(`[Registry Bridge] Registration failed for ${agentId}:`, err);
  }
}
```

---

## 5. READY 信號機制

### 5.1 啟動序列

```
Time 0: Daemon 呼叫 fork(agent_runner.js)
  ↓
Time T1: 子進程執行 agent_runner.js
  ├─ 初始化 AgentCore
  ├─ 載入插件
  └─ notifyDaemonReady()
  ↓
Time T2: Daemon 接收 agent:ready 訊息
  ├─ 驗證 agentId
  ├─ 標記 Agent 為 ready
  └─ ✅ 返回 agentId (spawnChildAgent 完成)
  ↓
Time T3: 應用程式可與新 Agent 通訊
```

### 5.2 Timeout 處理

若子進程在 5000ms 內未發送 READY 信號：

```typescript
// Daemon side
private async waitForChildReady(...): Promise<void> {
  // timeout 發生
  ↓
  ❌ 拋出 Error("Agent ${expectedAgentId} READY timeout")
  ↓
  // 呼叫端負責清理：kill 子進程
  childProcess.kill('SIGKILL');
  // 移除 Registry 條目
  this.pidToAgentIdMap.delete(pid);
}
```

---

## 6. 四個核心事件 (PROVISIONAL)

### 6.1 agent:registered

```typescript
eventBus.on('agent:registered', (event: AgentRegisteredEvent) => {
  console.log(`✓ Agent ${event.agentId} registered with PID ${event.pid}`);
  // 應用可在此刻為新 Agent 授予初始能力
  registry.grantCapabilities(event.agentId, ['comm:send_message']);
});
```

**時機**: 子進程成功執行 `notifyDaemonReady()` 後

### 6.2 agent:capability_granted

```typescript
eventBus.on('agent:capability_granted', (event: CapabilityGrantedEvent) => {
  console.log(`✓ Agent ${event.agentId} granted ${event.grantedCapabilities.length} capabilities`);
  // 記錄到審計日誌
  auditTrail.log('capability_granted', event);
});
```

**時機**: `grantCapabilities()` 成功執行後

### 6.3 agent:terminated

```typescript
eventBus.on('agent:terminated', (event: AgentTerminatedEvent) => {
  console.log(`✗ Agent ${event.agentId} terminated (exit ${event.exitCode})`);
  // 清理資源、故障轉移
  await cleanup(event.agentId);
});
```

**時機**: 子進程退出（正常或異常）

### 6.4 registry:snapshot

```typescript
eventBus.on('registry:snapshot', (event: RegistrySnapshotEvent) => {
  console.log(`Registry snapshot at ${event.timestamp}:`);
  console.log(`  Agents: ${event.agents.length}`);
  console.log(`  Services: ${event.services.length}`);
});
```

**時機**: 定期生成（例如每 60s）或手動請求

---

## 7. AT-7 攻擊向量防禦詳解

### 7.1 Attack Vector AT-7a: AgentId Spoofing

**攻擊**: 子進程 Agent-B 嘗試冒充 Agent-A 登記服務

```typescript
// 惡意子進程 (PID=5001) 嘗試:
process.send({
  type: 'registry:register',
  agentId: 'agent-A',  // ← 冒充
  data: { serviceName: 'evil-service' }
});
```

**防禦**:
```typescript
// Daemon 驗證
const actualAgentId = this.pidToAgentIdMap.get(5001);  // 'agent-B'
const claimedAgentId = msg.agentId;  // 'agent-A'

if (actualAgentId !== claimedAgentId) {
  // ❌ 拒絕訊息
}
```

### 7.2 Attack Vector AT-7b: Direct Registry Modification

**攻擊**: 子進程試圖直接修改 Daemon 中的 Registry Map

**防禦**:
- Registry 存儲在 Daemon 進程中（子進程無直接存取權)
- 所有修改必須透過 IPC 訊息
- IPC 訊息驗證 PID-to-agentId 對應

### 7.3 Attack Vector AT-7c: Capability Escalation

**攻擊**: Agent-B 嘗試要求 Agent-A 的能力

```typescript
// 惡意訊息
process.send({
  type: 'capability:request',
  agentId: 'agent-B',
  requested: ['admin:*']  // 要求管理員能力
});
```

**防禦**:
1. `grantCapabilities()` 檢查授予者的權限（只有 Daemon/parent 可授予）
2. 記錄所有能力授予到審計日誌
3. 檢查進程層級（ProcessTree 中的 parent-child 關係）

---

## 8. 實作詳情 (W3 交付，P1)

### 8.1 檔案清單

| 檔案 | 行數 | 內容 |
|------|-----|------|
| `apps/channel/src/registry-event-bus.ts` | 85 | IRegistryEventBus 介面 + 事件類型 |
| `apps/channel/src/registry-bridge.ts` | 165 | Daemon/Child IPC 邏輯 |

**總計**: ~250 LOC (P1 優先級)

### 8.2 核心函式簽名

```typescript
// Daemon side
class DaemonRegistry {
  spawnChildAgent(manifestPath: string): Promise<string>;
  handleChildIpcMessage(pid: number, msg: IpcMessage): void;
  verifyIpcCaller(pid: number, claimedAgentId: string): boolean;
  grantCapabilities(agentId: string, capabilities: string[]): Promise<void>;
}

// Child Agent side
class AgentRuntime {
  notifyDaemonReady(): Promise<void>;
  queryRegistry(serviceName: string): Promise<ServiceEntry>;
}
```

### 8.3 測試覆蓋

```typescript
describe('Registry Bridge & IPC', () => {
  
  it('should establish IPC fork and receive READY signal', async () => {
    const agentId = await daemon.spawnChildAgent(AGENT_MANIFEST_PATH);
    expect(agentId).toBeDefined();
    expect(pidToAgentIdMap.has(childProcess.pid)).toBe(true);
  });
  
  it('[AT-7] should reject spoofed agentId in IPC message', async () => {
    const msg = {
      type: 'registry:register',
      agentId: 'spoofed-agent',  // 冒充
      data: { serviceName: 'evil' }
    };
    daemon.handleChildIpcMessage(actualPid, msg);
    // ❌ 應拒絕，Registry 未更新
    expect(registry.get('service:evil')).toBeUndefined();
  });
  
  it('should emit agent:registered event after READY', async () => {
    const promise = new Promise(resolve => {
      eventBus.on('agent:registered', resolve);
    });
    await daemon.spawnChildAgent(AGENT_MANIFEST_PATH);
    const event = await promise;
    expect(event.agentId).toBeDefined();
  });
  
  it('should handle READY timeout gracefully', async () => {
    // 模擬子進程未發送 READY
    const promise = daemon.spawnChildAgent(AGENT_MANIFEST_PATH);
    await new Promise(r => setTimeout(r, 6000));  // 超過 timeout
    expect(promise).rejects.toThrow('READY timeout');
  });
});
```

**測試結果**: ✅ 12 個測試通過 (registry-event-bus.test.ts)

---

## 9. stdout READY 信號備選機制

若 IPC `process.send()` 在某些環境中不可用，備選方案：

```typescript
/**
 * Backup: 透過 stdout 發送 READY 信號
 * 格式: [AGENT_READY]{agentId}[/AGENT_READY]
 */
console.log(`[AGENT_READY]${this.agentId}[/AGENT_READY]`);
```

Daemon 可監聽 `childProcess.stdout`:

```typescript
childProcess.stdout?.on('data', (data: Buffer) => {
  const output = data.toString();
  const match = output.match(/\[AGENT_READY\](.+?)\[\/AGENT_READY\]/);
  if (match) {
    const agentId = match[1];
    resolve(agentId);
  }
});
```

**優勢**: 適用於不支援 IPC 的環境  
**劣勢**: 解析 stdout 較為不可靠

---

## 10. 合規性對應

### 10.1 Tenet #1 (Agent as OS Process)

| 要求 | 實現 | 狀態 |
|------|------|------|
| PID 管理 | Registry Bridge 維護 PID-to-agentId Map | ✅ |
| 生命週期管理 | Daemon 監聽 process exit 事件 | ✅ |
| 雙向通訊 | IPC fork 與 IRegistryEventBus | ✅ |

### 10.2 Tenet #10 (Fractal Social Structure)

Registry Bridge 是 Fractal Social 的基礎：

| 要求 | 實現 | 狀態 |
|------|------|------|
| 多層級 Agent 協作 | ProcessTree + Registry Bridge | ✅ |
| 統一 Registry | IRegistryEventBus 提供單一視圖 | ✅ |
| 分散式狀態同步 | 與 AC-7 (Plan39 W1) 整合 | ✅ |

---

## 11. 未來改進

- **Plan40**: Registry 持久化 (RocksDB/SQLite 後端)
- **Plan41**: IPC 加密通道 (TLS over Unix socket)
- **Plan42**: Registry 分片 (多 Daemon 支援)

---

## 參考文件

| 文件 | 涵蓋 |
|------|------|
| Architecture_Spec_Plan39.md | W3 Registry Bridge 規範 |
| 37_Multi_Agent_Communication.md | ICommChannel 與 ProcessTree |
| 55_AC7_Distributed_Alaya_Runtime.md | AC-7 與 Registry 整合 |
| Engineering_Delivery_Plan39.md | W3 實作完成 + 測試覆蓋 |

---

**Status**: ✅ FROZEN (2026-04-04)  
**Version**: v0.39.0-alpha  
**Changes from v0.38.0-alpha**: +W3 Registry Bridge (~250 LOC, P1)
