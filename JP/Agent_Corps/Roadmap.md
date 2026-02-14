# OpenStarry — ロードマップ & バージョン管理

設立日：2026-02-12

---

## バージョン履歴 & 現在のステータス

| バージョン | Cycle | Plan | 日付 | ステータス |
|---------|-------|------|------|--------|
| v0.1-alpha | Cycle 1-2 | Plan01-02 | 2026-02-04 to 2026-02-05 | ✅ COMPLETE |
| v0.2-alpha | Cycle 3-4 | Plan03-04 | 2026-02-05 to 2026-02-06 | ✅ COMPLETE |
| v0.2-beta | Cycle 5+ | Plan05 | 2026-02-07+ | ✅ COMPLETE |
| v0.2.1-beta | Cycle 1 (Rework) | Plan05.1, Plan05.2, Plan05.5-① | 2026-02-10 | ✅ COMPLETE |
| v0.2.2-beta | Cycle 2 | Plan05.5-②, Plan05.5-③ | 2026-02-11 | ✅ COMPLETE |
| v0.3.0-beta | Cycle 3 | Plan06-P1 (MCP Client) | 2026-02-11 | ✅ COMPLETE |
| v0.3.1-beta | Cycle 4 | Plan06-P2 (MCP Server) | 2026-02-11 | ✅ COMPLETE |
| v0.4 | Cycle 5 | Plan07 (Sandbox MVP) | 2026-02-11 | ✅ COMPLETE |
| v0.4.1-beta | Cycle 6 | Plan07.1 (Sandbox Hardening) | 2026-02-11 | ✅ COMPLETE |
| v0.4.2-beta | Cycle 7 | Plan07.2 (Sandbox Advanced) | 2026-02-11 | ✅ COMPLETE (1 rework) |
| v0.4.3-beta | Cycle 8 | Plan07.3 (Custom require + audit) | 2026-02-11 | ✅ COMPLETE |
| v0.5.0-beta | Cycle 9 | Plan08 (TUI Dashboard MVP) | 2026-02-11 | ✅ COMPLETE |
| v0.5.1-beta | Cycle 10 | Plan09 (Interactive TUI) | 2026-02-12 | ✅ COMPLETE |
| v0.6.0-beta | Cycle 11 | Plan10 (CLI Foundation & Runner) | 2026-02-12 | ✅ COMPLETE (1 rework) |
| v0.7.0-beta | Cycle 12 | Plan11 (DevTools & E2E Testing) | 2026-02-12 | ✅ COMPLETE (1 rework) |
| v0.8.0-beta | Cycle 13 | Plan12 (Daemon Mode MVP) | 2026-02-12 | ✅ COMPLETE (1 rework) |
| v0.9.0-beta | Cycle 14 | Plan13 (Seamless Attach) | 2026-02-12 | ✅ COMPLETE (1 rework) |
| v0.10.0-beta | Cycle 15 | Plan06-P3 (MCP Resources + OAuth) | 2026-02-12 | ✅ COMPLETE |
| v0.11.0-beta | Cycle 16 | Plan14 (Multi-client Attach & Session) | 2026-02-12 | ✅ COMPLETE |
| v0.12.0-beta | Cycle 17 | Plan06-P4 (MCP Sampling & Extensions) | 2026-02-12 | ✅ COMPLETE |
| v0.13.0-beta | Cycle 18 | Plan15 (SDK Context Extensions) | 2026-02-12 | ✅ COMPLETE |
| v0.14.0-beta | Cycle 19 | Plan16 (Security Hardening) | 2026-02-12 | ✅ COMPLETE (1 rework) |
| v0.15.0-beta | Cycle 20 | Plan17 (Plugin Developer Experience) | 2026-02-12 | ✅ COMPLETE |
| v0.16.0-beta | Cycle 21 | Plan18 (Plugin Sync & System Directory) | 2026-02-12 | ✅ COMPLETE |
| v0.17.0-beta | Cycle 22 | Plan19 (Plugin Dependency Wiring) | 2026-02-12 | ✅ COMPLETE (1 rework) |
| **v0.18.0-beta** | **Cycle 23** | **Plan20 (Workflow Engine MVP)** | **2026-02-12** | **✅ COMPLETE (spec rewrite)** |
| **v0.19.0-beta** | **Cycle 24** | **Plan21 (Web-based Remote Attach)** | **2026-02-12** | **✅ COMPLETE** |
| **v0.20.0-beta** | **Cycle 25** | **Plan22 (Plugin Marketplace MVP)** | **2026-02-13** | **✅ COMPLETE** |
| **v0.20.1-beta** | **Cycle 25 hotfix** | **Windows クロスプラットフォーム修正 (43→0 failures)** | **2026-02-13** | **✅ COMPLETE** |
| v0.21.0-beta | Cycle 26 | Plan23 (TBD — 研究チームの提案確認待ち) | TBD | 📋 PLANNED |
| v0.22.0+ | Future | バックログ (State Machines, Multi-agent, Advanced) | TBD | ⬜ BACKLOG |

