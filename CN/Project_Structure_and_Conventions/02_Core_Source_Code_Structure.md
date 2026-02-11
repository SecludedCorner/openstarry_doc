# 02. Core 源码目录结构规范 (Core Source Code Structure)

本文档定义了 `packages/core/src/` 内部的代码组织方式。此结构旨在实现「高度解耦、可测试、且符合拟人化架构」的开发目标。

## 源码目录树状图

```text
src/
├── agents/             # 代理人主体类别 (Agent Entities)
├── execution/          # 执行、调度与事件流 (The Loop)
├── memory/             # 上下文管理与记忆策略 (Working Memory)
├── infrastructure/     # 插件加载、注册与隔离 (Plugin Infra)
├── security/           # 安全拦截与断路器 (Guardrails)
├── state/              # 状态快照与持久化 (State Manager)
├── transport/          # 核心通讯桥接层 (Communication Bridge)
├── types/              # TypeScript 强类型定义
└── index.ts            # 库入口文件
```

---

## 模块职责详解 (含五蕴哲学映射)

### 1. `execution/` (魂魄：执行系统) —— [映射：行蕴 Samskara / 识蕴 Vijnana]
这是 Agent 的生命中枢，实现了「执行回路」与「控制理论」模型。
*   **`loop/`**: 实现 `tick()` 机制，负责从事件队列取件、调用 LLM 并触发行动（意志与造作）。
*   **`queue/`**: 管理异步事件优先级。

### 2. `memory/` (记忆：上下文策略) —— [映射：想蕴 Samjna]
实现 `10_Context_Management_Strategy` 中定义的逻辑。
*   **`context/`**: 负责对接收到的信息进行识别、关联与压缩（认知与模式识别）。

### 3. `infrastructure/` (器官：插件管理) —— [映射：色蕴 Rupa]
实现「一切皆插件」的物理加载。
*   **`loader/` & `registry/`**: 负责扫描并存储 Agent 的「身体部件」。当插件加载时，它将其具体功能（色）拆解并注册到系统中。

### 4. `security/` (超我：安全层) —— [映射：受蕴 Vedana 的防御面]
夹在「决策」与「执行」之间的防火墙。
*   **`guardrails/`**: 负责捕获异常并将其转化为 Agent 的「负面感受（痛觉）」，从而触发自我修正。

### 5. `transport/` (感官：通讯桥接) —— [映射：受蕴 Vedana 的感知面]
实现 `02_Communication_Interface` 定义的 API。
*   **`bridge/`**: 负责对接外部刺激（眼、耳、身等监听器），将外界信息转化为内部感受。

---

## 命名惯例 (Naming Conventions)

1.  **类别 (Classes):** 使用大驼峰命名，如 `AgentCore`, `ExecutionLoop`。
2.  **接口 (Interfaces):** 以 `I` 开头，如 `IPlugin`, `ITool`。
3.  **文件后缀:** 
    *   逻辑实现：`.ts`
    *   单元测试：`.test.ts` (放在与源文件同级的目录)
4.  **入口:** 每个目录应包含一个 `index.ts` 用于导出公共 API，保持外部引用的简洁。

---

## 代码演进原则

*   **无副作用 (No Side Effects):** 除了 `infrastructure/` 和 `transport/` 之外，其余模块应尽量保持纯粹逻辑，不直接操作网络或文件系统。
*   **依赖注入 (Dependency Injection):** `AgentCore` 类在初始化时，应接收 `IContextManager`、`IStateManager` 的实例，以便在测试时可以轻易替换为 Mock 版本。
    *   **无副作用 (No Side Effects):** 除了 `infrastructure/` 和 `transport/` 之外，其余模块应尽量保持纯粹逻辑，不直接操作网络或文件系统。
*   **依赖注入 (Dependency Injection):** `AgentCore` 类在初始化时，应接收 `IContextManager`、`IStateManager` 的实例，以便在测试时可以轻易替换为 Mock 版本。
