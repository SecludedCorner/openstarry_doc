# OpenStarry Agent Corps — Risk Register

Established: 2026-02-09

**评分标准**：概率 (L/M/H) × 影响 (L/M/H) = 风险等级 (Low/Medium/High/Critical)

---

## 技术风险

### R-T01: Agent Context Window 溢出
- **概率**: H（大型功能实现时几乎必然）
- **影响**: M（agent 产出不完整或遗漏细节）
- **风险等级**: High
- **触发场景**: dev-core 需读取大量既有代码 + Spec + 写入大量变更
- **缓解策略**:
  - Coordinator 将大功能拆成多个小任务分次派遣
  - Agent prompt 中明确指定只读取必要的文件
  - 复杂变更分成 Step 2a / 2b / 2c 多步骤
- **应对计划**: 若 agent 产出不完整，Coordinator 检视已完成部分，派遣新 agent session 继续

### R-T02: pnpm install 或 build 在 agent_test 失败
- **概率**: M
- **影响**: M（Phase 2.5/3 卡住）
- **风险等级**: Medium
- **触发场景**: 依赖冲突、workspace protocol 问题、Windows 路径问题
- **状态**: **已发生**（Cycle 18 Phase 2.5）— stale node_modules（yaml/vitest module resolution failures）
- **缓解策略**:
  - sync 脚本复制 pnpm-lock.yaml 确保依赖一致
  - Phase 2 exit criteria 要求 build pass，理论上 sync 后也应 pass
  - **NEW (Cycle 18 lesson)**：建立 node_modules 清理基础设施（Plan16+ 任务）
- **应对计划**:
  - 若 sync build 失败但 agent_dev build 通过 → 检查 sync 脚本是否遗漏文件
  - 或执行手动 node_modules cleanup 和 pnpm install
  - Plan16: 实现 CI 钩子自动清理旧的 node_modules（跨迭代环境清洁）

### R-T03: Windows 环境兼容性
- **概率**: M
- **影响**: M（脚本或工具不如预期运作）
- **风险等级**: Medium
- **触发场景**: bash 脚本在 Windows Git Bash 中行为不同、路径分隔符、symlink 不支持
- **缓解策略**:
  - 脚本使用 `$(cd ... && pwd)` 获取绝对路径
  - 避免 symlink，使用 cp 复制
  - 脚本使用 `2>/dev/null || true` 处理非关键错误
- **应对计划**: 手动执行失败的步骤，记录问题供后续修正脚本

### R-T04: Plugin 依赖 SDK 版本不同步
- **概率**: M
- **影响**: H（plugin build 失败、runtime 类型不匹配）
- **风险等级**: High
- **触发场景**: dev-core 改了 SDK interface，dev-plugin 使用旧版 SDK
- **缓解策略**:
  - Interface Freeze 机制确保 Spec 冻结
  - Phase 2 Step 2a 先完成 SDK → build → 再派 dev-plugin
  - workspace protocol (`workspace:*`) 自动链接本地版本
- **应对计划**: Spec Addendum 流程处理必要的 interface 变更

---

## 流程风险

### R-P01: Coordinator 主 Session 崩溃
- **概率**: M（长时间作业、token 用尽）
- **影响**: H（迭代中断，状态可能不一致）
- **风险等级**: High
- **缓解策略**:
  - 所有状态通过文件持久化（报告 = 隐式状态标记）
  - SOP 8.3 定义了状态判断方法
  - 每个 Phase 的产出都是独立文件，不依赖 session 记忆
- **应对计划**: 新 session 读取 SOP + 检查 cycle 目录 → 判断 Phase → 继续

### R-P02: Rework 死循环
- **概率**: L
- **影响**: H（迭代永远完不成）
- **风险等级**: Medium
- **触发场景**: 设计缺陷导致反复 FAIL，每次 rework 引入新问题
- **缓解策略**:
  - Rework 上限 2 次，超过强制升级到 User
  - 每次 rework 前重建 baseline
- **应对计划**: User 裁决是否放弃此 Plan 或大幅重新设计

### R-P03: Agent 产出品质不一致
- **概率**: M
- **影响**: M（垃圾进垃圾出，Phase 3 才发现）
- **触发场景**: Agent prompt 不够明确、agent 误解 Spec
- **缓解策略**:
  - Communication Protocol 定义必要的 prompt 字段
  - Phase 3 双重验证（qa + architect）
  - dev log 记录所有变更供 review
- **应对计划**: Rework 流程处理

