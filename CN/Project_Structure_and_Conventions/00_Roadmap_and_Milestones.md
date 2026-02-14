# 00. 项目路线图与里程碑 (Project Roadmap and Milestones)

本文件定义了 OpenStarry 从核心原型到完整生态系的实现路径。此规划旨在确保项目演进的有序性，并为开源贡献者提供清晰的指引。

---

## 版本里程碑总览 (Version Milestones)

| 版本 | 内容 | 状态 |
|------|------|------|
| v0.1.0-alpha | MVP 基础架构 (Plan01-04) | ✅ 完成 |
| v0.2.0-beta | 多通道传输 — WebSocket + HTTP (Plan05) | ✅ 完成 |
| v0.2.1-beta | Session 隔离 + SSE + Health Check (Plan05.1, 05.2, 05.5-①) | ✅ 完成 (Cycle 1, 2026-02-10) |
| v0.2.2-beta | Metrics/Logging + Error Handling (Plan05.5-②③) | ✅ 完成 (Cycle 2, 2026-02-11) |
| v0.3.0-beta | MCP 协议整合 (Plan06) | ✅ 完成 (Cycle 3, 2026-02-11) |
| v0.3.5-beta | 管理层基础建设 (Management Zone Infra) | 规划中 |
| v0.4.0-beta | Runtime Sandbox MVP (Plan07) | ✅ 完成 (Cycle 5, 2026-02-11) |
| v0.4.1-beta | Sandbox Restart + Worker Pool (Plan07.1) | ✅ 完成 (Cycle 6, 2026-02-11) |
| v0.4.2-beta | Import Analysis + PKI Verification (Plan07.2) | ✅ 完成 (Cycle 7, 2026-02-11) |
| v0.4.3-beta | Audit Logging + Module._load (Plan07.3) | ✅ 完成 (Cycle 8, 2026-02-11) |
| v0.5.0-beta | TUI Dashboard MVP (Plan08) | ✅ 完成 (Cycle 9, 2026-02-11) |
| v0.6.0-beta | DevTools Plugin + E2E Testing (Plan11) | ✅ 完成 (Cycle 12, 2026-02-12) |
| v0.7.0-beta | DevTools Debugging + E2E Framework | ✅ 完成 (Cycle 12, 2026-02-12) |
| v0.8.0-beta | Daemon Mode MVP (Plan12) | ✅ 完成 (Cycle 13, 2026-02-12) |
| v0.9.0-beta | Seamless Attach (Plan13) | ✅ 完成 (Cycle 14, 2026-02-12) |
| v0.10.0-beta | MCP Resources + OAuth 2.1 (Plan06-P3) | ✅ 完成 (Cycle 15, 2026-02-12) |
| v0.11.0-beta | Multi-client Attach & Session Management (Plan14) | ✅ 完成 (Cycle 16, 2026-02-12) |
| v0.12.0-beta | MCP Sampling & Advanced Protocol Extensions (Plan06-P4) | ✅ 完成 (Cycle 17, 2026-02-12) |
| v0.13.0-beta | SDK Context Extensions & Provider Integration (Plan15) | ✅ 完成 (Cycle 18, 2026-02-12) |
| v0.14.0-beta | Security Hardening & Quality Polish (Plan16) | ✅ 完成 (Cycle 19, 2026-02-12) |
| v0.15.0-beta | Plugin Developer Experience (Plan17) | ✅ 完成 (Cycle 20, 2026-02-12) — 970 tests |
| v0.16.0-beta | Plugin Sync & System Plugin Directory (Plan18) | ✅ 完成 (Cycle 21, 2026-02-12) — 1009 tests |
| v0.17.0-beta | Plugin Dependency Wiring & Cross-Plugin Services (Plan19) | ✅ 完成 (Cycle 22, 2026-02-12) — 1067 tests |
| v0.18.0-beta | Workflow Engine MVP (Plan20) | ✅ 完成 (Cycle 23, 2026-02-12) — 1104 tests |
| v0.19.0-beta | Web-based Remote Attach (Plan21) | ✅ 完成 (Cycle 24, 2026-02-12) — 1132 tests |
| v0.20.0-beta | Plugin Marketplace MVP (Plan22) | ✅ 完成 (Cycle 25, 2026-02-13) — 1330 tests |
| v0.20.1-beta | Windows 跨平台修复 + Attach UX 改善 (Hotfix) | ✅ 完成 (Cycle 25-26, 2026-02-13~14) — 1339 tests |

---

## 阶段一：创世纪 (Phase 1: Genesis)
**目标：** 建立 Monorepo 物理基础与共用规范。
**状态：** 进行中 (In Progress) — 1.3 CI/CD 尚未建立

- [x] **1.1 Monorepo 初始化**
    - [x] 建立 `apps/` 与 `packages/` 目录结构。
    - [x] 配置 `pnpm-workspace.yaml` 与根目录 `package.json`。
    - [x] 统一 TypeScript、ESLint 与 Prettier 规范。
