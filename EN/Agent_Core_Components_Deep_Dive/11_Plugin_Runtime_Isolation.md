# 11. Plugin Runtime Isolation & Sandboxing

This document defines how the OpenStarry system isolates plugin execution environments to prevent malicious or buggy plugins from causing Core crashes, data leaks, or resource exhaustion.

## 1. Risk Model

*   **Stability Risks**: Plugins throwing uncaught exceptions that cause the main process to exit.
*   **Resource Risks**: Plugins writing infinite `while(true)` loops or causing memory leaks.
*   **Security Risks**: Plugins reading API Keys from environment variables or deleting files from the filesystem arbitrarily.

## 2. Isolation Levels

Depending on deployment environments and security requirements, the system supports four levels of isolation.

### Level 1: Function Wrapping [Development/Lightweight Mode]
*   **Mechanism**: All plugin code runs within the main process (Agent Core).
*   **Protection**:
    *   **Try-Catch**: `PluginLoader` forcibly wraps plugin `execute` methods in `try-catch` blocks.
    *   **Timeout**: Uses `Promise.race` to set execution timeouts (e.g., 10 seconds), throwing an error upon timeout.
*   **Drawbacks**: Cannot defend against `while(true)` loops (which freeze the Event Loop) and cannot restrict filesystem access.
*   **Use Case**: Internal trusted plugins, development and debugging phases.

### Level 2: VM Sandboxing [Standard Mode]
*   **Mechanism**: Uses Node.js's `vm` module or the `vm2` library to run plugin code.
*   **Protection**:
    *   **Context Isolation**: Plugins cannot access the global `process` object or read environment variables.
    *   **Limited Access**: Only explicitly injected APIs (e.g., `log`, `fetch`) can be called.
*   **Drawbacks**: Still resides in the same process; cannot fully defend against CPU-intensive attacks.
*   **Use Case**: Running third-party code where trustworthiness is moderate.

### Level 3: Process Isolation [Strict/Production Mode]
*   **Mechanism**: The **Orchestrator Daemon** is responsible for launching independent OS child processes or Worker Threads for each plugin (or group of plugins).
*   **Communication**: Core and Plugin communicate via IPC (Inter-Process Communication) or standard I/O (stdio) using JSON messages.
*   **Protection**:
    *   **Resource Quotas**: The Daemon can use OS mechanisms (cgroups, Docker) to limit the plugin process's CPU/RAM usage.
    *   **Physical Circuit Breaker**: Plugin deadlocks do not affect the Core; the Daemon can directly `kill` the plugin process.
*   **Drawbacks**: Higher performance overhead; IPC communication latency.
*   **Use Case**: Executing high-risk code (e.g., Code Interpreter) or multi-tenant environments.

### Level 4: WebAssembly (WASM) [Future Evolution/Extreme Security]
*   **Mechanism**: Plugins are compiled into WASM modules and executed within a WASM runtime (e.g., Wasmtime) embedded in the Core.
*   **Advantages**:
    *   **Memory Safety**: Plugins cannot access memory space outside the sandbox.
    *   **Capability-based Authorization**: Extremely fine-grained control over plugin file and network access via WASI interfaces.
    *   **Ultra-fast Startup**: Millisecond-level cold starts, far faster than process isolation.
*   **Use Case**: High-performance computing tools, untrusted third-party business plugins.

---

## 3. Recommended Implementation Path

### MVP Phase
Adopt **Level 1 (Try-Catch + Timeout)**.
*   Sufficient for most non-malicious bugs.
*   Lowest development cost.

### V1.0 Production Phase
For **Code Interpreters** and **third-party plugins**, **Level 3 (Independent Process)** must be adopted.
*   `Infrastructure Plugins` (e.g., local Brokers) are essentially manifestations of Level 3, managed independently by the Daemon.
*   Standard `Tool` plugins involving sensitive operations should be executed independently via a `Plugin Runner`.

---

## 4. Permission Manifest

Each plugin's `plugin.json` must declare required permissions, to be authorized by the user upon installation (similar to Android Apps).

```json
{
  "name": "FileSearchTool",
  "permissions": [
    "fs:read:./data",  // Allow reading the data directory
    "network:none"     // Prohibit network access
  ]
}
```

When loading a plugin, the Core builds a restricted execution context based on these declarations.
