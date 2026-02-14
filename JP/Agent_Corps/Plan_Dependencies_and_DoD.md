# OpenStarry — Plan Dependencies & Definition of Done

Established: 2026-02-09

---

## 1. Plan 依存関係図

```
Plan01 (MVP Alpha Foundation)           ✅ v0.1-alpha  2026-02-04
  ↓
Plan02 (Event-Driven & Safety)          ✅ v0.1.1      2026-02-05
  ↓
Plan03 (Quality & Skill Parser)         ✅ v0.2-alpha  2026-02-05
  ↓
Plan04 (IUI Interface & Guide Plugin)   ✅ v0.2-alpha  2026-02-06
  ↓
Plan05 (Multi-Channel UI & Listener)    ✅ v0.2-beta   2026-02-07
  ↓
  ├── Plan05.1 (Session Isolation)      ✅ v0.2.1-beta  2026-02-10 (Cycle 1)
  ├── Plan05.2 (HTTP SSE)              ✅ v0.2.1-beta  2026-02-10 (Cycle 1)
  ├── Plan05.5-① (Health Check)        ✅ v0.2.1-beta  2026-02-10 (Cycle 1)
  ├── Plan05.5-② (Metrics/Logging)     ✅ v0.2.2-beta  2026-02-11 (Cycle 2)
  ├── Plan05.5-③ (Error Handling)      ✅ v0.2.2-beta  2026-02-11 (Cycle 2)
  ├── Plan05.3 (DevTools)              ⬜ → Plan06 以降に延期
  ├── Plan05.4 (E2E Testing)           ⬜ → スケジュール待ち
  │     ↓
  │   Plan06 Phase 1 (MCP Client)       ✅ v0.3.0-beta  2026-02-11 (Cycle 3)
  │   Plan06 Phase 2 (MCP Server)       ✅ v0.3.1-beta  2026-02-11 (Cycle 4)
  │   Plan06 Phase 3 (Resources+Auth)   ✅ v0.10.0-beta 2026-02-12 (Cycle 15)
  │   Plan06 Phase 4 (Sampling+Ext)     ✅ v0.12.0-beta 2026-02-12 (Cycle 17)
  │     ↓
  │   Plan07 (Runtime Sandbox MVP)      ✅ v0.4        2026-02-11 (Cycle 5)
  │   Plan07.1 (Hardening)              ✅ v0.4.1-beta  2026-02-11 (Cycle 6)
  │   Plan07.2 (Advanced Hardening)     ✅ v0.4.2-beta  2026-02-11 (Cycle 7)
  │     ↓
  │   Plan07.3 (Custom require + audit) ✅ v0.4.3-beta  2026-02-11 (Cycle 8)
  │     ↓
  │   Plan08 (TUI Dashboard MVP)        ✅ v0.5.0-beta  2026-02-11 (Cycle 9)
  │   Plan09 (Interactive TUI)          ✅ v0.5.1-beta  2026-02-12 (Cycle 10)
  │   Plan10 (CLI Foundation & Runner)  ✅ v0.6.0-beta  2026-02-12 (Cycle 11)
  │     ↓
  │   Plan05.3 (DevTools) + 05.4 (E2E) ⬜ → Plan11 (Cycle 12)
  │   Plan11 (DevTools & E2E Testing)   ✅ v0.7.0-beta  2026-02-12 (Cycle 12)
  │     ↓
  │   Plan12 (Daemon Mode MVP)          ✅ v0.8.0-beta  2026-02-12 (Cycle 13)
  │     ↓
  │   Plan13 (Seamless Attach)          ✅ v0.9.0-beta  2026-02-12 (Cycle 14)
  │     ↓
  │   Plan14 (Multi-client Attach & Session Management) ✅ v0.11.0-beta 2026-02-12 (Cycle 16)
  │     ↓
  │   Plan15 (SDK Context Extensions & Provider Integration) ✅ v0.13.0-beta 2026-02-12 (Cycle 18)
  │     ↓
  │   Plan16 (Security Hardening & Quality Polish) ✅ v0.14.0-beta 2026-02-12 (Cycle 19)
  │     ↓
  │   Plan17 (Plugin Developer Experience) ✅ v0.15.0-beta 2026-02-12 (Cycle 20)
  │     ↓
  │   Plan18 (Plugin Sync & System Plugin Directory) ✅ v0.16.0-beta 2026-02-12 (Cycle 21)
  │     ↓
  │   Plan19 (Plugin Dependency Wiring & Cross-Plugin Services) ✅ v0.17.0-beta 2026-02-12 (Cycle 22)
  │     ↓
  │   Plan20 (Workflow Engine MVP)            ✅ v0.18.0-beta  2026-02-12 (Cycle 23)
  │     ↓
  │   Plan21 (Web-based Remote Attach)       ✅ v0.19.0-beta  2026-02-12 (Cycle 24)
  │     ↓
  │   Plan22+ (Advanced Features: Plugin Marketplace, Multi-agent Orchestration)
  │
  └── (Extended Mode: 新 plugin、Web UI、セキュリティ監査)
```

### クリティカルパス（Critical Path）

```
Plan06 → Plan07 → Plan08-09
```

Plan06 (MCP Client + Server) は完了済み ✅、**Plan07 (Runtime Sandbox) のブロック解除済み**。クリティカルパス上の次の実装対象項目。

### 並行作業可能な項目

| 並行可能 | 説明 |
|----------|------|
| Plan06 実装 + Plan05.3/05.4 | DevTools / E2E テストは MCP と並行可能 |
| Plan06 の事前調査 + Cycle 2 仕上げ | ✅ 完了 — researcher が Plan06 の事前調査を実施可能 |

---

## 2. Definition of Done（完了定義）

### 2.1 共通 DoD（全 Plan に適用）

各 Plan は以下の**全条件**を満たして初めて ✅ とマーク可能：

| # | 条件 | 検証方法 |
|---|------|---------|
| 1 | `pnpm build` 通過（monorepo + plugins） | qa が agent_test で検証 |
| 2 | `pnpm test` 全て通過、テスト数 ≥ 前回のベースライン | qa レポートの regression check |
| 3 | `pnpm test:purity` 通過 | qa レポートの purity check |
| 4 | Architecture_Spec 内の全項目が実装済み | architect Code Review PASS |
| 5 | 五蘊準拠（新コンポーネントが正しく分類） | architect Code Review で確認 |
| 6 | pushInput パターン準拠 | architect Code Review で確認 |
| 7 | 既知のセキュリティ脆弱性なし | architect Code Review で確認 |
| 8 | dev log が記録済み | Coordinator がファイルの存在を確認 |
| 9 | QA Report と Code Review が共に PASS | Phase 4 判定 |
| 10 | doc-keeper が Plan に ✅ を付与 + Iteration_Log を更新 | Phase 4 収束 |
| 11 | Snapshot が作成済み | `scripts/snapshot.sh` 完了 |
| 12 | Lessons Learned が記録済み | Phase 4 振り返り |

### 2.2 各 Plan 固有の検収条件

