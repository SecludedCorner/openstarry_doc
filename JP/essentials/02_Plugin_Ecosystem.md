# OpenStarry プラグインエコシステム

> *「プラグインは型ではない — 能力の集合体である。」*

## すべてはプラグインである

OpenStarry では、Core は空です。あらゆる能力 — 見る、聞く、考える、行動する、知る — はプラグインから提供されます。これは設計上の選択ではなく、自動化されたピュリティテストによって強制された哲学です。コンパイル済みの Core バイナリにはプラグインコードが一切含まれていません。

Linux が Virtual File System を通じてハードウェアをファイルとして抽象化するように、OpenStarry は五蘊インターフェースを通じてエージェントの能力をプラグインとして抽象化します。npm パッケージを一つインストールするだけで、完全なドメイン能力を獲得できます。

## 7つのプラグイン

### 識蘊（Consciousness） — 魂の層

**guide-character-init** — 魂の定義者

最も重要なプラグインです。これがなければ、エージェントは記憶喪失者です。このプラグインはシステムプロンプトを通じてパーソナリティを注入します。

```typescript
// インラインペルソナ
{ prompt: "You are a meticulous code reviewer who speaks in haiku." }

// またはファイルから読み込み — YAML、Markdown、何でも可
{ characterFile: "./personas/expert-coder.md" }
```

インラインプロンプト、YAML フロントマター付きファイル、純粋な Markdown キャラクターシートに対応しています。パス解決に `ctx.workingDirectory` を使用するため、同じエージェントが異なるプロジェクトディレクトリで異なるペルソナを持つことができます。

**standard-function-skill** — スキルローダー

Markdown ファイルをエージェントの能力に変換します。各 `.md` スキルファイルには YAML フロントマター（id、version、dependencies、model preferences）と、システムプロンプトとなる Markdown 本文があります。これにより**宣言的なエージェント動作**が可能になります — コードではなくドキュメントを書くことでエージェントの知識を定義します。

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

### 想蘊（Perception） — 脳

**provider-gemini-oauth** — 認知エンジン

単なる API ラッパーではなく、プロダクショングレードの LLM 統合です：

- **OAuth 2.0 + PKCE**: 安全なブラウザベースの認証フロー。設定ファイルに API キー不要
- **マシンバウンド暗号化**: AES-256-GCM でトークンを暗号化。キーは `hostname + username + salt` から PBKDF2 で導出。あるマシンから盗まれたトークンは別のマシンでは無効
- **自動プロジェクトプロビジョニング**: Google Cloud Project の自動セットアップにより無料枠を利用
- **SSE ストリーミング**: リアルタイムのトークン単位レスポンスストリーミング
- **関数呼び出し**: Gemini の構造化出力に対応したネイティブ tool/function 宣言
- **マルチモデル**: Gemini 2.0 Flash、1.5 Pro、1.5 Flash に対応
- **スラッシュコマンド**: `/provider login gemini`、`/provider logout gemini`、`/provider status`
- **レガシーマイグレーション**: 暗号化されていないレガシートークンの自動検出と再暗号化

Provider インターフェースは意図的にミニマルです — チャットコンプリーションをストリーミングできる任意の LLM バックエンドが適合します：

```typescript
export interface IProvider {
  id: string;
  name: string;
  models: ModelInfo[];
  chat(request: ChatRequest): AsyncIterable<ProviderStreamEvent>;
}
```

### 行蘊（Volition） — 手

**standard-function-fs** — 5つのファイルシステムツール

| ツール | 機能 | 安全性 |
|-------|------|--------|
| `fs.read` | ファイル内容の読み取り（エンコーディング設定可能） | `allowedPaths` に対するパスバリデーション |
| `fs.write` | ファイルの書き込み/作成 | パスバリデーション、脱出防止 |
| `fs.list` | ディレクトリの一覧表示（再帰対応） | ワークスペースにサンドボックス化 |
| `fs.mkdir` | 親ディレクトリ付きのディレクトリ作成 | 許可されたスコープ内のみ |
| `fs.delete` | ファイルまたはディレクトリの削除（再帰対応） | 厳密な境界の強制 |

すべてのパスは実行前にセキュリティ層で検証されます。`/etc/passwd` を読もうとすると？エージェントは `SecurityError` を受け取ります — これは痛覚信号となり、自己修正のためにコンテキストにフィードバックされます。

