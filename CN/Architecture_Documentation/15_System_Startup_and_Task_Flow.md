# OpenStarry 系统：完整启动与任务流程

本文档详细阐述了 OpenStarry 系统从「零启动」到「完成一个复杂多代理人协作任务」的完整生命周期和工作流程。

## 核心流程总览

整个系统的运作，是基于 `Orchestrator Daemon` 对「代理人核心」进程的生命周期管理，以及「代理人核心」之间通过 `MCP` 进行的智能协作。

---

### 阶段一：系统引导与主代理人启动

1.  **启动 `Orchestrator Daemon`:**
    *   **启动者：** 系统管理员。
    *   **动作：** 执行 `openstarry-daemon start` 命令。
    *   **结果：** `Orchestrator Daemon` 作为一个独立的后台 OS 进程启动。它成为整个 OpenStarry 系统的进程守护者。

2.  **`Daemon` 启动「主代理人」:**
    *   **角色：** `Orchestrator Daemon`。
    *   **动作：** `Daemon` 根据其主配置文件，启动一个新的 OS 进程，运行**「主代理人 (Master Agent)」**。
    *   **结果：** 「主代理人」的进程被创建，并由 `Daemon` 负责其健康监控和生命周期管理。

3.  **「主代理人」自我初始化:**
    *   **角色：** 「主代理人」进程 (一个 `Agent Core` 实例)。
    *   **动作：**
        *   实例化其内部的 **`PluginLoader`**。
        *   `PluginLoader` 扫描默认插件目录（如 `~/.config/openstarry/system_plugins/`），加载所有为「主代理人」配置的核心能力插件（如 `UI_Listener`, `MCP_Listener`, `AgentManagerTool`, `WorkflowEngineTool`, `AgentDesignerTool` 等）。
    *   **结果：** 「主代理人」完全配置好，具备与人类交互、管理其他代理人、编排工作流等最高级能力，并进入待命状态。

---

### 阶段二：任务触发与主代理人决策

4.  **接收外部事件 (任务触发):**
    *   **角色：** 「主代理人」的 `Listener` 插件 (例如 `WhatsApp_Listener`)。
    *   **动作：** 一个人类用户通过 WhatsApp 发送了一条新的请求（例如，「请帮我分析上季度的销售数据并生成报告」）。`WhatsApp_Listener` 接收该消息。
    *   **结果：** `Listener` 将消息转换为标准事件，推入「主代理人」的**内部事件队列**。

5.  **「主代理人」决策与委派任务:**
    *   **角色：** 「主代理人」的「执行循环」和 LLM。
    *   **动作：**
        *   执行循环从队列中取出事件，将其与上下文发送给 LLM。
        *   LLM 分析后，判断这是一个需要专业数据分析能力的任务，决定需要启动一个「工作代理人」来执行。
        *   LLM 输出一个对 `AgentManagerTool` 的工具调用请求：`agent:start(template_id: 'DataAnalystAgent_v1', project_path: '/path/to/sales_data')`。

---

#### **阶段三：工作代理人按需创建**

6.  **`AgentManagerTool` 请求 Daemon 创建工作代理人:**
    *   **角色：** `AgentManagerTool`。
    *   **动作：** 该工具的 `execute` 方法被调用。它向本地运行的 **`Orchestrator Daemon`** 发起一个 API 请求，请求创建一个基于 `DataAnalystAgent_v1` 模板的工作代理人。

7.  **`Daemon` 创建工作代理人进程:**
    *   **角色：** `Orchestrator Daemon`。
    *   **动作：**
        *   `Daemon` 向「代理人设计层」服务查询 `DataAnalystAgent_v1` 模板的详细配置。
        *   创建一个新的 OS 进程，用于运行「工作代理人」。
        *   将模板配置和 `project_path` 等启动参数传递给这个新进程。
        *   将新进程纳入自己的监控列表。

8.  **工作代理人自我初始化:**
    *   **角色：** 新的「工作代理人」进程 (一个 `Agent Core` 实例)。
    *   **动作：**
        *   实例化它自己的 `PluginLoader`。
        *   `PluginLoader` 根据传递的模板配置和 `project_path` (用于加载项目级插件)，加载该工作代理人所需的精确插件集（例如 `code_interpreter_suite`, `database_query_tool`, `MCP_Listener`）。
    *   **结果：** 一个轻量级、专门化的「数据分析代理人」启动完毕，并进入待命状态。

---

#### **阶段四：跨代理人协作与任务完成**

9.  **主代理人分配具体任务:**
    *   **角色：** 「主代理人」的 LLM。
    *   **动作：** `agent:start` 工具成功返回后，LLM 知道工作代理人已就绪。它生成一个对 `mcp:send` 工具的调用，将具体的数据和分析指令发送给刚刚创建的工作代理人。

10. **工作代理人执行子任务:**
    *   **角色：** 「工作代理人」。
    *   **动作：**
        *   它的 `MCP_Listener` 接收到任务指令，并将其推入自己的事件队列。
        *   它的执行循环调用自己的 LLM 和专业工具（如 `code_interpreter`）处理数据。
        *   完成后，通过 `mcp:send` 工具将分析结果（报告、图表）发回给「主代理人」。

11. **主代理人整理并响应:**
    *   **角色：** 「主代理人」。
    *   **动作：** 收到工作代理人返回的结果，进行最终的整理和包装。
    *   **结果：** 通过其 `UI_Listener`（例如 `WhatsApp_Listener` 的输出接口）将最终报告发回给人类用户。
    *   **结果：** 通过其 `UI_Listener`（例如 `WhatsApp_Listener` 的输出接口）将最终报告发回给人类用户。
