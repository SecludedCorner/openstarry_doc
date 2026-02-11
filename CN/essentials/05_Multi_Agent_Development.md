# OpenStarry 是如何建造的：多 Agent 开发

> OpenStarry 不仅仅是一个 AI Agent 框架。它是一个**由 AI Agent 建造的** AI Agent 框架。

## 6-Agent 团队

OpenStarry 由一个 6 名专业 AI Agent 组成的团队开发，遵循正式的标准作业程序（SOP）。每个 Agent 都有明确的角色、专属的工作区，以及结构化的产出要求。这不是一个玩具展示——而是一个真正的软件工程流程，产出的是经过架构审查、质量验证、有完整文档的代码。

### 团队成员

| Agent | 角色 | 模型 | 工作区 | 产出 |
|-------|------|------|--------|------|
| **Architect** | 架构守护者 | Sonnet | 文档 + 代码（只读） | 设计规格、代码审查、安全审计 |
| **Dev-Core** | 核心开发者 | Sonnet | `agent_dev/openstarry/` | SDK、Core、Shared、Runner 实现 |
| **Dev-Plugin** | 插件开发者 | Sonnet | `agent_dev/openstarry_plugin/` | Transport、Provider、Tool、Guide 插件 |
| **QA** | 质量保证 | Sonnet | `agent_test/`（隔离副本） | 构建验证、测试报告、纯度检查 |
| **Doc-Keeper** | 文档管理者 | Haiku | `share/openstarry_doc/` | 决策记录、迭代日志、文档更新 |
| **Researcher** | 技术研究员 | Sonnet | `share/ref/` | 前期研究报告、参考项目分析 |

一位人类**协调者**使用 Claude Opus 来指挥团队、做最终决策、解决争议，并管理周期转换。

### 为什么是这样的结构？

每个角色的存在都有其理由：

- **Architect** 防止架构侵蚀。没有它，五蕴的纯度会随着捷径的累积而逐渐退化。Architect 对每次提交都强制执行合规性检查。
- **独立的 Dev-Core 和 Dev-Plugin** Agent 可以平行工作，因为 SDK 接口已被冻结。Dev-Core 建造框架，Dev-Plugin 基于框架开发——同步进行。
- **QA 在隔离副本中工作**（`agent_test/`），不是在开发目录中。这防止了经典的「在我的机器上可以跑」问题——如果在测试环境中通过，那就是真的通过了。
- **Doc-Keeper 使用较轻量的模型**（Haiku），因为文档更新不需要代码生成的推理能力，但需要保持一致性和完整性。
- **Researcher** 在实作开始前进行前期研究，研究参考项目（OpenClaw 用于 Agent 路由、OpenCode 用于 TUI 模式、OpenOctopus 用于 MCP 整合），以此为设计决策提供依据。

## 迭代周期

每个功能都经历一个严谨的 8 阶段周期，搭配质量关卡：

### 第 0 阶段：规划
```
Coordinator: "Cycle 1 范围：Session 隔离 + HTTP SSE + 健康检查"
Researcher: 研究 Transport 模式、SSE 最佳实践、Session 管理策略
Doc-Keeper: 在 Iteration_Log.md 中记录计划，附上周期 ID（20260210_cycle1）
```

### 第 1 阶段：设计
```
Architect: 产出 Architecture_Spec_Cycle1.md，包含：
  - 新增/修改的接口（ISession、ISessionManager、InputEvent.sessionId）
  - Session 生命周期的序列图
  - 安全考量（Session 隔离防止跨 Session 干扰）

⚠️ 接口冻结：一旦发布，接口不可更改，除非通过正式的 Spec Addendum
```

### 第 1.5 阶段：基线备份
```
Coordinator: 执行 scripts/baseline.sh → 存储实作前的快照
  └─ 安全网：如果出了问题，可以复原到这个时间点
```

### 第 2 阶段：实作
```
Phase 2a: Dev-Core 先实作 SDK 接口变更
  └─ ISession、ISessionManager 加入 @openstarry/sdk
  └─ InputEvent 新增选用的 sessionId 字段（向下兼容）
  └─ pnpm build 必须通过才能进入 Phase 2b

Phase 2b: Dev-Core + Dev-Plugin 平行工作
  └─ Dev-Core: Session 管理器、执行循环的 Session 路由
  └─ Dev-Plugin: WebSocket Session 绑定、HTTP SSE 端点
  └─ 各自产出结构化的 DevLog，记录决策、挑战与解决方案
```

