# 開発者ガイド：単機実行とデバッグ (Standalone Execution)

このガイドは、開発者が完全な `Orchestrator Daemon` や複雑なインフラを起動することなく、軽量な `Agent Core` インスタンスを迅速に起動して、プラグインの開発、デバッグ、およびテストを行うためのものです。

## なぜ単機モードが必要なのか？

完全な本番環境では、エージェントはデーモンによって管理され、設定は設計層から提供され、ステートはデータベースに保存されます。これは日常的な開発には重すぎます。
**単機モード (Standalone Mode)** では、以下のことが可能になります：
1.  ローカルの JSON 設定ファイルを使用してエージェントを直接初期化する。
2.  データベースの代わりにメモリ (In-Memory) ストレージを使用する。
3.  Web UI なしで、コンソール (Console) でエージェントと直接対話する。
4.  新しく作成した Tool や Listener プラグインを迅速にテストする。

---

## 1. 最小限の起動スクリプト (`run_local.js`)

エージェントをインスタンス化して実行するためのシンプルな Node.js スクリプトを作成できます。

```javascript
// run_local.js
const { AgentCore } = require('./src/core/AgentCore');
const { PluginLoader } = require('./src/infrastructure/PluginLoader');
const { InMemoryStateManager } = require('./src/core/state/InMemoryStateManager');

async function main() {
  console.log("🚀 OpenStarry 単機開発モードを起動しています...");

  // 1. プラグインローダーの初期化
  // ローカルのプラグインディレクトリを指すようにします
  const pluginLoader = new PluginLoader({
    pluginDirs: ['./plugins/my-new-tools', './plugins/standard-library']
  });

  // 2. 設定の準備 (本来は Daemon/Design Layer から提供されるもの)
  const agentConfig = {
    id: 'local-dev-agent-001',
    name: 'Local Developer Agent',
    role: 'assistant',
    llmProvider: {
      provider: 'gemini', // または Token を消費しないテスト用の 'mock'
      model: 'gemini-pro',
      apiKey: process.env.GEMINI_API_KEY
    },
    systemPrompt: "あなたは単機デバッグ助手です。ユーザーの質問に簡潔に答えてください。",
    plugins: [
      '@openstarry-plugin/stdio-interface', // ターミナルで入出力を行うための UI プラグイン
      '@openstarry-plugin/file-system-tools',
      '@openstarry-plugin/my-custom-plugin' // 開発中のプラグイン
    ]
  };

  // 3. コアのインスタンス化
  // メモリ内ステートマネージャーを使用。再起動するとデータは失われますが、テストには適しています。
  const stateManager = new InMemoryStateManager();
  const agent = new AgentCore(agentConfig, pluginLoader, stateManager);

  // 4. エージェントの起動
  await agent.start();
  console.log("✅ エージェントの準備が整いました！以下にコマンドを入力してください：");
}

main().catch(err => {
  console.error("❌ 起動に失敗しました:", err);
});
```

---

## 2. 主要コンポーネントの Mock テクニック

開発中に実際の外部依存関係がない場合があります。それらをどのように Mock するかは以下の通りです。

### A. Mock LLM Provider
API のクォータを消費したくない場合や、特定の LLM 出力に対するエージェントの反応をテストしたい場合は、 `MockProvider` を使用します。

```javascript
// plugins/mock-provider/index.js
class MockLLMProvider {
  constructor(config) {
    this.responses = config.cannedResponses || {};
  }

  async generate(prompt, context) {
    console.log(`[MockLLM] Prompt を受信: ${prompt.substring(0, 50)}...`);
    
    // シンプルなキーワードマッチングによる応答
    if (prompt.includes('ファイルをリスト')) {
      return { 
        content: "ファイルをリストアップします。", 
        tool_calls: [{ name: 'fs:list_dir', args: { path: '.' } }]
      };
    }
    
    return { content: "これは Mock LLM からの自動返信です。" };
  }
}
```

### B. Console UI プラグイン (`stdio-interface`)
ターミナルで直接対話するには、シンプルなアダプタープラグインが必要です。

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
    // コアからのメッセージを監視
    this.core.on('onNewMessage', (msg) => {
      console.log(`
🤖 Agent: ${msg.content}`);
      this.prompt();
    });

    this.core.on('onToolCallRequest', (req) => {
      console.log(`
🛠️  ツールの実行リクエスト: ${req.toolName}`);
      console.log(`   引数: ${JSON.stringify(req.args)}`);
      // 開発モードではデフォルトで自動承認するか、ユーザーに問い合せます
      this.core.provideConfirmation(req.confirmationId, true);
    });

    this.prompt();
  }

  prompt() {
    this.rl.question('
👤 User: ', (input) => {
      this.core.submitUserInput(input);
      // 注意：ここでは再帰的に prompt を呼び出さず、エージェントの返信を待ってからトリガーします
    });
  }
}
```

---

## 3. デバッグテクニック (Debugging Tips)

1.  **詳細ログの有効化 (Verbose Logging):**
    `AgentCore` の初期化時に `logger` 設定を渡し、ログレベルを `DEBUG` に設定します。これにより、「イベントキュー」の流れと「ステート遷移」の全容を確認できます。

2.  **ブレークポイントデバッグ:**
    VS Code の "JavaScript Debug Terminal" を使用して `node run_local.js` を実行すると、任意のプラグインの `execute` メソッド内にブレークポイントを設定できます。

3.  **ステートスナップショットの検査:**
    対話の途中で、いつでも `agent.stateManager.snapshot()` を呼び出して現在の完全な Context オブジェクトを表示し、 `short_term_memory` に正しく対話履歴が蓄積されているかを確認できます。

---

## 4. 単機から本番へ

単機モードでプラグインのテストが合格したら：
1.  プラグインのコードを標準のプラグインディレクトリに移動します。
2.  （設計層にある） `Agent Template` JSON ファイルを更新し、新しいプラグイン名を追加します。
3.  デーモンを再起動するか、新しいエージェントインスタンスを作成すれば、本番環境で使用可能になります。
