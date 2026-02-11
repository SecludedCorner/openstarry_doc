# OpenStarry：系统概览与架构演进

> *"我们不只构建 Chatbot，我们构建的是数字物种的操作系统。"*

## OpenStarry 包含什么

一个完整的、Headless 的 AI Agent 操作系统：

| 层级 | 组件 |
|------|------|
| **五蕴 SDK** | TypeScript 接口：IUI、IListener、IProvider、ITool、IGuide |
| **Agent Core** | 6 状态执行循环、EventBus、上下文管理、Session 隔离 |
| **安全系统** | 3 层断路器（资源级 → 行为级 → 人为覆写） |
| **插件生态系** | Transport（stdio、WebSocket、HTTP+SSE）、Provider（Gemini OAuth）、Tools（文件系统）、Guides（角色、技能） |
| **Session 管理** | 每连接隔离、Session 恢复、统一 TraceId |
| **Orchestrator Daemon** | `openstarryd`——生命周期管理、程序隔离（Docker、WASM）、状态持久化 |
| **MCP 协议** | 跨 Agent 协作，基于 JSON-RPC 2.0、工具暴露、递归防护 |
| **TUI 仪表板** | 实时 Agent 监控、交互式设计器、插件编排 |
| **可观测性** | 结构化 JSON 日志、Trace 上下文传播、健康检查 |

## 如何建造的：八个演进阶段

OpenStarry 不是一次建成的。它经历了八个精心规划的架构阶段逐步演进，每个阶段都增加一层新的能力——就像一个数字生物体从胚胎发育到成熟。

### 第一阶段：创世纪——骨架

Monorepo 脚手架、TypeScript strict 设定、pnpm workspace protocol，以及五蕴 SDK 接口。在这个阶段，Agent 只是一组类型定义——纯粹的潜能，就像生物体成形前的 DNA。

**关键产出：** 11 个 workspace 套件，具有清晰的依赖边界。

### 第二阶段：意识核心——心跳

执行循环状态机、EventBus、上下文管理，以及带有断路器的安全监控器。Agent 获得了「心跳」——一个持续的感知 → 思考 → 行动 → 学习循环。

**关键产出：**
- 6 状态执行循环（WAITING → ASSEMBLING → AWAITING_LLM → PROCESSING → EXECUTING_TOOLS → SAFETY_LOCKOUT）
- Token 预算（100k）、循环上限（50 次迭代）、重复失败侦测（SHA-256 指纹识别）
- 异步 FIFO 事件队列，解耦生产者与消费者
- 滑动视窗上下文管理，搭配 System Prompt 锚定

### 第三阶段：身体与感官——第一批器官

插件基础设施：Plugin Loader 自动 Hook 注册、Tool Registry 搭配 Zod→JSON Schema 转换、Provider Registry、UI Registry、Listener Registry、Guide Registry。空的 Core 现在可以接收器官了。

**关键产出：** 一个通用的插件加载系统，「安装一个 npm 套件 = 获得完整的领域能力」。

### 第四阶段：第一口呼吸——活了

CLI Runner（`apps/runner`）、stdio 插件（终端 I/O）、Gemini OAuth Provider（大脑）、文件系统工具（双手）、以及角色初始化 Guide（灵魂）。一个 OpenStarry Agent 第一次能够感知、思考、行动并说话。

**关键产出：**
- 端到端 Agent 生命周期：启动 → 加载插件 → 监听 → 响应 → 关闭
- OAuth 2.0 + PKCE 认证搭配 AES-256-GCM 机器绑定 Token 加密
- 路径沙箱化的文件系统工具，搭配 Zod 验证
- 痛觉机制：错误作为反馈，而非崩溃

### 第五阶段：多通道——多副身体

WebSocket 和 HTTP 的 Transport 插件。同一个 Agent 现在可以同时通过终端、WebSocket 或 HTTP API 被访问。Session 隔离为每个连接提供了独立的对话状态。

**关键产出：**
- WebSocket：双向通讯、ping/pong 健康检查、Session 恢复
- HTTP：REST 端点 + Server-Sent Events 用于实时流式传输
- Transport Bridge：将事件从 Core 路由至所有已注册的 UI，具备错误隔离
- 每连接 Session 管理，向下兼容默认 Session

### 第六阶段：碎形社会——Agent 协作

MCP（Model Context Protocol）整合。Agent 可以将工具暴露给其他 Agent、调用其他 Agent 的工具，并形成动态团队。

**关键产出：**
- MCP Server：任何 Agent 都可以通过 JSON-RPC 2.0 暴露其工具
- MCP Client：任何 Agent 都可以调用其他 Agent 的工具
- 工具白名单：`expose_tools` / `private_tools` 设定
- 递归防护：TraceId + 深度计数器（最大 5 层）防止无限循环
- DevTools：用于检视 Agent 内部状态的调试界面
- 碎形组合：Agent 团队对外暴露与个别 Agent 相同的接口

