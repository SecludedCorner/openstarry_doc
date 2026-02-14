# 00. プロジェクトロードマップとマイルストーン (Project Roadmap and Milestones)

本文書は OpenStarry のコアプロトタイプから完全なエコシステムへの実装パスを定義する。この計画はプロジェクト進化の秩序を確保し、オープンソースコントリビューターに明確な指針を提供することを目的とする。

---

## バージョンマイルストーン概覧 (Version Milestones)

| バージョン | 内容 | ステータス |
|-----------|------|----------|
| v0.1.0-alpha | MVP 基礎アーキテクチャ (Plan01-04) | ✅ 完了 |
| v0.2.0-beta | マルチチャネルトランスポート — WebSocket + HTTP (Plan05) | ✅ 完了 |
| v0.2.1-beta | Session 隔離 + SSE + Health Check (Plan05.1, 05.2, 05.5-①) | ✅ 完了 (Cycle 1, 2026-02-10) |
| v0.2.2-beta | Metrics/Logging + Error Handling (Plan05.5-②③) | ✅ 完了 (Cycle 2, 2026-02-11) |
| v0.3.0-beta | MCP プロトコル統合 (Plan06) | ✅ 完了 (Cycle 3, 2026-02-11) |
| v0.3.5-beta | 管理層基盤構築 (Management Zone Infra) | 計画中 |
| v0.4.0-beta | Runtime Sandbox MVP (Plan07) | ✅ 完了 (Cycle 5, 2026-02-11) |
| v0.4.1-beta | Sandbox Restart + Worker Pool (Plan07.1) | ✅ 完了 (Cycle 6, 2026-02-11) |
| v0.4.2-beta | Import Analysis + PKI Verification (Plan07.2) | ✅ 完了 (Cycle 7, 2026-02-11) |
| v0.4.3-beta | Audit Logging + Module._load (Plan07.3) | ✅ 完了 (Cycle 8, 2026-02-11) |
| v0.5.0-beta | TUI Dashboard MVP (Plan08) | ✅ 完了 (Cycle 9, 2026-02-11) |
| v0.6.0-beta | DevTools Plugin + E2E Testing (Plan11) | ✅ 完了 (Cycle 12, 2026-02-12) |
| v0.7.0-beta | DevTools Debugging + E2E Framework | ✅ 完了 (Cycle 12, 2026-02-12) |
| v0.8.0-beta | Daemon Mode MVP (Plan12) | ✅ 完了 (Cycle 13, 2026-02-12) |
| v0.9.0-beta | Seamless Attach (Plan13) | ✅ 完了 (Cycle 14, 2026-02-12) |
| v0.10.0-beta | MCP Resources + OAuth 2.1 (Plan06-P3) | ✅ 完了 (Cycle 15, 2026-02-12) |
| v0.11.0-beta | Multi-client Attach & Session Management (Plan14) | ✅ 完了 (Cycle 16, 2026-02-12) |
| v0.12.0-beta | MCP Sampling & Advanced Protocol Extensions (Plan06-P4) | ✅ 完了 (Cycle 17, 2026-02-12) |
| v0.13.0-beta | SDK Context Extensions & Provider Integration (Plan15) | ✅ 完了 (Cycle 18, 2026-02-12) |
| v0.14.0-beta | Security Hardening & Quality Polish (Plan16) | ✅ 完了 (Cycle 19, 2026-02-12) |
| v0.15.0-beta | Plugin Developer Experience (Plan17) | ✅ 完了 (Cycle 20, 2026-02-12) — 970 tests |
| v0.16.0-beta | Plugin Sync & System Plugin Directory (Plan18) | ✅ 完了 (Cycle 21, 2026-02-12) — 1009 tests |
| v0.17.0-beta | Plugin Dependency Wiring & Cross-Plugin Services (Plan19) | ✅ 完了 (Cycle 22, 2026-02-12) — 1067 tests |
| v0.18.0-beta | Workflow Engine MVP (Plan20) | ✅ 完了 (Cycle 23, 2026-02-12) — 1104 tests |
| v0.19.0-beta | Web-based Remote Attach (Plan21) | ✅ 完了 (Cycle 24, 2026-02-12) — 1132 tests |
| v0.20.0-beta | Plugin Marketplace MVP (Plan22) | ✅ 完了 (Cycle 25, 2026-02-13) — 1330 tests |
| v0.20.1-beta | Windows クロスプラットフォーム修正 + Attach UX 改善 (Hotfix) | ✅ 完了 (Cycle 25-26, 2026-02-13~14) — 1339 tests |

---

## Phase 1: 創世記 (Genesis)
**目標：** Monorepo 物理基盤と共通規約の確立。
**ステータス：** 進行中 (In Progress) — 1.3 CI/CD 未構築

- [x] **1.1 Monorepo 初期化**
    - [x] `apps/` と `packages/` ディレクトリ構造の構築。
    - [x] `pnpm-workspace.yaml` とルートディレクトリ `package.json` の設定。
    - [x] TypeScript、ESLint、Prettier 規約の統一。
- [x] **1.2 コア SDK と型定義**
    - [x] `packages/sdk` を構築し、完全な**五蘊プラグインインターフェース**を定義。
        *   **色 (IUI)**: インターフェースと表現 (Form/UI)。
        *   **受 (IListener)**: 感覚リスニング (Perception/Input)。
        *   **想 (IProvider)**: 思考生成 (Cognition/LLM)。
        *   **行 (ITool)**: ツール実行 (Action/Will)。
        *   **識 (IGuide)**: 魂とペルソナ (Consciousness/Persona)。
    - [x] `packages/shared` を構築し、グローバルエラーハンドリングとログモジュールを提供。