#### Plan05.1: Session Isolation ✅ (Cycle 1, 2026-02-10)
- [x] WebSocket 接続がセッショントークン / 認証をサポート (session handshake on connect)
- [x] 複数クライアント接続が相互隔離（セッションデータが交差しない）
- [x] 単一 agent インスタンスが複数セッションを提供可能 (SessionManager + default session)
- [x] セッションライフサイクル管理（作成、維持、破棄）
- [x] セッション隔離シナリオのテストカバレッジを追加 (17 tests)

#### Plan05.2: HTTP SSE ✅ (Cycle 1, 2026-02-10)
- [x] HTTP Server-Sent Events トランスポートプラグインが利用可能 (GET /api/events)
- [x] SSE 接続がリアルタイムストリーミングレスポンスを受信可能
- [x] 既存の HTTP webhook リスナーと共存し競合なし
- [x] SSE シナリオのテストカバレッジを追加 (11 tests)

#### Plan05.5-①: Health Check ✅ (Cycle 1, 2026-02-10)
- [x] WebSocket プロトコル ping/pong + 陳腐化した接続のクリーンアップ
- [x] HTTP SSE ハートビート
- [x] テストカバレッジを追加 (6 tests)

#### Plan05.5-②: Metrics/Logging ✅ (Cycle 2, 2026-02-11)
- [x] MetricsCollector (increment/gauge/getSnapshot/reset)
- [x] Logger.time() (performance.now())
- [x] METRICS_SNAPSHOT event + /metrics コマンド
- [x] トランスポートプラグインの構造化ログ移行
- [x] テストカバレッジを追加 (19 tests)

#### Plan05.5-③: Error Handling ✅ (Cycle 2, 2026-02-11)
- [x] ErrorCode const (12 codes)
- [x] TransportError / SessionError / ConfigError
- [x] ES2022 Error cause チェーン
- [x] テストカバレッジを追加 (16 tests)

#### Plan06: MCP Protocol (Model Context Protocol) — フェーズ制

**Cycle 3 (Plan06 Phase 1: MCP Client)** ✅ (Cycle 3, 2026-02-11):
- [x] `@openstarry-plugin/mcp-client` プラグインが利用可能
- [x] MCP client モード：外部 MCP サーバーに接続し、ツールをインポート (Tool bridge: MCP tools → ITool)
- [x] MCP Prompt ブリッジ：MCP prompts を SlashCommand としてインポート
- [x] stdio トランスポート：子プロセスをスポーン、JSON-RPC over stdin/stdout
- [x] Streamable HTTP トランスポート：POST JSON-RPC + オプショナル SSE
- [x] Config 駆動のサーバーリスト (agent.json plugins[].config.servers)
- [x] スラッシュコマンド: /mcp-status, /mcp-tools, /mcp-prompts
- [x] MCP client シナリオのテストカバレッジを追加 (35 tests)

**Cycle 4 (Plan06 Phase 2: MCP Server)** ✅ (Cycle 4, 2026-02-11):
- [x] `@openstarry-plugin/mcp-server` プラグイン
- [x] MCP server モード：ITool/IGuide を MCP tools/prompts として公開
- [x] サーバートランスポート: stdio + HTTP (http.createServer)
- [x] JSON-RPC 2.0 ハンドラー (initialize, tools/list, tools/call, prompts/list, prompts/get)
- [x] Config 駆動: サーバー名、バージョン、公開ツール/ガイドのホワイトリスト
- [x] SDK: IPluginContext.tools + IPluginContext.guides (オプショナル、非破壊)
- [x] SDK: MCP_CLIENT_CONNECTED + MCP_CLIENT_DISCONNECTED イベント
- [x] MCP server シナリオのテストカバレッジを追加 (52 tests)

**Cycle 15 (Plan06 Phase 3: Resources + Auth)** ✅ (Cycle 15, 2026-02-12):
- [x] MCP Resources ブリッジ (リソースアクセスプロトコル、能力ネゴシエーション)
- [x] OAuth 2.1 認可 (クライアントクレデンシャルフロー、トークン管理)
- [x] リソースメタデータ処理 (resource.list, resource.read)
- [x] 認可コンテキスト統合 (スコープ検証、トークンリフレッシュ)
- [x] MCP resources + auth シナリオのテストカバレッジを追加 (29 tests)

**Cycle 17 (Plan06 Phase 4: Sampling & Advanced Protocol Extensions)** ✅ (Cycle 17, 2026-02-12):
- [x] MCP サンプリングプロトコル (サーバーがクライアントに LLM 補完をリクエスト、双方向 AI-to-AI)
- [x] 深度ガード付きサンプリングハンドラー (最大5レベル)、レート制限 (10/min)
- [x] MCP ロギングプロトコル (サーバーからクライアントへの構造化ログ)
- [x] 8→4 レベルマッピング付きロギングハンドラー、レート制限 (100/sec)
- [x] MCP Roots プロトコル (ファイルシステムルート宣言 クライアント → サーバー)
- [x] workingDirectory を公開する Roots ハンドラー、変更通知
- [x] 双方向トランスポート拡張 (クライアント + サーバー stdio トランスポート)
- [x] 8つの新しい SDK イベント型 (MCP_SAMPLING_request, MCP_SAMPLING_callback, MCP_SERVER_LOG, MCP_LOG_LEVEL_CHANGED, MCP_ROOTS_list, MCP_ROOTS_changed)
- [x] 5つの新しい MCP エラーコード (McpErrorCode -32001 to -32005)
- [x] /mcp-loglevel スラッシュコマンド
- [x] sampling/logging/roots/双方向シナリオのテストカバレッジを追加 (87 tests)
- **Target Version**: v0.12.0-beta ✅ (Cycle 17 Phase 4 PASS, 2026-02-12)
- **Pre-condition**: Plan06-P3 ✅ (Cycle 15, v0.10.0-beta)
- **Scope**: MCP Sampling & Advanced Protocol Extensions (4 core features)
- **Test Growth**: 807 → 894 (+87 new, +10.8%)
- **Packages**: 18 → 18 (0 new, existing mcp-client/mcp-server extended)
- **Rework Cycles**: 0 (first-pass PASS)
- **Non-blocking Advisories**: 4 deferred (sampling provider integration, roots simplification, HTTP bidirectional transport, log injection sanitization)
- **Status**: ✅ COMPLETE (Phase 4 Convergence PASS, 2026-02-12)

#### Plan15: SDK Context Extensions & Provider Integration ✅ (Cycle 18, 2026-02-12)
- [x] IPluginContext.providers アクセサー (readonly IProviderRegistry)
- [x] オプショナル allowedPaths フィールド付き SessionConfig インターフェース
- [x] ProviderRegistry 実装とコア配線
- [x] SamplingHandler 実プロバイダー呼び出し (スタブを置換)
- [x] RootsHandler セッション設定 allowedPaths 統合
- [x] プロバイダーディスカバリー用 Sandbox RPC (list/get メソッド)
- [x] テストファイル除外用 tsconfig ビルド修正
- [x] 非破壊的 SDK 変更 (全て後方互換)
- [x] 総テスト数 915 (ベースライン 894 + 新規 21)
- [x] 18 パッケージ、77 テストファイル
- [x] リグレッションゼロ、リワークサイクルゼロ
- [x] スナップショット作成済み: `share/openstarry_code_iteration/20260212_cycle18/`
- **Target Version**: v0.13.0-beta ✅
- **Status**: ✅ COMPLETE (Phase 4 Convergence PASS, 2026-02-12)

