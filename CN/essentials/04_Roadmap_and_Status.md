# OpenStarry：系统概览与架构演进

> *"我们不只构建 Chatbot，我们构建的是数字物种的操作系统。"*

## OpenStarry 包含什么

一个完整的无头 AI 代理操作系统：

| 层次 | 组件 |
|-----|------|
| **五蕴 SDK** | TypeScript 接口：IUI、IListener、IProvider、ITool、IGuide |
| **Agent Core** | 6 状态执行循环、EventBus、上下文管理、会话隔离 |
| **安全系统** | 3 级断路器（资源级 → 行为级 → 人工覆盖） |
| **插件生态系统** | 传输层（stdio、WebSocket、HTTP+SSE）、Provider（Gemini OAuth）、工具（文件系统）、Guide（人设、技能） |
| **会话管理** | 逐连接隔离、会话恢复、统一 TraceId |
| **编排守护进程** | `openstarryd` — 生命周期管理、进程隔离（Docker、WASM）、状态持久化 |
| **MCP 协议** | 代理间协作，通过 JSON-RPC 2.0、工具暴露、递归守卫 |
| **TUI 仪表板** | 实时代理监控、交互式设计器、插件编排 |
| **可观测性** | 结构化 JSON 日志、Trace 上下文传播、健康检查 |

## 如何构建的：八个演进阶段

OpenStarry 不是一次性构建完成的。它经历了八个精心策划的架构阶段，每个阶段添加新的能力层——就像一个数字有机体从胚胎发育到成熟。

### 阶段 1：创世 — 骨架

Monorepo 脚手架、TypeScript 严格配置、pnpm workspace 协议以及五蕴 SDK 接口。在这个阶段，代理只是一组类型定义——纯粹的潜能，就像有机体形成之前的 DNA。

**关键交付物：** 11 个具有清晰依赖边界的 workspace 包。

### 阶段 2：意识内核 — 心跳

执行循环状态机、EventBus、上下文管理以及带断路器的安全监控器。代理获得了它的「心跳」——一个持续的感知→思考→行动→学习循环。

**关键交付物：**
- 6 状态执行循环（WAITING → ASSEMBLING → AWAITING_LLM → PROCESSING → EXECUTING_TOOLS → SAFETY_LOCKOUT）
- Token 预算（100k）、循环上限（50 次迭代）、重复失败检测（SHA-256 指纹）
- 异步 FIFO 事件队列，解耦生产者和消费者
- 滑动窗口上下文管理，带系统提示词锚定

### 阶段 3：身体与感官 — 第一批器官

插件基础设施：带自动钩子注册的 Plugin Loader、带 Zod→JSON Schema 转换的 Tool Registry、Provider Registry、UI Registry、Listener Registry、Guide Registry。空的 Core 现在可以接收器官了。

**关键交付物：** 一个通用插件加载系统，`安装一个 npm 包 = 获得完整的领域能力`。

### 阶段 4：第一次呼吸 — 活了

CLI Runner（`apps/runner`）、stdio 插件（终端 I/O）、Gemini OAuth Provider（大脑）、文件系统工具（双手）和人设初始化 Guide（灵魂）。第一次，一个 OpenStarry 代理可以感知、思考、行动和说话了。

**关键交付物：**
- 端到端代理生命周期：启动 → 加载插件 → 监听 → 响应 → 关闭
- OAuth 2.0 + PKCE 认证，带 AES-256-GCM 机器绑定令牌加密
- 路径沙箱化的文件系统工具，带 Zod 验证
- 痛觉机制：将错误视为反馈而非崩溃

### 阶段 5：多通道 — 多个躯体

WebSocket 和 HTTP 传输插件。同一个代理现在可以同时通过终端、WebSocket 或 HTTP API 接入。会话隔离赋予每个连接独立的对话状态。

**关键交付物：**
- WebSocket：双向通信、ping/pong 健康检查、会话恢复
- HTTP：REST 端点 + Server-Sent Events 实时流式传输
- Transport Bridge：将事件从 Core 路由到所有已注册的 UI，带错误隔离
- 逐连接会话管理，带向后兼容的默认会话

