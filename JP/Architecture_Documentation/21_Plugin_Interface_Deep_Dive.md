# 21. 深度解析：プラグインインターフェースと五蘊の関係 (Plugin Interface & Aggregates Deep Dive)

> **技術仕様に関する注意 (Technical Specification Note):**
> このドキュメントは、アーキテクチャの哲学と概念の説明に重点を置いています。 `IPlugin`, `ITool`, `IAgentGuide` などのインターフェースの厳密な TypeScript 定義および最新の API 仕様については、 **[03_Plugin_Interface_Definitions.md](../Technical_Specifications/03_Plugin_Interface_Definitions.md)** を参照してください。

このドキュメントの目的は、 `IPlugin` (プラグイン容器)、 `IPluginContext` (対話環境)、および `ITool/IListener` (五蘊の成分) の3者のアーキテクチャ上の関係を明確にすることです。

## 1. なぜ五蘊のインターフェースがあるのに、さらに `IPlugin` が必要なのか？

これは「容器 (Container)」と「内容物 (Content)」の違いです。

*   **五蘊 (ITool, IListener...)：** これらは **「部品」** です。例えば、天気予報関数は1つの `ITool` であり、Discord リスナーは1つの `IListener` です。
*   **プラグイン (IPlugin)：** これは **「小包 / 配送箱」** です。

コアは、ハードディスク内にどのような `ITool` があるかを何もないところから知ることはできません。「小包を開けて」中から部品を取り出すための標準的なエントリポイントが必要です。 `IPlugin` は、まさにその標準的な入り口です。

### 比喩
*   **コア：** コンピュータの本体です。
*   **ITool (行蘊)：** マウスです。
*   **IListener (受蘊)：** キーボードの入力端です。
*   **IUI (色蘊)：** モニターの出力端です。
*   **IPlugin：** **USB コネクタとドライバ** です。本体に対して「ここにマウスとキーボード入力端とモニター出力端があるから、これらを動かしてくれ」と伝える役割を担います。

**重要な区別：** `IListener` と `IUI` は分離されたインターフェースです。
- `IListener` (受蘊) は外部入力の受信に専念し、 `ctx.pushInput()` を介してイベントをプッシュします。
- `IUI` (色蘊) は出力のレンダリングに専念し、 `onEvent()` を介してイベントを受信して提示します。

### コードの関係
```typescript
// 開発者は IPlugin インターフェースに従って「小包」を定義します
export default class MyPlugin implements IPlugin {
  // コアがこのメソッドを呼び出すことは、「USB を差し込む」ことに相当します
  async initialize(context: IPluginContext) {
    // ここで開発者は「部品（五蘊）」を取り出してコアに渡します
    context.registerTool(new WeatherTool()); // WeatherTool は ITool に従う
    context.registerListener(new DiscordListener()); // DiscordListener は IListener に従う
  }
}
```

---

## 2. なぜ `IPluginContext` が必要なのか？

`IPluginContext` は、コアとプラグインの間の **「取引窓口」** または **「交換プロトコル」** です。これは2つの方向の情報フローを支えています。

### 方向 A：コア -> プラグイン (能力の付与)
プラグインが動作するためには、コアが提供するインフラストラクチャが必要です。
*   **Logger:** プラグインがログを記録できるようにします。
*   **Config:** プラグインが `agent.json` 内の設定を読み取れるようにします。
*   **Dependencies:** (重要) プラグインが他のプラグインが提供するサービスを取得できるようにします。

### 方向 B：プラグイン -> コア (成分の引き渡し)
プラグインは、自身が持つ五蘊の成分をコアシステムに「登録」するための経路を必要とします。
*   `registerTool()`
*   `registerListener()`
*   `pushInput(event: InputEvent)` — プラグインが能動的にコアへ入力イベントをプッシュできるようにします（例：Stdio リスナーがユーザー入力を受け取った後、コンストラクタのコールバックではなく、 `context.pushInput()` を介してイベントを実行ループに注入します）。

`IPluginContext` がなければ、プラグインは孤島になってしまい、コアからリソースを受け取ることも、自身の機能をコアに渡すこともできなくなります。

> **設計上の注釈：コンストラクタのコールバックに代わる `pushInput`**
> 初期の形式では、リスナープラグインはコンストラクタで注入されたコールバックを介してユーザー入力をコアに返していました。これはプラグインとコアの間の密結合を引き起こしていました。現在のアーキテクチャでは、すべてのプラグインが `context.pushInput(event)` を介して入力イベントをプッシュするように統一されており、完全な制御の反転 (IoC) を実現しています。

> **設計上の注釈：BUILTIN_FACTORIES に代わる動的ロード**
> 旧バージョンのアーキテクチャには、短い名前を組み込みプラグインファクトリに対応させる `BUILTIN_FACTORIES` マップが存在していました。このメカニズムは完全に削除されました。すべてのプラグイン（公式の標準プラグインを含む）は `import()` を介して動的にロードされ、コアはプラグインへの参照を一切ハードコードしなくなりました。これにより、カーネルの絶対的な純粋性が確保されます。

---

## 3. 調整レイヤーと `dependencies` フィールド

「調整レイヤーに dependencies フィールドが追加された」というのは、実行時の **依存性注入メカニズム** のことを指します。

### シナリオ：Workflow プラグインが MD ファイルを理解する必要がある
1.  **Skill Plugin (提供者):** `MarkdownParser` サービスを実装しています。
2.  **Workflow Plugin (消費者):** 動作するためにこのパーサーを必要としています。

### 調整レイヤー (Daemon) の仕事
Workflow プラグインを起動する前に、調整レイヤーは以下の処理を行います。

```typescript
// 1. まず Skill Plugin からパーサーのインスタンスを取得する
const mdParser = skillPlugin.getService();

// 2. Workflow プラグイン用の Context を準備する
const contextForWorkflow = {
  logger: ...,
  // 3. パーサーを dependencies フィールドに詰め込む
  dependencies: {
    markdownParser: mdParser 
  }
};

// 4. Workflow プラグインを初期化する
workflowPlugin.initialize(contextForWorkflow);
```

### 開発者の視点
Workflow プラグインのコード内：

```typescript
initialize(context: IPluginContext) {
  // Context から調整レイヤーが準備した依存関係を取り出す
  const parser = context.dependencies.markdownParser;
  
  // それを使用し始める
  parser.parse("workflow.md");
}
```

## 4. まとめ

*   **五蘊インターフェース ( `ITool` など)**： **「何であるか」** （それはツールである）を定義します。
*   **`IPlugin`**： **「どうロードするか」** （初期化のエントリポイント）を定義します。
*   **`IPluginContext`**： **「どうコミュニケーションするか」** （リソースと能力を交換するためのパイプライン）を定義します。

これら3つはどれも欠かすことができず、共にプラグイン可能で協調可能なエコシステムを構成しています。
*   **動的ロードの適用**：旧バージョンのアーキテクチャには、短い名前を組み込みプラグインファクトリに対応させる `BUILTIN_FACTORIES` マップが存在していました。このメカニズムは完全に削除されました。すべてのプラグイン（公式の標準プラグインを含む）は `import()` を介して動的にロードされます。
*   **IUI と IListener の分離**： `IListener` (受蘊) は外部入力の受信に専念し、 `IUI` (色蘊) は出力のレンダリングに専念します。