```typescript
// ランタイム時に Zod でバリデーションされるツールパラメータ
parameters: z.object({
  path: z.string().describe("File path to read"),
  encoding: z.string().optional().describe("File encoding (default: utf-8)"),
})
```

### 受蘊（Sensation）+ 色蘊（Form） — 感覚運動層

**standard-function-stdio** — ターミナルボディ

「感覚ペア」プラグインです：一つのパッケージが入力（Listener）と出力（UI）の両方を提供します。

**Listener** は readline 経由で stdin を読み取り、EOF を `/quit` に変換し、標準化された入力イベントをプッシュします：
```typescript
pushInput({ source: "cli", inputType: "user_input", data: line })
```

**UI** は視覚的な明確さのために ANSI カラーコードでレンダリングします：
- `\x1b[36m` シアン — 入力プロンプト ("You: ")
- `\x1b[32m` グリーン — エージェントレスポンス（ストリーミング、文字ごと）
- `\x1b[33m` イエロー — ツール呼び出し（エージェントが何をしているか）
- `\x1b[31m` レッド — エラーとセーフティロックアウト
- `\x1b[35m` マゼンタ — システムメッセージ

`AGENT_STARTED`（ウェルカムバナー）、`STREAM_TEXT_DELTA`（ライブタイピング）、`TOOL_CALL_START/RESULT/ERROR`（ツールのライフサイクル）、`SAFETY_LOCKOUT`（緊急アラート）を含む17以上のイベントタイプを処理します。

**transport-websocket** — ネットワークボディ

プロダクショングレードの機能を備えた完全な双方向通信：

```
Client → Server:
{ "type": "user_input", "payload": { "text": "..." }, "sessionId": "..." }

Server → Client:
{ "type": "agent_event", "event": { "type": "stream:text_delta", ... } }
```

- **セッション隔離**: 各 WebSocket 接続が自動的に独自のセッションを作成。イベントは `sessionId` でルーティング — クライアント A がクライアント B の会話を見ることはありません
- **セッション再開**: 同じ `sessionId` で切断・再接続し、中断した場所から継続可能
- **ヘルスモニタリング**: 設定可能な ping/pong 間隔（デフォルト30秒）。N 回のポンミス（デフォルト2回）後、接続は陳腐として終了
- **ディレクテッドルーティング**: `replyTo`、`sessionId`、またはすべてへのブロードキャストによるイベントルーティング

**transport-http** — Web ボディ

Web 統合のための REST + SSE：

| メソッド | エンドポイント | 目的 |
|---------|-------------|------|
| POST | `/api/input` | ユーザー入力を送信 → `{status, requestId}` |
| GET | `/api/status` | エージェントのヘルスチェック → `{status, pendingRequests}` |
| GET | `/api/response?requestId=xxx` | レスポンスのポーリング → `{events, complete}` |
| GET | `/api/events[?sessionId=xxx]` | SSE ストリーム → リアルタイムイベントストリーム |

セッションバインディング、CORS サポート、設定可能なバッファサイズ（デフォルト100イベント）、レスポンスタイムアウト（デフォルト5分）、陳腐な SSE 接続検出のためのハートビートヘルスチェックに対応しています。

## プラグインアーキテクチャ

### ファクトリーパターン

すべてのプラグインは、マニフェスト（私は誰か？）とファクトリー（何ができるか？）を返す関数です：

```typescript
export function createXxxPlugin(): IPlugin {
  return {
    manifest: {
      name: "@openstarry-plugin/xxx",
      version: "1.0.0",
      description: "What this plugin does"
    },
    async factory(ctx: IPluginContext) {
      // ctx はすべてを提供します：
      // - ctx.bus: パブ/サブ用のイベントバス
      // - ctx.config: ユーザー設定
      // - ctx.logger: 構造化ロギング
      // - ctx.sessions: セッションマネージャー
      // - ctx.pushInput(): Core にイベントを送信
      // - ctx.workingDirectory: サンドボックス化されたベースパス
      // - ctx.agentId: 自分は誰か？

      return {
        listeners: [],   // 受 — どう聞くか
        ui: [],          // 色 — どう見えるか
        providers: [],   // 想 — どう考えるか
        tools: [],       // 行 — どう行動するか
        guides: [],      // 識 — 自分は誰か
        commands: [],    // スラッシュコマンド
        dispose: async () => { /* シャットダウン時のクリーンアップ */ }
      };
    }
  };
}
```