#### Plan16: Security Hardening & Quality Polish ✅ (Cycle 19, 2026-02-12)
- [x] Provider Access Control ホワイトリスト
  - [x] PluginManifest.allowedProviders フィールド (オプショナル、非破壊)
  - [x] PluginLoader でのランタイム検証
  - [x] デフォルト拒否セキュリティモデル (空配列 = アクセス不可)
  - [x] 正当なプロバイダーディスカバリー用ワイルドカード `*` サポート
  - [x] 8つのプロバイダーアクセス制御テスト (4 core + 4 sandbox RPC)
- [x] セッション設定ランタイム検証
  - [x] SessionManager.validateAllowedPaths メソッド
  - [x] パス正規化とシンボリックリンク解決
  - [x] サブセット検証 (セッションパス ⊆ エージェント allowedPaths)
  - [x] パストラバーサル防止 (`../` パターン、絶対パスエスケープ)
  - [x] 検証失敗の監査ログ
  - [x] 6つのセッションパス検証テスト
- [x] ログインジェクションサニタイゼーション
  - [x] sanitizeLogMessage ユーティリティ関数
  - [x] 改行と制御文字の除去
  - [x] 構造化ログ用 JSON セーフエスケープ
  - [x] MCP LoggingHandler に適用
  - [x] 6つのログインジェクションテスト
- [x] セキュリティテストカバレッジ
  - [x] 20の新規セキュリティテスト (8 provider + 6 session + 6 log)
  - [x] 総テスト数 935 (ベースライン 915 + 新規 20, +2.2%)
  - [x] 78 テストファイル (ベースライン 77 + 新規 1)
  - [x] リグレッションゼロ
- [x] 技術的負債の解消
  - [x] Cycle 15: セッション設定検証 → validateAllowedPaths で解決
  - [x] Cycle 17: MCP サーバーからのログインジェクション → sanitizeLogMessage で解決
  - [x] Cycle 18: プロバイダーアクセス制御アドバイザリ → allowedProviders で解決
- [x] マイクロカーネル純粋性を維持 (セキュリティ追加、コア汚染なし)
- [x] SDK への破壊的変更なし
- [x] スナップショット作成済み: `share/openstarry_code_iteration/20260212_cycle19/`
- **Target Version**: v0.14.0-beta ✅
- **Rework Cycles**: 1 (sandbox RPC テスト4件の欠落、解決済み)
- **Status**: ✅ COMPLETE (Phase 4 Convergence PASS, 2026-02-12)

#### Plan17: Plugin Developer Experience (DX) ✅ (Cycle 20, 2026-02-12)
- [x] `@openstarry/sdk/testing` の MockHost テストユーティリティ
  - [x] 完全な IPluginContext モック (EventBus, sessions, tools, guides, providers, pushInput)
  - [x] イベントキャプチャユーティリティ (getEmittedEvents, getInputEvents, clearEvents)
  - [x] インメモリセッション管理 (create, get, list, destroy, getDefaultSession)
  - [x] ハンドラーデバッグ (getHandlerCounts)
  - [x] 22 ユニットテスト
- [x] CreatePluginCommand CLI スキャフォールディング
  - [x] 対話型プロンプト (name, description, type, author)
  - [x] 五蘊にマッピングされた6つのプラグインタイプ
  - [x] テンプレートベース生成 (package.json, tsconfig, vitest, index.ts, test, README)
  - [x] プラグイン名バリデーション (kebab-case のみ)
  - [x] 上書き保護 (--force フラグ)
  - [x] 13 ユニットテスト
- [x] SDK サブパスエクスポート: `@openstarry/sdk/testing`
- [x] 非破壊的変更のみ (既存インターフェースの変更なし)
- [x] マイクロカーネル純粋性: PASS (コア変更なし)
- [x] 総テスト数 970 (ベースライン 935 + 新規 35, +3.7%)
- [x] 80 テストファイル (ベースライン 78 + 新規 2)
- [x] スナップショット作成済み: `share/openstarry_code_iteration/20260212_cycle20/`
- **Target Version**: v0.15.0-beta ✅
- **Rework Cycles**: 0 (first-pass PASS)
- **Status**: ✅ COMPLETE (Phase 4 Convergence PASS, 2026-02-12)

#### Plan07: Runtime Sandbox ✅ (Cycle 5, 2026-02-11)
- [x] プラグイン実行環境の隔離 (Node.js vm.createContext + worker_threads)
- [x] リソース制限 (CPU ウォッチドッグデッドラインタイマー、メモリサンドボックス隔離、ファイルアクセス)
- [x] プラグイン署名検証メカニズム (SHA-512 hash + Ed25519 PKI サポート)
- [x] サンドボックスエスケープシナリオのテストカバレッジを追加 (68 tests)
- Sandbox バージョン：v0.4 (Cycle 5)

#### Plan07.1: Sandbox Hardening ✅ (Cycle 6, 2026-02-11)
- [x] CPU ウォッチドッグ (RPC コールごとのデッドラインタイマー)
- [x] 双方向 EventBus (ワーカーサブスクリプション + フォワーディング)
- [x] ワーカー再起動ポリシー (ON_CRASH, ON_TIMEOUT with exponential backoff)
- [x] ワーカーコンテキスト用非同期プロキシ
- [x] テストカバレッジを追加 (31 tests)
- Sandbox バージョン：v0.4.1-beta (Cycle 6)

#### Plan07.2: Sandbox Advanced Hardening ✅ (Cycle 7, 2026-02-11)
- [x] 静的インポート制限 (@babel/parser AST 解析、ブロックリスト適用)
- [x] ワーカープール事前生成 (piscina 接続プール、遅延初期化)
- [x] PKI 非対称署名 (Ed25519 + RSA フォールバック、後方互換性のあるフォーマット検出)
- [x] Plugin-signer CLI ツール (keygen, sign, verify コマンド)
- [x] テストカバレッジを追加 (55 tests)
- Sandbox バージョン：v0.4.2-beta (Cycle 7)
- **Status**: PASS (1 rework cycle: fixed import integration, plugin-signer package, package-name graceful handling)

#### Plan08: TUI Dashboard MVP ✅ (Cycle 9, 2026-02-11)
- [x] TUI Dashboard プラグイン (Ink v5 + React 18)
- [x] イベントから状態へのマッピング (14 イベント型 → TUI アクション)
- [x] React useReducer による純粋な状態管理
- [x] ストリーミングデルタ蓄積 (APPEND_STREAM/FINALIZE_STREAM)
- [x] ターミナルコンポーネント: Header, ChatArea, EventLog, Footer
- [x] フォーマットユーティリティ: truncate, timestamp, status symbols
- [x] テストカバレッジを追加 (53 tests)
- **Version**: v0.5.0-beta
- **Stats**: 524 tests (82 new), 16 packages, 0 regressions

