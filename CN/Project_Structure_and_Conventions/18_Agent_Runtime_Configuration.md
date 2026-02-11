# 18. 代理人运行时配置逻辑 (Agent Runtime Configuration)

本文件补充了 `05_Agent_Manifest_Specification.md`，专注于 **Runtime (运行时)** 如何解析、验证并应用 `agent.json` 中的配置，特别是插件的启用与参数注入。

## 1. 配置加载流程

当 `openstarry start` 或 `Daemon` 启动一个 Agent 时：

1.  **读取：** 从 `configs/agent.json` 读取原始 JSON。
2.  **合并 (Merger):**
    *   如果 `agent.json` 定义了 `extends: "generic-assistant-v1"`，系统会先加载该模板的配置，然后将当前项目的配置覆盖上去 (Deep Merge)。
    *   这允许项目只定义差异部分（例如：只修改了 Prompt 或增加了一个 Tool）。
3.  **环境变量注入：** 解析 `${ENV_VAR}` 语法，注入 API Key 等敏感信息。

## 2. 插件启用与参数注入

这是配置中最关键的部分：如何让 Agent 知道自己有哪些能力。

### 配置结构

```json
"capabilities": {
  "plugins": {
    "@openstarry-plugin/fs": {
      "enabled": true,
      "config": { "root": "./workspace" } // 注入给 initialize(context)
    },
    "@openstarry-plugin/weather": {
      "enabled": false // 显式禁用
    }
  }
}
```

### 运行时逻辑

1.  **过滤：** Loader 遍历 `plugins` 列表，过滤掉 `enabled: false` 的项目。
2.  **上下文构建：**
    *   对于每个启用的插件（例如 `@openstarry-plugin/fs`），Loader 提取其 `config` 对象（例如 `{ "root": "./workspace" }`）。
    *   将此对象放入 `IPluginContext.config` 中。
3.  **初始化：** 调用插件的 `initialize(context)`。
    *   插件内部通过 `context.config.root` 获取参数，并据此设定 `fs` 工具的路径限制。

## 3. 五蕴配置映射

`agent.json` 的 `cognition` 与 `identity` 区块，实际上是直接配置 **Guide (识)** 类型的插件。

*   **System Prompt:** 如果 `agent.json` 定义了 `system_prompt`，Core 会将其封装为一个临时的 `AnonymousGuidePlugin`，并优先级设为最高。
*   **Provider:** `cognition.provider` 的配置会直接传递给对应的 Provider 插件（如 `gemini`）。

## 4. 错误处理与缺省行为

*   **无插件启动 (Zero Plugin State):**
    *   若系统启动时检测不到任何 Plugin (或 `~/.openstarry/plugins` 为空)，Agent Core **不应崩溃**。
    *   系统应进入 **「等待模式 (Idle Mode)」**，并在终端或 Dashboard 显示醒目提示：
        > "⚠️ No plugins found. Please run `openstarry plugin sync` to install standard capabilities."
*   **缺少依赖：** 如果配置启用了 `plugin-x` 但系统找不到，Agent 启动应报错并终止（Fail Fast），除非该插件被标记为 `optional: true`。
*   **配置无效：** 插件应在 `initialize` 阶段验证传入的 `config`，若无效则抛出 `ConfigurationError`。
