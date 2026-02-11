# 18. 插件加载与注册协议 (Plugin Loading Protocol)

本文档定义了 OpenStarry 系统如何从磁碟加载插件代码，并将其与核心系统进行对接的详细协议。这是 Phase 3 的核心技术规范。

## 1. 核心挑战

由于插件是「五蕴聚合体」，而 Core 内部的运作是分模块的（ToolManager, ListenerManager 等），加载器必须解决以下问题：
1.  **动态性：** 插件代码在编译时未知。
2.  **解构 (Destructuring)：** 将一个插件包拆解为多个功能单元。
3.  **依赖注入 (Dependency Injection)：** 让插件获得 Core 的能力（如 Logger, Config），而不让插件依赖 Core 的实现。

## 2. 加载协议：工厂模式 (Factory Pattern)

为了最大化灵活性，我们放弃「类别扫描」模式，采用**工厂函数模式**。

> **重要更新：** 原有的 `BUILTIN_FACTORIES` 硬编码映射表已被移除。所有插件（包括内建插件）现在统一通过动态加载机制载入：优先使用 `ref.path`（本地路径），否则使用 `import(ref.name)`（NPM 包名）。这确保了插件加载逻辑的一致性，消除了内建与外部插件之间的区别。

### 插件入口规范 (`index.ts`)

每个插件必须导出一个名为 `initialize` 的异步函数，或者是实现了 `IPluginFactory` 接口的默认导出。

```typescript
import { IPluginContext, IPlugin } from '@openstarry/sdk';
import { MyTool } from './tools/my-tool';
import { MyListener } from './listeners/my-listener';

// 插件工厂函数
export default async function initialize(context: IPluginContext): Promise<void> {
  // 1. 获取 Core 注入的依赖
  const logger = context.logger;
  logger.info('Initializing My Plugin...');

  // 2. 注册行蕴 (Tools)
  // 这里插件主动将自己的 Tool 实例交给 Core
  context.registerTool(new MyTool());

  // 3. 注册受蕴 (Listeners)
  context.registerListener(new MyListener(context.config));
  
  // 4. 注册识蕴 (Guide)
  context.registerGuide({
    systemPrompt: "...",
    memoryPolicy: "sliding-window"
  });
}
```

## 3. 宿主端加载流程 (`PluginLoader` Logic)

`PluginLoader` 运行在 **协调层 (Coordination Layer / Daemon)** 或 **Host Process** 中。

### 步骤 A: 物理读取
1.  读取 `~/.openstarry/plugins/<plugin-id>/package.json`。
2.  解析 `main` 字段找到入口文件 (e.g. `dist/index.js`)。
3.  使用 `import()` (ESM) 或 `require()` (CJS) 动态载入该模块。

### 步骤 B: 上下文构建 (Context Construction)
Loader 为该插件创建一个专属的 `PluginContext` 实例。这个 Context 是插件与外界沟通的唯一管道。

```typescript
const context: IPluginContext = {
  logger: new ScopedLogger(`Plugin:${pluginId}`),
  config: agentConfig.plugins[pluginId] || {}, // 从 agent.json 读取配置
  
  // 注册回调 (Callback)
  registerTool: (tool) => core.toolRegistry.add(tool),
  registerListener: (listener) => core.listenerRegistry.add(listener),
  registerProvider: (provider) => core.providerRegistry.add(provider),
  registerGuide: (guide) => core.guideRegistry.set(guide),

  // 输入注入 — 插件可透过 pushInput 向 Agent 主动推送讯息
  pushInput: (input) => core.inputQueue.push(input),
};
```

### 步骤 C: 初始化与解构
调用插件的 `initialize(context)`。
*   此时，插件内部的逻辑开始执行，并透过 `register*` 方法将其五蕴成分「交出」。
*   Core 接收到这些成分后，将其分别归档到对应的管理器中。

## 4. 错误隔离与沙箱

*   **加载失败：** 如果 `initialize` 抛出异常，Loader 应捕获并记录「痛觉」，但不能让整个 Agent 崩溃。该插件将被标记为 `FAILED`。
*   **依赖注入安全：** `IPluginContext` 只暴露安全的 API。插件无法直接访问 Core 的私有属性或宿主的 `process` 对象（除非在 Level 0 信任模式）。

## 5. 协调层注册

所有成功加载的插件，其元数据（ID, Version, Capabilities）会被回报给 **Agent Coordination Layer**。这使得协调层能够回答「谁有天气查询功能？」这类问题，实现跨 Agent 的动态路由。

---

## 6. 依赖解析与拓扑加载 (Dependency Resolution & Topological Loading)

为了防止因加载顺序错误导致的系统崩溃（例如：Workflow 插件在 Skill 插件之前加载，导致无法获取 Markdown Parser），Loader **必须** 实现依赖图解析算法。

### 两阶段加载流程 (Two-Phase Loading)

#### Phase A: 扫描与建图 (Scan & Graph)
1.  Loader 扫描所有目标插件目录。
2.  读取每个插件的 `package.json` 中的 `openstarry.dependencies` 字段。
    ```json
    "dependencies": {
      "plugins": ["@openstarry-plugin/skill"] // 声明对其他插件的硬依赖
    }
    ```
3.  在内存中构建 **依赖有向图 (Dependency Directed Graph)**。
    *   节点：Plugin ID
    *   边：Dependency -> Dependent

#### Phase B: 排序与执行 (Sort & Execute)
1.  **循环侦测 (Cycle Detection):** 检查图中是否存在循环依赖 (A -> B -> A)。若发现，立即抛出 `FatalDependencyError` 并终止启动。
2.  **拓扑排序 (Topological Sort):** 使用 Kahn 算法或 DFS 计算出线性的加载顺序。
    *   *Result:* `['@openstarry-plugin/skill', '@openstarry-plugin/gemini', '@openstarry-plugin/workflow']`
3.  **依序初始化:** 按照排序后的清单，依次调用 `initialize(context)`。
    *   这确保了当 `workflow` 初始化时，`skill` 已经准备好并将 Parser 注入到了系统中。

### 依赖注入的时机
跨插件的服务（如 `markdownParser`）必须在 **Phase B** 的排序过程中被动态注入。
*   当 `@openstarry-plugin/skill` 初始化完成后，它会返回（或注册）其服务实例。
*   Loader 暂存这些实例。
*   当轮到 `@openstarry-plugin/workflow` 初始化时，Loader 从暂存区取出 `markdownParser` 并放入 `context.dependencies`。