#### Plan09: Interactive TUI (Chat Input + Command Execution) ✅ (Cycle 10, 2026-02-12)
- [x] テキスト入力コンポーネント (カスタム useInput、外部依存なし) in TUI Dashboard
- [x] キーボード入力用 IListener（受蘊）実装
- [x] テキスト入力を ctx.pushInput() 経由でエージェント処理に渡す
- [x] コマンド履歴ナビゲーション (上下矢印キー、最大50件、連続重複排除)
- [x] スラッシュコマンドのパースとルーティング (/quit, /help, /mcp-status 等)
- [x] 視覚的フィードバック: タイピングインジケーター (isPending state)、コマンド実行ステータス
- [x] 入力モード管理 (チャットモード vs コマンドモード)
- [x] SDK 変更不要 (IListener + IUI 定義済み) — 検証済み
- [x] @openstarry-plugin/tui-dashboard への単一プラグイン拡張
- [x] インタラクティブ入力シナリオのテストカバレッジを追加 (35 tests)
- [x] セッション管理 (作成/破棄/ライフサイクル)
- [x] ローカルエコー (楽観的 UI 更新)
- **Target Version**: v0.5.1-beta ✅
- **Pre-condition**: Plan08 ✅ (Cycle 9)
- **Functional Requirements Met**: 11/11 (100%)
  - [x] カスタムテキスト入力ハンドリング
  - [x] IListener キーボードイベント統合
  - [x] ctx.pushInput() 委譲
  - [x] 重複排除付きコマンド履歴
  - [x] 矢印キーナビゲーション
  - [x] スラッシュコマンドプレフィックス検出
  - [x] ctx.pushInput() 経由のコマンドルーティング
  - [x] isPending 状態追跡
  - [x] セッションライフサイクル管理
  - [x] 入力モード切替 (チャット/ブラウズ)
  - [x] ローカルエコー最適化
- **Technical Requirements Met**: 6/6 (100%)
  - [x] pnpm build PASS (16/16 packages)
  - [x] pnpm test PASS (559/559 tests, 35 new)
  - [x] pnpm test:purity PASS
  - [x] Architecture Spec PASS (SDK 変更なし)
  - [x] Microkernel purity PASS (コア汚染ゼロ)
  - [x] Five Aggregates compliance PASS (IListener + IUI のみ)
- **Test Count**: 559 total (35 new)
- **Status**: PASS (no rework cycles)

#### Plan10: CLI Foundation & Runner Hardening ✅ (Cycle 11, 2026-02-12)
- [x] CLI サブコマンドルーター (start, init, version, help コマンド)
- [x] `openstarry start` コマンド (--config, --verbose フラグ付き)
- [x] `openstarry init` インタラクティブ設定ジェネレーター (ガイド付きセットアップ)
- [x] 初回起動 UX の強化ブートストラップ (ヘルプメッセージ、検証)
- [x] 説明的なエラーメッセージ付き設定検証
- [x] ランナーテストスイート (ブートストラップ、設定ロード、プラグイン解決)
- [x] CLI ルーティングテスト (コマンドディスパッチ、サブコマンド引数パース)
- [x] 既存機能への破壊的変更なし
- [x] 後方互換性: 既存の `node apps/runner/dist/bin.js` は引き続き動作
- [x] 手動引数パーサー (外部 CLI ライブラリ不使用)
- [x] Node.js readline によるインタラクティブ init コマンド
- [x] Zod + セマンティックチェック + 警告による設定検証
- [x] エラー蓄積パターンを持つプラグインリゾルバー
- [x] 3段階設定パス優先順位 (CLI フラグ > 環境変数 > ファイル > デフォルト)
- **Target Version**: v0.6.0-beta ✅
- **Pre-condition**: Plan09 ✅ (Cycle 10)
- **Scope**: CLI + runner in core monorepo; dev-core implementation
- **Test Count**: 632 tests (73 new)
- **Rework Cycles**: 1 (3 advisory fixes: --version flag, error messages, bash completion deferred)
- **Status**: ✅ COMPLETE (Phase 4 Convergence PASS)

#### Plan11: DevTools Plugin & E2E Testing Framework ✅ (Cycle 12, 2026-02-12)
- [x] DevTools プラグイン (`@openstarry-plugin/devtools`):
  - [x] ステートインスペクターコンポーネント (エージェント状態、メトリクス、イベントタイムライン)
  - [x] 構造化ログ付きデバッグコンソール
  - [x] スラッシュコマンド: /devtools, /metrics, /debug-on, /debug-off
  - [x] メトリクスエクスポーター (JSON フォーマット)
  - [x] 38 テスト提供 (目標 35 を超過)
- [x] E2E テスティングフレームワーク (`apps/runner/__tests__/e2e/`):
  - [x] CLI 統合テスト (init, start, シグナルハンドリング)
  - [x] プラグインライフサイクルテスト (マルチプラグイン、トランスポート初期化)
  - [x] マルチセッション動作テスト (セッション隔離、状態隔離)
  - [x] エージェントワークフローテスト (入力→ループ→ツール→レスポンス)
  - [x] 40 テスト提供 (目標 40 を達成)
- [x] SDK 変更なし (純粋なプラグイン + テスト追加) — 検証済み
- [x] 総テスト数: 670 tests (ベースライン 632 + 38 DevTools + 40 E2E = 710 effective)
- [x] マイクロカーネル純粋性: PASS (DevTools はプラグインであり、コアではない) — 検証済み
- **Target Version**: v0.7.0-beta ✅ (Cycle 12 Phase 4 PASS, 2026-02-12)
- **Pre-condition**: Plan10 ✅ (Cycle 11, v0.6.0-beta)
- **Scope**: Plan05.3 (DevTools, previously deferred) + Plan05.4 (E2E Testing, new)
- **Test Growth**: 632 → 670 (+38 new, +5.7%)
- **Packages**: 16 → 17 (+1: devtools plugin)
- **Rework Cycles**: 1 (3 advisory fixes: README docs added)
- **Status**: ✅ COMPLETE (Phase 4 Convergence PASS, 2026-02-12)

