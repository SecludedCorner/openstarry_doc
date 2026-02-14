# OpenStarry テストおよびリリースガイド

**バージョン**: v0.20.1-beta (Cycle 26 — Attach UX + Token Persistence Fix)
**日付**: 2026-02-14
**ステータス**: release ビルド完了、手動テスト待ち

---

## プロジェクト構造と Clone 手順

OpenStarry は 2 つの GitHub リポジトリに分かれており、**必ず同じ親フォルダの直下に配置する必要があります**：

```
your-workspace/              ← 任意の親フォルダ
├── openstarry/              ← Repo 1: コアモノレポ (SDK, Core, Shared, Runner)
│   ├── packages/
│   │   ├── sdk/             ← @openstarry/sdk
│   │   ├── core/            ← @openstarry/core
│   │   ├── shared/          ← @openstarry/shared
│   │   └── plugin-signer/   ← @openstarry/plugin-signer
│   ├── apps/
│   │   └── runner/          ← @openstarry/runner (CLI エントリーポイント)
│   ├── configs/             ← Agent 設定ファイルのサンプル
│   ├── pnpm-workspace.yaml  ← 重要: ../openstarry_plugin/* を含む
│   └── package.json
└── openstarry_plugin/       ← Repo 2: プラグインエコシステム (15 個の公式プラグイン)
    ├── standard-function-fs/
    ├── standard-function-stdio/
    ├── transport-websocket/
    ├── web-ui/
    ├── workflow-engine/
    ├── ... (その他のプラグイン)
    └── package.json
```

### Clone 手順

```bash
# 作業ディレクトリを作成
mkdir openstarry-workspace && cd openstarry-workspace

# 2 つのリポジトリを同じ階層に Clone
git clone <openstarry-repo-url> openstarry
git clone <openstarry-plugin-repo-url> openstarry_plugin
```

### なぜ同じ階層に配置する必要があるのか

`openstarry/pnpm-workspace.yaml` に以下の設定があります：

```yaml
packages:
  - apps/*
  - packages/*
  - ../openstarry_plugin/*     ← 相対パスでプラグインリポジトリを参照
```

すべてのプラグインの `package.json` も `workspace:*` や `link:../../openstarry/packages/sdk` を通じてコアパッケージを参照しています。2 つのリポジトリが同じ階層にない場合、pnpm はクロスリポジトリの依存関係を解決できません。

---

## 前提条件

- **Node.js** >= 20
- **pnpm** >= 10 (`npm install -g pnpm`)
- 2 つのリポジトリを同じ親フォルダに Clone 済み（上記参照）
- Agent の会話機能をテストする場合は、Gemini OAuth ログインが必要（`/provider login gemini`）

---

## Step 1: クリーニング（再テストの場合）

以前にビルドしたことがある場合は、まず完全なクリーニングを行ってください：

```bash
cd openstarry        # コアモノレポに移動
pnpm clean:all
```

これにより、すべての `node_modules/`、`dist/`、`tsconfig.tsbuildinfo`、`pnpm-lock.yaml`、および `../openstarry_plugin/` 配下の `node_modules/` と `dist/` が削除されます。

> `clean:all` はクロスプラットフォーム対応の Node スクリプト（`scripts/clean-all.mjs`）を使用しており、Windows と Linux の両方で正常に動作します。
> 新規 Clone の場合は、このステップをスキップできます。

---

## Step 2: 依存関係のインストール

```bash
cd openstarry        # コアモノレポに移動（まだ移動していない場合）
pnpm install
```

期待される結果: エラーなし。21 個のワークスペースパッケージ（openstarry_plugin 配下のプラグインを含む）がすべてインストール完了。

> **注意**: 必ず `openstarry/` ディレクトリから実行してください。pnpm はワークスペース設定を通じて `../openstarry_plugin/*` 配下のプラグインを自動的に解決します。

---

## Step 3: ビルド

```bash
pnpm build
```

期待される結果: 20 個のパッケージがすべてビルド成功。

---

## Step 4: 自動テストの実行

```bash
pnpm test
```

期待される結果:
- **Test Files**: 117 passed (117)
- **Tests**: 1339 passed | 3 skipped (1342)
- 既知のスキップ: attach.test.ts 内の daemon socket タイムアウトテスト 1 件 + 環境依存のテスト 2 件（バグではありません）

---

## Step 5: CLI コマンドテスト

以下のすべてのコマンドは `openstarry/` ルートディレクトリから実行してください：

### 5.1 バージョン情報

```bash
node apps/runner/dist/bin.js version
```

期待: バージョン番号が表示されます。

### 5.2 ヘルプメッセージ

```bash
node apps/runner/dist/bin.js --help
```

