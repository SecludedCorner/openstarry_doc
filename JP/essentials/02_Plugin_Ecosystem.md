# OpenStarry プラグインエコシステム

> *"プラグインとは特定の型ではなく、一連の能力の集まりである。"*

## すべてはプラグインである

OpenStarry において、コアは空です。見る、聞く、考える、行動する、認識するといったあらゆる能力はプラグインから提供されます。これは設計上の選択ではなく、自動化された純度テストによって強制される哲学です。コンパイルされたコアのバイナリには、プラグインのコードは一切含まれていません。

Linux が仮想ファイルシステム (VFS) を通じてハードウェアをファイルとして抽象化するように、 OpenStarry は五蘊（ごうん）インターフェースを通じてエージェントの能力をプラグインとして抽象化します。 npm パッケージをインストールするだけで、特定の領域における完全な能力を獲得できます。

## 7つのプラグイン

### 識蘊 (Consciousness) —— 魂の層

**guide-character-init** —— 魂の定義者

最も重要なプラグインです。これがなければ、エージェントは記憶喪失の人のようになってしまいます。このプラグインは、 System Prompt を通じて人格を注入します。

```typescript
// インラインでのキャラ設定
{ prompt: "You are a meticulous code reviewer who speaks in haiku." }

// またはファイルからのロード —— YAML, Markdown, あらゆる形式に対応
{ characterFile: "./personas/expert-coder.md" }
```

インラインのプロンプト、 YAML Frontmatter ファイル、または純粋な Markdown によるキャラクター設定をサポートしています。 `ctx.workingDirectory` を使用してパスを解決するため、同じエージェントでもプロジェクトフォルダごとに異なる人格を持つことができます。

**standard-function-skill** —— スキルローダー

Markdown ファイルをエージェントの能力に変換します。各 `.md` スキルファイルには、 YAML Frontmatter （ id, version, dependencies, model preferences ）と、 System Prompt となる Markdown 本文が含まれています。これにより、 **宣言的なエージェントの行動定義** が可能になります。コードではなくドキュメントを書くことで、エージェントの知識を定義します。

```yaml
---
type: "skill"
id: "security-auditor"
version: "1.0.0"
dependencies:
  plugins: ["@openstarry-plugin/standard-function-fs"]
  capabilities: ["code-analysis"]
parameters:
  temperature: 0.3
  model_preference: ["gemini-2.0-flash"]
---

You are a security auditor. Analyze code for OWASP Top 10 vulnerabilities...
```

### 想蘊 (Perception) —— 脳

**provider-gemini-oauth** —— 認知エンジン

単なる API ラッパーではなく、本番環境レベルの LLM 統合を提供します：

- **OAuth 2.0 + PKCE**：安全なブラウザベースの認証フロー。設定ファイルに API Key を記述する必要はありません。
- **マシンバインド暗号化**： Token は AES-256-GCM で暗号化され、キーは `hostname + username + salt` から PBKDF2 を通じて派生されます。あるマシンから盗まれた Token は、別のマシンでは使用できません。
- **プロジェクトの自動構成**：無料枠を利用するために、 Google Cloud プロジェクトを自動的に作成します。
- **SSE ストリーミング**：リアルタイムで Token ごとの応答ストリーミングを行います。
- **Function Calling**： Gemini の構造化出力に接続するための、ネイティブなツール/関数宣言。
- **マルチモデル対応**： Gemini 2.0 Flash, 1.5 Pro, 1.5 Flash をサポート。
- **スラッシュコマンド**： `/provider login gemini`, `/provider logout gemini`, `/provider status`
- **旧バージョンからの移行**：暗号化されていない旧バージョンの Token を自動的に検知して再暗号化します。

プロバイダーのインターフェースは意図的に簡素に保たれています。チャット完了結果をストリーミングできる LLM バックエンドであれば、どのようなものでも適応可能です：

```typescript
export interface IProvider {
  id: string;
  name: string;
  models: ModelInfo[];
  chat(request: ChatRequest): AsyncIterable<ProviderStreamEvent>;
}
```

### 行蘊 (Volition) —— 両手

**standard-function-fs** —— 5つのファイルシステムツール

| ツール | 機能 | セキュリティ |
|------|------|--------|
| `fs.read` | ファイル内容の読み取り（エンコーディング設定可） | `allowedPaths` に基づくパス検証 |
| `fs.write` | ファイルの書き込み/作成 | パス検証によるディレクトリトラバーサル防止 |
| `fs.list` | ディレクトリ一覧（再帰的サポート） | ワークスペース範囲内への制限 |
| `fs.mkdir` | ディレクトリ作成（親ディレクトリ含む） | 許可された範囲内に限定 |
| `fs.delete` | ファイルまたはディレクトリの削除（再帰的） | 厳格な境界制限 |