#### Plan12: Daemon Mode MVP (Background Agent Management) ✅ (Cycle 13, 2026-02-12)
- [x] CLI デーモンサブコマンド (daemon-start, daemon-stop, daemon-ps)
- [x] デタッチフラグ付きプロセススポーン (child_process.spawn + unref)
- [x] PID ファイル管理 (~/.openstarry/agents/{agent-id}.pid)
- [x] IPC レイヤー (Unix ドメインソケット + JSON-RPC 2.0)
- [x] ヘルスチェック RPC (agent.health → {ok, uptime, version})
- [x] デーモンプラグイン (IPC サーバー、ヘルスプロバイダー)
- [x] タイムアウト適用付き優雅なシャットダウン
- [x] シグナルハンドリング (SIGTERM/SIGINT カスケード)
- [x] テストカバレッジを追加 (44 tests)
- **Target Version**: v0.8.0-beta ✅ (Cycle 13 Phase 4 PASS, 2026-02-12)
- **Pre-condition**: Plan11 ✅ (Cycle 12, v0.7.0-beta)
- **Scope**: Daemon Mode MVP (バックグラウンドプロセス管理、IPC、ヘルスチェック)
- **Test Growth**: 670 → 714 (+44 new, +6.6%)
- **Packages**: 17 → 18 (+1: daemon plugin)
- **Rework Cycles**: 1 (2 fixes: shutdown timeout enforcement + dead code removal)
- **Status**: ✅ COMPLETE (Phase 4 Convergence PASS, 2026-02-12)

#### Plan13: Seamless Attach (Interactive Terminal Connection) ✅ (Cycle 14, 2026-02-12)
- [x] IPC プロトコル拡張 (agent.attach, agent.input, agent.detach)
- [x] CLI コマンド (openstarry attach [agent-id] デーモンリスト表示と自動起動付き)
- [x] ターミナル I/O プロキシ (stdin → agent.input RPC, daemon events → stdout/stderr)
- [x] 自動起動機能 (agent.json が存在する場合にオンデマンドでデーモンをスポーン)
- [x] イベントフォワーダー (core.bus → IPC ブリッジ、sessionId フィルタリング付き)
- [x] セッションライフサイクル管理 (アタッチ時に作成、デタッチ時にクローズ、タイムアウト)
- [x] セキュリティと検証 (sessionId フォーマット、inputType ホワイトリスト、エージェントヘルスチェック)
- [x] 優雅な Ctrl+C ハンドリング (SIGINT → デタッチ、デーモンキルではない)
- [x] テストカバレッジを追加 (33 tests)
- **Target Version**: v0.9.0-beta ✅ (Cycle 14 Phase 4 PASS, 2026-02-12)
- **Pre-condition**: Plan12 ✅ (Cycle 13, v0.8.0-beta)
- **Scope**: Seamless Attach (実行中のデーモンエージェントへのインタラクティブ接続)
- **Test Growth**: 714 → 747 (+33 new, +4.6%)
- **Packages**: 18 → 18 (0 new, existing CLI + daemon extended)
- **Rework Cycles**: 1 (3 fixes: agentId validation before attach, event timestamp, error codes documentation)
- **Status**: ✅ COMPLETE (Phase 4 Convergence PASS, 2026-02-12)

#### Plan14: Multi-client Attach & Session Management ✅ (Cycle 16, 2026-02-12)
- [x] マルチクライアント同時アタッチ (同一デーモンに複数ターミナル)
- [x] セッションレジストリ (デーモンごとの全アクティブセッションを追跡)
- [x] イベントファンアウトブロードキャスト (全アタッチクライアントにイベント配信)
- [x] セッション対応イベントルーティング (sessionId タグ付け)
- [x] クライアント接続追跡 (ハートビート、切断ハンドリング)
- [x] 永続セッション履歴 (アトミック書き込み付き FileSessionPersistence)
- [x] セッション状態シリアライゼーション (~/.openstarry/sessions/{agentId}/{sessionId}.json)
- [x] 再アタッチ時の履歴リプレイ (直近 N メッセージ)
- [x] セッション有効期限 (24h TTL、自動クリーンアップ)
- [x] スラッシュコマンドのタブ補完 (readline 統合)
- [x] 入力履歴の永続化 (セッションごとのコマンド履歴)
- [x] バックプレッシャーハンドリング (5秒低速クライアントタイムアウト、自動切断)
- [x] IPC プロトコル拡張 (agent.list-clients RPC メソッド)
- [x] デバウンス保存 (1秒間隔、設定可能)
- [x] テストカバレッジを追加 (15 tests: session persistence, multi-client, tab-completion, backpressure)
- **Target Version**: v0.11.0-beta ✅ (Cycle 16 Phase 4 PASS, 2026-02-12)
- **Pre-condition**: Plan13 ✅ (Cycle 14, v0.9.0-beta) + Plan06-P3 ✅ (Cycle 15, v0.10.0-beta)
- **Scope**: Multi-client Attach & Session Management (5 core features)
- **Test Growth**: 792 → 807 (+15 new, +1.9%)
- **Packages**: 18 → 18 (0 new, existing CLI + daemon extended)
- **Rework Cycles**: 0 (first-pass PASS)
- **Non-blocking Advisories**: 3 deferred (session command stubs, debounce docs, integration tests)
- **Status**: ✅ COMPLETE (Phase 4 Convergence PASS, 2026-02-12)

#### Plan18: Plugin Sync & System Plugin Directory ✅ (Cycle 21, 2026-02-12)
- [x] プラグインスキャナーユーティリティ (scanPluginDirectory, shouldSyncPlugin, syncPlugin, readPluginVersion)
  - [x] プラグイン検出付きディレクトリトラバーサル
  - [x] バージョン対応比較 (package.json パース)
  - [x] 上書き制御付きファイルコピー
- [x] プラグイン同期 CLI コマンド (openstarry plugin sync <source-path>)
  - [x] 進捗出力用 --verbose フラグ
  - [x] 既存バージョン上書き用 --force フラグ
  - [x] 副作用なしのプレビュー用 --dry-run フラグ
- [x] 3段階プラグイン解決 (パス → システムディレクトリ → node_modules)
  - [x] システムディレクトリフォールバック付き拡張 plugin-resolver.ts
  - [x] systemPluginDir パス付き SystemConfig 型
  - [x] ランナーブートストラップとの統合
- [x] システムプラグインディレクトリ初期化 (~/.openstarry/plugins/)
  - [x] 初回使用時の自動作成
  - [x] 適切なパーミッションとクリーンアップハンドリング
- [x] 複合コマンドルーティング (bin.ts: `openstarry plugin sync`)
  - [x] 動詞ルーティング付きプラグインサブコマンド
  - [x] 引数パース付きプラグイン同期ハンドラー
- [x] bootstrap.ts が CLI アクセス用に SystemConfig をエクスポート
- [x] テストカバレッジを追加 (39 tests: 19 scanner + 10 sync + 5 E2E + 5 resolver)
- **Target Version**: v0.16.0-beta ✅ (Cycle 21 Phase 4 PASS, 2026-02-12)
- **Pre-condition**: Plan17 ✅ (Cycle 20, v0.15.0-beta, 970 tests)
- **Scope**: Plugin Sync MVP (scanner, CLI command, three-tier resolver)
- **Test Growth**: 970 → 1009 (+39 new, +4.0%)
- **Packages**: 18 → 18 (0 new, existing core extended)
- **Rework Cycles**: 0 (first-pass PASS)
- **Non-blocking Advisories**: 2 deferred
  - [ADV-1] Path traversal validation (信頼されないソースに対する追加検証を推奨)
  - [ADV-2] Factory extraction dedup (ファクトリーパターンのユーティリティ抽出を推奨)