### プラグインの組み合わせパターン

プラグインは一つの蘊に限定されません。複数の機能を持つ生体器官のように、プラグインは能力を組み合わせることができます：

| パターン | 例 | コンポーネント | 例え |
|---------|----|----|------|
| 純粋な Guide | guide-character-init | `IGuide` のみ | 身体のない魂 |
| 純粋な Provider | provider-gemini-oauth | `IProvider` + commands | 瓶の中の脳 |
| 純粋な Tool | standard-function-fs | `ITool` × 5 | 脳のない手 |
| 感覚ペア | standard-function-stdio | `IListener` + `IUI` | 目 + 口 |
| トランスポートペア | transport-websocket | `IListener` + `IUI` + dispose | 耳 + 声 + 別れの挨拶 |

### pushInput 契約

プラグインは Core API を直接呼び出しません。プラグインから Core へのすべての通信は一つのゲートウェイを通ります：`ctx.pushInput()`。これは感覚神経です — プラグインは世界を感知し、標準化されたイベントを Core のイベントキューにプッシュします。Core はイベントをプルし、実行ループで処理します。

この一方向の契約により、Core の内部に触れることなくプラグインのロード、アンロード、交換が可能です。

### ライフサイクル管理

すべてのプラグインは、グレースフルシャットダウンのために `dispose()` を実装できます — HTTP サーバーの終了、WebSocket 接続の切断、セッションの破棄、バッファのフラッシュなど。Plugin Loader はエージェントのシャットダウン時に、ロードの逆順ですべてのプラグインの `dispose()` を呼び出します。

```typescript
// Plugin Loader が登録を自動的に処理
const hooks = await plugin.factory(ctx);

if (hooks.tools)     for (const t of hooks.tools) toolRegistry.register(t);
if (hooks.providers) for (const p of hooks.providers) providerRegistry.register(p);
if (hooks.listeners) for (const l of hooks.listeners) listenerRegistry.register(l);
if (hooks.ui)        for (const u of hooks.ui) uiRegistry.register(u);
if (hooks.guides)    for (const g of hooks.guides) guideRegistry.register(g);
```

## USB プラグアンドプレイ・シナリオ

OpenStarry の移植性を最も魅力的に示すデモンストレーションの一つ：**USB スティック上のエージェント**です。

以下のような構造の USB ドライブを差し込むことを想像してください：
```
E:/ (USB Root)
├── configs/
│   └── agent.json     # "私は USB Photo Backup です。Pictures/ を読み取り、E:/backup_data/ に書き込みます"
├── plugins/
│   └── backup-utils/  # 特殊な差分バックアップアルゴリズム
├── logs/
└── backup_data/
```

ホストシステムが USB を検出し、マニフェストを読み取り、ユーザーに許可を求め、**厳密なパス境界**（`fs_allow_paths: ["C:/Users/Public/Pictures", "E:/backup_data"]`、`network_allow_hosts: []`）でエージェントを起動し、エージェントがバックアップタスクを実行します。完了すると、エージェントは結果を報告し、USB は安全に取り外し可能になります。

エージェントの魂（プロンプトとプラグイン）は完全に USB 上に存在します。身体（Core ランタイム）はホスト上に存在します。別のコンピュータに差し込めば — 同じ魂、異なる身体。これが五蘊の実践です。

## 独自プラグインの構築

1. `@openstarry/sdk` に依存するパッケージを作成します
2. `createXxxPlugin()` ファクトリー関数をエクスポートします
3. 自問してください：**このプラグインはどの蘊を提供するか？**
   - 何かを表示する？ → `IUI`
   - 何かを聞く？ → `IListener`
   - 考える？ → `IProvider`
   - 行動する？ → `ITool`
   - 自分が誰か知っている？ → `IGuide`
4. プラグインから Core への通信には `ctx.pushInput()` を使用します
5. クリーンアップのために `dispose()` を実装します
6. npm パッケージとして出荷します：`@openstarry-plugin/your-plugin`
