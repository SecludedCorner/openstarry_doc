# OpenStarry Agent Corps — 角色与标准操作程序 (Roles & SOP)

建立日期：2026-02-09
最后更新：2026-02-09 (PMP 审计 v2)

---

## 1. 代理人名册 (Agent Roster)

| Agent | 角色 | 模型 | 工作目录 | 输出目录 |
|-------|------|-------|-------------------|------------------|
| `architect` | 架构守护者，负责设计规范、代码审查、安全审计 | sonnet | share/openstarry_doc/, agent_dev/ (只读) | share/test/reports/arch_reviews/ |
| `dev-core` | 核心 monorepo 开发 (sdk, core, shared, runner) | sonnet | agent_dev/openstarry/ | share/test/reports/dev_logs/ |
| `dev-plugin` | 插件生态开发 | sonnet | agent_dev/openstarry_plugin/ | share/test/reports/dev_logs/ |
| `qa` | 质量保证 — 构建、测试、纯度验证 | sonnet | agent_test/ | share/test/reports/qa_results/ |
| `doc-keeper` | 文档管理、决策持久化、计划跟踪 | haiku | share/openstarry_doc/ | share/openstarry_doc/ |
| `researcher` | 技术预研、参考项目分析 | sonnet | share/ref/ | share/test/reports/research/ |
| **Coordinator** | 主会话 — 编排、同步、收敛、升级 | opus | all | share/test/reports/sys_summary/ |

---

## 2. RACI 矩阵

**R** = Responsible (执行), **A** = Accountable (负最终责任), **C** = Consulted (被咨询), **I** = Informed (被通知)

| Phase | Coordinator | architect | dev-core | dev-plugin | qa | doc-keeper | researcher | User |
|-------|:-----------:|:---------:|:--------:|:----------:|:--:|:----------:|:----------:|:----:|
| **Phase 0: Planning** | **R/A** | C | I | I | I | **R** (记录) | **R** (预研) | **A** (核准) |
| **Phase 1: Design** | I | **R/A** | C | C | I | **R** (记录设计决策) | C (提供研究) | I |
| **Phase 1.5: Baseline** | **R/A** | I | I | I | I | I | I | I |
| **Phase 2: Implement** | I | C (澄清 spec) | **R/A** (core) | **R/A** (plugin) | I | I | R (预研下轮) | I |
| **Phase 2.5: Sync** | **R/A** | I | I | I | I | I | I | I |
| **Phase 3: Verify** | I | **R** (代码审查) | I | I | **R/A** (测试) | I | I | I |
| **Phase 4: Converge** | **R/A** | I | I | I | I | **R** (更新文档) | I | **A** (裁决 PASS/FAIL) |
| **Rework** | **R** (分配) | C/R (若是设计问题) | R (若是 core 问题) | R (若是 plugin 问题) | I | I | I | I |
| **Escalation** | **R** (提报) | C | C | C | C | I | I | **A** (裁决) |

---

## 3. 标准迭代周期

### 命名规范

每轮迭代的 ID 格式：`{YYYYMMDD}_cycle{N}`

例：`20260210_cycle1`、`20260210_cycle2`（同天第二轮）

所有该轮的报告放在对应的 cycle 子目录：
```
share/test/reports/arch_reviews/20260210_cycle1/Architecture_Spec_Plan05.1.md
share/test/reports/dev_logs/20260210_cycle1/dev-core_Plan05.1.md
share/test/reports/qa_results/20260210_cycle1/QA_Report_Plan05.1.md
share/test/reports/research/20260210_cycle1/Research_SessionIsolation.md
share/test/reports/sys_summary/20260210_cycle1/Summary.md
```

---

### Phase 0: Planning

**触发条件**：User 下达指令，或上一轮 Phase 4 PASS 后 Coordinator 提议下一轮

**参与者**：Coordinator (R/A), researcher (R), doc-keeper (R), User (A)

