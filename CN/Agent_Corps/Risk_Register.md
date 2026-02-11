# OpenStarry Agent Corps — 风险登记册 (Risk Register)

建立日期：2026-02-09

**评分标准**：概率 (L/M/H) × 影响 (L/M/H) = 风险等级 (Low/Medium/High/Critical)

---

## 技术风险

### R-T01: Agent Context Window 溢出
- **概率**: H（实现大型功能时几乎必然发生）
- **影响**: M（Agent 产出不完整或遗漏细节）
- **风险等级**: High
- **触发场景**: dev-core 需读取大量既有代码 + Spec + 写入大量变更
- **缓解策略**:
  - Coordinator 将大功能拆分为多个小任务分次派遣
  - Agent prompt 中明确指定仅读取必要文件
  - 复杂变更拆分为 Step 2a / 2b / 2c 多步骤
- **应对计划**: 若 Agent 产出不完整，Coordinator 检视已完成部分，派遣新 Agent session 继续

### R-T02: pnpm install 或 build 在 agent_test 失败
- **概率**: M
- **影响**: M（Phase 2.5/3 卡住）
- **风险等级**: Medium
- **触发场景**: 依赖冲突、workspace protocol 问题、Windows 路径问题
- **缓解策略**:
  - sync 脚本复制 pnpm-lock.yaml 以确保依赖一致
  - Phase 2 exit criteria 要求 build pass，理论上 sync 后也应通过
- **应对计划**: 若 sync build 失败但 agent_dev build 通过 → 检查 sync 脚本是否遗漏文件

### R-T03: Windows 环境兼容性
- **概率**: M
- **影响**: M（脚本或工具未按预期运行）
- **风险等级**: Medium
- **触发场景**: bash 脚本在 Windows Git Bash 中行为不同、路径分隔符、不支持 symlink
- **缓解策略**:
  - 脚本使用 `$(cd ... && pwd)` 获取绝对路径
  - 避免 symlink，使用 cp 复制
  - 脚本使用 `2>/dev/null || true` 处理非关键错误
- **应对计划**: 手动执行失败的步骤，记录问题供后续修正脚本

### R-T04: Plugin 依赖 SDK 版本不同步
- **概率**: M
- **影响**: H（plugin build 失败、runtime 类型不匹配）
- **风险等级**: High
- **触发场景**: dev-core 修改了 SDK interface，dev-plugin 使用旧版 SDK
- **缓解策略**:
  - Interface Freeze 机制确保 Spec 冻结
  - Phase 2 Step 2a 先完成 SDK → build → 再派遣 dev-plugin
  - workspace protocol (`workspace:*`) 自动链接本地版本
- **应对计划**: 通过 Spec Addendum 流程处理必要的 interface 变更

---

## 流程风险

### R-P01: Coordinator 主 Session 崩溃
- **概率**: M（长时间作业、token 耗尽）
- **影响**: H（迭代中断，状态可能不一致）
- **风险等级**: High
- **缓解策略**:
  - 所有状态通过文件持久化（报告 = 隐式状态标记）
  - SOP 8.3 定义了状态判断方法
  - 每个 Phase 的产出都是独立文件，不依赖 session 记忆
- **应对计划**: 新 session 读取 SOP + 检查 cycle 目录 → 判断 Phase → 继续

### R-P02: Rework 死循环
- **概率**: L
- **影响**: H（迭代永远无法完成）
- **风险等级**: Medium
- **触发场景**: 设计缺陷导致反复 FAIL，每次 rework 引入新问题
- **缓解策略**:
  - Rework 上限 2 次，超过则强制升级到 User
  - 每次 rework 前重建 baseline
- **应对计划**: 由 User 裁决是否放弃此计划或进行大幅度重新设计

### R-P03: Agent 产出质量不一致
- **概率**: M
- **影响**: M（垃圾进垃圾出，Phase 3 才发现）
- **触发场景**: Agent prompt 不够明确、Agent 误解 Spec
- **缓解策略**:
  - Communication Protocol 定义必要的 prompt 字段
  - Phase 3 双重验证（qa + architect）
  - dev log 记录所有变更供 review
- **应对计划**: 通过 Rework 流程处理

### R-P04: 报告或文档内容冲突
- **概率**: L
- **影响**: L（混淆但不致命）
- **触发场景**: 多个 Agent 同时写入相邻文件、doc-keeper 和 Coordinator 同时更新 Iteration_Log
- **缓解策略**:
  - 每个 Agent 有明确的写入目录，互不重叠
  - doc-keeper 是 Iteration_Log 的唯一写入者
- **应对计划**: Coordinator 发现冲突时手动合并

---

## 外部风险

### R-E01: Claude Pro Token 额度耗尽
- **概率**: M（大型迭代可能消耗大量 token）
- **影响**: H（整个迭代被迫暂停）
- **缓解策略**:
  - doc-keeper 使用 haiku（低成本）
  - 仅在需要时派遣 Agent，避免不必要的调用
  - 大功能拆分为多轮小迭代
- **应对计划**: 暂停迭代，等待额度恢复，按 SOP 8.3 状态判断恢复

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
| R-T02 | agent_test build 失败 | Medium | 监控中 |
| R-T03 | Windows 兼容性 | Medium | 监控中 |
| R-T04 | SDK 版本不同步 | High | 已缓解 (Interface Freeze) |
| R-P01 | 主 Session 崩溃 | High | 已缓解 (状态持久化) |
| R-P02 | Rework 死循环 | Medium | 已缓解 (上限 2 次) |
| R-P03 | Agent 产出质量不一致 | Medium | 已缓解 (Communication Protocol) |
| R-P04 | 报告内容冲突 | Low | 已缓解 (写入目录分离) |
| R-E01 | Token 额度耗尽 | Medium | 监控中 |
| R-E02 | Reference 项目过时 | Low | 接受 |
