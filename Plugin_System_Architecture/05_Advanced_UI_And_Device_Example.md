# 插件範例：高級 UI 與設備集成

本文件闡述了超越文本的高級 UI 和設備集成插件的設計思路。

---

### 1. Canvas UI

這種類型的 UI 插件允許代理人創建和操作圖形化、可視化的輸出，而不僅僅是文本。

*   **插件類型：** `ui`
*   **職責：**
    1.  實現「雙向通信接口」。
    2.  提供一個圖形繪製區域（如 HTML Canvas）。
    3.  將來自核心的「消息」解析為「繪圖指令」。
*   **`plugin.json` 示例：**
    ```json
    {
      "name": "Canvas-Visualizer-UI",
      "type": "ui",
      "entryPoint": "./CanvasUI.js"
    }
    ```
*   **實現思路：**
    *   **出向 (Core -> UI):** `CanvasUI.js` 監聽 `onNewMessage` 事件。它會檢查消息的 `metadata` 或格式。如果消息是一個繪圖指令（例如 `{ type: 'draw_command', shape: 'rectangle', coords: [10, 10, 50, 50], color: 'red' }`），它就在 Canvas 上繪製對應的圖形。如果是一個普通文本消息，它就在 Canvas 的某个區域顯示文本。
    *   **入向 (UI -> Core):** `CanvasUI.js` 監聽 Canvas 上的鼠標點擊等事件。當用戶點擊 Canvas 時，它可以將點擊坐標 `{ x: 55, y: 120 }` 包裝成一個用戶輸入，通過 `submitUserInput` 發送給核心，讓 LLM 知道用戶點擊了什麼位置，並作出反應。

---

### 2. 設備集成 (Device Integration)

設備集成可以通過 `Tool` 和 `Listener` 兩種插件來實現，讓代理人感知和交互物理世界。

#### 作為 `Tool` (主動獲取)

*   **插件類型：** `tool`
*   **範例：**
    *   `device:get_location`: 獲取設備的 GPS 坐標。
    *   `device:take_picture`: 調用設備的攝像頭拍照。
    *   `device:get_battery_level`: 獲取電池電量。
*   **實現思路：** 這些工具的 `execute` 方法會調用特定於操作系統（iOS, Android, Windows）的底層 API 來獲取信息。這通常需要一個跨平台的橋接層（如 React Native, Flutter, or Tauri 的 JS-Rust 橋）。

#### 作為 `Listener` (被動監聽)

*   **插件類型：** `listener`
*   **範例：**
    *   `device:accelerometer_listener`: 持續監聽加速度計的變化。
    *   `device:gps_location_listener`: 當地理位置發生顯著變化時觸發。
*   **實現思路：**
    *   插件的 `start(eventQueue)` 方法會註冊一個系統級的事件監聽器。
    *   當物理設備的傳感器數據發生變化時，系統回調會被觸發。
    *   插件將傳感器數據（如 `{ x: 0.1, y: -0.5, z: 9.8 }`）打包成標準事件。
    *   調用 `eventQueue.push({ source: 'device', type: 'accelerometer_update', payload: ... })` 將事件推入核心的事件隊列，讓代理人能夠對物理世界的變化作出反應。