期待: 利用可能なすべてのコマンド一覧が表示されます。以下を含みます：
- `start`, `daemon start`, `daemon stop`, `attach`, `ps`
- `init`, `create-plugin`
- `plugin install`, `plugin uninstall`, `plugin list`, `plugin search`, `plugin info`, `plugin sync`
- `version`

### 5.3 設定の初期化

```bash
node apps/runner/dist/bin.js init
```

期待: 現在のディレクトリに `agent.json` 設定ファイルが生成されます（既に存在する場合はプロンプトが表示されます）。

---

## Step 6: Plugin Marketplace コマンドテスト

### 6.1 プラグインの検索

```bash
node apps/runner/dist/bin.js plugin search fs
```

期待: "fs" を含むプラグイン一覧が表示されます（例: `standard-function-fs`）。

```bash
node apps/runner/dist/bin.js plugin search websocket
```

期待: `transport-websocket` 関連のプラグインが表示されます。

### 6.2 プラグイン情報の表示

```bash
node apps/runner/dist/bin.js plugin info standard-function-fs
```

期待: 該当プラグインの name、version、description、aggregates、tags が表示されます。

```bash
node apps/runner/dist/bin.js plugin info @openstarry-plugin/web-ui
```

期待: 上記と同様。完全名でも解決できます。

### 6.3 全プラグインの一覧表示（カタログ）

```bash
node apps/runner/dist/bin.js plugin list --all
```

期待: 15 個の公式プラグインすべてが一覧表示され、`[installed]` または `[available]` のマークが付きます。

### 6.4 インストール済みプラグインの一覧表示

```bash
node apps/runner/dist/bin.js plugin list
```

期待: まだインストールしていない場合は "No plugins installed" と表示されます。

### 6.5 単一プラグインのインストール

```bash
node apps/runner/dist/bin.js plugin install standard-function-fs
```

期待: ワークスペースから解決してインストールし、ロックファイル（`~/.openstarry/plugins/lock.json`）を更新します。

### 6.6 インストール成功の確認

```bash
node apps/runner/dist/bin.js plugin list
```

期待: インストールしたばかりの `@openstarry-plugin/standard-function-fs` が表示されます。

### 6.7 全プラグインのインストール

```bash
node apps/runner/dist/bin.js plugin install --all
```

期待: 15 個の公式プラグインすべてがインストールされ、既にインストール済みのものはスキップされます。

### 6.8 再度一覧表示

```bash
node apps/runner/dist/bin.js plugin list
```

期待: 15 個のプラグインすべてがインストール済みとして表示されます。

### 6.9 プラグインのアンインストール

```bash
node apps/runner/dist/bin.js plugin uninstall standard-function-fs
```

期待: 該当プラグインが削除され、ロックファイルが更新されます。

### 6.10 強制再インストール

```bash
node apps/runner/dist/bin.js plugin install standard-function-fs --force
```

期待: 既にインストール済みであっても再インストールされます。

---

## Step 7: Agent 起動テスト

### 7.1 Basic Agent（CLI モード）

```bash
node apps/runner/dist/bin.js start --config configs/basic-agent.json
```

期待: Agent が起動し、CLI がユーザー入力を待機します。任意のテキストを入力して会話をテストし、`Ctrl+C` で終了してください。

> **注意**: 初回起動時には `[gemini-oauth] Not logged in` と表示されます。`/provider login gemini` でログインするとトークンが永続化され、以降は再ログイン不要です。

### 7.2 TUI Agent（ターミナルインターフェースモード）

```bash
node apps/runner/dist/bin.js start --config configs/tui-agent.json
```

期待: Ink ベースの TUI インターフェースが起動し、ダッシュボードが表示されます。`Ctrl+C` で終了してください。

### 7.3 WebSocket Agent（ヘッドレスモード）

```bash
node apps/runner/dist/bin.js start --config configs/websocket-agent.json
```

期待: Agent が起動し、`ws://0.0.0.0:8080/ws` でリッスンします。WebSocket クライアントで接続テストが可能です。

### 7.4 Web Agent（ブラウザインターフェース）

```bash
node apps/runner/dist/bin.js start --config configs/web-agent.json
```

期待:
- WebSocket サーバーが `ws://0.0.0.0:8080/ws` で動作
- Web UI が `http://0.0.0.0:8081` で動作
- ブラウザで `http://localhost:8081` を開くとチャットインターフェースが表示

---

## Step 8: Daemon モードテスト

### 8.1 Daemon の起動

```bash
node apps/runner/dist/bin.js daemon start --config configs/basic-agent.json
```

期待: Agent がバックグラウンド Daemon として起動します。

