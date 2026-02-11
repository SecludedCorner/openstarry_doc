# 09. CLI Design & Management Commands

The OpenStarry CLI (`openstarry`) serves as the unified entry point for users to interact with the Agent system. It is context-aware, automatically switching between "Project Mode" and "System Mode" based on its execution location.

## 1. Core Design Principles

### Context Awareness
The CLI's "Design Mode" (`openstarry design`) is highly context-aware, its behavior determined by **where you execute it**.
*   **Global Design Mode**: When executed outside of a project directory, it provides system-level management (e.g., creating new projects, managing global templates).
*   **Project Design Mode**: When executed in a directory containing `agent.json`, it automatically enters the editor and configuration interface for that specific Agent.

### Separation of Runtime and Design
*   **`openstarry` (Runtime)**: Focuses on **execution and monitoring**. Regardless of location, it connects to the daemon to provide a real-time system dashboard.
*   **`openstarry design` (Design)**: Focuses on **definition and configuration**. It is the workbench where you build aggregates and assemble Agents.

### Foreground vs. Background
*   **Project Mode (Foreground)**: The Agent runs as a sub-process of the current shell. Closing the terminal stops the Agent.
*   **Daemon Mode (Background)**: Launched via the Daemon, the Agent continues to run in the background even if the terminal is closed.

---

## 2. Detailed Command Set

### A. Core Entry Points

```bash
# Starts the OpenStarry Runtime Environment
$ openstarry
# -> Checks if the Orchestrator Daemon is running.
# -> If not running, automatically starts the Daemon and Master Agent (System Mode).
# -> If already running, enters the interactive Console, displaying current Agent statuses and system metrics.
# -> [Console Feature] Select a specific Agent and press 'a' to connect (Attach) for dialogue.

# Context Aware - Automatically switches based on current directory:
#    - Global Mode: Displays menus for "Create New Project," "Download Template," and "Manage Global Settings."
#    - Project Mode: Displays current Agent structure, guiding the user to define the **Five Aggregates**:
#      * Set Persona and Memory (Consciousness)
#      * Configure Interface and Name (Form)
#      * Add Tool Plugins (Volition)
#      * Choose Sensory Listeners (Sensation)
#      * Specify AI Model (Perception)
# -> Launches an interactive TUI (Text User Interface).
# -> Automatically generates or updates agent.json and the directory structure.
```

### B. Project Lifecycle

These commands manage "Project-level Agents" in the current directory.

```bash
# Initializes a new Agent project
$ openstarry init [template-name]
# -> Creates agent.json, package.json, and the .openstarry/ structure

# [Interactive Mode] Start and enter dialogue
$ openstarry attach
# -> When executed in a project directory:
#    - If Agent is not running: Automatically starts the Agent and enters dialogue.
#    - If Agent is already running: Directly attaches to that Agent.
# -> This is the most common command for starting conversations with an Agent.

# [Start Mode] Starts the process in the background only
$ openstarry start
# -> Reads ./agent.json and launches the Agent as a background daemon process.
# -> Writes to ./openstarry/agent.pid; logs to files.
# -> Does not occupy the current terminal.

# Manual Tool Invocation
$ openstarry run-tool <tool-name> [args...]
# -> Invokes a specific ITool directly, bypassing the LLM.
# -> Commonly used for: logging in (google-login), initial setup, or testing tool functionality.
# -> Example: openstarry run-tool google-login

# Cleans project runtime data
$ openstarry clean
# -> Deletes temporary files (logs, state cache) in ./.openstarry/
```

### C. Daemon Management

These commands interact with the daemon in `~/.openstarry`.

```bash
# Starts the system daemon (if not already running)
$ openstarry daemon start

# View system status (list all hosted Agents)
$ openstarry ps
# OUTPUT:
# ID      NAME        STATUS    PID    MODE      LOCATION
# sys-01  Scheduler   RUNNING   1024   System    ~/.openstarry/agents/sys-01
# prj-01  MyBot       RUNNING   4521   Project   D:/Code/MyBot (Registered)

# Stops the daemon (and all child Agents)
$ openstarry daemon stop
```

### D. Registration & Hosting

Elevates a "Project-level Agent" to a "System-level Service."

```bash
# Registers the current directory's Agent with the Daemon
$ openstarry register
# -> Daemon records the path "D:/Code/MyBot" in ~/.openstarry/registry.json
# -> Daemon is now responsible for monitoring and restarting this Agent.

# Attach to global or specific Agent
$ openstarry attach [agent-id]
# -> If agent-id is provided: Attaches to the specified background Agent.
# -> If in a project directory without agent-id: Attaches to the current project's Agent.
# -> Press Ctrl+C to Detach; the Agent continues running in the background.

# Refresh Plugin Registry (Manual Refresh)
$ openstarry plugin refresh
# -> Forces the Daemon to rescan the system plugin directory.
# -> Used to detect new plugins copied in manually.
```

## 3. Data Flow Summary

| Mode | Command Example | Config Source | Runtime State (PID/State) | Log Location | Lifecycle |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **Project (Foreground)** | `openstarry start` | `./agent.json` | `./.openstarry/` | `./.openstarry/logs/` | Ends with shell |
| **Project (Hosted)** | `openstarry start -d` | `./agent.json` | `~/.openstarry/state/` | `~/.openstarry/logs/` | Managed by Daemon |
| **System (Built-in)** | `openstarry system start` | `~/.openstarry/agents/` | `~/.openstarry/state/` | `~/.openstarry/logs/` | Persistent service |

This design ensures that developmental flexibility (Project Mode) perfectly coexists with operational stability (Hosted Mode).
