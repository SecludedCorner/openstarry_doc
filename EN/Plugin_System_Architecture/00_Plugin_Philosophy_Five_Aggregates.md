# 00. Plugin Philosophy: Plugin as Five Aggregates

> **[Important Note]** This document describes the **architectural theoretical mapping** behind the OpenStarry plugin system.

## Core Philosophy

**A plugin is not a type; it is an aggregation of capabilities.**

Just as a living organism is an aggregation of the Five Aggregates (Pañcaskandha), a fully functional plugin often requires multiple components simultaneously. OpenStarry's plugin system allows a single plugin package to provide any combination of these components.

---

## Defining the Aggregates

Within `PluginHooks`, we use semantic fields to declare these components, which correspond strictly to the Five Aggregates in our architecture:

```typescript
// PluginHooks Definition (from @openstarry/sdk)
interface PluginHooks {
  ui?: IUI[];          // Form (Rupa) - Interface and manifestation; receives events and presents output.
  listeners?: IListener[]; // Sensation (Vedana) - Sensory monitoring; receives external input.
  providers?: IProvider[]; // Perception (Samjna) - Cognitive brain; LLM adapter.
  tools?: ITool[];     // Volition (Samskara) - Execution tools; files/APIs/code.
  guides?: IGuide[];   // Consciousness (Vijnana) - Soul guidance and skills; system prompt.
  commands?: SlashCommand[];
  dispose?: () => Promise<void> | void;
}
```

## Common Aggregation Patterns

### 1. Pure Volition Plugin (Pure Executor)
*   **Components:** Only `tools`.
*   **Example:** `fs-utils`.

### 2. Full Sensory Plugin
*   **Components:** `listeners` + `tools`.
*   **Example:** `discord-connector` (can both receive and send messages).

### 3. Soul Injector Plugin
*   **Components:** `guides` (potentially with specific `tools`).
*   **Example:** `expert-coder-skill`.
    *   **Consciousness (Guide):** Injects an engineer persona.
    *   **Volition (Tool):** Bundles a specific `linter` tool.

---

## Architectural Advantages

1.  **High Cohesion:** Related capabilities are kept within the same package.
2.  **Atomic Loading:** A user installs one npm package and acquires full domain capabilities.
3.  **Philosophical Consistency:** This aligns perfectly with the Core's Five Aggregates architecture. The Core is the container, while the Plugin is the provider of the aggregates.
