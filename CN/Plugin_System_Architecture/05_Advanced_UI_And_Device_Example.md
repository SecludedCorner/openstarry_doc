# 插件示例：高级 UI 与设备集成

本文档阐述了超越文本的高级 UI 和设备集成插件的设计思路。

---

### 1. Canvas UI

这种类型的 UI 插件允许代理人创建和操作图形化、可视化的输出，而不仅仅是文本。

*   **插件类型：** `ui`
*   **职责：**
    1.  实现「双向通信接口」。
    2.  提供一个图形绘制区域（如 HTML Canvas）。
    3.  将来自核心的「消息」解析为「绘图指令」。
*   **`plugin.json` 示例：**
    ```json
    {
      "name": "Canvas-Visualizer-UI",
      "type": "ui",
      "entryPoint": "./CanvasUI.js"
    }
    ```
*   **实现思路：**
    *   **出向 (Core -> UI):** `CanvasUI.js` 监听 `onNewMessage` 事件。它会检查消息的 `metadata` 或格式。如果消息是一个绘图指令（例如 `{ type: 'draw_command', shape: 'rectangle', coords: [10, 10, 50, 50], color: 'red' }`），它就在 Canvas 上绘制对应的图形。如果是一个普通文本消息，它就在 Canvas 的某个区域显示文本。
    *   **入向 (UI -> Core):** `CanvasUI.js` 监听 Canvas 上的鼠标点击等事件。当用户点击 Canvas 时，它可以将点击坐标 `{ x: 55, y: 120 }` 包装成一个用户输入，通过 `submitUserInput` 发送给核心，让 LLM 知道用户点击了什么位置，并作出反应。

---

### 2. 设备集成 (Device Integration)

设备集成可以通过 `Tool` 和 `Listener` 两种插件来实现，让代理人感知和交互物理世界。

#### 作为 `Tool` (主动获取)

*   **插件类型：** `tool`
*   **范例：**
    *   `device:get_location`: 获取设备的 GPS 坐标。
    *   `device:take_picture`: 调用设备的摄像头拍照。
    *   `device:get_battery_level`: 获取电池电量。
*   **实现思路：** 这些工具的 `execute` 方法会调用特定于操作系统（iOS, Android, Windows）的底层 API 来获取信息。这通常需要一个跨平台的桥接层（如 React Native, Flutter, or Tauri 的 JS-Rust 桥）。

#### 作为 `Listener` (被动监听)

*   **插件类型：** `listener`
*   **范例：**
    *   `device:accelerometer_listener`: 持续监听加速度计的变化。
    *   `device:gps_location_listener`: 当地理位置发生显著变化时触发。
*   **实现思路：**
    *   插件的 `start(eventQueue)` 方法会注册一个系统级的事件监听器。
    *   当物理设备的传感器数据发生变化时，系统回调会被触发。
    *   插件将传感器数据（如 `{ x: 0.1, y: -0.5, z: 9.8 }`）打包成标准事件。
    *   调用 `eventQueue.push({ source: 'device', type: 'accelerometer_update', payload: ... })` 将事件推入核心的事件队列，让代理人能够对物理世界的变化作出反应。
    *   调用 `eventQueue.push({ source: 'device', type: 'accelerometer_update', payload: ... })` 将事件推入核心的事件队列，让代理人能够对物理世界的变化作出反应。
