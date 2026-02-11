# OpenStarry Agent Corps — 反復ログ (Iteration Log)

このファイルは、各反復サイクルの意思決定と結果を追跡します。

---

## 2026-02-09: Agent Corps 設立

- **アクション**: 6つのエージェントによる開発軍団を設立
- **エージェント**: architect, dev-core, dev-plugin, qa, doc-keeper, researcher
- **インフラ**: ディレクトリ構造、エージェント定義、プロジェクト CLAUDE.md、設定の作成
- **ステータス**: セットアップ完了。最初の反復サイクルへの準備が整う
- **次回**: 実装サイクル 1 (Plan05.1 + 05.2 + 05.5 → v0.2.1-beta) を開始

---

## 20260210_cycle1: 実装サイクル 1

- **日付**: 2026-02-10
- **ターゲットプラン**: Plan05.1 (Session Isolation) + Plan05.2 (HTTP SSE) + Plan05.5-① (Health Check)
- **ターゲットバージョン**: v0.2.1-beta
- **スコープ**:
  - Plan05.1: WebSocket マルチクライアントセッション隔離（セッショントークン、ライフサイクル、データ隔離）
  - Plan05.2: HTTP Server-Sent Events トランスポート（ポーリングに代わるリアルタイムストリーミング）
  - Plan05.5-①: 接続ヘルスチェック（ハートビート、再接続、タイムアウト処理）
- **戦略**: 2サイクルアプローチ
  - サイクル 1 (今回): 05.1 + 05.2 + 05.5-① — SOP ワークフローの確立、クリティカルパス項目の提供
  - サイクル 2 (次回): 05.5-② (指標/ログ) + 05.5-③ (エラー処理) — Plan06 の事前調査と並行可能
- **前提条件**: Plan05 ✅ (v0.2-beta, 2026-02-07)

### Phase 1 — Design

- **アーキテクチャ仕様書 (Architecture Spec)**: `share/test/reports/arch_reviews/20260210_cycle1/Architecture_Spec_Cycle1.md`
- **インターフェース凍結**: はい — セクション 2 のすべてのインターフェースを発行時に凍結
- **主要な凍結インターフェース**:
  - `ISession` (新規) — id, createdAt, updatedAt, metadata を持つセッション実体
  - `ISessionManager` (新規) — セッションライフサイクル：create/get/list/destroy/getStateManager/getDefaultSession
  - `InputEvent.sessionId` (追加) — オプションフィールド。省略時はデフォルトセッションへフォールバック
  - `IPluginContext.sessions` (追加) — プラグインへ ISessionManager を公開
  - `AgentEventType.SESSION_CREATED / SESSION_DESTROYED` (追加) — ライフサイクルイベント
  - `SSEConnection` (新規、内部) — HTTP トランスポート SSE 接続追跡
  - `HealthCheckConfig` (新規、プラグインごと) — ハートビート/有効期限閾値の共有設定構造
  - `ClientConnection.alive / sessionId` (追加) — WebSocket ヘルスとセッションバインド
- **主要な設計決定**:
  1. SessionManager は **コアインフラ** （ StateManager と同様）であり、プラグインではない — マイクロカーネルの純度を維持。セッション隔離はフレームワークの関心事であり、機能ではない
  2. **デフォルトセッション (`__default__`)** を構築時に作成 — 後方互換性の維持。 sessionId なしの入力は以前と同様に共有ステートを使用
  3. `getStateManager(sessionId?)` はセッションと ExecutionLoop の架け橋 — ループは processEvent() の開始時にセッションごとのステートを解決。再構築は不要
  4. TransportBridge は **セッションを意識しない** 状態を維持 — インテリジェンスはプラグインの UI 実装に存在し、 Bridge はダム（無知な）インフラである（マイクロカーネルの原則）
  5. SSE の配信は、 HTTP UI を介さず、リスナー内の **接続ごとのバスサブスクリプション** で処理 — ポーリングとストリーミングの関心事を分離
  6. WebSocket プロトコルの ping/pong は既存のアプリレベル ping/pong と **共存** する — 目的が異なる（サーバー主導のヘルスチェック vs クライアント主導の生存確認）
  7. `IPluginContext.sessions` は個別のメソッドではなく完全な `ISessionManager` を公開する — トランスポートプラグインのための単一で型定義されたインターフェース
  8. デフォルトセッションは **破棄できない** — 後方互換性を保護
  9. イベントのペイロードに `sessionId` + `replyTo` を追加 — イベントインターフェースを変更せずにセッションを認識したルーティングを可能にする
- **実装順序**:
  - **Phase 2a** (dev-core のみ): SDK インターフェース → Core SessionManager → ExecutionLoop 変更 → AgentCore 変更 → テスト更新 (~14の新規テスト)
  - **Phase 2b** (dev-core + dev-plugin 並行): transport-websocket セッション/ping/ルーティング + transport-http SSE/ハートビート/セッション (~12の新規テスト)
  - Phase 2a は Phase 2b の前に完了する必要がある（プラグインが SDK + Core に依存するため）
- **期待されるテスト数**: 既存 82 + 新規 ~26 = 合計 ~108
- **リスクアセスメント**: 8つのリスクを特定（アーキテクチャ仕様書セクション 9 参照）。最高リスク： AgentCore.stateManager の削除（中、機械的修正）および無制限のセッションによるメモリ増大（中、将来のサイクルへ延期）

- **ステータス**: Phase 1 — 設計完了 → Phase 1.5 Baseline

### Phase 2 — Implementation (2026-02-10)

- Phase 2a 完了: SDK インターフェース + Core SessionManager (dev-core)
  - 4つの新しい SDK ファイル/変更、4つの新しい Core ファイル/変更
  - 18の新規テスト、合計 100 → 検証済み
- Phase 2b 完了: トランスポートプラグイン (dev-plugin)
  - transport-websocket: セッションハンドシェイク、プロトコル ping/pong、セッション認識ルーティング (6つの新規テスト)
  - transport-http: SSE エンドポイント、ハートビート、セッション入力 (11の新規テスト)
  - Core tsconfig 修正: ビルドからテストファイルを除外
  - 合計: 118のテストすべて PASS
- ビルド: ✅ pnpm build (11 パッケージ)
- テスト: ✅ 118 PASS
- 純度: ✅ PASS

### Phase 2.5 — Sync 待機中

- Windows のシンボリックリンクの問題 (node_modules symlinks) により sync-to-test.sh が失敗
- 再試行前にスクリプトの修正が必要

- **ステータス**: Phase 2 完了、 Phase 2.5 待機中（同期スクリプトの Windows 向け修正が必要）
- **再開ポイント**: 同期スクリプトの修正 → Phase 2.5 → Phase 3 (QA + architect レビュー) → Phase 4 (収束)
