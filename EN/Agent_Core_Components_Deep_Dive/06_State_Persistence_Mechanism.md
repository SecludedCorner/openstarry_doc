# 14. State Management & Persistence Mechanism

This document defines how the OpenStarry system implements "Crash Recovery" and "State Consistency."

The design follows the principle of **"Core defines state, Plugins handle storage,"** ensuring the Agent Core remains stateless and lightweight while delegating complex I/O operations to specialized plugins.

## 1. Core Concept: The Snapshot

To keep the Core concise, the Agent Core maintains a pure JSON state object at runtime, referred to as the **`AgentSnapshot`**.

### 1.1 `AgentSnapshot` Data Structure

This is the sole contract between the Core and the Storage Plugin. The Core is responsible only for generating this object and does not concern itself with how it is stored.

```json
{
  "agent_id": "worker_007",
  "tick_index": 42,            // Logical clock, increments by 1 per loop iteration
  "timestamp": 1706582400000,  // Snapshot generation time
  "status": "WAITING_FOR_EVENT", // Current state machine status
  "memory": {
    "short_term_history": [    // Dialogues within the current Context Window
      { "role": "user", "content": "..." },
      { "role": "assistant", "content": "..." }
    ],
    "working_variables": {     // Variables currently held by the Agent
      "current_file_path": "/tmp/report.txt",
      "retry_count": 0
    }
  },
  "event_queue_backup": [      // Unprocessed events (used for state restoration)
    { "source": "mcp", "type": "task", "payload": "..." }
  ]
}
```

---

## 2. Architectural Design: Storage Plugin

In accordance with the "Everything is a Plugin" principle, the persistence layer is abstracted as a new plugin type.

### 2.1 New Plugin Type: `storage`

*   **Responsibility:** Implements CRUD operations for the `AgentSnapshot`.
*   **Interface Definition:**
    All `storage` plugins must implement the following methods:
    *   `save(snapshot: AgentSnapshot): Promise<void>`
    *   `load(agent_id: string): Promise<AgentSnapshot | null>`
    *   `delete(agent_id: string): Promise<void>`

### 2.2 Implementation Examples

*   **`FileStoragePlugin` (Development):** Saves snapshots as `.json` files (e.g., `./data/snapshots/worker_007.json`).
*   **`RedisStoragePlugin` (Production):** Serializes snapshots and stores them in Redis keys, supporting extremely high R/W speeds.
*   **`SQLiteStoragePlugin` (Edge Computing):** Stores data in local SQLite database files, providing ACID transaction guarantees.

---

## 3. Workflow

### 3.1 Startup and Restoration (Hydration)

When the `Orchestrator Daemon` starts an Agent Core process:

1.  **Core Startup:** Instantiation and loading of the `PluginLoader`.
2.  **Load Storage Plugin:** The `PluginLoader` loads the specified `storage` plugin (e.g., `RedisStorage`) based on configuration.
3.  **Request State:** The Core calls `await storage.load(my_agent_id)`.
4.  **State Hydration:**
    *   **If archive exists:** The Core populates its in-memory variables with data from the `AgentSnapshot` (restoring `tick_index`, `history`, and `event_queue`). **The Agent is "resurrected," remembering everything prior to the crash.**
    *   **If no archive exists:** The Core initializes to a fresh blank state.

### 3.2 Runtime Persistence

To ensure rigorous data safety, a **"Save-After-Write"** strategy is adopted.

At the end of each iteration in **`01_Execution_Loop`**:

1.  **Event Processing Complete:** LLM thinking and tool execution are finished.
2.  **State Update:** The Core updates the `history` and `variables` in memory.
3.  **Trigger Archive:** The Core calls `await storage.save(current_snapshot)`.
4.  **Respond Externally:** **Only after `save` returns successfully** does the Core send the response to the UI or execute the next Action.
    *   *This ensures that if the system crashes before responding, the latest execution result is not lost upon restart, and already-saved but unresponded steps are not repeated (requires idempotent design).*

---

## 4. Distinguishing "State" and "Memory"

It is crucial to strictly distinguish these two concepts within the architecture to avoid confusion:

| Feature | **State (Persistence)** | **Memory (RAG)** |
| :--- | :--- | :--- |
| **Purpose** | **Survival** (Crash Recovery) | **Knowledge** (Knowledge Retrieval) |
| **Stored Content** | Full context stack, variables | Processed documents, experience summaries |
| **Storage Location** | `storage` plugin (Redis/SQL) | `RAG Engine` (Vector DB) |
| **Lifecycle** | Deletable upon task completion (Ephemeral) | Permanent storage (Long-term) |
| **Ownership** | **Managed directly by Agent Core** | **External service (Supporting Engine)** |

---

## 5. Summary

By introducing the `storage` plugin interface:

1.  **The Core remains pure:** It does not need to depend on specific database drivers, only needing to call `save()` / `load()`.
2.  **Extreme flexibility:** Users can seamlessly switch the persistence layer from local files to cloud databases by swapping plugins, without modifying core code.
3.  **High reliability:** Coupled with the Daemon's restart mechanism, the system achieves "never-forgetting" robustness.
