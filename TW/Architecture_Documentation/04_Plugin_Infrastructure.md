# 04. 插件基礎設施 (Plugin Infrastructure)

本文件定義了 OpenStarry 的插件系統架構。我們採用 **聚合體模式 (Aggregate Pattern)**，允許一個插件包同時提供多種能力。

> **哲學備註 (Architectural Note):**
> 本架構深度融合「五蘊 (Five Aggregates)」思想，定義了插件的五種基本形態：
> 1.  **UI (色蘊 Rupa):** 物理的顯現與介面。
> 2.  **Listener (受蘊 Vedana):** 感知的接口 (Input)。
> 3.  **Provider (想蘊 Samjna):** 認知的引擎 (Processing)。
> 4.  **Tool (行蘊 Samskara):** 執行的能力 (Output/Action)。
> 5.  **Guide (識蘊 Vijnana):** 靈魂的指引 (Identity/Protocol/Logic)。Core 本身是空的容器，是 Guide 賦予了它「識別」與「自我意識」。

---

## 插件結構規範

在 OpenStarry 中，一個 **Plugin** 是一個功能單元，它可以包含以下任意組合的組件。

### `plugin.json` 定義

```json
{
  "id": "openstarry-full-stack",
  "components": {
    "ui": { ... },        // [色]
    "listeners": [ ... ], // [受]
    "providers": [ ... ], // [想]
    "tools": [ ... ],     // [行]
    "guides": [           // [識]
      { "name": "expert-persona", "entry": "./skills/expert.md" },
      { "name": "mcp-protocol", "entry": "./protocols/mcp-logic.js" }
    ]
  }
}
```

---

## 插件加載流程 (Loading Mechanism)

當 `PluginLoader` 加載一個插件時，它會執行「解構與註冊」的過程：

1.  **讀取 Manifest:** 讀取 `plugin.json`。
2.  **成分解構 (Destructuring):**
    *   掃描 `components.tools`，將其註冊到 **ToolRegistry**。
    *   掃描 `components.listeners`，將其註冊到 **ListenerRegistry** 並啟動。
    *   掃描 `components.providers`，將其註冊到 **ProviderRegistry**。
3.  **依賴注入:** 為每個成分注入其所需的 Core API。

這意味著，開發者以「功能包」的形式發布插件（例如 `npm install @openstarry-plugin/whatsapp`），而 Core 會自動將其中的手腳耳目拆解並歸位。

### 動態加載取代硬編碼工廠 (Dynamic Loading Replaces Hardcoded Factories)

早期版本中，核心透過 `BUILTIN_FACTORIES` 映射表硬編碼內建插件的加載邏輯。此方式已被完全移除，取而代之的是統一的**兩階段動態解析 (Two-Tier Resolution)**：

1.  **`ref.path` 優先：** 若 `agent.json` 中的插件參考包含 `path` 欄位（指向本地檔案系統路徑），Loader 直接透過該路徑載入插件模組。
2.  **`import(ref.name)` 備援：** 若無 `path`，Loader 使用 `import(ref.name)` 按照 NPM 包名進行動態載入（例如 `import('@openstarry-plugin/fs')`）。

這種統一機制消除了「內建插件」與「第三方插件」的邊界，所有插件享有相同的加載路徑與生命週期管理。

---

## 全域插件註冊表 (The Global Plugin Registry)

在這些能力進入 Core 內部的註冊表之前，必須先經過協調層 (Daemon) 的發現與索引。這是一個系統級的服務。

### 1. 職責 (Responsibilities)

*   **掃描與發現 (Scanning):** 啟動時掃描 `~/.openstarry/plugins/` (系統層) 與 `./.openstarry/plugins/` (專案層)。
*   **索引構建 (Indexing):** 讀取所有插件的 Manifest，建立一份 `Plugin ID -> Physical Path` 的映射表。
*   **能力解析 (Capability Resolution):** 當設計層查詢「誰有 `send_message` 能力？」時，註冊表負責返回對應的插件列表。

### 2. 發現流程

1.  **啟動:** Daemon 啟動 `PluginRegistryService`。
2.  **遍歷:** 服務遍歷所有已知路徑，尋找 `package.json` 或 `plugin.json`。
3.  **註冊:** 將有效插件的元數據存入內存資料庫。
4.  **供給:** 當具體的 Agent 啟動時，Loader 向此服務請求指定插件的路徑，然後才進行加載。

---

## 標準能力註冊表 (The Registry)

Core 內部維護著幾個全域註冊表：

*   `core.registry.tools`: 存儲所有可用的工具函數。
*   `core.registry.listeners`: 管理所有活躍的監聽器實例。
*   `core.registry.providers`: 存儲可用的 LLM 後端。

這種設計既保證了工程上的直觀性，又在底層邏輯上維持了五蘊架構的完整性。
