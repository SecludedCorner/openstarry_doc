# 技術仕様書 01: コマンド登録、ディスカバリーおよび依存性注入メカニズム

> **ステータス**: Draft
> **対応バージョン**: OpenStarry v0.1.0+
> **関連アーキテクチャドキュメント**: 08_Command_And_Tool_Design, 19_Agent_Coordination_Layer

本仕様書は、Plugin コマンド (`/command`) の登録フロー、ファイル構造、および Agent 起動時の依存性注入の実装詳細を具体的に定義し、アーキテクチャ設計とコード実装の間の乖離を解消することを目的としています。

## 1. ファイルシステム構造 (The System Folder)

コーディネーションレイヤー (Coordinator) が管理する「システムフォルダ」と「Registry」は、以下の物理構造に従う必要があります：

```text
~/.openstarry/
├── config.toml                 # Daemon メイン設定
├── registry.json               # [自動生成] グローバルコマンドおよびプラグインインデックスファイル (The Source of Truth)
└── plugins/                    # プラグインインストールディレクトリ
    ├── @openstarry-plugin/provider-gemini/
    │   ├── package.json
    │   ├── plugin.json         # [重要] このプラグインが提供するコマンドを定義
    │   └── dist/index.js
    └── @openstarry-plugin/provider-claude/
        ├── ...
```

### 1.1 Registry インデックスファイル (`registry.json`)
これは CLI とコーディネーションレイヤーが「どのコマンドが利用可能か」を照会するためのキャッシュファイルです。Plugin が変更された際、コーディネーションレイヤーがこのファイルを更新します。

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

## 2. Plugin 定義仕様 (`plugin.json`)

Plugin 開発者は `plugin.json` で提供する CLI コマンドを明示的に宣言する必要があります。これにより「Plugin のダイナミック登録」の要件が実現されます。

**例：Gemini Provider の `plugin.json`**

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

*   **trigger**: コマンドパスを定義し、スペースはサブコマンドの階層を表します。
*   **method**: Plugin エントリポイントクラス内の具体的な関数名に対応します。

---

## 3. 実装フロー：ダイナミックスキャンと登録

コーディネーションレイヤー (`Coordinator`) は、「システムフォルダのコマンド関連ファイルを更新する」ために以下のロジックを実装する必要があります：

### 3.1 擬似コードロジック (Coordinator Daemon)

```typescript
class PluginRegistry {
  private registryPath = "~/.openstarry/registry.json";

  // 1. プラグインディレクトリをスキャン
  async refreshRegistry() {
    const plugins = await scanPluginDir("~/.openstarry/plugins");
    const newRegistry = { commands: {}, capabilities: {} };

    for (const plugin of plugins) {
      const manifest = await readJson(plugin.path + "/plugin.json");

      // 2. CLI コマンドを登録
      if (manifest.cli_commands) {
        for (const cmd of manifest.cli_commands) {
          // "provider gemini" をネストされたオブジェクト構造にパース
          this.mergeCommandTree(newRegistry.commands, cmd.trigger, {
            plugin_id: manifest.id,
            handler: cmd.method,
            description: cmd.description
          });
        }
      }

      // 3. ケイパビリティを登録 (Agent インジェクション用)
      if (manifest.capabilities) {
        for (const cap of manifest.capabilities) {
          newRegistry.capabilities[cap] = manifest.id;
        }
      }
    }

    // 4. システムファイルを更新
    await writeJson(this.registryPath, newRegistry);
  }
}
```

---

## 4. Agent の生成と依存性注入 (Dependency Injection)

このセクションでは、「異なる Agent に異なる Provider を設定する」という要件を解決します。

### 4.1 リクエスト構造 (Request)
CLI または API が Agent の作成をリクエストする場合：

```json
// POST /agents/spawn
{
  "name": "my-coding-bot",
  "config": {
    "provider": "gemini",  // gemini の使用を指定
    "model": "gemini-1.5-pro",
    "tools": ["fs", "terminal"]
  }
}
```

### 4.2 コーディネーションレイヤーの処理ロジック (Resolution)

コーディネーションレイヤーはリクエストを受信後、「解決 (Resolution)」を実行する必要があります：

1.  `config.provider` の値 `"gemini"` を読み取ります。
2.  `registry.json` またはインメモリインデックスを照会し、`llm:gemini` ケイパビリティを提供する Plugin ID -> `@openstarry-plugin/provider-gemini` を見つけます。
3.  該当 Plugin をロードします。
4.  Agent Runtime をインスタンス化し、Provider インスタンスを注入します。

```typescript
// Coordinator Code
async function spawnAgent(request) {
  // 1. Provider Plugin を解決
  const providerPluginId = registry.capabilities[`llm:${request.config.provider}`];
  if (!providerPluginId) throw new Error("Unknown provider");

  const plugin = await loadPlugin(providerPluginId);

  // 2. Provider インスタンスを取得 (Factory Pattern)
  const providerInstance = await plugin.createProvider({
    model: request.config.model
    // ここで /provider gemini login で以前保存されたトークンが自動的に読み取られます
  });

  // 3. Agent に注入
  const agent = new AgentCore({
    name: request.name,
    provider: providerInstance, // <--- インジェクションポイント
    tools: request.config.tools
  });

  await agent.start();
}
```

## 5. まとめ：開発者実装チェックリスト

本仕様に準拠するために、以下を実装する必要があります：

1.  **Registry Manager**: `packages/core` または `daemon` 内で、`plugins/` をスキャンして `registry.json` を生成するロジックを実装します。
2.  **CLI Router**: CLI (`apps/runner`) は起動時に `registry.json` を読み取り、`commander` コマンドをダイナミックに登録します。ユーザーが `/provider gemini login` を入力すると、CLI は IPC を介して Daemon を呼び出し、Daemon が Plugin の `handleCliCommand` に転送します。
3.  **Plugin Interface**: すべての Provider Plugin が標準インターフェース（例：`IProviderPlugin`）を実装していることを確認します。
