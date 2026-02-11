# 01. The Ideal Agent Architecture: Overall Overview

**OpenStarry is, in itself, a complete "Agent Coordination Layer" solution.**

It is not just an Agent development framework, but a set of OS-level infrastructure for building, running, managing, and orchestrating multi-agent systems. Under this coordination layer, we deconstruct the system into the following key sub-systems.

## Core Sub-systems

These five sub-systems are like five organs that together maintain the operation of this coordination layer:

*   **1. Agent Core / Kernel:**
    *   **Role:** "The Brain and the Heart."
    *   **Architecture:** **Microkernel**. It contains only the most fundamental execution loop, state machine, and event bus. All specific functionalities (including file operations, network communication, and LLM calls) are moved out of the kernel and into user space (the plugin layer).
    *   **Responsibility:** A pure, event-driven execution engine.
    *   (See `02_Headless_Agent_Core.md` for details)

*   **2. Agent Design and Template Service:**
    *   **Role:** "Gene Pool and Creation Workshop."
    *   **Responsibility:** Manages the Agent's "Templates." It defines what personality an Agent has and what plugins it loads. It supports dynamic creation of new roles at runtime.
    *   (See `03_Agent_Design_and_Template_Service.md` for details)

*   **3. Plugin Infrastructure:**
    *   **Role:** "Limbs and Senses."
    *   **Responsibility:** Provides standardized interfaces for the Core to mount Tools, communication protocols (Listeners), and AI models (Providers).
    *   (See `04_Plugin_Infrastructure.md` for details)

*   **4. Orchestrator Daemon:**
    *   **Role:** "Lifecycle Manager (Init Process)."
    *   **Responsibility:** Responsible for starting, monitoring, restarting, and terminating Agent OS processes. It is the cornerstone for ensuring persistent system operation.
    *   (See `13_Orchestrator_Daemon_Design.md` for details)

*   **5. Supporting Engines:**
    *   **Role:** "Logistics Support Department."
    *   **Responsibility:** Provides shared backend capabilities, such as:
        *   **Memory Engine (RAG):** Knowledge retrieval.
        *   **Security Engine (Guardrails):** Permission and compliance review.
        *   **Communication Strategy:** Multi-protocol adaptation and routing.
    *   (See documents `07` and `09` for details)

## Collaborative Workflow Overview

1.  **Design:** A human or a Master Agent defines a new Agent template via the **Design Service**.
2.  **Spawn:** The **Orchestrator Daemon** reads the template and spawns a new OS process to run the **Agent Core**.
3.  **Assemble:** The new Core initializes the **Plugin Infrastructure** and loads the specified tool and communication plugins.
4.  **Run:** The Agent begins executing tasks with the assistance of **Supporting Engines** (e.g., memory, security).
5.  **Collaborate:** Multiple Agents communicate via standard protocols (e.g., MCP) to collectively achieve complex goals.