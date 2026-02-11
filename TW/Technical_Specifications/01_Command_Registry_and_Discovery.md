# 技術規格書 01: 指令註冊、發現與依賴注入機制

> **狀態**: Draft
> **適用版本**: OpenStarry v0.1.0+
> **相關架構文檔**: 08_Command_And_Tool_Design, 19_Agent_Coordination_Layer

本規格書旨在具體定義 Plugin 指令 (`/command`) 的註冊流程、檔案結構以及 Agent 啟動時的依賴注入實作細節，以消除架構設計與程式碼實作之間的誤差。

## 1. 檔案系統結構 (The System Folder)

協調層 (Coordinator) 維護的「系統資料夾」與「Registry」必須遵循以下物理結構：

```text
~/.openstarry/
├── config.toml                 # Daemon 主設定
├── registry.json               # [自動生成] 全域指令與插件索引檔 (The Source of Truth)
└── plugins/                    # 插件安裝目錄
    ├── @openstarry-plugin/provider-gemini/
    │   ├── package.json
    │   ├── plugin.json         # [關鍵] 定義了該插件提供哪些指令
    │   └── dist/index.js
    └── @openstarry-plugin/provider-claude/
        ├── ...
```

### 1.1 Registry 索引檔 (`registry.json`)
這是 CLI 與協調層查詢「有哪些指令可用」的快取檔案。當 Plugin 變動時，協調層會更新此檔。

```json
{
  "last_updated": "2026-02-04T12:00:00Z",
  "commands": {
    "provider": {
      "description": "Manage AI Providers",
      "subcommands": {
        "gemini": {
          "plugin_id": "@openstarry-plugin/provider-gemini",
          "handler": "handleCliCommand", 
          "subcommands": {
             "login": { "description": "Authenticate Gemini" }
          }
        },
        "claude": {
          "plugin_id": "@openstarry-plugin/provider-claude",
          "handler": "handleCliCommand",
          "subcommands": {
             "login": { "description": "Authenticate Claude" }
          }
        }
      }
    }
  },
  "capabilities": {
    "llm:gemini": "@openstarry-plugin/provider-gemini",
    "llm:claude": "@openstarry-plugin/provider-claude"
  }
}
```

---

## 2. Plugin 定義規範 (`plugin.json`)

Plugin 開發者必須在 `plugin.json` 中顯式宣告它提供的 CLI 指令。這實現了「Plugin 動態註冊」的需求。

**範例：Gemini Provider 的 `plugin.json`**

```json
{
  "id": "@openstarry-plugin/provider-gemini",
  "version": "1.0.0",
  "type": "provider",
  "capabilities": ["llm:gemini"],
  "cli_commands": [
    {
      "trigger": "provider gemini", 
      "description": "Gemini Provider Management",
      "method": "handleCliCommand",
      "help_text": "Use 'login' to authenticate or 'config' to set options."
    }
  ]
}
```

*   **trigger**: 定義指令路徑，空格代表子指令層級。
*   **method**: 對應到 Plugin 入口點 class 中的具體函數名稱。

---

## 3. 實作流程：動態掃描與註冊

協調層 (`Coordinator`) 需實作以下邏輯來達成「更新系統資料夾有關指令的檔案」：

### 3.1 偽代碼邏輯 (Coordinator Daemon)

```typescript
class PluginRegistry {
  private registryPath = "~/.openstarry/registry.json";

  // 1. 掃描插件目錄
  async refreshRegistry() {
    const plugins = await scanPluginDir("~/.openstarry/plugins");
    const newRegistry = { commands: {}, capabilities: {} };

    for (const plugin of plugins) {
      const manifest = await readJson(plugin.path + "/plugin.json");
      
      // 2. 註冊 CLI 指令
      if (manifest.cli_commands) {
        for (const cmd of manifest.cli_commands) {
          // 將 "provider gemini" 解析為巢狀物件結構
          this.mergeCommandTree(newRegistry.commands, cmd.trigger, {
            plugin_id: manifest.id,
            handler: cmd.method,
            description: cmd.description
          });
        }
      }

      // 3. 註冊能力 (用於 Agent 注入)
      if (manifest.capabilities) {
        for (const cap of manifest.capabilities) {
          newRegistry.capabilities[cap] = manifest.id;
        }
      }
    }

    // 4. 更新系統檔案
    await writeJson(this.registryPath, newRegistry);
  }
}
```

---

## 4. Agent 產生與依賴注入 (Dependency Injection)

這部分解決「針對不同 Agent 配置不同 Provider」的需求。

### 4.1 請求結構 (Request)
當 CLI 或 API 請求創建 Agent 時：

```json
// POST /agents/spawn
{
  "name": "my-coding-bot",
  "config": {
    "provider": "gemini",  // 指定使用 gemini
    "model": "gemini-1.5-pro",
    "tools": ["fs", "terminal"]
  }
}
```

### 4.2 協調層處理邏輯 (Resolution)

協調層收到請求後，必須進行「解析 (Resolution)」：

1.  讀取 `config.provider` 值為 `"gemini"`。
2.  查詢 `registry.json` 或記憶體索引，找到提供 `llm:gemini` 能力的 Plugin ID -> `@openstarry-plugin/provider-gemini`。
3.  載入該 Plugin。
4.  實例化 Agent Runtime，並將 Provider 實例注入。

```typescript
// Coordinator Code
async function spawnAgent(request) {
  // 1. 解析 Provider Plugin
  const providerPluginId = registry.capabilities[`llm:${request.config.provider}`];
  if (!providerPluginId) throw new Error("Unknown provider");

  const plugin = await loadPlugin(providerPluginId);
  
  // 2. 獲取 Provider 實例 (Factory Pattern)
  const providerInstance = await plugin.createProvider({
    model: request.config.model
    // 這裡會自動讀取先前透過 /provider gemini login 存下的 token
  });

  // 3. 注入 Agent
  const agent = new AgentCore({
    name: request.name,
    provider: providerInstance, // <--- 注入點
    tools: request.config.tools
  });

  await agent.start();
}
```

## 5. 總結：開發者實作清單

為了符合此規格，你需要實作：

1.  **Registry Manager**: 在 `packages/core` 或 `daemon` 中實作掃描 `plugins/` 並生成 `registry.json` 的邏輯。
2.  **CLI Router**: CLI (`apps/runner`) 啟動時讀取 `registry.json`，並動態註冊 `commander` 指令。當使用者輸入 `/provider gemini login` 時，CLI 透過 IPC 呼叫 Daemon，Daemon 再轉發給 Plugin 的 `handleCliCommand`。
3.  **Plugin Interface**: 確保所有 Provider Plugin 都實作標準介面 (如 `IProviderPlugin`)。
