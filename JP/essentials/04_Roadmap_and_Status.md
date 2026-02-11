# OpenStarry：システム概要とアーキテクチャの進化

> *"私たちは単に Chatbot を構築しているわけではありません。デジタル種のオペレーティングシステムを構築しているのです。"*

## OpenStarry に含まれるもの

完全なヘッドレス AI エージェント用オペレーティングシステム：

| レイヤー | コンポーネント |
|------|------|
| **五蘊 SDK** | TypeScript インターフェース： IUI, IListener, IProvider, ITool, IGuide |
| **エージェントコア** | 6状態実行ループ、 EventBus 、コンテキスト管理、セッション隔離 |
| **セキュリティシステム** | 3層の遮断器（リソースレベル → 行動レベル → 人間によるオーバーライド） |
| **プラグインエコシステム** | Transport （ stdio, WebSocket, HTTP+SSE ）、 Provider （ Gemini OAuth ）、 Tools （ファイルシステム）、 Guides （キャラクター、スキル） |
| **セッション管理** | 接続ごとの隔離、セッション復旧、統合 TraceId |
| **Orchestrator Daemon** | `openstarryd` —— ライフサイクル管理、プロセス隔離（ Docker, WASM ）、状態の永続化 |
| **MCP プロトコル** | エージェント間協調。 JSON-RPC 2.0 ベース、ツールの公開、再帰防止 |
| **TUI ダッシュボード** | リアルタイムのエージェント監視、対話型デザイナー、プラグイン編成 |
| **可観測性** | 構造化 JSON ログ、 Trace コンテキストの伝播、ヘルスチェック |

## どのように構築されたか：8つの進化段階

OpenStarry は一度に完成したわけではありません。8つの綿密に計画されたアーキテクチャ段階を経て段階的に進化してきました。これは、デジタル生物が胚から成熟へと成長していく過程に似ています。

### 第1段階：創世記 —— 骨格

Monorepo の構築、厳格な TypeScript 設定、 pnpm workspace プロトコル、および五蘊 SDK インターフェース。この段階では、エージェントは単なる型定義の集まり —— つまり、生物が形を成す前の DNA のような純粋な潜在能力でした。

**主要な成果：** 明確な依存関係の境界を持つ 11 個のワークスペースパッケージ。

### 第2段階：意識の核心 —— 鼓動

実行ループのステートマシン、 EventBus 、コンテキスト管理、および遮断器を備えた安全監視。エージェントは「鼓動」—— 継続的な「感知 → 思考 → 行動 → 学習」のサイクルを獲得しました。

**主要な成果：**
- 6状態実行ループ (WAITING → ASSEMBLING → AWAITING_LLM → PROCESSING → EXECUTING_TOOLS → SAFETY_LOCKOUT)
- Token 予算 (100k) 、ループ上限 (50回) 、重複失敗検知 (SHA-256 フィンガープリント)
- 非同期 FIFO イベントキューによる生産者と消費者の分離
- System Prompt アンカリングを備えたスライディングウィンドウ型コンテキスト管理

### 第3段階：身体と感覚 —— 最初の器官

プラグインインフラ： Plugin Loader による自動 Hook 登録、 Zod→JSON Schema 変換を備えた Tool Registry 、 Provider Registry 、 UI Registry 、 Listener Registry 、 Guide Registry 。空だったコアが器官を受け入れられるようになりました。

**主要な成果：** 汎用プラグインロードシステム。「1つの npm パッケージをインストール = 特定領域の完全な能力を獲得」。

### 第4段階：最初の呼吸 —— 生命の誕生

CLI Runner ( `apps/runner` )、 stdio プラグイン（ターミナル I/O ）、 Gemini OAuth Provider （脳）、ファイルシステムツール（両手）、およびキャラクター初期化 Guide （魂）。 OpenStarry エージェントが初めて感知し、考え、行動し、話せるようになりました。

**主要な成果：**
- エンドツーエンドのエージェント・ライフサイクル：起動 → プラグインロード → 監視 → 応答 → 終了
- OAuth 2.0 + PKCE 認証と AES-256-GCM マシンバインド Token 暗号化
- パスがサンドボックス化されたファイルシステムツールと Zod 検証
- 痛覚メカニズム：エラーをクラッシュではなくフィードバックとして処理

### 第5段階：マルチチャネル —— 複数の身体

WebSocket および HTTP の Transport プラグイン。同じエージェントに対して、ターミナル、 WebSocket 、または HTTP API を通じて同時にアクセスできるようになりました。セッション隔離により、各接続に独立した対話状態が提供されます。

**主要な成果：**
- WebSocket：双方向通信、 ping/pong ヘルスチェック、セッション復旧
- HTTP： REST エンドポイント + リアルタイムストリーミング用の Server-Sent Events
- Transport Bridge：コアから登録済みのすべての UI へイベントをルーティング（エラー隔離機能付き）
- 接続ごとのセッション管理（デフォルトセッションへの後方互換性あり）

### 第6段階：フラクタル社会 —— エージェント間の協調

MCP (Model Context Protocol) の統合。エージェントが他のエージェントにツールを公開したり、他のエージェントのツールを呼び出したりして、動的なチームを形成できるようになりました。

**主要な成果：**
- MCP Server：すべてのエージェントが JSON-RPC 2.0 を介して自身のツールを公開可能
- MCP Client：すべてのエージェントが他のエージェントのツールを呼び出し可能
- ツールのホワイトリスト： `expose_tools` / `private_tools` 設定
- 再帰防止： TraceId + 深度カウンター（最大5層）による無限ループ防止
- DevTools：エージェントの内部状態を確認するためのデバッグインターフェース
- フラクタルな組み合わせ：エージェントチームが個々のエージェントと同じインターフェースを外部に公開

