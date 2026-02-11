# 00. プロジェクト路線図と里程標 (Project Roadmap and Milestones)

このドキュメントでは、 OpenStarry のコアプロトタイプから完全なエコシステムに至るまでの実装パスを定義します。この計画は、プロジェクトの進化を秩序立てて進め、オープンソース貢献者に明確な指針を提供することを目的としています。

---

## 🏷️ バージョン里程標の概要 (Version Milestones)

| バージョン | 内容 | ステータス |
|------|------|------|
| v0.1.0-alpha | MVP 基礎アーキテクチャ (Plan01-04) | ✅ 完了 |
| v0.2.0-beta | マルチチャネル伝送 — WebSocket + HTTP (Plan05) | ✅ 完了 |
| v0.2.1-beta | セッション隔離 + SSE + TraceId (Plan05.1, 05.2, 05.5) | 📋 計画中 |
| v0.2.2-beta | E2E テストスキャフォールディング (Plan05.4) | 📋 計画中 |
| v0.3.0-beta | MCP プロトコル統合 (Plan06 + DevTools) | 📋 計画中 |
| v0.3.5-beta | 管理レイヤーインフラ (Management Zone Infra) | 📋 計画中 |
| v0.4.0-beta | 永続化と状態復元 (Plan07) | 📋 計画中 |
| v0.5.0-beta | TUI ダッシュボード + プラグイン調整レイヤー (Plan08-09) | 📋 計画中 |

---

## 📅 第1段階：創世記 (Phase 1: Genesis)
**目標：** Monorepo の物理的基盤と共通仕様の確立。
**ステータス：** 🟡 進行中 (In Progress) — 1.3 CI/CD は未構築

- [x] **1.1 Monorepo 初期化**
    - [x] `apps/` および `packages/` のディレクトリ構造を作成。
    - [x] `pnpm-workspace.yaml` とルートディレクトリの `package.json` を設定。
    - [x] TypeScript, ESLint, Prettier の仕様を統一。
- [x] **1.2 コア SDK と型定義**
    - [x] `packages/sdk` を作成し、完全な **五蘊（ごうん）プラグインインターフェース** を定義。
        *   **色 (IUI)**: インターフェースと現れ (Form/UI)。
        *   **受 (IListener)**: 感覚の監視 (Perception/Input)。
        *   **想 (IProvider)**: 思考の生成 (Cognition/LLM)。
        *   **行 (ITool)**: ツールの実行 (Action/Will)。
        *   **識 (IGuide)**: 魂と人格設定 (Consciousness/Persona)。
    - [x] `packages/shared` を作成し、グローバルなエラー処理とログモジュールを提供。
- [ ] **1.3 エンジニアリングとテスト基盤 (Engineering & Testing Infrastructure)**
    - [x] ユニットテストフレームワーク (Vitest) の設定。 *(Plan02 Phase C — 56のテストを通過)*
    - [x] 純度チェックスクリプト ( `pnpm test:purity` )。 *(Plan03 Phase A4)*
    - [ ] CI/CD フロー (GitHub Actions) の構築。

## 🧠 第2段階：意識の核心 (Phase 2: The Conscious Kernel)
**目標：** 「ヘッドレス (Headless)」、イベント駆動型の `Agent Core` の実現。
**ステータス：** 🟢 完了 (Done)

- [x] **2.1 実行ループ (The Loop)**
    - [x] `packages/core/execution` の実装（イベントキューに基づく tick メカニズム）。
    - [x] ExecutionLoop のイベント駆動リファクタリング： EventQueue からイベントを pull し、ステートマシンに WAITING_FOR_EVENT / SAFETY_LOCKOUT を含める。 *(Plan02 Phase A)*
    - [x] InputEvent ペイロードの標準化（ source, inputType, data, replyTo ）。 *(Plan02 Phase A4)*
- [x] **2.2 ステートと記憶の管理**
    - [x] `packages/core/state` の実装（スナップショットと永続化インターフェースのサポート）。
    - [x] `packages/core/memory` の実装（スライディングウィンドウなどのプラグイン可能なコンテキスト戦略）。
- [x] **2.3 安全遮断器とエラーフィードバック (Safety & Correction)**
    - [x] Token 消費上限と無限ループ検知 (Circuit Breakers) の実装。 *(Plan02 Phase B — SafetyMonitor)*
    - [x] 「エラーは痛覚」メカニズムの実装。実行時のエラーをコンテキストへのフィードバックに変換し、エージェントの自己修復を促す。 *(Plan02 Phase B — 挫折カウンター + 重複呼び出し検知 + エラー連鎖遮断)*

## 🦾 第3段階：手足と感覚 (Phase 3: Body and Senses)
**目標：** 完全な五蘊プラグインインフラと標準ライブラリの構築、セキュリティ境界の確立。
**ステータス：** 🟡 進行中 (In Progress) — コアプラグインは完了、高度な機能は未実装

