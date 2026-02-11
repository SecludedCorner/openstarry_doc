# 深度解析：插件基础设施集成

本文档深入探讨「插件基础设施」是如何被「无头代理人核心」集成和使用的。

## 核心定位：核心的「设备管理器」

虽然插件基础设施的「加载引擎」是一个通用的、可被多处共享的库，但在代理人核心内部，它扮演着类似操作系统「设备管理器」的角色。核心本身包含了一系列**注册表 (Registries)** 和**管理器 (Managers)**，用于「接线」(Wiring) 和管理由引擎加载的插件。

## 核心内部的注册表与管理器

*   **`ToolRegistry` (工具注册表):**
    *   **职责：** 维护一个当前会话可用的所有 `Tool` 插件的列表。
    *   **功能：** 提供 `register`, `unregister`, `getTool(name)`, `getToolDefinitions()` 等方法。`getToolDefinitions()` 方法尤其重要，它会生成一个格式化的工具描述列表，供「执行循环」发送给 LLM。

*   **`ProviderManager` (Provider 管理器):**
    *   **职责：** 管理所有已加载的 `Provider` 插件，并维护当前激活的是哪一个。
    *   **功能：** 提供 `add(provider)`, `setActive(providerName)`, `getActive()` 等方法。

*   **`UIManager` (UI 管理器):**
    *   **职责：** 负责将加载的 `UI` 插件与核心的「双向通信接口」进行对接。

## 工作流程：启动与运行

1.  **核心启动:**
    *   代理人核心实例被创建。
    *   核心内部实例化 `ToolRegistry`, `ProviderManager`, `UIManager` 等组件。

2.  **实例化加载器:**
    *   核心创建一个通用的 `PluginLoader` 实例，并为其注入**针对核心自身**的注册策略（Handlers）。
    *   例如，`tool` 类型的插件，其注册策略就是调用 `this.toolRegistry.register(plugin)`。

3.  **加载插件:**
    *   核心指示 `PluginLoader` 从配置的插件目录中加载所有插件。
    *   `PluginLoader` 每成功加载一个插件，就会根据其 `type` 调用对应的注册策略，将插件「接线」到正确的管理器上。

4.  **运行时使用:**
    *   在「执行循环」的「打包上下文」阶段，它会调用 `toolRegistry.getToolDefinitions()` 来获取当前所有可用工具的描述，并将其包含在发送给 LLM 的提示中。
    *   当需要执行工具时，它会调用 `toolRegistry.getTool(name)` 来获取工具实例并执行它。

这种设计将通用的加载机制与具体的集成逻辑分离开来，使得核心能够灵活地管理其功能，同时保持了代码的整洁和高内聚。
