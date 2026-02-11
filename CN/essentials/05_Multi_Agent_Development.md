# OpenStarry 如何构建：多代理开发

> OpenStarry 不仅仅是一个 AI 代理框架。它是一个**由 AI 代理构建的** AI 代理框架。

## 6 代理团队

OpenStarry 由一个 6 个专业 AI 代理组成的团队开发，遵循正式的标准操作流程 (SOP)。每个代理都有明确的角色、专用的工作空间和结构化的输出要求。这不是一个玩具演示——而是一个真正的软件工程流程，产出经过架构评审、质量验证和文档化的代码。

### 团队

| 代理 | 角色 | 模型 | 工作空间 | 输出 |
|-----|------|------|---------|------|
| **Architect** | 架构守护者 | Sonnet | 文档 + 代码（只读） | 设计规范、代码评审、安全审计 |
| **Dev-Core** | 核心开发者 | Sonnet | `agent_dev/openstarry/` | SDK、Core、Shared、Runner 实现 |
| **Dev-Plugin** | 插件开发者 | Sonnet | `agent_dev/openstarry_plugin/` | 传输层、Provider、Tool、Guide 插件 |
| **QA** | 质量保证 | Sonnet | `agent_test/`（隔离副本） | 构建验证、测试报告、纯净度检查 |
| **Doc-Keeper** | 文档管理员 | Haiku | `share/openstarry_doc/` | 决策记录、迭代日志、文档更新 |
| **Researcher** | 技术研究员 | Sonnet | `share/ref/` | 预研报告、参考项目分析 |

一位人类**协调员**使用 Claude Opus 编排团队，做出最终决策，解决分歧，并管理周期转换。

### 为什么采用这种结构？

每个角色的存在都有其原因：

- **Architect** 防止架构腐蚀。没有它，随着捷径的积累，五蕴纯净度会逐渐退化。Architect 在每次提交时强制执行合规检查。
- **分离的 Dev-Core 和 Dev-Plugin** 代理可以并行工作，因为 SDK 接口是冻结的。Dev-Core 构建框架，而 Dev-Plugin 在框架之上构建——同时进行。
- **QA 在隔离副本**（`agent_test/`）中工作，而不是开发目录。这防止了经典的「在我的机器上能用」问题——如果在测试环境中通过了，它就是真正通过了。
- **Doc-Keeper 使用更轻量的模型**（Haiku），因为文档更新不需要代码生成的推理能力，但需要一致和全面。
- **Researcher** 在实施开始之前进行预研，研究参考项目（OpenClaw 用于代理路由、OpenCode 用于 TUI 模式、OpenOctopus 用于 MCP 集成）以辅助设计决策。

## 迭代周期

每个功能都经过一个包含质量关卡的规范化 8 阶段周期：

### 阶段 0：规划
```
Coordinator: "Cycle 1 scope: Session Isolation + HTTP SSE + Health Check"
Researcher: Studies transport patterns, SSE best practices, session management strategies
Doc-Keeper: Records the plan in Iteration_Log.md with cycle ID (20260210_cycle1)
```

### 阶段 1：设计
```
Architect: Produces Architecture_Spec_Cycle1.md with:
  - New/modified interfaces (ISession, ISessionManager, InputEvent.sessionId)
  - Sequence diagrams for session lifecycle
  - Security considerations (session isolation prevents cross-talk)

⚠️ INTERFACE FREEZE: Once published, interfaces cannot change without a formal Spec Addendum
```

### 阶段 1.5：基线
```
Coordinator: runs scripts/baseline.sh → saves pre-implementation snapshot
  └─ Safety net: if anything goes wrong, restore to this point
```

### 阶段 2：实施
```
Phase 2a: Dev-Core implements SDK interface changes first
  └─ ISession, ISessionManager added to @openstarry/sdk
  └─ InputEvent gains optional sessionId field (backward compatible)
  └─ pnpm build must pass before Phase 2b begins

Phase 2b: Dev-Core + Dev-Plugin work in parallel
  └─ Dev-Core: Session manager, execution loop session routing
  └─ Dev-Plugin: WebSocket session binding, HTTP SSE endpoints
  └─ Each produces a structured DevLog with decisions, challenges, solutions
```