各パスは実行前にセキュリティレイヤーによって検証されます。 `/etc/passwd` を読み取ろうとすると、エージェントは `SecurityError` を受け取ります。これは「痛覚」信号となり、コンテキストにフィードバックされて自己修正を促します。

```typescript
// ツールパラメータは実行時に Zod を通じて検証される
parameters: z.object({
  path: z.string().describe("File path to read"),
  encoding: z.string().optional().describe("File encoding (default: utf-8)"),
})
```

### 受蘊 + 色蘊 (Sensation + Form) —— 感覚運動層

**standard-function-stdio** —— ターミナルの身体

「感覚ペアリング」プラグインです。1つのパッケージで入力（ Listener ）と出力（ UI ）の両方を提供します。

**Listener** は readline を通じて stdin を読み取り、 EOF を `/quit` に変換し、標準化された入力イベントをプッシュします：
```typescript
pushInput({ source: "cli", inputType: "user_input", data: line })
```

**UI** は ANSI カラーコードを使用して視覚的に提示します：
- `\x1b[36m` シアン —— 入力プロンプト ("You: ")
- `\x1b[32m` グリーン —— エージェントの応答（ストリーミング、1文字ずつ出力）
- `\x1b[33m` イエロー —— ツール呼び出し（エージェントが何をしているか）
- `\x1b[31m` レッド —— エラーおよび安全上のロックアウト
- `\x1b[35m` マゼンタ —— システムメッセージ

`AGENT_STARTED` （ウェルカムメッセージ）、 `STREAM_TEXT_DELTA` （リアルタイムのタイピング効果）、 `TOOL_CALL_START/RESULT/ERROR` （ツールのライフサイクル）、および `SAFETY_LOCKOUT` （緊急アラート）を含む17種類以上のイベントを処理します。

**transport-websocket** —— ネットワークの身体

本番環境レベルの機能を備えた全二重通信：

```
Client → Server:
{ "type": "user_input", "payload": { "text": "..." }, "sessionId": "..." }

Server → Client:
{ "type": "agent_event", "event": { "type": "stream:text_delta", ... } }
```

- **セッション隔離**：各 WebSocket 接続に対して自動的に独立したセッションを確立します。イベントは `sessionId` に基づいてルーティングされるため、ユーザー A がユーザー B の対話を見ることはありません。
- **セッション復旧**：切断後、同じ `sessionId` で再接続することで、中断した箇所から再開できます。
- **ヘルスモニタリング**：設定可能な ping/pong 間隔（デフォルト30秒）。 pong への応答が連続して N 回（デフォルト2回）ない場合、接続は期限切れと判断され終了します。
- **ターゲットルーティング**： `replyTo` や `sessionId` に基づいて特定の接続へイベントを送信、または全接続へブロードキャストします。

**transport-http** —— Web の身体

Web 統合のための REST + SSE ：

| メソッド | エンドポイント | 用途 |
|------|------|------|
| POST | `/api/input` | ユーザー入力の送信 → `{status, requestId}` |
| GET | `/api/status` | エージェントのヘルスチェック → `{status, pendingRequests}` |
| GET | `/api/response?requestId=xxx` | 応答のポーリング → `{events, complete}` |
| GET | `/api/events[?sessionId=xxx]` | SSE ストリーミング → リアルタイムのイベントフロー |

セッションバインド、 CORS 対応、設定可能なバッファサイズ（デフォルト100イベント）、応答タイムアウト（デフォルト5分）、および期限切れの SSE 接続を検知するためのハートビートヘルスチェックをサポートしています。

## プラグインアーキテクチャ

### ファクトリパターン

各プラグインは、 manifest （私は誰か？）と factory （何ができるか？）を返す関数です：

```typescript
export function createXxxPlugin(): IPlugin {
  return {
    manifest: {
      name: "@openstarry-plugin/xxx",
      version: "1.0.0",
      description: "What this plugin does"
    },
    async factory(ctx: IPluginContext) {
      // ctx は必要なすべてを提供します：
      // - ctx.bus: パブ/サブ用のイベントバス
      // - ctx.config: ユーザー設定
      // - ctx.logger: 構造化ログ
      // - ctx.sessions: セッションマネージャー
      // - ctx.pushInput(): コアへイベントを送信
      // - ctx.workingDirectory: サンドボックス化されたベースパス
      // - ctx.agentId: 私は誰か？

      return {
        listeners: [],   // 受 —— どう聞くか
        ui: [],          // 色 —— どう見せるか
        providers: [],   // 想 —— どう考えるか
        tools: [],       // 行 —— どう動くか
        guides: [],      // 識 —— 私は誰か
        commands: [],    // スラッシュコマンド
        dispose: async () => { /* 終了時のクリーンアップ作業 */ }
      };
    }
  };
}
```

