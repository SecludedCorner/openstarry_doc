# OpenStarry 插件生态系

> *"一个插件不是某种类型，它是一组能力的聚合。"*

## 一切皆插件

在 OpenStarry 中，Core 是空的。每一项能力——看、听、想、行动、认知——都来自插件。这不是设计选择，而是一种由自动化纯度测试所强制执行的哲学。编译后的 Core 二进制文件包含零插件代码。

就像 Linux 通过虚拟文件系统（VFS）将硬件抽象为文件，OpenStarry 通过五蕴接口将 Agent 能力抽象为插件。安装一个 npm 套件 → 获得完整的领域能力。

## 七个插件

### 识蕴（Consciousness）——灵魂层

**guide-character-init** ——灵魂定义者

最重要的插件：少了它，Agent 就是个失忆的人。这个插件通过 System Prompt 注入人格。

```typescript
// 行内人设
{ prompt: "You are a meticulous code reviewer who speaks in haiku." }

// 或从文件加载——YAML、Markdown、任何格式
{ characterFile: "./personas/expert-coder.md" }
```

支持行内 Prompt、YAML Frontmatter 文件，或纯 Markdown 角色设定。使用 `ctx.workingDirectory` 进行路径解析，因此同一个 Agent 可以在不同的项目文件夹中拥有不同的人格。

**standard-function-skill** ——技能加载器

将 Markdown 文件转化为 Agent 能力。每个 `.md` 技能文件都有 YAML Frontmatter（id、version、dependencies、model preferences）和一个成为 System Prompt 的 Markdown 正文。这实现了**声明式的 Agent 行为**——通过撰写文档而非代码来定义 Agent 的知识。

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

### 想蕴（Perception）——大脑

**provider-gemini-oauth** ——认知引擎

不只是 API 包装器——而是生产级的 LLM 整合：

- **OAuth 2.0 + PKCE**：安全的浏览器认证流程，配置文件中不需要 API Key
- **机器绑定加密**：Token 以 AES-256-GCM 加密，密钥通过 PBKDF2 从 `hostname + username + salt` 衍生。从某台机器窃取的 Token 在另一台机器上无法使用
- **自动项目配置**：自动创建 Google Cloud Project 以使用免费方案
- **SSE 串流**：实时逐 Token 的响应串流
- **Function Calling**：原生的工具/函数宣告，对接 Gemini 的结构化输出
- **多模型支持**：支持 Gemini 2.0 Flash、1.5 Pro 和 1.5 Flash
- **斜杠命令**：`/provider login gemini`、`/provider logout gemini`、`/provider status`
- **旧版迁移**：自动侦测并重新加密未加密的旧版 Token

Provider 的接口刻意保持精简——任何能够串流聊天完成结果的 LLM 后端都能适配：

```typescript
export interface IProvider {
  id: string;
  name: string;
  models: ModelInfo[];
  chat(request: ChatRequest): AsyncIterable<ProviderStreamEvent>;
}
```

### 行蕴（Volition）——双手

**standard-function-fs** ——五个文件系统工具

| 工具 | 功能 | 安全性 |
|------|------|--------|
| `fs.read` | 读取文件内容（可设置编码） | 依 `allowedPaths` 进行路径验证 |
| `fs.write` | 写入/创建文件 | 路径验证，防止逃逸 |
| `fs.list` | 列出目录（支持递归） | 限制在工作区范围内 |
| `fs.mkdir` | 创建目录（含父目录） | 仅限许可范围内 |
| `fs.delete` | 删除文件或目录（支持递归） | 严格边界限制 |

每个路径在执行前都会通过安全层验证。尝试读取 `/etc/passwd`？Agent 会收到一个 `SecurityError`——这会成为一个痛觉信号，反馈到其上下文中进行自我修正。

```typescript
// 工具参数在运行时通过 Zod 验证
parameters: z.object({
  path: z.string().describe("File path to read"),
  encoding: z.string().optional().describe("File encoding (default: utf-8)"),
})
```

### 受蕴 + 色蕴（Sensation + Form）——感觉运动层

**standard-function-stdio** ——终端机身体

一个「感官配对」插件：一个套件同时提供输入（Listener）和输出（UI）。

**Listener** 通过 readline 读取 stdin，将 EOF 转换为 `/quit`，并推送标准化的输入事件：
```typescript
pushInput({ source: "cli", inputType: "user_input", data: line })
```

**UI** 使用 ANSI 色码进行视觉呈现：
- `\x1b[36m` 青色——输入提示（"You: "）
- `\x1b[32m` 绿色——Agent 回应（串流，逐字符输出）
- `\x1b[33m` 黄色——工具调用（Agent 正在做什么）
- `\x1b[31m` 红色——错误与安全锁定
- `\x1b[35m` 洋红色——系统消息

处理 17 种以上的事件类型，包括 `AGENT_STARTED`（欢迎消息）、`STREAM_TEXT_DELTA`（实时打字效果）、`TOOL_CALL_START/RESULT/ERROR`（工具生命周期）、以及 `SAFETY_LOCKOUT`（紧急警报）。

**transport-websocket** ——网络身体

具备生产级特性的全双工通讯：

```
Client → Server:
{ "type": "user_input", "payload": { "text": "..." }, "sessionId": "..." }

Server → Client:
{ "type": "agent_event", "event": { "type": "stream:text_delta", ... } }
```

- **Session 隔离**：每个 WebSocket 连接自动建立独立的 Session。事件依 `sessionId` 路由——用户 A 永远看不到用户 B 的对话
- **Session 恢复**：断线后用相同的 `sessionId` 重新连接，即可从中断处继续
- **健康监控**：可设置的 ping/pong 间隔（默认 30 秒）。连续 N 次未响应 pong（默认 2 次），连接即被判定为过期并终止
- **定向路由**：事件依 `replyTo`、`sessionId` 路由，或广播至所有连接

