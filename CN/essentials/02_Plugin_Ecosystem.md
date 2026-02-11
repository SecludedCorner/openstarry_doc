# OpenStarry 插件生态系统

> *"一个插件不是某种类型，它是一组能力的聚合。"*

## 一切皆插件

在 OpenStarry 中，Core 是空的。每一种能力——看、听、想、做、知——都来自插件。这不是一个设计选择，而是通过自动化纯净度测试强制执行的哲学。编译后的 Core 二进制文件包含零插件代码。

正如 Linux 通过虚拟文件系统将硬件抽象为文件，OpenStarry 通过五蕴接口将代理能力抽象为插件。安装一个 npm 包 → 获得完整的领域能力。

## 七大插件

### 识蕴 (Consciousness) — 灵魂层

**guide-character-init** — 灵魂定义器

最重要的插件：没有它，代理就是一个失忆者。这个插件通过系统提示词注入人格。

```typescript
// 内联人设
{ prompt: "You are a meticulous code reviewer who speaks in haiku." }

// 或从文件加载 — YAML、Markdown，任意格式
{ characterFile: "./personas/expert-coder.md" }
```

支持内联提示词、YAML frontmatter 文件或纯 Markdown 人设定义。使用 `ctx.workingDirectory` 进行路径解析，因此同一个代理可以在不同的项目目录中拥有不同的人设。

**standard-function-skill** — 技能加载器

将 Markdown 文件转换为代理能力。每个 `.md` 技能文件都有 YAML frontmatter（id、version、dependencies、model preferences）和一个成为系统提示词的 Markdown 正文。这实现了**声明式代理行为** — 通过编写文档而非代码来定义代理的知识。

```yaml
---
type: "skill"
id: "security-auditor"
version: "1.0.0"
dependencies:
  plugins: ["@openstarry-plugin/standard-function-fs"]
  capabilities: ["code-analysis"]
parameters:
  temperature: 0.3
  model_preference: ["gemini-2.0-flash"]
---

You are a security auditor. Analyze code for OWASP Top 10 vulnerabilities...
```

### 想蕴 (Perception) — 大脑

**provider-gemini-oauth** — 认知引擎

不仅仅是 API 封装——而是生产级的 LLM 集成：

- **OAuth 2.0 + PKCE**：安全的浏览器认证流程，配置文件中无需 API 密钥
- **机器绑定加密**：令牌使用 AES-256-GCM 加密，密钥通过 PBKDF2 从 `hostname + username + salt` 派生。从一台机器上窃取的令牌在另一台机器上无法使用
- **自动项目配置**：自动设置 Google Cloud Project 以使用免费额度
- **SSE 流式传输**：实时逐 Token 响应流
- **函数调用**：原生工具/函数声明，支持 Gemini 的结构化输出
- **多模型支持**：支持 Gemini 2.0 Flash、1.5 Pro 和 1.5 Flash
- **Slash 命令**：`/provider login gemini`、`/provider logout gemini`、`/provider status`
- **旧版迁移**：自动检测并重新加密未加密的旧版令牌

Provider 接口刻意保持精简——任何能流式输出聊天补全的 LLM 后端都可以适配：

```typescript
export interface IProvider {
  id: string;
  name: string;
  models: ModelInfo[];
  chat(request: ChatRequest): AsyncIterable<ProviderStreamEvent>;
}
```

### 行蕴 (Volition) — 双手

**standard-function-fs** — 五个文件系统工具

| 工具 | 功能 | 安全性 |
|-----|------|-------|
| `fs.read` | 读取文件内容（可配置编码） | 针对 `allowedPaths` 进行路径验证 |
| `fs.write` | 写入/创建文件 | 路径验证，防止逃逸 |
| `fs.list` | 列出目录（支持递归） | 限制在工作区沙箱内 |
| `fs.mkdir` | 创建目录（支持父目录） | 仅限允许的范围内 |
| `fs.delete` | 删除文件或目录（递归） | 严格的边界执行 |