- [x] **1.2 核心 SDK 与类型定义**
    - [x] 建立 `packages/sdk` 定义完整的**五蕴插件接口**。
        *   **色 (IUI)**: 接口与呈现 (Form/UI)。
        *   **受 (IListener)**: 感官监听 (Perception/Input)。
        *   **想 (IProvider)**: 思考生成 (Cognition/LLM)。
        *   **行 (ITool)**: 工具执行 (Action/Will)。
        *   **识 (IGuide)**: 灵魂与人设 (Consciousness/Persona)。
    - [x] 建立 `packages/shared` 提供全局错误处理与日志模块。
- [ ] **1.3 工程化与测试基建 (Engineering & Testing Infrastructure)**
    - [x] 配置单元测试框架 (Vitest)。 *(Plan02 Phase C — 56 项测试通过)*
    - [x] 纯净性检查脚本 (`pnpm test:purity`)。 *(Plan03 Phase A4)*
    - [ ] 建立 CI/CD 流程 (GitHub Actions)。

## 阶段二：意识内核 (Phase 2: The Conscious Kernel)
**目标：** 实现「无头 (Headless)」、事件驱动的 `Agent Core`。
**状态：** 已完成 (Done)

- [x] **2.1 执行循环 (The Loop)**
    - [x] 实现 `packages/core/execution` (基于事件队列的 tick 机制)。
    - [x] ExecutionLoop 事件驱动重构：从 EventQueue pull 事件，状态机含 WAITING_FOR_EVENT / SAFETY_LOCKOUT。 *(Plan02 Phase A)*
    - [x] InputEvent payload 标准化（source, inputType, data, replyTo）。 *(Plan02 Phase A4)*
- [x] **2.2 状态与记忆管理**
    - [x] 实现 `packages/core/state` (支持 Snapshot 与持久化接口)。
    - [x] 实现 `packages/core/memory` (可插拔的上下文策略，如滑动窗口)。
- [x] **2.3 安全断路器与错误反馈 (Safety & Correction)**
    - [x] 实现 Token 消耗上限与无限循环侦测 (Circuit Breakers)。 *(Plan02 Phase B — SafetyMonitor)*
    - [x] 实现「错误即痛觉」机制，将执行时错误转化为 Context 反馈，触发 Agent 自我修正。 *(Plan02 Phase B — 挫折计数器 + 重复调用侦测 + 错误级联熔断)*

## 阶段三：肢体与感官 (Phase 3: Body and Senses)
**目标：** 构建完整的五蕴插件基础设施与标准库，建立安全边界。
**状态：** 进行中 (In Progress) — 核心插件已完成，进阶功能待实现

- [x] **3.1 核心加载协议 (Loading Protocol)**
    - [x] 实现 `PluginLoader`，支持**工厂模式 (Factory Pattern)** 初始化。 *(Plan01)*
    - [x] 实现 `IPluginContext`：包含 Logger, Config, 以及**跨插件服务注入 (dependencies 字段)**。 *(Plan19)*
    - [x] **实现回路编织逻辑 (Dependency Wiring):** 根据 `20` 号文件，在加载时自动对接 OODA 回路。 *(Plan19 — topologicalSort, cycle detection, AgentCore.serviceRegistry)*
    - [x] **实现指令注册表 (CommandRegistry):** 支持 CLI `run-tool` 与 Chat `/slash` 指令的映射与执行逻辑。 *(Plan01)*
    - [x] **实现动态插件加载 (Dynamic Loading):** CLI 支持 `agent.json` 的 `plugins[].path` 字段，通过 `import()` 动态加载第三方插件。 *(Plan02 Phase A3)*
    - [x] 实现 `agent.json` 的运行时验证器（Zod schema 验证）。 *(v0.1.5 - Plan03 补强完成)*
- [ ] **3.2 协调层与注册机制 (Coordination Layer)**
    - [x] **双轨扫描机制 (Dual-Path Scanning):** 同时扫描系统与项目私有插件目录。
    - [ ] **一键同步 (Sync):** 实现 `openstarry plugin sync` 指令，将官方 **`openstarry_plugin`** 仓库内容同步至系统目录。
    - [x] 在 Daemon 中实现 **Plugin Registry Service**，建立内存索引。
- [ ] **3.2 标准功能聚合插件库 (Standard Aggregate Plugins)**
    > **开发位置：** `openstarry_plugin` 仓库 (Ecosystem Repo)
    - [x] `@openstarry-plugin/standard-function-stdio`: 聚合 受(Stdio Listener) + 色(CLI Body) + 识(Default Guide)。 *(Plan01)*
    - [x] `@openstarry-plugin/provider-gemini-oauth`: 聚合 想(Gemini Provider)。PKCE + OAuth 2.0 认证。 *(Plan01)*
    - [x] `@openstarry-plugin/standard-function-fs`: 聚合 行(FS Tools) + 识(Path Scoping Policy)。 *(Plan01)*
    - [ ] `@openstarry-plugin/guide-mcp`: 聚合 识(MCP Guide)。赋予标准化通信能力。
    - [ ] `@openstarry-plugin/guide-pain-mechanism`: 实现拟人化痛觉诠释逻辑。
    - [x] `@openstarry-plugin/standard-function-skill`: 聚合 识(Skill Guide)。**核心依赖**：负责解析 Markdown 技能文件，是工作流与复杂 Agent 的基础。 *(Plan03 Phase B1)*
