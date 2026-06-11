# 07. 管理層與編排器技術規範 (Management Zone & Orchestrator Spec)

本文件定義了 OpenStarry 系統中「管理層 (Management Zone)」的技術實作標準。管理層負責 Agent 的生命週期管理、資源隔離與跨 Agent 調度。

## 1. 守護進程架構 (Daemon Architecture)

OpenStarry Daemon (`openstarryd`) 是一個常駐的系統服務，充當所有 Agent 的宿主。

### 1.1 核心職責
*   **Process Management:** 啟動、停止、重啟 Agent 實例。
*   **Resource Monitoring:** 監控 CPU、記憶體與 Token 消耗。
*   **Registry Service:** 維護本地插件與 Agent 的註冊表。

### 1.2 內部 API (Local Control Plane)
Daemon 必須暴露一個輕量級的控制平面 (Control Plane) 供 CLI 工具調用。

```typescript
interface IDaemonControlPlane {
  // 啟動一個 Agent 實例
  spawnAgent(manifestPath: string, options?: ISpawnOptions): Promise<string>; // returns PID
  
  // 強制終止
  killAgent(pid: string, signal: 'SIGTERM' | 'SIGKILL'): Promise<void>;
  
  // 查詢狀態
  listAgents(): Promise<IAgentStatus[]>;
}
```

---

## 2. 容器層規範 (Container Layer Spec)

為了確保安全與隔離，Agent 不應直接運行在宿主進程中，而應運行在受控的容器環境內。

### 2.1 隔離級別 (Isolation Levels)
管理層應支援多種隔離驅動 (Driver)：

1.  **Process Driver (預設):** 使用 Node.js `child_process` 或是 Worker Threads。適用於開發環境。
2.  **Docker Driver (生產):** 為每個 Agent 啟動一個 Docker 容器。適用於伺服器部署。
3.  **WASM Driver (實驗):** 在 WebAssembly 沙盒中運行 Core。適用於邊緣計算。

### 2.2 依賴注入 (DI) 協議
容器層負責在 Agent 啟動前，準備好其所需的環境變數與插件路徑。

*   **環境變數注入:** `OPENSTARRY_AGENT_ID`, `OPENSTARRY_HOME`
*   **插件掛載:** 將 `agent.json` 中定義的插件路徑映射到容器內的可讀路徑。

---

## 3. 編排與調度 (Orchestration)

調度器負責處理跨 Agent 的因果鏈 (Causality Chain)。

### 3.1 事件觸發器 (Event Triggers)
調度器監聽「邊界事件」並根據規則喚醒 Agent。

```yaml
# orchestration.yaml
rules:
  - on: "git:commit_pushed"
    if: "branch == 'main'"
    action: "spawn(code-reviewer-agent)"
    
  - on: "alert:server_down"
    action: "spawn(devops-agent)"
```

### 3.2 狀態投影 (State Projection)
調度器定期從各個活躍 Agent 的 `IInternalBus` 拉取狀態快照，並匯總為全域視圖 (Global Dashboard)。

---

## 4. 硬體抽象層 (HAL - Hardware Abstraction Layer)

為了讓 Agent 能運行於 IoT 設備，管理層需提供標準化的 HAL。

```typescript
interface IHAL {
  getCameraStream(deviceId: string): ReadableStream;
  getSensorData(sensorType: string): Promise<number>;
  controlActuator(deviceId: string, command: any): Promise<void>;
}
```

所有物理操作必須透過 HAL 介面，嚴禁 Agent 直接存取 `/dev/*` 設備檔。
