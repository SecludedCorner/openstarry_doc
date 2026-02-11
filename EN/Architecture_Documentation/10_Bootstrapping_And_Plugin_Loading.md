# 09. Bootstrapping Sequence: Starting with the Daemon

This document details the system bootstrapping sequence under the final architecture. This process takes the `Orchestrator Daemon` as the highest-level starting point, ensuring the robustness and manageability of all Agent processes.

## Core Principle: Daemon -> Master Agent -> Worker Agent

System startup is a clear, three-stage process with separated responsibilities.

---

### Phase 1: Daemon Boot

1.  **Initiator:** System administrator or the operating system's service manager (e.g., `systemd`).
2.  **Command:** `opennexus-daemon start`
3.  **Actions:**
    *   The unique **"Orchestrator Daemon"** process is started and runs in the background.
    *   Upon startup, the Daemon immediately executes its primary task: starting and monitoring the OS process of the **"Master Agent"** based on the main configuration file.
4.  **Phase Outcome:**
    *   The system's core manager, the `Daemon`, is ready.
    *   The "Master Agent" process, the logical core of the system, is started, with its stability ensured by the `Daemon`.

---

### Phase 2: Master Agent Loading

1.  **Initiator:** `Orchestrator Daemon`.
2.  **Goal:** Start a fully functional "Master Agent" instance.
3.  **Process:**
    *   The "Master Agent" process begins execution.
    *   Its internal "Plugin Loader" scans one or more preset plugin directories.
    *   It loads all core capability plugins configured for the Master Agent, such as `AgentManagerTool`, `WorkflowEngineTool`, `UI_Listener`, `MCP_Listener`, etc.
4.  **Phase Outcome:**
    *   The "Master Agent" is fully initialized and enters a standby state via its `Listener` plugins, ready to receive events from users or MCP.

---

### Phase 3: Worker Agent On-demand Creation

1.  **Initiator:** The `AgentManagerTool` inside the "Master Agent."
2.  **Trigger:** The Master Agent's LLM decides that a sub-agent is needed to handle a task.
3.  **Process:**
    *   The `execute` method of the `AgentManagerTool` is invoked with parameters like `{ template_id: ... }`.
    *   This method **issues an API request to the locally running `Orchestrator Daemon`** (e.g., `POST /agents`), including the template ID to be used.
    *   **The `Orchestrator Daemon` receives the request** and takes responsibility for executing all OS-related low-level work:
        1.  Queries the "Agent Design Layer" for the template configuration.
        2.  Creates a new, independent OS process.
        3.  Injects the configuration required for that template into the new process's startup parameters.
        4.  Adds the new process to its monitoring list.
4.  **Phase Outcome:**
    *   A "tailor-made" "Worker Agent" process is safely created and started, with its lifecycle managed by the `Daemon`.
    *   The entire system achieves a perfect separation between "Decision" (Master Agent) and "Execution" (Daemon).