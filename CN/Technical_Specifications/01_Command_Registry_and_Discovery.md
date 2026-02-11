# 技术规格书 01: 指令注册、发现与依赖注入机制

> **状态**: Draft
> **适用版本**: OpenStarry v0.1.0+
> **相关架构文档**: 08_Command_And_Tool_Design, 19_Agent_Coordination_Layer

本规格书旨在具体定义 Plugin 指令 (`/command`) 的注册流程、文件结构以及 Agent 启动时的依赖注入实现细节，以消除架构设计与代码实现之间的误差。

## 1. 文件系统结构 (The System Folder)

协调层 (Coordinator) 维护的「系统文件夹」与「Registry」必须遵循以下物理结构：

```text
~/.openstarry/
├── config.toml                 # Daemon 主设定
├── registry.json               # [自动生成] 全局指令与插件索引文件 (The Source of Truth)
└── plugins/                    # 插件安装目录
    ├── @openstarry-plugin/provider-gemini/
    │   ├── package.json
    │   ├── plugin.json         # [关键] 定义了该插件提供哪些指令
    │   └── dist/index.js
    └── @openstarry-plugin/provider-claude/
        ├── ...
```

### 1.1 Registry 索引文件 (`registry.json`)
这是 CLI 与协调层查询「有哪些指令可用」的缓存文件。当 Plugin 变动时，协调层会更新此文件。

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

## 2. Plugin 定义规范 (`plugin.json`)

Plugin 开发者必须在 `plugin.json` 中显式声明它提供的 CLI 指令。这实现了「Plugin 动态注册」的需求。

**范例：Gemini Provider 的 `plugin.json`**

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

*   **trigger**: 定义指令路径，空格代表子指令层级。
*   **method**: 对应到 Plugin 入口点 class 中的具体函数名称。

---

## 3. 实现流程：动态扫描与注册

协调层 (`Coordinator`) 需实现以下逻辑来达成「更新系统文件夹有关指令的文件」：

### 3.1 伪代码逻辑 (Coordinator Daemon)

```typescript
class PluginRegistry {
  private registryPath = "~/.openstarry/registry.json";

  // 1. 扫描插件目录
  async refreshRegistry() {
    const plugins = await scanPluginDir("~/.openstarry/plugins");
    const newRegistry = { commands: {}, capabilities: {} };

    for (const plugin of plugins) {
      const manifest = await readJson(plugin.path + "/plugin.json");

      // 2. 注册 CLI 指令
      if (manifest.cli_commands) {
        for (const cmd of manifest.cli_commands) {
          // 将 "provider gemini" 解析为嵌套对象结构
          this.mergeCommandTree(newRegistry.commands, cmd.trigger, {
            plugin_id: manifest.id,
            handler: cmd.method,
            description: cmd.description
          });
        }
      }

      // 3. 注册能力 (用于 Agent 注入)
      if (manifest.capabilities) {
        for (const cap of manifest.capabilities) {
          newRegistry.capabilities[cap] = manifest.id;
        }
      }
    }

    // 4. 更新系统文件
    await writeJson(this.registryPath, newRegistry);
  }
}
```

---

## 4. Agent 产生与依赖注入 (Dependency Injection)

这部分解决「针对不同 Agent 配置不同 Provider」的需求。

### 4.1 请求结构 (Request)
当 CLI 或 API 请求创建 Agent 时：

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

### 4.2 协调层处理逻辑 (Resolution)

协调层收到请求后，必须进行「解析 (Resolution)」：

1.  读取 `config.provider` 值为 `"gemini"`。
2.  查询 `registry.json` 或内存索引，找到提供 `llm:gemini` 能力的 Plugin ID -> `@openstarry-plugin/provider-gemini`。
3.  载入该 Plugin。
4.  实例化 Agent Runtime，并将 Provider 实例注入。

```typescript
// Coordinator Code
async function spawnAgent(request) {
  // 1. 解析 Provider Plugin
  const providerPluginId = registry.capabilities[`llm:${request.config.provider}`];
  if (!providerPluginId) throw new Error("Unknown provider");

  const plugin = await loadPlugin(providerPluginId);

  // 2. 获取 Provider 实例 (Factory Pattern)
  const providerInstance = await plugin.createProvider({
    model: request.config.model
    // 这里会自动读取先前通过 /provider gemini login 存下的 token
  });

  // 3. 注入 Agent
  const agent = new AgentCore({
    name: request.name,
    provider: providerInstance, // <--- 注入点
    tools: request.config.tools
  });

  await agent.start();
}
```

## 5. 总结：开发者实现清单

为了符合此规格，你需要实现：

1.  **Registry Manager**: 在 `packages/core` 或 `daemon` 中实现扫描 `plugins/` 并生成 `registry.json` 的逻辑。
2.  **CLI Router**: CLI (`apps/runner`) 启动时读取 `registry.json`，并动态注册 `commander` 指令。当用户输入 `/provider gemini login` 时，CLI 通过 IPC 调用 Daemon，Daemon 再转发给 Plugin 的 `handleCliCommand`。
3.  **Plugin Interface**: 确保所有 Provider Plugin 都实现标准接口 (如 `IProviderPlugin`)。