### 阶段 2.5：同步
```
Coordinator: runs scripts/sync-to-test.sh
  └─ Atomic copy from agent_dev/ → agent_test/
  └─ Ensures test environment has exact copy of development code
  └─ pnpm build in test environment must pass
```

### 阶段 3：验证（并行）
```
QA (in agent_test/):                    Architect (reading agent_dev/):
├─ pnpm build → all 11 packages pass   ├─ Five Aggregates compliance check
├─ pnpm test → 118+ tests pass         ├─ Microkernel purity review
├─ pnpm test:purity → zero violations  ├─ Security audit
└─ Produces QA_Report_Cycle1.md        └─ Produces CodeReview_Cycle1.md
```

### 阶段 4：收敛
```
PASS: Coordinator runs scripts/snapshot.sh → versioned snapshot saved
FAIL: Issues classified:
  ├─ Code Fix → back to Phase 2 (bug in implementation)
  ├─ Design Fix → back to Phase 1 (architecture issue)
  └─ Plan Fix → back to Phase 0 (requirements problem)

Max 2 rework cycles before human escalation
```

## 质量关卡

### 接口冻结
一旦 Architect 发布了架构规范，接口就被冻结。Dev-Core 可以实现它们，但不能更改它们。这防止了困扰大多数项目的范围蔓延和接口不稳定。如果接口需要更改，需要一个正式的规范补充——一个经过深思熟虑的审查流程。

### 微内核纯净度验证
QA 运行 `pnpm test:purity`，扫描编译后的 Core 二进制文件中是否有任何插件导入。**零违规容忍。** 这个自动化关卡防止了随时间推移逐渐侵蚀微内核架构的污染。

### 测试隔离
QA 从不在开发目录中测试。同步脚本将代码复制到单独的 `agent_test/` 环境中。如果开发者留下了一个只在本地状态下才能通过测试的临时 hack，QA 会发现它。

### 返工分类
并非所有失败都是相同的：
- **代码修复** → 快速补丁，回到阶段 2
- **设计修复** → 需要接口变更，回到阶段 1（需要规范补充）
- **计划修复** → 需求有误，回到阶段 0（罕见但严重）

## 结构化产出物

每个周期都产生可追溯的结构化报告：

```
share/test/reports/
├── research/20260210_cycle1/
│   └── Research_Transport_Enhancement.md      ← Researcher
├── arch_reviews/20260210_cycle1/
│   ├── Architecture_Spec_Cycle1.md            ← Architect (FROZEN)
│   └── CodeReview_Cycle1.md                   ← Architect
├── dev_logs/20260210_cycle1/
│   ├── DevLog_Phase2a_SDK_Core.md             ← Dev-Core
│   └── DevLog_Phase2b_Transport_Plugins.md    ← Dev-Plugin
├── qa_results/20260210_cycle1/
│   └── QA_Report_Cycle1.md                    ← QA
└── sys_summary/20260210_cycle1/
    └── Convergence_Summary.md                 ← Coordinator
```

全部以周期 ID（`{YYYYMMDD}_cycle{N}`）命名以实现完整的可追溯性。六个月后，你可以将任何一行代码追溯到激发它的研究、定义它的规范、实现它的开发日志以及验证它的 QA 报告。

## 这对社区意味着什么

当 OpenStarry 发布时，贡献者将继承：

1. **一套完整的开发流程** — 不仅仅是代码，而是一种工作方式
2. **架构护栏** — 自动化纯净度测试防止善意的贡献侵蚀架构
3. **可追溯的决策** — 每个「为什么」都有文档记录，不会丢失在聊天历史中
4. **并行安全的开发** — Dev-Core 和 Dev-Plugin 可以是同时工作的不同贡献者，因为 SDK 接口是冻结的
5. **可重现的质量** — 无论开发者是人类还是 AI，SOP 都是相同的

> 流程是产品的一部分。
