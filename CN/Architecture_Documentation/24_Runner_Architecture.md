# 24. Runner 架构 (Runner Architecture)

本文档说明 `apps/runner` 的设计理念与职责边界。

---

## 1. 什么是 Runner？

Runner 是 OpenStarry 的**宿主引导程序**（Host Bootstrap Program）。它是 Agent 的入口点，但本身不包含任何业务逻辑。

```
apps/runner/src/bin.ts
```

Runner 的职责仅有四步：
1. 读取 `agent.json` 配置（或使用内置默认值）
2. 建立 `AgentCore` 实例
3. 动态解析并载入所有插件
4. 启动 Agent

---

## 2. 为什么不叫 CLI？

Runner 曾经叫做 `apps/cli`，但在 Plan03 Phase C 重构中更名。原因：

| 问题 | 说明 |
|------|------|
| 名不副实 | CLI 的互动体验（readline、颜色输出）由 `standard-function-stdio` 插件提供，不在 Runner 中 |
| 暗示耦合 | 「CLI」这个名字暗示它只能用于命令行，但 Runner 不知道也不关心 UI 是什么 |
| 微核心精神 | Runner 应该是一个与 UI 无关的纯启动器 |

---

## 3. Runner 的依赖

Runner 只依赖三个核心套件：

```json
{
  "dependencies": {
    "@openstarry/core": "workspace:*",
    "@openstarry/sdk": "workspace:*",
    "@openstarry/shared": "workspace:*"
  }
}
```

**零插件依赖。** 所有 `@openstarry-plugin/*` 套件都在运行时通过 `import()` 动态载入。

---

## 4. 插件如何推送输入？

在 Runner 解耦之前，stdio 插件依赖宿主注入的 `onInput` 回调。这迫使 Runner 必须认识 stdio 插件并做特殊处理。

现在的架构：

```
SDK 层：IPluginContext.pushInput(event: InputEvent)
         ↓
Core 层：getPluginContext() 自动注入 pushInput → core.pushInput
         ↓
插件层：stdio listener 收到用户输入 → ctx.pushInput({ source: "cli", data: text })
         ↓
Core 层：pushInput → 斜杠指令快速路径 / EventQueue → ExecutionLoop
```

**所有 Listener 插件**都可以通过 `ctx.pushInput` 推送输入。Runner 不参与这个流程。

---

## 5. 默认配置

当没有提供 `agent.json` 时，Runner 使用内置的 `defaultConfig()`：

```typescript
function defaultConfig(): IAgentConfig {
  return {
    identity: { id: "openstarry-agent", name: "OpenStarry Agent", ... },
    cognition: { provider: "gemini-oauth", model: "gemini-2.0-flash", ... },
    plugins: [
      { name: "@openstarry-plugin/provider-gemini-oauth" },
      { name: "@openstarry-plugin/standard-function-fs" },
      { name: "@openstarry-plugin/standard-function-stdio" },
    ],
    guide: "default-guide",
  };
}
```

注意：即使是默认配置，插件也是用完整套件名通过动态 import 载入。

---

## 6. 进程级职责

Runner 保留少数进程级关注点，这些不属于任何插件的责任范围：

| 职责 | 说明 |
|------|------|
| `process.exit()` | 监听 `__QUIT__` 事件，执行 process exit |
| `SIGINT / SIGTERM` | 收到终止信号时，优雅关闭 Agent |
| Config 载入 | 读取 `agent.json`、Zod 验证、fallback 到默认值 |

---

## 7. 未来演进

Runner 可以被不同的宿主取代，而不需要修改任何插件：

| 宿主 | 说明 |
|------|------|
| `apps/runner` | 当前的 Node.js 进程宿主 |
| `apps/daemon` | 未来的守护进程，管理多个 Agent 实例 |
| `apps/web-server` | 未来的 Web 服务宿主，通过 HTTP 接收输入 |

每个宿主做的事情都一样：读 config → 建 core → 载入插件 → 启动。差异只在进程管理和外部接口。
