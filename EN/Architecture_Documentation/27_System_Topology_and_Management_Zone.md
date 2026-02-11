# 27. System Topology & Management Zone

This document defines the macro-operational logic of OpenStarry: it is a modern AI operating system where the "Management Zone" provides the soil and nutrients, allowing the "Core Zone" to achieve species diversity through "Plugins."

---

## 1. Agent Coordination Layer (The Management Zone)

**Positioning: The system's host environment and administrative hub.**
This layer is not responsible for "thinking"; it is responsible for "ensuring the stability and security of the thinking environment."

### 1.1 Container Layer (Plumbing)
*   **Mechanism**: Employs dynamic linking or containerization technologies (e.g., WebAssembly sandboxing or specific process isolation).
*   **Responsibility**: Implements `AgentLoader`. When the system requires a "Developer," the container layer pulls the `AgentCore` image from the registry and mounts specified `Plugins` via **Dependency Injection (DI)**.

### 1.2 Orchestration Layer (Planning & Scheduling)
*   **Mechanism**: An event-driven architecture based on a **Causality Chain**.
*   **Responsibility**: It does not monitor an Agent's private internal state; instead, it monitors **"Boundary Events."** When Agent A emits a `TaskCompleted` event, the orchestration layer determines if this is the "condition (缘)" to awaken Agent B.

### 1.3 Security & Policy Layer (Policy)
*   **Mechanism**: Interceptors and Quota Management.
*   **Responsibility**: Enforces "precepts (戒律)." For example, restricting network access for a test Agent or enforcing Token usage budgets for a development Agent.

### 1.4 Environment Layer (Resources)
*   **Mechanism**: Hardware Abstraction Layer (HAL).
*   **Responsibility**: Transforms physical entities (e.g., PiKVM visuals, NVIDIA Jetson temperatures) into standardized **sensory data streams** for the Agents.

### 1.5 Interface Layer (Multimodal Interaction)
*   **Mechanism**: State Projection.
*   **Responsibility**: Transforms invisible Agent thought processes (exported data from protocol plugins) into human-readable dashboards.

---

## 2. Agent Core (The Autonomous Life Zone)

**Positioning: A pure "Five Aggregates" computational cycle.**
It is empty; its sole responsibility is to **maintain the operation of the cycle.**

### Five Aggregates Layer
*   **Sensation (Input)**: Receives signals from the environment layer.
*   **Perception (Reasoning)**: Invokes large models for semantic analysis.
*   **Volition (Action)**: Generates intentions to operate tools.
*   **Consciousness (Integrator)**: Integrates information from all plugins to maintain self-continuity.

**Core Property**: It is **Stateless**. All state resides in "Memory Plugins," and all communication specifications depend on "Protocol Plugins." This means the Core can migrate losslessly between different containers.

---

## 3. Capability Plugins (The Plugins)

**Positioning: Functional components that endow the Agent with personality, expertise, and soul.**

### 3.1 Internal Protocol Plugins (Data & Protocol)
*   **Engineering Detail**: Defines the Internal Message Bus.
*   **Applications**:
    *   **High-speed Mode**: Developer Agents use the Protobuf protocol for rapid code generation and transmission.
    *   **Transparent Mode**: Tester Agents use a "Log-tracing Protocol" to record every second of neural conduction for causal backtracking during failures.

### 3.2 Reflection Plugins (Evaluation & Evolution)
*   **Engineering Detail**: Implements a "dual-layer reasoning" mechanism. Actions are intercepted by the reflection plugin for a "self-check" before output.
*   **Applications**:
    *   **Lightweight**: Simple document organization Agents can omit this plugin to reduce latency.
    *   **Deep Decision-making**: Agents involved in factory equipment operation must mount reflection plugins with "physical rule validation" to prevent hazardous actions.

### 3.3 Memory Plugins (State & Memory)
*   **Engineering Detail**: Abstracted storage interface `IMemoryStore`.
*   **Applications**:
    *   **Short-term Battlefield**: Uses `In-Memory` plugins, allowing developers extremely fast context switching when writing current functions.
    *   **Long-term Experience**: Mounts `Vector-RAG` plugins, allowing the Agent to remember experiences in solving similar bugs from three months ago.

---

## 4. The Ultimate Operational Workflow (The Lifecycle)

This is a complete lifecycle from "Origination" to "Cessation":

1.  **Origination**: The environment layer detects a hardware error.
2.  **Scheduling**: The orchestration layer determines a "Maintenance Expert" is needed and notifies the container layer.
3.  **Arising**: The container layer loads the **Agent Core** and attaches three plugins: **Diagnostic Protocol**, **High-level Reflection**, and **Equipment Knowledge Base Memory**.
4.  **Operation**: Supported by the plugins, the Agent Core begins processing "pain signals" and generates repair instructions.
5.  **Cessation**: Task complete; the container layer destroys the Core instance, and the memory plugin stores the experience back to the database.
