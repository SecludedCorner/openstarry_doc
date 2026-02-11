# 00. 项目路线图与里程碑 (Project Roadmap and Milestones)

本文档定义了 OpenStarry 从核心原型到完整生态系统的实现路径。此规划旨在确保项目演进的有序性，并为开源贡献者提供清晰的指引。

---

## 🏷️ 版本里程碑总览 (Version Milestones)

| 版本 | 内容 | 状态 |
|------|------|------|
| v0.1.0-alpha | MVP 基础架构 (Plan01-04) | ✅ 完成 |
| v0.2.0-beta | 多通道传输 — WebSocket + HTTP (Plan05) | ✅ 完成 |
| v0.2.1-beta | Session 隔离 + SSE + TraceId (Plan05.1, 05.2, 05.5) | 📋 规划中 |
| v0.2.2-beta | E2E 测试脚手架 (Plan05.4) | 📋 规划中 |
| v0.3.0-beta | MCP 协议整合 (Plan06 + DevTools) | 📋 规划中 |
| v0.3.5-beta | 管理层基础设施 (Management Zone Infra) | 📋 规划中 |
| v0.4.0-beta | 持久化与状态恢复 (Plan07) | 📋 规划中 |
| v0.5.0-beta | TUI Dashboard + 插件协调层 (Plan08-09) | 📋 规划中 |

---

## 📅 阶段一：创世纪 (Phase 1: Genesis)
**目标：** 建立 Monorepo 物理基础与共用规范。
**状态：** 🟡 进行中 (In Progress) — 1.3 CI/CD 尚未建立

- [x] **1.1 Monorepo 初始化**
    - [x] 建立 `apps/` 与 `packages/` 目录结构。
    - [x] 配置 `pnpm-workspace.yaml` 与根目录 `package.json`。
    - [x] 统一 TypeScript、ESLint 与 Prettier 规范。
- [x] **1.2 核心 SDK 与类型定义**
    - [x] 建立 `packages/sdk` 定义完整的**五蕴插件接口**。
        *   **色 (IUI)**: 界面与呈现 (Form/UI)。
        *   **受 (IListener)**: 感官监听 (Perception/Input)。
        *   **想 (IProvider)**: 思考生成 (Cognition/LLM)。
        *   **行 (ITool)**: 工具执行 (Action/Will)。
        *   **识 (IGuide)**: 灵魂与人设 (Consciousness/Persona)。
    - [x] 建立 `packages/shared` 提供全局错误处理与日志模块。
- [ ] **1.3 工程化与测试基建 (Engineering & Testing Infrastructure)**
    - [x] 配置单元测试框架 (Vitest)。 *(Plan02 Phase C — 56 项测试通过)*
    - [x] 纯净性检查脚本 (`pnpm test:purity`)。 *(Plan03 Phase A4)*
    - [ ] 建立 CI/CD 流程 (GitHub Actions)。

## 🧠 阶段二：意识内核 (Phase 2: The Conscious Kernel)
**目标：** 实现「无头 (Headless)」、事件驱动的 `Agent Core`。
**状态：** 🟢 已完成 (Done)

- [x] **2.1 执行循环 (The Loop)**
    - [x] 实作 `packages/core/execution` (基于事件队列的 tick 机制)。
    - [x] ExecutionLoop 事件驱动重构：从 EventQueue pull 事件，状态机含 WAITING_FOR_EVENT / SAFETY_LOCKOUT。 *(Plan02 Phase A)*
    - [x] InputEvent payload 标准化（source, inputType, data, replyTo）。 *(Plan02 Phase A4)*
- [x] **2.2 状态与记忆管理**
    - [x] 实作 `packages/core/state` (支持 Snapshot 与持久化接口)。
    - [x] 实作 `packages/core/memory` (可插拔的上下文策略，如滑动窗口)。
- [x] **2.3 安全断路器与错误反馈 (Safety & Correction)**
    - [x] 实作 Token 消耗上限与无限循环侦测 (Circuit Breakers)。 *(Plan02 Phase B — SafetyMonitor)*
    - [x] 实作「错误即痛觉」机制，将运行时错误转化为 Context 反馈，触发 Agent 自我修正。 *(Plan02 Phase B — 挫折计数器 + 重复调用侦测 + 错误级联熔断)*

## 🦾 阶段三：肢体与感官 (Phase 3: Body and Senses)
**目标：** 构建完整的五蕴插件基础设施与标准库，建立安全边界。
**状态：** 🟡 进行中 (In Progress) — 核心插件已完成，进阶功能待实作

