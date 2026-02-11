# 03. 标准代理人目录解剖学与层级规范 (Standard Agent Directory Anatomy & Hierarchy)

本文档定义了 OpenStarry 系统中「代理人工作空间」的标准布局。此规范同时适用于 **系统级 (System-level)** 与 **项目级 (Project-level)** 的环境。

## 核心哲学：自相似性 (Self-Similarity)

在 OpenStarry 中，系统管理员（System Agent）与具体的执行者（Project Agent）在物理结构上是完全对称的。这种「分形」设计确保了任何一个项目在未来都有可能演化成另一个子系统的管理器。

---

## 标准目录结构 (The Anatomy)

无论是在系统路径还是项目路径，都必须遵循以下布局：

```text
[Agent_Root]/
├── plugins/           # 聚合体插件存放区 (Aggregates)
│   ├── tool-A/        # 插件 A
│   └── listener-B/    # 插件 B
├── configs/           # 代理人蓝图与配置 (Manifests)
│   └── agent.json     # 定义该 Agent 加载哪些插件
├── states/            # 状态与持久化 (States/Snapshots)
│   └── memory.db      # 识蕴 (Vijnana) 的物理存储
└── logs/              # 运行时日志 (受蕴的历史记录)
```

---

## 插件发现与继承机制

Agent 启动时，其 `PluginLoader` 会扫描多个层级的 `plugins/` 目录：

1.  **优先级 1 (项目级):** `[Current_Project_Root]/plugins/`
2.  **优先级 2 (系统级):** `[openstarry_system_root]/plugins/`

**规则：**
*   项目 Agent **可以** 调用系统级插件，实现资源共享（如使用系统统一提供的 `git-tool`）。
*   如果两处存在同名插件，**项目级优先**，允许项目覆盖系统默认行为。

---

## 角色权限与隔离 (Security Policy)

虽然结构相同，但不同层级的 Agent 受到严格的职责限制：

### 1. 系统代理人 (System Agent / Manager)
*   **驻留位置：** `openstarry_system_root`
*   **核心任务：** 管理项目 Agent 的生命周期（创建、监控、销毁）。
*   **严格限制：** **严禁加载任何业务插件**（如 WhatsApp, 数据库操作工具）。
*   **操作逻辑：** 如果系统 Agent 需要执行业务操作，它必须启动一个特定的「项目 Agent」来代为执行。它本身只拥有 `AgentManagerTool`。

### 2. 项目代理人 (Project Agent / Worker)
*   **驻留位置：** 具体向项目目录。
*   **核心任务：** 执行具体业务。
*   **操作逻辑：** 拥有其目录下 `plugins/` 内所有工具的完整操作权限。

---

## 强制关闭机制 (The Kill Switch)

系统 Agent 与人类管理员拥有对项目 Agent 的最高控制权：

1.  **进程控制：** 系统 Agent 通过 `Orchestrator Daemon` 维护项目 Agent 的 PID。
2.  **强制终止：** 当项目 Agent 失控或任务完成时，系统 Agent 可以发送 `SIGKILL` 信号。
3.  **哲学意义：** 这代表了「识蕴 (主体意识)」对「行蕴 (行动)」的绝对控制。当一个行动不再符合系统目标时，管理中心必须能物理性地切断其能量供应（结束进程）。
    *   **进程控制：** 系统 Agent 通过 `Orchestrator Daemon` 维护项目 Agent 的 PID。
2.  **强制终止：** 当项目 Agent 失控或任务完成时，系统 Agent 可以发送 `SIGKILL` 信号。
3.  **哲学意义：** 这代表了「识蕴 (主体意识)」对「行蕴 (行动)」的绝对控制。当一个行动不再符合系统目标时，管理中心必须能物理性地切断其能量供应（结束进程）。