- [ ] **1.3 エンジニアリングとテスト基盤 (Engineering & Testing Infrastructure)**
    - [x] ユニットテストフレームワーク (Vitest) の設定。 *(Plan02 Phase C — 56 テスト通過)*
    - [x] 純粋性検査スクリプト (`pnpm test:purity`)。 *(Plan03 Phase A4)*
    - [ ] CI/CD プロセス (GitHub Actions) の構築。

## Phase 2: 意識カーネル (The Conscious Kernel)
**目標：** 「ヘッドレス (Headless)」でイベント駆動の `Agent Core` の実現。
**ステータス：** 完了 (Done)

- [x] **2.1 実行ループ (The Loop)**
    - [x] `packages/core/execution` の実装 (イベントキューベースの tick メカニズム)。
    - [x] ExecutionLoop イベント駆動リファクタリング：EventQueue から pull してイベント取得、状態機械に WAITING_FOR_EVENT / SAFETY_LOCKOUT を含む。 *(Plan02 Phase A)*
    - [x] InputEvent payload の標準化（source, inputType, data, replyTo）。 *(Plan02 Phase A4)*
- [x] **2.2 状態とメモリ管理**
    - [x] `packages/core/state` の実装 (Snapshot と永続化インターフェースのサポート)。
    - [x] `packages/core/memory` の実装 (プラガブルなコンテキスト戦略、例：スライディングウィンドウ)。
- [x] **2.3 安全ブレーカーとエラーフィードバック (Safety & Correction)**
    - [x] Token 消費上限と無限ループ検出の実装 (Circuit Breakers)。 *(Plan02 Phase B — SafetyMonitor)*
    - [x] 「エラーは痛覚」メカニズムの実装。実行時エラーを Context フィードバックに変換し、Agent の自己修正をトリガー。 *(Plan02 Phase B — 挫折カウンター + 重複呼び出し検出 + エラーカスケードブレーカー)*

## Phase 3: 肢体と感覚 (Body and Senses)
**目標：** 完全な五蘊プラグインインフラと標準ライブラリの構築、セキュリティ境界の確立。
**ステータス：** 進行中 (In Progress) — コアプラグイン完了、先進機能は未実装

- [x] **3.1 コアロードプロトコル (Loading Protocol)**
    - [x] `PluginLoader` の実装、**ファクトリーパターン (Factory Pattern)** 初期化をサポート。 *(Plan01)*
    - [x] `IPluginContext` の実装：Logger, Config、および**クロスプラグインサービスインジェクション (dependencies フィールド)** を含む。 *(Plan19)*
    - [x] **ループ編成ロジック (Dependency Wiring) の実装:** `20` 号文書に基づき、ロード時に OODA ループを自動接続。 *(Plan19 — topologicalSort, cycle detection, AgentCore.serviceRegistry)*
    - [x] **コマンドレジストリ (CommandRegistry) の実装:** CLI `run-tool` と Chat `/slash` コマンドのマッピングと実行ロジックをサポート。 *(Plan01)*
    - [x] **動的プラグインロード (Dynamic Loading) の実装:** CLI が `agent.json` の `plugins[].path` フィールドをサポートし、`import()` で動的にサードパーティプラグインをロード。 *(Plan02 Phase A3)*
    - [x] `agent.json` のランタイムバリデーター (Zod スキーマ検証) の実装。 *(v0.1.5 - Plan03 補強完了)*
- [ ] **3.2 調整レイヤーとレジストリメカニズム (Coordination Layer)**
    - [x] **デュアルパススキャンメカニズム (Dual-Path Scanning):** システムとプロジェクトプライベートプラグインディレクトリの同時スキャン。
    - [ ] **ワンクリック同期 (Sync):** `openstarry plugin sync` コマンドの実装。公式 **`openstarry_plugin`** リポジトリの内容をシステムディレクトリに同期。
    - [x] Daemon での **Plugin Registry Service** の実装、インメモリインデックスの構築。
- [ ] **3.2 標準機能集約プラグインライブラリ (Standard Aggregate Plugins)**
    > **開発場所：** `openstarry_plugin` リポジトリ (Ecosystem Repo)
    - [x] `@openstarry-plugin/standard-function-stdio`: 受(Stdio Listener) + 色(CLI Body) + 識(Default Guide) を集約。 *(Plan01)*
    - [x] `@openstarry-plugin/provider-gemini-oauth`: 想(Gemini Provider) を集約。PKCE + OAuth 2.0 認証。 *(Plan01)*
    - [x] `@openstarry-plugin/standard-function-fs`: 行(FS Tools) + 識(Path Scoping Policy) を集約。 *(Plan01)*
    - [ ] `@openstarry-plugin/guide-mcp`: 識(MCP Guide) を集約。標準化通信能力の付与。
    - [ ] `@openstarry-plugin/guide-pain-mechanism`: 擬人化痛覚解釈ロジックの実装。
    - [x] `@openstarry-plugin/standard-function-skill`: 識(Skill Guide) を集約。**コア依存**：Markdown スキルファイルのパースを担当し、ワークフローと複雑な Agent の基盤。 *(Plan03 Phase B1)*
