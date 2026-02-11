# 10. 能力插件设计：代理人管理器

本文档详细阐述了在最终架构下，`AgentManagerTool` 的设计和职责。

## 核心定位：决策者，而非执行者

在引入 `Orchestrator Daemon` 的最终架构中，`AgentManagerTool` 的角色被进一步提纯。它不再亲自执行创建 OS 进程的「脏活」，而是扮演一个**「决策接口」**的角色。

它负责将「主代理人」的 LLM 作出的「我需要一个新代理人」的**决策**，转化为一个对 `Daemon` 的**标准化请求**。

---

## `AgentManagerTool` 设计

*   **插件类型：** `tool`

*   **提供的方法 (作为独立的工具):**
    *   **`agent:start(agent_id: string, template_id: string)`**
        *   **功能：** **请求** `Orchestrator Daemon` 根据一个预定义的模板，创建并启动一个新的工作代理人。
        *   **实现：** 此方法的 `execute` 逻辑现在非常简单：
            1.  构造一个 JSON 请求体，如 `{ "agent_id": "...", "template_id": "..." }`。
            2.  向 `Orchestrator Daemon` 暴露的管理 API 端点（例如 `http://localhost:5050/agents`）发送一个 `POST` 请求。
            3.  等待 `Daemon` 的 API 返回结果（例如，包含成功状态和 `agent_id` 的 JSON），并将其返回给 LLM。
    *   **`agent:stop(agent_id: string)`**
        *   **功能：** **请求** `Orchestrator Daemon` 停止并销毁一个正在运行的工作代理人。
        *   **实现：** 向 `Daemon` 的 API 端点（例如 `http://localhost:5050/agents/{agent_id}`）发送一个 `DELETE` 请求。
    *   **`agent:status(agent_id: string)`**
    *   **`agent:list()`**
        *   **实现：** 同样地，这些方法都会转化为对 `Daemon` 管理 API 的 `GET` 请求。

---

## 总结

这种设计的优势是显而易见的：

*   **职责分离：**
    *   **主代理人 (LLM):** 负责最高层 combat 业务决策（**Why & What** - 为什么需要一个代理人，需要什么样的代理人）。
    *   **`AgentManagerTool`:** 负责将决策转化为标准的 API 请求（**How to Ask** - 如何提出创建请求）。
    *   **`Orchestrator Daemon`:** 负责所有底层的、与操作系统相关的实现细节（**How to Do** - 如何真正创建和管理一个进程）。
*   **可扩展性：** 这种基于 API 的设计，未来可以轻松地将 `Daemon` 扩展为一个能够在多台机器上分布式地创建代理人的集群管理器，而 `AgentManagerTool` 的接口无需任何改变。
*   **安全性：** 所有创建/销毁进程的权力都集中在 `Daemon` 一处，便于进行统一的审计和权限控制。
    *   **主代理人 (LLM):** 负责最高层的业务决策（**Why & What** - 为什么需要一个代理人，需要什么样的代理人）。
    *   **`AgentManagerTool`:** 负责将决策转化为标准的 API 请求（**How to Ask** - 如何提出创建请求）。
    *   **`Orchestrator Daemon`:** 负责所有底层的、与操作系统相关的实现细节（**How to Do** - 如何真正创建和管理一个进程）。
*   **可扩展性：** 这种基于 API 的设计，未来可以轻松地将 `Daemon` 扩展为一个能够在多台机器上分布式地创建代理人的集群管理器，而 `AgentManagerTool` 的接口无需任何改变。
*   **安全性：** 所有创建/销毁进程的权力都集中在 `Daemon` 一处，便于进行统一的审计和权限控制。
