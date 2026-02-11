# 18. プラグインのロードと登録プロトコル (Plugin Loading Protocol)

このドキュメントでは、OpenStarry システムがディスクからプラグインコードをロードし、それをコアシステムと接続するための詳細なプロトコルを定義します。これは Phase 3 の中心となる技術仕様です。

## 1. 核心的な課題

プラグインは「五蘊の集合体」であり、コア内部の動作はモジュール（ToolManager, ListenerManager など）に分かれているため、ローダーは以下の問題を解決する必要があります。
1.  **動的性質：** プラグインコードはコンパイル時には不明である。
2.  **解体 (Destructuring)：** 1つのプラグインパッケージを複数の機能ユニットに分解する。
3.  **依存性の注入 (Dependency Injection)：** プラグインがコアの実装に依存することなく、コアの能力（Logger, Config など）を取得できるようにする。

## 2. ロードプロトコル：ファクトリパターン (Factory Pattern)

柔軟性を最大化するため、私たちは「クラススキャン」モードを廃止し、 **ファクトリ関数モード** を採用します。

> **重要な更新：** 従来の `BUILTIN_FACTORIES` ハードコードマップは削除されました。すべてのプラグイン（組み込みプラグインを含む）は、統合された動的ロードメカニズムを介してロードされます。まず `ref.path` （ローカルパス）が優先され、それがない場合は `import(ref.name)` （NPM パッケージ名）が使用されます。これにより、プラグインのロードロジックの整合性が確保され、組み込みプラグインと外部プラグインの区別がなくなりました。

### プラグインエントリの仕様 (`index.ts`)

各プラグインは、 `initialize` という名前の非同期関数をエクスポートするか、 `IPluginFactory` インターフェースを実装したデフォルトエクスポートを提供する必要があります。

```typescript
import { IPluginContext, IPlugin } from '@openstarry/sdk';
import { MyTool } from './tools/my-tool';
import { MyListener } from './listeners/my-listener';

// プラグインファクトリ関数
export default async function initialize(context: IPluginContext): Promise<void> {
  // 1. コアから注入された依存関係を取得する
  const logger = context.logger;
  logger.info('Initializing My Plugin...');

  // 2. 行蘊 (Tools) の登録
  // ここでプラグインは自身の Tool インスタンスを能動的にコアに渡す
  context.registerTool(new MyTool());

  // 3. 受蘊 (Listeners) の登録
  context.registerListener(new MyListener(context.config));
  
  // 4. 識蘊 (Guide) の登録
  context.registerGuide({
    systemPrompt: "...",
    memoryPolicy: "sliding-window"
  });
}
```

## 3. ホスト側のロードプロセス (`PluginLoader` Logic)

`PluginLoader` は、 **調整レイヤー (Coordination Layer / Daemon)** または **ホストプロセス (Host Process)** 内で動作します。

### ステップ A: 物理的な読み取り
1.  `~/.openstarry/plugins/<plugin-id>/package.json` を読み取ります。
2.  `main` フィールドを解析して、エントリファイル (例: `dist/index.js`) を見つけます。
3.  ESM の `import()` または CJS の `require()` を使用して、そのモジュールを動的にロードします。

### ステップ B: コンテキストの構築 (Context Construction)
ローダーは、そのプラグイン専用の `PluginContext` インスタンスを作成します。このコンテキストは、プラグインが外部と通信するための唯一のチャネルです。

```typescript
const context: IPluginContext = {
  logger: new ScopedLogger(`Plugin:${pluginId}`),
  config: agentConfig.plugins[pluginId] || {}, // agent.json から設定を読み取る
  
  // 登録コールバック (Callback)
  registerTool: (tool) => core.toolRegistry.add(tool),
  registerListener: (listener) => core.listenerRegistry.add(listener),
  registerProvider: (provider) => core.providerRegistry.add(provider),
  registerGuide: (guide) => core.guideRegistry.set(guide),

  // 入力の注入 — プラグインは pushInput を介してエージェントに能動的にメッセージをプッシュできる
  pushInput: (input) => core.inputQueue.push(input),
};
```

