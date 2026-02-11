# OpenStarry 实施计划 01 — MVP Alpha Foundation

> **状态**: ✅ 已完成 (2026-02-04)

## Target
实施阶段 1~4 以达成 **MVP v0.1 Alpha** 里程碑：
- 本地 CLI Agent 运行
- Gemini 作为 LLM Provider (PKCE + OAuth, 参考: `ref/openoctopus/packages/provider-gemini-oauth`)
- 本地文件系统操作 (fs 工具)
- 基础记忆 (5 轮滑动窗口)
- 完整的「感知 → 思考 → 工具调用 → 修正 → 响应」循环

## Architecture Overview
- **Monorepo**: `openstarry/` 包含 `apps/` 与 `packages/`
- **Plugin Ecosystem**: `openstarry_plugin/` (扁平结构，独立仓库)
- **Language**: TypeScript (ESM 模块)
- **Package Manager**: pnpm workspaces
- **Reference**: OpenOctopus 模式（工厂函数、EventBus、Zod、异步生成器）

---

## Structure

```
openstarry/
├── package.json
├── pnpm-workspace.yaml
├── tsconfig.base.json
├── tsconfig.json
├── example-agent.json
├── apps/runner/src/
│   ├── bin.ts          # CLI 入口 + 宿主引导
│   └── index.ts        # 重新导出
├── packages/sdk/src/
│   ├── types/          # message, tool, provider, plugin, listener, guide, agent, events
│   ├── errors/         # AgentError 等级体系
│   └── interfaces/     # IContextManager, IStateManager
├── packages/shared/src/
│   ├── logger/         # 结构化日志
│   ├── utils/          # UUID, Zod 验证辅助函数
│   └── constants/      # 事件常量
└── packages/core/src/
    ├── bus/            # EventBus (发布/订阅)
    ├── execution/      # EventQueue, ExecutionLoop (状态机)
    ├── state/          # StateManager (内存中)
    ├── memory/         # ContextManager (滑动窗口)
    ├── infrastructure/ # ToolRegistry, ProviderRegistry, ListenerRegistry,
    │                   # GuideRegistry, CommandRegistry, PluginLoader
    ├── security/       # SecurityLayer (路径护栏)
    ├── agents/         # AgentCore (主编排器)
    └── transport/      # TransportBridge (事件路由至监听器)

openstarry_plugin/
├── provider-gemini-oauth/src/index.ts   # Gemini PKCE+OAuth provider
├── standard-function-fs/src/index.ts    # fs.read/write/list/mkdir/delete
└── standard-function-stdio/src/index.ts # CLI I/O 监听器 + 默认 guide
```

## Implementation Steps (已完成)

1. **Monorepo Scaffolding** — pnpm workspace, tsconfig, package.json
2. **SDK Types** — Message, Tool, Provider, Plugin, Listener, Guide, Agent, Events, Errors
3. **Shared Utils** — Logger, UUID, Zod 辅助函数, 事件常量
4. **Core: EventBus + EventQueue** — 发布/订阅 + 异步 FIFO 队列
5. **Core: StateManager + ContextManager** — 内存历史记录 + 滑动窗口 (5 轮)
6. **Core: Registries** — Tool, Provider, Listener, Guide, Command 注册表 + PluginLoader
7. **Core: SecurityLayer** — 路径规范化 + 范围验证
8. **Core: ExecutionLoop** — IDLE → ASSEMBLING_CONTEXT → AWAITING_LLM → PROCESSING_RESPONSE → (tool_use 循环)
9. **Core: AgentCore + Transport** — 主编排器 + 事件广播桥接
10. **Plugin: provider-gemini-oauth** — PKCE, OAuth 回调服务器, AES-256-GCM 令牌存储, SSE 串流, 函数调用
11. **Plugin: standard-function-fs** — fs.read/write/list/mkdir/delete 及其路径安全
12. **Plugin: standard-function-stdio** — stdin 监听器 + ANSI stdout UI + 默认 guide 人设
13. **CLI: bin.ts** — 配置加载, 插件解析, AgentCore 引导
14. **Integration** — `pnpm install && pnpm build` 通过, CLI 启动并响应命令

## Key Design Decisions

1. **ESM only** — 所有包均使用 ES 模块
2. **Zod validation** — 工具参数在运行时验证
3. **Factory pattern** — 插件暴露工厂函数
4. **Async generators** — Provider 串流使用异步生成器
5. **No Daemon** — MVP CLI 直接引导 AgentCore (独立运行)
6. **PKCE + OAuth** — Gemini 使用 Google OAuth, 令牌通过机器绑定的 AES-256-GCM 加密
7. **In-memory state** — MVP 中无持久化, 状态存储在进程内存中
8. **Path security** — FS 工具通过路径规范化 + allowedPaths 限制

## Verification

```bash
cd openstarry
pnpm install && pnpm build           # 所有 7 个包编译通过
node apps/runner/dist/bin.js             # 使用默认配置启动
node apps/runner/dist/bin.js --config ./example-agent.json  # 使用自定义配置启动

# 首次使用: /provider login gemini   → 打开浏览器进行 OAuth
# 测试:      /help                    → 列出 4 个命令
# 测试:      /reset                   → 重置对话
# 测试:      /quit                    → 正常退出
# 对话:      "你好"                   → Gemini 响应（在 OAuth 登录后）
# 工具:      "列出 . 目录下的文件"     → Agent 调用 fs.list
```