- [ ] **3.4 プラグイン開発体験 (DX)**
    - [ ] `openstarry create-plugin` スキャフォールディングの実装。
    - [ ] `MockHost` テスト環境の提供。プラグインの独立開発と検証をサポート。

> **セキュリティ注記：**
> Phase 3 の `fs` ツールは Docker 内で実行されないが、`packages/shared` のパス正規化コンポーネントを通じて操作範囲を厳格に制限する必要がある。これにより Agent が「フォルダの作成/削除」等の必要な操作を実行しつつ、システムの重要ディレクトリを保護可能。

## Phase 4: 誕生 (The First Breath)
**目標：** 初のスタンドアロン CLI Agent の実現、エンドツーエンドフローの検証。
**ステータス：** 進行中 (In Progress) — Runner は実行可能、エンドツーエンド LLM 検証は OAuth ログイン後のテスト待ち

- [x] **4.1 ローカルランナー (Local Runner)**
    - [x] `apps/runner` ブートストラッププログラムの実装（純粋な起動 Bootstrap Runner）、`agent.json` 起動をサポート。 *(Plan01)*
    - [x] 動的プラグイン解決のサポート（path / node_modules 二層戦略、BUILTIN_FACTORIES は削除、全プラグインが動的ロード）。 *(Plan02 Phase A3)*
- [ ] **4.2 統合検証**
    - [ ] 「感知 -> 思考 -> ツール呼び出し -> 修正 -> 応答」の完全なクローズドループの達成。*(OAuth ログイン後の手動テストが必要)*

> **マイルストーン：v0.1 Alpha (MVP)**
> *   ✅ CLI で単一 Agent を実行可能。
> *   ✅ Gemini を Provider として使用（PKCE + OAuth 認証）。
> *   ✅ ローカルファイルシステム操作可能 (`fs` tool)。
> *   ✅ 基本記憶能力を具備 (5 ターンの対話)。
> *   ✅ 安全ブレーカーメカニズム（Token 予算、ループ上限、エラーカスケード、挫折カウンター）。
> *   ✅ 構造化 JSON ログ（LOG_FORMAT=json, LOG_LEVEL フィルタリング）。
> *   ✅ agent.json Zod ランタイム検証（設定エラーの即時報告）。
> *   ✅ ツール呼び出し Timeout（Promise.race でハング防止）。
> *   ✅ TraceID メカニズム（ログで完全な処理サイクルを追跡）。
> *   ✅ Markdown スキルロード（standard-function-skill プラグイン）。
> *   ✅ guideFile 外部ファイル参照（system_prompt を .md からロード）。
> *   ✅ 純粋性検査スクリプト（pnpm test:purity）。
> *   ✅ 56 ユニットテスト通過（Vitest）。
> *   ⬜ エンドツーエンド LLM 通話検証（手動テスト待ち）。

---

> **Agent コアの完了。以下は「エージェントオペレーティングシステム」とエコシステムフェーズ。**

---

## Phase 4.5：マルチチャネルトランスポート (Multi-Channel Transport)
**目標：** IUI/IListener 分離アーキテクチャの拡張性を検証。非 stdio チャネルの実装。
**ステータス：** 完了 (v0.2.0-beta)

- [x] **4.5.1 WebSocket トランスポートプラグイン**
    - [x] `@openstarry-plugin/transport-websocket`: WebSocket Listener + UI
    - [x] マルチクライアント接続、指定返信 (replyTo) のサポート
- [x] **4.5.2 HTTP Webhook トランスポートプラグイン**
    - [x] `@openstarry-plugin/transport-http`: HTTP Listener + UI
    - [x] POST /api/input, GET /api/status, GET /api/response
- [x] **4.5.3 マルチ UI 同時出力**
    - [x] TransportBridge ブロードキャストメカニズムの検証通過
    - [x] stdio + WebSocket が同時にイベントを受信

> **マイルストーン：v0.2.0 Beta (Multi-Channel)**
> *   ✅ WebSocket トランスポートプラグイン実装完了
> *   ✅ HTTP Webhook トランスポートプラグイン実装完了
> *   ✅ マルチ UI 同時出力検証通過

---

## Phase 4.6：実装サイクル1 (v0.2.1-beta)
**目標：** マルチユーザーのプライバシー問題の解決、トランスポート効率の最適化、接続ヘルスチェックの確立。
**ステータス：** 完了 (Cycle 1, 2026-02-10) — 118 tests, QA PASS, Architect PASS WITH NOTES

### Plan05.1: Session 隔離とメッセージルーティング ✅
- [x] ISession / ISessionManager インターフェース (SDK) + SessionManager 実装 (Core)
- [x] InputEvent.sessionId フィールド (オプショナル、後方互換)
- [x] IPluginContext.sessions が ISessionManager をプラグインに公開
- [x] Default session (`__default__`) の自動作成、破棄不可
- [x] WebSocket session-aware routing (セッションごとのブロードキャスト)
- [x] 17 の新規テストで session isolation シナリオをカバー

### Plan05.2: HTTP SSE サポート ✅
- [x] 新規 `GET /api/events` SSE エンドポイント
- [x] EventSource 互換フォーマット (text/event-stream)
- [x] Session-scoped イベントフィルタリング
- [x] SSE heartbeat 定期送信
- [x] 既存の HTTP webhook listener と共存
- [x] 11 の新規テストで SSE シナリオをカバー

