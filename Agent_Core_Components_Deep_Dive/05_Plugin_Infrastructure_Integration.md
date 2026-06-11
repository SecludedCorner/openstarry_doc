# 深度解析：插件基礎設施集成

本文件深入探討「插件基礎設施」是如何被「無頭代理人核心」集成和使用的。

## 核心定位：核心的「設備管理器」

雖然插件基礎設施的「加載引擎」是一個通用的、可被多處共享的庫，但在代理人核心內部，它扮演著類似操作系統「設備管理器」的角色。核心本身包含了一系列**註冊表 (Registries)** 和**管理器 (Managers)**，用於「接線」(Wiring) 和管理由引擎加載的插件。

## 核心內部的註冊表與管理器

*   **`ToolRegistry` (工具註冊表):**
    *   **職責：** 維護一個當前會話可用的所有 `Tool` 插件的列表。
    *   **功能：** 提供 `register`, `unregister`, `getTool(name)`, `getToolDefinitions()` 等方法。`getToolDefinitions()` 方法尤其重要，它會生成一個格式化的工具描述列表，供「執行循環」發送給 LLM。

*   **`ProviderManager` (Provider 管理器):**
    *   **職責：** 管理所有已加載的 `Provider` 插件，並維護當前激活的是哪一個。
    *   **功能：** 提供 `add(provider)`, `setActive(providerName)`, `getActive()` 等方法。

*   **`UIManager` (UI 管理器):**
    *   **職責：** 負責將加載的 `UI` 插件與核心的「雙向通信接口」進行對接。

## 工作流程：啟動與運行

1.  **核心啟動:**
    *   代理人核心實例被創建。
    *   核心內部實例化 `ToolRegistry`, `ProviderManager`, `UIManager` 等組件。

2.  **實例化加載器:**
    *   核心創建一個通用的 `PluginLoader` 實例，並為其注入**針對核心自身**的註冊策略（Handlers）。
    *   例如，`tool` 類型的插件，其註冊策略就是調用 `this.toolRegistry.register(plugin)`。

3.  **加載插件:**
    *   核心指示 `PluginLoader` 從配置的插件目錄中加載所有插件。
    *   `PluginLoader` 每成功加載一個插件，就會根據其 `type` 調用對應的註冊策略，將插件「接線」到正確的管理器上。

4.  **運行時使用:**
    *   在「執行循環」的「打包上下文」階段，它會調用 `toolRegistry.getToolDefinitions()` 來獲取當前所有可用工具的描述，並將其包含在發送給 LLM 的提示中。
    *   當需要執行工具時，它會調用 `toolRegistry.getTool(name)` 來獲取工具實例並執行它。

這種設計將通用的加載機制與具體的集成邏輯分離開來，使得核心能夠靈活地管理其功能，同時保持了代碼的整潔和高內聚。
