# 07. Design and Positioning of Commands

In our architecture, `Command` is a multi-layered concept handled by different parts of the system depending on its context. This document aims to clarify the relationship between `Command` and `Tool`, and its specific positioning within the architecture.

---

## 1. As "User System Commands" (in UI Plugins)

This is the most direct user-facing command, used to control the Agent client or the system itself, rather than tasks assigned to the LLM.

*   **Definition:** A string starting with a special prefix (e.g., `/`), such as `/help`, `/reset`, or `/status`.
*   **Handler:** **UI Plugin**.
*   **Workflow:** When receiving user input, the `UI` plugin first performs interception judgment. If identified as a system command, it is parsed and processed by the `UI` plugin itself, without entering the "Agent Core" conversation flow. This design clearly separates user operations on the "system" from conversational tasks directed at the "Agent."

---

## 2. As a Synonym for "Tool" (in Tool Plugins)

In many contexts, the concepts of `Command` and `Tool` are interchangeable.

*   **Definition:** An atomic, single-responsibility function callable by the LLM, such as `read_file`.
*   **Handler:** **Tool Plugin**, scheduled by the "Agent Core" execution loop.
*   **Conclusion:** In our architectural documentation, to maintain terminology consistency, we uniformly refer to such functions as **`Tool`**. Therefore, if `Command` is used in this sense, it corresponds to `Tool` in our design.

---

## 3. As "High-Level Workflow Commands" (in the Agent Coordination Layer)

This is the most advanced and powerful type of command, representing a callable, multi-step complex task.

*   **Definition:** A named, reusable **"Workflow Graph."** For example, a command named `generate_weekly_report` might be backed by a workflow containing multiple nodes such as data querying, chart generation, LLM summarization, and email sending.
*   **Handler:** **Agent Coordination Layer**.
*   **Workflow:**
    1.  A user, a scheduled task, or an external API triggers a high-level command.
    2.  The **Routing Component** or **Command Dispatcher** of the "Agent Coordination Layer" receives the request and identifies the command name.
    3.  The coordination layer loads the predefined workflow graph corresponding to that command.
    4.  The **Workflow Engine** of the coordination layer is responsible for executing all nodes in that graph to complete the entire complex task.

This design allows us to encapsulate complex business logic into simple, callable "commands," significantly enhancing the system's reusability and capability ceiling.