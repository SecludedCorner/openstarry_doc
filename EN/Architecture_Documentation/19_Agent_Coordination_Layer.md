# 19. Agent Coordination Layer

This document defines the "central nervous system" of the OpenStarry system—the Coordination Layer. Located within the Daemon, it is responsible for managing all plugin registrations, Agent communication routing, and lifecycles.

## 1. Core Responsibilities

The Coordination Layer is a logical module within the **Daemon** process that handles the following responsibilities:

1.  **Plugin Registry Authority:**
    *   Maintains a list of all available plugins across the system (scanned from `~/.openstarry/plugins`).
    *   Responds to Agent loading requests by providing the physical paths of plugins.
    *   Processes the `openstarry plugin sync` command to synchronize updates from the `openstarry_plugin` repository.

2.  **Agent Registry:**
    *   Records the PID, communication ports (Socket/Pipe), and status of all active Agents.
    *   Provides the `getAgent(id)` interface, allowing the CLI (`attach`) or other Agents to locate a target.

3.  **Message Routing:**
    *   When an Agent issues a `send("manager", "help")`, the Coordination Layer is responsible for resolving who "manager" is and forwarding the message.

## 2. Plugin Registration Process

When a developer installs a new plugin or when the system starts:

1.  **Scanning:** The Coordination Layer scans the plugin directories.
2.  **Validation:** Checks the validity of `plugin.json`.
3.  **Indexing:** Stores the `id`, `version`, and `capabilities` of the plugin in an in-memory database (e.g., LokiJS or SQLite).
    *   Indexing Example: `Capability("weather") -> Plugin("openstarry-weather-v1")`.

## 3. Interaction at Agent Startup

1.  **Request:** A newly started Agent Core sends a request to the Coordination Layer: "I need `standard/fs` and `gemini`."
2.  **Resolution:** The Coordination Layer queries the index to confirm these plugins exist and are version-compatible.
3.  **Authorization:** The Coordination Layer checks the permission declarations in `agent.json` to confirm the Agent is authorized to use these plugins.
4.  **Return:** The Coordination Layer returns the entry file paths of the plugins.
5.  **Registration:** After successful loading, the Agent reports back to the Coordination Layer: "I am online, I am `data-bot-01`, and I have the `analyze-data` capability."

## 4. Implementation Technology

*   **Communication Protocol:** Uses **gRPC** or **Named Pipes** (IPC) for efficient communication between the Core and the Daemon.
*   **Storage:** Uses a lightweight embedded database to store registry information, ensuring it is not lost after a restart.

---

## 5. Persistence and Boot Strategy

### A. Physical Persistence
Plugins installed via `openstarry plugin sync` or `add` have their source code **stored permanently** in the `~/.openstarry/plugins/` directory. They persist after a restart unless manually deleted.

### B. Registry Indexing
To ensure high system performance and consistency, the Coordination Layer employs the following boot strategy:

1.  **Fast Scan:** Each time the Daemon starts, it first scans the plugin directory for **file modification times (mtime)**.
2.  **Incremental Refresh:** 
    *   If there are no changes to the directory, the cache is read directly from `registry.db`.
    *   If new folders or version changes are detected, `plugin.json` is automatically parsed, and the database index is updated.
3.  **Invalidation Cleanup:** If a plugin recorded in the database no longer exists on the disk, it is automatically removed from the registry.

### C. Runtime State
Note that **persistence of the "Registry" is not the same as persistence of "Plugin Instances."** Each time the system starts, plugin code is reloaded and `initialize()` is executed according to Protocol `18`. If a plugin needs to save runtime data, it must do so via the `state` interface injected by the Core (see document 06).

---

## 6. Health Check and Self-Healing

The Coordination Layer possesses robust fault-tolerance capabilities to handle various abnormal states resulting from manual operations.

### A. Manual Drop-in
When a user copies a folder directly into `~/.openstarry/plugins/`:
1.  **Discovery:** The Daemon's file watcher or startup scan detects the new directory.
2.  **Validation:** Attempts to parse `plugin.json`.
    *   **Success:** Automatic registration, status set to `READY`.
    *   **Failure:** Status set to `INVALID_MANIFEST`, and a yellow warning is displayed on the Dashboard.

### B. Load Failure
If plugin code contains errors (e.g., syntax errors) causing `initialize()` to crash:
1.  **Isolation:** The Loader catches the exception to prevent the Daemon from crashing.
2.  **Tagging:** Status set to `QUARANTINED`.
3.  **Reporting:** A red ❌ is displayed on the Dashboard, with an option to view error logs.

### C. Dependency Conflict
If Plugin A declares a dependency on Plugin B, but B does not exist:
1.  **Check:** A broken link is discovered during dependency graph construction.
2.  **Blocking:** Refuses to load Plugin A and sets its status to `UNSATISFIED`.
3.  **Guidance:** The CLI outputs a clear recommendation:
    > "Plugin [A] requires [B]. Please run `openstarry plugin add B` or `openstarry plugin sync` to resolve."