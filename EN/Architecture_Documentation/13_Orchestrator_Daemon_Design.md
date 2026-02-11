# 12. Runtime Core: Orchestrator Daemon

This document details the design of the `Orchestrator Daemon`, a critical background service responsible for managing the lifecycle of all Agent OS processes in our final architecture.

## Core Positioning: The "init/systemd" of the Agent System

The `Orchestrator Daemon` is a **system-level service that resides in the background and is independent of agent business logic.** It does not participate in any LLM thinking or task execution; its responsibilities are pure and low-level.

*   **Analogy:** It is like the `systemd` or `init` process in a Linux system, or `Docker Compose` or `Kubernetes Kubelet` in the containerized world.
*   **Responsibilities:**
    1.  **Process Lifecycle Management:** Responsible for starting, stopping, monitoring, and restarting the independent OS processes of all "Agent Cores."
    2.  **Infrastructure Plugin Hosting:** Responsible for loading `infrastructure` type plugins to provide local shared services for Agents (such as message buses and state databases), ensuring environment out-of-the-box readiness and replaceability.
    3.  **Resource Management:** Monitors the resource consumption (CPU, memory) of each Agent process and can manage them according to policies (e.g., terminating processes that consume excessive resources).
    4.  **Providing Management Interfaces:** Exposes a set of internal APIs to be invoked by the `AgentManagerTool` or external management tools for programmatic management of Agent instances.

---

## Workflow

### 1. System Startup

*   A system administrator executes the `openstarry-daemon start` command on the server to start the daemon.
*   Upon startup, the first thing the `Orchestrator Daemon` does is start and begin monitoring the OS process of the **"Master Agent"** based on its main configuration file.

### 2. Agent Creation (Triggered by `AgentManagerTool`)

1.  The `AgentManagerTool` of the "Master Agent" is invoked with the intent to start a new Worker Agent.
2.  The `execute` method of the `AgentManagerTool` no longer creates the process itself but instead issues an API request to the locally running `Orchestrator Daemon`.
    *   **Request Example:** `POST http://localhost:5050/agents`
    *   **Request Body:**
        ```json
        {
          "agent_id": "data_worker_007",
          "template_id": "DataAnalystAgent_v1"
        }
        ```
3.  Upon receiving the API request, the `Orchestrator Daemon` performs the following operations:
    *   Queries the "Agent Design Layer" for the configuration of the `DataAnalystAgent_v1` template.
    *   Prepares the environment variables and command-line arguments required to start the new Agent process.
    *   Creates a new, independent OS process using `child_process.spawn()` or a similar system call.
    *   Records the `PID` and `agent_id` of the new process in its monitoring list.
    *   Starts monitoring the health of the new process (e.g., via heartbeats or checking if the `PID` exists).

### 3. Agent Destruction

*   When the `AgentManagerTool` calls `agent:stop(agent_id)`, it issues a `DELETE /agents/{agent_id}` API request to the Daemon.
*   Upon receiving the request, the `Orchestrator Daemon` is responsible for safely terminating the corresponding OS process and cleaning up related resources.

---

## Conclusion

The introduction of the `Orchestrator Daemon` represents the final form of our architectural evolution. It perfectly addresses resource management and lifecycle monitoring in a multi-process architecture, while keeping the responsibilities of the "Master Agent" and `AgentManagerTool` pure—they are only responsible for **decisions** ("I need a new agent"), leaving the "dirty work" of **execution** ("how to actually create and monitor an OS process") to this specialized low-level service.