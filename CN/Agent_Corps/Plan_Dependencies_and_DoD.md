# OpenStarry — 计划依赖与完成定义 (Plan Dependencies & DoD)

建立日期：2026-02-09

---

## 1. 计划依赖关系图 (Plan Dependencies)

```
Plan01 (MVP Alpha Foundation)           ✅ v0.1-alpha  2026-02-04
  ↓
Plan02 (Event-Driven & Safety)          ✅ v0.1.1      2026-02-05
  ↓
Plan03 (Quality & Skill Parser)         ✅ v0.2-alpha  2026-02-05
  ↓
Plan04 (IUI Interface & Guide Plugin)   ✅ v0.2-alpha  2026-02-06
  ↓
Plan05 (Multi-Channel UI & Listener)    ✅ v0.2-beta   2026-02-07
  ↓
  ├── Plan05.1 (Session Isolation)      ⬜ → v0.2.1-beta
  ├── Plan05.2 (HTTP SSE)              ⬜ → v0.2.1-beta   ← 可与 05.1 并行
  ├── Plan05.5 (其他)                   ⬜ → v0.2.1-beta   ← 可与 05.1 并行
  │     ↓
  │   Plan06 (MCP Protocol)             ⬜ → v0.3       ← 被 Plan05.1 阻塞
  │     ↓
  │   Plan07 (Runtime Sandbox)          ⬜ → v0.4       ← 被 Plan06 阻塞
  │     ↓
  │   Plan08-09 (TUI Dashboard)         ⬜ → v0.5
  │
  └── (Extended Mode: 新插件、Web UI、安全审计)
```

### 关键路径 (Critical Path)

```
Plan05.1 → Plan06 → Plan07 → Plan08-09
```

Plan05.1 (Session Isolation) 是关键路径上的第一个未完成项目。Plan06 (MCP) 被其阻塞。

### 可并行的工作

| 可并行 | 说明 |
|--------|------|
| Plan05.1 + Plan05.2 + Plan05.5 | 三者可合并为一轮迭代 (Implementation Cycle 1) |
| Plan06 的预研 + Plan05.x 的实作 | researcher 可在开发代理人实现 05.x 时同步预研 Plan06 |

---

## 2. 完成定义 (Definition of Done)

### 2.1 通用 DoD (适用于所有计划)

每个 Plan 必须满足以下**全部条件**才能标记 ✅：

| # | 条件 | 验证方式 |
|---|------|---------|
| 1 | `pnpm build` 通过 (monorepo + plugins) | qa 在 agent_test 验证 |
| 2 | `pnpm test` 全部通过，且测试数 ≥ 前一轮基线 | qa 报告中的回归检查 (regression check) |
| 3 | `pnpm test:purity` 通过 | qa 报告中的纯净度检查 (purity check) |
| 4 | Architecture_Spec 中所有项目已实现 | architect 代码审查 (Code Review) PASS |
| 5 | 五蕴合规（新组件正确归类） | architect 代码审查确认 |
| 6 | pushInput 模式合规 | architect 代码审查确认 |
| 7 | 无已知安全漏洞 | architect 代码审查确认 |
| 8 | 开发日志 (dev log) 已写入 | Coordinator 确认文件存在 |
| 9 | QA 报告和代码审查均为 PASS | Phase 4 判定 |
| 10 | doc-keeper 已更新计划标记 ✅ + 迭代日志 | Phase 4 收敛 |
| 11 | 快照 (Snapshot) 已建立 | `scripts/snapshot.sh` 完成 |
| 12 | 经验教训 (Lessons Learned) 已记录 | Phase 4 回顾 |

### 2.2 各计划专项验收条件

#### Plan05.1: Session Isolation
- [ ] WebSocket 连接支持 session token / auth
- [ ] 多个 client 连接互相隔离（会话数据不交叉）
- [ ] 单个 agent 实例可服务多个 session
- [ ] 会话生命周期管理（建立、维持、销毁）
- [ ] 新增测试覆盖会话隔离场景

#### Plan05.2: HTTP SSE
- [ ] HTTP Server-Sent Events 传输插件可用
- [ ] SSE 连接可接收实时流式响应
- [ ] 与现有 HTTP webhook listener 共存且不冲突
- [ ] 新增测试覆盖 SSE 场景

#### Plan06: MCP Protocol (Model Context Protocol)
- [ ] 实现 MCP 规格中的 tool、resource、prompt 三大能力
- [ ] MCP server 模式：OpenStarry 作为 MCP server 被外部客户端连接
- [ ] MCP client 模式：OpenStarry 连接外部 MCP server 获取工具
- [ ] 符合 MCP 官方规格 (researcher 预研确认)
- [ ] 新增测试覆盖 MCP 场景

#### Plan07: Runtime Sandbox
- [ ] 插件执行环境隔离 (vm 或 WASM)
- [ ] 资源限制（CPU、内存、文件访问）
- [ ] 插件签名验证机制
- [ ] 新增测试覆盖沙盒逃逸 (sandbox escape) 场景

#### Plan08-09: TUI Dashboard
- [ ] 终端交互式仪表板 (Ink 或 Blessed)
- [ ] 实时显示 Agent 状态、对话、工具调用
- [ ] 支持多会话切换
- [ ] 新增测试覆盖 TUI 场景

---

## 3. 迭代排期建议

| 迭代 | 涵盖计划 | 目标版本 | 前置条件 |
|------|-----------|---------|---------|
| Cycle 1 | Plan05.1 + 05.2 + 05.5 | v0.2.1-beta | Plan05 ✅（已满足） |
| Cycle 2 | Plan06 (MCP) | v0.3 | Plan05.1 ✅ |
| Cycle 3 | Plan07 (Sandbox) | v0.4 | Plan06 ✅ |
| Cycle 4 | Plan08-09 (TUI) | v0.5 | Plan07 ✅（或可提前） |
| Extended | 新功能、安全审计 | v1.0 | 所有计划 ✅ |

---

## 4. 使用方式

- **Coordinator** 在开始新迭代前，查阅本文档确认：
  1. 前置计划是否已完成 ✅
  2. 该计划的专项验收条件
  3. 是否有可并行的工作
- **architect** 在设计规范 (Spec) 时，参考专项验收条件以确保覆盖
- **qa** 验证时，逐项检查通用 DoD + 专项条件
- **doc-keeper** 在计划标记 ✅ 时，确认 DoD 全部满足