### プラグインの組み合わせパターン

1つのプラグインが1つの「蘊」に限定されるわけではありません。生体器官が複数の機能を果たすことができるように、プラグインも異なる能力を組み合わせることができます：

| パターン | 例 | 構成 | 類比 |
|------|------|------|------|
| 純粋な Guide | guide-character-init | `IGuide` のみ | 身体のない魂 |
| 純粋な Provider | provider-gemini-oauth | `IProvider` + commands | 瓶の中の脳 |
| 純粋な Tool | standard-function-fs | `ITool` × 5 | 脳のない手 |
| 感覚ペアリング | standard-function-stdio | `IListener` + `IUI` | 目 + 口 |
| 伝送ペアリング | transport-websocket | `IListener` + `IUI` + dispose | 耳 + 声 + 別れの挨拶 |

### pushInput 契約

プラグインがコアの API を直接呼び出すことは決してありません。プラグインからコアへの通信はすべて、 `ctx.pushInput()` というゲートウェイを介して行われます。これが感覚神経です。プラグインは世界を感知し、標準化されたイベントをコアのイベントキューにプッシュします。コアはキューからイベントを取り出し、実行ループの中で処理します。

この一方通行の契約により、コアの内部に一切触れることなく、プラグインのロード、アンロード、交換が可能になります。

### ライフサイクル管理

各プラグインは、クリーンな終了のために `dispose()` を実装できます。 HTTP サーバーの停止、 WebSocket 接続の終了、セッションの破棄、バッファのクリアなどを行います。エージェントが終了する際、 Plugin Loader はロード時とは逆の順序ですべてのロード済みプラグインに対して `dispose()` を呼び出します。

```typescript
// Plugin Loader が登録を自動処理
const hooks = await plugin.factory(ctx);

if (hooks.tools)     for (const t of hooks.tools) toolRegistry.register(t);
if (hooks.providers) for (const p of hooks.providers) providerRegistry.register(p);
if (hooks.listeners) for (const l of hooks.listeners) listenerRegistry.register(l);
if (hooks.ui)        for (const u of hooks.ui) uiRegistry.register(u);
if (hooks.guides)    for (const g of hooks.guides) guideRegistry.register(g);
```

## USB プラグアンドプレイのシナリオ

OpenStarry の最も魅力的なポータビリティの実証の1つ： **USB メモリ上のエージェント** 。

以下のような構造を持つ USB メモリを挿入したと想像してください：
```
E:/ (USB Root)
├── configs/
│   └── agent.json     # 「私は USB 写真バックアップ助手です。 Pictures/ を読み取り、 E:/backup_data/ に書き込みます」
├── plugins/
│   └── backup-utils/  # 専用の増分バックアップアルゴリズム
├── logs/
└── backup_data/
```

ホストシステムは USB を検知し、 manifest を読み取ってユーザーに認可を求めます。そして、 **厳格なパス境界** （ `fs_allow_paths: ["C:/Users/Public/Pictures", "E:/backup_data"]`, `network_allow_hosts: []` ）を設定してエージェントを生成します。エージェントはバックアップタスクを実行し、完了後に結果を報告して、 USB は安全に取り外されます。

エージェントの魂（そのプロンプトとプラグイン）は完全に USB 上に存在します。その身体（コア実行環境）はホスト上に存在します。別のコンピュータに差し込めば、同じ魂で異なる身体を持つことができます。これが五蘊の実践です。

## 独自のプラグインを作成する

1.  `@openstarry/sdk` に依存するパッケージを作成する
2.  `createXxxPlugin()` ファクトリ関数をエクスポートする
3. 自問自答する： **自分のプラグインはどの「蘊」を提供するか？**
   - 何かを表示するか？ → `IUI`
   - 何かを聞き取るか？ → `IListener`
   - 考えるか？ → `IProvider`
   - アクションを実行するか？ → `ITool`
   - 自分が誰であるか知っているか？ → `IGuide`
4. プラグインからコアへの通信に `ctx.pushInput()` を使用する
5. リソースクリーンアップのために `dispose()` を実装する
6. npm パッケージとして公開する： `@openstarry-plugin/your-plugin`