**步骤**：
1. **Coordinator** 接收 User 指令，识别对应的 Implementation Plan
2. **Coordinator** 建立本轮 cycle 目录（所有 reports 子目录下）
3. **researcher** 预研该 Plan 的技术方案（Shift-Left），产出研究报告
4. **doc-keeper** 将本轮迭代计划记录到 `share/openstarry_doc/Agent_Corps/Iteration_Log.md`
5. **Coordinator** 确认研究报告已就绪，通知 architect 进入 Phase 1

**交付物**：
| 产出 | 路径 | 负责人 |
|------|------|--------|
| 研究报告 | `share/test/reports/research/{cycle_id}/Research_{Topic}.md` | researcher |
| 迭代计划记录 | `share/openstarry_doc/Agent_Corps/Iteration_Log.md` (追加) | doc-keeper |

**Exit Criteria**：
- [ ] 研究报告已存在且内容完整
- [ ] 迭代计划已记录
- [ ] Coordinator 确认可进入 Phase 1

---

### Phase 1: Design

**Entry Criteria**：Phase 0 Exit Criteria 全部满足

**参与者**：architect (R/A), doc-keeper (R), researcher (C)

**步骤**：
1. **architect** 读取：
   - Implementation Plan：`share/openstarry_doc/Implementation_Plans/Plan{XX}_{Name}.md`
   - 研究报告：`share/test/reports/research/{cycle_id}/Research_{Topic}.md`
   - 现有架构代码：`agent_dev/`（只读）
2. **architect** 产出 Architecture_Spec，必须包含：
   - **冻结的 Interface 定义**（TypeScript types）— dev-core 和 dev-plugin 共同依据
   - 技术约束与设计决策理由
   - Five Aggregates 归属
   - 安全考量
3. **doc-keeper** 记录设计决策理由到 `share/openstarry_doc/Agent_Corps/Iteration_Log.md`

**交付物**：
| 产出 | 路径 | 负责人 |
|------|------|--------|
| Architecture_Spec | `share/test/reports/arch_reviews/{cycle_id}/Architecture_Spec_{PlanName}.md` | architect |
| 设计决策记录 | `share/openstarry_doc/Agent_Corps/Iteration_Log.md` (追加) | doc-keeper |

**Exit Criteria**：
- [ ] Architecture_Spec 已存在
- [ ] Spec 包含冻结的 Interface 定义
- [ ] 设计决策已记录
- [ ] Coordinator 确认 Spec 完整，可进入 Phase 2

**重要**：Architecture_Spec 中的 Interface 定义一经发布即**冻结**。Phase 2 期间若需修改，必须走升级流程（见第 6 节）。

---

### Phase 1.5: Baseline（安全网）

**Entry Criteria**：Phase 1 Exit Criteria 全部满足

**参与者**：Coordinator (R/A)

**目的**：在 Phase 2 实作开始前，备份当前 agent_dev 状态。若 Phase 2 失败无法修复，可快速 rollback。

**执行**：`bash scripts/baseline.sh {cycle_id}`

**产出**：`share/openstarry_code_iteration/{cycle_id}_baseline/`

**恢复指令**：`bash scripts/restore.sh {cycle_id} baseline`

---

### Phase 2: Implementation

**Entry Criteria**：Phase 1.5 Baseline 已建立

**参与者**：dev-core (R/A), dev-plugin (R/A), architect (C), researcher (R 预研下轮)

**执行顺序**：

```
Step 2a: dev-core 实作 SDK Interface 变更（若有）
         → pnpm build 验证
         ↓
Step 2b: dev-core (剩余实作) + dev-plugin (plugin 实作) 并行
         → 各自 pnpm build 验证
```

若 Architecture_Spec 未变更 SDK Interface，则 Step 2a 跳过，dev-core 和 dev-plugin 直接并行。