### ステップ C: 初期化と解体
プラグインの `initialize(context)` を呼び出します。
*   このとき、プラグイン内部のロジックが実行を開始し、 `register*` メソッドを介して自身の「五蘊」の成分を「引き渡し」ます。
*   コアはこれらの成分を受け取ると、それぞれ対応するマネージャーにアーカイブ（保存）します。

## 4. エラーの隔離とサンドボックス

*   **ロード失敗：** `initialize` が例外をスローした場合、ローダーはそれを「痛覚」としてキャッチして記録する必要がありますが、エージェント全体をクラッシュさせてはいけません。そのプラグインは `FAILED` としてマークされます。
*   **依存性注入のセキュリティ：** `IPluginContext` は安全な API のみを公開します。プラグインは、コアのプライベート属性やホストの `process` オブジェクトに直接アクセスすることはできません（レベル 0 の信頼モードでない限り）。

## 5. 調整レイヤーへの登録

正常にロードされたすべてのプラグインのメタデータ（ID, バージョン, 能力）は、 **エージェント調整レイヤー (Agent Coordination Layer)** に報告されます。これにより、調整レイヤーは「天気予報機能を持っているのは誰か？」といった質問に答えることができ、エージェント間での動的なルーティングを実現します。

---

## 6. 依存関係の解決とトポロジカル・ロード (Dependency Resolution & Topological Loading)

ロード順序の誤りによるシステムのクラッシュ（例：Workflow プラグインが Skill プラグインより前にロードされ、Markdown パーサーが取得できないなど）を防ぐため、ローダーは **必ず** 依存関係グラフの解決アルゴリズムを実装しなければなりません。

### 2段階のロードプロセス (Two-Phase Loading)

#### フェーズ A: スキャンとグラフ作成 (Scan & Graph)
1.  ローダーはすべてのターゲットプラグインディレクトリをスキャンします。
2.  各プラグインの `package.json` にある `openstarry.dependencies` フィールドを読み取ります。
    ```json
    "dependencies": {
      "plugins": ["@openstarry-plugin/skill"] // 他のプラグインへのハード依存を宣言
    }
    ```
3.  メモリ内に **依存関係の有向グラフ (Dependency Directed Graph)** を構築します。
    *   ノード：Plugin ID
    *   辺：依存先 (Dependency) -> 依存元 (Dependent)

#### フェーズ B: ソートと実行 (Sort & Execute)
1.  **循環参照の検知 (Cycle Detection):** グラフ内に循環依存 (A -> B -> A) が存在しないかチェックします。発見した場合は、直ちに `FatalDependencyError` をスローして起動を中止します。
2.  **トポロジカルソート (Topological Sort):** Kahn アルゴリズムまたは DFS を使用して、線形なロード順序を計算します。
    *   *結果例:* `['@openstarry-plugin/skill', '@openstarry-plugin/gemini', '@openstarry-plugin/workflow']`
3.  **順次初期化:** ソートされたリストに従って、順番に `initialize(context)` を呼び出します。
    *   これにより、 `workflow` が初期化されるときには、 `skill` の準備が整っており、パーサーがシステムに注入されていることが保証されます。

### 依存性注入のタイミング
プラグインをまたぐサービス（ `markdownParser` など）は、 **フェーズ B** のソート過程で動的に注入される必要があります。
*   `@openstarry-plugin/skill` の初期化が完了すると、そのサービスインスタンスを返し（または登録し）ます。
*   ローダーはこれらのインスタンスを一時保存します。
*   `@openstarry-plugin/workflow` の初期化の番が来ると、ローダーは一時保存場所から `markdownParser` を取り出し、 `context.dependencies` に入れます。
*   `@openstarry-plugin/workflow` の初期化の番が来ると、ローダーは一時保存場所から `markdownParser` を取り出し、 `context.dependencies` に入れます。
