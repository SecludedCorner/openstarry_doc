# 08. System & Project Runtime Layouts

This document implements the core philosophy of **"Directory as Permission."** OpenStarry distinguishes between "Global System Space" and "Local Project Space" to achieve permission isolation and environment portability.

---

## 1. System-wide Space

This is the primary directory for the Orchestrator Daemon, typically located under the user's home folder.

*   **Path (Windows):** `%USERPROFILE%\.openstarry`
*   **Path (Linux/macOS):** `~/.openstarry/`

### Directory Structure

```text
~/.openstarry/
├── daemon.pid              # Daemon process lock file
├── config.json             # Daemon global configuration (API Port, storage paths, etc.)
├── agents/                 # [System-level Agents] All entities managed by the Daemon
│   ├── agent-001/          # Standard directory conforming to Document 04
│   └── agent-002/ 
│   
├── plugins/                # [System-level Plugins] Globally available tools and Providers
│   
├── storage/                # Global persistent data
│   └── main.db             # SQLite/LevelDB (stores Agent state indices)
│   
└── logs/                   # Daemon system logs
```

---

## 2. Project-specific Space

When a developer starts OpenStarry within a specific project directory, the system automatically recognizes the project space.

*   **Path:** `[Your-Project-Path]/.openstarry/`

### Directory Structure

```text
.openstarry/
├── agent.json              # Project-specific Agent configuration
├── plugins/                # [Project Plugins] Capabilities visible only to this project
│   └── custom-tools/       # Project-private tools
│   
├── data/                   # Local data produced by the project
│   
└── logs/                   # Project operational logs
```

---

## 3. Loading Priorities & Overlay Mechanism

OpenStarry employs a loading logic similar to Linux UnionFS. When an Agent searches for a plugin or configuration, the search order is as follows:

1.  **Project Space (`./.openstarry/`)**: Highest priority, used for customization and debugging.
2.  **System Space (`~/.openstarry/`)**: Standard capability provider.
3.  **Built-in Space**: Original default configurations distributed with the binary.

### Permission Security Principles

*   **Isolation**: Project Agents cannot access Agent data in the system space by default unless explicitly granted permission.
*   **Read-only**: System-level plugins are typically read-only for Project Agents to prevent accidental damage to system stability.

---

## 4. Philosophical Mapping

*   **System Space = "Environment"**: The shared ecosystem where digital species reside.
*   **Project Space = "Habitat"**: The specific execution territory of a digital species.
*   **Directory as Permission**: Whether a plugin is placed in `~/.openstarry` or `./.openstarry` directly determines its trust level and visibility scope.