---

## ロードマップフェーズ

### Phase 1: コア基盤 & イベント (COMPLETE ✅)
- Plan01-05: MVP Alpha、Event-Driven、UI/Listener/Guide
- v0.1-0.2-beta: 基盤構築
- ステータス: Cycles 1-5 で提供

### Phase 2: MCP 統合 & サンドボックス (COMPLETE ✅)
- Plan06 (全フェーズ): MCP Client → Server → Resources → Sampling
- Plan07 (全フェーズ): セキュリティ堅牢化を備えた Runtime Sandbox
- v0.3-0.4.3-beta: MCP プロトコル完了、サンドボックス成熟
- ステータス: Cycles 3-8 で提供

### Phase 3: ユーザーインターフェース & 開発者ツール (COMPLETE ✅)
  - **Phase 3a**: TUI & CLI (Cycles 9-11)
    - Plan08: TUI Dashboard MVP → v0.5.0-beta
    - Plan09: Interactive TUI → v0.5.1-beta
    - Plan10: CLI & Runner → v0.6.0-beta

  - **Phase 3b**: 開発者体験 (Cycles 12-14)
    - Plan11: DevTools & E2E Testing → v0.7.0-beta
    - Plan12: Daemon Mode MVP → v0.8.0-beta
    - Plan13: Seamless Attach → v0.9.0-beta

  - **Phase 3c**: セッション & マルチクライアント (Cycles 15-16)
    - Plan14: Multi-client Attach → v0.11.0-beta
    - Plan06-P3: MCP Resources → v0.10.0-beta

  - **Phase 3d**: SDK 堅牢化 & 品質 (Cycles 17-21)
    - Plan06-P4: MCP Sampling → v0.12.0-beta
    - Plan15: SDK Context Extensions → v0.13.0-beta
    - Plan16: Security Hardening → v0.14.0-beta
    - Plan17: Plugin DX (MockHost, scaffolding) → v0.15.0-beta
    - Plan18: Plugin Sync → v0.16.0-beta

  - **Phase 3e**: プラグインサービス (Cycle 22)
    - Plan19: Dependency Wiring & Service Registry → **v0.17.0-beta** ✅ COMPLETE

### Phase 4: 高度な機能 (COMPLETE ✅)
  - **Phase 4a**: ワークフロー & オーケストレーション (Cycle 23)
    - Plan20: Workflow Engine MVP → **v0.18.0-beta** ✅ COMPLETE

  - **Phase 4b**: Web & リモートアクセス (Cycle 24)
    - Plan21: Web-based Remote Attach → **v0.19.0-beta** ✅ COMPLETE

  - **Phase 4c**: プラグインマーケットプレイス (Cycle 25)
    - Plan22: Plugin Marketplace MVP → **v0.20.0-beta** ✅ COMPLETE

### Phase 5: 次世代 (PLANNED 📋)
  - **Plan23**: TBD — 研究チームの提案確認後に割り当て

  - **バックログ**（未整序、優先度評価待ち）:
    - State Machines & Event Routing
    - Multi-agent Orchestration
    - Service Mesh Patterns
    - Plugin Signature Verification Integration
    - Network Fetch Sandbox

---

## ロードマップマイルストーン

### 完了済みマイルストーン

| マイルストーン | Cycles | テスト数 | ステータス | 日付 |
|-----------|--------|-----------|--------|------|
| MVP 基盤 | 1-5 | 407 tests | ✅ | 2026-02-11 |
| MCP プロトコル | 3-8 | 442 tests | ✅ | 2026-02-11 |
| ユーザーインターフェース | 9-11 | 632 tests | ✅ | 2026-02-12 |
| DevTools & Daemon | 12-14 | 747 tests | ✅ | 2026-02-12 |
| マルチクライアントセッション | 15-16 | 807 tests | ✅ | 2026-02-12 |
| SDK 堅牢化 | 17-21 | 1009 tests | ✅ | 2026-02-12 |
| プラグインサービス | 22 | **1067 tests** | ✅ | 2026-02-12 |
| ワークフローエンジン | 23 | **1104 tests** | ✅ | 2026-02-12 |
| Web リモートアタッチ | 24 | **1132 tests** | ✅ | 2026-02-12 |
| プラグインマーケットプレイス | 25 | **1330 tests** | ✅ | 2026-02-13 |
| Windows クロスプラットフォーム | 25 hotfix | **1332 tests** | ✅ | 2026-02-13 |