**transport-http** ——Web 身体

REST + SSE 用于 Web 整合：

| 方法 | 端点 | 用途 |
|------|------|------|
| POST | `/api/input` | 提交用户输入 → `{status, requestId}` |
| GET | `/api/status` | Agent 健康检查 → `{status, pendingRequests}` |
| GET | `/api/response?requestId=xxx` | 轮询响应 → `{events, complete}` |
| GET | `/api/events[?sessionId=xxx]` | SSE 串流 → 实时事件流 |

支持 Session 绑定、CORS 支持、可设置的缓冲区大小（默认 100 个事件）、响应超时（默认 5 分钟），以及用于侦测过期 SSE 连接的心跳健康检查。

## 插件架构

### 工厂模式

每个插件都是一个函数，回传 manifest（我是谁？）和 factory（我能做什么？）：

```typescript
export function createXxxPlugin(): IPlugin {
  return {
    manifest: {
      name: "@openstarry-plugin/xxx",
      version: "1.0.0",
      description: "What this plugin does"
    },
    async factory(ctx: IPluginContext) {
      // ctx 提供你所需的一切：
      // - ctx.bus: Event Bus，用于发布/订阅
      // - ctx.config: 用户设置
      // - ctx.logger: 结构化日志
      // - ctx.sessions: Session 管理器
      // - ctx.pushInput(): 向 Core 发送事件
      // - ctx.workingDirectory: 沙箱化的基础路径
      // - ctx.agentId: 我是谁？

      return {
        listeners: [],   // 受 — 如何听
        ui: [],          // 色 — 如何呈现
        providers: [],   // 想 — 如何思考
        tools: [],       // 行 — 如何行动
        guides: [],      // 识 — 我是谁
        commands: [],    // 斜杠命令
        dispose: async () => { /* 关闭时的清理工作 */ }
      };
    }
  };
}
```

### 插件组合模式

一个插件不限于单一蕴。就像一个活体器官可以服务多种功能，插件可以组合不同的能力：

| 模式 | 范例 | 组成 | 类比 |
|------|------|------|------|
| 纯 Guide | guide-character-init | 仅 `IGuide` | 没有身体的灵魂 |
| 纯 Provider | provider-gemini-oauth | `IProvider` + commands | 瓶中的大脑 |
| 纯 Tool | standard-function-fs | `ITool` × 5 | 没有大脑的手 |
| 感官配对 | standard-function-stdio | `IListener` + `IUI` | 眼睛 + 嘴巴 |
| 传输配对 | transport-websocket | `IListener` + `IUI` + dispose | 耳朵 + 声音 + 告别 |

### pushInput 契约

插件绝不直接调用 Core API。所有插件→Core 的通讯都通过一个闸道：`ctx.pushInput()`。这就是感觉神经——插件感知世界，并将标准化的事件推送到 Core 的事件队列中。Core 从队列中取出事件，在其执行回路中处理。

这种单向契约意味着插件可以被加载、卸载和替换，而完全不需要触碰 Core 的内部。

### 生命周期管理

每个插件都可以实现 `dispose()` 进行优雅关闭——关闭 HTTP 服务器、终止 WebSocket 连接、销毁 Session、清空缓冲区。Plugin Loader 在 Agent 关闭时，会以加载的逆序对所有已加载的插件调用 `dispose()`。

```typescript
// Plugin Loader 自动处理注册
const hooks = await plugin.factory(ctx);

if (hooks.tools)     for (const t of hooks.tools) toolRegistry.register(t);
if (hooks.providers) for (const p of hooks.providers) providerRegistry.register(p);
if (hooks.listeners) for (const l of hooks.listeners) listenerRegistry.register(l);
if (hooks.ui)        for (const u of hooks.ui) uiRegistry.register(u);
if (hooks.guides)    for (const g of hooks.guides) guideRegistry.register(g);
```

## USB 随插即用情境

OpenStarry 最引人注目的可移植性展示之一：**USB 闪存盘上的 Agent**。

想象插入一个 USB 闪存盘，里面有这样的结构：
```
E:/ (USB Root)
├── configs/
│   └── agent.json     # "我是 USB 照片备份助手。我可以读取 Pictures/ 并写入 E:/backup_data/"
├── plugins/
│   └── backup-utils/  # 专用的增量备份算法
├── logs/
└── backup_data/
```

主机系统侦测到 USB，读取 manifest，询问用户授权，以**严格的路径边界**（`fs_allow_paths: ["C:/Users/Public/Pictures", "E:/backup_data"]`、`network_allow_hosts: []`）生成 Agent，然后 Agent 执行备份任务。完成后回报结果，USB 可安全退出。

Agent 的灵魂（它的 Prompt 与插件）完全存在于 USB 上。它的身体（Core 执行环境）存在于主机上。插到另一台电脑——同样的灵魂，不同的身体。这就是五蕴的实践。

## 建立你自己的插件

1. 建立一个依赖 `@openstarry/sdk` 的套件
2. 导出一个 `createXxxPlugin()` 工厂函数
3. 问问自己：**我的插件提供哪些蕴？**
   - 它要显示什么吗？→ `IUI`
   - 它要听取什么吗？→ `IListener`
   - 它会思考吗？→ `IProvider`
   - 它要执行动作吗？→ `ITool`
   - 它知道自己是谁吗？→ `IGuide`
4. 使用 `ctx.pushInput()` 进行插件→Core 通讯
5. 实现 `dispose()` 进行资源清理
6. 发布为 npm 套件：`@openstarry-plugin/your-plugin`