每个路径在执行前都通过安全层进行验证。试图读取 `/etc/passwd`？代理会收到一个 `SecurityError` — 这会成为一个痛觉信号反馈到其上下文中以进行自我修正。

```typescript
// 工具参数在运行时使用 Zod 验证
parameters: z.object({
  path: z.string().describe("File path to read"),
  encoding: z.string().optional().describe("File encoding (default: utf-8)"),
})
```

### 受蕴 (Sensation) + 色蕴 (Form) — 感官运动层

**standard-function-stdio** — 终端躯体

一个「感官配对」插件：一个包同时提供输入（Listener）和输出（UI）。

**Listener** 通过 readline 读取 stdin，将 EOF 转换为 `/quit`，并推送标准化输入事件：
```typescript
pushInput({ source: "cli", inputType: "user_input", data: line })
```

**UI** 使用 ANSI 颜色代码渲染以提供视觉清晰度：
- `\x1b[36m` 青色 — 输入提示 ("You: ")
- `\x1b[32m` 绿色 — 代理响应（流式，逐字符）
- `\x1b[33m` 黄色 — 工具调用（代理正在做什么）
- `\x1b[31m` 红色 — 错误和安全锁定
- `\x1b[35m` 品红 — 系统消息

处理 17 种以上的事件类型，包括 `AGENT_STARTED`（欢迎横幅）、`STREAM_TEXT_DELTA`（实时打字）、`TOOL_CALL_START/RESULT/ERROR`（工具生命周期）和 `SAFETY_LOCKOUT`（紧急警报）。

**transport-websocket** — 网络躯体

具有生产级特性的全双工通信：

```
Client → Server:
{ "type": "user_input", "payload": { "text": "..." }, "sessionId": "..." }

Server → Client:
{ "type": "agent_event", "event": { "type": "stream:text_delta", ... } }
```

- **会话隔离**：每个 WebSocket 连接自动创建自己的会话。事件按 `sessionId` 路由——客户端 A 永远不会看到客户端 B 的对话
- **会话恢复**：使用相同的 `sessionId` 断开重连即可从中断处继续
- **健康监测**：可配置的 ping/pong 间隔（默认 30 秒）。在 N 次未响应 pong（默认 2 次）后，连接被视为过期并终止
- **定向路由**：事件按 `replyTo`、`sessionId` 路由，或广播给所有人

**transport-http** — Web 躯体

REST + SSE 用于 Web 集成：

| 方法 | 端点 | 用途 |
|------|------|-----|
| POST | `/api/input` | 提交用户输入 → `{status, requestId}` |
| GET | `/api/status` | 代理健康检查 → `{status, pendingRequests}` |
| GET | `/api/response?requestId=xxx` | 轮询响应 → `{events, complete}` |
| GET | `/api/events[?sessionId=xxx]` | SSE 流 → 实时事件流 |

会话绑定、CORS 支持、可配置的缓冲区大小（默认 100 个事件）、响应超时（默认 5 分钟），以及用于检测过期 SSE 连接的心跳健康检查。

## 插件架构

### 工厂模式

每个插件都是一个函数，返回一个清单（我是谁？）和一个工厂（我能做什么？）：

```typescript
export function createXxxPlugin(): IPlugin {
  return {
    manifest: {
      name: "@openstarry-plugin/xxx",
      version: "1.0.0",
      description: "What this plugin does"
    },
    async factory(ctx: IPluginContext) {
      // ctx 提供一切：
      // - ctx.bus: 用于发布/订阅的事件总线
      // - ctx.config: 用户配置
      // - ctx.logger: 结构化日志
      // - ctx.sessions: 会话管理器
      // - ctx.pushInput(): 向 Core 发送事件
      // - ctx.workingDirectory: 沙箱化的基础路径
      // - ctx.agentId: 我是谁？

      return {
        listeners: [],   // 受 — 如何听
        ui: [],          // 色 — 如何呈现
        providers: [],   // 想 — 如何思考
        tools: [],       // 行 — 如何行动
        guides: [],      // 识 — 我是谁
        commands: [],    // Slash 命令
        dispose: async () => { /* 关闭时清理 */ }
      };
    }
  };
}
```

