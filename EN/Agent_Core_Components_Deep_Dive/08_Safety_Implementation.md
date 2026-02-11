# 16. Allocation of Safety Responsibilities & Implementation Locations

This document defines in detail how the safety mechanisms mentioned in `15_Safety_Circuit_Breakers.md` should be specifically implemented within the system's three-tier architecture, ensuring that security cannot be bypassed and adheres to decoupling principles.

---

## 1. Overview of Responsibility Allocation

Safety defense is not the responsibility of a single component but is composed of three levels: "Policy Definition," "Real-time Execution," and "Physical Enforcement."

| Level | Physical Component | Role | Implementation Details |
| :--- | :--- | :--- | :--- |
| **Policy Definition Layer** | **Agent Design Layer** | **Rule Maker** | Defines thresholds (e.g., Max Tokens) within the Agent template (JSON). |
| **Logic Execution Layer** | **Agent Core (SafetyMonitor)** | **Front-line Enforcer** | Performs counting, fingerprint matching, and real-time interception within the execution loop. |
| **Environment Guard Layer** | **Orchestrator Daemon** | **Final Arbiter** | Monitors process resources at the OS level and executes forced terminations (Kill). |

---

## 2. Implementation Details for Each Level

### 2.1 Agent Design Layer: Threshold Configuration
The design layer is not responsible for "running" the monitoring logic, but it determines the **standards** for monitoring.
*   **Implementation Location:** `agent_template.json`.
*   **Content:** Defines `safety_policies` for each Agent role.
*   **Advantages:** Provides differentiated safety standards for Agents with different capabilities (e.g., senior Agents have higher budgets, while junior Agents have stricter limits).

### 2.2 Agent Core: Real-time Monitoring (SafetyMonitor)
The core is the central execution point for safety logic and must be embedded within the execution loop.
*   **Implementation Location:** The internal `SafetyMonitor` module of the core (not a plugin, as it is part of the Kernel).
*   **Key Actions:**
    *   **Before every loop:** Reads `safety_policies` from the design layer.
    *   **Before LLM calls:** Checks if cumulative Token consumption exceeds the limit.
    *   **After tool execution:** Hashes the tool name and arguments to check for repetitive calls (preventing "infinite loops").
    *   **Priority check:** Checks the `Priority Queue` for `SYSTEM_HALT` instructions from humans.

### 2.3 Orchestrator Daemon: Watchdog
When the core itself goes out of control due to a bug (e.g., entering an unresponsive deadlock), the guard layer executes physical circuit breaking.
*   **Implementation Location:** The process monitoring module inside the Daemon.
*   **Key Actions:**
    *   **Resource Quotas:** If the Core process CPU usage or memory consumption exceeds the OS budget, the Daemon executes `kill -9`.
    *   **Forced Reset:** Receives emergency stop requests from the UI and terminates the process directly, bypassing Core logic.
    *   **Heartbeat Detection:** Checks if the Core completes a Tick within the specified time; if there is no response, it is determined as "frozen thought," and a forced restart is triggered.

---

## 3. Design Advantages

1.  **Non-Bypassability**:
    Even if the Agent Core's LLM attempts to disable safety mechanisms by modifying memory, the Agent cannot unlock itself because the Daemon performs resource monitoring from the outside, and the startup configuration is forcibly injected by the design layer.
2.  **Clean Core**:
    The core does not need to know "why" the budget is 1000 Tokens; it only needs to execute simple "count > limit" logic.
3.  **Flexibility**:
    Administrators can adjust the entire system's safety levels in real-time by modifying JSON templates in the design layer without changing any code.

---

## 4. Related References
*   See `02_Agent_Coordination_Layer.md` for template definitions.
*   See `12_Orchestrator_Daemon_Design.md` for process monitoring.
*   See `15_Safety_Circuit_Breakers.md` for specific circuit breaking logic.