- [ ] **3.4 插件开发体验 (DX)**
    - [ ] 实现 `openstarry create-plugin` 脚手架。
    - [ ] 提供 `MockHost` 测试环境，支持插件的独立开发与验证。

> **安全备注：**
> 阶段三的 `fs` 工具虽然不运行在 Docker 中，但必须通过 `packages/shared` 的路径规范化组件，严格限制其操作范围。这允许 Agent 执行「增删文件夹」等必要操作，同时保护系统关键目录。

## 阶段四：降生 (Phase 4: The First Breath)
**目标：** 实现第一个单机版 CLI Agent，验证端到端流程。
**状态：** 进行中 (In Progress) — Runner 可运行，端到端 LLM 验证待 OAuth 登录后测试

- [x] **4.1 本地执行器 (Local Runner)**
    - [x] 实现 `apps/runner` 引导程序（纯启动 Bootstrap Runner），支持 `agent.json` 启动。 *(Plan01)*
    - [x] 支持动态插件解析（path / node_modules 双层策略，BUILTIN_FACTORIES 已移除，所有插件皆动态加载）。 *(Plan02 Phase A3)*
- [ ] **4.2 整合验证**
    - [ ] 达成「感知 -> 思考 -> 工具调用 -> 修正 -> 响应」的完整闭环。*(需 OAuth 登录后手动测试)*

> **里程碑：v0.1 Alpha (MVP)**
> *   ✅ 能在 CLI 中运行单一 Agent。
> *   ✅ 使用 Gemini 作为 Provider（PKCE + OAuth 认证）。
> *   ✅ 能操作本地文件系统 (`fs` tool)。
> *   ✅ 具备基本记忆能力 (5 轮对话)。
> *   ✅ 安全熔断机制（Token 预算、循环上限、错误级联、挫折计数器）。
> *   ✅ 结构化 JSON 日志（LOG_FORMAT=json, LOG_LEVEL 过滤）。
> *   ✅ agent.json Zod 运行时验证（配置错误即时报错）。
> *   ✅ 工具调用 Timeout（Promise.race 防卡死）。
> *   ✅ TraceID 机制（日志追踪一次完整处理周期）。
> *   ✅ Markdown 技能加载（standard-function-skill 插件）。
> *   ✅ guideFile 外部文件引用（system_prompt 从 .md 加载）。
> *   ✅ 纯净性检查脚本（pnpm test:purity）。
> *   ✅ 56 项单元测试通过（Vitest）。
> *   ⬜ 端到端 LLM 通话验证（待手动测试）。

---

> **以上完成 Agent 核心。以下进入「代理人操作系统」与生态系阶段。**

---

## 阶段 4.5：多通道传输 (Phase 4.5: Multi-Channel Transport)
**目标：** 验证 IUI/IListener 分离架构的可扩展性，实现非 stdio 的通道。
**状态：** 已完成 (v0.2.0-beta)

- [x] **4.5.1 WebSocket 传输插件**
    - [x] `@openstarry-plugin/transport-websocket`: WebSocket Listener + UI
    - [x] 支持多客户端连接、定向回复 (replyTo)
- [x] **4.5.2 HTTP Webhook 传输插件**
    - [x] `@openstarry-plugin/transport-http`: HTTP Listener + UI
    - [x] POST /api/input, GET /api/status, GET /api/response
- [x] **4.5.3 多 UI 同时输出**
    - [x] TransportBridge 广播机制验证通过
    - [x] stdio + WebSocket 同时收到事件

> **里程碑：v0.2.0 Beta (Multi-Channel)**
> *   ✅ WebSocket 传输插件实现完成
> *   ✅ HTTP Webhook 传输插件实现完成
> *   ✅ 多 UI 同时输出验证通过

---

## 阶段 4.6：实现周期一 (v0.2.1-beta)
**目标：** 解决多用户隐私问题，优化传输效率，建立连接健康检查。
**状态：** 已完成 (Cycle 1, 2026-02-10) — 118 tests, QA PASS, Architect PASS WITH NOTES

### Plan05.1: Session 隔离与消息路由 ✅
- [x] ISession / ISessionManager 接口 (SDK) + SessionManager 实现 (Core)
- [x] InputEvent.sessionId 字段 (可选，向下兼容)
- [x] IPluginContext.sessions 暴露 ISessionManager 给插件
- [x] Default session (`__default__`) 自动建立，无法销毁
- [x] WebSocket session-aware routing (per-session 广播)
- [x] 17 个新增测试覆盖 session isolation 场景

### Plan05.2: HTTP SSE 支持 ✅
- [x] 新增 `GET /api/events` SSE 端点
- [x] EventSource 兼容格式 (text/event-stream)
- [x] Session-scoped 事件过滤
- [x] SSE heartbeat 定时发送
- [x] 与现有 HTTP webhook listener 共存
- [x] 11 个新增测试覆盖 SSE 场景

