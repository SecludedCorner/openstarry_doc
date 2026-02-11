# 07. Management Zone & Orchestrator Technical Specification

This document defines the technical implementation standards for the "Management Zone" in the OpenStarry system. The Management Zone is responsible for Agent lifecycle management, resource isolation, and cross-Agent scheduling.

## 1. Daemon Architecture

The OpenStarry Daemon (`openstarryd`) is a persistent system service that serves as the host for all Agents.

### 1.1 Core Responsibilities
*   **Process Management:** Start, stop, and restart Agent instances.
*   **Resource Monitoring:** Monitor CPU, memory, and token consumption.
*   **Registry Service:** Maintain the local registry of plugins and Agents.

### 1.2 Internal API (Local Control Plane)
The Daemon must expose a lightweight Control Plane for CLI tool invocations.

```typescript
interface IDaemonControlPlane {
  // Spawn an Agent instance
  spawnAgent(manifestPath: string, options?: ISpawnOptions): Promise<string>; // returns PID

  // Force termination
  killAgent(pid: string, signal: 'SIGTERM' | 'SIGKILL'): Promise<void>;

  // Query status
  listAgents(): Promise<IAgentStatus[]>;
}
```

---

## 2. Container Layer Specification

To ensure security and isolation, Agents should not run directly in the host process but rather within a controlled container environment.

### 2.1 Isolation Levels
The Management Zone should support multiple isolation drivers:

1.  **Process Driver (Default):** Uses Node.js `child_process` or Worker Threads. Suitable for development environments.
2.  **Docker Driver (Production):** Launches a Docker container for each Agent. Suitable for server deployments.
3.  **WASM Driver (Experimental):** Runs the Core within a WebAssembly sandbox. Suitable for edge computing.

### 2.2 Dependency Injection (DI) Protocol
The container layer is responsible for preparing the required environment variables and plugin paths before an Agent starts.

*   **Environment Variable Injection:** `OPENSTARRY_AGENT_ID`, `OPENSTARRY_HOME`
*   **Plugin Mounting:** Maps the plugin paths defined in `agent.json` to readable paths within the container.

---

## 3. Orchestration & Scheduling

The scheduler is responsible for handling cross-Agent causality chains.

### 3.1 Event Triggers
The scheduler listens for "boundary events" and awakens Agents based on rules.

```yaml
# orchestration.yaml
rules:
  - on: "git:commit_pushed"
    if: "branch == 'main'"
    action: "spawn(code-reviewer-agent)"

  - on: "alert:server_down"
    action: "spawn(devops-agent)"
```

### 3.2 State Projection
The scheduler periodically pulls state snapshots from each active Agent's `IInternalBus` and aggregates them into a Global Dashboard.

---

## 4. Hardware Abstraction Layer (HAL)

To enable Agents to run on IoT devices, the Management Zone must provide a standardized HAL.

```typescript
interface IHAL {
  getCameraStream(deviceId: string): ReadableStream;
  getSensorData(sensorType: string): Promise<number>;
  controlActuator(deviceId: string, command: any): Promise<void>;
}
```

All physical operations must go through the HAL interface. Agents are strictly prohibited from directly accessing `/dev/*` device files.
