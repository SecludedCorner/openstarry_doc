# 13. 复合体与依赖机制 (Composites & Dependencies)

本文档定义了如何构建 **「复合体 (Composites)」** —— 即那些不生产底层能力，而是组合现有 Provider (想) 与 Tool (行)，并加上自定义 Listener (受) 来创造新应用场景的硬代码模块。

## 1. 核心概念：能力依赖 (Capability Dependency)

为了保持解耦，复合体 (Composite) **不应** 直接依赖 Plugin A 的代码。它应该依赖 Plugin A 所提供的 **「能力标签 (Capability Tag)」**。

*   ❌ **错误依赖:** `import { GoogleSearch } from 'openstarry-plugin-google'`
*   ✅ **正确依赖:** `dependencies: { "tools": ["search"] }`

这样，无论用户安装的是 Google Search 还是 Bing Search，只要它提供了 `search` 能力，Plugin C 都能运作。

## 2. 依赖声明 (Manifest Declaration)

在 `plugin.json` 或 `package.json` 中，新增 `dependencies` 字段，指向其他**插件 ID**：

```json
{
  "id": "workflow-engine",
  "version": "1.0.0",
  "openstarry": {
    "type": "composite",
    "dependencies": {
      "plugins": ["@openstarry-plugin/standard-function-skill"] // 硬依赖
    }
  }
}
```

## 3. 基础设施依赖 (Infrastructure Dependency)

某些高阶插件（如工作流引擎）依赖于系统的基础解析能力。

*   **场景：** 需要解析 Markdown 技能文件。
*   **开发实作：**
    ```typescript
    initialize(context: IPluginContext) {
      // 从协调层注入的依赖中获取 Parser
      const parser = context.dependencies['@openstarry-plugin/standard-function-skill'].parser;
      
      if (!parser) throw new Error("Missing Skill Parser dependency");
    }
    ```

## 4. 协调层的职责 (Validation)

当 `PluginLoader` 加载 Plugin C 时，它会执行 **依赖检查 (Dependency Check)**：

1.  检查 Core 的 `ProviderRegistry` 是否已注册了 LLM。
2.  检查 Core 的 `ToolRegistry` 是否存在带有 `web-search` 和 `fs-write` 标签的工具。
3.  **结果:** 如果检查失败，Plugin C 将**拒绝启动**，并报错提示用户安装缺失的插件。

## 4. 开发实作：如何调用能力？

在插件代码中，我们通过 `AgentContext` (注意：不是初始化时的 `PluginContext`，而是运行时事件带来的 `AgentContext`) 来获取能力。

```typescript
// plugins/experimental/research-assistant/index.ts

export default class ResearchPlugin implements IPlugin {
  async initialize(context: IPluginContext) {
    // 注册一个监听器，等待任务
    context.registerListener({
      id: 'research-trigger',
      onEvent: async (event, agentOps: IAgentOperations) => {
        
        // 1. 获取大脑 (想)
        const llm = agentOps.getLLM();
        
        // 2. 获取工具 (行)
        // 系统会自动查找标签为 'web-search' 的工具
        const searchTool = agentOps.findToolByTag('web-search');
        
        if (!searchTool) {
          throw new Error("Missing required capability: web-search");
        }

        // 3. 执行逻辑
        const query = await llm.generate(`Extract keywords from: ${event.content}`);
        const results = await searchTool.call({ query });
        
        // 4. 反馈
        agentOps.reply(`Search results: ${results}`);
      }
    });
  }
}
```

## 5. 总结：生态系的层级

这种机制自然形成了生态系的层级：

*   **Level 1 (Foundation):** 提供原子能力的插件 (`fs`, `http`, `gemini`).
*   **Level 2 (Composites):** 组合 L1 能力的复合插件 (`researcher`, `coder`, `writer`).
*   **Level 3 (Agents):** 组合 L2 Composites 的完整数字物种。

这让开发者可以像搭积木一样构建复杂应用。
