# 开发者指南：单机运行与调试 (Standalone Execution)

本指南旨在帮助开发者在不启动完整 `Orchestrator Daemon` 和复杂基础设施的情况下，快速启动一个轻量级的 `Agent Core` 实例进行插件开发、调试和测试。

## 为什么需要单机模式？

在完整的生产环境中，Agent 由 Daemon 管理，配置由设计层提供，状态存储在数据库中。这对于日常开发来说太过厚重。
**单机模式 (Standalone Mode)** 允许你：
1.  使用本地的 JSON 配置文件直接初始化 Agent。
2.  使用内存 (In-Memory) 存储替代数据库。
3.  在控制台 (Console) 直接与 Agent 交互，无需 Web UI。
4.  快速测试新编写的 Tool 或 Listener 插件。

---

## 1. 最小化启动脚本 (`run_local.js`)

你可以创建一个简单的 Node.js 脚本来实例化和运行 Agent。

```javascript
// run_local.js
const { AgentCore } = require('./src/core/AgentCore');
const { PluginLoader } = require('./src/infrastructure/PluginLoader');
const { InMemoryStateManager } = require('./src/core/state/InMemoryStateManager');

async function main() {
  console.log("🚀 正在启动 OpenStarry 单机开发模式...");

  // 1. 初始化插件加载器
  // 指向你的本地插件目录
  const pluginLoader = new PluginLoader({
    pluginDirs: ['./plugins/my-new-tools', './plugins/standard-library']
  });

  // 2. 准备配置 (原本由 Daemon/Design Layer 提供)
  const agentConfig = {
    id: 'local-dev-agent-001',
    name: 'Local Developer Agent',
    role: 'assistant',
    llmProvider: {
      provider: 'gemini', // 或 'mock' 用于不消耗 Token 的测试
      model: 'gemini-pro',
      apiKey: process.env.GEMINI_API_KEY
    },
    systemPrompt: "你是一个单机调试助手。请简洁地回答用户的问题。",
    plugins: [
      '@openstarry-plugin/stdio-interface', // 一个用于在终端输入输出的 UI 插件
      '@openstarry-plugin/file-system-tools',
      '@openstarry-plugin/my-custom-plugin' // 你正在开发的插件
    ]
  };

  // 3. 实例化核心
  // 使用内存状态管理器，重启后数据丢失，适合测试
  const stateManager = new InMemoryStateManager();
  const agent = new AgentCore(agentConfig, pluginLoader, stateManager);

  // 4. 启动代理人
  await agent.start();
  console.log("✅ 代理人已就绪！请在下方输入指令：");
}

main().catch(err => {
  console.error("❌ 启动失败:", err);
});
```

---

## 2. 关键组件 Mock 技巧

在开发过程中，你可能没有实际的外部依赖。以下是如何 Mock 这些组件：

### A. Mock LLM Provider
如果你不想消耗 API Quota，或者想测试 Agent 对特定 LLM 输出的反应，可以使用 `MockProvider`。

```javascript
// plugins/mock-provider/index.js
class MockLLMProvider {
  constructor(config) {
    this.responses = config.cannedResponses || {};
  }

  async generate(prompt, context) {
    console.log(`[MockLLM] 收到 Prompt: ${prompt.substring(0, 50)}...`);
    
    // 简单的关键字匹配回应
    if (prompt.includes('列出文件')) {
      return { 
        content: "我将为您列出文件。", 
        tool_calls: [{ name: 'fs:list_dir', args: { path: '.' } }]
      };
    }
    
    return { content: "这是 Mock LLM 的自动回覆。" };
  }
}
```

### B. Console UI 插件 (`stdio-interface`)
为了在终端机直接对话，你需要一个简单的适配器插件。

```javascript
// plugins/stdio-interface/index.js
const readline = require('readline');

class StdioInterfacePlugin {
  constructor(coreApi) {
    this.core = coreApi;
    this.rl = readline.createInterface({
      input: process.stdin,
      output: process.stdout
    });
  }

  start() {
    // 监听核心消息
    this.core.on('onNewMessage', (msg) => {
      console.log(`
🤖 Agent: ${msg.content}`);
      this.prompt();
    });

    this.core.on('onToolCallRequest', (req) => {
      console.log(`
🛠️  请求执行工具: ${req.toolName}`);
      console.log(`   参数: ${JSON.stringify(req.args)}`);
      // 开发模式默认自动批准，或询问用户
      this.core.provideConfirmation(req.confirmationId, true);
    });

    this.prompt();
  }

  prompt() {
    this.rl.question('
👤 User: ', (input) => {
      this.core.submitUserInput(input);
      // 注意：这里不递归调用 prompt，而是等待 Agent 回覆后触发
    });
  }
}
```

---

## 3. 调试技巧 (Debugging Tips)

1.  **启用详细日志 (Verbose Logging):**
    在 `AgentCore` 初始化时，传入 `logger` 配置，将日志级别设为 `DEBUG`，这样可以看到完整的「事件队列」流动和「状态变更」。

2.  **断点调试:**
    使用 VS Code 的 "JavaScript Debug Terminal" 运行 `node run_local.js`，你可以在任何插件的 `execute` 方法中设置断点。

3.  **状态快照检查:**
    在交互过程中，你可以随时调用 `agent.stateManager.snapshot()` 来查看当前的完整 Context 对象，检查 `short_term_memory` 是否正确累积了对话历史。

---

## 4. 从单机到生产

当你的插件在单机模式测试通过后：
1.  将插件代码移动到标准的插件目录。
2.  更新你的 `Agent Template` JSON 文件（在设计层），加入新插件的名称。
3.  重启 Daemon 或创建新的 Agent 实例，即可在生产环境中使用。
