# OpenStarry: The Agent Operating System

**OpenStarry** is a core architecture that redefines how AI Agents are built. Inspired by modern operating system design philosophy and integrated with Eastern "Five Aggregates" concepts, it aims to create a highly modular, secure coordination layer for agents with anthropomorphic characteristics.

We don't just build Chatbots; we build **Operating Systems for Digital Species**.

---

## 🏗️ Macro-System Architecture

OpenStarry employs a three-layer progressive architecture, simulating the symbiotic relationship between biological organisms and their environment:

### 1. Management Zone (Agent Coordination Layer)
**Positioning: Host environment and administrative hub.**
Responsible for providing soil and nutrients. This layer ensures stability and security, including container isolation (Plumbing), causal-chain-based event scheduling (Orchestration), safety policies (Policy), and Hardware Abstraction Layers (HAL). It converts physical world signals into data streams understandable by Agents.

### 2. Agent Core (Autonomous Life Zone)
**Positioning: Pure "Five Aggregates" computation loop.**
A "Headless" and "Stateless" life kernel. Its sole responsibility is to maintain the computation loop of "Sensation, Perception, Volition, and Consciousness." The Core is essentially empty; it exhibits different life forms through various plugins.

### 3. Capability Plugins
**Positioning: Functional components giving Agents personality, expertise, and soul.**
Plugins define the Agent's boundaries. They include communication protocols, self-reflection mechanisms, and state memory plugins. This allows the same Core to transform from a "Code Expert" to an "Equipment Monitor" instantly.

---

## 🔄 The Lifecycle (Causal Chain)

In OpenStarry, task execution is seen as the arising and cessation of life:
1. **Origination**: The environment layer detects a requirement.
2. **Scheduling**: The management layer matches required plugins based on needs.
3. **Arising**: The container layer loads the core and dynamically injects capabilities.
4. **Operation**: The core processes "Pain" to achieve goals.
5. **Cessation**: Task completion; experience saved to memory; instance destroyed.

```mermaid
graph TD
    subgraph Host [🛡️ Management Zone (Host Environment)]
        direction TB
        Orchestrator[Orchestration] --> Container[Container]
        Policy[Safety Policy] -.-> Container
        HAL[HAL] --> InputFlow((Perception Flow))
    end

    subgraph Runtime [⚡ Running Instance]
        direction LR
        InputFlow --> Core
        
        subgraph Core [🧠 Agent Core (Microkernel)]
            Loop[Execution Loop]
            State[State Machine]
            Interceptor[Anomaly Interceptor]
        end

        Core --> |1. Load| Plugins
        
        subgraph Plugins [🔌 Capability Plugins (The 5 Aggregates)]
            Guide[Consciousness: Guide]
            Tool[Volition: Tools]
            LLM[Perception: Provider]
            Mem[Memory: Memory]
            Pain[Reflex: Pain]
        end
        
        Plugins --> |2. Inject| Core
        Interceptor -.-> |3. Pain Signal| Guide
        Guide -.-> |4. Correction| Loop
    end
```

---

## 💻 The Shape of an Agent (Core Config)

The power of OpenStarry lies in its declarative configuration. Here is a standard Agent definition with "Pain" and "File System" capabilities:

```jsonc
// agent.json
{
  "identity": { "id": "dev-bot-01", "name": "Resilient Developer" },
  "plugins": [
    // [Perception] Brain: Inject cognitive engine
    { "name": "@openstarry-plugin/provider-gemini" },
    
    // [Volition] Limbs: Inject file system operations
    { "name": "@openstarry-plugin/standard-function-fs" },
    
    // [Sensation] Senses: Listen to terminal input
    { "name": "@openstarry-plugin/standard-function-stdio" },
    
    // [Consciousness] Soul: Inject pain mechanism (defines how to face errors)
    { "name": "@openstarry-plugin/guide-pain-mechanism" }
  ],
  "policy": {
    // Management rules: Halt after 3 consecutive errors
    "safety": { "max_consecutive_errors": 3 } 
  }
}
```

---

## 🌟 The Ten Tenets

### 1. Agent as OS Process
Agents are not one-off scripts but digital entities with persistent lifecycles, managed, monitored, and restarted by Daemons. They have PIDs and states, behaving like living processes.

### 2. Everything is a Plugin
Every organ of the system is replaceable. Tools, listeners, LLM brains, and even memory strategies are plugins. The Core is just an empty power strip; all capabilities are external mounts.

### 3. Five Aggregates Architecture
The system design integrates Eastern philosophy. **The Core is essentially "Sunyata" (Empty).** Its life characteristics are granted by five types of plugins:
*   **Form (UI)**, **Sensation (Listener)**, **Perception (Provider)**, **Volition (Tool)**.
*   **Consciousness (Guide):** The most critical component. Guide Plugins inject memory and persona, giving the Core "Self-awareness" (Vijnana). Without a Guide, the Core is just unconscious computing power.

### 4. Directory as Protocol
Whether system or project, local disk or USB, as long as the directory structure follows `plugins/` and `configs/` standards, the system recognizes and loads it automatically. Physical structure maps directly to runtime logic.

### 5. Directory as Permission
System and project layers use isomorphic design but strict permission isolation. Plugin location determines visibility; Agent execution location determines permission boundaries.

### 6. Anthropomorphic Cognitive Flow & Pain
Errors are transformed into "Pain" (Negative Feedback). Built-in feedback loops inject runtime errors into the Context, forcing the Agent to reflect and self-correct, simulating trial-and-error learning.

### 7. Microkernel & Absolute Purity
The Agent Core adopts a strict **Microkernel Architecture**.
*   **Isolation:** The compiled Core binary **must not contain any plugin code**.
*   **Purity:** The Core only depends on abstract interfaces (SDK). All capabilities are injected at runtime.
*   **Headless:** The kernel is decentralized and independent of specific UI or IO devices. The "Soul" can be ported to any "Body" (CLI, Web, Docker, IoT).

### 8. Control-Theoretic Loop Model
Not just an execution loop, but a control loop. User goals are reference inputs, Context is state feedback, and Tool Calls are control variables. An Agent is an intelligent controller minimizing "Goal vs. Reality" error.

### 9. Pluggable Context Strategy
Memory management is no longer hard-coded. Developers can dynamically change memory strategies (sliding window, dynamic summary, state extraction) based on role requirements.

### 10. Fractal Social Structure
The system is self-similar. A complex Agent can be composed of multiple sub-Agents, exposing a unified MCP interface. This allows building infinite-level collaboration networks.

---

## 📚 Documentation Map

### 1. Architecture Documentation
*   [00_Design Philosophy](./Architecture_Documentation/00_OpenStarry_Design_Philosophy.md)
*   [01_Architecture Overview](./Architecture_Documentation/01_Architecture_Overview.md)
*   [02_Headless Agent Core](./Architecture_Documentation/02_Headless_Agent_Core.md)
*   ... (Refer to the TW README for full list)

### 2. Core Components Deep Dive
*   [00_Core Philosophy](./Agent_Core_Components_Deep_Dive/00_Core_Philosophy.md)
*   [01_Execution Loop](./Agent_Core_Components_Deep_Dive/01_Execution_Loop.md)
*   ... (Refer to the TW README for full list)

### 3. Project Structure & Conventions
*   [00_Roadmap & Milestones](./Project_Structure_and_Conventions/00_Roadmap_and_Milestones.md)
*   ... (Refer to the TW README for full list)

---

## 🛠️ Getting Started

Ready to start? See **[Developer_Guide_Standalone_Execution.md](./Implementation_Examples/Developer_Guide_Standalone_Execution.md)** to run your first Agent.