**步骤（dev-core）**：
1. 读取 Spec：`share/test/reports/arch_reviews/{cycle_id}/Architecture_Spec_{PlanName}.md`
2. 若有 Interface 变更：先修改 `packages/sdk/`，执行 `pnpm build` 确认编译
3. 实作 `packages/core/`、`packages/shared/`、`apps/runner/` 相关变更
4. 执行 `pnpm build` 验证
5. 写 dev log

**步骤（dev-plugin）**：
1. 读取 Spec（同上路径）
2. **等待 Step 2a 完成**（若有 SDK Interface 变更）
3. 实作 plugin 变更
4. 执行 `pnpm build` 验证
5. 写 dev log

**步骤（researcher，并行）**：
- 预研下一轮迭代的技术方案（若已知）

**交付物**：
| 产出 | 路径 | 负责人 |
|------|------|--------|
| dev-core 日志 | `share/test/reports/dev_logs/{cycle_id}/dev-core_{PlanName}.md` | dev-core |
| dev-plugin 日志 | `share/test/reports/dev_logs/{cycle_id}/dev-plugin_{PlanName}.md` | dev-plugin |

**Exit Criteria**：
- [ ] dev-core `pnpm build` PASS
- [ ] dev-plugin `pnpm build` PASS（若有 plugin 变更）
- [ ] dev log 已写入
- [ ] Coordinator 确认 build 均通过，可进入 Phase 2.5

---

### Phase 2.5: Sync to Test Environment

**Entry Criteria**：Phase 2 Exit Criteria 全部满足

**参与者**：Coordinator (R/A)

**目的**：将 `agent_dev/` 的最新代码同步到 `agent_test/`，确保 QA 测试的是最新版本。

**快速执行**：`bash scripts/sync-to-test.sh`（从 openstarry_eco 根目录执行）

**Sync SOP（脚本内容）**：

```bash
# Step 1: 清除 agent_test 旧的源码（保留 node_modules 加速）
rm -rf agent_test/openstarry/packages agent_test/openstarry/apps
rm -rf agent_test/openstarry/dist agent_test/openstarry/.turbo

# Step 2: 复制源码（排除 node_modules, dist, .turbo）
cp -r agent_dev/openstarry/packages agent_test/openstarry/packages
cp -r agent_dev/openstarry/apps agent_test/openstarry/apps

# 复制根层配置文件（如有变更）
cp agent_dev/openstarry/package.json agent_test/openstarry/package.json
cp agent_dev/openstarry/tsconfig*.json agent_test/openstarry/
cp agent_dev/openstarry/vitest*.* agent_test/openstarry/ 2>/dev/null
cp agent_dev/openstarry/pnpm-workspace.yaml agent_test/openstarry/ 2>/dev/null

# Step 3: 同步 plugin（同样策略）
# 逐一复制每个 plugin 的 src, package.json, tsconfig
for dir in agent_dev/openstarry_plugin/*/; do
  plugin=$(basename "$dir")
  rm -rf "agent_test/openstarry_plugin/$plugin/src" "agent_test/openstarry_plugin/$plugin/dist"
  cp -r "agent_dev/openstarry_plugin/$plugin/src" "agent_test/openstarry_plugin/$plugin/src"
  cp "agent_dev/openstarry_plugin/$plugin/package.json" "agent_test/openstarry_plugin/$plugin/package.json"
  cp "agent_dev/openstarry_plugin/$plugin/tsconfig.json" "agent_test/openstarry_plugin/$plugin/tsconfig.json" 2>/dev/null
done
cp agent_dev/openstarry_plugin/package.json agent_test/openstarry_plugin/package.json 2>/dev/null
cp agent_dev/openstarry_plugin/pnpm-workspace.yaml agent_test/openstarry_plugin/pnpm-workspace.yaml 2>/dev/null

# Step 4: 安装依赖（处理新增/变更的 dependencies）
cd agent_test/openstarry && pnpm install
cd agent_test/openstarry_plugin && pnpm install

# Step 5: 验证基本 build
cd agent_test/openstarry && pnpm build
```

