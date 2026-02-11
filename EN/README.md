# OpenStarry: The Agent Operating System

**OpenStarry** is a core architecture that redefines how AI agents are built. Drawing on the design philosophy of modern operating systems and integrating the Eastern concept of the "Five Aggregates," it aims to create a highly modular, secure, and anthropomorphic agent coordination layer with lifelike characteristics.

We don't just build chatbots — we build **an operating system for digital species**.

---

## Macro-System Architecture

OpenStarry employs a three-tier progressive architecture that mirrors the symbiotic relationship between living organisms and their environment:

### 1. Agent Coordination Management Layer (Management Zone)
**Role: The host environment and administrative hub.**
Responsible for providing the soil and nutrients. This layer ensures environmental stability and security, encompassing container isolation (Plumbing), causal-chain-based event scheduling (Orchestration), security policies (Policy), and a Hardware Abstraction Layer (HAL). It translates physical-world signals into data streams that agents can understand.

### 2. Agent Core (Autonomous Life Zone)
**Role: A pure Five Aggregates computation loop.**
It is a "headless" and "stateless" life kernel. Its sole responsibility is to maintain the computation loop of Sensation, Perception, Volition, and Consciousness. The Core is inherently empty — under different plugin configurations, it manifests different life forms.

### 3. Capability Plugins Layer
**Role: Functional components that give agents personality, expertise, and soul.**
Plugins define an agent's capability boundaries. They include communication protocols (Protocol), self-reflection (Reflection), and state memory (Memory) plugins. This allows the same Core to transform from a "coding expert" into a "device monitor" at any time.

---

## The Causal Lifecycle

In OpenStarry, executing a task is viewed as the arising and ceasing of a life:
1. **Origination**: The environment layer detects a need.
2. **Scheduling**: The management layer matches the required plugins to the need.
3. **Arising**: The container layer loads the core and dynamically injects capabilities.
4. **Operation**: The core processes "pain signals" and works toward the goal.
5. **Cessation**: Task complete, experience stored to memory, instance dissolved.

```mermaid
graph TD
    subgraph Host [Management Zone (Host Environment)]
        direction TB
        Orchestrator[Scheduling Layer] --> Container[Container Layer]
        Policy[Security Policy Layer] -.-> Container
        HAL[Hardware Abstraction Layer] --> InputFlow((Perception Flow))
    end

    subgraph Runtime [Running Instance]
        direction LR
        InputFlow --> Core

        subgraph Core [Agent Core (Microkernel)]
            Loop[Execution Loop]
            State[State Machine]
            Interceptor[Anomaly Interceptor]
        end

        Core --> |1. Load| Plugins

        subgraph Plugins [Capability Plugins (The 5 Aggregates)]
            Guide[Consciousness: Guide]
            Tool[Volition: Tools]
            LLM[Perception: Provider]
            Mem[Memory]
            Pain[Pain Reflex]
        end

        Plugins --> |2. Inject| Core
        Interceptor -.-> |3. Pain Signal| Guide
        Guide -.-> |4. Correction| Loop
    end
```

---

## Core Configuration Example (The Shape of an Agent)

OpenStarry's power lies in its declarative configuration. Below is a standard agent definition with "pain mechanism" and "file operation" capabilities:

```jsonc
// agent.json
{
  "identity": { "id": "dev-bot-01", "name": "Resilient Developer" },
  "plugins": [
    // [Perception] Brain: inject cognitive engine
    { "name": "@openstarry-plugin/provider-gemini" },

    // [Volition] Hands & Feet: inject filesystem operation capabilities
    { "name": "@openstarry-plugin/standard-function-fs" },

    // [Sensation] Senses: listen to terminal input
    { "name": "@openstarry-plugin/standard-function-stdio" },

    // [Consciousness] Soul: inject pain mechanism (defines how to handle errors)
    { "name": "@openstarry-plugin/guide-pain-mechanism" }
  ],
  "policy": {
    // Management layer rule: 3 consecutive errors triggers a physical circuit breaker
    "safety": { "max_consecutive_errors": 3 }
  }
}
```

---

## The Ten Tenets

### 1. Agent as OS Process
An agent is not a one-off script — it is a digital entity with a persistent lifecycle, manageable by a daemon, monitorable, and restartable. It has its own PID, its own state, like a living process.