- [ ] **3.1 コア・ロードプロトコル (Loading Protocol)**
    - [x] **ファクトリパターン (Factory Pattern)** による初期化をサポートする `PluginLoader` の実装。 *(Plan01)*
    - [ ] `IPluginContext` の実装：ロガー、設定、および **プラグイン間サービス注入 (dependencies フィールド)** を含む。
    - [ ] **依存関係の編み込みロジック (Dependency Wiring) の実装：** ドキュメント `20` に基づき、ロード時に OODA ループを自動接続する。
    - [x] **コマンドレジストリ (CommandRegistry) の実装：** CLI の `run-tool` とチャットの `/slash` コマンドのマッピングと実行ロジックをサポート。 *(Plan01)*
    - [x] **動的プラグインロード (Dynamic Loading) の実装：** CLI で `agent.json` の `plugins[].path` フィールドをサポートし、 `import()` を介してサードパーティプラグインを動的にロードする。 *(Plan02 Phase A3)*
    - [x] `agent.json` の実行時検証器（ Zod スキーマ検証）の実装。 *(Plan03 で補完完了)*
- [ ] **3.2 調整レイヤーと登録メカニズム (Coordination Layer)**
    - [x] **二重経路スキャンメカニズム (Dual-Path Scanning)：** システムとプロジェクト固有のプラグインディレクトリを同時にスキャン。
    - [ ] **ワンクリック同期 (Sync)：** 公式の **`openstarry_plugin`** リポジトリの内容をシステムディレクトリに同期する `openstarry plugin sync` コマンドの実装。
    - [x] デーモン内での **Plugin Registry Service** の実装、メモリ内インデックスの構築。
- [ ] **3.3 標準機能集約プラグインライブラリ (Standard Aggregate Plugins)**
    > **開発場所：** `openstarry_plugin` リポジトリ (Ecosystem Repo)
    - [x] `@openstarry-plugin/standard-function-stdio`：受（ Stdio リスナー）+ 色（ CLI 身体）+ 識（デフォルトガイド）を集約。 *(Plan01)*
    - [x] `@openstarry-plugin/provider-gemini-oauth`：想（ Gemini プロバイダー）を集約。 PKCE + OAuth 2.0 認証。 *(Plan01)*
    - [x] `@openstarry-plugin/standard-function-fs`：行（ FS ツール）+ 識（パススコープポリシー）を集約。 *(Plan01)*
    - [ ] `@openstarry-plugin/guide-mcp`：識（ MCP ガイド）を集約。標準化された通信能力を付与。
    - [ ] `@openstarry-plugin/guide-pain-mechanism`：擬人化された痛覚解釈ロジックの実装。
    - [x] `@openstarry-plugin/standard-function-skill`：識（スキルガイド）を集約。 **コア依存関係**： Markdown スキルファイルの解析を担当。ワークフローと複雑なエージェントの基礎。 *(Plan03 Phase B1)*
- [ ] **3.4 プラグイン開発体験 (DX)**
    - [ ] `openstarry create-plugin` スキャフォールディングの実装。
    - [ ] プラグインの独立した開発と検証をサポートする `MockHost` テスト環境の提供。

> **🔒 セキュリティに関する注意：**
> 第3段階の `fs` ツールは Docker 内で動作しませんが、 `packages/shared` のパス正規化コンポーネントを通過させ、操作範囲を厳格に制限しなければなりません。これにより、エージェントが必要な操作を実行できるようにしつつ、システムの重要なディレクトリを保護します。

## 👶 第4段階：降生 (Phase 4: The First Breath)
**目標：** 初の単体 CLI エージェントを実現し、エンドツーエンドのフローを検証する。
**ステータス：** 🟡 進行中 (In Progress) — Runner は動作可能、エンドツーエンドの LLM 検証は OAuth ログイン後のテスト待ち

- [x] **4.1 ローカル実行器 (Local Runner)**
    - [x] `agent.json` による起動をサポートする `apps/runner` 引導プログラム（純粋なブートストラップ・ランナー）の実装。 *(Plan01)*
    - [x] 動的なプラグイン解決（ path / node_modules の2段階戦略。 BUILTIN_FACTORIES は削除済み。すべてのプラグインは動的にロード）のサポート。 *(Plan02 Phase A3)*
- [ ] **4.2 統合検証**
    - [ ] 「感知 -> 思考 -> ツール呼び出し -> 修正 -> 応答」の完全な閉ループを達成。 *( OAuth ログイン後の手動テストが必要 )*

