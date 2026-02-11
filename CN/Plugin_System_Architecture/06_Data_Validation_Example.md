# 插件示例：数据验证 (Zod)

本文档阐述了像 `Zod` 这样的数据验证库，如何在我们的架构中发挥作用。

## 定位：库/依赖，而非插件

首先，`Zod` 本身**不是一个插件**，而是一个可以被多个组件引用的**第三方库 (Library / Dependency)**。它的作用是为系统的各个部分提供运行时的数据类型检查和验证，从而极大地增强系统的健壮性。

---

## 应用场景

`Zod` 可以在我们架构的以下几个关键位置被集成和使用：

### 1. 在 `Tool` 插件的定义中

这是 `Zod` 最直觉的应用场景。

*   **目标：** 确保 LLM 生成的工具调用参数，在传递给 `execute` 方法之前是类型安全且符合预期的。
*   **实现思路：**
    1.  在 `Tool` 插件的定义中，`args` 字段可以直接是一个 `Zod` schema 对象。
        ```javascript
        import { z } from 'zod';

        class MyTool {
          name = "create_user";
          description = "Creates a new user in the system.";
          
          args = z.object({
            username: z.string().min(3),
            email: z.string().email(),
            age: z.number().optional()
          });

          async execute(args) {
            // 此处的 'args' 已经是被 Zod 验证和转换过的对象
            // ...
          }
        }
        ```
    2.  在「代理人核心」的「执行循环」中，当它准备调用一个工具时，它会先从 `ToolRegistry` 获取工具的 `args` schema。
    3.  然后，它使用 `schema.safeParse(llm_generated_args)` 来验证 LLM 提供的参数。
    4.  如果验证失败，它会将 `Zod` 生成的详细错误信息作为「工具执行失败」的结果，反馈给 LLM，让 LLM 自我纠正并生成正确的参数。
    5.  如果验证成功，才调用工具的 `execute` 方法，并传入经过 `Zod` 清洗和类型转换后的参数。

### 2. 在 `MCP` 消息总线中

*   **目标：** 确保在「消息总线」上流动的所有消息封包都符合预定义的格式。
*   **实现思路：**
    *   在「消息总线引擎」中，定义一个 `MCP_Packet_Schema` 的 Zod schema。
    *   当消息总线从任何来源（如 `mcp:send` 工具）接收到一个消息时，它会首先使用 `MCP_Packet_Schema.safeParse(message)` 进行验证。
    *   只有验证通过的消息才会被路由到目标组件，格式错误的消息会被拒绝并记录日志。

### 3. 在 `Listener` 插件中

*   **目标：** 验证来自不可信外部事件源的数据。
*   **实现思路：**
    *   一个 `Webhook_Listener` 在接收到外部的 HTTP `POST` 请求时，其 `body` 内容是不可信的。
    *   在将请求内容打包成核心事件对象之前，`Listener` 应该使用一个预定义的 Zod schema 来验证 `body` 的结构和类型。
    *   这确保了推入核心「内部事件队列」的 `payload` 始终是干净和结构化的。

通过在这些关键数据交换的「边界」上使用 `Zod`，我们可以构建一个防御性的、类型安全的代理人系统，有效避免因 LLM 的「幻觉」或外部脏数据导致的的运行时错误。
