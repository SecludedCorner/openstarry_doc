# 11. 能力插件设计：工作流引擎 (WorkflowEngineTool)

本文档描述了将宏观、多步骤的任务编排能力，封装成一个强大的 `Tool` 插件的设计。

## 核心定位：从「架构层」到「能力插件」

在「以代理人为中心」的模式下，复杂工作流的执行能力，不再由一个独立的架构层来承担，而是被**封装成一个名为 `WorkflowEngineTool` 的工具插件**，供「主代理人」或其他被授权的高级代理人按需调用。

这使得「代理人核心」保持了纯净，它无需知道工作流如何执行，只需知道如何调用一个名为 `workflow:execute` 的工具。

---

## `WorkflowEngineTool` 设计

*   **插件类型：** `tool`

*   **提供的工具：**
    *   `workflow:execute(workflow_id: string, initial_payload: object)`: 执行一个预定义好的工作流。
    *   `workflow:status(execution_id: string)`: 查询一个正在运行的工作流的状态。

*   **`plugin.json` 示例：**
    ```json
    {
      "name": "Workflow-Engine-Tool",
      "type": "tool",
      "entryPoint": "./WorkflowEngineTool.js"
    }
    ```

*   **内部实现：**
    1.  **工作流定义：** 插件内部包含一个解析器，用于读取工作流定义文件（例如 `./workflows/my_flow.yaml`）。这些文件描述了工作流的图结构、节点和数据流。
    2.  **执行引擎：** 当 `workflow:execute` 被调用时，插件内部的执行引擎会：
        *   加载对应 `workflow_id` 的定义文件。
        *   创建一个工作流实例，并用 `initial_payload` 初始化其上下文。
        *   开始异步地、按照图的依赖关系执行各个节点。
    3.  **节点执行器：** 插件内部会有多种类型的「节点执行器」。
        *   `ApiCallNodeExecutor`: 负责执行 HTTP API 调用。
        *   `DatabaseNodeExecutor`: 负责执行数据库查询。
        *   `CodeNodeExecutor`: 负责执行一小段数据转换代码。
        *   `AgentNodeExecutor`: **(关键)** 如果工作流中的一个节点需要一个代理人来完成，这个执行器会调用 `AgentManagerTool` 来创建一个临时的「工作代理人」，并通过 MCP 与其通信来完成该节点的任务。
    4.  **返回结果：** `workflow:execute` 工具的调用可以被设计为异步的。它会立即返回一个 `execution_id`，代理人可以稍后使用 `workflow:status` 来查询结果。或者，工作流执行完毕后，可以通过 `mcp:send` 将最终结果异步地发送回调用者。

---

## 优势

*   **能力内聚：** 所有与工作流相关的复杂逻辑，都被完美地封装在这一个 `Tool` 插件中。
*   **核心纯净：** 「代理人核心」完全无需关心工作流是如何被定义和执行的。
*   **按需使用：** 只有「主代理人」或被授权的高级代理人才会被加载这个 `WorkflowEngineTool`，普通的「工作代理人」则不需要，保持了轻量。
