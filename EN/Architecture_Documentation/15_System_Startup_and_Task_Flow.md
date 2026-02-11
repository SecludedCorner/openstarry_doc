# OpenStarry System: Complete Startup and Task Workflow

This document details the complete lifecycle and workflow of the OpenStarry system, from "zero boot" to the "completion of a complex multi-agent collaborative task."

## Core Workflow Overview

The entire system's operation is based on the `Orchestrator Daemon`'s management of the lifecycle of "Agent Core" processes and the intelligent collaboration between "Agent Cores" via `MCP`.

---

### Phase 1: System Bootstrapping and Master Agent Startup

1.  **Start `Orchestrator Daemon`:**
    *   **Initiator:** System administrator.
    *   **Action:** Execute the `openstarry-daemon start` command.
    *   **Result:** The `Orchestrator Daemon` starts as an independent background OS process, becoming the process guardian for the entire OpenStarry system.

2.  **`Daemon` Starts the "Master Agent":**
    *   **Role:** `Orchestrator Daemon`.
    *   **Action:** The `Daemon` starts a new OS process to run the **"Master Agent"** based on its main configuration file.
    *   **Result:** The "Master Agent" process is created, with the `Daemon` responsible for its health monitoring and lifecycle management.

3.  **Master Agent Self-Initialization:**
    *   **Role:** The "Master Agent" process (an `Agent Core` instance).
    *   **Actions:**
        *   Instantiates its internal **`PluginLoader`**.
        *   The `PluginLoader` scans preset plugin directories (e.g., `~/.config/openstarry/system_plugins/`) and loads all core capability plugins configured for the Master Agent (e.g., `UI_Listener`, `MCP_Listener`, `AgentManagerTool`, `WorkflowEngineTool`, `AgentDesignerTool`, etc.).
    *   **Result:** The "Master Agent" is fully configured with high-level capabilities such as human interaction, agent management, and workflow orchestration, and enters a standby state.

---

### Phase 2: Task Triggering and Master Agent Decision Making

4.  **Receive External Event (Task Trigger):**
    *   **Role:** The Master Agent's `Listener` plugin (e.g., `WhatsApp_Listener`).
    *   **Action:** A human user sends a new request via WhatsApp (e.g., "Please analyze last quarter's sales data and generate a report"). The `WhatsApp_Listener` receives the message.
    *   **Result:** The `Listener` transforms the message into a standardized event and pushes it into the Master Agent's **internal event queue**.

5.  **Master Agent Decision and Task Delegation:**
    *   **Role:** The Master Agent's "Execution Loop" and LLM.
    *   **Actions:**
        *   The execution loop retrieves the event from the queue and sends it along with context to the LLM.
        *   The LLM analyzes the request, determines that it requires specialized data analysis capabilities, and decides that a "Worker Agent" needs to be started.
        *   The LLM outputs a tool call request for the `AgentManagerTool`: `agent:start(template_id: 'DataAnalystAgent_v1', project_path: '/path/to/sales_data')`.

---

#### **Phase 3: Worker Agent On-demand Creation**

6.  **`AgentManagerTool` Requests Daemon to Create a Worker Agent:**
    *   **Role:** `AgentManagerTool`.
    *   **Action:** The tool's `execute` method is invoked. It issues an API request to the locally running **`Orchestrator Daemon`** to create a Worker Agent based on the `DataAnalystAgent_v1` template.

7.  **`Daemon` Creates the Worker Agent Process:**
    *   **Role:** `Orchestrator Daemon`.
    *   **Actions:**
        *   The `Daemon` queries the "Agent Design Layer" service for the detailed configuration of the `DataAnalystAgent_v1` template.
        *   Creates a new OS process for running the "Worker Agent."
        *   Passes startup parameters, such as the template configuration and `project_path`, to the new process.
        *   Adds the new process to its monitoring list.

8.  **Worker Agent Self-Initialization:**
    *   **Role:** The new "Worker Agent" process (an `Agent Core` instance).
    *   **Actions:**
        *   Instantiates its own `PluginLoader`.
        *   The `PluginLoader` loads the precise set of plugins required for that Worker Agent (e.g., `code_interpreter_suite`, `database_query_tool`, `MCP_Listener`) based on the passed template configuration and `project_path` (used for loading project-level plugins).
    *   **Result:** A lightweight, specialized "Data Analysis Agent" is fully started and enters a standby state.

---

#### **Phase 4: Inter-Agent Collaboration and Task Completion**

9.  **Master Agent Assigns Specific Task:**
    *   **Role:** The Master Agent's LLM.
    *   **Action:** After the `agent:start` tool returns successfully, the LLM knows the Worker Agent is ready. It generates a call to the `mcp:send` tool to send specific data and analysis instructions to the newly created Worker Agent.

10. **Worker Agent Executes Sub-task:**
    *   **Role:** The "Worker Agent."
    *   **Actions:**
        *   Its `MCP_Listener` receives the task instructions and pushes them into its own event queue.
        *   Its execution loop invokes its own LLM and specialized tools (e.g., `code_interpreter`) to process the data.
        *   Upon completion, it sends the analysis results (reports, charts) back to the Master Agent via the `mcp:send` tool.

11. **Master Agent Finalizes and Responds:**
    *   **Role:** The "Master Agent."
    *   **Action:** Receives the results from the Worker Agent and performs final organization and packaging.
    *   **Result:** Sends the final report back to the human user via its `UI_Listener` (e.g., the output interface of the `WhatsApp_Listener`).