### 阶段 6：分形社会 — 代理协作

MCP（Model Context Protocol）集成。代理可以向其他代理暴露工具、调用其他代理的工具并组建动态团队。

**关键交付物：**
- MCP Server：任何代理都可以通过 JSON-RPC 2.0 暴露其工具
- MCP Client：任何代理都可以调用其他代理的工具
- 工具白名单：`expose_tools` / `private_tools` 配置
- 递归守卫：TraceId + 深度计数器（最大 5 层）防止无限循环
- DevTools：用于检查代理内部状态的调试接口
- 分形组合：代理团队暴露与单个代理相同的接口

### 阶段 7：守护进程 — 持久生命

编排守护进程（`openstarryd`）。代理成为真正的操作系统级进程，具有持久的生命周期管理。

**关键交付物：**
- 代理生命周期管理：生成、监控、重启、终止
- 状态持久化和恢复：代理在重启后存活，记忆延续
- 进程隔离：Docker 容器、WASM 沙箱
- 硬件抽象层 (HAL)：摄像头、传感器、执行器——物理世界中的代理
- 自动启动注册表：代理在系统启动时启动

### 阶段 8：操作系统演进 — 操作系统

TUI 仪表板和交互式代理设计器。完整愿景实现：一个数字生命的操作系统。

**关键交付物：**
- **TUI 仪表板**（`openstarry`）：实时监控所有运行中代理——CPU、内存、思考/秒、状态
- **交互式设计器**（`openstarry design`）：可视化组装代理的五蕴——选择大脑、感官、工具、灵魂
- **工作流引擎**（`openstarry run workflow.yaml`）：将代理串联成多步骤工作流，支持动态交接
- **插件同步**（`openstarry plugin sync`）：管理和更新能力库
- **代理注册**（`openstarry register`）：注册代理以进行守护进程管理

## 技术规范

七份技术规范定义了系统的契约：

| 规范 | 范围 | 关键细节 |
|-----|------|---------|
| **01: Command Registry** | 插件 CLI 命令注册 | 动态发现，`registry.json` 作为唯一数据源 |
| **02: Event Bus Protocol** | 标准事件信封 | UUID、timestamp、traceId、source、sessionId、priority |
| **03: Plugin Interfaces** | 五蕴 SDK | IUI、IListener、IProvider、ITool、IGuide + IPluginContext |
| **04: Context Management** | 分层记忆 | 即时层（5-10 轮，锁定）→ 短期层（滑动窗口）→ 长期层（RAG） |
| **05: Security Protocol** | 多层防御 | 文件系统沙箱、命令白名单、资源配额（50 tick、100k token、30s 超时） |
| **06: MCP Protocol** | 代理间通信 | JSON-RPC 2.0、工具暴露白名单、递归守卫（最大 5 层） |
| **07: Management Zone** | 守护进程架构 | 进程/Docker/WASM 隔离、IoT 的 HAL、YAML 编排规则 |

## 可插拔的记忆策略

不同的代理需要不同的记忆。OpenStarry 的上下文管理是插件而非硬编码：

| 策略 | 工作原理 | 最适用于 |
|-----|---------|---------|
| **滑动窗口**（默认） | FIFO — 保留最近 N 轮，丢弃最旧的 | 简单问答、短任务 |
| **动态摘要** | 使用轻量 LLM 将旧对话轮次压缩为自然语言摘要 | 长期陪伴、复杂项目 |
| **关键状态提取** | 从对话中提取结构化 JSON 状态，丢弃散文 | 表单填写机器人、预订代理 |

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

## 代理配置

一个完整的代理通过其五蕴以声明式方式定义：

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
    // [受+色] 网络躯体：WebSocket 传输
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
|-----|------|
| Workspace 包 | 11 |
| 插件包 | 7+ |
| 架构文档 | 27 |
| 深度解析文章 | 14 |
| 技术规范 | 7 |
| 实施计划 | 9+ |
| 执行循环状态 | 6 |
| 事件类型 | 25+ |
| 文档行数 | 10,000+ |
