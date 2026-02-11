# 01. Headless Agent Core

## Final Core Architecture: A Pure "Event-Driven Execution Engine"

Guided by the design philosophies of "Everything is a Plugin" and "Keeping the Core Absolutely Pure," the positioning of the "Headless Agent Core" has been further refined. It is a **pure, single-threaded, asynchronous "Event-Driven Execution Engine."**

Its core is no longer built around specific "Hooks" for `UI` or `MCP`, but rather a **unified "Internal Event Queue."** All behaviors of the Core are driven by processing events within this queue.

## Overview of Core Responsibilities

The responsibilities of the Agent Core can be viewed as a standardized set of processes built around the "Event Queue."

1.  **Internal Event Queue - [New Core]**:
    *   Serves as the **sole input source** for the Core, receiving standardized events from various "Listener Plugins."
2.  **Execution Loop - [Simplified Responsibility]**:
    *   No longer listens externally; it is only responsible for retrieving events from the "Event Queue" and triggering subsequent workflows (typically packaging the context and calling the LLM) based on the event type.
3.  **Plugin Infrastructure:**
    *   Responsibilities remain the same, but support for **`listener`** type plugins has been added.
4.  **Listener Plugin - [Sensation / Vedana]**:
    *   An active, long-running plugin responsible for **transforming external asynchronous events into standardized internal events** and pushing them into the "Event Queue."
    *   For example, a CLI stdin listener receives user terminal input and pushes it into the queue as a `user_input` event.
    *   `MCP` message reception is implemented by an `MCP_Listener` plugin.
5.  **UI Plugin - [Form / Rupa]**:
    *   UI plugins have an independent `IUI` interface, responsible for receiving events and rendering output.
    *   Listener plugins (Sensation) focus on receiving external input. The two roles are distinct and no longer mixed.
6.  **Security Layer & State Manager:**
    *   Responsibilities remain unchanged, continuing to serve as critical processing links within the execution loop.

---

Through this refinement, the Core itself becomes completely decoupled from any specific input source (UI, message bus, Webhook). As long as a functionality can be implemented as a `Listener` plugin that pushes messages to the "Event Queue," the Core can handle it. This provides the system with ultimate flexibility and purity.