**Exit Criteria**：
- [ ] agent_test 源码已同步
- [ ] `pnpm install` 成功
- [ ] `pnpm build` 成功
- [ ] Coordinator 确认可进入 Phase 3

**失败处理**：若 sync 后 build 失败，问题在 Phase 2，退回 Phase 2 修正（不进入 Phase 3）。

---

### Phase 3: Verification

**Entry Criteria**：Phase 2.5 Exit Criteria 全部满足

**参与者**：qa (R/A), architect (R)

**可并行**：qa 和 architect 可同时进行，互不依赖。

**步骤（qa）**：
1. 在 `agent_test/openstarry/` 执行：
   - `pnpm build` — Build 验证
   - `pnpm test` — 单元测试（基线：82 tests）
   - `pnpm test:purity` — 微核心纯度检查
2. 在 `agent_test/openstarry_plugin/` 执行：
   - `pnpm build` — Plugin build 验证
   - `pnpm test` — Plugin 测试（若存在）
3. 产出 QA Report

**步骤（architect）**：
1. 读取 Architecture_Spec：`share/test/reports/arch_reviews/{cycle_id}/Architecture_Spec_{PlanName}.md`
2. 读取实作代码：`agent_dev/`（只读）
3. 逐项验证：
   - Interface 实作是否符合冻结的 Spec
   - Five Aggregates 合规
   - 微核心纯度
   - pushInput pattern
   - 安全漏洞
4. 产出 Code Review Report（PASS / FAIL / CONDITIONAL）

**交付物**：
| 产出 | 路径 | 负责人 |
|------|------|--------|
| QA Report | `share/test/reports/qa_results/{cycle_id}/QA_Report_{PlanName}.md` | qa |
| Code Review | `share/test/reports/arch_reviews/{cycle_id}/Code_Review_{PlanName}.md` | architect |

**Exit Criteria**：
- [ ] QA Report 已存在
- [ ] Code Review Report 已存在
- [ ] Coordinator 已读取两份报告，进入 Phase 4

---

### Phase 4: Convergence

**Entry Criteria**：Phase 3 Exit Criteria 全部满足

**参与者**：Coordinator (R/A), doc-keeper (R), User (A)

**步骤**：
1. **Coordinator** 读取 QA Report 和 Code Review Report
2. **Coordinator** 汇总为 Summary Report
3. **Coordinator** 判定：

   **→ 若全部 PASS：**
   - Snapshot：`bash scripts/snapshot.sh {cycle_id}`（排除 node_modules/dist）
   - **doc-keeper** 更新：
     - Implementation Plan 中完成项打 ✅
     - Iteration_Log.md 追加本轮结果
   - **Coordinator** 向 User 报告结果，询问下一步

   **→ 若任一 FAIL：** 进入 Rework 流程（见第 4 节）

**交付物**：
| 产出 | 路径 | 负责人 |
|------|------|--------|
| Summary Report | `share/test/reports/sys_summary/{cycle_id}/Summary.md` | Coordinator |
| 更新的 Iteration Log | `share/openstarry_doc/Agent_Corps/Iteration_Log.md` | doc-keeper |
| 更新的 Plan（✅） | `share/openstarry_doc/Implementation_Plans/` | doc-keeper |

---

## 4. Rework 流程（Phase 4 FAIL）

### 4.1 问题分级

| 级别 | 描述 | 退回到 | 示例 |
|------|------|--------|------|
| **Code Fix** | 实作 bug、测试失败、build 错误 | Phase 2（仅修正部分） | 类型错误、测试断言失败 |
| **Design Fix** | Spec 本身有缺陷或不完整 | Phase 1（architect 修订 Spec） | Interface 设计不合理、遗漏边界情况 |
| **Plan Fix** | Plan 需求本身有问题 | Phase 0（重新规划） | 技术方案不可行、需求矛盾 |

### 4.2 Rework SOP

