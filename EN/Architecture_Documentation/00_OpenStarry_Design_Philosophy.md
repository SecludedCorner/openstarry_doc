# 00. OpenStarry Design Philosophy

This document is the starting point for understanding the OpenStarry architecture. It does not cover specific code implementation but rather explains why we are building such a system and the core values that guide all our technical decisions.

---

## Core Vision: From "Scripts" to "Species"

Most current AI Agents still remain at the "Script" stage: they are triggered, execute a piece of logic, and then terminate. They are like single-run functions.

OpenStarry's vision is to elevate the Agent to an **"OS Process"** or even a **"Digital Species"**.

We believe a true Agent should possess the following qualities:
1.  **Persistence:** It has its own lifecycle, can sleep and wake up, and its memory is continuous.
2.  **Identity:** It exists independently of external triggers. It has its own state, configuration, and "soul."
3.  **Sociality:** It is naturally capable of collaborating with other Agents, just as humans are naturally capable of language.

---

## Four Design Pillars

### 1. Kernel-Peripheral Separation
This is the most important lesson we've learned from the Linux design philosophy.

*   **Agent Core (Kernel)** is pure. It only handles thinking (LLM interaction), memory (Context management), and decision-making (Action generation). It doesn't know whether it's running in a CLI or on WhatsApp.
*   **Plugins (Peripherals)** are responsible for all interactions with the physical world. All I/O (input/output), all communication protocols (WebSocket, MCP), and all specific capabilities (reading files, sending emails) are plugins.

**Significance:** This ensures that the Agent's "soul" can be transplanted into any "body."

### 2. Anthropomorphic Cognitive Flow
We don't view the Agent as a `Request -> Response` server, but as an entity with a **"Stream of Consciousness."**

*   **Execution Loop:** This is the Agent's heartbeat. It constantly perceives the environment (event queue), processes information, and takes action.
*   **Working Memory:** Just as human attention is limited, the Agent's Context Window is a scarce resource. We must manage it strategically (forgetting, summarizing) rather than simply piling up data.

### 3. Fractal Collaboration
The system structure of OpenStarry is fractal.

*   An Agent can be just a simple tool (e.g., "Search Agent").
*   But it can also be a complex team (e.g., "R&D Department Agent"), composed of countless small Agents internally.
*   Externally, they all expose the same interface (MCP). This allows us to build extremely complex organizations using simple Agents like building blocks, without changing the architecture.

### 4. Error as Feedback
In traditional software, an error (Exception) usually means a crash or interruption.
In OpenStarry, **errors are opportunities to learn.**

*   When a tool call fails, the Core doesn't crash but feeds the error message back to the LLM as "sensory input."
*   The LLM will "realize" it made a mistake and try to correct it (e.g., by changing a parameter or using another tool).
*   This **"Self-Correction Loop"** is an manifestation of intelligence.

---

## Conclusion

OpenStarry is not just a simple SDK; it is a blueprint for an **"Agent Operating System."** We hope developers keep this in mind as they read the subsequent documentation: **You are not just writing code; you are creating a life.**