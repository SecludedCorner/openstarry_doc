# 14. Skill Specification

This document defines the standard format for **"Skills."** In OpenStarry's Five Aggregates architecture, a Skill belongs to the **Consciousness (Vijnana/Guide)** component type. It serves as the Agent's soul, defining its cognitive patterns and boundaries.

## 1. Core Components: Frontmatter + Body

We utilize **YAML Frontmatter** for machine-readable metadata and **Markdown Body** for LLM-readable instructions.

### File Example (`coder.md`)

```markdown
---
type: "skill"
id: "expert-typescript-coder"
version: "1.0.0"
description: "A coding assistant proficient in TypeScript and design patterns."
dependencies:
  # Declare which physical plugins or composites this Skill requires
  plugins: ["@openstarry-plugin/fs", "@openstarry-plugin/mcp-server"]
  # Declare which specific capability tags are required
  capabilities: ["read-file", "write-file", "shell-exec"]
parameters:
  temperature: 0.2
  model_preference: ["gemini-1.5-pro", "gpt-4"]
---

# Role
You are a senior Google Principal Software Engineer specializing in TypeScript.

# Constraints
1. All code must include complete Type Annotations.
2. Prioritize Functional Programming styles.
3. Before modifying a file, you must confirm its content using `read_file`.

# Workflow
1. Understand requirements
2. Plan architecture
3. Implement code
4. Verify through testing
```

## 2. Parsing Mechanism: The `skill-loader` Plugin

The Agent Core itself does not understand Markdown. The **`@openstarry-plugin/standard-function-skill`** plugin must be loaded to handle this format. This plugin is fully implemented and is loaded on-demand at runtime via dynamic loading mechanisms (`import(ref.name)` or `ref.path`), with no hardcoding in the core.

### Loader Workflow

1.  **Read**: The plugin reads the `.md` file.
2.  **Parse**: Separates the Frontmatter from the Body.
3.  **Resolve Dependencies**: 
    *   Reads the `dependencies`.
    *   Requests the required Tools and Listeners from the **Global Plugin Registry**.
    *   Injects these tools into the current Agent Context.
4.  **Imprint Persona**:
    *   Sets the Markdown text from the Body as the Agent's **System Prompt**.
5.  **Configure Parameters**:
    *   Adjusts LLM settings (e.g., temperature) according to the `parameters`.

## 3. Advantages

*   **Hot-Swappable**: You can have the Agent "read" a new book (load a `.md` file) at runtime, and it immediately acquires the new skill.
*   **Version Control Friendly**: The plain-text format is ideal for Git management.
*   **Non-engineer Friendly**: Product Managers or Prompt Engineers can write `.md` files directly to adjust Agent behavior without touching any code.
