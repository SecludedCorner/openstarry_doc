# Developer Guide: Standalone Execution & Debugging

This guide is designed to help developers quickly launch a lightweight `Agent Core` instance for plugin development, debugging, and testing, without the need to start the full `Orchestrator Daemon` and complex infrastructure.

## Why Standalone Mode?

In a full production environment, Agents are managed by the Daemon, configurations are provided by the Design Layer, and states are stored in a database. This is too heavyweight for daily development.
**Standalone Mode** allows you to:
1.  Initialize an Agent directly using a local JSON configuration file.
2.  Use In-Memory storage instead of a database.
3.  Interact with the Agent directly in the console, without a Web UI.
4.  Quickly test newly written Tool or Listener plugins.

---

## 1. Minimal Startup Script (`run_local.js`)

You can create a simple Node.js script to instantiate and run an Agent.

```javascript
// run_local.js
const { AgentCore } = require('./src/core/AgentCore');
const { PluginLoader } = require('./src/infrastructure/PluginLoader');
const { InMemoryStateManager } = require('./src/core/state/InMemoryStateManager');

async function main() {
  console.log("🚀 Starting OpenStarry Standalone Development Mode...");

  // 1. Initialize Plugin Loader
  // Point to your local plugin directories
  const pluginLoader = new PluginLoader({
    pluginDirs: ['./plugins/my-new-tools', './plugins/standard-library']
  });

  // 2. Prepare Configuration (normally provided by Daemon/Design Layer)
  const agentConfig = {
    id: 'local-dev-agent-001',
    name: 'Local Developer Agent',
    role: 'assistant',
    llmProvider: {
      provider: 'gemini', // or 'mock' for testing without consuming Tokens
      model: 'gemini-pro',
      apiKey: process.env.GEMINI_API_KEY
    },
    systemPrompt: "You are a standalone debugging assistant. Please answer user questions concisely.",
    plugins: [
      '@openstarry-plugin/stdio-interface', // A UI plugin for terminal input/output
      '@openstarry-plugin/file-system-tools',
      '@openstarry-plugin/my-custom-plugin' // The plugin you are developing
    ]
  };

  // 3. Instantiate Core
  // Use memory-based state manager; data is lost on restart, ideal for testing
  const stateManager = new InMemoryStateManager();
  const agent = new AgentCore(agentConfig, pluginLoader, stateManager);

  // 4. Start the Agent
  await agent.start();
  console.log("✅ Agent ready! Please enter instructions below:");
}

main().catch(err => {
  console.error("❌ Startup failed:", err);
});
```

---

## 2. Mocking Key Components

During development, you may not have actual external dependencies. Here is how to mock these components:

### A. Mock LLM Provider
If you want to avoid consuming API quotas or test how the Agent reacts to specific LLM outputs, use a `MockProvider`.

```javascript
// plugins/mock-provider/index.js
class MockLLMProvider {
  constructor(config) {
    this.responses = config.cannedResponses || {};
  }

  async generate(prompt, context) {
    console.log(`[MockLLM] Received Prompt: ${prompt.substring(0, 50)}...`);
    
    // Simple keyword matching for responses
    if (prompt.includes('list files')) {
      return { 
        content: "I will list the files for you.", 
        tool_calls: [{ name: 'fs:list_dir', args: { path: '.' } }]
      };
    }
    
    return { content: "This is an automated reply from the Mock LLM." };
  }
}
```

### B. Console UI Plugin (`stdio-interface`)
To converse directly in the terminal, you need a simple adapter plugin.

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
    // Listen for core messages
    this.core.on('onNewMessage', (msg) => {
      console.log(`
🤖 Agent: ${msg.content}`);
      this.prompt();
    });

    this.core.on('onToolCallRequest', (req) => {
      console.log(`
🛠️  Tool execution request: ${req.toolName}`);
      console.log(`   Arguments: ${JSON.stringify(req.args)}`);
      // Auto-approve by default in dev mode, or ask the user
      this.core.provideConfirmation(req.confirmationId, true);
    });

    this.prompt();
  }

  prompt() {
    this.rl.question('
👤 User: ', (input) => {
      this.core.submitUserInput(input);
      // Note: Do not recursively call prompt here; wait for Agent reply
    });
  }
}
```

---

## 3. Debugging Tips

1.  **Enable Verbose Logging:**
    When initializing `AgentCore`, pass in a `logger` configuration with the log level set to `DEBUG`. This allows you to see the complete flow of the "Event Queue" and all "State Transitions."

2.  **Breakpoint Debugging:**
    Run `node run_local.js` using VS Code's "JavaScript Debug Terminal." You can set breakpoints within the `execute` method of any plugin.

3.  **State Snapshot Inspection:**
    During interaction, you can call `agent.stateManager.snapshot()` at any time to view the current full Context object and verify if `short_term_memory` is correctly accumulating conversation history.

---

## 4. From Standalone to Production

Once your plugin passes testing in standalone mode:
1.  Move the plugin code to the standard plugin directory.
2.  Update your `Agent Template` JSON file (in the Design Layer) to include the new plugin name.
3.  Restart the Daemon or create a new Agent instance to use it in production.