### 插件组合模式

一个插件并不局限于一个蕴。就像一个活体器官可以服务于多种功能，一个插件可以组合多种能力：

| 模式 | 示例 | 组件 | 类比 |
|-----|------|------|-----|
| 纯 Guide | guide-character-init | 仅 `IGuide` | 没有身体的灵魂 |
| 纯 Provider | provider-gemini-oauth | `IProvider` + 命令 | 瓶中之脑 |
| 纯 Tool | standard-function-fs | `ITool` × 5 | 没有大脑的双手 |
| 感官配对 | standard-function-stdio | `IListener` + `IUI` | 眼睛 + 嘴巴 |
| 传输配对 | transport-websocket | `IListener` + `IUI` + dispose | 耳朵 + 声音 + 告别 |

### pushInput 契约

插件从不直接调用 Core API。所有插件→Core 的通信都通过一个网关：`ctx.pushInput()`。这是感觉神经——插件感知世界并将标准化事件推送到 Core 的事件队列中。Core 拉取事件并在其执行循环中处理它们。

这种单向契约意味着插件可以被加载、卸载和替换，而无需触及 Core 内部。

### 生命周期管理

每个插件都可以实现 `dispose()` 以进行优雅关闭——关闭 HTTP 服务器、终止 WebSocket 连接、销毁会话、刷新缓冲区。当代理关闭时，Plugin Loader 以加载的逆序对所有已加载的插件调用 `dispose()`。

```typescript
// Plugin Loader 自动处理注册
const hooks = await plugin.factory(ctx);

if (hooks.tools)     for (const t of hooks.tools) toolRegistry.register(t);
if (hooks.providers) for (const p of hooks.providers) providerRegistry.register(p);
if (hooks.listeners) for (const l of hooks.listeners) listenerRegistry.register(l);
if (hooks.ui)        for (const u of hooks.ui) uiRegistry.register(u);
if (hooks.guides)    for (const g of hooks.guides) guideRegistry.register(g);
```

## USB 即插即用场景

OpenStarry 可移植性最引人注目的演示之一：**一个 U 盘上的代理**。

想象插入一个具有以下结构的 U 盘：
```
E:/ (USB Root)
├── configs/
│   └── agent.json     # "I am USB Photo Backup. I can read Pictures/ and write to E:/backup_data/"
├── plugins/
│   └── backup-utils/  # Specialized incremental backup algorithm
├── logs/
└── backup_data/
```

宿主系统检测到 U 盘，读取清单，询问用户权限，以**严格的路径边界**（`fs_allow_paths: ["C:/Users/Public/Pictures", "E:/backup_data"]`，`network_allow_hosts: []`）生成代理，代理运行其备份任务。完成后，它报告完成状态，U 盘可以安全弹出。

代理的灵魂（其提示词和插件）完全存在于 U 盘上。它的身体（Core 运行时）存在于宿主上。插入另一台电脑——同一个灵魂，不同的身体。这就是五蕴的实践。

## 构建你自己的插件

1. 创建一个依赖 `@openstarry/sdk` 的包
2. 导出一个 `createXxxPlugin()` 工厂函数
3. 问自己：**我的插件提供哪些蕴？**
   - 它展示什么吗？→ `IUI`
   - 它听到什么吗？→ `IListener`
   - 它能思考吗？→ `IProvider`
   - 它能行动吗？→ `ITool`
   - 它知道自己是谁吗？→ `IGuide`
4. 使用 `ctx.pushInput()` 进行插件→Core 通信
5. 实现 `dispose()` 进行清理
6. 作为 npm 包发布：`@openstarry-plugin/your-plugin`