### R-P04: 报告或文件内容冲突
- **概率**: L
- **影响**: L（混淆但不致命）
- **触发场景**: 多个 agent 同时写入相邻文件、doc-keeper 和 Coordinator 同时更新 Iteration_Log
- **缓解策略**:
  - 每个 agent 有明确的写入目录，不重叠
  - doc-keeper 是 Iteration_Log 的唯一写入者
- **应对计划**: Coordinator 发现冲突时手动合并

### R-P05: 集成点遗漏（Integration Point Missing）
- **概率**: M（新功能集成时容易遗漏）
- **影响**: M（功能无法运作，Phase 3 才发现）
- **触发场景**: Phase 2 实现写好独立模块但未集成到主流程；或用 TODO 注释代替实际代码
- **状态**: 已发现，Cycle 7 FAIL-1（import analysis 未集成到 sandbox-manager）
- **缓解策略**:
  - Phase 1 Spec 明确列出所有集成点
  - Phase 2 Code Review checklist: 检查是否有 TODO/FIXME 注释在集成点
  - Phase 2 建立「集成测试」专项检验每个集成点有被调用
- **应对计划**: 后续新功能必须有明确的「集成验证测试」，不可跳过

### R-P06: Spec 遗漏项目（Spec Omission）
- **概率**: M（复杂 Spec 易有遗漏）
- **影响**: M（实现完成但 Spec 中定义的组件未创建，Phase 3 FAIL）
- **触发场景**: Architecture Spec 定义了文件/组件清单，但 Phase 2 未全部创建；例如 Cycle 7 plugin-signer 包
- **状态**: 已发现，Cycle 7 FAIL-2（plugin-signer 包在 spec 中但未实现）
- **缓解策略**:
  - Phase 2 开始前，Coordinator 建立「Spec 文件清单检查清单」
  - Phase 2 结束时强制执行检查清单逐项验证
  - dev 日志中记录「创建的文件清单」供检查
- **应对计划**: 未来新 Plan 前必须准备检查清单

### R-P07: 未验证的假设（Unvalidated Assumptions）
- **概率**: M（复杂系统常含隐含假设）
- **影响**: M（code path 在特定情况下崩溃，例如 package-name plugins）
- **触发场景**: 代码假设某个条件成立（如「所有 plugin 都有文件路径」），但未在输入验证中确认
- **状态**: 已发现，Cycle 7 FAIL-3（signature verification 假设有 filePath）
- **缓解策略**:
  - Phase 1 Spec 列举所有边界条件（boundary conditions）
  - Phase 2 每个假设都要有显式 guard（if 检查）
  - 测试涵盖边界情况（package-name plugins、null values 等）
- **应对计划**: Code Review checklist 新增「假设验证」项目

---

## 外部风险

### R-E01: Claude Pro Token 额度用尽
- **概率**: M（大型迭代可能消耗大量 token）
- **影响**: H（整个迭代被迫暂停）
- **缓解策略**:
  - doc-keeper 使用 haiku（低成本）
  - 只在需要时派遣 agent，避免不必要的调用
  - 大功能拆成多轮小迭代
- **应对计划**: 暂停迭代，等待额度恢复，从 SOP 8.3 状态判断恢复

### R-E02: Reference 项目版本过时
- **概率**: L
- **影响**: L（researcher 报告基于旧版信息）
- **触发场景**: share/ref/ 中的 openclaw/opencode/openoctopus 不再更新
- **缓解策略**: researcher 在报告中标注参考版本和日期
- **应对计划**: 需要时手动更新 share/ref/

---

## 风险总览

| ID | 风险 | 等级 | 状态 |
|----|------|:----:|:----:|
| R-T01 | Context Window 溢出 | High | 监控中 |
| R-T02 | agent_test build 失败 | Medium | 已发生（Cycle 18），待缓解（基础设施） |
| R-T03 | Windows 兼容性 | Medium | 监控中 |
| R-T04 | SDK 版本不同步 | High | 已缓解（Interface Freeze） |
| R-P01 | 主 Session 崩溃 | High | 已缓解（状态持久化） |
| R-P02 | Rework 死循环 | Medium | 已缓解（上限 2 次） |
| R-P03 | Agent 产出品质不一致 | Medium | 已缓解（Communication Protocol） |
| R-P04 | 报告内容冲突 | Low | 已缓解（写入目录分离） |
| R-P05 | 集成点遗漏 | Medium | 已发现（Cycle 7），待缓解（检查清单） |
| R-P06 | Spec 遗漏项目 | Medium | 已发现（Cycle 7），待缓解（检查清单） |
| R-P07 | 未验证的假设 | Medium | 已发现（Cycle 7），待缓解（Guard 检查） |
| R-E01 | Token 额度用尽 | Medium | 监控中 |
| R-E02 | Reference 项目过时 | Low | 接受 |
