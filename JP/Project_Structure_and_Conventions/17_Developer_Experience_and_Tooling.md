# 17. 開発者体験とツールチェーン (Developer Experience and Tooling)

開発者が Agent とプラグインを迅速にビルド、テスト、共有できるよう、包括的な CLI ツールチェーンと開発環境を提供しています。

## 1. OpenStarry CLI (`openstarry`)

`openstarry` は、ユーザーと開発者の唯一のエントリーポイントです。基本コマンドの定義については **[09. CLI 設計と管理コマンド](./09_CLI_Design_and_Management_Commands.md)** を参照してください。

### よく使う開発コマンド (Alias / Extensions)
*   `openstarry init <name>`: 新しい Agent プロジェクトを作成します（`agent.json`, `config/`, `.env` を含む）。
*   `openstarry create-plugin <name>`: テンプレートを使用して新しいプラグインプロジェクトを作成します。
*   `openstarry dev`: ローカル開発モードを起動します (Hot Reload)。
*   `openstarry plugin add`: 依存プラグインをインストールします。
*   `openstarry start`: Agent を起動します。

## 2. プラグインスキャフォールディング (Scaffolding)

`openstarry create-plugin my-tool` を実行すると、以下の標準構造が生成されます：

```text
my-tool/
├── src/
│   ├── index.ts        # エントリーポイント
│   └── tools.ts        # ツールロジック
├── tests/
│   └── index.test.ts   # 事前設定済みのユニットテスト
├── package.json        # 依存関係の設定
├── tsconfig.json       # TypeScript 設定
└── README.md           # ドキュメントテンプレート
```

**テンプレートの特徴：**
*   **Pre-configured Build:** `tsup` または `rollup` の設定が組み込まれており、ワンコマンドでバンドルできます。
*   **Type Safety:** `@openstarry/sdk` の型定義が自動的にインポートされます。
*   **Linting:** ESLint/Prettier ルールが事前設定されています。

## 3. MockHost: ラボ環境 (The Lab)

プラグイン開発時に、完全な Agent（LLM 接続を含む）を起動するのはコストが高く、時間もかかります。そのため、`MockHost` テスト環境を提供しています。

### 機能
*   **Core の動作をシミュレート:** テストスクリプト内で Core が発行するコマンド（例：`callTool`）をシミュレートできます。
*   **仮想コンテキスト:** 実際の LLM は不要で、モックの Context データを直接注入できます。
*   **高速フィードバック:** Watch モードをサポートしており、コード変更後すぐにテストが再実行されます。

### 使用例（プラグインテスト内）

```typescript
import { TestAgentHost } from '@openstarry/sdk/testing';
import { MyPlugin } from '../src';

test('MyPlugin loads correctly', async () => {
  const host = new TestAgentHost();
  const plugin = new MyPlugin();

  await host.load(plugin);

  // Core がツールを呼び出すことをシミュレート
  const result = await host.executeTool('my_tool_function', { arg: 'value' });

  expect(result).toBe('success');
});
```

## 4. デバッグと可視化 (Debugging)

*   **Inspector:** `openstarry dev` の起動時にローカルの WebSocket ポートが開かれます。開発者は `OpenStarry DevTools`（Web UI）を使用して接続し、Agent の思考プロセス、Context の変化、ツール呼び出しログをリアルタイムで確認できます。
*   **VS Code Extension:** （将来計画）エディター内で Agent DSL のシンタックスハイライトを直接提供し、プラグインコードの補完機能も提供します。
