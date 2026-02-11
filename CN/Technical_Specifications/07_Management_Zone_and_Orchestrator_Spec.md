# 07. 管理层与编排器技术规范 (Management Zone & Orchestrator Spec)

本文件定义了 OpenStarry 系统中「管理层 (Management Zone)」的技术实现标准。管理层负责 Agent 的生命周期管理、资源隔离与跨 Agent 调度。

## 1. 守护进程架构 (Daemon Architecture)

OpenStarry Daemon (`openstarryd`) 是一个常驻的系统服务，充当所有 Agent 的宿主。

### 1.1 核心职责
*   **Process Management:** 启动、停止、重启 Agent 实例。
*   **Resource Monitoring:** 监控 CPU、内存与 Token 消耗。
*   **Registry Service:** 维护本地插件与 Agent 的注册表。

### 1.2 内部 API (Local Control Plane)
Daemon 必须暴露一个轻量级的控制平面 (Control Plane) 供 CLI 工具调用。

```typescript
interface IDaemonControlPlane {
  // 启动一个 Agent 实例
  spawnAgent(manifestPath: string, options?: ISpawnOptions): Promise<string>; // returns PID

  // 强制终止
  killAgent(pid: string, signal: 'SIGTERM' | 'SIGKILL'): Promise<void>;

  // 查询状态
  listAgents(): Promise<IAgentStatus[]>;
}
```

---

## 2. 容器层规范 (Container Layer Spec)

为了确保安全与隔离，Agent 不应直接运行在宿主进程中，而应运行在受控的容器环境内。

### 2.1 隔离级别 (Isolation Levels)
管理层应支持多种隔离驱动 (Driver)：

1.  **Process Driver (预设):** 使用 Node.js `child_process` 或是 Worker Threads。适用于开发环境。
2.  **Docker Driver (生产):** 为每个 Agent 启动一个 Docker 容器。适用于服务器部署。
3.  **WASM Driver (实验):** 在 WebAssembly 沙盒中运行 Core。适用于边缘计算。

### 2.2 依赖注入 (DI) 协议
容器层负责在 Agent 启动前，准备好其所需的环境变量与插件路径。

*   **环境变量注入:** `OPENSTARRY_AGENT_ID`, `OPENSTARRY_HOME`
*   **插件挂载:** 将 `agent.json` 中定义的插件路径映射到容器内的可读路径。

---

## 3. 编排与调度 (Orchestration)

调度器负责处理跨 Agent 的因果链 (Causality Chain)。

### 3.1 事件触发器 (Event Triggers)
调度器监听「边界事件」并根据规则唤醒 Agent。

```yaml
# orchestration.yaml
rules:
  - on: "git:commit_pushed"
    if: "branch == 'main'"
    action: "spawn(code-reviewer-agent)"

  - on: "alert:server_down"
    action: "spawn(devops-agent)"
```

### 3.2 状态投影 (State Projection)
调度器定期从各个活跃 Agent 的 `IInternalBus` 拉取状态快照，并汇总为全局视图 (Global Dashboard)。

---

## 4. 硬件抽象层 (HAL - Hardware Abstraction Layer)

为了让 Agent 能运行于 IoT 设备，管理层需提供标准化的 HAL。

```typescript
interface IHAL {
  getCameraStream(deviceId: string): ReadableStream;
  getSensorData(sensorType: string): Promise<number>;
  controlActuator(deviceId: string, command: any): Promise<void>;
}
```

所有物理操作必须通过 HAL 接口，严禁 Agent 直接存取 `/dev/*` 设备文件。
