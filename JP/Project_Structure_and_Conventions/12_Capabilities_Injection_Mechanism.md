# 12. 能力発見と依存性の注入メカニズム (Capabilities Discovery & Injection)

このドキュメントでは、 OpenStarry 調整レイヤー (Coordination Layer) がどのようにプラグインをロードし、それらが提供する機能をエージェントコアの各サブシステムに注入するかを定義します。これは、「マイクロカーネル・アーキテクチャ」と「動的な拡張」を実現するための核心的なメカニズムです。

## 1. メカニズムの概要 (Overview)

私たちは、 **制御の反転 (IoC)** と **依存性の注入 (DI)** パターンを採用しています。

*   **Host (The Driver)：** エージェントを起動するホストプロセス (CLI または Daemon) です。 **ホストのみが物理的な I/O 権限を持ち** 、ディスク上のプラグインファイルを読み取ることができます。
*   **PluginLoader (The Injector)：** ホストのコンテキストで動作し、物理的なファイルをカーネルオブジェクトに変換する責任を負います。
*   **Core (The Recipient)：** 純粋な受信者であり、注入された能力を受動的に取得します。

> **💡 設計哲学 (Design Philosophy)：**
> これは、五蘊における「色不異空（しきふいくう）」に対応しています。プラグイン（色）は具体的な機能の集まりであり、コア（空）は純粋な実行容器です。ローダーの責務は、この集まりを分解し、能力を容器に充填することです。

---

## 2. 核心的なインターフェース定義 (Core Interfaces)

すべての対話は、 `@openstarry/sdk` で定義された契約に基づいています。

### 2.1 プラグインインターフェース (The Plugin Contract)

各プラグインは、このインターフェースを実装したクラスをエクスポートしなければなりません。

```typescript
export interface IPlugin {
  readonly id: string;
  readonly version: string;
  
  /**
   * プラグインを初期化する。
   * @param context - 調整レイヤーが提供する登録コンテキスト。能力を返却するために使用される。
   */
  initialize(context: IPluginContext): Promise<void>;
  
  /**
   * リソースのクリーンアップ (例： WebSocket 接続の切断)。
   */
  shutdown(): Promise<void>;
}
```

### 2.2 登録コンテキスト (The Registration Context)

これは、能力を収集するために調整レイヤーがプラグインに公開する API です。

```typescript
export interface IPluginContext {
  // インフラストラクチャ
  readonly logger: ILogger;
  readonly config: Record<string, any>; // agent.json の当該プラグインの設定セクションから取得

  // [行] ツールの登録： LLM から呼び出し可能な関数
  registerTool(tool: ITool): void;
  
  // [受] リスナーの登録：エージェントの実行をトリガーする外部イベントソース
  registerListener(listener: IListener): void;
  
  // [想] モデルの設定：エージェントの脳 (通常は排他的。後から登録されたものが上書きするか、エラーになる)
  setLLMProvider(provider: ILLMProvider): void;
}
```

---

## 3. ロードと注入のフロー (Loading Sequence)

以下は、エージェントコアの起動時における `PluginLoader` の標準操作手順です：

1.  **設定の解決と発見 (Resolve & Discovery)：** 
    *   `agent.json` 内の `plugins` ID リストを読み取ります。
    *   **グローバルレジストリの照会：** `PluginRegistryService.resolve(id)` を呼び出します。
    *   **パスの取得：** レジストリはプラグインの正確な物理パスを返します（システム/プロジェクトの優先順位は解決済み）。

2.  **動的ロード (Dynamic Import)：**
    Node.js の `import()` または `require()` を使用して、そのパスにあるエントリファイルをロードします。

3.  **インスタンス化 (Instantiate)：**
    `IPlugin` のインスタンスを作成します。

4.  **初期化と注入 (Initialize & Inject)：**
    *   ローダーは、現在のコアインスタンス (ToolRegistry, EventBus) に紐付けられた `PluginContext` インスタンスを作成します。
    *   `plugin.initialize(context)` を呼び出します。
    *   **プラグインコードの実行：** プラグイン内部で `context.registerTool(...)` が呼び出されます。
    *   **実際の紐付け：** `PluginContext` がツールを受け取ると、それをコアの `ToolRegistry` に保存します。

5.  **ライフサイクル管理：**
    ローダーは、エージェントの終了時に `shutdown()` を呼び出すために、プラグインインスタンスを内部の Map に保持します。

---

## 4. コード例 (Implementation Example)

### プラグイン側の書き方 ( `plugins/standard/fs/index.ts` )

```typescript
import { IPlugin, IPluginContext } from '@openstarry/sdk';
import { ReadFileTool } from './tools/ReadFile';

export default class FileSystemPlugin implements IPlugin {
  id = 'openstarry-fs';
  version = '1.0.0';

  async initialize(context: IPluginContext) {
    context.logger.info('ファイルシステム能力をマウントしています...');
    
    // ツールの注入
    context.registerTool(new ReadFileTool());
    
    // 設定で許可されている場合は、書き込みツールも注入できる
    if (context.config.enableWrite) {
       // ... WriteFileTool を登録
    }
  }
  
  async shutdown() {}
}
```

### コア側の書き方 ( `packages/core/infrastructure/PluginLoader.ts` )

```typescript
class PluginContextImpl implements IPluginContext {
  constructor(private core: AgentCore, private pluginConfig: any) {}

  registerTool(tool: ITool) {
    // コアのレジストリを直接操作する
    this.core.toolRegistry.register(tool);
  }
  
  // ... その他のメソッド
}
```

---

## 5. エラー処理

*   **ロード失敗：** `require` に失敗した場合、ローダーはエラーを記録すべきですが、コアをクラッシュさせてはいけません（重要なプラグインである場合を除く）。
*   **初期化タイムアウト：** `initialize` メソッドにはタイムアウト制限を設け、プラグインが起動プロセスをフリーズさせるのを防ぐべきです。
*   **衝突の処理：** 異なるプラグインが同名のツールを登録した場合（例：2つのプラグインが共に `read_file` を持っている場合）、ローダーは設定ポリシー（上書き、エラー、または名前空間による隔離）に基づいて処理を行うべきです。
