# 03. Standard Agent Directory Anatomy & Hierarchy

This document defines the standard layout for "Agent Workspaces" within the OpenStarry system. This specification applies to both **System-level** and **Project-level** environments.

## Core Philosophy: Self-Similarity

In OpenStarry, the system administrator (System Agent) and specific executors (Project Agents) are entirely symmetrical in their physical structure. This "fractal" design ensures that any project has the potential to evolve into a manager for another subsystem in the future.

---

## Standard Directory Structure (The Anatomy)

Regardless of whether it is a system path or a project path, the following layout must be followed:

```text
[Agent_Root]/
├── plugins/           # Aggregate plugin storage (Aggregates)
│   ├── tool-A/        # Plugin A
│   └── listener-B/    # Plugin B
├── configs/           # Agent blueprints and configurations (Manifests)
│   └── agent.json     # Defines which plugins the Agent loads
├── states/            # State and persistence (States/Snapshots)
│   └── memory.db      # Physical storage for Consciousness (Vijnana)
└── logs/              # Runtime logs (historical records of Sensation)
```

---

## Plugin Discovery and Inheritance Mechanism

Upon Agent startup, the `PluginLoader` scans multiple levels of `plugins/` directories:

1.  **Priority 1 (Project-level):** `[Current_Project_Root]/plugins/`
2.  **Priority 2 (System-level):** `[openstarry_system_root]/plugins/`

**Rules:**
*   Project Agents **can** invoke system-level plugins to achieve resource sharing (e.g., using a system-provided `git-tool`).
*   If a plugin with the same name exists in both locations, the **Project-level takes precedence**, allowing the project to override system default behaviors.

---

## Role Permissions and Isolation (Security Policy)

While the structure remains identical, Agents at different levels are subject to strict responsibility limitations:

### 1. System Agent (Manager)
*   **Residence:** `openstarry_system_root`
*   **Core Tasks:** Manage the lifecycle of Project Agents (creation, monitoring, destruction).
*   **Strict Restriction:** **Forbidden from loading any business plugins** (e.g., WhatsApp, database tools).
*   **Operational Logic:** If the System Agent needs to perform a business operation, it must spawn a specific "Project Agent" to execute it. The System Agent itself only possesses the `AgentManagerTool`.

### 2. Project Agent (Worker)
*   **Residence:** Specific project directories.
*   **Core Tasks:** Execute specific business logic.
*   **Operational Logic:** Has full operational permissions for all tools within the `plugins/` directory under its path.

---

## The Kill Switch

The System Agent and human administrators hold ultimate control over Project Agents:

1.  **Process Control:** The System Agent maintains the PIDs of Project Agents via the `Orchestrator Daemon`.
2.  **Forced Termination:** When a Project Agent loses control or completes its task, the System Agent can issue a `SIGKILL` signal.
3.  **Philosophical Significance:** This represents the absolute control of "Consciousness (Vijnana)" over "Volition (Samskara)." When an action no longer aligns with system goals, the management center must be capable of physically cutting off its energy supply (terminating the process).
