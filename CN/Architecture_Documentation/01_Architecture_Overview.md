# 01. 理想的代理人架构：整体概述

**OpenStarry 本身即为一个完整的「代理人协调层 (Agent Coordination Layer)」解决方案。**

它不仅是一个 Agent 开发框架，更是一套用于构建、运行、管理和编排多智能体系统的操作系统级基础设施。在此协调层之下，我们将系统解构为以下几个关键子系统。

## 核心子系统 (Sub-systems)

这五个子系统如同五个器官，共同维持着这个协调层的运转：

*   **1. 代理人核心 (Agent Core / Kernel):**
    *   **角色：** 「大脑与心脏」。
    *   **架构：** **微内核 (Microkernel)**。它只包含最基础的执行回路 (Loop)、状态机与事件总线。所有具体功能（包括文件操作、网络通讯、LLM 调用）都被移出内核，放入用户空间（插件层）。
    *   **职责：** 一个纯粹的、事件驱动的执行引擎。
    *   (详情请参阅 `02_Headless_Agent_Core.md`)

*   **2. 代理人设计与模板服务 (Agent Design Service):**
    *   **角色：** 「基因库与造物工坊」。
    *   **职责：** 管理代理人的「蓝图 (Templates)」。它定义了 Agent 拥有什么性格、加载什么插件。支持运行时动态创建新角色。
    *   (详情请参阅 `03_Agent_Design_and_Template_Service.md`)

*   **3. 插件基础设施 (Plugin Infrastructure):**
    *   **角色：** 「肢体与感官」。
    *   **职责：** 提供标准化的接口，让 Core 可以挂载工具 (Tools)、通讯协议 (Listeners) 和 AI 模型 (Providers)。
    *   (详情请参阅 `04_Plugin_Infrastructure.md`)

*   **4. 编排器守护进程 (Orchestrator Daemon):**
    *   **角色：** 「生命周期管理者 (Init Process)」。
    *   **职责：** 负责启动、监控、重启和终止 Agent 的 OS 进程。它是确保系统持久运行的基石。
    *   (详情请参阅 `13_Orchestrator_Daemon_Design.md`)

*   **5. 支撑引擎 (Supporting Engines):**
    *   **角色：** 「后勤支援部门」。
    *   **职责：** 提供共用的后端能力，如：
        *   **记忆引擎 (RAG):** 知识检索。
        *   **安全引擎 (Guardrails):** 权限与合规审查。
        *   **通讯策略:** 多协议适配与路由。
    *   (详情请参阅 `07` 和 `09` 号文件)

## 协作工作流简述

1.  **设计 (Design):** 人类或主代理人通过 **设计服务** 定义一个新的 Agent 模板。
2.  **孵化 (Spawn):** **Orchestrator Daemon** 读取模板，拉起一个新的 OS 进程运行 **Agent Core**。
3.  **装配 (Assemble):** 新的 Core 启动 **插件基础设施**，加载指定的工具与通讯插件。
4.  **运行 (Run):** Agent 开始在 **支撑引擎** (如记忆、安全) 的辅助下执行任务。
5.  **协作 (Collaborate):** 多个 Agent 通过标准协议 (如 MCP) 进行通讯，共同完成复杂目标。