### 2. Everything is a Plugin
Every organ of the system is replaceable. Tools are plugins, listeners are plugins, the LLM brain is a plugin — even memory strategies and communication protocols are plugins. The Core is just an empty socket board; all capabilities come from external mounting.

### 3. Five Aggregates Architecture
The system design deeply integrates Eastern philosophy. **The Core is inherently an "empty" (Sunyata) container.** Its life characteristics are entirely bestowed by five types of plugins (the Five Aggregates):
*   **Form (UI)**, **Sensation (Listener)**, **Perception (Provider)**, **Volition (Tool)**.
*   **Consciousness (Guide):** This is the most critical component. The Guide Plugin injects memory and persona, granting the Core "self-awareness" (Vijnana). Without a Guide, the Core is mere unconscious computing power.

### 4. Directory as Protocol
Whether it's the system or a project, whether on a local hard drive or a USB device — as long as the directory structure conforms to the standard `plugins/`, `configs/` conventions, the system can automatically discover and load it. Physical structure directly maps to runtime logic.

### 5. Directory as Permission
System-level and project-level directories share a uniform design, but permissions are strictly isolated. A plugin's placement determines its visibility scope; an agent's running location determines its permission boundaries. System administrators cannot directly tamper with business plugins, ensuring security isolation.

### 6. Anthropomorphic Cognitive Flow & Pain
Errors are transformed into the agent's "pain" (Negative Feedback). The system has a built-in feedback loop that injects runtime errors into the Context, compelling the agent to self-reflect and self-correct after failures — simulating biological trial-and-error learning.

### 7. Microkernel & Absolute Purity
The Agent Core follows a strict **microkernel architecture**.
*   **Physical Isolation:** The compiled Core binary is **strictly prohibited from containing any plugin code**.
*   **Absolute Purity:** The Core depends only on abstract interfaces (SDK) and has no concrete capabilities of its own. All capabilities must be dynamically injected at runtime through external plugins.
*   **Headless Design:** The kernel is decentralized, depending on no specific UI or IO device. This ensures the agent's "soul" can be transplanted into any "body" — from CLI to web, from Docker to IoT devices.
*   **Significance:** No built-in code means no built-in bugs.

### 8. Control-Theoretic Loop Model
It's not just an execution loop — it's a control loop. The system treats the user's goal as a reference input, the Context as state feedback, and Tool Calls as control variables. An agent is fundamentally an intelligent controller that continuously minimizes the "error between goal and current state."

### 9. Pluggable Context Strategy
Memory management is no longer hardcoded logic. Developers can dynamically swap memory strategies (sliding window, dynamic summarization, state extraction) based on the agent's role requirements, flexibly balancing cost and memory depth.

### 10. Fractal Social Structure
The system is self-similar. A complex agent can be composed of multiple sub-agents, exposing a unified MCP interface to the outside. This fractal design allows building infinitely layered collaboration networks — achieving "from one, all things arise" in the digital society.

---

## Documentation Map