### 8.2 実行中の Agent の確認

```bash
node apps/runner/dist/bin.js ps
```

期待: `basic-agent`（または設定内の agent ID）が一覧表示されます。

### 8.3 Daemon へのアタッチ

```bash
node apps/runner/dist/bin.js attach basic-agent
```

期待:
- バックグラウンド Agent の IPC チャネルに接続
- ウェルカムメッセージが表示（`/help` のヒントを含む）
- **プロバイダーのログイン状態が自動表示**（例: "Gemini OAuth: Logged in" または "Not logged in"）

### 8.4 Daemon の停止

```bash
node apps/runner/dist/bin.js daemon stop basic-agent
```

期待: バックグラウンド Agent が停止します。

---

## Step 9: Plugin Scaffolding テスト

```bash
cd /tmp
node <path-to-openstarry>/apps/runner/dist/bin.js create-plugin my-test-plugin
```

期待: `/tmp` に `my-test-plugin/` ディレクトリが生成され、以下を含みます：
- `package.json`
- `tsconfig.json`
- `src/index.ts`（factory テンプレート付き）

---

## Step 10: Plugin Sync テスト

```bash
node apps/runner/dist/bin.js plugin sync --source ../openstarry_plugin
```

期待: ソースディレクトリからシステムプラグインディレクトリ（`~/.openstarry/plugins/system/`）にプラグインが同期されます。

---

## Step 11: Workflow Engine テスト（Cycle 23 機能）

```bash
# テスト用 workflow YAML を作成
cat > /tmp/test-workflow.yaml << 'EOF'
name: hello-workflow
version: "1.0"
steps:
  - id: greet
    type: tool
    tool: fs.list
    input:
      path: "."
EOF
```

workflow コマンドを使用するには、Agent を起動し workflow-engine プラグインをロードする必要があります。

---

## 完全なプラグイン一覧（15 個）

| # | プラグイン名 | 五蘊分類 | 説明 |
|---|----------|---------|------|
| 1 | standard-function-fs | ITool | ファイル読み書き操作 |
| 2 | standard-function-stdio | ITool | 標準入出力 |
| 3 | standard-function-skill | ITool | スキル実行 |
| 4 | guide-character-init | IGuide | キャラクター初期化ガイド |
| 5 | provider-gemini-oauth | IProvider | Gemini OAuth プロバイダー |
| 6 | tui-dashboard | IUI | ターミナルインターフェース (Ink) |
| 7 | transport-websocket | IListener | WebSocket トランスポート |
| 8 | transport-http | IListener | HTTP トランスポート |
| 9 | http-static | IListener | 静的ファイルサービング |
| 10 | web-ui | IUI | ブラウザチャットインターフェース |
| 11 | mcp-client | IListener/ITool | MCP クライアント |
| 12 | mcp-server | IListener | MCP サーバー |
| 13 | mcp-common | (共用) | MCP 共通型定義 |
| 14 | devtools | ITool | 開発者ツール |
| 15 | workflow-engine | ITool | ワークフローエンジン |

---

## 設定ファイル説明 (configs/)

| 設定ファイル | 説明 | プラグイン |
|--------|------|------|
| `basic-agent.json` | 最小構成 CLI Agent | fs + stdio + guide + gemini |
| `example-agent.json` | サンプル Agent | （basic と同一） |
| `full-agent.json` | 全機能 Agent | すべてのプラグイン |
| `mcp-agent.json` | MCP 統合 Agent | mcp-client + mcp-server |
| `tui-agent.json` | ターミナルインターフェース Agent | tui-dashboard |
| `web-agent.json` | ブラウザインターフェース Agent | transport-websocket + web-ui |
| `websocket-agent.json` | WebSocket ヘッドレス Agent | transport-websocket |

---

## テスト結果記録

| ステップ | 項目 | 結果 | 備考 |
|------|------|------|------|
| 2 | pnpm install | | |
| 3 | pnpm build | | |
| 4 | pnpm test (1339) | | |
| 5.1 | version | | |
| 5.2 | --help | | |
| 5.3 | init | | |
| 6.1 | plugin search | | |
| 6.2 | plugin info | | |
| 6.3 | plugin list --all | | |
| 6.5 | plugin install (single) | | |
| 6.7 | plugin install --all | | |
| 6.9 | plugin uninstall | | |
| 7.1 | basic-agent start | | |
| 7.2 | tui-agent start | | |
| 7.3 | websocket-agent start | | |
| 7.4 | web-agent start | | |
| 8.1 | daemon start | | |
| 8.3 | attach (auto provider status) | | |
| 8.4 | daemon stop | | |
| 9 | create-plugin | | |
| 10 | plugin sync | | |
