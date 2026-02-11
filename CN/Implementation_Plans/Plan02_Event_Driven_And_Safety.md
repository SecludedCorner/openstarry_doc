# OpenStarry 实施计划 02 — 架构补齐与质量提升

> **状态**: ✅ 已完成 (2026-02-05)

## 背景

Plan01 完成了 MVP v0.1 Alpha 的基础骨架（可编译、可启动、可接收指令）。
但经由测试人员的验证报告（`test/20260204/`），以及与 `openstarry_doc` 设计文件的对比，
识别出以下几类**实作与设计之间的落差 (Gaps)**。

本计划定义 Plan02 的改进范围，目标是**补齐 Phase 2.3 / Phase 3.1 的缺失，并达成设计文件的要求**。

---

## 一、测试报告识别的问题

| # | 来源 | 问题 | 严重度 |
|---|------|------|--------|
| G1 | Developer_Handoff_Report / Engineering_Implementation_Gaps | CLI 插件载入被硬编码 (`switch-case`)，无法动态 `import()` 第三方插件 | 🔴 高 |
| G2 | Engineering_Implementation_Gaps | `ExecutionLoop.run(string)` 接受字符串，非事件驱动；与文件「核心唯一输入源是事件队列」不符 | 🔴 高 |
| G3 | QA_MultiInput_Verification_Report | 工具执行期间的异步竞争风险 — 新输入与当前 loop 可能状态冲突 | 🟡 中 |
| G4 | Developer_Handoff_Report / QA_Report | 缺少并发单元测试，`EventQueue` FIFO 稳定度未验证 | 🟡 中 |

---

## 二、设计文件对比 — 未实作功能

| # | 文件来源 | 缺失功能 | 属于 Phase |
|---|----------|----------|-----------|
| D1 | `02_Headless_Agent_Core.md` | ExecutionLoop 应为 `WAITING_FOR_EVENT` 状态机，从 EventQueue pull 事件，而非直接接收 string | 2.1 |
| D2 | `12_Capabilities_Injection_Mechanism.md` | PluginLoader 应支持动态 `import()` 载入，根据 `agent.json` 路径解析 | 3.1 |
| D3 | `07_Safety_Circuit_Breakers.md` | 资源级熔断：Token 预算上限、循环次数上限 (Loop Cap) | 2.3 |
| D4 | `07_Safety_Circuit_Breakers.md` | 行为级熔断：重复工具调用侦测、错误级联熔断 | 2.3 |
| D5 | `12_Error_Handling_and_Self_Correction.md` | 「错误即痛觉」机制：工具错误标准化 + 挫折计数器 + 强制求助 | 2.3 |
| D6 | `09_Observability_and_Tracing.md` | 结构化 JSON 日志（含 agent_id, trace_id, module）| 3.1 |
| D7 | `15_Testing_Strategy_and_Infrastructure.md` | 单元测试基建 (Vitest)、核心纯净性检查、MockHost | 1.3 |
| D8 | `01_Execution_Loop.md` | 输出路由：根据事件 `source` 回传至正确渠道 | 2.1 |

---

## 三、Plan02 实作步骤

### Phase A：事件驱动转型（对应 G1, G2, D1, D2, D8）

**目标**：ExecutionLoop 不再直接接收 string，改为监听 EventQueue；CLI 支持动态插件载入。

#### A1. ExecutionLoop 事件化重构
- `ExecutionLoop` 新增 `start()` 方法：持续从 `EventQueue.pull()` 取出事件
- 事件 payload 标准化：`{ source: string, type: string, data: unknown }`
- `run(userInput)` 降级为内部方法，仅由事件触发器调用
- 加入 `isProcessing` 锁防止同时处理多事件（解决 G3 竞争问题）
- 输出路由：根据 `event.source` 决定回复渠道

#### A2. AgentCore 串接
- `AgentCore.processInput()` 改为推入 EventQueue 而非直接调用 loop
- Listener 插件推入事件至 EventQueue（而非直接调用 AgentCore）
- 保留 slash command 的快速路径（不进入 LLM 循环）

#### A3. CLI 动态插件载入
- 新增 `DynamicPluginLoader`：根据 `agent.json` 的 `plugins[].path` 使用 `import()` 动态载入
- 保留 builtin 插件的快速路径（无 path 时使用 switch-case fallback）
- 支持 `node_modules` 包名解析（`import(pluginName)`）

#### A4. 事件 Payload 标准化
- 定义 `InputEvent` 类型：
  ```typescript
  interface InputEvent {
    source: string;      // "cli", "webhook", "mcp"
    type: string;        // "user_input", "system_command"
    data: unknown;
    replyTo?: string;    // 回复渠道标识
  }
  ```

---

### Phase B：安全熔断与自我修正（对应 D3, D4, D5）

