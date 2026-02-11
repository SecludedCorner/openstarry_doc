# 04. Plugin Infrastructure

This document defines the plugin system architecture of OpenStarry. We adopt the **Aggregate Pattern**, allowing a single plugin package to provide multiple capabilities simultaneously.

> **Architectural Note:**
> This architecture deeply integrates the "Five Aggregates" philosophy, defining five basic forms of plugins:
> 1.  **UI (Form / Rupa):** Physical manifestation and interface.
> 2.  **Listener (Sensation / Vedana):** Interface for perception (Input).
> 3.  **Provider (Perception / Samjna):** Engine for cognition (Processing).
> 4.  **Tool (Volition / Samskara):** Capability for execution (Output/Action).
> 5.  **Guide (Consciousness / Vijnana):** Guidance of the soul (Identity/Protocol/Logic). The Core itself is an empty container; it is the Guide that endows it with "recognition" and "self-awareness."

---

## Plugin Structure Specification

In OpenStarry, a **Plugin** is a functional unit that can contain any combination of the following components.

### `plugin.json` Definition

```json
{
  "id": "openstarry-full-stack",
  "components": {
    "ui": { ... },        // [Form]
    "listeners": [ ... ], // [Sensation]
    "providers": [ ... ], // [Perception]
    "tools": [ ... ],     // [Volition]
    "guides": [           // [Consciousness]
      { "name": "expert-persona", "entry": "./skills/expert.md" },
      { "name": "mcp-protocol", "entry": "./protocols/mcp-logic.js" }
    ]
  }
}
```

---

## Plugin Loading Mechanism

When the `PluginLoader` loads a plugin, it performs a process of "Destructuring and Registration":

1.  **Read Manifest:** Reads `plugin.json`.
2.  **Component Destructuring:**
    *   Scans `components.tools` and registers them into the **ToolRegistry**.
    *   Scans `components.listeners`, registers them into the **ListenerRegistry**, and starts them.
    *   Scans `components.providers` and registers them into the **ProviderRegistry**.
3.  **Dependency Injection:** Injects required Core APIs into each component.

This means that developers publish plugins in the form of "functional packages" (e.g., `npm install @openstarry-plugin/whatsapp`), and the Core automatically disassembles and assigns the limbs, ears, and eyes within them.

### Dynamic Loading Replaces Hardcoded Factories

In earlier versions, the core hardcoded the loading logic for built-in plugins via a `BUILTIN_FACTORIES` map. This method has been completely removed and replaced by a unified **Two-Tier Resolution**:

1.  **`ref.path` Priority:** If a plugin reference in `agent.json` contains a `path` field (pointing to a local filesystem path), the Loader loads the plugin module directly from that path.
2.  **`import(ref.name)` Fallback:** If there is no `path`, the Loader uses `import(ref.name)` to dynamically load based on the NPM package name (e.g., `import('@openstarry-plugin/fs')`).

This unified mechanism eliminates the boundary between "built-in plugins" and "third-party plugins," with all plugins enjoying the same loading path and lifecycle management.

---

## The Global Plugin Registry

Before these capabilities enter the Core's internal registries, they must first undergo discovery and indexing by the coordination layer (Daemon). This is a system-level service.

### 1. Responsibilities

*   **Scanning and Discovery:** Scans `~/.openstarry/plugins/` (system level) and `./.openstarry/plugins/` (project level) upon startup.
*   **Indexing:** Reads the manifests of all plugins to build a mapping table of `Plugin ID -> Physical Path`.
*   **Capability Resolution:** When the design layer queries "Who has the `send_message` capability?", the registry is responsible for returning the corresponding list of plugins.

### 2. Discovery Process

1.  **Startup:** The Daemon starts the `PluginRegistryService`.
2.  **Traversal:** The service traverses all known paths looking for `package.json` or `plugin.json`.
3.  **Registration:** Metadata of valid plugins is stored in an in-memory database.
4.  **Provision:** When a specific Agent starts, the Loader requests the path of the specified plugin from this service before proceeding with the load.

---

## Standard Capability Registries (The Registry)

The Core maintains several global registries internally:

*   `core.registry.tools`: Stores all available tool functions.
*   `core.registry.listeners`: Manages all active listener instances.
*   `core.registry.providers`: Stores available LLM backends.

This design ensures engineering intuitiveness while maintaining the integrity of the Five Aggregates architecture at the logical level.