### 1. Architecture Documentation
*Defines the system's vision, roles, and macro-level boot processes.*
* [00 Design Philosophy](./Architecture_Documentation/00_OpenStarry_Design_Philosophy.md)
* [01 Architecture Overview](./Architecture_Documentation/01_Architecture_Overview.md)
* [02 Headless Agent Core](./Architecture_Documentation/02_Headless_Agent_Core.md)
* [03 Agent Design & Template Service](./Architecture_Documentation/03_Agent_Design_and_Template_Service.md)
* [04 Plugin Infrastructure](./Architecture_Documentation/04_Plugin_Infrastructure.md)
* [05 Linux Design Principles Inspiration](./Architecture_Documentation/05_Linux_Design_Principles_Inspiration.md)
* [06 Plugin Interface Examples](./Architecture_Documentation/06_Plugin_Interface_Examples.md)
* [07 Supporting Engines Ecosystem](./Architecture_Documentation/07_Supporting_Engines_Ecosystem.md)
* [08 Command & Tool Design](./Architecture_Documentation/08_Command_And_Tool_Design.md)
* [09 Communication Protocol Strategy](./Architecture_Documentation/09_Communication_Protocol_Strategy.md)
* [10 Bootstrapping & Plugin Loading](./Architecture_Documentation/10_Bootstrapping_And_Plugin_Loading.md)
* [11 Agent Manager Tool Design](./Architecture_Documentation/11_Agent_Manager_Tool_Design.md)
* [12 Workflow Engine Tool Design](./Architecture_Documentation/12_Workflow_Engine_Tool_Design.md)
* [13 Orchestrator Daemon Design](./Architecture_Documentation/13_Orchestrator_Daemon_Design.md)
* [14 System Boot Sequence](./Architecture_Documentation/14_System_Boot_Sequence.md)
* [15 System Startup & Task Flow](./Architecture_Documentation/15_System_Startup_and_Task_Flow.md)
* [16 Plugin Types Philosophical Mapping](./Architecture_Documentation/16_Plugin_Types_Philosophical_Mapping.md)
* [17 Host Bootstrapping Pattern](./Architecture_Documentation/17_Host_Bootstrapping_Pattern.md)
* [18 Plugin Loading Protocol](./Architecture_Documentation/18_Plugin_Loading_Protocol.md)
* [19 Agent Coordination Layer](./Architecture_Documentation/19_Agent_Coordination_Layer.md)
* [20 Dependency Wiring & Control Loop](./Architecture_Documentation/20_Dependency_Injection_and_Control_Loop.md)
* [21 Plugin Interface Deep Dive](./Architecture_Documentation/21_Plugin_Interface_Deep_Dive.md)
* [22 Agent Coordination Layer: Normalization & Adaptation](./Architecture_Documentation/22_Agent_Coordination_Layer_Normalization.md)
* [23 Dynamic Plugin Loading & Naming](./Architecture_Documentation/23_Dynamic_Plugin_Loading_and_Naming.md)
* [24 Runner Architecture](./Architecture_Documentation/24_Runner_Architecture.md)
* [25 PushInput Event Architecture](./Architecture_Documentation/25_PushInput_Event_Architecture.md)
* [26 Plugin Service & Lifecycle Management](./Architecture_Documentation/26_Plugin_Service_And_Lifecycle_Management.md)
* [27 System Topology & Management Zone](./Architecture_Documentation/27_System_Topology_and_Management_Zone.md)

### 2. Agent Core Components Deep Dive
*Deep dives into the kernel, examining specific technical mechanisms and theoretical models.*
* [00 Core Philosophy](./Agent_Core_Components_Deep_Dive/00_Core_Philosophy.md)
* [01 Execution Loop](./Agent_Core_Components_Deep_Dive/01_Execution_Loop.md)
* [02 Communication Interface](./Agent_Core_Components_Deep_Dive/02_Communication_Interface.md)
* [03 Security Layer](./Agent_Core_Components_Deep_Dive/03_Security_Layer.md)
* [04 State Manager](./Agent_Core_Components_Deep_Dive/04_State_Manager.md)
* [05 Plugin Infrastructure Integration](./Agent_Core_Components_Deep_Dive/05_Plugin_Infrastructure_Integration.md)
* [06 State Persistence Mechanism](./Agent_Core_Components_Deep_Dive/06_State_Persistence_Mechanism.md)
* [07 Safety Circuit Breakers](./Agent_Core_Components_Deep_Dive/07_Safety_Circuit_Breakers.md)
* [08 Safety Implementation](./Agent_Core_Components_Deep_Dive/08_Safety_Implementation.md)
* [09 Observability and Tracing](./Agent_Core_Components_Deep_Dive/09_Observability_and_Tracing.md)
* [10 Context Management Strategy](./Agent_Core_Components_Deep_Dive/10_Context_Management_Strategy.md)
* [11 Plugin Runtime Isolation](./Agent_Core_Components_Deep_Dive/11_Plugin_Runtime_Isolation.md)
* [12 Error Handling & Self Correction](./Agent_Core_Components_Deep_Dive/12_Error_Handling_and_Self_Correction.md)
* [13 Agent Core as Control System](./Agent_Core_Components_Deep_Dive/13_Agent_Core_as_Control_System.md)
* [14 Agent Core Philosophy: Five Aggregates](./Agent_Core_Components_Deep_Dive/14_Agent_Core_Philosophy_Five_Aggregates.md)
* [16 OpenStarry Standard Protocol](./Agent_Core_Components_Deep_Dive/16_OpenStarry_Standard_Protocol.md)

