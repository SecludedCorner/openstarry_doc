# 17. 开发者体验与工具链 (Developer Experience and Tooling)

为了让开发者能快速构建、测试并分享 Agent 与插件，我们提供了一套完整的 CLI 工具链与开发环境。

## 1. OpenStarry CLI (`openstarry`)

`openstarry` 是用户与开发者的唯一入口。相关基础指令定义请参阅 **[09. CLI 设计与管理指令](./09_CLI_Design_and_Management_Commands.md)**。

### 常用开发指令 (Alias / Extensions)
*   `openstarry init <name>`: 创建一个新的 Agent 项目 (包含 `agent.json`, `config/`, `.env`)。
*   `openstarry create-plugin <name>`: 使用模板创建一个新的插件项目。
*   `openstarry dev`: 启动本地开发模式 (Hot Reload)。
*   `openstarry plugin add`: 安装依赖插件。
*   `openstarry start`: 启动 Agent。

## 2. 插件脚手架 (Scaffolding)

执行 `openstarry create-plugin my-tool` 将会生成以下标准结构：

```text
my-tool/
├── src/
│   ├── index.ts        # 入口点
│   └── tools.ts        # 工具逻辑
├── tests/
│   └── index.test.ts   # 预置的单元测试
├── package.json        # 依赖配置
├── tsconfig.json       # TypeScript 配置
└── README.md           # 文档模板
```

**模板特色：**
*   **Pre-configured Build:** 内置 `tsup` 或 `rollup` 配置，一键打包。
*   **Type Safety:** 自动引入 `@openstarry/sdk` 类型定义。
*   **Linting:** 预设 ESLint/Prettier 规则。

## 3. MockHost: 实验室环境 (The Lab)

开发插件时，启动一个完整的 Agent (包含 LLM 连接) 既昂贵又缓慢。我们提供 `MockHost` 测试环境。

### 功能
*   **模拟 Core 行为:** 开发者可以在测试脚本中模拟 Core 发出的指令 (如 `callTool`)。
*   **虚拟上下文:** 无需真实 LLM，直接注入伪造的 Context 数据。
*   **快速反馈:** 支持 Watch 模式，修改代码后立即重跑测试。

### 使用范例 (在插件测试中)

```typescript
import { TestAgentHost } from '@openstarry/sdk/testing';
import { MyPlugin } from '../src';

test('MyPlugin loads correctly', async () => {
  const host = new TestAgentHost();
  const plugin = new MyPlugin();

  await host.load(plugin);

  // 模拟 Core 调用工具
  const result = await host.executeTool('my_tool_function', { arg: 'value' });

  expect(result).toBe('success');
});
```

## 4. 调试与可视化 (Debugging)

*   **Inspector:** `openstarry dev` 启动时会开启一个本地 WebSocket 端口。开发者可以使用 `OpenStarry DevTools` (Web UI) 连接，实时查看 Agent 的思考过程、Context 变化与工具调用日志。
*   **VS Code Extension:** (未来规划) 直接在编辑器中语法高亮 Agent DSL，并提供插件代码补全。