> **🏆 里程標： v0.1 Alpha (MVP)**
> *   ✅ CLI で単一のエージェントを実行可能。
> *   ✅ プロバイダーとして Gemini を使用（ PKCE + OAuth 認証）。
> *   ✅ ローカルファイルシステムの操作が可能 ( `fs` tool )。
> *   ✅ 基礎的な記憶能力 (5ターンの対話)。
> *   ✅ 安全遮断メカニズム（ Token 予算、ループ上限、エラー連鎖、挫折カウンター）。
> *   ✅ 構造化 JSON ログ ( LOG_FORMAT=json, LOG_LEVEL フィルタリング)。
> *   ✅ agent.json の Zod 実行時検証（設定ミスを即座に通知）。
> *   ✅ ツール呼び出しのタイムアウト ( Promise.race )。
> *   ✅ TraceID メカニズム（一連の処理サイクルをログで追跡）。
> *   ✅ Markdown スキルのロード ( standard-function-skill プラグイン )。
> *   ✅ guideFile 外部ファイル参照 ( system_prompt を .md からロード )。
> *   ✅ 純度チェックスクリプト ( pnpm test:purity )。
> *   ✅ 56のユニットテスト通過 ( Vitest )。
> *   ⬜ エンドツーエンドの LLM 通話検証（手動テスト待ち）。

---

> **🚀 以上でエージェントコアが完了。以下、エージェント用 OS とエコシステムの段階に入ります。**

---

## 📡 第 4.5 段階：マルチチャネル伝送 (Phase 4.5: Multi-Channel Transport)
**目標：** IUI/IListener 分離アーキテクチャの拡張性を検証し、 stdio 以外のチャネルを実装する。
**ステータス：** 🟢 完了 (v0.2.0-beta)

- [x] **4.5.1 WebSocket トランスポートプラグイン**
    - [x] `@openstarry-plugin/transport-websocket`： WebSocket リスナー + UI 。
    - [x] 複数クライアント接続、ターゲット指定返信 (replyTo) をサポート。
- [x] **4.5.2 HTTP Webhook トランスポートプラグイン**
    - [x] `@openstarry-plugin/transport-http`： HTTP リスナー + UI 。
    - [x] POST /api/input, GET /api/status, GET /api/response 。
- [x] **4.5.3 複数 UI への同時出力**
    - [x] TransportBridge ブロードキャストメカニズムの検証通過。
    - [x] stdio + WebSocket で同時にイベントを受信。

> **🏆 里程標： v0.2.0 Beta (Multi-Channel)**
> *   ✅ WebSocket トランスポートプラグインの実装完了。
> *   ✅ HTTP Webhook トランスポートプラグインの実装完了。
> *   ✅ 複数 UI への同時出力の検証通過。

---

## 🔐 第 4.6 段階：実装サイクル 1 (v0.2.1-beta)
**目標：** マルチユーザーのプライバシー問題の解決、伝送効率の最適化、全リンク追跡の確立。
**ステータス：** 📋 計画中

### Plan05.1: セッション隔離とメッセージルーティング 🔴 最高優先度
- [ ] リスナーによる `sessionId` の付与。
- [ ] コアによる `sessionId` の出力イベントへの透過。
- [ ] UI による `sessionId` に基づくプッシュ配信のフィルタリング。
- [ ] 受入基準： WebSocket ユーザー A がユーザー B の対話を見ることができないこと。

### Plan05.2: HTTP SSE サポート 🟢 高優先度
- [ ] `GET /api/stream` エンドポイントの新設。
- [ ] ポーリングに代わる Server-Sent Events の使用。
- [ ] EventSource クライアントのサポート。

### Plan05.5: 統一 TraceId 🔵 低優先度 (前倒しで実装)
- [ ] コアによる統一 `TraceId` 生成器の提供。
- [ ] すべてのイベントとログへの自動付加。

## 🧪 第 4.7 段階：実装サイクル 2 (v0.2.2-beta)
**目標：** 自動検証能力とテストスキャフォールディングの構築。
**ステータス：** 📋 計画中

### Plan05.4: E2E テストスキャフォールディング 🟡 中優先度
- [ ] `createTestAgent()` ユーティリティ関数。
- [ ] MockLLM プラグインのサポート。
- [ ] コミュニティ貢献のハードルを下げる。

> **⚠️ 注意： Plan05.3 DevTools は Plan06 (MCP) の後に実装を延期。**

---

## 🏰 第5段階：守護プロセス (Phase 5: The Orchestrator Daemon) — Plan07
**目標：** プロセスレベルの管理と永続化の実現。
**ステータス：** 📋 計画中 (v0.4.0-beta)

- [ ] **5.1 デーモンコア (Daemon)**
    - [ ] 複数のエージェントインスタンスのライフサイクルを管理する `apps/daemon` を構築。
    - [ ] **エージェント調整管理レイヤー (Management Zone)** におけるスケジューリングロジックの実装。
- [ ] **5.2 API ゲートウェイと永続化**
    - [ ] リモート管理のための HTTP/gRPC API を提供。
    - [ ] エージェントの長期状態を保存するためのデータベース（ SQLite/LevelDB ）の統合。
