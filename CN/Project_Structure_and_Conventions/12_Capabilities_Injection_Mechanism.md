# 12. 能力发现与依赖注入机制 (Capabilities Discovery & Injection)

本文档定义了 OpenStarry 协调层 (Coordination Layer) 如何加载插件，并将其提供的功能注入到 Agent Core 的各个子系统中。这是实现「微内核架构」与「动态扩展」的关键机制。

## 1. 机制概述 (Overview)

我们采用 **控制反转 (IoC)** 与 **依赖注入 (DI)** 模式。

*   **Host (The Driver):** 这是启动 Agent 的宿主进程（CLI 或 Daemon）。**唯有 Host 具备物理 I/O 权限**，能读取磁盘上的插件文件。
*   **PluginLoader (The Injector):** 运行于 Host 语境中，负责将物理文件转化为内核对象。
*   **Core (The Recipient):** 纯净的接收者，坐享其成地获取注入的能力。

> **💡 设计哲学 (Design Philosophy):**
> 这对应了「五蕴」中的「色不异空」。Plugin (色) 是具体功能的聚合，Core (空) 是纯粹的执行容器。Loader 的职责就是打破聚合，将能力填入容器。

---

## 2. 核心接口定义 (Core Interfaces)

所有的交互都基于 `@openstarry/sdk` 中定义的契约。

### 2.1 插件接口 (The Plugin Contract)

每个插件必须导出一个实现此接口的类。

```typescript
export interface IPlugin {
  readonly id: string;
  readonly version: string;
  
  /**
   * 初始化插件。
   * @param context - 协调层提供的注册上下文，用于回传能力。
   */
  initialize(context: IPluginContext): Promise<void>;
  
  /**
   * 资源清理 (如关闭 WebSocket 连线)。
   */
  shutdown(): Promise<void>;
}
```

### 2.2 注册上下文 (The Registration Context)

这是协调层暴露给插件的 API，用于收集能力。

```typescript
export interface IPluginContext {
  // 基础设施
  readonly logger: ILogger;
  readonly config: Record<string, any>; // 来自 agent.json 的该插件配置段落

  // [行] 注册工具：供 LLM 调用的函数
  registerTool(tool: ITool): void;
  
  // [受] 注册监听器：触发 Agent 运行的外部事件源
  registerListener(listener: IListener): void;
  
  // [想] 设定模型：Agent 的大脑 (通常互斥，后注册者覆盖或报错)
  setLLMProvider(provider: ILLMProvider): void;
}
```

---

## 3. 加载与注入流程 (Loading Sequence)

以下是 `Agent Core` 启动时，`PluginLoader` 的标准作业程序：

1.  **解析配置与发现 (Resolve & Discovery):** 
    *   读取 `agent.json` 中的 `plugins` ID 列表。
    *   **查询全局注册表:** 调用 `PluginRegistryService.resolve(id)`。
    *   **获取路径:** 注册表返回插件的准确物理路径（已处理好系统/项目优先级）。

2.  **动态载入 (Dynamic Import):**
    使用 Node.js 的 `import()` 或 `require()` 加载该路径下的入口文件。

3.  **实例化 (Instantiate):**
    创建 `IPlugin` 的实例。

4.  **初始化与注入 (Initialize & Inject):**
    *   Loader 创建一个 `PluginContext` 实例，绑定到当前的 Core 实例（ToolRegistry, EventBus）。
    *   调用 `plugin.initialize(context)`。
    *   **插件代码执行:** 插件内部调用 `context.registerTool(...)`。
    *   **实际绑定:** `PluginContext` 接收到 Tool 后，将其存入 Core 的 `ToolRegistry`。

5.  **生命周期管理:**
    Loader 将插件实例保存在内部 Map 中，以便在 Agent 关闭时调用 `shutdown()`。

---

## 4. 代码示例 (Implementation Example)

### 插件端写法 (`plugins/standard/fs/index.ts`)

```typescript
import { IPlugin, IPluginContext } from '@openstarry/sdk';
import { ReadFileTool } from './tools/ReadFile';

export default class FileSystemPlugin implements IPlugin {
  id = 'openstarry-fs';
  version = '1.0.0';

  async initialize(context: IPluginContext) {
    context.logger.info('正在挂载文件系统能力...');
    
    // 注入工具
    context.registerTool(new ReadFileTool());
    
    // 如果配置允许，还可以注入写入工具
    if (context.config.enableWrite) {
       // ... register WriteFileTool
    }
  }
  
  async shutdown() {}
}
```

### 核心端写法 (`packages/core/infrastructure/PluginLoader.ts`)

```typescript
class PluginContextImpl implements IPluginContext {
  constructor(private core: AgentCore, private pluginConfig: any) {}

  registerTool(tool: ITool) {
    // 直接操作核心的注册表
    this.core.toolRegistry.register(tool);
  }
  
  // ... 其他方法
}
```

---

## 5. 错误处理

*   **加载失败:** 若 `require` 失败，Loader 应记录错误但不应导致 Core 崩溃（除非是关键插件）。
*   **初始化超时:** `initialize` 方法应设有超时限制，防止插件卡死启动流程。
*   **冲突处理:** 若不同插件注册了同名的 Tool（如两个插件都有 `read_file`），Loader 应根据配置策略（覆盖、报错、或命名空间隔离）进行处理。
