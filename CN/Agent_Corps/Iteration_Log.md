# OpenStarry Agent Corps — 迭代日志 (Iteration Log)

此文件记录每个迭代周期的决策与结果。

---

## 2026-02-09: Agent Corps 成立

- **行动**: 成立了由 6 个代理人组成的开发军团
- **代理人**: architect, dev-core, dev-plugin, qa, doc-keeper, researcher
- **基础设施**: 创建了目录结构、代理人定义、项目 CLAUDE.md、设置
- **状态**: 设置完成，准备进行第一个迭代周期
- **后续**: 开始实施周期 1 (Plan05.1 + 05.2 + 05.5 → v0.2.1-beta)

---

## 20260210_cycle1: 实施周期 1

- **日期**: 2026-02-10
- **目标计划**: Plan05.1 (Session Isolation) + Plan05.2 (HTTP SSE) + Plan05.5-① (Health Check)
- **目标版本**: v0.2.1-beta
- **范围**:
  - Plan05.1: WebSocket 多客户端会话隔离（会话令牌、生命周期、数据隔离）
  - Plan05.2: HTTP Server-Sent Events 传输（实时流式传输，替代轮询）
  - Plan05.5-①: 连接健康检查（心跳、重连、超时处理）
- **策略**: 双周期方法
  - 周期 1 (当前): 05.1 + 05.2 + 05.5-① — 建立 SOP 工作流，交付关键路径项
  - 周期 2 (后续): 05.5-② (指标/日志) + 05.5-③ (错误处理) — 可与 Plan06 预研并行
- **前提条件**: Plan05 ✅ (v0.2-beta, 2026-02-07)

### Phase 1 — Design

- **架构规范 (Architecture Spec)**: `share/test/reports/arch_reviews/20260210_cycle1/Architecture_Spec_Cycle1.md`
- **接口冻结**: 是 — 第 2 节的所有接口在发布时冻结
- **关键冻结接口**:
  - `ISession` (新增) — 包含 id, createdAt, updatedAt, metadata 的会话实体
  - `ISessionManager` (新增) — 会话生命周期：create/get/list/destroy/getStateManager/getDefaultSession
  - `InputEvent.sessionId` (添加) — 可选字段；省略则回退到默认会话
  - `IPluginContext.sessions` (添加) — 向插件暴露 ISessionManager
  - `AgentEventType.SESSION_CREATED / SESSION_DESTROYED` (添加) — 生命周期事件
  - `SSEConnection` (新增，内部) — HTTP 传输 SSE 连接跟踪
  - `HealthCheckConfig` (新增，按插件) — 用于心跳/过期阈值的共享配置结构
  - `ClientConnection.alive / sessionId` (添加) — WebSocket 健康与会话绑定
- **关键设计决策**:
  1. SessionManager 是 **核心基础设施** (类似 StateManager)，而非插件 — 保持微内核纯度；会话隔离是框架关注点，而非功能特性
  2. **默认会话 (`__default__`)** 在构造时创建 — 保持向后兼容；无 sessionId 的输入像以前一样使用共享状态
  3. `getStateManager(sessionId?)` 是会话与 ExecutionLoop 之间的桥梁 — 循环在 processEvent() 开始时解析每个会话的状态，无需重构
  4. TransportBridge 保持 **会话无关** — 智能逻辑存在于插件 UI 实现中，Bridge 是哑基础设施（微内核原则）
  5. SSE 交付通过监听器中 **按连接的总线订阅** 处理，而非通过 HTTP UI — 分离轮询与流式传输关注点
  6. WebSocket 协议 ping/pong 与现有的应用级 ping/pong **共存** — 目的不同（服务器发起的健康检查 vs 客户端发起的存活检查）
  7. `IPluginContext.sessions` 暴露完整的 `ISessionManager` 而非单个方法 — 为传输插件提供单一且类型良好的接口
  8. 默认会话 **不可销毁** — 保护向后兼容性
  9. 事件负载富含 `sessionId` + `replyTo` — 在不改变事件接口的情况下实现会话感知路由
- **实施顺序**:
  - **Phase 2a** (仅 dev-core): SDK 接口 → Core SessionManager → ExecutionLoop 变更 → AgentCore 变更 → 测试更新 (~14 个新测试)
  - **Phase 2b** (dev-core + dev-plugin 并行): transport-websocket 会话/ping/路由 + transport-http SSE/心跳/会话 (~12 个新测试)
  - Phase 2a 必须在 Phase 2b 之前完成（插件依赖于 SDK + Core）
- **预期测试数量**: 82 个现有测试 + ~26 个新测试 = 共计 ~108 个
- **风险评估**: 识别出 8 个风险（参见架构规范第 9 节）；最高风险：AgentCore.stateManager 移除（中等，机械性修复）和不受限会话导致的内存增长（中等，推迟到未来周期）

- **状态**: Phase 1 — 设计完成 → Phase 1.5 基线

### Phase 2 — Implementation (2026-02-10)

- Phase 2a 完成: SDK 接口 + Core SessionManager (dev-core)
  - 4 个新的 SDK 文件/变更，4 个新的 Core 文件/变更
  - 18 个新测试，总计 100 个 → 已验证
- Phase 2b 完成: 传输插件 (dev-plugin)
  - transport-websocket: 会话握手、协议 ping/pong、会话感知路由 (6 个新测试)
  - transport-http: SSE 端点、心跳、会话输入 (11 个新测试)
  - Core tsconfig 修复: 从构建中排除测试文件
  - 总计: 118 个测试全部通过
- 构建: ✅ pnpm build (11 个包)
- 测试: ✅ 118 个通过
- 纯度: ✅ 通过

### Phase 2.5 — Sync 挂起

- sync-to-test.sh 由于 Windows 符号链接问题 (node_modules symlinks) 失败
- 重试前需要修复脚本

- **状态**: Phase 2 完成，Phase 2.5 挂起（同步脚本需要 Windows 修复）
- **恢复点**: 修复同步脚本 → Phase 2.5 → Phase 3 (QA + 架构师审查) → Phase 4 (收敛)