### Plan05.5-①: 连接健康检查 ✅
- [x] WebSocket protocol ping/pong (server-initiated)
- [x] missedPongs 计数，超过 staleThreshold 断开
- [x] HTTP SSE heartbeat 定时检查
- [x] HealthCheckConfig 可配置 (enabled, intervalMs, staleThreshold)
- [x] 6 个新增测试覆盖 health check 场景

> **里程碑：v0.2.1-beta**
> *   ✅ 多客户端 session 隔离 (WebSocket + HTTP)
> *   ✅ HTTP SSE 即时流式传输
> *   ✅ 连接健康检查 (ping/pong + heartbeat)
> *   ✅ 118 tests, 11 packages, snapshot saved

---

## 阶段 4.7：实现周期二 (v0.2.2-beta)
**目标：** 建立可观测性基础建设与标准化错误处理。
**状态：** 已完成 (Cycle 2, 2026-02-11) — 165 tests, QA PASS, Architect PASS WITH NOTES

### Plan05.5-②: Metrics / Logging 基础建设 ✅
- [x] MetricsCollector (Core): increment / gauge / getSnapshot / reset
- [x] Logger.time() 方法 (performance.now() 计时)
- [x] METRICS_SNAPSHOT 事件类型
- [x] /metrics slash command (AgentCore)
- [x] Transport 插件 console.log → createLogger 迁移
- [x] 19 个新增测试

### Plan05.5-③: 标准化错误处理 ✅
- [x] ErrorCode const (12 个错误码)
- [x] TransportError / SessionError / ConfigError 类别
- [x] ES2022 Error cause chain 支持
- [x] 既有 2-arg 构造子向下兼容
- [x] 16 个新增测试

> **里程碑：v0.2.2-beta**
> *   ✅ MetricsCollector 可观测性基础
> *   ✅ 标准化错误层级 (ErrorCode + cause chain)
> *   ✅ Transport 插件结构化日志
> *   ✅ 165 tests, 11 packages, snapshot saved

---

## 阶段 4.8：MCP 协议整合 (v0.3.0-beta)
**目标：** 实现 OpenStarry Agent 与外部 MCP Server 的标准化互联。
**状态：** 已完成 (Cycle 3, 2026-02-11) — 86 tests, Architect PASS WITH NOTES

### Plan06: MCP Client Plugin ✅
- [x] `@openstarry-plugin/mcp-client` 主要实现
  - [x] 通用 MCP 传输层抽象 (stdio, SSE)
  - [x] StdioClientTransport 实现 (cross-platform: Windows/Unix)
  - [x] SSEClientTransport 实现
  - [x] RPC 通信层 (JSONRPCClient)
- [x] MCP Tool Bridge
  - [x] `mcp:` namespace 机制
  - [x] `tools/list` → OpenStarry Tool Registry 映射
  - [x] `tools/call` 执行与结果回传
- [x] MCP Prompt Bridge
  - [x] Slash command (`/mcp-prompt-name`) 映射
  - [x] `prompts/list` 清单
  - [x] `prompts/get` 取得完整 prompt 内容
