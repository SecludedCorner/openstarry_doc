# Coordinator Quick-Reference Checklist (协调员快速参考清单)

每轮迭代的操作清单。Coordinator（主会话）逐项执行。

---

## 开始新迭代

```
Cycle ID: {YYYYMMDD}_cycle{N}
Target Plan: Plan__________
```

### Phase 0: Planning
- [ ] 确认 User 指令，识别 target Plan
- [ ] 建立 cycle 目录：
  ```bash
  CYCLE="{YYYYMMDD}_cycle{N}"
  mkdir -p share/test/reports/arch_reviews/$CYCLE
  mkdir -p share/test/reports/dev_logs/$CYCLE
  mkdir -p share/test/reports/qa_results/$CYCLE
  mkdir -p share/test/reports/research/$CYCLE
  mkdir -p share/test/reports/sys_summary/$CYCLE
  ```
- [ ] 派 **researcher** 预研 → 等待研究报告产出
- [ ] 派 **doc-keeper** 记录迭代计划到 Iteration_Log.md
- [ ] ✅ Exit: 研究报告存在 + 迭代计划已记录

### Phase 1: Design
- [ ] 通知 **architect**，提供：cycle_id + Plan 路径 + 研究报告路径
- [ ] 等待 Architecture_Spec 产出
- [ ] 确认 Spec 包含 **冻结的 Interface 定义**
- [ ] 确认 Spec 标注是否有 **SDK Interface 变更**（决定 Phase 2 顺序）
- [ ] 派 **doc-keeper** 记录设计决策
- [ ] ✅ Exit: Spec 存在 + Interface 冻结 + 决策已记录

### Phase 1.5: Baseline (安全网)
- [ ] 建立 pre-Phase-2 baseline：
  ```bash
  bash scripts/baseline.sh $CYCLE
  ```
- [ ] 确认输出 "Baseline Saved"
- [ ] ✅ 若 Phase 2 失败需要 rollback：`bash scripts/restore.sh $CYCLE baseline`

### Phase 2: Implementation
- [ ] **SDK Interface 有变更？**
  - YES →
    1. 派 **dev-core** Step 2a，prompt 明确指定：
       「只实作 Architecture_Spec 中的 SDK Interface 变更（packages/sdk/），完成后 pnpm build 验证。不要实作其他部分。」
    2. 等 dev-core Step 2a 完成（build pass）
    3. 并行派出：
       - **dev-core** Step 2b，prompt 明确指定：
         「SDK Interface 已在 Step 2a 完成，请实作 Spec 中 packages/core、packages/shared、apps/runner 的剩余变更。」
       - **dev-plugin**，prompt 正常指定 Spec 路径
  - NO → 直接派 **dev-core** + **dev-plugin** 并行
- [ ] 等待两者 `pnpm build` PASS + dev log 写入
- [ ] （可选）同时派 **researcher** 预研下一轮
- [ ] ⚠️ 若收到 `INTERFACE CHANGE NEEDED`：
  - 暂停 Phase 2
  - 通知 architect 评估 Spec Addendum
  - Addendum 发布后通知 dev-plugin 重读
- [ ] ✅ Exit: 所有 build PASS + dev log 存在

### Phase 2.5: Sync
- [ ] 执行 sync 脚本：
  ```bash
  bash scripts/sync-to-test.sh
  ```
- [ ] 确认 sync 完成（最后输出 "Sync Complete"）
- [ ] ⚠️ 若 sync 后 build 失败 → 退回 Phase 2
- [ ] ✅ Exit: agent_test build PASS

### Phase 3: Verify
- [ ] 派 **qa** 跑测试（在 agent_test/）→ 等待 QA Report
- [ ] 派 **architect** 做 Code Review（读 agent_dev/）→ 等待 Code Review Report
- [ ] （qa 和 architect 可并行）
- [ ] ✅ Exit: 两份报告都已产出

### Phase 4: Convergence
- [ ] 读取 QA Report + Code Review Report
- [ ] 判定结果：

**→ 全部 PASS：**
- [ ] Snapshot（排除 node_modules/dist）：
  ```bash
  bash scripts/snapshot.sh $CYCLE
  ```
- [ ] 派 **doc-keeper** 更新 Plan ✅ + Iteration_Log
- [ ] 写 Summary Report 到 `share/test/reports/sys_summary/$CYCLE/`
- [ ] **Lessons Learned 回顾**：
  - 本轮迭代中遇到哪些问题？（技术、流程、沟通）
  - 什么做法有效？什么做法需要改进？
  - 是否需要更新 Risk Register？（新风险、风险状态变更）
  - 派 **doc-keeper** 追加到 `Lessons_Learned.md`
- [ ] 向 User 报告结果

**→ 任一 FAIL：**
- [ ] 分类每个 FAIL 项目：
  - `Code Fix` → 退回 Phase 2（仅修正部分）
  - `Design Fix` → 退回 Phase 1（architect 修订 Spec）
  - `Plan Fix` → 退回 Phase 0（重新规划）
- [ ] Rework 次数 +1（本轮已 rework ___/2 次）
- [ ] 若 rework > 2 → 升级到 User
- [ ] 写 Rework 任务到 Summary Report
- [ ] 派 **doc-keeper** 记录 FAIL 原因到 Iteration_Log + Lessons_Learned

---

## 升级 Checklist

当需要升级到 User 时：
- [ ] 收集各方意见
- [ ] 整理：问题描述 + 各方立场 + Coordinator 建议
- [ ] 呈报 User 裁决
- [ ] 派 **doc-keeper** 记录裁决结果

---

## 故障恢复手册

### 情境 A：Phase 2 实作把 agent_dev 改坏了（build 不过、逻辑错乱）
```bash
# 恢复到 Phase 2 开始前的状态
bash scripts/restore.sh $CYCLE baseline
```
恢复后从 Phase 2 重新开始。

### 情境 B：sync 脚本中途断掉（agent_test 不完整）
```bash
# 重新执行即可（使用 staging 策略，断掉时 agent_test 仍保持旧状态）
bash scripts/sync-to-test.sh
```
若 agent_dev 本身也有问题 → 先恢复 agent_dev（情境 A），再重新 sync。

### 情境 C：Phase 4 错误地 PASS，snapshot 了坏代码
```bash
# 删除坏的 snapshot
rm -rf share/openstarry_code_iteration/$CYCLE

# 恢复 agent_dev 到上一轮的好 snapshot
bash scripts/restore.sh {上一轮_cycle_id} snapshot
```
然后重新跑整轮迭代。

### 情境 D：Agent session 中途崩溃（文件写到一半）
1. 检查 agent_dev 中的文件状态
2. 若代码不一致 → `bash scripts/restore.sh $CYCLE baseline`
3. 若只是报告没写完 → 重新派遣对应 agent 即可

### 情境 E：想退回到更早的版本
```bash
# 列出所有可用的 snapshot
ls share/openstarry_code_iteration/

# 恢复到指定版本
bash scripts/restore.sh {任意_cycle_id} snapshot
```

### 预防措施
- **Phase 1.5 Baseline 是必做步骤**，不可跳过
- 每轮 Phase 4 PASS 后的 snapshot 是永久保留的（除非手动删除）
- 如果不确定 agent_dev 状态，先跑 `cd agent_dev/openstarry && pnpm build` 验证