- [ ] **3.1 核心加载协议 (Loading Protocol)**
    - [x] 实作 `PluginLoader`，支持**工厂模式 (Factory Pattern)** 初始化。 *(Plan01)*
    - [ ] 实作 `IPluginContext`：包含 Logger, Config, 以及**跨插件服务注入 (dependencies 字段)**。
    - [ ] **实作回路编织逻辑 (Dependency Wiring):** 根据 `20` 号文件，在加载时自动对接 OODA 回路。
    - [x] **实作指令注册表 (CommandRegistry):** 支援 CLI `run-tool` 与 Chat `/slash` 指令的映射与执行逻辑。 *(Plan01)*
    - [x] **实作动态插件载入 (Dynamic Loading):** CLI 支援 `agent.json` 的 `plugins[].path` 字段，透过 `import()` 动态载入第三方插件。 *(Plan02 Phase A3)*
    - [x] 实作 `agent.json` 的运行时验证器（Zod schema 验证）。 *(v0.1.5 - Plan03 补强完成)*
- [ ] **3.2 协调层与注册机制 (Coordination Layer)**
    - [x] **双轨扫描机制 (Dual-Path Scanning):** 同时扫描系统与项目私有插件目录。
    - [ ] **一键同步 (Sync):** 实作 `openstarry plugin sync` 指令，将官方 **`openstarry_plugin`** 仓库内容同步至系统目录。
    - [x] 在 Daemon 中实作 **Plugin Registry Service**，建立内存索引。
- [ ] **3.2 标准功能聚合插件库 (Standard Aggregate Plugins)**
    > **开发位置：** `openstarry_plugin` 仓库 (Ecosystem Repo)
    - [x] `@openstarry-plugin/standard-function-stdio`: 聚合 受(Stdio Listener) + 色(CLI Body) + 识(Default Guide)。 *(Plan01)*
    - [x] `@openstarry-plugin/provider-gemini-oauth`: 聚合 想(Gemini Provider)。PKCE + OAuth 2.0 认证。 *(Plan01)*
    - [x] `@openstarry-plugin/standard-function-fs`: 聚合 行(FS Tools) + 识(Path Scoping Policy)。 *(Plan01)*
    - [ ] `@openstarry-plugin/guide-mcp`: 聚合 识(MCP Guide)。赋予标准化通讯能力。
    - [ ] `@openstarry-plugin/guide-pain-mechanism`: 实作拟人化痛觉诠释逻辑。
    - [x] `@openstarry-plugin/standard-function-skill`: 聚合 识(Skill Guide)。**核心依赖**：负责解析 Markdown 技能文件，是工作流与复杂 Agent 的基础。 *(Plan03 Phase B1)*
- [ ] **3.4 插件开发体验 (DX)**
    - [ ] 实作 `openstarry create-plugin` 脚手架。
    - [ ] 提供 `MockHost` 测试环境，支持插件的独立开发与验证。

> **🔒 安全备注：**
> 阶段三的 `fs` 工具虽然不运行在 Docker 中，但必须通过 `packages/shared` 的路径规范化组件，严格限制其操作范围。这允许 Agent 执行「增删文件夹」等必要操作，同时保护系统关键目录。

## 👶 阶段四：降生 (Phase 4: The First Breath)
**目标：** 实现第一个单机版 CLI Agent，验证端到端流程。
**状态：** 🟡 进行中 (In Progress) — Runner 可运行，端到端 LLM 验证待 OAuth 登录后测试

- [x] **4.1 本地执行器 (Local Runner)**
    - [x] 实作 `apps/runner` 引导程序（纯启动 Bootstrap Runner），支持 `agent.json` 启动。 *(Plan01)*
    - [x] 支持动态插件解析（path / node_modules 双层策略，BUILTIN_FACTORIES 已移除，所有插件皆动态载入）。 *(Plan02 Phase A3)*
- [ ] **4.2 整合验证**
    - [ ] 达成「感知 -> 思考 -> 工具调用 -> 修正 -> 回应」的完整闭环。*(需 OAuth 登录后手动测试)*

> **🏆 里程碑：v0.1 Alpha (MVP)**
> *   ✅ 能在 CLI 中运行单一 Agent。
> *   ✅ 使用 Gemini 作为 Provider（PKCE + OAuth 认证）。
> *   ✅ 能操作本地文件系统 (`fs` tool)。
> *   ✅ 具备基本记忆能力 (5 轮对话)。
> *   ✅ 安全熔断机制（Token 预算、循环上限、错误级联、挫折计数器）。
> *   ✅ 结构化 JSON 日志（LOG_FORMAT=json, LOG_LEVEL 过滤）。
> *   ✅ agent.json Zod 运行时验证（配置错误即时报错）。
> *   ✅ 工具调用 Timeout（Promise.race 防卡死）。
> *   ✅ TraceID 机制（日志追踪一次完整处理周期）。
> *   ✅ Markdown 技能载入（standard-function-skill 插件）。
> *   ✅ guideFile 外部文件引用（system_prompt 从 .md 载入）。
> *   ✅ 纯净性检查脚本（pnpm test:purity）。
> *   ✅ 56 项单元测试通过（Vitest）。
> *   ⬜ 端到端 LLM 通话验证（待手动测试）。

