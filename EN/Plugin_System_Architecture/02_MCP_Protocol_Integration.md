# 08. MCP: Inter-Agent Communication Protocol

This document details the implementation strategy for the MCP (Multi-Component/Agent Communication Protocol) under the refined architecture.

## Core Positioning: One Protocol, Two Plugins

In the new architecture, the MCP is no longer borne by a standalone "Message Bus Engine." Instead, it is viewed as a **communication protocol specification**, with its functionality collectively implemented by two different types of plugins, thereby enhancing system decoupling.

---

## Component Design

### 1. `MCP_Listener` Plugin

*   **Plugin Type:** **`listener`**
*   **Responsibility:** Implements the **receiver side** of the MCP.
*   **Workflow:**
    1.  This is a long-running plugin. In its `start()` method, it establishes a connection to the underlying message middleware (e.g., RabbitMQ, Redis Pub/Sub, or a simple Node.js EventEmitter).
    2.  It subscribes to messages addressed to its host Agent Core ID.
    3.  Upon receiving a message from the middleware, it validates the message against the MCP packet format.
    4.  Once validated, it transforms the message packet into a standard internal event object for the Core, such as:
        ```json
        {
          "source": "mcp_bus",
          "type": "mcp_message",
          "payload": { /* body of the MCP message */ }
        }
        ```
    5.  Finally, it invokes the `eventQueue.push()` method injected by the Core to push the event into the "Internal Event Queue."

### 2. `mcp:send` Plugin

*   **Plugin Type:** **`tool`**
*   **Responsibility:** Implements the **sender side** of the MCP.
*   **Workflow:**
    1.  This is a standard, passively invoked tool.
    2.  When the LLM decides to send a message, the "Execution Loop" of the Agent Core invokes the `execute` method of this tool.
    3.  The `execute` method receives `args` containing `target_id`, `action`, `payload`, etc.
    4.  The method assembles these `args` into a message packet conforming to the MCP specification.
    5.  It publishes this message packet to the underlying message middleware via a shared client.

---

## Advantages

This design completely decouples the specific implementation of the MCP from the Core:

*   **Replaceable Message Middleware:** The Agent Core is entirely unaware of whether RabbitMQ or Redis is being used underneath. The middleware can be easily swapped by replacing the `MCP_Listener` plugin implementation without any modifications to the Core.
*   **Maintains Core Purity:** The Core's responsibility is strictly limited to processing its internal event queue; all asynchronous communication with the external world is abstracted away into `Listener` plugins.
*   **Clear Separation of Concerns:** The responsibilities for sending (`Tool`) and receiving (`Listener`) are clearly separated into two distinct plugin types.