### 今後のマイルストーン

| マイルストーン | Plan | ターゲットバージョン | ステータス | 予想 Cycle |
|-----------|------|-----------------|--------|-----------------|
| Plan23 (TBD) | Plan23 | v0.21.0-beta | 📋 PLANNED | Cycle 26 |

### バックログ（未整序）

| マイルストーン | ターゲットバージョン | ステータス |
|-----------|---------------|--------|
| State Machines & Event Routing | TBD | ⬜ BACKLOG |
| Multi-agent Orchestration | TBD | ⬜ BACKLOG |
| Service Mesh Patterns | TBD | ⬜ BACKLOG |
| Plugin Signature Verification | TBD | ⬜ BACKLOG |
| Network Fetch Sandbox | TBD | ⬜ BACKLOG |

---

## テスト増加の推移

```
Cycle 1-2:   118 tests    (Foundation)
Cycle 3:     200 tests    (MCP Client MVP)
Cycle 4:     252 tests    (MCP Server MVP)
Cycle 5:     320 tests    (Sandbox MVP)
Cycle 6:     351 tests    (Sandbox Hardening)
Cycle 7:     407 tests    (Advanced Hardening, +1 rework)
Cycle 8:     442 tests    (Custom require)
Cycle 9:     524 tests    (TUI Dashboard)
Cycle 10:    559 tests    (Interactive TUI)
Cycle 11:    632 tests    (CLI Foundation)
Cycle 12:    670 tests    (DevTools & E2E, +1 rework)
Cycle 13:    714 tests    (Daemon Mode, +1 rework)
Cycle 14:    747 tests    (Seamless Attach, +1 rework)
Cycle 15:    792 tests    (MCP Resources)
Cycle 16:    807 tests    (Multi-client Session)
Cycle 17:    894 tests    (MCP Sampling Extensions)
Cycle 18:    915 tests    (SDK Context Extensions)
Cycle 19:    935 tests    (Security Hardening, +1 rework)
Cycle 20:    970 tests    (Plugin DX)
Cycle 21:   1009 tests    (Plugin Sync)
Cycle 22:   1067 tests    (Plugin Services, +1 rework)
Cycle 23:   1104 tests    (Workflow Engine MVP, spec rewrite)
Cycle 24:   1132 tests    (Web Remote Attach)
Cycle 25:   1330 tests    (Plugin Marketplace MVP)
Cycle 25+:  1332 tests    (Windows Cross-Platform Fix)
```

---

## アーキテクチャフェーズ

### Phase A: 五蘊 (COMPLETE ✅)
- IUI (色) — ユーザーインターフェースハンドラー
- IListener (受) — イベントリスナー
- IProvider (想) — サービスプロバイダー
- ITool (行) — 実行可能ツール
- IGuide (識) — システムプロンプト/ガイド
- ステータス: すべてのインターフェースが定義され統合済み (Cycles 1-5)

### Phase B: プラグインシステムの成熟 (COMPLETE ✅)
- ファクトリーパターン & マニフェストバリデーション
- サンドボックス分離 & セキュリティ堅牢化
- プラグインのロード & ライフサイクル管理
- クロスプラグイン通信 (Plans 18-19)
- ステータス: サービスレジストリにより完了 (Cycles 5-22)

### Phase C: 開発者体験 (COMPLETE ✅)
- CLI 基盤 & ランナーインフラ
- TUI ダッシュボード & インタラクティブ入力
- Daemon モード & シームレスアタッチ
- DevTools プラグイン & E2E テストフレームワーク
- MockHost テストユーティリティ & スキャフォールディング CLI
- プラグイン同期 & システムプラグインディレクトリ
- ステータス: 成熟した DX ツール (Cycles 9-21)

### Phase D: 高度なオーケストレーション (COMPLETE ✅)
- ワークフローエンジン（YAML ベース）✅ (Cycle 23)
- Web ベースリモートアタッチ ✅ (Cycle 24)
- Plugin Marketplace MVP ✅ (Cycle 25)
- ステータス: 計画されたすべてのオーケストレーション機能が提供済み (Cycles 23-25)

### Phase E: 次世代 (PLANNED 📋)
- Plan23: TBD — 研究チームの提案確認待ち
- バックログ: State machines、Multi-agent、Service mesh、Plugin signature、Network sandbox

---

## Cycle 別の主要成果物

### Cycle 22 — Plan19: Plugin Dependency Wiring & Cross-Plugin Services

**バージョン**: v0.17.0-beta