### Plan05.5-①: 接続ヘルスチェック ✅
- [x] WebSocket プロトコル ping/pong (server-initiated)
- [x] missedPongs カウント、staleThreshold 超過で切断
- [x] HTTP SSE heartbeat 定期チェック
- [x] HealthCheckConfig 設定可能 (enabled, intervalMs, staleThreshold)
- [x] 6 の新規テストで health check シナリオをカバー

> **マイルストーン：v0.2.1-beta**
> *   ✅ マルチクライアント session 隔離 (WebSocket + HTTP)
> *   ✅ HTTP SSE リアルタイムストリーミング
> *   ✅ 接続ヘルスチェック (ping/pong + heartbeat)
> *   ✅ 118 tests, 11 packages, snapshot saved

---

## Phase 4.7：実装サイクル2 (v0.2.2-beta)
**目標：** 可観測性基盤の構築と標準化エラーハンドリング。
**ステータス：** 完了 (Cycle 2, 2026-02-11) — 165 tests, QA PASS, Architect PASS WITH NOTES

### Plan05.5-②: Metrics / Logging 基盤 ✅
- [x] MetricsCollector (Core): increment / gauge / getSnapshot / reset
- [x] Logger.time() メソッド (performance.now() 計測)
- [x] METRICS_SNAPSHOT イベント型
- [x] /metrics スラッシュコマンド (AgentCore)
- [x] トランスポートプラグイン console.log → createLogger 移行
- [x] 19 の新規テスト

### Plan05.5-③: 標準化エラーハンドリング ✅
- [x] ErrorCode const (12 エラーコード)
- [x] TransportError / SessionError / ConfigError クラス
- [x] ES2022 Error cause chain サポート
- [x] 既存の 2-arg コンストラクタの後方互換性
- [x] 16 の新規テスト

> **マイルストーン：v0.2.2-beta**
> *   ✅ MetricsCollector 可観測性基盤
> *   ✅ 標準化エラー階層 (ErrorCode + cause chain)
> *   ✅ トランスポートプラグインの構造化ログ
> *   ✅ 165 tests, 11 packages, snapshot saved

---

## Phase 4.8：MCP プロトコル統合 (v0.3.0-beta)
**目標：** OpenStarry Agent と外部 MCP Server の標準化相互接続の実現。
**ステータス：** 完了 (Cycle 3, 2026-02-11) — 86 tests, Architect PASS WITH NOTES

### Plan06: MCP Client Plugin ✅
- [x] `@openstarry-plugin/mcp-client` 主要実装
  - [x] 汎用 MCP トランスポート層抽象 (stdio, SSE)
  - [x] StdioClientTransport 実装 (cross-platform: Windows/Unix)
  - [x] SSEClientTransport 実装
  - [x] RPC 通信層 (JSONRPCClient)
- [x] MCP Tool Bridge
  - [x] `mcp:` namespace メカニズム
  - [x] `tools/list` → OpenStarry Tool Registry マッピング
  - [x] `tools/call` 実行と結果返却
- [x] MCP Prompt Bridge
  - [x] スラッシュコマンド (`/mcp-prompt-name`) マッピング
  - [x] `prompts/list` リスト
  - [x] `prompts/get` 完全な prompt 内容の取得
