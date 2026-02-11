# 27. 系统拓扑与管理层架构 (System Topology & Management Zone)

本文档定义了 OpenStarry 的宏观运作逻辑：它是一个由「管理层」提供土壤与养分，并让「核心层」通过「插件」实现物种多样性的现代 AI 操作系统。

---

## 1. Agent 协调层 (The Management Zone)

**定位：系统的宿主环境（Host）与行政中枢。**
这一层不负责「思考」，它负责「确保思考环境的稳定与安全」。

### 1.1 容器层 (Plumbing)
*   **机制**：采用动态链接或容器化技术（如 WebAssembly 沙盒或特定进程隔离）。
*   **职责**：实现 `AgentLoader`。当系统需要一个「开发者」时，容器层负责从仓库抓取 `AgentCore` 镜像，并将指定的 `Plugins` 通过 **依赖注入 (DI)** 挂载进去。

### 1.2 规划与调度层 (Orchestration)
*   **机制**：基于 **因果链 (Causality Chain)** 的事件驱动架构。
*   **职责**：它不监控 Agent 的私有隐私，它监控 **「边界事件」**。当 A Agent 抛出一个 `TaskCompleted` 事件，调度层负责判断这是否为唤醒 B Agent 的「缘」。

### 1.3 策略与安全层 (Policy)
*   **机制**：拦截器 (Interceptors) 与 资源配额管理 (Quota Management)。
*   **职责**：执行「戒律」。例如限制测试 Agent 的网络访问权限，或强制限制开发 Agent 的 Token 使用预算。

### 1.4 资源与环境层 (Environment)
*   **机制**：硬件抽象层 (HAL)。
*   **职责**：将物理实体（如 PiKVM 的画面、NVIDIA Jetson 的温度）转换为标准的 **感官数据流** 供应给 Agent。

### 1.5 多元交互层 (Interface)
*   **机制**：状态投影 (State Projection)。
*   **职责**：将看不见的 Agent 思考过程（来自协议插件的导出数据）转化为人类可读的仪表板。

---

## 2. Agent Core (The Autonomous Life Zone)

**定位：纯粹的「五蕴」计算循环。**
它是空的，它唯一的职责是**维持循环的运行**。

### 五蕴能力层 (Aggregate)
*   **受 (Input)**：接收来自环境层的信号。
*   **想 (Reasoning)**：调用大模型进行语义解析。
*   **行 (Action)**：产生操作工具的意图。
*   **识 (Integrator)**：整合所有插件的信息，维持自我的连贯性。

**核心特性**：它是 **Stateless (无状态)** 的。所有的状态都寄存在「记忆插件」中，所有的通讯规范都依赖「协议插件」。这意味着 Core 可以在不同的容器间无损迁移。

---

## 3. 能力插件 (The Plugins)

**定位：赋予 Agent 个性、专业与灵魂的功能组件。**

### 3.1 数据与协议插件 (Internal Protocol Plugin)
*   **工程细节**：定义内部的消息传递总线（Internal Bus）。
*   **场景应用**：
    *   **高速模式**：开发人员 Agent 使用 Protobuf 协议，追求极速的代码生成与传输。
    *   **透明模式**：测试人员 Agent 使用「日志追踪协议」，记录下每一秒的神经传导，以便在出错时进行因果回溯。

### 3.2 评估与进化插件 (Reflection Plugin)
*   **工程细节**：实现「双层推理」机制。在 Action 输出前，由评估插件进行拦截（Self-check）。
*   **场景应用**：
    *   **轻量化**：简单的文档整理 Agent 可以不挂载此插件，降低延迟。
    *   **深度决策**：涉及工厂设备操作的 Agent，必须挂载具备「物理规则验证」的反思插件，避免执行危险动作。

### 3.3 状态与记忆插件 (Memory Plugin)
*   **工程细节**：抽象化存储接口 `IMemoryStore`。
*   **场景应用**：
    *   **短期战场**：使用 `In-Memory` 插件，让开发人员在撰写当前 Function 时拥有极快的 Context 切换。
    *   **长期经验**：挂载 `Vector-RAG` 插件，让 Agent 记得三个月前解决类似 Bug 的经验。

---

## 4. OpenStarry 架构的终极运作流程 (The Lifecycle)

这是一个从「缘起」到「寂灭」的完整生命周期：

1.  **缘起 (Origination)**：环境层侦测到硬件错误。
2.  **调度 (Scheduling)**：调度层判断需要「维修专家」，通知容器层。
3.  **生起 (Arising)**：容器层载入 **Agent Core**，并插上 **诊断协议**、**高阶反思** 与 **设备知识库记忆** 三个插件。
4.  **运行 (Operation)**：Agent Core 在插件的加持下，开始处理「痛觉」，产生修复指令。
5.  **寂灭 (Cessation)**：任务完成，容器层销毁 Core 实例，记忆插件将经验存回数据库。