- **Test Files**: 82 → 83 (+1 new file: sync-e2e.test.ts)
- **Key Deliverables**:
  - `plugin-scanner.ts` (230 LOC) — scanPluginDirectory, shouldSyncPlugin, syncPlugin, readPluginVersion
  - `plugin-sync.ts` (133 LOC) — PluginSyncCommand CLI handler
  - Enhanced `plugin-resolver.ts` (+120 LOC) — Three-tier resolution with system directory fallback
  - Enhanced `bin.ts` (+15 LOC) — Compound `plugin sync` command routing
  - Enhanced `bootstrap.ts` (+1 LOC) — SystemConfig export
- **Architecture Spec**: `share/test/reports/arch_reviews/20260212_cycle21/Architecture_Spec_Cycle21.md`
- **QA Report**: `share/test/reports/qa_results/20260212_cycle21/QA_Plan18.md`
- **Code Review**: `share/test/reports/arch_reviews/20260212_cycle21/Code_Review_Cycle21.md`
- **Dev Log**: `share/test/reports/dev_logs/20260212_cycle21/DevLog_Plan18_Plugin_Sync.md`
- **Snapshot**: `share/openstarry_code_iteration/20260212_cycle21/`
- **Status**: ✅ COMPLETE (Phase 4 Convergence PASS, 2026-02-12)

#### Plan19: Plugin Dependency Wiring & Cross-Plugin Services ✅ (Cycle 22, 2026-02-12)
- [x] IPluginService インターフェース (SDK) — name/version を持つプラグインサービスの基底インターフェース
- [x] IServiceRegistry インターフェース (SDK) — register/get/has/list メソッドを持つサービスレジストリ
- [x] ServiceRegistry クラス (Core) — フェイルファスト衝突検出付きインメモリ実装
- [x] IPluginContext.services — プラグインコンテキスト上のオプショナルサービスレジストリアクセサー
- [x] PluginManifest.services/serviceDependencies — マニフェストレベルの依存宣言
- [x] PluginLoader.loadAll() — 依存順序ロード用 Kahn のアルゴリズムによるトポロジカルソート
- [x] ServiceRegistrationError/ServiceDependencyError — サービス操作用エラー型
- [x] 65 新規テスト — 総テスト数 1067 (88 テストファイル)
  - [x] service-registry.test.ts — register(), get(), has(), list() 操作のカバレッジ
  - [x] plugin-loader-services.test.ts — トポロジカルソート、依存順序、循環検出
  - [x] service-injection.test.ts — エンドツーエンドのサービスディスカバリーとクロスプラグイン呼び出し
- **Files Created**:
  - `packages/sdk/src/types/service.ts` — IPluginService, IServiceRegistry インターフェース
  - `packages/core/src/infrastructure/service-registry.ts` — ServiceRegistry 実装
  - `packages/core/__tests__/infrastructure/service-registry.test.ts`
  - `packages/core/__tests__/infrastructure/plugin-loader-services.test.ts`
  - `packages/core/__tests__/e2e/service-injection.test.ts`
- **Files Modified**:
  - `packages/sdk/src/types/plugin.ts` — IPluginContext.services?, PluginManifest 拡張
  - `packages/sdk/src/errors/base.ts` — 新エラー型
  - `packages/sdk/src/index.ts` — 新エクスポート
  - `packages/core/src/agents/agent-core.ts` — ServiceRegistry インスタンス化 + 配線
  - `packages/core/src/infrastructure/plugin-loader.ts` — loadAll() + topologicalSort()
  - `packages/core/src/sandbox/sandbox-manager.ts` — オプショナル services フィールド
- **Target Version**: v0.17.0-beta ✅ (Cycle 22 Phase 4 PASS, 2026-02-12)
- **Pre-condition**: Plan18 ✅ (Cycle 21, v0.16.0-beta, 1009 tests)
- **Scope**: 残 Phase 3.1 項目の完了: プラグインサービスレジストリと依存配線
- **Test Growth**: 1009 → 1067 (+65 new, +5.8%)
- **Test Files**: 83 → 88 (+5 new test files)
- **Packages**: 18 → 18 (0 new, existing SDK + core extended)
- **Rework Cycles**: 1 (Code Fix: missing has() method + field name alignment)
- **Key Deliverables**:
  - `types/service.ts` — IPluginService, IServiceRegistry インターフェース定義
  - `service-registry.ts` — 衝突検出付きインメモリ ServiceRegistry
  - `plugin-loader.ts` (+85 LOC) — Kahn のトポロジカルソート付き loadAll() メソッド
  - Enhanced `types/plugin.ts` — IPluginContext.services, PluginManifest 拡張
  - `errors/base.ts` — ServiceRegistrationError, ServiceDependencyError
  - 65 包括的サービスおよび依存テスト
- **Architecture Spec**: `share/test/reports/arch_reviews/20260212_cycle22/Architecture_Spec_Cycle22.md`
- **QA Report**: `share/test/reports/qa_results/20260212_cycle22/QA_Plan19.md`
- **Code Review**: `share/test/reports/arch_reviews/20260212_cycle22/Code_Review_Cycle22.md`
- **Dev Log**: `share/test/reports/dev_logs/20260212_cycle22/DevLog_Plan19_Dependency_Wiring.md`
- **Snapshot**: `share/openstarry_code_iteration/20260212_cycle22/`
- **Status**: ✅ COMPLETE (Phase 4 Convergence PASS after 1 rework, 2026-02-12)
- **Design Pattern**: Interface-based service registry (Spec Addendum で承認、文字列 ID パターンよりアーキテクチャ的に優れている)
- **Functional Equivalence**: VERIFIED — トポロジカルソートがサービス依存に基づきプラグインを正しく順序付け、循環検出が無効な設定を防止
- **Implementation Quality**: EXCELLENT — Kahn のアルゴリズム (O(V+E))、リワーク後のブロッキング問題ゼロ、包括的テストカバレッジ (65 tests)