**成果物**:
- IPluginService インターフェース (SDK)
- IServiceRegistry インターフェース (SDK) — register/get/has/list
- ServiceRegistry クラス (Core) — インメモリ実装
- IPluginContext.services accessor
- PluginManifest.services/serviceDependencies フィールド
- PluginLoader.loadAll() — Kahn のトポロジカルソート
- 循環依存検出
- 65 新規テスト（サービスレジストリ、依存関係順序、e2e インジェクション）
- **Rework Cycle 22.1**: Code Fix — has() メソッドの欠落とフィールド名の整合を追加

**作成されたファイル**: 5 新規ファイル
- `packages/sdk/src/types/service.ts`
- `packages/core/src/infrastructure/service-registry.ts`
- 3 テストファイル

**変更されたファイル**: 6 ファイル
- SDK 型、エラー、エクスポート
- Core エージェント初期化、プラグインローダー、サンドボックスマネージャー

**品質メトリクス**:
- テスト: 1009 → 1067 (+65, +5.8%)
- テストファイル: 83 → 88 (+5)
- リワークサイクル: 1 (Code Fix)
- ブロッキング問題: 0
- 後方互換性: 完全（すべての変更は非破壊的）

**ステータス**: ✅ COMPLETE (Phase 4 Convergence PASS, 2026-02-12)

---

## 依存関係グラフ

```
Plan01-05 (MVP Foundation)
  ↓
Plan06 (MCP: Client → Server → Resources → Sampling) ⟵ Plan07 (Sandbox)
  ↓
Plan08-09 (TUI: Dashboard → Interactive)
  ↓
Plan10-11 (CLI + DevTools & E2E)
  ↓
Plan12-13 (Daemon + Seamless Attach)
  ↓
Plan14 (Multi-client Session) ⟵ Plan06-P3 (MCP Resources)
  ↓
Plan15 (SDK Context Extensions)
  ↓
Plan16 (Security Hardening)
  ↓
Plan17 (Plugin DX: MockHost + Scaffolding)
  ↓
Plan18 (Plugin Sync & System Directory)
  ↓
Plan19 (Plugin Dependency Wiring & Services) ✅
  ↓
Plan20 (Workflow Engine MVP) ✅
  ↓
Plan21 (Web-based Remote Attach) ✅
  ↓
Plan22 (Plugin Marketplace MVP) ✅
  ↓
Plan23 (TBD — 研究チームの提案確認待ち) 📋 PLANNED
  ↓
バックログ: State Machines, Multi-agent, Service Mesh, Plugin Signature, Network Sandbox
```

---

## 次のステップ

**最新リリース**: v0.20.1-beta — Windows クロスプラットフォーム修正 (Cycle 25 hotfix) ✅

**Cycle 25 hotfix (v0.20.1-beta) 提供内容**:
- Windows クロスプラットフォーム修正：43個の失敗テスト → 0個の失敗
- パス処理：`sep`、`basename()`、`pathToFileURL()` でハードコードされた Unix パスを置換
- Daemon IPC：新規 `platform.ts` — Windows は named pipe、Linux は Unix socket
- プラットフォームガード：SIGHUP、chmod、mkdirSync、unlinkSync を Windows 上でスキップ
- `plugin-installer.ts`：`cp()` dereference フォールバック + `rm()` maxRetries
- テスト数：1330 → 1332（+2、3 skipped）
- スナップショット: `20260213_cycle25_winfix`

**Cycle 25 提供内容**:
- バンドルされたプラグインカタログ (`plugin-catalog.json`) — 全15個の公式プラグイン
- プラグインロックファイル (`~/.openstarry/plugins/lock.json`) — インストール済みプラグインの追跡
- ワークスペース優先解決 + npm フォールバックによるプラグインインストーラー
- 5個の新規 CLI コマンド: `plugin install`, `plugin uninstall`, `plugin list`, `plugin search`, `plugin info`
- `plugin install --all` で全15個の公式プラグインを一括インストール
- 短縮名サポート（例：`standard-function-fs` → `@openstarry-plugin/standard-function-fs`）
- 77個の新規テスト（198 純増：1132 → 1330）
- 基盤修正: `sync-to-test.sh`、`snapshot.sh`、`baseline.sh` がプラグインの `__tests__/`、`vitest.config.ts`、`configs/`、`README.md` を自動コピー

**次の Cycle**: Plan23 — 研究チームの提案確認後に内容を割り当て

**バックログ候補**（未整序）:
- State Machines & Event Routing（探索完了、仕様草案あり）
- Multi-agent Orchestration（2-3 サイクル必要）
- Service Mesh Patterns
- Plugin Signature Verification Integration（plugin-signer は存在するが未有効化）
- Network Fetch Sandbox（allowedDomains は定義済みだが未実装）

**ステータス**: Plan01-22 すべて完了 ✅ | テスト: 1330 | snapshot: 20260213_cycle25
