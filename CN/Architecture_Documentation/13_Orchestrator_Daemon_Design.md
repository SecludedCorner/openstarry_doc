# 12. 运行时核心：编排器守护进程 (Orchestrator Daemon)

本文档详细阐述了 `Orchestrator Daemon` 的设计，这是在我们最终架构中，负责管理所有代理人 OS 进程生命周期的关键后台服务。

## 核心定位：代理人系统的「init/systemd」

`Orchestrator Daemon` 是一个**常驻后台的、与代理人业务逻辑无关**的系统级服务。它不参与任何 LLM 的思考或任务的执行，其职责非常纯粹和底层。

*   **类比：** 它就像 Linux 系统中的 `systemd` 或 `init` 进程，或者容器化世界中的 `Docker Compose` 或 `Kubernetes Kubelet`。
*   **职责：**
    1.  **进程生命周期管理：** 负责启动、停止、监控和重启所有「代理人核心」的独立操作系统进程。
    2.  **基础设施插件托管：** 负责加载 `infrastructure` 类型的插件，为代理人提供本地共享服务（如消息总线、状态数据库），确保环境的开箱即用与可替换性。
    3.  **资源管理：** 监控每个代理人进程的资源消耗（CPU、内存），并可以根据策略进行管理（例如，终止消耗过多的进程）。
    4.  **提供管理接口：** 暴露一组内部 API，供 `AgentManagerTool` 或外部管理工具调用，以程序化的方式管理代理人实例。

---

## 工作流程

### 1. 系统启动

*   系统管理员在服务器上执行 `openstarry-daemon start` 命令，启动守护进程。
*   `Orchestrator Daemon` 启动后，它做的第一件事就是根据其主配置文件，启动并开始监护**「主代理人 (Master Agent)」**的 OS 进程。

### 2. 代理人创建 (由 `AgentManagerTool` 触发)

1.  「主代理人」的 `AgentManagerTool` 被调用，意图启动一个新的工作代理人。
2.  `AgentManagerTool` 的 `execute` 方法不再是自己创建进程，而是向本地运行的 `Orchestrator Daemon` 发起一个 API 请求。
    *   **请求示例：** `POST http://localhost:5050/agents`
    *   **请求体：**
        ```json
        {
          "agent_id": "data_worker_007",
          "template_id": "DataAnalystAgent_v1"
        }
        ```
3.  `Orchestrator Daemon` 接收到 API 请求后，执行以下操作：
    *   向「代理人设计层」查询 `DataAnalystAgent_v1` 模板的配置。
    *   准备好启动新代理人进程所需的环境变量和命令行参数。
    *   使用 `child_process.spawn()` 或类似的系统调用，创建一个新的、独立的 OS 进程。
    *   将新进程的 `PID` 和 `agent_id` 记录在自己的监控列表中。
    *   开始监控新进程的健康状况（例如，通过心跳或 `PID` 是否存在）。

### 3. 代理人销毁

*   当 `AgentManagerTool` 调用 `agent:stop(agent_id)` 时，它会向 Daemon 发起 `DELETE /agents/{agent_id}` 的 API 请求。
*   `Orchestrator Daemon` 收到请求后，负责安全地终止对应的 OS 进程并清理相关资源。

---

## 结论

引入 `Orchestrator Daemon` 是我们架构演进的最终形态。它完美地解决了多进程架构下的资源管理和生命周期监护问题，同时让「主代理人」和 `AgentManagerTool` 的职责保持纯粹——它们只负责**决策**（「我需要一个新代理人」），而将**执行**（「如何实际创建一个 OS 进程并监控它」）的脏活累活交给了这个专业的底层服务。