---

> **🚀 以上完成 Agent 核心。以下进入「代理人操作系统」与生态阶段。**

---

## 📡 阶段 4.5：多通道传输 (Phase 4.5: Multi-Channel Transport)
**目标：** 验证 IUI/IListener 分离架构的可扩展性，实作非 stdio 的通道。
**状态：** 🟢 已完成 (v0.2.0-beta)

- [x] **4.5.1 WebSocket 传输插件**
    - [x] `@openstarry-plugin/transport-websocket`: WebSocket Listener + UI
    - [x] 支援多客户端连线、定向回复 (replyTo)
- [x] **4.5.2 HTTP Webhook 传输插件**
    - [x] `@openstarry-plugin/transport-http`: HTTP Listener + UI
    - [x] POST /api/input, GET /api/status, GET /api/response
- [x] **4.5.3 多 UI 同时输出**
    - [x] TransportBridge 广播机制验证通过
    - [x] stdio + WebSocket 同时收到事件

> **🏆 里程碑：v0.2.0 Beta (Multi-Channel)**
> *   ✅ WebSocket 传输插件实作完成
> *   ✅ HTTP Webhook 传输插件实作完成
> *   ✅ 多 UI 同时输出验证通过

---

## 🔐 阶段 4.6：实作周期一 (v0.2.1-beta)
**目标：** 解决多用户隐私问题，优化传输效率，建立全链路追踪。
**状态：** 📋 规划中

### Plan05.1: Session 隔离与消息路由 🔴 最高优先级
- [ ] Listener 标记 `sessionId`
- [ ] Core 透传 `sessionId` 到输出事件
- [ ] UI 依据 `sessionId` 过滤推送
- [ ] 验收：WebSocket 用户 A 看不到用户 B 的对话

### Plan05.2: HTTP SSE 支援 🟢 高优先级
- [ ] 新增 `GET /api/stream` 端点
- [ ] 使用 Server-Sent Events 取代轮询
- [ ] 支援 EventSource 客户端

### Plan05.5: 统一 TraceId 🔵 低优先级 (提前实作)
- [ ] Core 提供统一 `TraceId` 产生器
- [ ] 所有事件与日志自动附加

## 🧪 阶段 4.7：实作周期二 (v0.2.2-beta)
**目标：** 建立自动化验证能力与测试脚手架。
**状态：** 📋 规划中

### Plan05.4: E2E 测试脚手架 🟡 中优先级
- [ ] `createTestAgent()` 工具函数
- [ ] 支援 MockLLM 插件
- [ ] 降低社区贡献门槛

> **⚠️ 注意：Plan05.3 DevTools 延后至 Plan06 (MCP) 之后实作。**

---

## 🏰 阶段五：守护进程 (Phase 5: The Orchestrator Daemon) — Plan07
**目标：** 实现进程级管理与持久化。
**状态：** 📋 规划中 (v0.4.0-beta)

- [ ] **5.1 守护进程核心 (Daemon)**
    - [ ] 建立 `apps/daemon`，管理多个 Agent 实例的生命周期。
    - [ ] 实作 **Agent 协调管理层 (Management Zone)** 中的调度逻辑。
- [ ] **5.2 API 网关与持久化**
    - [ ] 提供 HTTP/gRPC API 进行远程管理。
    - [ ] 整合数据库（SQLite/LevelDB）存储 Agent 长期状态。
- [ ] **5.3 安全与治理 (Security & Governance)**
    - [ ] 实作插件签名与验证机制 (确保载入的插件是受信任的)。
    - [ ] 实作运行时沙箱 (Sandbox) 策略 (如 Node.js vm 或 WASM 容器化)。
    - [ ] **实作环境隔离与资源配额 (Quota Management)**。
- [ ] **5.4 系统可观测性与硬件抽象 (Observability & HAL)**
    - [x] 结构化 JSON 日志，支援 agent_id / trace_id / module 字段。 *(Plan02 Phase D)*
    - [ ] 整合 OpenTelemetry 或 Event Tracing，追踪跨 Agent 的任务流动。
    - [ ] 实作日志收集，支援 Dashboard 的视觉化呈现。
    - [ ] **实作硬件抽象层 (HAL) 标准感官数据流**。

## 🕸️ 阶段六：分形社会 (Phase 6: Fractal Society) — Plan06
**目标：** 实现代理人协作与 MCP 协议。
**状态：** 📋 规划中 (v0.3.0-beta)
**前置条件：** Plan05.1 Session 隔离完成 ✓