### 第7段階： Daemon —— 永続的な生命

Orchestrator Daemon ( `openstarryd` )。エージェントが真の OS レベルのプロセスとなり、永続的なライフサイクル管理が可能になりました。

**主要な成果：**
- エージェントのライフサイクル管理：生成、監視、再起動、終了
- ステート（状態）の永続化と復元：再起動後もエージェントが存続し、記憶が継承される
- プロセス隔離： Docker コンテナ、 WASM サンドボックス
- ハードウェア抽象化レイヤー (HAL)：カメラ、センサー、アクチュエータ —— エージェントが物理世界へ進出
- 自動起動登録：システム起動時にエージェントが自動開始

### 第8段階： OS の進化 —— オペレーティングシステム

TUI ダッシュボードと対話型エージェントデザイナー。デジタル生命のためのオペレーティングシステムという、全ビジョンの実現です。

**主要な成果：**
- **TUI ダッシュボード** ( `openstarry` )：実行中のすべてのエージェント（ CPU 、メモリ、思考速度、ステート）をリアルタイム監視
- **対話型デザイナー** ( `openstarry design` )：エージェントの五蘊を視覚的に組み立て（脳、感覚、ツール、魂の選択）
- **ワークフローエンジン** ( `openstarry run workflow.yaml` )：エージェントを繋いで多段階のワークフローを作成。動的なハンドオフをサポート
- **プラグイン同期** ( `openstarry plugin sync` )：能力ライブラリの管理と更新
- **エージェント登録** ( `openstarry register` )：エージェントを Daemon の管理下に登録

## 技術仕様

7つの技術仕様がシステムの契約を定義しています：

| 仕様 | 範疇 | 主要な詳細 |
|------|------|---------|
| **01: Command Registry** | プラグインの CLI コマンド登録 | 動的発見。 `registry.json` が唯一の信頼できる情報源 |
| **02: Event Bus Protocol** | 標準イベントパケット | UUID, timestamp, traceId, source, sessionId, priority |
| **03: Plugin Interfaces** | 五蘊 SDK | IUI, IListener, IProvider, ITool, IGuide + IPluginContext |
| **04: Context Management** | 階層型メモリ | 即時（5-10ターン、ロック）→ 短期（スライディングウィンドウ）→ 長期（ RAG ） |
| **05: Security Protocol** | 多層防御 | ファイルシステム・サンドボックス、コマンドのホワイトリスト、リソース配分（ 50 ticks, 100k tokens, 30秒タイムアウト） |
| **06: MCP Protocol** | エージェント間通信 | JSON-RPC 2.0 、ツールの公開ホワイトリスト、再帰防止（最大5層） |
| **07: Management Zone** | Daemon アーキテクチャ | プロセス/ Docker / WASM 隔離、 IoT 用 HAL 、 YAML 編成ルール |

## プラグイン可能な記憶戦略

エージェントの役割によって、必要となる記憶の形は異なります。 OpenStarry のコンテキスト管理は、ハードコードされたものではなくプラグインです：

| 戦略 | 仕組み | 最適なシナリオ |
|------|---------|-----------|
| **スライディングウィンドウ**（デフォルト） | FIFO —— 直近の N ターンを保持し、古いものを破棄 | シンプルな質疑応答、短期的なタスク |
| **動的な要約** | 軽量な LLM を使用して古い履歴を自然言語の要約に圧縮 | 長期的なパートナー、複雑なプロジェクト |
| **主要状態の抽出** | 対話から構造化された JSON 状態を抽出し、ナラティブなテキストを破棄 | フォーム入力ボット、予約エージェント |

```json
{
  "contextStrategy": {
    "type": "plugin",
    "pluginId": "std-summarization-strategy",
    "config": { "compressionModel": "gemini-1.5-flash", "threshold": 20 }
  }
}
```

> *"コアは常に軽量であり続け、複雑な記憶管理ロジックは専用の戦略モジュールに委ねられます。"*

## エージェントの設定

完全なエージェントは、その五蘊を通じて宣言的に定義されます：

```jsonc
{
  "identity": { "id": "dev-bot-01", "name": "Resilient Developer" },
  "plugins": [
    // [想] 脳：認知エンジン
    { "name": "@openstarry-plugin/provider-gemini" },
    // [行] 両手：ファイルシステム操作
    { "name": "@openstarry-plugin/standard-function-fs" },
    // [受] 感覚：ターミナル入力
    { "name": "@openstarry-plugin/standard-function-stdio" },
    // [受+色] ネットワークの身体： WebSocket トランスポート
    { "name": "@openstarry-plugin/transport-websocket" },
    // [識] 魂：キャラ設定と痛覚メカニズム
    { "name": "@openstarry-plugin/guide-character-init",
      "config": { "characterFile": "./personas/developer.md" } }
  ],
  "policy": {
    "safety": { "max_consecutive_errors": 3 }
  }
}
```

## データで見る OpenStarry

| 指標 | 数値 |
|------|------|
| ワークスペースパッケージ | 11 |
| プラグインパッケージ | 7+ |
| アーキテクチャドキュメント | 27 |
| 深度剖析（ディープダイブ）記事 | 14 |
| 技術仕様書 | 7 |
| 実装計画書 | 9+ |
| 実行ループのステート数 | 6 |
| イベントタイプ | 25+ |
| ドキュメントの総行数 | 10,000+ |