1. **Coordinator** 读取 QA Report 和 Code Review Report 中的 FAIL 项目
2. **Coordinator** 对每个 FAIL 项目分级（Code Fix / Design Fix / Plan Fix）
3. **Coordinator** 产出 Rework Task，写入 Summary Report：
   ```
   ## Rework Tasks
   - [ ] [FAIL-1] (Code Fix) 描述 → 指派给 dev-core/dev-plugin
   - [ ] [FAIL-2] (Design Fix) 描述 → 指派给 architect
   ```
4. **Coordinator** 重建 Baseline：`bash scripts/baseline.sh {cycle_id}_rework{N}`
   （保留 rework 前的状态，以便二次 rollback）
5. 依分级退回对应 Phase，**只有 FAIL 的部分需要重做**
6. 修正完成后，从修正 Phase 的下一个 Phase 重新往下走（Phase 2.5 Sync 必须重跑）

### 4.3 Rework 上限

- 同一轮迭代最多 **2 次 Rework**
- 超过 2 次 → **升级到 User** 裁决（见第 6 节）

---

## 5. Phase 2 并行策略：Interface 冻结机制

### 5.1 原则

Architecture_Spec 中定义的 **TypeScript Interface** 一经 Phase 1 完成即**冻结**。dev-core 和 dev-plugin 共同依据此冻结 Interface 开发。

### 5.2 执行顺序

```
Architecture_Spec 是否包含 SDK Interface 变更？
│
├── 是 → dev-core 先行实作 SDK Interface（Step 2a）
│        → pnpm build 通过后
│        → dev-core (剩余) + dev-plugin 并行（Step 2b）
│
└── 否 → dev-core + dev-plugin 直接并行
```

### 5.3 冻结期间的例外处理

若 dev-core 在实作中发现 Interface 必须修改：
1. dev-core **暂停实作**
2. dev-core 在 dev log 中记录问题
3. **Coordinator** 通知 architect
4. architect 评估 → 发布 **Spec Addendum**（追加修订）到同一 cycle 目录
5. dev-plugin 读取 Addendum 后调整
6. 若影响范围过大 → 升级到 User

---

## 6. 升级与争议处理

### 6.1 升级条件

| 情境 | 升级路径 |
|------|---------|
| Agent 之间对 Spec 理解有分歧 | → Coordinator 仲裁 |
| Coordinator 无法仲裁（设计方向问题） | → User 裁决 |
| Rework 超过 2 次 | → User 裁决 |
| Phase 2 发现 Interface 需大幅修改 | → User 裁决 |
| 技术方案不可行（researcher 发现） | → User 裁决 |

### 6.2 升级流程

1. 发现方在报告中明确标记 `⚠️ ESCALATION NEEDED`
2. **Coordinator** 收集各方意见，整理为决策摘要
3. **Coordinator** 向 User 提出：
   - 问题描述
   - 各方立场
   - Coordinator 建议（若有）
4. **User** 裁决
5. **doc-keeper** 记录裁决到 Iteration_Log.md

---

## 7. doc-keeper 全程参与时间表

| Phase | doc-keeper 职责 |
|-------|----------------|
| Phase 0 | 记录：本轮迭代目标、涉及的 Plan、研究方向 |
| Phase 1 | 记录：architect 的设计决策理由、Interface 冻结内容 |
| Phase 1.5 | 无主动任务（Coordinator 建 baseline） |
| Phase 2 | 待命：收到 Spec Addendum 时记录变更 |
| Phase 3 | 待命：无主动任务 |
| Phase 4 PASS | 更新 Plan ✅、写 Iteration 结果、更新 Roadmap |
| Phase 4 FAIL | 记录 FAIL 原因和 Rework 决策 |

---

## 8. 沟通协议 (Communication Protocol)

Coordinator 派遣 agent 时，prompt 必须包含以下信息，避免 agent 猜测或工作在错误的上下文。

### 8.1 必要字段（所有派遣）

每次派遣 agent 的 prompt **必须包含**：

