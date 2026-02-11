# 00. OpenStarry 设计哲学 (Design Philosophy)

本文档是理解 OpenStarry 架构的起点。它不涉及具体的代码实现，而是阐述我们为什么要构建这样一个系统，以及指导我们所有技术决策的核心价值观。

---

## 核心愿景：从「脚本」到「物种」

目前的 AI Agent 大多仍停留在「脚本 (Script)」的阶段：它们被触发，执行一段逻辑，然后结束。它们像是一个个单次运行的函数。

OpenStarry 的愿景是将 Agent 升华为 **「操作系统进程 (OS Process)」** 甚至是一个 **「数字物种 (Digital Species)」**。

我们认为，一个真正的 Agent 应该具备以下特质：
1.  **持久性 (Persistence):** 它拥有自己的生命周期，可以休眠、唤醒，且记忆是连续的。
2.  **主体性 (Identity):** 它不依赖于外部的触发者而存在。它有自己的状态、配置和「灵魂」。
3.  **社会性 (Sociality):** 它天生具备与其他 Agent 协作的能力，就像人类天生具备语言能力一样。

---

## 四大设计支柱

### 1. 内核与外设分离 (The Kernel-Peripheral Separation)
这是我们从 Linux 设计哲学中学到的最重要一课。

*   **Agent Core (内核)** 是纯粹的。它只负责思考 (LLM 交互)、记忆 (Context 管理) 和决策 (Action 生成)。它不知道自己是运行在 CLI 里，还是在 WhatsApp 上。
*   **Plugins (外设)** 负责所有与物理世界的交互。所有的 I/O（输入/输出）、所有的通讯协议（WebSocket, MCP）、所有的具体能力（读文件, 发邮件）都是插件。

**意义：** 这保证了 Agent 的「灵魂」可以移植到任何「躯壳」中。

### 2. 拟人化的认知流 (Anthropomorphic Cognitive Flow)
我们不把 Agent 看作是一个 `Request -> Response` 的服务器，而是看作一个拥有 **「意识流 (Stream of Consciousness)」** 的实体。

*   **执行回路 (Execution Loop):** 这是 Agent 的心跳。它不断地感知环境（事件队列），处理信息，并做出行动。
*   **短期记忆 (Working Memory):** 就像人类的注意力有限，Agent 的 Context Window 也是稀缺资源。我们必须有策略地管理它（遗忘、摘要），而不是简单地堆砌数据。

### 3. 分形协作结构 (Fractal Collaboration)
OpenStarry 的系统结构是分形的 (Fractal)。

*   一個 Agent 可以只是一個简单的工具（如「搜索 Agent」）。
*   但它也可以是一个复杂的团队（如「研发部 Agent」），其内部由无数个小 Agent 组成。
*   对外，它们都暴露相同的接口 (MCP)。这使得我们可以像搭积木一样，用简单的 Agent 构建出极其复杂的组织，而无需改变架构。

### 4. 错误即反馈 (Error as Feedback)
在传统软件中，错误 (Exception) 通常意味着崩溃或中断。
在 OpenStarry 中，**错误是学习的机会**。

*   当工具调用失败，Core 不会崩溃，而是将错误信息作为「感官输入」反馈给 LLM。
*   LLM 会「意识到」自己做错了，并尝试修正（例如：换一个参数，或改用另一个工具）。
*   这种**「自我修正回路」**是智能的体现。

---

## 结语

OpenStarry 不是一个简单的 SDK，它是一套**「代理人操作系统」**的蓝图。我们希望开发者在阅读后续文档时，始终牢记：**你不是在写代码，你是在创造一个生命。**