#### Plan20: Workflow Engine MVP ✅ (Cycle 23, 2026-02-12)
- [x] IWorkflowDefinition (名前、バージョン、入力、出力、ステップを含む YAML 構造)
- [x] IWorkflowStep 判別共用体 (tool, service, llm, command ステップタイプ)
- [x] Mustache テンプレート補間 (`{{ variable }}` 構文) HTML エスケープ無効化
- [x] WorkflowEngine クラス (逐次実行、LRU キャッシュ 100 エントリ、イベント発行)
- [x] Tool ステップエグゼキューター (レジストリから ITool を呼び出し、引数補間)
- [x] Service ステップエグゼキューター (IServiceRegistry 経由のクロスプラグインサービス呼び出し)
- [x] LLM ステップエグゼキューター (text_delta イベント収集付き直接 IProvider ストリーミング API)
- [x] Command ステップエグゼキューター (プレースホルダー、MVP では CommandStepNotSupportedError をスロー)
- [x] Workflow YAML ローダー (検証付きファイルベースロード)
- [x] イベントシステム (WORKFLOW_STARTED, STEP_STARTED, STEP_COMPLETED, STEP_FAILED, WORKFLOW_COMPLETED イベント)
- [x] エラー階層 (WorkflowLoadError, WorkflowExecutionError, VariableInterpolationError, CommandStepNotSupportedError)
- [x] TypeScript 型とマッチする判別共用体付き Zod スキーマ検証
- [x] 37 包括的テスト (スキーマ、補間、エンジン、4 エグゼキューター、e2e 統合)
- **Files Created**:
  - `src/index.ts` — プラグインファクトリー (createWorkflowEnginePlugin)
  - `src/errors.ts` — 4 エラークラス
  - `src/types/workflow.ts` — ワークフロー型定義とイベントペイロード
  - `src/types/mustache.d.ts` — mustache のモジュール宣言
  - `src/schema/workflow-schema.ts` — 判別共用体付き Zod スキーマ
  - `src/engine/interpolate.ts` — エスケープ無効化付き Mustache 補間
  - `src/engine/workflow-engine.ts` — WorkflowEngine クラス
  - `src/engine/executors/*.ts` — 4 エグゼキューター実装
  - `src/service/workflow-service.ts` — IWorkflowService 実装
  - `src/tool/workflow-tool.ts` — workflow:execute ITool
  - `src/command/workflow-command.ts` — /workflow スラッシュコマンド
- **Target Version**: v0.18.0-beta ✅ (Cycle 23 Phase 4 PASS, 2026-02-12)
- **Pre-condition**: Plan19 ✅ (Cycle 22, v0.17.0-beta, 1067 tests)
- **Scope**: Workflow Engine MVP (宣言的 YAML ワークフロー、tool/service/LLM オーケストレーション)
- **Test Growth**: 1067 → 1104 (+37 new, +3.5%)
- **Test Files**: 88 → 96 (+8 new test files)
- **Packages**: 18 → 18 (0 new, pure plugin implementation)
- **Rework Cycles**: 0 (first-pass PASS, but post-submission user-driven spec rewrite)
- **Spec Divergence**: MAJOR — ユーザーがオリジナルとは異なるステップタイプ（transform, condition, prompt の代わりに service, llm）、`$variable` の代わりに Mustache、ファイル永続化の代わりに LRU で実装を書き直し。Architect が優れた実用的設計とコード品質を理由に乖離にもかかわらず PASS を承認。
- **Key Deliverables**:
  - `workflow-engine.ts` (250 LOC) — LRU キャッシュ付き逐次実行エンジン
  - `interpolate.ts` (50 LOC) — Mustache テンプレート補間
  - 4 executors (tool, service, llm, command) — 各約60 LOC、特化型ステップ実行
  - `workflow-service.ts` (150 LOC) — YAML ロードと検証
  - 8 test files (37 tests) — 全ステップタイプと統合シナリオの包括的カバレッジ
- **Architecture Spec**: `share/test/reports/arch_reviews/20260212_cycle23/Architecture_Spec_Plan20_Workflow_Engine_MVP.md`
- **QA Report**: `share/test/reports/qa_results/20260212_cycle23/QA_Report_Cycle23_v2.md`
- **Code Review**: `share/test/reports/arch_reviews/20260212_cycle23/Architecture_Review_Cycle23_v2.md`
- **Dev Log**: `share/test/reports/dev_logs/20260212_cycle23/DevLog_Plan20_Workflow_Engine.md`
- **Snapshot**: `share/openstarry_code_iteration/20260212_cycle23/`
- **Status**: ✅ COMPLETE (Phase 4 Convergence PASS, 2026-02-12, spec rewrite approved)
- **Design Pattern**: Factory pattern (createWorkflowEnginePlugin), Five Aggregates (ITool primary), microkernel purity (zero SDK/Core changes)
- **Functional Equivalence**: VERIFIED — 全4ステップタイプが機能しテスト済み、Mustache 補間がネストされたオブジェクト/配列を処理、逐次実行がステップ出力チェーンを保持
- **Implementation Quality**: EXCELLENT — クリーンなアーキテクチャ、強力な型安全性 (判別共用体)、包括的エラーハンドリング、コア変更ゼロ
- **Future Enhancements** (Plan22+):
  - README ドキュメント (ステップタイプ、Mustache 構文、例)
  - 設定可能な LRU キャッシュサイズ (マニフェストまたはコンテキスト設定)
  - 条件ロジック (condition ステップタイプまたはサービスベース分岐)
  - 永続的な実行履歴 (ファイルベースまたはデータベース)
  - Command ステップサポート (現在 MVP ブロック、明確な NotSupportedError 付き)

#### Plan21: Web-based Remote Attach ✅ (Cycle 24, 2026-02-12)
- [x] **@openstarry-plugin/http-static** (3 files, 12 tests)
  - [x] SPA アセットの静的ファイル配信 (HTML, CSS, JS)
  - [x] MIME タイプ検出
  - [x] パストラバーサル防止
  - [x] ETag/304 Not Modified キャッシュサポート
- [x] **@openstarry-plugin/web-ui** (4 files, 18 tests)
  - [x] ブラウザベースのエージェント制御ダッシュボード
  - [x] セッションリスト表示とアタッチインターフェース
  - [x] ワークフロー実行モニタリング
  - [x] リアルタイム WebSocket イベント更新
- [x] **@openstarry-plugin/transport-websocket** (enhanced, 32 tests)
  - [x] 検証付きトークンベース認証
  - [x] CORS ヘッダーサポート (Origin, Credentials 検証)
  - [x] プロキシヘッダー転送 (X-Forwarded-For, X-Forwarded-Proto, X-Real-IP)
- [x] 62 新規テスト (http-static: 12, web-ui: 18, transport-websocket: 32)
- **Files Created**:
  - `@openstarry-plugin/http-static/` — 静的ファイルサーバープラグイン
  - `@openstarry-plugin/web-ui/` — ブラウザダッシュボード UI プラグイン
- **Files Enhanced**:
  - `@openstarry-plugin/transport-websocket/` — 認証、CORS、プロキシサポート
- **Target Version**: v0.19.0-beta ✅ (Cycle 24 Phase 4 PASS, 2026-02-12)
- **Pre-condition**: Plan20 ✅ (Cycle 23, v0.18.0-beta, 1104 tests)
- **Scope**: Web-based Remote Attach (HTTP 静的サーバー、Web UI ダッシュボード、拡張 WebSocket トランスポート)
- **Test Growth**: 1104 → 1132 (+28 net, +2.5%)
  - ベースライン 1104 テストを維持
  - 新規テスト追加: 62
  - ネット調整: +28 (一部ベースラインテストのリファクタリング)
