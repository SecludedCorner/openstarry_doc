# OpenStarry Eco 向け Windows 移行ガイド

**最終更新日**: 2026-02-13
**バージョン**: v0.19.0-beta (Cycle 25 + Windows クロスプラットフォーム修正)
**ステータス**: Linux 上での検証はすべて完了。Windows デプロイの準備が整った状態

本ガイドは、Windows 上で再現できるよう、Claude Code 環境および OpenStarry プロジェクトの完全な状態を記録するものです。

---

## 目次

1. [移行前チェックリスト](#移行前チェックリスト)
2. [Claude Code グローバル設定](#1-claude-code-グローバル設定)
3. [プロジェクト設定](#2-プロジェクト設定)
4. [エージェント定義](#3-エージェント定義)
5. [Claude Code メモリファイル](#4-claude-code-メモリファイル)
6. [現在のコード状態](#5-現在のコード状態)
7. [Windows 固有の修正](#6-windows-固有の修正)
8. [Windows セットアップ手順](#7-windows-セットアップ手順)
9. [Windows 検証チェックリスト](#8-windows-検証チェックリスト)
10. [Windows トラブルシューティング](#9-windows-トラブルシューティング)

---

## 移行前チェックリスト

- [ ] Windows 10/11（最新の更新プログラム適用済み）
- [ ] 開発者モード有効化済み（pnpm のシンボリックリンク作成に必要）
- [ ] Git Bash、WSL2、または bash がインストールされた PowerShell
- [ ] Node.js 20.0+（`node --version`）
- [ ] pnpm 9+（`pnpm --version`）
- [ ] Claude Code CLI 最新バージョン
- [ ] シンボリックリンク作成に必要な管理者アクセス権限（必要に応じて）

---

## 1. Claude Code グローバル設定

### 場所: `~/.claude/settings.json`

グローバル Claude Code 設定を作成または更新してください：

```json
{
  "alwaysThinkingEnabled": true
}
```

**Windows でのパス展開**:
- ホームディレクトリが `C:\Users\YourUsername` の場合、パスは以下のようになります：
  ```
  C:\Users\YourUsername\.claude\settings.json
  ```
- あるいは、PowerShell では `$env:USERPROFILE\.claude\settings.json` を使用してください

### 検証

```powershell
# PowerShell
cat $env:USERPROFILE\.claude\settings.json

# Git Bash
cat ~/.claude/settings.json
```

---

## 2. プロジェクト設定

### 2.1 ルート `CLAUDE.md`

**場所**: `openstarry_eco/CLAUDE.md`

このファイルはリポジトリに含まれており、コードと一緒に移行されます。内容は以下のとおりです：
- プロジェクト概要とディレクトリ構造
- Agent Corps 定義（6 エージェント）
- 標準イテレーションサイクル（SOP）
- アーキテクチャ原則（五蘊）
- ビルドおよびテストコマンド
- 命名規則
- 制約事項と要件

**変更不要** — プロジェクトルートにそのまま保持してください。

### 2.2 プロジェクト設定: `.claude/settings.local.json`

**場所**: `openstarry_eco/.claude/settings.local.json`

このファイルは **Windows パスへの更新が必要** です。

#### Linux バージョン（現在リポジトリに存在するもの）
```json
{
  "env": {
    "CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS": "1",
    "CLAUDE_CODE_TEAMMATE_MODE": "tmux"
  },
  "permissions": {
    "allow": [
      "Bash(pnpm install:*)",
      "Bash(pnpm build)",
      "Bash(pnpm add:*)",
      "Bash(pnpm --filter \"@openstarry-plugin/*\" add -D @types/node)",
      "Bash(timeout 10 node:*)",
      "Bash(printf:*)",
      "Bash(node:*)",
      "Bash(pnpm test:*)",
      "Bash(LOG_FORMAT=json echo:*)",
      "Bash(export LOG_FORMAT=json)",
      "Bash(pnpm clean:*)",
      "Bash(find:*)",
      "Bash(bash:*)",
      "Bash(pnpm -r run build:*)",
      "Bash(dir:*)",
      "Bash(pnpm test:purity:*)",
      "Bash(ls:*)",
      "Bash(pnpm clean:all:*)",
      "Bash(pnpm list:*)",
      "Bash(pnpm ls:*)",
      "Bash(grep:*)",
      "Bash(mkdir:*)",
      "WebSearch",
      "Bash(npx tsc:*)",
      "Bash(pnpm --filter @openstarry/sdk build:*)",
      "Bash(npx --yes rimraf:*)",
      "Bash(for f in scripts/*.sh)",
      "Bash(do sed -i 's/\\\\r$//' \"$f\")",
      "Bash(done)",
      "Bash(npm run build:*)",
      "Bash(npm test:*)",
      "Write(//data/openstarry_eco/agent_dev/**)",
      "Edit(//data/openstarry_eco/agent_dev/**)",
      "Write(//data/openstarry_eco/share/test/reports/**)",
      "Edit(//data/openstarry_eco/share/test/reports/**)",
      "Write(//data/openstarry_eco/share/openstarry_doc/**)",
      "Edit(//data/openstarry_eco/share/openstarry_doc/**)"
    ]
  }
}
```

#### Windows バージョン（パスを更新）

パスベースの権限を実際の Windows パスに置き換えてください。例：

```json
{
  "env": {
    "CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS": "1",
    "CLAUDE_CODE_TEAMMATE_MODE": "tmux"
  },
  "permissions": {
    "allow": [
      "Bash(pnpm install:*)",
      "Bash(pnpm build)",
      "Bash(pnpm add:*)",
      "Bash(pnpm --filter \"@openstarry-plugin/*\" add -D @types/node)",
      "Bash(timeout 10 node:*)",
      "Bash(printf:*)",
      "Bash(node:*)",
      "Bash(pnpm test:*)",
      "Bash(LOG_FORMAT=json echo:*)",
      "Bash(export LOG_FORMAT=json)",
      "Bash(pnpm clean:*)",
      "Bash(find:*)",
      "Bash(bash:*)",
      "Bash(pnpm -r run build:*)",
      "Bash(dir:*)",
      "Bash(pnpm test:purity:*)",
      "Bash(ls:*)",
      "Bash(pnpm clean:all:*)",
      "Bash(pnpm list:*)",
      "Bash(pnpm ls:*)",
      "Bash(grep:*)",
      "Bash(mkdir:*)",
      "WebSearch",
      "Bash(npx tsc:*)",
      "Bash(pnpm --filter @openstarry/sdk build:*)",
      "Bash(npx --yes rimraf:*)",
      "Bash(for f in scripts/*.sh)",
      "Bash(do sed -i 's/\\\\r$//' \"$f\")",
      "Bash(done)",
      "Bash(npm run build:*)",
      "Bash(npm test:*)",
      "Write(//C:/Users/YourUsername/Desktop/openstarry-eco/agent_dev/**)",
      "Edit(//C:/Users/YourUsername/Desktop/openstarry-eco/agent_dev/**)",
      "Write(//C:/Users/YourUsername/Desktop/openstarry-eco/share/test/reports/**)",
      "Edit(//C:/Users/YourUsername/Desktop/openstarry-eco/share/test/reports/**)",
      "Write(//C:/Users/YourUsername/Desktop/openstarry-eco/share/openstarry_doc/**)",
      "Edit(//C:/Users/YourUsername/Desktop/openstarry-eco/share/openstarry_doc/**)"
    ]
  }
}
```

**Windows 向けの主な変更点**:
- `/data/openstarry_eco` を実際の Windows パスに置き換えてください（例: `C:/Users/YourUsername/Desktop/openstarry-eco`）
- パスにはバックスラッシュではなくスラッシュ（`/`）を使用してください
- `//` プレフィックスが必須です（URL スタイルのパスセパレータ）

### 2.3 スクリプト

**場所**: `openstarry_eco/scripts/`

4 つの bash スクリプトが含まれています：

| スクリプト | 用途 |
|--------|---------|
| `baseline.sh` | Phase 1.5: 実装前のバックアップ |
| `sync-to-test.sh` | Phase 2.5: agent_dev → agent_test への同期 |
| `snapshot.sh` | Phase 4: バージョン付きスナップショット |
| `restore.sh` | リカバリ: ベースライン/スナップショットからの復元 |

**Windows での実行**:
- Git Bash を使用（`bash scripts/baseline.sh`）、または WSL を使用してください
- PowerShell で実行する場合は、`bash` が利用可能であることを確認してください：
  ```powershell
  bash scripts/baseline.sh
  ```

**CRLF 警告**: Windows 上の Git は LF を CRLF に変換する場合があります。問題を防ぐには以下のように設定してください：
```bash
# Git Bash にて
git config core.autocrlf input
```

または手動で改行コードを修正してください：
```bash
# Git Bash にて
for f in scripts/*.sh; do sed -i 's/\r$//' "$f"; done
```

---

## 3. エージェント定義

### 場所: `openstarry_eco/.claude/agents/`

6 つのエージェント定義ファイルが含まれています。Windows プロジェクトにそのままコピーしてください：

| エージェント | ファイル | 役割 |
|-------|------|------|
| architect | `architect.md` | アーキテクチャガーディアン、設計仕様、コードレビュー |
| dev-core | `dev-core.md` | コアモノレポ開発者 |
| dev-plugin | `dev-plugin.md` | プラグイン開発者 |
| doc-keeper | `doc-keeper.md` | ドキュメント管理、意思決定の永続化 |
| qa | `qa.md` | テスト実行、品質検証 |
| researcher | `researcher.md` | 技術事前調査、リファレンスプロジェクト分析 |

**変更不要** — ディレクトリをそのままコピーしてください。

---

## 4. Claude Code メモリファイル

Claude Code はプロジェクト固有のメモリを以下の 2 か所に保存します：

### 4.1 Eco レベルメモリ

**Linux パス**: `~/.claude/projects/-data-openstarry-eco/memory/`

**Windows パス**: `$env:USERPROFILE\.claude\projects\{encoded-windows-path}\memory\`

パスのエンコーディングは Windows では異なります。プロジェクトが `C:\Users\YourUsername\Desktop\openstarry-eco` にある場合、エンコードされたパスは以下のようになります：
```
C--Users-YourUsername-Desktop-openstarry-eco
```

**このディレクトリにコピーするファイル**:

#### `MEMORY.md`
```markdown
# OpenStarry Eco Memory

## Current State (as of Cycle 25 complete, 2026-02-13)
- **Version**: v0.19.0-beta (Cycle 25 snapshot + Windows fixes verified)
- **Active Cycle**: 25 complete (Plan22: Plugin Marketplace MVP)
- **Tests**: 1330 in agent_test (117 files), ~1336 in agent_dev (117 files)
- **Packages**: 18 core workspace + 15 plugins
- **Latest snapshot**: `20260213_cycle25`
- **All Plans 01-22 complete**
- **Windows Status**: Ready for deployment

## Key Patterns
- See [patterns.md](patterns.md) for implementation patterns
- See [debugging.md](debugging.md) for known issues and fixes
- See [windows-fixes.md](windows-fixes.md) for Windows-specific fixes applied

## Critical Conventions
- Plugin factory: `createXxxPlugin()` → IPlugin with manifest + factory(ctx)
- Five Aggregates: IUI(色), IListener(受), IProvider(想), ITool(行), IGuide(識)
- pushInput pattern: plugins use ctx.pushInput(), never direct API calls
- Lazy accessors: ctx.tools, ctx.guides, ctx.providers — all optional, sync, read-only

## Known Build Issues
- **sync-to-test.sh**: Stale tsbuildinfo — ALWAYS clean before sync:
  `find agent_test -name "tsconfig.tsbuildinfo" -not -path "*/node_modules/*" -delete`
- **snapshot.sh**: Fails if dir exists — delete first: `rm -rf share/openstarry_code_iteration/{id}`
- **runner build**: plugin-catalog.json must be copied to dist/data/ (handled by build script)

## Agent Corps SOP
- Standard cycle: Phase 0→1→1.5→2→2.5→3→4
- Interface freeze after Phase 1 (Arch Spec)
- QA + Architect run in parallel in Phase 3
- Max 2 rework cycles before user escalation
```

#### `patterns.md`
```markdown
# Implementation Patterns

## Lazy Accessor Pattern (IPluginContext)
Used for tools, guides, providers. Sync stubs in sandbox + RPC for actual access.

## Sandbox RPC Pattern
REQUEST/RESPONSE pairs for cross-boundary communication:
- TOOLS_LIST_REQUEST → TOOLS_LIST_RESPONSE
- GUIDES_LIST_REQUEST → GUIDES_LIST_RESPONSE
- PROVIDERS_LIST_REQUEST → PROVIDERS_LIST_RESPONSE
- PROVIDERS_GET_REQUEST → PROVIDERS_GET_RESPONSE

## Event Type Constants
All in `packages/sdk/src/events/types.ts` as `AgentEventType`.

## SessionConfig Pattern
Typed metadata access: `getSessionConfig(session?.metadata)`

## Plugin Integration Order
SDK first → Core → Plugins (dependencies flow downward)
```

#### `windows-fixes.md`
```markdown
# Windows Cross-Platform Fixes (Applied 2026-02-13)

## Status: Applied on Linux (1330 tests all passing), ready for Windows verification

## Fix 1: Sandbox Worker Plugin Path Resolution
**File**: `apps/runner/src/utils/plugin-resolver.ts` (lines 122-131)
**Change**: Use `import.meta.resolve()` (ESM native, Node 20.6+) as primary, `createRequire` as fallback
**Impact**: Plugin loading in sandbox worker works on Windows with long paths

## Fix 2: Plugin Catalog Copying
**File**: `apps/runner/package.json` (build script)
**Change**: `"build": "tsc -b && node -e \"require('fs').cpSync('src/data','dist/data',{recursive:true})\""`
**Impact**: plugin-catalog.json copied to dist/ for runtime loading

## Fix 3: Symlink Dereferencing
**File**: `apps/runner/src/utils/plugin-installer.ts` (lines 100, 144)
**Change**: Added `dereference: true` to both `cp()` calls
**Impact**: Handles Windows symlink permissions gracefully

## Fix 4: Audit Logger Test Timing
**File**: `packages/core/src/sandbox/__tests__/audit-logger.test.ts` (lines 256-274)
**Change**: bufferSize 1→100, removed setTimeout, use dispose() properly
**Impact**: Test reliability on Windows with slower I/O

## Verification Checklist
- [ ] pnpm build succeeds
- [ ] apps/runner/dist/data/plugin-catalog.json exists
- [ ] pnpm test passes all 1330 tests
- [ ] CLI works: node apps/runner/dist/bin.js --help
- [ ] Plugin search works: node apps/runner/dist/bin.js plugin search fs
- [ ] Plugin loading works with actual agent config
```

### 4.2 モノレポレベルメモリ

**Linux パス**: `~/.claude/projects/-data-openstarry-eco-agent-dev-openstarry/memory/`

**Windows パス**: `$env:USERPROFILE\.claude\projects\{encoded-windows-path-agent-dev-openstarry}\memory\`

**コピーするファイル**:

#### `MEMORY.md`
上記と同じ内容（Cycle 25 の状態）

#### `patterns.md`
全実装パターンを含む完全なパターンドキュメント

#### `debugging.md`
既知の問題とデバッグ手法（Windows ビルドで問題が発生した場合に参照してください）

---

## 5. 現在のコード状態

### バージョン情報
- **バージョン**: v0.19.0-beta
- **サイクル**: 25 完了（Plan22: Plugin Marketplace MVP）
- **ビルド状態**: クリーン（dist/、tsconfig.tsbuildinfo はクリーニング済み）

### パッケージ

#### コアワークスペース（18 パッケージ）
```
agent_dev/openstarry/
├── packages/
│   ├── sdk/                  ← SDK 型定義
│   ├── core/                 ← エージェントコアエンジン
│   ├── shared/               ← 共有ユーティリティ
│   └── 15 more core packages
└── apps/
    └── runner/               ← CLI エントリーポイント
```

**依存関係**:
- ルート: vitest 4.0.18
- 全パッケージ: Node.js >= 20.0.0
- パッケージマネージャ: pnpm 9+

#### プラグインワークスペース（15 プラグイン）
```
agent_dev/openstarry_plugin/
├── mcp-server/               ← MCP プロトコルサーバー
├── mcp-client/               ← MCP クライアント統合
├── tui-dashboard/            ← ターミナル UI ダッシュボード
├── workflow-engine/          ← ワークフロー実行
├── devtools/                 ← 開発ツール
├── transport-websocket/      ← WebSocket トランスポート
├── transport-http/           ← HTTP トランスポート
├── http-static/              ← 静的 HTTP サーバー
├── web-ui/                   ← Web UI プラグイン
├── provider-gemini-oauth/    ← Gemini OAuth プロバイダー
├── standard-function-*/ (3)  ← ファイルシステム、stdio、スキル
├── guide-character-init/     ← キャラクター初期化
└── More plugins in development
```

### テスト状態
- **Agent Dev**: テストファイル 117 件、テスト 1330 件、すべて合格
- **Agent Test**: 同期の準備完了
- **Windows での予想結果**: 同じ 1330 件のテストが合格するはずです

---

## 6. Windows 固有の修正

Windows 互換性を確保するため、4 つの重要な修正が適用されています：

### 修正 1: プラグインリゾルバーのパス解決

**ファイル**: `agent_dev/openstarry/apps/runner/src/utils/plugin-resolver.ts` (122-131 行目)

**問題**: ESM/CJS のパス解決の違いにより、サンドボックスワーカーが Windows 上でプラグインを解決できない問題が発生していました。

**解決策**: `import.meta.resolve()`（ESM ネイティブ、Node 20.6+）を主要な解決方法として使用し、`createRequire` をフォールバックとして使用します。

```typescript
// Strategy 3: Try import.meta.resolve() first (ESM native)
try {
  _resolvedModulePath = await import.meta.resolve(ref.name);
} catch {
  // Fallback to createRequire
  const { createRequire } = await import('module');
  const require = createRequire(import.meta.url);
  _resolvedModulePath = require.resolve(ref.name);
}
```

### 修正 2: プラグインカタログデータのコピー

**ファイル**: `agent_dev/openstarry/apps/runner/package.json`

**問題**: TypeScript コンパイラは `.ts` 以外のファイルをコピーしません。`plugin-catalog.json` はランタイム時に `dist/data/` に存在する必要があります。

**解決策**: ビルドスクリプトを変更し、明示的にデータディレクトリをコピーするようにしました：

```json
{
  "scripts": {
    "build": "tsc -b && node -e \"require('fs').cpSync('src/data','dist/data',{recursive:true})\""
  }
}
```

### 修正 3: シンボリックリンクの参照解決

**ファイル**: `agent_dev/openstarry/apps/runner/src/utils/plugin-installer.ts` (100 行目、144 行目)

**問題**: pnpm は node_modules 内にシンボリックリンクを作成します。Windows ではシンボリックリンクの作成に特別な権限が必要です。`cp()` 関数はデフォルトではシンボリックリンクを再作成しようとします。

**解決策**: シンボリックリンクを再作成する代わりにリンク先を辿るよう、両方の `cp()` 呼び出しに `dereference: true` を追加しました：

```typescript
// Line 100: workspace copy
cp(src, dest, { dereference: true, recursive: true });

// Line 144: npm pack extract copy
cp(srcPath, destPath, { dereference: true, recursive: true });
```

### 修正 4: 監査ロガーのテストタイミング

**ファイル**: `agent_dev/openstarry/packages/core/src/sandbox/__tests__/audit-logger.test.ts` (256-274 行目)

**問題**: バッファフラッシュのタイミングに起因する Windows 上でのテストの不安定性。`bufferSize: 1` により自動フラッシュが fire-and-forget として実行され、適切なエラーキャプチャができませんでした。

**解決策**: bufferSize を増加させ、setTimeout を削除しました：

```typescript
// Changed from bufferSize: 1
const logger = new AuditLogger({ bufferSize: 100 });

// Removed setTimeout(100)
// Let dispose() properly await the flush
await logger.dispose();
```

---

## 7. Windows セットアップ手順

### ステップ 1: 前提条件

```powershell
# Node.js の確認
node --version  # 20.0+ が必要

# pnpm の確認
pnpm --version  # 9+ が必要

# 開発者モードの有効化（シンボリックリンクに必要）
# 設定 → システム → 開発者向け → 開発者モードを有効にする
```

### ステップ 2: プロジェクトのクローン/コピー

Git からクローンする場合：
```powershell
git clone https://github.com/your-repo/openstarry-eco.git
cd openstarry-eco

# 必要に応じて改行コードを修正
git config core.autocrlf input
for ($f in dir scripts\*.sh) { (Get-Content $f) -replace "`r$", "" | Set-Content $f }
```

または、プロジェクトディレクトリを直接 Windows マシンにコピーしてください。

### ステップ 3: Claude Code の設定

1. `$env:USERPROFILE\.claude\settings.json` を開き、以下を追加してください：
   ```json
   {
     "alwaysThinkingEnabled": true
   }
   ```

2. `openstarry_eco\.claude\settings.local.json` を更新してください：
   - `/data/openstarry_eco` パスを実際の Windows パスに置き換えてください
   - 例: `C:/Users/YourUsername/Desktop/openstarry-eco`

3. `.claude/agents/` ディレクトリに 6 つのエージェントファイルが存在することを確認してください

### ステップ 4: 依存関係のインストール

```powershell
cd openstarry_eco
pnpm install
```

pnpm のシンボリックリンク作成が失敗した場合：
- 開発者モードが有効であることを確認してください
- または PowerShell を管理者として実行してください
- または `pnpm install --force` を使用してください

### ステップ 5: プロジェクトのビルド

```powershell
pnpm build
```

ビルド成果物の確認：
```powershell
# plugin-catalog.json がコピーされたことを確認
Test-Path agent_dev\openstarry\apps\runner\dist\data\plugin-catalog.json
```

### ステップ 6: Claude Code メモリの初期化

1. Claude Code でプロジェクトを開いてください
2. `openstarry_eco` ディレクトリに移動してください
3. これにより `~\.claude\projects\` にプロジェクトパスのエンコーディングが作成されます
4. メモリディレクトリを作成してください：
   ```powershell
   mkdir $env:USERPROFILE\.claude\projects\{encoded-path}\memory -Force
   ```
5. メモリファイルをコピーしてください

---

## 8. Windows 検証チェックリスト

セットアップ後、以下のチェックを実行してすべてが正常に動作することを確認してください：

```powershell
# 1. インストールとビルド
cd openstarry_eco
pnpm clean:all
pnpm install
pnpm build

# 2. plugin-catalog.json の確認
Test-Path agent_dev\openstarry\apps\runner\dist\data\plugin-catalog.json
# 出力: True が期待されます

# 3. テストの実行
pnpm test
# 1330 件のテストが合格するはずです

# 4. CLI スモークテスト
node agent_dev\openstarry\apps\runner\dist\bin.js --help
# 使用方法の情報が表示されるはずです

# 5. プラグイン検索テスト
node agent_dev\openstarry\apps\runner\dist\bin.js plugin search fs
# ファイルシステムプラグインが一覧表示されるはずです

# 6. (オプション) エージェントの起動
# テスト設定を作成するか、既存の agent.json を使用してください
node agent_dev\openstarry\apps\runner\dist\bin.js start --config path\to\config.json
# プラグインがエラーなしでロードされるはずです
```

すべてのチェックが合格すれば、Windows セットアップは完了です。

---

## 9. Windows トラブルシューティング

### 問題: `pnpm install` がシンボリックリンクエラーで失敗する

**解決策**:
1. Windows 開発者モードを有効にしてください: 設定 → システム → 開発者向け → 開発者モードを有効にする
2. または PowerShell を管理者として実行してください
3. または `pnpm install --force` を使用してください

### 問題: `tsc` や `vitest` が見つからない

**解決策**:
```powershell
# クリーンキャッシュで再インストール
pnpm install --force
pnpm build
```

### 問題: プラグインの読み込みに失敗する（サンドボックスワーカーエラー）

**解決策**:
- `plugin-catalog.json` が存在することを確認してください: `agent_dev\openstarry\apps\runner\dist\data\plugin-catalog.json`
- リビルドしてください: `pnpm build`
- パス解決エラーがないかログを確認してください

### 問題: テストがハングまたはタイムアウトする

**解決策**:
- タイムアウトを延長してください: `pnpm test -- --reporter=verbose`
- 低速なファイルシステム（USB ドライブ、ネットワーク共有）でのシンボリックリンクの問題を確認してください
- `pnpm install --prefer-offline` を試してください

### 問題: スクリプトで CRLF 改行コードエラーが発生する

**解決策**:
```bash
# Git Bash を使用
for f in scripts/*.sh; do sed -i 's/\r$//' "$f"; done

# または dos2unix を使用（インストール済みの場合）
dos2unix scripts/*.sh
```

### 問題: パス長が Windows の 260 文字制限を超えている

**解決策**:
1. Windows でロングパスを有効にしてください：
   ```powershell
   New-ItemProperty -Path "HKLM:\SYSTEM\CurrentControlSet\Control\FileSystem" -Name "LongPathsEnabled" -Value 1 -Force
   ```
2. マシンを再起動してください
3. ビルドを再試行してください

### 問題: Claude Code がプロジェクトを見つけられない

**解決策**:
1. `openstarry_eco\CLAUDE.md` が存在することを確認してください
2. `openstarry_eco\.claude\settings.local.json` のパスが正しいことを確認してください
3. Claude Code を再起動してください
4. プロジェクトフォルダを明示的に開いてください

---

## 移行後チェックリスト

Windows セットアップ完了後に確認してください：

- [ ] 1330 件のテストがすべて合格
- [ ] CLI スモークテストが正常に動作
- [ ] 6 つのエージェントすべてが定義済みでアクセス可能
- [ ] メモリファイルが配置済み
- [ ] プロジェクト設定が構成済み
- [ ] ドキュメントにアクセス可能（share/openstarry_doc/）
- [ ] スクリプトが実行可能（baseline.sh、sync-to-test.sh など）
- [ ] 次のイテレーションサイクルの準備完了

---

## 追加リソース

- **プロジェクト手順書**: `openstarry_eco/CLAUDE.md`
- **エージェント SOP**: `share/openstarry_doc/Agent_Corps/Agent_Roles_and_SOP.md`
- **アーキテクチャドキュメント**: `share/openstarry_doc/Architecture_Documentation/`
- **実装例**: `share/openstarry_doc/Implementation_Examples/`
- **現在のメモリ**: Claude Code メモリを `~/.claude/projects/` で確認してください

---

## Windows 固有の注意事項

1. **シンボリックリンク**: pnpm はシンボリックリンクを使用します。Windows では開発者モードを有効にするか、管理者として実行してください。
2. **パスセパレータ**: 設定ファイルにはスラッシュ（`/`）を使用し、PowerShell/cmd ではバックスラッシュ（`\`）を使用してください。
3. **Bash スクリプト**: Git Bash または WSL を使用して実行してください。PowerShell では `bash` が利用可能である必要があります。
4. **ビルド成果物**: 初回ビルドでは dist/ と tsbuildinfo がクリーンアップされます。以降のビルドはインクリメンタルです。
5. **環境変数**: export 文（`export VAR=val`）を PowerShell の形式（`$env:VAR = "val"`）に変更してください。

---

**ご質問や問題がありましたら、** トラブルシューティングセクションを参照するか、プロジェクトのメインドキュメントをご確認ください。
