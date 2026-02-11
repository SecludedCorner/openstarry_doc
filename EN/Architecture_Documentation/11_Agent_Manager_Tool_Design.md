# 10. Capability Plugin Design: Agent Manager

This document details the design and responsibilities of the `AgentManagerTool` under the final architecture.

## Core Positioning: Decision Maker, Not Executor

In the final architecture that introduces the `Orchestrator Daemon`, the role of the `AgentManagerTool` has been further distilled. It no longer performs the "dirty work" of creating OS processes itself but instead acts as a **"Decision Interface."**

It is responsible for transforming the **decision** of "I need a new Agent" made by the Master Agent's LLM into a **standardized request** to the `Daemon`.

---

## `AgentManagerTool` Design

*   **Plugin Type:** `tool`

*   **Provided Methods (as an independent tool):**
    *   **`agent:start(agent_id: string, template_id: string)`**
        *   **Function:** **Requests** the `Orchestrator Daemon` to create and start a new Worker Agent based on a predefined template.
        *   **Implementation:** The `execute` logic for this method is now very simple:
            1.  Construct a JSON request body, e.g., `{ "agent_id": "...", "template_id": "..." }`.
            2.  Send a `POST` request to the management API endpoint exposed by the `Orchestrator Daemon` (e.g., `http://localhost:5050/agents`).
            3.  Wait for the API response from the `Daemon` (e.g., a JSON containing success status and the `agent_id`) and return it to the LLM.
    *   **`agent:stop(agent_id: string)`**
        *   **Function:** **Requests** the `Orchestrator Daemon` to stop and destroy a running Worker Agent.
        *   **Implementation:** Sends a `DELETE` request to the `Daemon` API endpoint (e.g., `http://localhost:5050/agents/{agent_id}`).
    *   **`agent:status(agent_id: string)`**
    *   **`agent:list()`**
        *   **Implementation:** Similarly, these methods are transformed into `GET` requests to the `Daemon` management API.

---

## Summary

The advantages of this design are evident:

*   **Separation of Responsibilities:**
    *   **Master Agent (LLM):** Responsible for high-level business decisions (**Why & What** - why an agent is needed, and what kind of agent is needed).
    *   **`AgentManagerTool`:** Responsible for transforming decisions into standard API requests (**How to Ask** - how to submit a creation request).
    *   **`Orchestrator Daemon`:** Responsible for all low-level, OS-related implementation details (**How to Do** - how to actually create and manage a process).
*   **Scalability:** This API-based design allows the `Daemon` to be easily extended into a cluster manager capable of creating Agents across multiple machines in a distributed manner, without requiring any changes to the `AgentManagerTool` interface.
*   **Security:** All authority to create/destroy processes is centralized in the `Daemon`, facilitating unified auditing and permission control.