> **进入检查清单：**
> - ☐ Plan05.1 Session 隔离已实作
> - ☐ 多客户端场景验证通过
> - ☐ 消息不会跨 Session 泄漏

- [ ] **6.1 MCP 深度整合 (Plan06)**
    - [ ] 实作 `packages/mcp-protocol`，让 Agent 具备标准化互联能力。
    - [ ] 支援多外部客户端同时连接
- [ ] **6.2 DevTools 调试界面 (Plan05.3)**
    - [ ] 静态 HTML 调试界面 `openstarry-devtools`
    - [ ] 左侧：对话气泡视图
    - [ ] 右侧：State/Context 检视
    - [ ] 底部：事件过滤器
- [ ] **6.3 工作流引擎 (Workflow Engine)**
    - [ ] 实作 `WorkflowEngineTool`，支持 YAML 定义的任务编排。
    - [ ] 实作 Agent 动态孵化与销毁机制 (Ephemeral Agents)。
- [ ] **6.3 跨 Agent 协作与因果调度 (Orchestration)**
    - [ ] 实现任务拆解与分发至不同子代理人的机制。
    - [ ] **实作基于因果链的事件调度 (Causality Chain)**。

## 📦 阶段七：视觉化与生态 (Phase 7: Ecosystem and UI)
**目标：** 提供 Web 图形化管理与部署工具。
**状态：** 🔴 待启动

- [ ] **7.1 Dashboard 与多元交互层 (Interface)**
    - [ ] 建立 `apps/dashboard` (React/Next.js)，可视化监控所有 Agent。
    - [ ] **实作状态投影 (State Projection) 仪表板**。
- [ ] **7.2 模板服务 (Agent Design Service)**
    - [ ] 提供 Agent 商店/模板下载功能，实现「目录即协议」的极简部署。

## 🖥️ 阶段八：终端机演进 (Phase 8: CLI & TUI Evolution) — Plan08 / Plan09
**目标：** 实现「操作系统级」的终端体验，达成 User Scenario Guide 的愿景。
**状态：** 📋 规划中 (v0.5.0-beta)

- [ ] **8.1 运行时仪表板 (Runtime Dashboard)**
    - [ ] 实作 `openstarry` 的 TUI 界面 (使用 Ink 或 Blessed)。
    - [ ] 整合 Daemon 状态监控、资源图表与即时日志串流。
- [ ] **8.2 交互式设计器 (Interactive Designer)**
    - [ ] 实作 `openstarry design` 的上下文感知菜单。
    - [ ] 建立「五蕴配置」的引导式 Wizard (Inquirer.js)。
- [ ] **8.3 无缝连线机制 (Seamless Attach)**
    - [ ] 实作 `openstarry attach` 的 Socket 连线逻辑。
    - [ ] 支援从 TUI 仪表板直接跳转至 Agent 对话视窗 (按 'a' 键)。

---

## 📊 计划依赖关系图

```
                    ┌─────────────────────────────────────────┐
                    │           已完成 (v0.2.0-beta)           │
                    │  Plan01 → Plan02 → Plan03 → Plan04 → Plan05  │
                    └─────────────────────┬───────────────────┘
                                          │
              ┌───────────────────────────┼────────────────────────────────┐
              │                           │                                │
              ▼                           ▼                                ▼
        ┌──────────┐              ┌──────────────┐                  ┌──────────┐
        │实作周期一 │              │实作周期二     │                  │ Plan06   │
        │v0.2.1-beta│              │v0.2.2-beta   │                  │ MCP 整合  │
        │(Sess+SSE+ │              │(E2E 测试)     │                  │ (+DevTools)│
        │ TraceId)  │              └──────────────┘                  └────┬─────┘
        └────┬─────┘                                                      │
             │                                                            │
             │ (前置条件)                                                   │
             ▼                                                            ▼
        ┌──────────┐                                                ┌──────────┐
        │ Plan06   │────────────────────────────────────────────────▶│ Plan07   │
        │ MCP 整合 │                                                │ 持久化    │
        └──────────┘                                                └──────────┘
```

---

## 🟡 待讨论议题 (Issues to Discuss)

以下议题尚未有明确决议，需要在开发过程中讨论：

| 议题 | 状态 | 相关计划 |
|------|------|----------|
| WebSocket 认证机制 | 🟡 待讨论 | Plan05.1 |
| 多实例负载均衡 | 🟡 待讨论 | Plan07 |
| Session 过期策略 | 🟡 待讨论 | Plan05.1 |
| SSE 断线重连 | 🟡 待讨论 | Plan05.2 |

### 详细说明

1. **WebSocket 认证机制**
   - 是否需要 Token 认证？
   - 连线时验证 vs 每次消息验证？
   - 与 Session 隔离的关系？

2. **多实例负载均衡**
   - 多个 Agent 实例如何共享 WebSocket 连线？
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
