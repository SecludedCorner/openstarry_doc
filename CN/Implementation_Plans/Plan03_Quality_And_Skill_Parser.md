# OpenStarry 实施计划 03 — 质量补强与架构纯化

> **状态**: ✅ 已完成 (2026-02-05)

## 背景

Plan01 完成了 MVP 骨架，Plan02 补齐了事件驱动架构与安全熔断。
测试人员于 2026-02-05 确认所有测试通过，并提出下一步建议。
本计划根据设计文件差距分析结果，聚焦于 **v0.1 → v0.2 的质量升级与架构纯化**。

---

## 目标

1. **质量补强**：让现有系统更健壮，避免配置错误崩溃、工具卡死、调试困难。
2. **能力扩展**：实现 Markdown 技能解析，让 Agent 能动态载入人设与行为准则。
3. **架构纯化**：Runner 完全解耦插件、所有插件动态载入、统一完整包名。

---

## Phase A：质量补强（v0.2 基础） ✅ 已完成

### A1. agent.json Zod 运行时验证 ✅

- 路径：`packages/shared/src/utils/config-schema.ts`
- 使用 Zod 定义 `AgentConfigSchema`，涵盖所有配置字段
- 在 `apps/runner/src/bin.ts` 的 `loadConfig()` 中使用 schema 验证
- 验证失败时输出清晰的错误信息

### A2. 工具调用 Timeout ✅

- 路径：`packages/core/src/execution/loop.ts` 的 `executeTool()`
- 使用 `Promise.race([tool.execute(...), timeoutPromise])` 包裹
- Timeout 值从 `config.policy.toolTimeout`（默认 30000ms）获取

### A3. TraceID 机制 ✅

- 在 `processEvent()` 开头生成 `traceId`
- 注入 logger 和事件 payload，串联一次完整处理周期

### A4. CI 纯净性检查 ✅

- `scripts/check-purity.sh` + `pnpm test:purity`
- core 不引用 plugin/apps、sdk 不引用 core/shared

---

## Phase B：能力扩展 ✅ 已完成

### B1. standard-function-skill 插件 ✅

- 路径：`openstarry_plugin/standard-function-skill/`
- 读取 `.md` 技能文件，解析 YAML Frontmatter + Markdown Body
- 作为 Guide 插件注册到 GuideRegistry
- **动态载入**：不硬编码在 Runner 中，通过 `agent.json` 配置：
  ```json
  {
    "name": "@openstarry-plugin/standard-function-skill",
    "config": { "skillPath": "./skills/my-agent.md" }
  }
  ```
- 示例技能文件位于 `openstarry_plugin/standard-function-skill/examples/coder.md`
- 7 项单元测试覆盖 frontmatter 解析

### B2. system_prompt 外部文件引用 ✅

- `IAgentConfig` 新增 `guideFile?: string` 字段
- AgentCore 启动时读取外部 `.md` 文件，注册为 FileGuide
- 两种方式并存：`guide`（Guide ID）或 `guideFile`（文件路径）

---

## Phase C：架构纯化 ✅ 已完成

### C1. IPluginContext.pushInput ✅

**问题：** stdio 插件依赖宿主注入的 `onInput` 回调来推送用户输入。这使得 CLI 必须认识 stdio 插件并做特殊处理，违反了完全动态载入的目标。

**解决：**
- SDK 的 `IPluginContext` 新增 `pushInput(event: InputEvent)` 方法
- Core 的 `getPluginContext()` 自动注入 `pushInput → core.pushInput`
- 所有 Listener 插件通过标准化的 context 推送输入，不再依赖宿主回调

### C2. 移除 BUILTIN_FACTORIES ✅

**问题：** Runner 的 `BUILTIN_FACTORIES` 硬编码了三个插件的 import，每新增插件就要改 Runner。

**解决：**
- 完全移除 `BUILTIN_FACTORIES` 和所有 `@openstarry-plugin/*` import
- 插件解析仅剩两层：
  1. `ref.path` → 文件路径直接载入
  2. `import(ref.name)` → 完整包名从 workspace / node_modules 动态载入
- Runner 的 `package.json` 不再依赖任何插件包

### C3. 统一完整包名 ✅

- `agent.json` 中所有插件使用完整包名：`@openstarry-plugin/xxx`
- defaultConfig 的 plugins 列表也使用完整包名
- 消除短名/完整名混用的问题

### C4. apps/cli → apps/runner ✅

- 重新命名为 `apps/runner`，package name 改为 `@openstarry/runner`
- 反映其「纯启动器」角色：读 config → 建 core → 动态载入插件 → 启动
- 不认识任何具体插件

---

## 实施顺序（已全部完成）

```
A1 agent.json Zod 验证          ✅
A2 工具调用 Timeout             ✅
A3 TraceID 机制                 ✅
A4 CI 纯净性检查               ✅
B1 standard-function-skill      ✅
B2 system_prompt 外部文件引用   ✅
C1 IPluginContext.pushInput     ✅
C2 移除 BUILTIN_FACTORIES       ✅
C3 统一完整包名               ✅
C4 apps/cli → apps/runner       ✅
```

---

## 验证结果

1. `pnpm build` — ✅ 全部 8 个 workspace project 编译通过
2. `pnpm test` — ✅ 56 项测试通过（7 个测试文件）
3. `pnpm test:purity` — ✅ 纯净性检查通过
4. Runner 零插件依赖 — ✅ `apps/runner/package.json` 只依赖 core/sdk/shared
