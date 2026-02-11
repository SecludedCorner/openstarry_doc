# 02. Core Source Code Structure Specification

This document defines the organization of code within `packages/core/src/`. This structure aims to achieve highly decoupled, testable, and anthropomorphic architectural design goals.

## Source Code Directory Tree

```text
src/
├── agents/             # Agent entities
├── execution/          # Execution, scheduling, and event flow (The Loop)
├── memory/             # Context management and memory strategies (Working Memory)
├── infrastructure/     # Plugin loading, registration, and isolation (Plugin Infra)
├── security/           # Safety interception and circuit breakers (Guardrails)
├── state/              # State snapshots and persistence (State Manager)
├── transport/          # Core communication bridge layer
├── types/              # TypeScript strong type definitions
└── index.ts            # Library entry point
```

---

## Module Responsibility Details (with Philosophical Mapping)

### 1. `execution/` (Execution System) —— [Mapping: Volition (Samskara) / Consciousness (Vijnana)]
The central life force of the Agent, implementing the "execution loop" and "control theory" models.
*   **`loop/`**: Implements the `tick()` mechanism, responsible for retrieving items from the event queue, invoking the LLM, and triggering actions (will and formation).
*   **`queue/`**: Manages asynchronous event priorities.

### 2. `memory/` (Working Memory) —— [Mapping: Perception (Samjna)]
Implements the logic defined in `10_Context_Management_Strategy`.
*   **`context/`**: Responsible for identifying, correlating, and compressing received information (cognition and pattern recognition).

### 3. `infrastructure/` (Plugin Management) —— [Mapping: Form (Rupa)]
Implements the physical loading for the "Everything is a Plugin" principle.
*   **`loader/` & `registry/`**: Responsible for scanning and storing the Agent's "body parts." When a plugin is loaded, its specific functionalities (Form) are disassembled and registered into the system.

### 4. `security/` (Security Layer) —— [Mapping: Sensation (Vedana) - Defensive Aspect]
The firewall positioned between "decision" and "execution."
*   **`guardrails/`**: Responsible for capturing exceptions and transforming them into the Agent's "negative sensations (pain)," thereby triggering self-correction.

### 5. `transport/` (Communication Bridge) —— [Mapping: Sensation (Vedana) - Perceptual Aspect]
Implements the APIs defined in `02_Communication_Interface`.
*   **`bridge/`**: Responsible for interfacing with external stimuli (listeners for eyes, ears, body, etc.) and transforming external information into internal sensations.

---

## Naming Conventions

1.  **Classes:** Use PascalCase, e.g., `AgentCore`, `ExecutionLoop`.
2.  **Interfaces:** Prefix with `I`, e.g., `IPlugin`, `ITool`.
3.  **File Suffixes:** 
    *   Logic implementation: `.ts`
    *   Unit tests: `.test.ts` (placed in the same directory as the source file).
4.  **Entry Points:** Each directory should contain an `index.ts` to export public APIs, keeping external references concise.

---

## Code Evolution Principles

*   **No Side Effects**: Except for `infrastructure/` and `transport/`, modules should strive to remain pure logic, avoiding direct manipulation of the network or filesystem.
*   **Dependency Injection**: The `AgentCore` class should receive instances of `IContextManager` and `IStateManager` upon initialization, allowing for easy replacement with Mock versions during testing.
