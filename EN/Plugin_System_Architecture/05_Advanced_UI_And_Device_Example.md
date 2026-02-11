# Plugin Example: Advanced UI & Device Integration

This document outlines the design concepts for advanced UI and device integration plugins that go beyond text.

---

### 1. Canvas UI

This type of UI plugin allows the Agent to create and manipulate graphical and visual outputs, not just text.

*   **Plugin Type:** `ui`
*   **Responsibilities:**
    1.  Implement the "Bi-directional Communication Interface."
    2.  Provide a graphical drawing area (e.g., HTML Canvas).
    3.  Parse "messages" from the Core into "drawing instructions."
*   **`plugin.json` Example:**
    ```json
    {
      "name": "Canvas-Visualizer-UI",
      "type": "ui",
      "entryPoint": "./CanvasUI.js"
    }
    ```
*   **Implementation Strategy:**
    *   **Outbound (Core -> UI):** `CanvasUI.js` listens for the `onNewMessage` event. It checks the `metadata` or format of the message. If the message is a drawing instruction (e.g., `{ type: 'draw_command', shape: 'rectangle', coords: [10, 10, 50, 50], color: 'red' }`), it renders the corresponding graphic on the Canvas. If it's a plain text message, it displays the text in a designated area of the Canvas.
    *   **Inbound (UI -> Core):** `CanvasUI.js` listens for mouse clicks and other events on the Canvas. When a user clicks the Canvas, the plugin can wrap the click coordinates `{ x: 55, y: 120 }` into a user input and send it to the Core via `submitUserInput`, informing the LLM of where the user clicked and triggering a reaction.

---

### 2. Device Integration

Device integration can be achieved through both `Tool` and `Listener` plugins, allowing the Agent to perceive and interact with the physical world.

#### As a `Tool` (Active Retrieval)

*   **Plugin Type:** `tool`
*   **Examples:**
    *   `device:get_location`: Retrieves the device's GPS coordinates.
    *   `device:take_picture`: Invokes the device's camera to take a photo.
    *   `device:get_battery_level`: Retrieves the battery level.
*   **Implementation Strategy:** The `execute` method of these tools calls low-level APIs specific to the operating system (iOS, Android, Windows). This typically requires a cross-platform bridge layer (e.g., JS-Rust bridge in Tauri, React Native, or Flutter).

#### As a `Listener` (Passive Monitoring)

*   **Plugin Type:** `listener`
*   **Examples:**
    *   `device:accelerometer_listener`: Continuously monitors changes in the accelerometer.
    *   `device:gps_location_listener`: Triggers when the geographic location changes significantly.
*   **Implementation Strategy:**
    *   The `start(eventQueue)` method of the plugin registers a system-level event listener.
    *   When the physical device's sensor data changes, the system callback is triggered.
    *   The plugin packages the sensor data (e.g., `{ x: 0.1, y: -0.5, z: 9.8 }`) into a standard internal event.
    *   Invokes `eventQueue.push({ source: 'device', type: 'accelerometer_update', payload: ... })` to push the event into the Core's event queue, allowing the Agent to react to changes in the physical world.
    *   Invokes `eventQueue.push({ source: 'device', type: 'accelerometer_update', payload: ... })` to push the event into the Core's event queue, allowing the Agent to react to changes in the physical world.