### 第 2.5 阶段：同步
```
Coordinator: 执行 scripts/sync-to-test.sh
  └─ 从 agent_dev/ 原子性复制至 agent_test/
  └─ 确保测试环境拥有开发代码的完整副本
  └─ 测试环境中的 pnpm build 必须通过
```

### 第 3 阶段：验证（平行进行）
```
QA（在 agent_test/ 中）：                Architect（读取 agent_dev/）：
├─ pnpm build → 全部 11 个套件通过     ├─ 五蕴合规性检查
├─ pnpm test → 118+ 测试通过           ├─ 微内核纯度审查
├─ pnpm test:purity → 零违规           ├─ 安全审计
└─ 产出 QA_Report_Cycle1.md           └─ 产出 CodeReview_Cycle1.md
```

### 第 4 阶段：收敛
```
PASS: Coordinator 执行 scripts/snapshot.sh → 存储版本化快照
FAIL: 问题分类：
  ├─ Code Fix → 回到第 2 阶段（实作中的 Bug）
  ├─ Design Fix → 回到第 1 阶段（架构问题）
  └─ Plan Fix → 回到第 0 阶段（需求问题）

最多 2 次返工循环，之后升级至人类处理
```

## 质量关卡

### 接口冻结
一旦 Architect 发布了 Architecture Spec，接口就被冻结。Dev-Core 可以实作它们，但不能更改它们。这防止了困扰大多数项目的范围蔓延和接口不稳定。如果接口需要变更，必须通过正式的 Spec Addendum——一个刻意的、经过审查的流程。

### 微内核纯度验证
QA 执行 `pnpm test:purity`，扫描编译后的 Core 二进制文件是否有任何插件导入。**不允许任何违规。**这个自动化关卡防止了随时间推移逐渐侵蚀微内核架构的污染。

### 测试隔离
QA 绝不在开发目录中测试。同步脚本将代码复制到独立的 `agent_test/` 环境。如果开发者留下了一个只在本地状态下才能让测试通过的暂时性修改，QA 会抓到它。

### 返工分类
并非所有失败都是相同的：
- **Code Fix** → 快速修补，回到第 2 阶段
- **Design Fix** → 需要接口变更，回到第 1 阶段（需要 Spec Addendum）
- **Plan Fix** → 需求有误，回到第 0 阶段（罕见但严重）

## 结构化产出物

每个周期都产出可追溯的、结构化的报告：

```
share/test/reports/
├── research/20260210_cycle1/
│   └── Research_Transport_Enhancement.md      ← Researcher
├── arch_reviews/20260210_cycle1/
│   ├── Architecture_Spec_Cycle1.md            ← Architect（已冻结）
│   └── CodeReview_Cycle1.md                   ← Architect
├── dev_logs/20260210_cycle1/
│   ├── DevLog_Phase2a_SDK_Core.md             ← Dev-Core
│   └── DevLog_Phase2b_Transport_Plugins.md    ← Dev-Plugin
├── qa_results/20260210_cycle1/
│   └── QA_Report_Cycle1.md                    ← QA
└── sys_summary/20260210_cycle1/
    └── Convergence_Summary.md                 ← Coordinator
```

所有文件都以周期 ID（`{YYYYMMDD}_cycle{N}`）命名，确保完整的可追溯性。六个月后，你可以追溯任何一行代码，回到启发它的研究、定义它的规格、实作它的开发日志，以及验证它的 QA 报告。

## 这对社区意味着什么

当 OpenStarry 发布时，贡献者继承的不只是代码：

1. **一套完整的开发流程**——不只有代码，还有一套工作方法
2. **架构护栏**——自动化纯度测试防止善意的贡献侵蚀架构
3. **可追溯的决策**——每一个「为什么」都有文件记录，不会迷失在聊天记录中
4. **平行安全的开发**——Dev-Core 和 Dev-Plugin 可以是同时工作的不同贡献者，因为 SDK 接口是冻结的
5. **可重现的质量**——无论开发者是人类还是 AI，SOP 都是一样的

> 流程本身就是产品的一部分。
