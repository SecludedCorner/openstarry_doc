# OpenStarry 実装計画 01 — MVP Alpha Foundation

> **ステータス**: ✅ 完了 (2026-02-04)

## 目標 (Target)
フェーズ 1〜4 を実施し、 **MVP v0.1 Alpha** マイルストーンを達成する：
- ローカル CLI エージェントの実行
- LLM プロバイダーとしての Gemini (PKCE + OAuth, 参照: `ref/openoctopus/packages/provider-gemini-oauth`)
- ローカルファイルシステム操作 (fs ツール)
- 基礎的な記憶 (5ターンのスライディングウィンドウ)
- 完全な「感知 → 思考 → ツール呼び出し → 修正 → 応答」ループ

## アーキテクチャ概要 (Architecture Overview)
- **Monorepo**: `apps/` と `packages/` を含む `openstarry/`
- **プラグインエコシステム**: `openstarry_plugin/` (フラット構造、別リポジトリ)
- **言語**: TypeScript (ESM モジュール)
- **パッケージマネージャー**: pnpm workspaces
- **リファレンス**: OpenOctopus パターン（ファクトリ関数、 EventBus、 Zod、 非同期ジェネレーター）

---

## 構造 (Structure)

```
openstarry/
├── package.json
├── pnpm-workspace.yaml
├── tsconfig.base.json
├── tsconfig.json
├── example-agent.json
├── apps/runner/src/
│   ├── bin.ts          # CLI エントリ + ホストブートストラップ
│   └── index.ts        # 再エクスポート
├── packages/sdk/src/
│   ├── types/          # message, tool, provider, plugin, listener, guide, agent, events
│   ├── errors/         # AgentError 階層体系
│   └── interfaces/     # IContextManager, IStateManager
├── packages/shared/src/
│   ├── logger/         # 構造化ログ
│   ├── utils/          # UUID, Zod 検証ヘルパー
│   └── constants/      # イベント定数
└── packages/core/src/
    ├── bus/            # EventBus (パブ/サブ)
    ├── execution/      # EventQueue, ExecutionLoop (ステートマシン)
    ├── state/          # StateManager (メモリ内)
    ├── memory/         # ContextManager (スライディングウィンドウ)
    ├── infrastructure/ # ToolRegistry, ProviderRegistry, ListenerRegistry,
    │                   # GuideRegistry, CommandRegistry, PluginLoader
    ├── security/       # SecurityLayer (パスガードレール)
    ├── agents/         # AgentCore (メインオーケストレーター)
    └── transport/      # TransportBridge (リスナーへのイベントルーティング)

openstarry_plugin/
├── provider-gemini-oauth/src/index.ts   # Gemini PKCE+OAuth プロバイダー
├── standard-function-fs/src/index.ts    # fs.read/write/list/mkdir/delete
└── standard-function-stdio/src/index.ts # CLI I/O リスナー + デフォルトガイド
```

## 実装ステップ (完了済み)

1. **Monorepo Scaffolding** — pnpm workspace, tsconfig, package.json
2. **SDK Types** — Message, Tool, Provider, Plugin, Listener, Guide, Agent, Events, Errors
3. **Shared Utils** — Logger, UUID, Zod ヘルパー, イベント定数
4. **Core: EventBus + EventQueue** — パブ/サブ + 非同期 FIFO キュー
5. **Core: StateManager + ContextManager** — メモリ内履歴 + スライディングウィンドウ (5ターン)
6. **Core: Registries** — Tool, Provider, Listener, Guide, Command レジストリ + PluginLoader
7. **Core: SecurityLayer** — パスの正規化 + スコープ検証
8. **Core: ExecutionLoop** — IDLE → ASSEMBLING_CONTEXT → AWAITING_LLM → PROCESSING_RESPONSE → (tool_use ループ)
9. **Core: AgentCore + Transport** — メインオーケストレーター + イベントブロードキャストブリッジ
10. **Plugin: provider-gemini-oauth** — PKCE, OAuth コールバックサーバー, AES-256-GCM トークンストレージ, SSE ストリーミング, 関数呼び出し
11. **Plugin: standard-function-fs** — パスセキュリティを備えた fs.read/write/list/mkdir/delete
12. **Plugin: standard-function-stdio** — stdin リスナー + ANSI stdout UI + デフォルトガイド人格
13. **CLI: bin.ts** — 設定のロード, プラグイン解決, AgentCore ブートストラップ
14. **統合 (Integration)** — `pnpm install && pnpm build` が通過し、 CLI が起動してコマンドに応答する

## 主要な設計決定

1. **ESM のみ** — すべてのパッケージで ES モジュールを使用
2. **Zod 検証** — ツール引数を実行時に検証
3. **ファクトリパターン** — プラグインはファクトリ関数を公開
4. **非同期ジェネレーター** — プロバイダーのストリーミングに非同期ジェネレーターを使用
5. **Daemon なし** — MVP CLI が AgentCore を直接ブートストラップ (単独実行)
6. **PKCE + OAuth** — Gemini は Google OAuth を使用。トークンはマシンバインドされた AES-256-GCM で暗号化
7. **メモリ内ステート** — MVP では永続化なし。ステートはプロセスメモリ内に保持
8. **パスセキュリティ** — パスの正規化 + allowedPaths により FS ツールの利用を制限

## 検証 (Verification)

```bash
cd openstarry
pnpm install && pnpm build           # 全7パッケージがコンパイルされる
node apps/runner/dist/bin.js             # デフォルト設定で起動
node apps/runner/dist/bin.js --config ./example-agent.json  # カスタム設定で起動

# 初回使用: /provider login gemini   → ブラウザを開いて OAuth 認証
# テスト:      /help                    → 4つのコマンドをリスト表示
# テスト:      /reset                   → 対話をリセット
# テスト:      /quit                    → 正常終了
# チャット:    "こんにちは"               → Gemini が応答（ OAuth ログイン後）
# ツール:      " . 内のファイルをリストして" → エージェントが fs.list を呼び出し
```