| 字段 | 说明 | 示例 |
|------|------|------|
| `cycle_id` | 本轮迭代 ID | `20260210_cycle1` |
| `plan` | 目标 Plan 名称 | `Plan05.1 Session Isolation` |
| `task` | 具体要做什么（一句话） | 「预研 Session Isolation 的技术方案」 |
| `output_path` | 报告/产出要写到哪里 | `share/test/reports/research/20260210_cycle1/` |

### 8.2 Phase 专属字段

| Phase | Agent | 额外必要字段 |
|-------|-------|-------------|
| Phase 0 | researcher | `research_topic`, `reference_projects`（要扫描 share/ref/ 的哪些目录） |
| Phase 0 | doc-keeper | `iteration_goals`（本轮目标摘要） |
| Phase 1 | architect | `plan_path`（Plan 文件路径）, `research_report_path`（研究报告路径） |
| Phase 1 | doc-keeper | `spec_summary`（Spec 的关键设计决策摘要） |
| Phase 2a | dev-core | `spec_path`, 明确指示「仅实作 SDK Interface 变更」 |
| Phase 2b | dev-core | `spec_path`, 明确指示「SDK Interface 已完成，实作剩余部分」 |
| Phase 2 | dev-plugin | `spec_path`, 若有 SDK 变更需指示「SDK Interface 已就绪」 |
| Phase 3 | qa | `cycle_id`（qa 不需要额外路径，它在 agent_test/ 工作） |
| Phase 3 | architect | `spec_path`（Architecture_Spec 路径，用于 code review 比对） |
| Phase 4 | doc-keeper | `qa_result`（PASS/FAIL）, `review_result`（PASS/FAIL）, `lessons_learned`（本轮回顾） |

### 8.3 Prompt 范本

**Phase 0 — researcher:**
```
Cycle: 20260210_cycle1
Plan: Plan05.1 Session Isolation
Task: 预研 Session Isolation 技术方案
Reference: share/ref/opencode/, share/ref/openclaw/, share/ref/openclaw研究/
Output: share/test/reports/research/20260210_cycle1/Research_SessionIsolation.md
重点研究：WebSocket auth 机制、multi-instance session 管理、session 生命周期
```

**Phase 2a — dev-core (SDK only):**
```
Cycle: 20260210_cycle1
Plan: Plan05.1 Session Isolation
Spec: share/test/reports/arch_reviews/20260210_cycle1/Architecture_Spec_Plan05.1.md
Task: 仅实作 Architecture_Spec 中的 SDK Interface 变更（packages/sdk/）。
      完成后执行 pnpm build 验证。不要实作 core/shared/runner 的变更。
Output: share/test/reports/dev_logs/20260210_cycle1/dev-core_Plan05.1_step2a.md
```

**Phase 2b — dev-core (remaining):**
```
Cycle: 20260210_cycle1
Plan: Plan05.1 Session Isolation
Spec: share/test/reports/arch_reviews/20260210_cycle1/Architecture_Spec_Plan05.1.md
Task: SDK Interface 已在 Step 2a 完成。请实作 Spec 中 packages/core、packages/shared、
      apps/runner 的剩余变更。完成后执行 pnpm build 验证。
Output: share/test/reports/dev_logs/20260210_cycle1/dev-core_Plan05.1.md
```

---

## 9. 经验回顾 (Lessons Learned)

### 9.1 时机

每轮迭代 **Phase 4 结束后**（无论 PASS 或 FAIL），Coordinator 执行回顾。

### 9.2 回顾流程

1. **Coordinator** 回顾本轮迭代，回答以下问题：
   - 什么做得好？（流程、协作、质量）
   - 什么做得不好？（卡住的地方、rework 原因、沟通问题）
   - 下次可以怎么改善？（流程调整、prompt 优化、新增检查项）
2. **Coordinator** 将回顾写入 Summary Report 的 `## Lessons Learned` 段落
3. **doc-keeper** 将可持久化的改善项追加到 `share/openstarry_doc/Agent_Corps/Lessons_Learned.md`
4. 若发现需要修改 SOP/Agent 定义/Checklist → Coordinator 在下一轮 Phase 0 执行

