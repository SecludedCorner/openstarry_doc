# 06. プラグインディレクトリ構造仕様 (Plugin Directory Conventions)

このドキュメントでは、 **"Directory as Protocol"** という核心的な設計哲学を具体化します。 OpenStarry システムがどのようにプラグインを識別、ロード、および検証するかを定義します。

## 1. プラグインの物理形態

標準的な OpenStarry プラグインは1つの **NPM パッケージ** です。 npm レジストリに公開することも、 `plugins/` ディレクトリ配下のフォルダとして直接存在させることもできます。

### 標準的なディレクトリレイアウト

```text
my-awesome-plugin/
├── src/
│   ├── index.ts            # エントリファイル (プラグインクラスをエクスポートする必要がある)
│   ├── tools/              # ツールロジック (存在する場合)
│   └── listeners/          # リスナーロジック (存在する場合)
├── tests/                  # ユニットテスト
├── package.json            # NPM 設定 (依存関係とメタデータを定義)
├── plugin.json             # [オプション] OpenStarry 専用メタデータ (または package.json に統合)
├── README.md               # 説明ドキュメント
└── tsconfig.json           # TypeScript 設定
```

## 2. プラグインエントリ (Entry Point)

システムローダーは、 `package.json` の `main` フィールドが指すファイルを探します。そのファイルは、 `IPlugin` インターフェースを実装したクラスを、デフォルトエクスポート (Default Export) または名前付きエクスポート (Named Export) として **必ず** エクスポートしていなければなりません。

```typescript
// src/index.ts
import { IPlugin, IPluginContext } from '@openstarry/sdk';

export default class MyAwesomePlugin implements IPlugin {
  id = 'my-awesome-plugin';
  name = 'My Awesome Plugin';
  
  async initialize(context: IPluginContext) {
    // 1. コアから注入された依存関係を取得
    const apiKey = context.config.apiKey; // agent.json から設定を取得
    const logger = context.logger;
    
    // 2. プラグイン間サービスを取得 (存在する場合)
    const mdParser = context.dependencies.markdownParser;

    logger.info('Initializing MyAwesomePlugin...');

    // 3. 五蘊の成分を登録
    context.registerTool(new MyTool(apiKey));
    context.registerListener(new MyListener());
  }
}
```

## 3. メタデータ仕様 (Manifest)

`Agent Design Service` がプラグインを発見および管理できるようにするために、 `package.json` に `openstarry` フィールドを追加するか、独立した `plugin.json` を提供する必要があります。

**推奨方法： `package.json` を直接拡張する**

```json
{
  "name": "@openstarry-plugin/weather",
  "version": "1.0.0",
  "main": "dist/index.js",
  "dependencies": {
    "@openstarry/sdk": "^0.1.0"
  },
  "openstarry": {
    "type": "tool", 
    "capabilities": ["weather-lookup", "forecast"],
    "permissions": {
      "network": ["api.weather.com"]
    }
  }
}
```

*   **type**: プラグインタイプ ( `tool` | `provider` | `listener` | `bundle` )。
*   **capabilities**: プラグインが提供する能力タグ ( AI による検索用)。
*   **permissions**: プラグインが要求する権限 (コアはこれに基づいて安全ポリシーを生成します)。

## 4. プラグイン分類ディレクトリ

**エコシステムリポジトリ ( `openstarry_plugin` )** においては、 **極めてシンプルなフラット構造** を採用し、すべての機能集約パッケージをリポジトリのルートに直接配置します：

```text
openstarry_plugin/
├── @openstarry-plugin/standard-function-fs/      # システムツールパッケージ
├── @openstarry-plugin/standard-function-stdio/   # コア対話パッケージ
├── @openstarry-plugin/provider-gemini-oauth/     # 認知適応パッケージ
├── @openstarry-plugin/guide-mcp/                 # MCP プロトコルパッケージ
├── @openstarry-plugin/standard-function-skill/   # スキルロードパッケージ (Markdown 解析)
└── @openstarry-plugin/experimental-weather-tool/ # 実験的プラグイン
```

> **命名規則：** すべての公式プラグインは `@openstarry-plugin/` 名前空間を使用して公開されます。 `agent.json` の `plugins` フィールドで引用する際は、必ず完全なパッケージ名（例： `@openstarry-plugin/standard-function-stdio` ）を使用しなければならず、短縮名はサポートされません。

この構造により、コアコードベース ( `openstarry` ) の絶対的な純粋性が確保され、すべての具体的な能力は外部モジュールとして存在することになります。