### 3. Project Structure and Conventions
*Defines the physical layout, source code organization, development workflow, and installation specifications.*
* [00 Roadmap & Milestones](./Project_Structure_and_Conventions/00_Roadmap_and_Milestones.md)
* [01 Monorepo Top-Level Structure](./Project_Structure_and_Conventions/01_Monorepo_Top_Level_Structure.md)
* [02 Core Source Code Structure](./Project_Structure_and_Conventions/02_Core_Source_Code_Structure.md)
* [03 Shared & SDK Structure](./Project_Structure_and_Conventions/03_Shared_and_SDK_Structure.md)
* [04 Standard Agent Directory Anatomy](./Project_Structure_and_Conventions/04_Standard_Agent_Directory_Anatomy.md)
* [05 Agent Manifest Specification](./Project_Structure_and_Conventions/05_Agent_Manifest_Specification.md)
* [06 Plugin Directory Conventions](./Project_Structure_and_Conventions/06_Plugin_Directory_Conventions.md)
* [07 Coding & Testing Standards](./Project_Structure_and_Conventions/07_Coding_and_Testing_Standards.md)
* [08 System & Project Runtime Layouts](./Project_Structure_and_Conventions/08_System_and_Project_Runtime_Layouts.md)
* [09 CLI Design & Management Commands](./Project_Structure_and_Conventions/09_CLI_Design_and_Management_Commands.md)
* [10 Build & Distribution Strategy](./Project_Structure_and_Conventions/10_Build_and_Distribution_Strategy.md)
* [11 Third-Party Plugin Installation](./Project_Structure_and_Conventions/11_Third_Party_Plugin_Installation.md)
* [12 Capabilities Injection Mechanism](./Project_Structure_and_Conventions/12_Capabilities_Injection_Mechanism.md)
* [13 Composite Plugins & Dependencies](./Project_Structure_and_Conventions/13_Composite_Plugins_and_Dependencies.md)
* [14 Markdown Skill Specification](./Project_Structure_and_Conventions/14_Markdown_Skill_Specification.md)

### 4. Plugin System Architecture
*Concrete applications, concepts, and specifications of the plugin system.*
* [00 Plugin Philosophy: Five Aggregates](./Plugin_System_Architecture/00_Plugin_Philosophy_Five_Aggregates.md)
* [01 MCP Plugin Example](./Plugin_System_Architecture/01_MCP_Plugin_Example.md)
* [02 MCP Protocol Integration](./Plugin_System_Architecture/02_MCP_Protocol_Integration.md)
* [03 Developer Tools Example](./Plugin_System_Architecture/03_Developer_Tools_Example.md)
* [04 Web Interaction Example](./Plugin_System_Architecture/04_Web_Interaction_Example.md)
* [05 Advanced UI & Device Example](./Plugin_System_Architecture/05_Advanced_UI_And_Device_Example.md)
* [06 Data Validation Example](./Plugin_System_Architecture/06_Data_Validation_Example.md)

### 5. Implementation Examples
*Hands-on code, learning by example.*
* [Context Strategy: Sliding Window](./Implementation_Examples/Context_Strategy_SlidingWindow.md)
* [Developer Guide: Standalone Execution](./Implementation_Examples/Developer_Guide_Standalone_Execution.md)
* [OpenClaw Coordination Layer](./Implementation_Examples/openclaw_Coordination_Layer.md)
* [OpenClaw UI Channel Adapters](./Implementation_Examples/openclaw_UI_Channel_Adapters.md)
* [OpenCode Code Interpreter Suite](./Implementation_Examples/opencode_Code_Interpreter_Suite.md)
* [Provider: Gemini Example](./Implementation_Examples/Provider_Gemini_Example.md)
* [Tool: Code Interpreter Example](./Implementation_Examples/Tool_CodeInterpreter_Example.md)
* [Tool: Read File Example](./Implementation_Examples/Tool_ReadFile_Example.md)
* [Transport: WebSocket Plugin](./Implementation_Examples/Transport_Plugin_Websocket.md)
* [UI Plugin Example](./Implementation_Examples/UI_Plugin_Example.md)
* [USB Plug-and-Play Agent Scenario](./Implementation_Examples/USB_Plug_and_Play_Agent_Scenario.md)
* [Pain Mechanism Demo](./Implementation_Examples/Pain_Mechanism_Demo.md)


---

## Quick Start

Ready to get started? See **[Developer_Guide_Standalone_Execution.md](./Implementation_Examples/Developer_Guide_Standalone_Execution.md)** to run your first agent.