### 第七阶段：Daemon——持久的生命

Orchestrator Daemon（`openstarryd`）。Agent 成为真正的 OS 层级程序，具备持久的生命周期管理。

**关键产出：**
- Agent 生命周期管理：生成、监控、重启、终止
- 状态持久化与复原：Agent 在重启后存活，记忆向前传承
- 程序隔离：Docker 容器、WASM 沙箱
- 硬件抽象层（HAL）：摄像头、传感器、执行器——Agent 进入物理世界
- 自动启动注册：Agent 在系统开机时启动

### 第八阶段：OS 演进——操作系统

TUI 仪表板与交互式 Agent 设计器。完整愿景的实现：数字生命的操作系统。

**关键产出：**
- **TUI 仪表板**（`openstarry`）：实时监控所有运行中的 Agent——CPU、内存、thoughts/sec、状态
- **交互式设计器**（`openstarry design`）：视觉化地组装 Agent 的五蕴——选择大脑、感官、工具、灵魂
- **工作流程引擎**（`openstarry run workflow.yaml`）：将 Agent 串联成多步骤工作流程，支持动态交接
- **插件同步**（`openstarry plugin sync`）：管理与更新能力库
- **Agent 注册**（`openstarry register`）：将 Agent 注册至 Daemon 进行管理

## 技术规格

七份技术规格定义了系统的契约：

| 规格 | 范畴 | 关键细节 |
|------|------|---------|
| **01: Command Registry** | 插件 CLI 命令注册 | 动态发现，`registry.json` 为唯一真实来源 |
| **02: Event Bus Protocol** | 标准事件封包 | UUID、timestamp、traceId、source、sessionId、priority |
| **03: Plugin Interfaces** | 五蕴 SDK | IUI、IListener、IProvider、ITool、IGuide + IPluginContext |
| **04: Context Management** | 层级式记忆 | 实时（5-10 轮，锁定）→ 短期（滑动视窗）→ 长期（RAG） |
| **05: Security Protocol** | 多层防御 | 文件系统沙箱、命令白名单、资源配额（50 ticks、100k tokens、30s 超时） |
| **06: MCP Protocol** | 跨 Agent 通讯 | JSON-RPC 2.0、工具暴露白名单、递归防护（最大 5 层） |
| **07: Management Zone** | Daemon 架构 | Process/Docker/WASM 隔离、HAL 用于 IoT、YAML 编排规则 |

## 可插拔的记忆策略

不同的 Agent 需要不同的记忆方式。OpenStarry 的上下文管理是插件，不是写死的：

| 策略 | 运作方式 | 最适用场景 |
|------|---------|-----------|
| **滑动视窗**（默认） | FIFO——保留最近 N 轮，丢弃最旧的 | 简单问答、短期任务 |
| **动态摘要** | 使用轻量 LLM 将旧对话压缩为自然语言摘要 | 长期陪伴、复杂项目 |
| **关键状态提取** | 从对话中提取结构化 JSON 状态，丢弃叙述文字 | 表单填写机器人、订票 Agent |

```json
{
  "contextStrategy": {
    "type": "plugin",
    "pluginId": "std-summarization-strategy",
    "config": { "compressionModel": "gemini-1.5-flash", "threshold": 20 }
  }
}
```

> *"Core 始终保持轻量，复杂的记忆管理逻辑下放给了专门的策略模块。"*

## Agent 设置

一个完整的 Agent 通过其五蕴以宣告式方式定义：

```jsonc
{
  "identity": { "id": "dev-bot-01", "name": "Resilient Developer" },
  "plugins": [
    // [想] 大脑：认知引擎
    { "name": "@openstarry-plugin/provider-gemini" },
    // [行] 双手：文件系统操作
    { "name": "@openstarry-plugin/standard-function-fs" },
    // [受] 感官：终端输入
    { "name": "@openstarry-plugin/standard-function-stdio" },
    // [受+色] 网络身体：WebSocket 传输
    { "name": "@openstarry-plugin/transport-websocket" },
    // [识] 灵魂：人设与痛觉机制
    { "name": "@openstarry-plugin/guide-character-init",
      "config": { "characterFile": "./personas/developer.md" } }
  ],
  "policy": {
    "safety": { "max_consecutive_errors": 3 }
  }
}
```

## 数据一览

| 指标 | 数值 |
|------|------|
| Workspace 套件 | 11 |
| 插件套件 | 7+ |
| 架构文档 | 27 |
| 深度剖析文章 | 14 |
| 技术规格 | 7 |
| 实施计划 | 9+ |
| 执行循环状态 | 6 |
| 事件类型 | 25+ |
| 文档行数 | 10,000+ |
