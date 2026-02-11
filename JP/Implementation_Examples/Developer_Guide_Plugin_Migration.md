# 開発者ガイド：プラグインロード仕様と移植 (Plugin Loading & Migration Guide)

このドキュメントでは、 OpenStarry のプラグインロード仕様を詳細に説明し、既存の Node.js モジュールやサードパーティ製プラグインを OpenStarry エコシステムへ移植するための具体的な手順を提供します。

## 1. コア・ロード仕様 (Loading Specification)

OpenStarry は、プラグインのロードに **「ファクトリパターン (Factory Pattern)」** と **「依存性の注入 (Dependency Injection)」** を採用しています。

### 1.1 物理構造
プラグインは標準的な NPM パッケージである必要があり、その `package.json` には **必ず** `openstarry` メタデータフィールドが含まれていなければなりません。

```json
{
  "name": "@openstarry-plugin/my-plugin",
  "main": "dist/index.js",
  "dependencies": {
    "@openstarry/sdk": "^1.0.0"
  },
  "openstarry": {
    "type": "aggregate", // または 'tool', 'provider'
    "components": {
      "providers": ["MyProvider"],
      "tools": ["LoginTool"]
    }
  }
}
```

### 1.2 エントリポイント (Entry Point)
`main` が指すファイルは、 `initialize` 関数をエクスポートするか、 `IPlugin` インターフェースを実装したクラスを提供する必要があります。

```typescript
import { IPlugin, IPluginContext } from '@openstarry/sdk';

export default class MyPlugin implements IPlugin {
  async initialize(context: IPluginContext) {
    // ここで初期化と登録を行います
  }
}
```

---

## 2. 移植ガイド (Migration Guide)

既存の機能モジュール（例えば他の AI フレームワークのプロバイダーなど）がある場合は、以下の手順に従って移植を行ってください。

### ステップ 1：依存関係の置き換え
元々依存していたフレームワーク（ `@ai-core/core` や `langchain` など）を削除し、 `@openstarry/sdk` に置き換えます。
*   **Context:** 元のコンテキストオブジェクトの代わりに `IAgentContext` を使用します。
*   **Logger:** `console.log` の代わりに `context.logger` を使用します。

### ステップ 2：インターフェースの適応 (Adapter Pattern)
OpenStarry のインターフェースは元のフレームワークと異なる場合があります。アダプター (Adapter) クラスを作成する必要があります。

*   **Provider:** `IProvider` を実装します。 OpenStarry の `generate(prompt)` を元のモジュールが必要とする形式（ `chat(messages)` など）に変換します。
*   **Tool:** `ITool` を実装します。 `execute(args)` を元の関数にマッピングします。

### ステップ 3：コマンドからツールへの変換
元のモジュールが CLI コマンド（ `/login` など）を登録していた場合は、それを **`ITool`** に変換してください。
*   これにより、 CLI から使用できるだけでなく、エージェントが思考プロセスの中で「ログインが必要だ」と判断できるようになります。

### ステップ 4：入力のプッシュ (コンストラクタのコールバックに代わる pushInput)
旧バージョンでは、一部のプラグインがコンストラクタのコールバック (constructor callback) を介してユーザー入力をエージェントに注入していました。現在は `IPluginContext.pushInput` メソッドを使用し、プラグインから能動的にエージェントへ入力メッセージをプッシュするように変更してください：
```typescript
// ❌ 旧方式：コンストラクタのコールバック経由
constructor(onInput: (msg: string) => void) { ... }

// ✅ 新方式：context.pushInput 経由
context.pushInput({ role: 'user', content: userMessage });
```

### ステップ 5：設定の注入
`process.env` から設定を読み込まないでください。 `context.config` から読み込むようにしてください。
```typescript
// ❌ 旧コード
const key = process.env.API_KEY;

// ✅ OpenStarry コード
const key = context.config.apiKey;
```

---

## 3. 例： OAuth プロバイダーの移植

`GeminiOAuthManager` クラスがあると仮定します。

1.  **プラグインとしてのカプセル化：** `IPlugin` を実装した `GeminiOAuthPlugin` を作成します。
2.  **初期化：** `initialize()` 内で `GeminiOAuthManager` をインスタンス化します。
3.  **プロバイダーの登録：** `GeminiProvider` クラスを作成し、内部でマネージャーの API を呼び出し、 `context.registerProvider()` を介して登録します。
4.  **ツールの登録：** `LoginTool` を作成し、内部で `manager.startOAuthFlow()` を呼び出し、 `context.registerTool()` を介して登録します。

これで、外部モジュールが OpenStarry の五蘊アーキテクチャに完璧に統合されました。
