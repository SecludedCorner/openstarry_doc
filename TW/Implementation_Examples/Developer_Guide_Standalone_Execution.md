# 開發者指南：單機運行與調試 (Standalone Execution)

本指南旨在幫助開發者在不啟動完整 `Orchestrator Daemon` 和複雜基礎設施的情況下，快速啟動一個輕量級的 `Agent Core` 實例進行插件開發、調試和測試。

## 為什麼需要單機模式？

在完整的生產環境中，Agent 由 Daemon 管理，配置由設計層提供，狀態存儲在資料庫中。這對於日常開發來說太過厚重。
**單機模式 (Standalone Mode)** 允許你：
1.  使用本地的 JSON 配置文件直接初始化 Agent。
2.  使用內存 (In-Memory) 存儲替代資料庫。
3.  在控制台 (Console) 直接與 Agent 交互，無需 Web UI。
4.  快速測試新編寫的 Tool 或 Listener 插件。

---

## 1. 最小化啟動腳本 (`run_local.js`)

你可以創建一個簡單的 Node.js 腳本來實例化和運行 Agent。

```javascript
// run_local.js
const { AgentCore } = require('./src/core/AgentCore');
const { PluginLoader } = require('./src/infrastructure/PluginLoader');
const { InMemoryStateManager } = require('./src/core/state/InMemoryStateManager');

async function main() {
  console.log("🚀 正在啟動 OpenStarry 單機開發模式...");

  // 1. 初始化插件加載器
  // 指向你的本地插件目錄
  const pluginLoader = new PluginLoader({
    pluginDirs: ['./plugins/my-new-tools', './plugins/standard-library']
  });

  // 2. 準備配置 (原本由 Daemon/Design Layer 提供)
  const agentConfig = {
    id: 'local-dev-agent-001',
    name: 'Local Developer Agent',
    role: 'assistant',
    llmProvider: {
      provider: 'gemini', // 或 'mock' 用於不消耗 Token 的測試
      model: 'gemini-pro',
      apiKey: process.env.GEMINI_API_KEY
    },
    systemPrompt: "你是一個單機調試助手。請簡潔地回答用戶的問題。",
    plugins: [
      '@openstarry-plugin/stdio-interface', // 一個用於在終端輸入輸出的 UI 插件
      '@openstarry-plugin/file-system-tools',
      '@openstarry-plugin/my-custom-plugin' // 你正在開發的插件
    ]
  };

  // 3. 實例化核心
  // 使用內存狀態管理器，重啟後數據丟失，適合測試
  const stateManager = new InMemoryStateManager();
  const agent = new AgentCore(agentConfig, pluginLoader, stateManager);

  // 4. 啟動代理人
  await agent.start();
  console.log("✅ 代理人已就緒！請在下方輸入指令：");
}

main().catch(err => {
  console.error("❌ 啟動失敗:", err);
});
```

---

## 2. 關鍵組件 Mock 技巧

在開發過程中，你可能沒有實際的外部依賴。以下是如何 Mock 這些組件：

### A. Mock LLM Provider
如果你不想消耗 API Quota，或者想測試 Agent 對特定 LLM 輸出的反應，可以使用 `MockProvider`。

```javascript
// plugins/mock-provider/index.js
class MockLLMProvider {
  constructor(config) {
    this.responses = config.cannedResponses || {};
  }

  async generate(prompt, context) {
    console.log(`[MockLLM] 收到 Prompt: ${prompt.substring(0, 50)}...`);
    
    // 簡單的關鍵字匹配回應
    if (prompt.includes('列出文件')) {
      return { 
        content: "我將為您列出文件。", 
        tool_calls: [{ name: 'fs:list_dir', args: { path: '.' } }]
      };
    }
    
    return { content: "這是 Mock LLM 的自動回覆。" };
  }
}
```

### B. Console UI 插件 (`stdio-interface`)
為了在終端機直接對話，你需要一個簡單的適配器插件。

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
    // 監聽核心消息
    this.core.on('onNewMessage', (msg) => {
      console.log(`\n🤖 Agent: ${msg.content}`);
      this.prompt();
    });

    this.core.on('onToolCallRequest', (req) => {
      console.log(`\n🛠️  請求執行工具: ${req.toolName}`);
      console.log(`   參數: ${JSON.stringify(req.args)}`);
      // 開發模式默認自動批准，或詢問用戶
      this.core.provideConfirmation(req.confirmationId, true);
    });

    this.prompt();
  }

  prompt() {
    this.rl.question('\n👤 User: ', (input) => {
      this.core.submitUserInput(input);
      // 注意：這裡不遞歸調用 prompt，而是等待 Agent 回覆後觸發
    });
  }
}
```

---

## 3. 調試技巧 (Debugging Tips)

1.  **啟用詳細日誌 (Verbose Logging):**
    在 `AgentCore` 初始化時，傳入 `logger` 配置，將日誌級別設為 `DEBUG`，這樣可以看到完整的「事件隊列」流動和「狀態變更」。

2.  **斷點調試:**
    使用 VS Code 的 "JavaScript Debug Terminal" 運行 `node run_local.js`，你可以在任何插件的 `execute` 方法中設置斷點。

3.  **狀態快照檢查:**
    在交互過程中，你可以隨時調用 `agent.stateManager.snapshot()` 來查看當前的完整 Context 對象，檢查 `short_term_memory` 是否正確累積了對話歷史。

---

## 4. 從單機到生產

當你的插件在單機模式測試通過後：
1.  將插件代碼移動到標準的插件目錄。
2.  更新你的 `Agent Template` JSON 文件（在設計層），加入新插件的名稱。
3.  重啟 Daemon 或創建新的 Agent 實例，即可在生產環境中使用。

```
