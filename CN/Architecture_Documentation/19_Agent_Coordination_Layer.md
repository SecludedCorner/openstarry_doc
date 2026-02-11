# 19. 代理协调层 (Agent Coordination Layer)

本文档定义了 OpenStarry 系统中的「中枢神经」——协调层。它位于 Daemon 内部，负责管理所有插件的注册、Agent 的通讯路由以及生命周期。

## 1. 核心职责

协调层 (Coordination Layer) 是 **Daemon** 进程中的一个逻辑模块，它承担以下职责：

1.  **插件总机 (Plugin Registry Authority):**
    *   维护全系统所有可用插件的清单（扫描自 `~/.openstarry/plugins`）。
    *   响应 Agent 的加载请求，提供插件的物理路径。
    *   处理 `openstarry plugin sync` 指令，从 `openstarry_plugin` 仓库同步更新。

2.  **代理人注册 (Agent Registry):**
    *   记录所有活跃 Agent 的 PID、通讯端口 (Socket/Pipe) 与状态。
    *   提供 `getAgent(id)` 接口，让 CLI (`attach`) 或其他 Agent 能够找到目标。

3.  **消息路由 (Message Routing):**
    *   当一个 Agent 发出 `send("manager", "help")` 时，协调层负责解析 "manager" 是谁，并将消息转发过去。

## 2. 插件注册流程

当开发者安装一个新插件，或者系统启动时：

1.  **扫描：** 协调层扫描插件目录。
2.  **验证：** 检查 `plugin.json` 的合法性。
3.  **索引：** 将插件的 `id`, `version`, `capabilities` 存入内存数据库 (如 LokiJS 或 SQLite)。
    *   索引示例：`Capability("weather") -> Plugin("openstarry-weather-v1")`。

## 3. Agent 启动时的交互

1.  **请求：** 新启动的 Agent Core 向协调层发送：「我需要 `standard/fs` 和 `gemini`」。
2.  **解析：** 协调层查询索引，确认这些插件存在且版本兼容。
3.  **授权：** 协调层检查 `agent.json` 的权限声明，确认该 Agent 有权使用这些插件。
4.  **返回：** 协调层返回插件的入口文件路径。
5.  **注册：** Agent 成功加载后，向协调层回报：「我已上线，我是 `data-bot-01`，我具备 `analyze-data` 的能力」。

## 4. 实现技术

*   **通讯协议：** 使用 **gRPC** 或 **Named Pipes** (IPC) 进行 Core 与 Daemon 之间的高效通讯。
*   **存储：** 使用轻型嵌入式数据库存储注册表信息，确保重启后不丢失。

---

## 5. 持久化与冷启动策略 (Persistence & Boot Strategy)

### A. 物理持久化 (Physical Persistence)
通过 `openstarry plugin sync` 或 `add` 安装的插件，其源码会**永久存放**于 `~/.openstarry/plugins/` 目录下。除非手动删除，否则重启后它们依然存在。

### B. 注册表索引 (Registry Indexing)
为了确保系统的高效能与一致性，协调层采用以下启动策略：

1.  **快速扫描 (Fast Scan):** 每次 Daemon 启动时，会先扫描插件目录的**文件变动时间 (mtime)**。
2.  **增量更新 (Incremental Refresh):** 
    *   若目录无变动，则直接从 `registry.db` 读取缓存。
    *   若侦测到新文件夹或版本变更，则自动解析 `plugin.json` 并更新数据库索引。
3.  **失效清理:** 若数据库中记录的插件在硬盘上已不存在，则自动将其从注册表中移除。

### C. 运行时状态
请注意，**「注册表」的持久化不等于「插件实例」的持久化**。每次系统启动，插件的代码会根据 `18` 号协议重新加载并执行 `initialize()`。插件若需保存运行数据，必须通过 Core 注入的 `state` 接口（见文档 06）进行。

---

## 6. 健康检查与自我修复 (Health Check & Self-Healing)

协调层具备强大的容错能力，能处理手动操作带来的各种异常状态。

### A. 手动放入 (Manual Drop-in)
当用户直接将文件夹复制到 `~/.openstarry/plugins/` 时：
1.  **发现：** Daemon 的文件监控 (Watcher) 或启动扫描侦测到新目录。
2.  **验证：** 尝试解析 `plugin.json`。
    *   **成功：** 自动注册，状态设为 `READY`。
    *   **失败：** 状态设为 `INVALID_MANIFEST`，并在 Dashboard 显示黄色警告。

### B. 载入失败 (Load Failure)
如果插件代码有误（如语法错误）导致 `initialize()` 崩溃：
1.  **隔离：** Loader 捕获异常，防止 Daemon 崩溃。
2.  **标记：** 状态设为 `QUARANTINED` (已隔离)。
3.  **报告：** 在 Dashboard 显示红色 ❌，并提供错误日志查看选项。

### C. 依赖缺失 (Dependency Conflict)
如果插件 A 声明依赖插件 B，但 B 不存在：
1.  **检查：** 在构建依赖图时发现断链。
2.  **阻断：** 拒绝载入插件 A，将其状态设为 `UNSATISFIED`。
3.  **引导：** CLI 输出明确建议：
    > "Plugin [A] requires [B]. Please run `openstarry plugin add B` or `openstarry plugin sync` to resolve."