**目标**：实作 `SafetyMonitor` 模块，防止失控与无限循环。

#### B1. SafetyMonitor 模块
- **位置**：`packages/core/src/security/safety-monitor.ts`
- **职责**：
  - `beforeLLMCall()`: Token 预算检查
  - `afterToolExecution()`: 重复调用侦测 + 错误率计算
  - `onLoopTick()`: 循环次数上限检查

#### B2. 资源级熔断
- `MAX_LOOP_TICKS`：单次任务最大循环次数（默认 50，可在 `policy` 配置）
- `MAX_TOKEN_USAGE`：Token 累计消耗上限（依赖 Provider 回传的 `usage`）
- 触发后状态切换为 `SAFETY_LOCKOUT`

#### B3. 行为级熔断
- `ToolCallFingerprint` 历史队列：hash(toolName + args)
- 连续 N 次相同失败指纹 → 强制注入系统提示：「STOP and analyze why」
- 滑动窗口错误率（最近 10 次中 8 次失败 → `EMERGENCY_HALT`）

#### B4. 挫折计数器与自我修正
- 连续失败计数器
- 超过阈值（默认 5）→ 强制注入 System Prompt：「ask the user for help」
- 工具错误标准化为 `{ code, message, suggestion }` 结构

---

### Phase C：测试基建（对应 G4, D7）

**目标**：建立 Vitest 测试环境，覆盖核心逻辑。

#### C1. 测试框架配置
- 根目录加入 Vitest 配置
- 各 package 的 `test` script

#### C2. 核心单元测试
- `EventBus.test.ts`：多 handler、wildcard、错误隔离
- `EventQueue.test.ts`：FIFO 顺序、并发 push/pull、压力测试
- `StateManager.test.ts`：add/clear/snapshot/restore
- `ContextManager.test.ts`：滑动视窗截断逻辑
- `SecurityLayer.test.ts`：路径验证正确性
- `SafetyMonitor.test.ts`：各级熔断触发条件

#### C3. 整合测试
- `MultiSourceEvent.test.ts`：模拟多来源事件同时注入
- `ExecutionLoop.test.ts`：mock provider，验证完整 loop 流程
- `PluginLoader.test.ts`：动态载入验证

#### C4. 核心纯净性检查
- 加入 `dependency-cruiser` 或 ESLint 规则
- 确保 `packages/core` 不 import `plugins/*` 或 `apps/*`

---

### Phase D：可观测性提升（对应 D6）

**目标**：结构化 JSON 日志。

#### D1. Logger 升级
- `packages/shared/src/logger` 输出 JSON 格式
- 新增 `agent_id`, `trace_id` 字段
- 支持日志等级过滤（环境变量 `LOG_LEVEL`）

---

## 四、实作优先顺序

```
Phase A (事件驱动转型)     ← 🔴 最高优先级，测试报告的核心问题
  └→ A1 ExecutionLoop 事件化
  └→ A2 AgentCore 串接
  └→ A3 动态插件载入
  └→ A4 事件 Payload 标准化

Phase B (安全熔断)          ← 🔴 高，Phase 2.3 未完成项
  └→ B1 SafetyMonitor
  └→ B2 资源级熔断
  └→ B3 行为级熔断
  └→ B4 挫折计数器

Phase C (测试基建)          ← 🟡 中，Phase 1.3 未完成项
  └→ C1 Vitest 配置
  └→ C2 核心单元测试
  └→ C3 整合测试
  └→ C4 纯净性检查

Phase D (可观测性)          ← 🟢 低，可与其他 phase 并行
  └→ D1 Logger 升级
```

---

## 五、完成后的预期状态

完成 Plan02 后，项目将达到：

- **Phase 1.3** ✅ 测试基建 + CI 规则
- **Phase 2** ✅ 完整的意识内核（含安全熔断、事件驱动、自我修正）
- **Phase 3.1** ✅ 动态插件载入 + 事件标准化
- `openstarry_doc` 中 Phase 1~3.1 的所有设计要求均已落地

**距离 Phase 3 完成还差**：
- 3.2 `openstarry plugin sync` 指令
- 3.2 `guide-mcp` + `standard-function-skill` 插件
- 3.4 `openstarry create-plugin` 脚手架 + `MockHost`

**距离 Phase 4 完成还差**：
- 4.2 端到端 LLM 通话验证（需 OAuth 登录后手动测试）

---

## 六、验证方式

1. `pnpm install && pnpm build` — 编译通过
2. `pnpm test` — 所有单元测试 + 整合测试通过
3. EventQueue 压力测试通过（1000 事件 FIFO 正确）
4. 动态载入：`agent.json` 配置自定义 plugin path → CLI 成功载入
5. 安全熔断：模拟连续失败 → SafetyMonitor 正确触发 halt
6. 多输入模拟：两个事件源同时注入 → 逐一处理，无冲突
