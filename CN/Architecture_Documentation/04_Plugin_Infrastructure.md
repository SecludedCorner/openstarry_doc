# 04. 插件基础设施 (Plugin Infrastructure)

本文档定义了 OpenStarry 的插件系统架构。我们采用 **聚合体模式 (Aggregate Pattern)**，允许一个插件包同时提供多种能力。

> **哲学备注 (Architectural Note):**
> 本架构深度融合「五蕴 (Five Aggregates)」思想，定义了插件 transit 的五种基本形态：
> 1.  **UI (色蕴 Rupa):** 物理的显现与界面。
> 2.  **Listener (受蕴 Vedana):** 感知的接口 (Input)。
> 3.  **Provider (想蕴 Samjna):** 认知的引擎 (Processing)。
> 4.  **Tool (行蕴 Samskara):** 执行的能力 (Output/Action)。
> 5.  **Guide (识蕴 Vijnana):** 灵魂的指引 (Identity/Protocol/Logic)。Core 本身是空的容器，是 Guide 赋予了它「识别」与「自我意识」。

---

## 插件结构规范

在 OpenStarry 中，一个 **Plugin** 是一个功能单元，它可以包含以下任意组合的组件。

### `plugin.json` 定义

```json
{
  "id": "openstarry-full-stack",
  "components": {
    "ui": { ... },        // [色]
    "listeners": [ ... ], // [受]
    "providers": [ ... ], // [想]
    "tools": [ ... ],     // [行]
    "guides": [           // [识]
      { "name": "expert-persona", "entry": "./skills/expert.md" },
      { "name": "mcp-protocol", "entry": "./protocols/mcp-logic.js" }
    ]
  }
}
```

---

## 插件加载流程 (Loading Mechanism)

当 `PluginLoader` 加载一个插件时，它会执行「解构与注册」的过程：

1.  **读取 Manifest:** 读取 `plugin.json`。
2.  **成分解构 (Destructuring):**
    *   扫描 `components.tools`，将其注册到 **ToolRegistry**。
    *   扫描 `components.listeners`，将其注册到 **ListenerRegistry** 并启动。
    *   扫描 `components.providers`，将其注册到 **ProviderRegistry**。
3.  **依赖注入:** 为每个成分注入其所需的 Core API。

这意味着，开发者以「功能包」的形式发布插件（例如 `npm install @openstarry-plugin/whatsapp`），而 Core 会自动将其中的手脚耳目拆解并归位。

### 动态加载取代硬编码工厂 (Dynamic Loading Replaces Hardcoded Factories)

早期版本中，核心通过 `BUILTIN_FACTORIES` 映射表硬编码内建插件的加载逻辑。此方式已被完全移除，取而代之的是统一的**两阶段动态解析 (Two-Tier Resolution)**：

1.  **`ref.path` 优先：** 若 `agent.json` 中的插件参考包含 `path` 字段（指向本地文件系统路径），Loader 直接通过该路径加载插件模块。
2.  **`import(ref.name)` 备援：** 若无 `path`，Loader 使用 `import(ref.name)` 按照 NPM 包名进行动态加载（例如 `import('@openstarry-plugin/fs')`）。

这种统一机制消除了「内建插件」与「第三方插件」的边界，所有插件享有相同的加载路径与生命周期管理。

---

## 全局插件注册表 (The Global Plugin Registry)

在这些能力进入 Core 内部的注册表之前，必须先经过协调层 (Daemon) 的发现与索引。这是一个系统级的服务。

### 1. 职责 (Responsibilities)

*   **扫描与发现 (Scanning):** 启动时扫描 `~/.openstarry/plugins/` (系统层) 与 `./.openstarry/plugins/` (项目层)。
*   **索引构建 (Indexing):** 读取所有插件的 Manifest，建立一份 `Plugin ID -> Physical Path` 的映射表。
*   **能力解析 (Capability Resolution):** 当设计层查询「谁有 `send_message` 能力？」时，注册表负责返回对应的插件列表。

### 2. 发现流程

1.  **启动:** Daemon 启动 `PluginRegistryService`。
2.  **遍历:** 服务遍历所有已知路径，寻找 `package.json` 或 `plugin.json`。
3.  **注册:** 将有效插件的元数据存入内存数据库。
4.  **供给:** 当具体的 Agent 启动时，Loader 向此服务请求指定插件的路径，然后再进行加载。

---

## 标准能力注册表 (The Registry)

Core 内部维护着几个全局注册表：

*   `core.registry.tools`: 存储所有可用的工具函数。
*   `core.registry.listeners`: 管理所有活跃的监听器实例。
*   `core.registry.providers`: 存储可用的 LLM 后端。

这种设计既保证了工程上的直观性，又在底层逻辑上维持了五蕴架构的完整性。