- [x] `@openstarry-plugin/mcp-server` 服务器实现
  - [x] StdioServerTransport 实现
  - [x] MCP Server Protocol 处理 (initialize, tools/list, tools/call, prompts/*)
  - [x] JSON-RPC 回复机制
  - [x] 配套完整错误处理
- [x] `@openstarry-plugin/mcp-common` 共用类型
  - [x] MCP Protocol 接口定义
  - [x] Transport 抽象
  - [x] RPC 通信类型
- [x] 86 项单元测试
  - [x] MCP Client: 53 tests
  - [x] MCP Server: 33 tests

> **里程碑：v0.3.0-beta**
> *   ✅ MCP Client Plugin 完整实现
> *   ✅ MCP Server Plugin 完整实现
> *   ✅ Stdio 与 SSE 传输支持
> *   ✅ Tool 与 Prompt Bridge 互联
> *   ✅ 86 tests, 3 新增 packages, snapshot saved

---

## 阶段 4.9：Runtime Sandbox (v0.4.0-beta ~ v0.4.3-beta)
**目标：** 实现完整的运行时沙箱机制，确保插件安全隔离与控制。
**状态：** 已完成 (Cycles 5-8, 2026-02-11) — 442 tests total, QA PASS, Architect PASS WITH NOTES

### Plan07: Runtime Sandbox MVP (Cycle 5) ✅
**目标：** 建立 Worker 执行线程隔离与基础签名验证机制。
- [x] SandboxManager 与 Worker 执行线程管理
  - [x] 单一 Worker 初始化与运行
  - [x] RPC 通信机制 (cross-thread MessagePort)
  - [x] Timeout 与 Memory Limit 强制执行
- [x] 签名验证机制
  - [x] SHA-256 套件级验证
  - [x] Manifest 签名检查
  - [x] 验证失败的明确错误报告
- [x] 82 项新增测试

> **里程碑：v0.4.0-beta**
> *   ✅ Worker 执行线程隔离实现
> *   ✅ SHA-256 签名验证
> *   ✅ Memory + CPU timeout 控制

### Plan07.1: Worker Lifecycle + Pool (Cycle 6) ✅
**目标：** 实现 Worker 重启策略、心跳监控与连接池管理。
- [x] Worker 重启机制
  - [x] 指数退避重启策略 (exponential backoff: 100, 200, 400, 800ms → cap)
  - [x] 自动复原故障 Worker
- [x] 心跳监控 (Heartbeat)
  - [x] 定期心跳检查
  - [x] Stall 侦测与自动隔离
  - [x] Missing heartbeat 计数器
- [x] Worker Pool 管理
  - [x] 预先生成 Worker (pool sizing)
  - [x] 自动回收与重用
  - [x] Plugin reset via protocol message
- [x] 110 项新增测试

> **里程碑：v0.4.1-beta**
> *   ✅ 指数退避重启策略
> *   ✅ 心跳监控与自动恢复
> *   ✅ Worker Pool 连接池机制

### Plan07.2: Import Analysis + PKI (Cycle 7) ✅
**目标：** 实现静态分析防护与 Ed25519 PKI 验证。
- [x] 静态导入分析器 (AST-based)
  - [x] 递归式 require/import 扫描
  - [x] 白名单 (allowed) / 黑名单 (blocked) 模块检查
  - [x] 违规时拒绝加载 (严格模式)
- [x] Ed25519 PKI 签名验证
  - [x] 公钥配置机制
  - [x] Ed25519 签名检验
  - [x] 多个签名者支持
- [x] SandboxConfig 接口
  - [x] allowed / blocked modules 清单
  - [x] publicKeys 配置
  - [x] Runtime policy (strict/warn/off)
- [x] Worker Pool interface (initialize/acquire/release/shutdown)
- [x] 135 项新增测试

> **里程碑：v0.4.2-beta**
> *   ✅ 静态导入分析防护
> *   ✅ Ed25519 PKI 验证机制
> *   ✅ 模块黑白名单控制

### Plan07.3: Audit Logging + Final Hardening (Cycle 8) ✅
**目标：** 实现审计日志与 Module._load 拦截，完成沙箱硬化。
- [x] AuditLogger 机制
  - [x] 缓冲式 JSONL 写入
  - [x] 异步日志记录
  - [x] 密钥隐匿 (password/token/key/auth/credential)
- [x] Module._load 拦截
  - [x] 运行时模块加载控制
  - [x] Strict / Warn / Off 三种模式
  - [x] 调用堆栈追踪
- [x] RPC 审计整合
  - [x] 工具执行日志
  - [x] 错误追踪
  - [x] 性能指标
- [x] 日志轮转与清理
- [x] 115 项新增测试

> **里程碑：v0.4.3-beta**
> *   ✅ 完整审计日志机制
> *   ✅ Module._load 运行时控制
> *   ✅ 密钥隐匿与安全硬化
> *   ✅ 442 tests (Plan07 全线), snapshot saved

---

## 待实现项目 (Deferred Plan05.x)

### Plan05.3: DevTools 调试界面 ⬜
> 延后至 Plan06 (MCP) 之后实现。

### Plan05.4: E2E 测试脚手架 ⬜
- [ ] `createTestAgent()` 工具函数
- [ ] 支持 MockLLM 插件
- [ ] 降低社区贡献门槛

---

## 阶段五：守护进程 (Phase 5: The Orchestrator Daemon) — Plan07
**目标：** 实现进程级管理与持久化。
**状态：** 已完成 (v0.4.3-beta, Cycles 5-8, 2026-02-11)

- [ ] **5.1 守护进程核心 (Daemon)**
    - [ ] 建立 `apps/daemon`，管理多个 Agent 实例的生命周期。
    - [ ] 实现 **Agent 协调管理层 (Management Zone)** 中的调度逻辑。
- [ ] **5.2 API 网关与持久化**
    - [ ] 提供 HTTP/gRPC API 进行远程管理。
    - [ ] 整合数据库（SQLite/LevelDB）存储 Agent 长期状态。
- [ ] **5.3 安全与治理 (Security & Governance)**
    - [ ] 实现插件签名与验证机制 (确保加载的插件是受信任的)。
    - [ ] 实现运行时沙箱 (Sandbox) 策略 (如 Node.js vm 或 WASM 容器化)。
    - [ ] **实现环境隔离与资源配额 (Quota Management)**。
- [ ] **5.4 系统可观测性与硬件抽象 (Observability & HAL)**
    - [x] 结构化 JSON 日志，支持 agent_id / trace_id / module 字段。 *(Plan02 Phase D)*
    - [ ] 整合 OpenTelemetry 或 Event Tracing，追踪跨 Agent 的任务流动。
    - [ ] 实现日志收集，支持 Dashboard 的可视化呈现。
    - [ ] **实现硬件抽象层 (HAL) 标准感官数据流**。

## 阶段六：分形社会 (Phase 6: Fractal Society) — Plan06
**目标：** 实现代理人协作与 MCP 协议。
**状态：** 已完成 (v0.3.0-beta → v0.10.0-beta, Cycle 3 → Cycle 15, 2026-02-11 → 2026-02-12)
**前置条件：** Plan05.1 Session 隔离完成 ✅ (v0.2.1-beta, 2026-02-10)

> **进入检查清单：**
> - ✅ Plan05.1 Session 隔离已实现 (Cycle 1)
> - ✅ 多客户端场景验证通过 (118 tests)
> - ✅ 消息不会跨 Session 泄漏 (session-aware routing 验证)

- [x] **6.1 MCP 深度整合 (Plan06-P1: Tools)**
    - [x] 实现 `@openstarry-plugin/mcp-client`，让 Agent 具备标准化互联能力。
    - [x] 实现 `@openstarry-plugin/mcp-server`，支持多外部客户端同时连接
    - [x] 86 项单元测试验证完成 (v0.3.0-beta, Cycle 3)
- [x] **6.1a MCP Prompts 整合 (Plan06-P2: Prompts)**
    - [x] MCP Prompts RFC 0005 Section 5.2 实现完成
    - [x] `/mcp-prompt-name` Slash 命令支持
    - [x] 动态提示清单与内容检索
    - [x] 42 项新测试 (v0.9.0-beta 之前, Cycle 9+)
- [x] **6.1b MCP Resources 整合 (Plan06-P3: Resources)**
    - [x] MCP Resources RFC 0005 Section 5.5 实现完成
    - [x] listResources / readResource RPC 处理
    - [x] OAuth 2.1 token 管理 (PKCE, auto-refresh, TTL)
    - [x] AES-256-GCM 加密 + PBKDF2 (100k iterations)
    - [x] `/mcp-resources` / `/mcp-server-resources` Slash 命令
    - [x] 45 项新测试 (v0.10.0-beta, Cycle 15)
- [ ] **6.2 DevTools 调试界面 (Plan05.3)**
    - [ ] 静态 HTML 调试界面 `openstarry-devtools`
    - [ ] 左侧：对话气泡视图
    - [ ] 右侧：State/Context 检视
    - [ ] 底部：事件过滤器
- [ ] **6.3 工作流引擎 (Workflow Engine)**
    - [ ] 实现 `WorkflowEngineTool`，支持 YAML 定义的任务编排。
    - [ ] 实现 Agent 动态孵化与销毁机制 (Ephemeral Agents)。
- [ ] **6.3 跨 Agent 协作与因果调度 (Orchestration)**
    - [ ] 实现任务拆解与分发至不同子代理人的机制。
    - [ ] **实现基于因果链的事件调度 (Causality Chain)**。

## 阶段七：可视化与生态 (Phase 7: Ecosystem and UI)
**目标：** 提供 Web 图形化管理与部署工具。
**状态：** 待启动

- [ ] **7.1 Dashboard 与多元交互层 (Interface)**
    - [ ] 建立 `apps/dashboard` (React/Next.js)，可视化监控所有 Agent。
    - [ ] **实现状态投影 (State Projection) 仪表板**。
- [ ] **7.2 模板服务 (Agent Design Service)**
    - [ ] 提供 Agent 商店/模板下载功能，实现「目录即协议」的极简部署。

## 阶段八：终端机演进 (Phase 8: CLI & TUI Evolution) — Plan08 / Plan09
**目标：** 实现「操作系统级」的终端机体验，达成 User Scenario Guide 的愿景。
**状态：** 进行中 (v0.5.0-beta Plan08 完成，Plan09 待实现)

### Plan08: TUI Dashboard MVP (v0.5.0-beta) ✅
**状态：** 已完成 (Cycle 9, 2026-02-11) — 524 tests (+82 new), 43 test files, QA PASS, Architect PASS_WITH_NOTES

- [x] **8.1 运行时仪表板 (Runtime Dashboard)**
    - [x] 实现 `@openstarry-plugin/tui-dashboard` (使用 Ink v5 + React 18)
    - [x] 整合事件监控、状态显示、消息流式传输
    - [x] 支持消息分类、工具调用追踪、错误计数
    - [x] 键盘快捷键 (q = 离开, Tab = 切换事件日志)

### Plan12: Daemon Mode MVP (v0.8.0-beta) ✅
**状态：** 已完成 (Cycle 13, 2026-02-12) — 714 tests (+44 new), 18 packages, QA PASS, Architect PASS (1 rework)

- [x] **后台进程管理 (Daemon Process Management)**
    - [x] CLI 命令：`daemon start`, `daemon stop`, `daemon ps`
    - [x] 进程生成与分离 (detached process, unref)
    - [x] PID 文件管理 (`~/.openstarry/agents/{agent-id}.pid`)
    - [x] 优雅关闭 (SIGTERM/SIGINT 级联)
- [x] **IPC 层 (JSON-RPC over Unix Domain Socket)**
    - [x] Socket 通信：`~/.openstarry/agents/{agent-id}.sock`
    - [x] JSON-RPC 2.0 协议
    - [x] 健康检查 RPC：`agent.health` → `{ok, uptime, version}`
- [x] **Daemon 插件** (`@openstarry-plugin/daemon`)
    - [x] IPC 服务器整合
    - [x] 健康检查提供者 (IProvider)
    - [x] 进程生命周期管理
- [x] **测试涵盖**: 44 个新测试 (进程管理、IPC 通信、健康检查)

> **里程碑：v0.8.0-beta**
> *   ✅ Daemon 后台进程实现
> *   ✅ IPC 通信基础设施
> *   ✅ 714 tests, 18 packages, snapshot saved

### Plan13: Seamless Attach (v0.9.0-beta) ✅
**状态：** 已完成 (Cycle 14, 2026-02-12) — 747 tests (+33 new), 18 packages, QA PASS, Architect PASS (1 rework)

- [x] **IPC 协议扩展**
    - [x] `agent.attach(agentId)` → 连接终端客户端，返回 sessionId
    - [x] `agent.input(sessionId, message)` → 从附加客户端发送用户输入
    - [x] `agent.detach(sessionId)` → 优雅关闭终端会话（daemon 继续运行）
    - [x] 事件转发：Core.bus → IPC 桥接，sessionId 过滤
- [x] **CLI 命令** (`openstarry attach [agent-id]`)
    - [x] 列出运行中的 daemon
    - [x] 自动启动 daemon（若 agent.json 存在但 daemon 未运行）
    - [x] 终端 I/O 代理：stdin → agent.input RPC，daemon 事件 → stdout/stderr
    - [x] 优雅 Ctrl+C 处理（detach，不 kill daemon）
- [x] **事件转发器** (`event-forwarder.ts`)
    - [x] 会话过滤的事件递送
    - [x] 支持 LLM 响应、工具执行、错误与指标事件
    - [x] JSON 序列化（含时间戳）
    - [x] 大小限制 (64KB 消息，1MB 事件)
- [x] **安全性与验证**
    - [x] sessionId 格式验证 (UUID v4)
    - [x] inputType 白名单 (user, system)
    - [x] Agent 健康检查（attach 前验证 daemon 活力）
    - [x] 数据大小强制执行
- [x] **测试涵盖**: 33 个新测试 (attach 命令、IPC 处理器、事件转发、I/O 代理)

> **里程碑：v0.9.0-beta**
> *   ✅ 无缝连接至运行中的 daemon
> *   ✅ 交互式终端对话 (attach/input/detach)
> *   ✅ 自动启动与事件转发
> *   ✅ 747 tests, 18 packages, snapshot saved

### Plan20: Workflow Engine MVP (v0.18.0-beta) ✅
**状态：** 已完成 (Cycle 23, 2026-02-12) — 1104 tests (+37 new), QA PASS, Architect PASS

- [x] **工作流引擎插件** (`@openstarry-plugin/workflow-engine`)
  - [x] YAML 声明式多步骤工作流程编排 (Zod 验证)
  - [x] 4 个步骤执行器：tool, service, llm, command
  - [x] Mustache 模板插值 (`{{ }}` 语法)
  - [x] LLM 流式整合 (直接 IProvider AsyncIterable API)
  - [x] EventBus 观测 (step:start/end, workflow:start/end/error)
  - [x] LRU 缓存 (内存内，最多 100 个条目)
- [x] **测试涵盖**: 37 个新测试 (8 文件)

> **里程碑：v0.18.0-beta**
> *   ✅ Workflow Engine MVP 完成
> *   ✅ SDK/Core 零变更 (微内核纯度保持)
> *   ✅ 1104 tests, snapshot saved

### Plan21: Web-based Remote Attach (v0.19.0-beta) ✅
**状态：** 已完成 (Cycle 24, 2026-02-12) — 1132 tests, QA CONDITIONAL PASS

- [x] **3 个新/修改插件**
  - [x] `@openstarry-plugin/http-static` (新) — 静态 HTTP 文件服务器 (路径遍历防护、MIME 处理)
  - [x] `@openstarry-plugin/web-ui` (新) — 浏览器代理界面 (HTML 配置注入、会话恢复)
  - [x] `@openstarry-plugin/transport-websocket` (修改) — 认证/CORS/代理 IP 解析
- [x] **测试涵盖**: 147 个新测试 (6 文件)

> **里程碑：v0.19.0-beta**
> *   ✅ Web-based Remote Attach 完成
> *   ✅ WebSocket Token 认证 + CORS
> *   ✅ 1132 tests, snapshot saved

### Plan22: Plugin Marketplace MVP (v0.20.0-beta) ✅
**状态：** 已完成 (Cycle 25, 2026-02-13) — 1330 tests, QA PASS

- [x] **插件目录与安装器**
  - [x] 插件目录 (`plugin-catalog.json`)：15 个官方插件清单
  - [x] 插件锁文件 (`~/.openstarry/plugins/lock.json`)：追踪已安装插件
  - [x] 安装器：workspace 优先解析 + npm 后备
  - [x] 短名称支持 (e.g. `standard-function-fs` → `@openstarry-plugin/standard-function-fs`)
  - [x] 批量安装 (`plugin install --all`)
- [x] **5 个新 CLI 命令**: plugin install/uninstall/list/search/info
- [x] **测试涵盖**: 198 个新测试 (77 marketplace + 基础设施)

> **里程碑：v0.20.0-beta**
> *   ✅ Plugin Marketplace MVP 完成
> *   ✅ 1330 tests, snapshot saved

### Hotfix: Windows 跨平台修复 + Attach UX (v0.20.1-beta) ✅
**状态：** 已完成 (Cycle 25 hotfix + Cycle 26, 2026-02-13~14) — 1339 tests

- [x] **Windows 跨平台修复** (Cycle 25 hotfix)
  - [x] 路径处理：`path.sep`、`basename()`、`pathToFileURL()` 取代硬编码 Unix 路径
  - [x] Daemon IPC：`platform.ts` — Windows named pipe / Linux Unix socket
  - [x] 平台守卫：SIGHUP、chmod、mkdirSync、unlinkSync 在 Windows 上跳过
  - [x] 插件安装器：`cp()` dereference 后备 + `rm()` maxRetries
- [x] **Attach UX 改善** (Cycle 26)
  - [x] Attach 连接后自动显示 provider 登录状态 (`/provider status`)
  - [x] 欢迎消息加 `/help` 提示
- [x] **Token 持久化修复** (Cycle 26)
  - [x] `provider-gemini-oauth` 的 `dispose()` 不再误删 token 文件
  - [x] 新增 `cleanup()` 方法，仅清理 callback server 资源

> **里程碑：v0.20.1-beta**
> *   ✅ 全平台（Windows/Linux）稳定运行
> *   ✅ Attach 自动显示 provider 状态
> *   ✅ OAuth token 跨重启持久化
> *   ✅ 1339 tests, 117 test files, snapshot saved

---

### Plan09: Interactive Designer (待实现) ⬜
- [ ] **8.2 交互式设计器 (Interactive Designer)**
    - [ ] 实现 `openstarry design` 的上下文感知选单。
    - [ ] 建立「五蕴配置」的引导式 Wizard (Inquirer.js)。

---

## 计划依赖关系图

```
                    ┌──────────────────────────────────────────────────┐
                    │           已完成 (v0.20.1-beta)                   │
                    │  Plan01 → Plan02 → Plan03 → Plan04 → Plan05    │
                    │  → Plan06-P1/P2/P3/P4 (MCP Tools/Prompts/Res)  │
                    │  → Plan07-07.3 (Sandbox Hardening)             │
                    │  → Cycle1-8: Sessions, MCP, Sandbox Hardening  │
                    │  → Cycle9: Plan08 TUI Dashboard                │
                    │  → Cycle10-11: Plan09-10 Interactive TUI + CLI │
                    │  → Cycle12: Plan11 DevTools + E2E Framework    │
                    │  → Cycle13: Plan12 Daemon Mode MVP             │
                    │  → Cycle14: Plan13 Seamless Attach             │
                    │  → Cycle15: Plan06-P3 Resources + OAuth 2.1    │
                    │  → Cycle16: Plan14 Multi-client Attach & Mgmt  │
                    │  → Cycle17: Plan06-P4 Sampling & Extensions    │
                    │  → Cycle18: Plan15 SDK Context Extensions      │
                    │  → Cycle19: Plan16 Security Hardening          │
                    │  → Cycle20: Plan17 Plugin Developer Experience │
                    │  → Cycle21: Plan18 Plugin Sync                 │
                    │  → Cycle22: Plan19 Dependency Wiring           │
                    │  → Cycle23: Plan20 Workflow Engine MVP          │
                    │  → Cycle24: Plan21 Web Remote Attach           │
                    │  → Cycle25: Plan22 Plugin Marketplace          │
                    │  → Cycle26: Hotfix (Windows + Attach UX)       │
                    └─────────────────────┬───────────────────────────┘
                                          │
                                          ▼
                                    ┌──────────┐
                                    │ Plan09   │
                                    │ Designer │
                                    │ (待排程)  │
                                    └──────────┘
```

---

## 待讨论议题 (Issues to Discuss)

以下议题尚未有明确决议，需要在开发过程中讨论：

| 议题 | 状态 | 相关计划 |
|------|------|----------|
| WebSocket 认证机制 | 待讨论 | Plan05.1 |
| 多实例负载均衡 | 待讨论 | Plan07 |
| Session 过期策略 | 待讨论 | Plan05.1 |
| SSE 断线重连 | 待讨论 | Plan05.2 |

### 详细说明

1. **WebSocket 认证机制**
   - 是否需要 Token 认证？
   - 连接时验证 vs 每次消息验证？
   - 与 Session 隔离的关系？

2. **多实例负载均衡**
   - 多个 Agent 实例如何共享 WebSocket 连接？
   - 是否需要 Redis PubSub？
   - Sticky Session 策略？

3. **Session 过期策略**
   - 多久没活动后自动清理？
   - 是否需要心跳机制？
   - 过期时如何通知客户端？

4. **SSE 断线重连**
   - 客户端断线后如何恢复事件流？
   - 是否需要 Last-Event-ID？
   - 重连时如何补发遗漏事件？
