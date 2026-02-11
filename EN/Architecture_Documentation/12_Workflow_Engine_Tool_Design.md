# 11. Capability Plugin Design: Workflow Engine (WorkflowEngineTool)

This document describes the design that encapsulates macro-level, multi-step task orchestration capabilities into a powerful `Tool` plugin.

## Core Positioning: From "Architectural Layer" to "Capability Plugin"

In the "Agent-centric" model, the capability to execute complex workflows is no longer borne by an independent architectural layer. Instead, it is **encapsulated as a tool plugin named `WorkflowEngineTool`**, to be invoked on demand by the "Master Agent" or other authorized high-level agents.

This keeps the "Agent Core" pure; it does not need to know how a workflow is executed, only how to call a tool named `workflow:execute`.

---

## `WorkflowEngineTool` Design

*   **Plugin Type:** `tool`

*   **Provided Tools:**
    *   `workflow:execute(workflow_id: string, initial_payload: object)`: Executes a predefined workflow.
    *   `workflow:status(execution_id: string)`: Queries the status of a running workflow.

*   **`plugin.json` Example:**
    ```json
    {
      "name": "Workflow-Engine-Tool",
      "type": "tool",
      "entryPoint": "./WorkflowEngineTool.js"
    }
    ```

*   **Internal Implementation:**
    1.  **Workflow Definition:** The plugin contains an internal parser for reading workflow definition files (e.g., `./workflows/my_flow.yaml`). These files describe the graph structure, nodes, and data flow of the workflow.
    2.  **Execution Engine:** When `workflow:execute` is called, the internal execution engine of the plugin will:
        *   Load the definition file corresponding to the `workflow_id`.
        *   Create a workflow instance and initialize its context with `initial_payload`.
        *   Start executing the various nodes asynchronously, according to the graph dependencies.
    3.  **Node Executors:** The plugin contains multiple types of "Node Executors."
        *   `ApiCallNodeExecutor`: Responsible for executing HTTP API calls.
        *   `DatabaseNodeExecutor`: Responsible for executing database queries.
        *   `CodeNodeExecutor`: Responsible for executing small snippets of data transformation code.
        *   `AgentNodeExecutor`: **(Critical)** If a node within the workflow requires an agent to complete, this executor calls the `AgentManagerTool` to create a temporary "Worker Agent" and communicates with it via MCP to complete the node's task.
    4.  **Returning Results:** The call to the `workflow:execute` tool can be designed as asynchronous. It returns an `execution_id` immediately, which the Agent can use later to query the result via `workflow:status`. Alternatively, once the workflow completes, the final result can be sent asynchronously back to the caller via `mcp:send`.

---

## Advantages

*   **Capability Cohesion:** All complex logic related to workflows is perfectly encapsulated within this single `Tool` plugin.
*   **Pure Core:** The "Agent Core" is completely unconcerned with how workflows are defined and executed.
*   **On-demand Usage:** Only the "Master Agent" or authorized high-level agents load the `WorkflowEngineTool`, while ordinary "Worker Agents" do not, maintaining a lightweight profile.
