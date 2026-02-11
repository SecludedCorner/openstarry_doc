# 04. Agent Manifest Specification

This document defines the data structure of `agent.json`. This is the core configuration file for every Agent instance, located at `[Agent_Root]/configs/agent.json`.

## Core Philosophy: Static Definition of Identity and Capabilities

`agent.json` is read by the Core upon Agent startup. It is **immutable** (cannot be modified at runtime), ensuring that Agent behavior is predictable and auditable.

---

## Structure Definition

```json
{
  "$schema": "./schemas/agent-manifest.schema.json",
  "identity": {
    "id": "log-analyst-01",
    "name": "Log Analysis Bot",
    "role": "data_engineer",
    "description": "Responsible for analyzing server logs and generating reports."
  },
  
  "cognition": {
    "system_prompt": "configs/prompts/system.md", // Points to an external file or an inline string
    "provider": {
      "id": "gemini-pro-adapter",
      "config": {
        "model": "gemini-1.5-pro",
        "temperature": 0.2
      }
    },
    "context_strategy": {
      "type": "summarizer", // References a strategy plugin ID
      "config": {
        "threshold": 20
      }
    }
  },

  "capabilities": {
    "plugins": [
      // References the full package names of plugins; Core searches following inheritance rules (Project priority -> System)
      "@openstarry-plugin/standard-function-fs",
      "@openstarry-plugin/standard-function-stdio",
      "project-log-parser"
    ],
    "permissions": {
      "fs_allow_paths": ["./logs", "./reports"], // Sandbox path restrictions
      "network_allow_hosts": ["github.com"]
    }
  },

  "policy": {
    "max_steps": 50, // Prevents infinite loops
    "circuit_breaker": {
      "error_threshold": 5
    }
  }
}
```

---

## Field Details

### 1. `identity`
*   **id:** A globally unique identifier used for log tracing and Daemon management.
*   **role:** Used for addressing during multi-agent collaboration (e.g., "Find a `data_engineer` to assist").

### 2. `cognition` (Perception Configuration)
*   **system_prompt:** Defines the Agent's core persona and responsibilities. Supports references to external Markdown files for easier authoring of long texts.
*   **provider:** Specifies which LLM acts as the brain.

## 3. Plugin Configuration

Plugins loaded by the Agent may contain multiple components; these can be finely configured within `agent.json`.

```json
"plugins": {
  "@openstarry-plugin/standard-function-fs": {
    "tools": { "read": true, "write": false } // Restricts Volition
  },
  "@openstarry-plugin/standard-function-skill": {
    "guides": { "autoload": true } // Configures Consciousness
  }
}
```

### Component Types

*   **tools (Volition):** Specific functions for execution.
*   **listeners (Sensation):** Event listeners.
*   **providers (Perception):** AI model backends.
*   **ui (Form):** Interface components.
*   **guides (Consciousness):** Skills, workflows, and protocol logic.

### 4. `policy` (Superego Configuration)
*   **max_steps:** Maximum execution steps for a single task, preventing resource exhaustion.
*   **circuit_breaker:** Defines when to trigger "forced cooling" or "request for help."