- [x] `@openstarry-plugin/mcp-server` サーバー実装
  - [x] StdioServerTransport 実装
  - [x] MCP Server Protocol 処理 (initialize, tools/list, tools/call, prompts/*)
  - [x] JSON-RPC 応答メカニズム
  - [x] 完全なエラーハンドリング
- [x] `@openstarry-plugin/mcp-common` 共有型
  - [x] MCP Protocol インターフェース定義
  - [x] Transport 抽象
  - [x] RPC 通信型
- [x] 86 ユニットテスト
  - [x] MCP Client: 53 tests
  - [x] MCP Server: 33 tests

> **マイルストーン：v0.3.0-beta**
> *   ✅ MCP Client Plugin 完全実装
> *   ✅ MCP Server Plugin 完全実装
> *   ✅ Stdio と SSE トランスポートサポート
> *   ✅ Tool と Prompt Bridge 相互接続
> *   ✅ 86 tests, 3 新規 packages, snapshot saved

---

## Phase 4.9：Runtime Sandbox (v0.4.0-beta ~ v0.4.3-beta)
**目標：** 完全なランタイムサンドボックスメカニズムの実現。プラグインの安全な隔離と制御を確保。
**ステータス：** 完了 (Cycles 5-8, 2026-02-11) — 442 tests total, QA PASS, Architect PASS WITH NOTES

### Plan07: Runtime Sandbox MVP (Cycle 5) ✅
**目標：** Worker スレッド隔離と基本的な署名検証メカニズムの確立。
- [x] SandboxManager と Worker スレッド管理
  - [x] 単一 Worker の初期化と実行
  - [x] RPC 通信メカニズム (cross-thread MessagePort)
  - [x] Timeout と Memory Limit の強制適用
- [x] 署名検証メカニズム
  - [x] SHA-256 パッケージレベル検証
  - [x] Manifest 署名チェック
  - [x] 検証失敗の明確なエラー報告
- [x] 82 の新規テスト

> **マイルストーン：v0.4.0-beta**
> *   ✅ Worker スレッド隔離実装
> *   ✅ SHA-256 署名検証
> *   ✅ Memory + CPU timeout 制御

### Plan07.1: Worker Lifecycle + Pool (Cycle 6) ✅
**目標：** Worker 再起動戦略、ハートビート監視、接続プール管理の実現。
- [x] Worker 再起動メカニズム
  - [x] 指数バックオフ再起動戦略 (exponential backoff: 100, 200, 400, 800ms → cap)
  - [x] 障害 Worker の自動復旧
- [x] ハートビート監視 (Heartbeat)
  - [x] 定期ハートビートチェック
  - [x] ストール検出と自動隔離
  - [x] Missing heartbeat カウンター
- [x] Worker Pool 管理
  - [x] Worker の事前生成 (pool sizing)
  - [x] 自動回収と再利用
  - [x] プロトコルメッセージによる Plugin reset
- [x] 110 の新規テスト

> **マイルストーン：v0.4.1-beta**
> *   ✅ 指数バックオフ再起動戦略
> *   ✅ ハートビート監視と自動復旧
> *   ✅ Worker Pool 接続プールメカニズム

### Plan07.2: Import Analysis + PKI (Cycle 7) ✅
**目標：** 静的解析防御と Ed25519 PKI 検証の実現。
- [x] 静的インポート分析器 (AST-based)
  - [x] 再帰的 require/import スキャン
  - [x] ホワイトリスト (allowed) / ブラックリスト (blocked) モジュールチェック
  - [x] 違反時のロード拒否 (strict モード)
- [x] Ed25519 PKI 署名検証
  - [x] 公開鍵設定メカニズム
  - [x] Ed25519 署名検証
  - [x] 複数署名者のサポート
- [x] SandboxConfig インターフェース
  - [x] allowed / blocked モジュールリスト
  - [x] publicKeys 設定
  - [x] ランタイムポリシー (strict/warn/off)
- [x] Worker Pool interface (initialize/acquire/release/shutdown)
- [x] 135 の新規テスト

> **マイルストーン：v0.4.2-beta**
> *   ✅ 静的インポート分析防御
> *   ✅ Ed25519 PKI 検証メカニズム
> *   ✅ モジュールブラック/ホワイトリスト制御

### Plan07.3: Audit Logging + Final Hardening (Cycle 8) ✅
**目標：** 監査ログと Module._load インターセプトの実現。サンドボックスハードニングの完了。
- [x] AuditLogger メカニズム
  - [x] バッファリング付き JSONL 書き込み
  - [x] 非同期ログ記録
  - [x] 機密情報隠蔽 (password/token/key/auth/credential)
- [x] Module._load インターセプト
  - [x] ランタイムモジュールロード制御
  - [x] Strict / Warn / Off 3つのモード
  - [x] コールスタック追跡
- [x] RPC 監査統合
  - [x] ツール実行ログ
  - [x] エラー追跡
  - [x] パフォーマンス指標
- [x] ログローテーションとクリーンアップ
- [x] 115 の新規テスト

> **マイルストーン：v0.4.3-beta**
> *   ✅ 完全な監査ログメカニズム
> *   ✅ Module._load ランタイム制御
> *   ✅ 機密情報隠蔽とセキュリティハードニング
> *   ✅ 442 tests (Plan07 全線), snapshot saved

---

## 延期項目 (Deferred Plan05.x)

### Plan05.3: DevTools デバッグインターフェース ⬜
> Plan06 (MCP) 以降に延期して実装。

### Plan05.4: E2E テストスキャフォールディング ⬜
- [ ] `createTestAgent()` ユーティリティ関数
- [ ] MockLLM プラグインのサポート
- [ ] コミュニティ貢献の敷居を低減

---

## Phase 5: ガーディアンデーモン (The Orchestrator Daemon) — Plan07
**目標：** プロセスレベル管理と永続化の実現。
**ステータス：** 完了 (v0.4.3-beta, Cycles 5-8, 2026-02-11)

- [ ] **5.1 デーモンコア (Daemon)**
    - [ ] `apps/daemon` を構築し、複数 Agent インスタンスのライフサイクルを管理。
    - [ ] **Agent 調整管理層 (Management Zone)** のスケジューリングロジックの実装。
- [ ] **5.2 API ゲートウェイと永続化**
    - [ ] HTTP/gRPC API を提供してリモート管理を実現。
    - [ ] データベース (SQLite/LevelDB) を統合し Agent の長期状態を保存。
- [ ] **5.3 セキュリティとガバナンス (Security & Governance)**
    - [ ] プラグイン署名と検証メカニズムの実装（ロードされるプラグインが信頼されていることを確保）。
    - [ ] ランタイムサンドボックス (Sandbox) 戦略の実装（例：Node.js vm または WASM コンテナ化）。
    - [ ] **環境隔離とリソースクォータ管理 (Quota Management) の実装**。
- [ ] **5.4 システム可観測性とハードウェア抽象 (Observability & HAL)**
    - [x] 構造化 JSON ログ、agent_id / trace_id / module フィールドをサポート。 *(Plan02 Phase D)*
    - [ ] OpenTelemetry または Event Tracing を統合し、Agent 間のタスクフローを追跡。
    - [ ] ログ収集を実装し、Dashboard の視覚化表示をサポート。
    - [ ] **ハードウェア抽象層 (HAL) 標準感覚データフローの実装**。

## Phase 6: フラクタル社会 (Fractal Society) — Plan06
**目標：** エージェント協力と MCP プロトコルの実現。
**ステータス：** 完了 (v0.3.0-beta → v0.10.0-beta, Cycle 3 → Cycle 15, 2026-02-11 → 2026-02-12)
**前提条件：** Plan05.1 Session 隔離完了 ✅ (v0.2.1-beta, 2026-02-10)

> **入場チェックリスト：**
> - ✅ Plan05.1 Session 隔離が実装済み (Cycle 1)
> - ✅ マルチクライアントシナリオの検証通過 (118 tests)
> - ✅ メッセージが Session 間で漏洩しない (session-aware routing 検証済み)

- [x] **6.1 MCP 深度統合 (Plan06-P1: Tools)**
    - [x] `@openstarry-plugin/mcp-client` の実装。Agent に標準化相互接続能力を付与。
    - [x] `@openstarry-plugin/mcp-server` の実装。複数の外部クライアントの同時接続をサポート。
    - [x] 86 ユニットテスト検証完了 (v0.3.0-beta, Cycle 3)
- [x] **6.1a MCP Prompts 統合 (Plan06-P2: Prompts)**
    - [x] MCP Prompts RFC 0005 Section 5.2 実装完了
    - [x] `/mcp-prompt-name` スラッシュコマンドサポート
    - [x] 動的プロンプトリストとコンテンツ検索
    - [x] 42 の新規テスト (v0.9.0-beta 以前, Cycle 9+)
- [x] **6.1b MCP Resources 統合 (Plan06-P3: Resources)**
    - [x] MCP Resources RFC 0005 Section 5.5 実装完了
    - [x] listResources / readResource RPC 処理
    - [x] OAuth 2.1 トークン管理 (PKCE, auto-refresh, TTL)
    - [x] AES-256-GCM 暗号化 + PBKDF2 (100k iterations)
    - [x] `/mcp-resources` / `/mcp-server-resources` スラッシュコマンド
    - [x] 45 の新規テスト (v0.10.0-beta, Cycle 15)
- [ ] **6.2 DevTools デバッグインターフェース (Plan05.3)**
    - [ ] 静的 HTML デバッグインターフェース `openstarry-devtools`
    - [ ] 左側：対話バブルビュー
    - [ ] 右側：State/Context 検査
    - [ ] 下部：イベントフィルター
- [ ] **6.3 ワークフローエンジン (Workflow Engine)**
    - [ ] `WorkflowEngineTool` の実装。YAML 定義のタスクオーケストレーションをサポート。
    - [ ] Agent の動的スポーンと破棄メカニズム (Ephemeral Agents) の実装。
- [ ] **6.3 Agent 間協力と因果スケジューリング (Orchestration)**
    - [ ] タスク分解と異なるサブエージェントへの分配メカニズムの実現。
    - [ ] **因果チェーンベースのイベントスケジューリング (Causality Chain) の実装**。

## Phase 7: 視覚化とエコシステム (Ecosystem and UI)
**目標：** Web グラフィカル管理とデプロイツールの提供。
**ステータス：** 未着手

- [ ] **7.1 Dashboard とマルチインタラクション層 (Interface)**
    - [ ] `apps/dashboard` (React/Next.js) を構築し、全 Agent を視覚的に監視。
    - [ ] **状態投影 (State Projection) ダッシュボードの実装**。
- [ ] **7.2 テンプレートサービス (Agent Design Service)**
    - [ ] Agent ストア/テンプレートダウンロード機能の提供。「ディレクトリ即プロトコル」の最小限デプロイを実現。

## Phase 8: ターミナル進化 (CLI & TUI Evolution) — Plan08 / Plan09
**目標：** 「オペレーティングシステムレベル」のターミナル体験の実現。User Scenario Guide のビジョンを達成。
**ステータス：** 進行中 (v0.5.0-beta Plan08 完了、Plan09 未実装)

### Plan08: TUI Dashboard MVP (v0.5.0-beta) ✅
**ステータス：** 完了 (Cycle 9, 2026-02-11) — 524 tests (+82 new), 43 test files, QA PASS, Architect PASS_WITH_NOTES

- [x] **8.1 ランタイムダッシュボード (Runtime Dashboard)**
    - [x] `@openstarry-plugin/tui-dashboard` の実装 (Ink v5 + React 18 使用)
    - [x] イベント監視、状態表示、メッセージストリーミングの統合
    - [x] メッセージ分類、ツール呼び出し追跡、エラーカウントのサポート
    - [x] キーボードショートカット (q = 終了, Tab = イベントログ切替)

### Plan12: Daemon Mode MVP (v0.8.0-beta) ✅
**ステータス：** 完了 (Cycle 13, 2026-02-12) — 714 tests (+44 new), 18 packages, QA PASS, Architect PASS (1 rework)

- [x] **バックグラウンドプロセス管理 (Daemon Process Management)**
    - [x] CLI コマンド：`daemon start`, `daemon stop`, `daemon ps`
    - [x] プロセス生成と分離 (detached process, unref)
    - [x] PID ファイル管理 (`~/.openstarry/agents/{agent-id}.pid`)
    - [x] 優雅なシャットダウン (SIGTERM/SIGINT カスケード)
- [x] **IPC 層 (JSON-RPC over Unix Domain Socket)**
    - [x] ソケット通信：`~/.openstarry/agents/{agent-id}.sock`
    - [x] JSON-RPC 2.0 プロトコル
    - [x] ヘルスチェック RPC：`agent.health` → `{ok, uptime, version}`
- [x] **Daemon プラグイン** (`@openstarry-plugin/daemon`)
    - [x] IPC サーバー統合
    - [x] ヘルスチェックプロバイダー (IProvider)
    - [x] プロセスライフサイクル管理
- [x] **テストカバレッジ**: 44 の新規テスト (プロセス管理、IPC 通信、ヘルスチェック)

> **マイルストーン：v0.8.0-beta**
> *   ✅ Daemon バックグラウンドプロセス実装
> *   ✅ IPC 通信基盤
> *   ✅ 714 tests, 18 packages, snapshot saved

### Plan13: Seamless Attach (v0.9.0-beta) ✅
**ステータス：** 完了 (Cycle 14, 2026-02-12) — 747 tests (+33 new), 18 packages, QA PASS, Architect PASS (1 rework)

- [x] **IPC プロトコル拡張**
    - [x] `agent.attach(agentId)` → ターミナルクライアントを接続、sessionId を返却
    - [x] `agent.input(sessionId, message)` → アタッチされたクライアントからユーザー入力を送信
    - [x] `agent.detach(sessionId)` → ターミナルセッションの優雅なクローズ（daemon は継続運行）
    - [x] イベント転送：Core.bus → IPC ブリッジ、sessionId フィルタリング
- [x] **CLI コマンド** (`openstarry attach [agent-id]`)
    - [x] 実行中の daemon を一覧表示
    - [x] daemon の自動起動（agent.json が存在するが daemon が未実行の場合）
    - [x] ターミナル I/O プロキシ：stdin → agent.input RPC、daemon イベント → stdout/stderr
    - [x] 優雅な Ctrl+C ハンドリング（デタッチ、daemon キルではない）
- [x] **イベントフォワーダー** (`event-forwarder.ts`)
    - [x] セッションフィルタリングされたイベント配信
    - [x] LLM レスポンス、ツール実行、エラー、メトリクスイベントのサポート
    - [x] JSON シリアライゼーション（タイムスタンプ付き）
    - [x] サイズ制限 (64KB メッセージ、1MB イベント)
- [x] **セキュリティと検証**
    - [x] sessionId フォーマット検証 (UUID v4)
    - [x] inputType ホワイトリスト (user, system)
    - [x] Agent ヘルスチェック（アタッチ前に daemon の活力を検証）
    - [x] データサイズの強制適用
- [x] **テストカバレッジ**: 33 の新規テスト (attach コマンド、IPC ハンドラー、イベント転送、I/O プロキシ)

> **マイルストーン：v0.9.0-beta**
> *   ✅ 実行中の daemon へのシームレス接続
> *   ✅ インタラクティブターミナル対話 (attach/input/detach)
> *   ✅ 自動起動とイベント転送
> *   ✅ 747 tests, 18 packages, snapshot saved

### Plan20: Workflow Engine MVP (v0.18.0-beta) ✅
**ステータス：** 完了 (Cycle 23, 2026-02-12) — 1104 tests (+37 new), QA PASS, Architect PASS

- [x] **ワークフローエンジンプラグイン** (`@openstarry-plugin/workflow-engine`)
  - [x] YAML 宣言型マルチステップワークフローオーケストレーション (Zod 検証)
  - [x] 4 ステップエグゼキューター：tool, service, llm, command
  - [x] Mustache テンプレート補間 (`{{ }}` 構文)
  - [x] LLM ストリーミング統合 (直接 IProvider AsyncIterable API)
  - [x] EventBus 監視 (step:start/end, workflow:start/end/error)
  - [x] LRU キャッシュ (インメモリ、最大 100 エントリ)
- [x] **テストカバレッジ**: 37 の新規テスト (8 ファイル)

> **マイルストーン：v0.18.0-beta**
> *   ✅ Workflow Engine MVP 完了
> *   ✅ SDK/Core 変更ゼロ (マイクロカーネル純粋性維持)
> *   ✅ 1104 tests, snapshot saved

### Plan21: Web-based Remote Attach (v0.19.0-beta) ✅
**ステータス：** 完了 (Cycle 24, 2026-02-12) — 1132 tests, QA CONDITIONAL PASS

- [x] **3 つの新規/変更プラグイン**
  - [x] `@openstarry-plugin/http-static` (新規) — 静的 HTTP ファイルサーバー (パストラバーサル防御、MIME 処理)
  - [x] `@openstarry-plugin/web-ui` (新規) — ブラウザエージェントインターフェース (HTML 設定注入、セッション復旧)
  - [x] `@openstarry-plugin/transport-websocket` (変更) — 認証/CORS/プロキシ IP 解析
- [x] **テストカバレッジ**: 147 の新規テスト (6 ファイル)

> **マイルストーン：v0.19.0-beta**
> *   ✅ Web-based Remote Attach 完了
> *   ✅ WebSocket Token 認証 + CORS
> *   ✅ 1132 tests, snapshot saved

### Plan22: Plugin Marketplace MVP (v0.20.0-beta) ✅
**ステータス：** 完了 (Cycle 25, 2026-02-13) — 1330 tests, QA PASS

- [x] **プラグインカタログとインストーラー**
  - [x] プラグインカタログ (`plugin-catalog.json`)：15 の公式プラグインリスト
  - [x] プラグインロックファイル (`~/.openstarry/plugins/lock.json`)：インストール済みプラグインの追跡
  - [x] インストーラー：workspace 優先解決 + npm フォールバック
  - [x] 短縮名サポート (例：`standard-function-fs` → `@openstarry-plugin/standard-function-fs`)
  - [x] 一括インストール (`plugin install --all`)
- [x] **5 つの新規 CLI コマンド**: plugin install/uninstall/list/search/info
- [x] **テストカバレッジ**: 198 の新規テスト (77 marketplace + インフラ)

> **マイルストーン：v0.20.0-beta**
> *   ✅ Plugin Marketplace MVP 完了
> *   ✅ 1330 tests, snapshot saved

### Hotfix: Windows クロスプラットフォーム修正 + Attach UX (v0.20.1-beta) ✅
**ステータス：** 完了 (Cycle 25 hotfix + Cycle 26, 2026-02-13~14) — 1339 tests

- [x] **Windows クロスプラットフォーム修正** (Cycle 25 hotfix)
  - [x] パス処理：`path.sep`、`basename()`、`pathToFileURL()` でハードコード Unix パスを置換
  - [x] Daemon IPC：`platform.ts` — Windows named pipe / Linux Unix socket
  - [x] プラットフォームガード：SIGHUP、chmod、mkdirSync、unlinkSync を Windows でスキップ
  - [x] プラグインインストーラー：`cp()` dereference フォールバック + `rm()` maxRetries
- [x] **Attach UX 改善** (Cycle 26)
  - [x] Attach 接続後に provider ログインステータスを自動表示 (`/provider status`)
  - [x] ウェルカムメッセージに `/help` ヒントを追加
- [x] **Token 永続化修正** (Cycle 26)
  - [x] `provider-gemini-oauth` の `dispose()` が token ファイルを誤って削除しなくなった
  - [x] 新規 `cleanup()` メソッド、callback server リソースのみをクリーンアップ

> **マイルストーン：v0.20.1-beta**
> *   ✅ 全プラットフォーム（Windows/Linux）での安定動作
> *   ✅ Attach で provider ステータスを自動表示
> *   ✅ OAuth token のリスタート間永続化
> *   ✅ 1339 tests, 117 test files, snapshot saved

---

### Plan09: Interactive Designer (未実装) ⬜
- [ ] **8.2 インタラクティブデザイナー (Interactive Designer)**
    - [ ] `openstarry design` のコンテキスト対応メニューの実装。
    - [ ] 「五蘊設定」のガイド付き Wizard の構築 (Inquirer.js)。

---

## 計画依存関係図

```
                    ┌──────────────────────────────────────────────────┐
                    │           完了済み (v0.20.1-beta)                  │
                    │  Plan01 → Plan02 → Plan03 → Plan04 → Plan05    │
                    │  → Plan06-P1/P2/P3/P4 (MCP Tools/Prompts/Res)  │
                    │  → Plan07-07.3 (Sandbox Hardening)             │
                    │  → Cycle1-8: Sessions, MCP, Sandbox Hardening  │
                    │  → Cycle9: Plan08 TUI Dashboard                │
                    │  → Cycle10-11: Plan09-10 Interactive TUI + CLI │
                    │  → Cycle12: Plan11 DevTools + E2E Framework    │
                    │  → Cycle13: Plan12 Daemon Mode MVP             │
                    │  → Cycle14: Plan13 Seamless Attach             │
                    │  → Cycle15: Plan06-P3 Resources + OAuth 2.1    │
                    │  → Cycle16: Plan14 Multi-client Attach & Mgmt  │
                    │  → Cycle17: Plan06-P4 Sampling & Extensions    │
                    │  → Cycle18: Plan15 SDK Context Extensions      │
                    │  → Cycle19: Plan16 Security Hardening          │
                    │  → Cycle20: Plan17 Plugin Developer Experience │
                    │  → Cycle21: Plan18 Plugin Sync                 │
                    │  → Cycle22: Plan19 Dependency Wiring           │
                    │  → Cycle23: Plan20 Workflow Engine MVP          │
                    │  → Cycle24: Plan21 Web Remote Attach           │
                    │  → Cycle25: Plan22 Plugin Marketplace          │
                    │  → Cycle26: Hotfix (Windows + Attach UX)       │
                    └─────────────────────┬───────────────────────────┘
                                          │
                                          ▼
                                    ┌──────────┐
                                    │ Plan09   │
                                    │ Designer │
                                    │ (未スケジュール) │
                                    └──────────┘
```

---

## 未決議題 (Issues to Discuss)

以下の議題はまだ明確な決議がなく、開発過程で議論が必要：

| 議題 | ステータス | 関連計画 |
|------|----------|---------|
| WebSocket 認証メカニズム | 要議論 | Plan05.1 |
| マルチインスタンス負荷分散 | 要議論 | Plan07 |
| Session 有効期限ポリシー | 要議論 | Plan05.1 |
| SSE 切断再接続 | 要議論 | Plan05.2 |

### 詳細説明

1. **WebSocket 認証メカニズム**
   - Token 認証は必要か？
   - 接続時検証 vs メッセージごとの検証？
   - Session 隔離との関係は？

2. **マルチインスタンス負荷分散**
   - 複数の Agent インスタンスが WebSocket 接続をどのように共有するか？
   - Redis PubSub は必要か？
   - Sticky Session 戦略？

3. **Session 有効期限ポリシー**
   - どの程度の非アクティブ後に自動クリーンアップするか？
   - ハートビートメカニズムは必要か？
   - 有効期限時にクライアントへどのように通知するか？

4. **SSE 切断再接続**
   - クライアント切断後にイベントストリームをどのように復旧するか？
   - Last-Event-ID は必要か？
   - 再接続時に欠落イベントをどのように補送するか？