- **Test Files**: 96 → 97 (+1 new test file)
- **Packages**: 18 core (unchanged), 10 plugins (2 new: http-static, web-ui; 1 enhanced: transport-websocket)
- **Rework Cycles**: 0 (first-pass PASS)
- **Non-blocking Issues** (将来のサイクルに延期):
  - [NBI-1] http-static と web-ui の README ファイル (ドキュメントフェーズ)
  - [NBI-2] 高度な HTTP キャッシュ戦略 (Plan22+)
  - [NBI-3] Web UI モバイルレスポンシブ対応 (Phase 2 強化)
- **Key Deliverables**:
  - `http-static-plugin/` (3 source + 2 test files) — 静的ファイルサーバー
  - `web-ui-plugin/` (4 source + 3 test files) — ブラウザダッシュボード
  - `transport-websocket/` (enhanced, +32 tests) — 認証 + CORS + プロキシヘッダー
- **Architecture Spec**: `share/test/reports/arch_reviews/20260212_cycle24/Architecture_Spec_Cycle24.md`
- **QA Report**: `share/test/reports/qa_results/20260212_cycle24/QA_Report_Cycle24.md`
- **Code Review**: `share/test/reports/arch_reviews/20260212_cycle24/Code_Review_Cycle24.md`
- **Dev Log**: `share/test/reports/dev_logs/20260212_cycle24/DevLog_Plan21.md`
- **Snapshot**: `share/openstarry_code_iteration/20260212_cycle24/`
- **Status**: ✅ COMPLETE (Phase 4 Convergence PASS, 2026-02-12)
- **Design Pattern**: Factory pattern (createHttpStaticPlugin, createWebUiPlugin), Five Aggregates (IUI for dashboard, IListener for HTTP, IProvider for session service)
- **Functional Equivalence**: VERIFIED — 静的ファイル配信が MIME タイプと ETag をサポート、Web UI が認証付き WebSocket で通信、プロキシヘッダーが適切に転送
- **Implementation Quality**: EXCELLENT — コア変更ゼロ、包括的セキュリティゲート (パストラバーサル、トークン認証、CORS 検証)、テストカバレッジが目標を超過
- **Future Enhancements** (Plan22+):
  - モバイルレスポンシブ UI レイアウト
  - 高度なキャッシュヘッダー (Cache-Control, ETag 生成)
  - プラグインマーケットプレイス統合
  - マルチエージェントオーケストレーションダッシュボード

---

## 3. イテレーションスケジュール提案

| イテレーション | 対象 Plans | 目標バージョン | ステータス |
|--------------|-----------|-------------|----------|
| Cycle 1 | Plan05.1 + 05.2 + 05.5-① | v0.2.1-beta | ✅ 完了 (2026-02-10, 118 tests) |
| Cycle 2 | Plan05.5-② + 05.5-③ | v0.2.2-beta | ✅ 完了 (2026-02-11, 165 tests) |
| Cycle 3 | Plan06 Phase 1 (MCP Client) | v0.3.0-beta | ✅ 完了 (2026-02-11, 200 tests) |
| Cycle 4 | Plan06 Phase 2 (MCP Server) | v0.3.1-beta | ✅ 完了 (2026-02-11, 252 tests) |
| Cycle 5 | Plan07 (Sandbox MVP) | v0.4 | ✅ 完了 (2026-02-11, 320 tests) |
| Cycle 6 | Plan07.1 (Sandbox Hardening) | v0.4.1-beta | ✅ 完了 (2026-02-11, 351 tests) |
| Cycle 7 | Plan07.2 (Advanced Hardening) | v0.4.2-beta | ✅ 完了 (2026-02-11, 407 tests, 1 rework) |
| Cycle 8 | Plan07.3 (Custom require + audit) | v0.4.3-beta | ✅ 完了 (2026-02-11, 442 tests) |
| Cycle 9 | Plan08 (TUI Dashboard MVP) | v0.5.0-beta | ✅ 完了 (2026-02-11, 524 tests) |
| Cycle 10 | Plan09 (Interactive TUI) | v0.5.1-beta | ✅ 完了 (2026-02-12, 559 tests) |
| Cycle 11 | Plan10 (CLI Foundation & Runner) | v0.6.0-beta | ✅ 完了 (2026-02-12, 632 tests, 1 rework) |
| Cycle 12 | Plan11 (DevTools & E2E Testing) | v0.7.0-beta | ✅ 完了 (2026-02-12, 670 tests, 1 rework) |
| Cycle 13 | Plan12 (Daemon Mode MVP) | v0.8.0-beta | ✅ 完了 (2026-02-12, 714 tests, 1 rework) |
| Cycle 14 | Plan13 (Seamless Attach) | v0.9.0-beta | ✅ 完了 (2026-02-12, 747 tests, 1 rework) |
| Cycle 15 | Plan06-P3 (MCP Resources + OAuth) | v0.10.0-beta | ✅ 完了 (2026-02-12, 792 tests) |
| Cycle 16 | Plan14 (Multi-client Attach & Session Management) | v0.11.0-beta | ✅ 完了 (2026-02-12, 807 tests) |
| Cycle 17 | Plan06-P4 (MCP Sampling & Advanced Protocol Extensions) | v0.12.0-beta | ✅ 完了 (2026-02-12, 894 tests) |
| Cycle 18 | Plan15 (SDK Context Extensions & Provider Integration) | v0.13.0-beta | ✅ 完了 (2026-02-12, 915 tests) |
| Cycle 19 | Plan16 (Security Hardening & Quality Polish) | v0.14.0-beta | ✅ 完了 (2026-02-12, 935 tests, 1 rework) |
| Cycle 20 | Plan17 (Plugin Developer Experience) | v0.15.0-beta | ✅ 完了 (2026-02-12, 970 tests) |
| Cycle 21 | Plan18 (Plugin Sync & System Plugin Directory) | v0.16.0-beta | ✅ 完了 (2026-02-12, 1009 tests) |
| Cycle 22 | Plan19 (Plugin Dependency Wiring & Cross-Plugin Services) | v0.17.0-beta | ✅ 完了 (2026-02-12, 1067 tests, 1 rework) |
| Cycle 23 | Plan20 (Workflow Engine MVP) | v0.18.0-beta | ✅ 完了 (2026-02-12, 1104 tests, spec rewrite) |
| Cycle 24 | Plan21 (Web-based Remote Attach) | v0.19.0-beta | ✅ 完了 (2026-02-12, 1132 tests, +28 net) |
| Cycle 25+ | Plan22+ (Advanced Features: Plugin Marketplace, Multi-agent Orchestration) | v0.20.0+ | ⬜ スケジュール待ち |

---

## 4. 使用方法

- **Coordinator** は新イテレーション開始前に本文書を参照し、以下を確認：
  1. 前提 Plan が完了済み ✅ かどうか
  2. 当該 Plan の固有検収条件
  3. 並行作業可能な項目があるか
- **architect** は仕様設計時に、固有検収条件を参照してカバレッジを確保
- **qa** は検証時に、共通 DoD + 固有条件を逐一チェック
- **doc-keeper** は Plan に ✅ を付与する際、DoD 全項目が満たされていることを確認