### 9.3 Lessons Learned 格式

追加到 `share/openstarry_doc/Agent_Corps/Lessons_Learned.md`：
```
## {cycle_id}

### What went well
- ...

### What went wrong
- ...

### Action items
- [ ] [改善项描述] → 影响的文件/流程
```

### 9.4 风险登记册更新

若回顾中发现新风险或已知风险的概率/影响需调整：
- **Coordinator** 更新 `share/openstarry_doc/Agent_Corps/Risk_Register.md`

---

## 10. 故障恢复

### 10.1 恢复脚本

| 脚本 | 用途 | 指令 |
|------|------|------|
| `scripts/baseline.sh` | Phase 2 前建立安全网 | `bash scripts/baseline.sh {cycle_id}` |
| `scripts/restore.sh` | 从 baseline 或 snapshot 恢复 agent_dev | `bash scripts/restore.sh {cycle_id} baseline\|snapshot` |
| `scripts/sync-to-test.sh` | agent_dev → agent_test 同步 | `bash scripts/sync-to-test.sh` |
| `scripts/snapshot.sh` | Phase 4 PASS 后存档 | `bash scripts/snapshot.sh {cycle_id}` |

### 10.2 恢复场景

| 场景 | 恢复方式 |
|------|---------|
| Phase 2 把 agent_dev 改坏了 | `restore.sh {cycle_id} baseline` → 重跑 Phase 2 |
| Sync 中途中断 | 重跑 `sync-to-test.sh`（staging 策略保护 agent_test） |
| 错误的 PASS + 坏 snapshot | 删除坏 snapshot → `restore.sh {上一轮} snapshot` |
| Agent session 崩溃 | 检查文件 → 若不一致则 restore baseline → 重派 agent |
| Rework 后又失败 | `restore.sh {cycle_id}_rework{N} baseline` |

### 10.3 迭代状态判断

若主 session 崩溃，新 session 可通过检查 cycle 目录中的报告判断当前 Phase：

| 存在的报告 | 代表完成到 |
|-----------|-----------|
| `research/{cycle_id}/` 有报告 | Phase 0 完成 |
| `arch_reviews/{cycle_id}/Architecture_Spec_*` | Phase 1 完成 |
| `openstarry_code_iteration/{cycle_id}_baseline/` | Phase 1.5 完成 |
| `dev_logs/{cycle_id}/` 有报告 | Phase 2 完成 |
| `qa_results/{cycle_id}/` 有报告 | Phase 3 完成 |
| `sys_summary/{cycle_id}/` 有报告 | Phase 4 完成 |

详细恢复步骤见 `share/openstarry_doc/Agent_Corps/Coordinator_Checklist.md` 故障恢复手册。

---

## 11. 设计原则

1. **Roles ≠ Tasks** — 每个 Agent 是永久角色，不为单一任务而建
2. **All Tier 1** — 6 个 Agent 全部是核心团队，无阶层
3. **Extensible** — 既定 Plan 完成后，同一批 Agent 自然承接延伸任务
4. **Decision Persistence** — 所有决策必须写入文件，不允许只存在记忆中
5. **Quality Gates** — 每个 Phase 有明确的 Entry/Exit Criteria，不满足不推进
6. **Interface Freeze** — Spec 发布后 Interface 冻结，变更需走升级流程

---

## 12. 扩展模式 (Extended Mode)

触发条件：`share/openstarry_doc/Implementation_Plans/` 中所有 Plan 已完成 ✅

Coordinator 评估后询问 User 是否继续：
- **researcher** → 深度分析 ref 项目，产出新 plugin 构想 + 可行性报告
- **architect** → 全面安全稽核 + 沙盒强化方案
- **dev-core/dev-plugin** → 实作新功能（Web UI、新 plugin 等）
- **qa** → 验证延伸功能
- **doc-keeper** → 记录所有延伸工作的决策与成果