- [ ] **5.3 セキュリティとガバナンス (Security & Governance)**
    - [ ] プラグイン署名と検証メカニズムの実装。
    - [ ] 実行時サンドボックス (Sandbox) 戦略（ Node.js vm または WASM コンテナ化）の実装。
    - [ ] **環境隔離とリソース配分管理 (Quota Management) の実装**。
- [ ] **5.4 システムの可観測性とハードウェア抽象化 (Observability & HAL)**
    - [x] 構造化 JSON ログの実装。 *(Plan02)*
    - [ ] OpenTelemetry または Event Tracing を統合し、エージェントをまたぐタスクの流れを追跡。
    - [ ] ログ収集の実装、ダッシュボードでの視覚化をサポート。
    - [ ] **ハードウェア抽象化レイヤー (HAL) 標準感覚データストリームの実装**。

## 🕸️ 第6段階：フラクタル社会 (Phase 6: Fractal Society) — Plan06
**目標：** エージェント間の協調と MCP プロトコルの実現。
**ステータス：** 📋 計画中 (v0.3.0-beta)
**前提条件：** Plan05.1 セッション隔離の完了 ✓

- [ ] **6.1 MCP の深い統合 (Plan06)**
    - [ ] エージェントに標準化された相互接続能力を与える `packages/mcp-protocol` の実装。
    - [ ] 複数の外部クライアントの同時接続をサポート。
- [ ] **6.2 DevTools デバッグインターフェース (Plan05.3)**
    - [ ] 静的 HTML デバッグインターフェース `openstarry-devtools` 。
    - [ ] 左側：対話バブルビュー、右側： State/Context 検査、下部：イベントフィルター。
- [ ] **6.3 ワークフローエンジン (Workflow Engine)**
    - [ ] YAML で定義されたタスク編成をサポートする `WorkflowEngineTool` の実装。
    - [ ] エージェントの動的な生成と破棄メカニズム (Ephemeral Agents) の実装。
- [ ] **6.4 エージェント間の協調と因果スケジューリング (Orchestration)**
    - [ ] タスクの分解と異なるサブエージェントへの分配メカニズムの実現。
    - [ ] **因果連鎖に基づくイベントスケジューリング (Causality Chain) の実装**。

## 📦 第7段階：視覚化とエコシステム (Phase 7: Ecosystem and UI)
**目標：** Web グラフィカル管理およびデプロイツールの提供。
**ステータス：** 🔴 待機中

- [ ] **7.1 ダッシュボードと多元対話レイヤー (Interface)**
    - [ ] すべてのエージェントを可視化して監視する `apps/dashboard` (React/Next.js) を構築。
    - [ ] **状態投影 (State Projection) ダッシュボードの実装**。
- [ ] **7.2 テンプレートサービス (Agent Design Service)**
    - [ ] エージェントストア/テンプレートダウンロード機能を提供し、「ディレクトリ即プロトコル」の極簡デプロイを実現。

## 🖥️ 第8段階：ターミナルの進化 (Phase 8: CLI & TUI Evolution) — Plan08 / Plan09
**目標：** 「 OS レベル」のターミナル体験を実現し、 User Scenario Guide のビジョンを達成。
**ステータス：** 📋 計画中 (v0.5.0-beta)

- [ ] **8.1 実行時ダッシュボード (Runtime Dashboard)**
    - [ ] `openstarry` の TUI インターフェース（ Ink または Blessed を使用）の実装。
    - [ ] デーモン状態監視、リソースグラフ、リアルタイムログストリーミングの統合。
- [ ] **8.2 対話型デザイナー (Interactive Designer)**
    - [ ] `openstarry design` のコンテキスト認識メニューの実装。
    - [ ] 「五蘊設定」のガイド付きウィザード (Inquirer.js) の構築。
- [ ] **8.3 シームレス接続メカニズム (Seamless Attach)**
    - [ ] `openstarry attach` のソケット接続ロジックの実装。
    - [ ] TUI ダッシュボードからエージェントの対話ウィンドウへ直接ジャンプ（ 'a' キー）をサポート。

---

## 📊 計画の依存関係図

（略。 CN/EN 版と同様の構造）

---

## 🟡 検討事項 (Issues to Discuss)

以下の議題についてはまだ明確な決議がなく、開発プロセスの中で議論する必要があります：

| 議題 | ステータス | 関連計画 |
|------|------|----------|
| WebSocket 認証メカニズム | 🟡 検討中 | Plan05.1 |
| マルチインスタンス負荷分散 | 🟡 検討中 | Plan07 |
| セッション有効期限ポリシー | 🟡 検討中 | Plan05.1 |
| SSE 切断後の再接続 | 🟡 検討中 | Plan05.2 |
