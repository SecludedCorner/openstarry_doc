# 17. 宿主引导模式 (Host Bootstrapping Pattern)

本文档解答了一个经典的架构悖论：**「如果 Agent Core 是绝对纯净且无 I/O 能力的，它如何读取硬盘上的配置并加载插件？」**

答案是：**它不读取，也不加载。这些都是『宿主 (Host)』的工作。**

## 1. 问题：鸡生蛋悖论 (The Bootstrap Paradox)

我们要求 `packages/core`：
1.  **无头 (Headless):** 不知道运行环境。
2.  **纯净 (Pure):** 不依赖 Node.js `fs` 或 `module` 模块。
3.  **微内核 (Microkernel):** 所有能力（包括读档）都来自插件。

但如果它连读档都不行，它怎么读取 `agent.json` 来知道自己需要加载 `fs-plugin` 呢？

## 2. 解法：宿主层 (The Host Layer)

我们在架构中明确区分了两个运行时角色：

### 角色 A: 宿主 (Host / Coordinator)
*   **实体:** `apps/runner`, `apps/daemon`, `apps/web-server`。
*   **权限:** 拥有完整的操作系统权限（读写文件、网络请求、进程管理）。
*   **职责:** **开拓者**。它负责准备环境、读取配置、将插件从硬盘搬运到内存。

### 角色 B: 内核 (Core)
*   **实体:** `packages/core` 的实例。
*   **权限:** 零。
*   **职责:** **继承者**。它被动接收 Host 准备好的资源，并开始逻辑运算。

## 3. 启动流程详解 (The Sequence)

### Step 1. 宿主启动 (Host Awake)
当用户执行 `node apps/runner/dist/bin.js` 时，启动的是 **Host (Runner)**。
Host 此时使用原生的 `fs` 模块，去读取 `./agent.json`。

> **Host 视角:** "我读到了配置，这个 Agent 需要 `fs` 和 `http` 插件。"

### Step 2. 物理加载 (Physical Loading)
Host 根据配置，对每个插件尝试两种解析策略：(1) `ref.path` 指定的文件路径、(2) `import(ref.name)` 包名动态载入。使用 ESM 的 `import()` 将其载入内存。

> **Host 视角:** "我已经把 `fs-plugin.js` 载入到 RAM 了，它现在是一个对象。"

### Step 3. 依赖注入 (Injection)
Host 实例化 Core，并将这些**已经载入好的对象**传进去。

```typescript
// Host 的代码 (伪代码)
const loadedPlugins = [ fsPluginObject, httpPluginObject ];
const core = new AgentCore({ plugins: loadedPlugins });
```

### Step 4. 内核觉醒 (Core Awake)
Core 启动，它不需要去硬盘找插件，因为插件已经在它手里了。它只需要调用 `plugin.initialize()`。

> **Core 视角:** "我醒来了，我手里有工具，我不知道这些工具哪来的，但我可以用。"

## 4. 总结

**协调层 (Host) 负责「生存」，核心 (Core) 负责「生活」。**

*   **载入什么插件？** -> 需要看文件夹内容 -> 这是 **Host** 的工作。
*   **如何使用插件？** -> 这是 **Core** 的工作。

这个模式让我们保持了 Core 的绝对纯净，同时解决了物理加载的问题。
