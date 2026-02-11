# 10. ビルドと配布戦略 (Build & Distribution Strategy)

このドキュメントでは、 OpenStarry の「分離型ビルド」仕様を定義します。核心的な目標は、 **エージェントコアと調整レイヤーの絶対的な純粋性** を維持し、カーネルにいかなるプラグインコードも含まれないことを保証し、インストール時に物理的な分離を実現することです。

## 1. 核心的な純粋性の原則 (The Purity Principle)

*   **カーネルへの同梱禁止 (Zero Bundling)：** `packages/core` のビルド成果物の中に、 `plugins/` ディレクトリ下のコードを含めることは **厳禁** です。カーネルは `packages/sdk` のインターフェース定義のみに依存します。
*   **動的ロード：** カーネルは実行時にプラグインローダー (PluginLoader) を介して、外部パスから動的に `.js` ファイルをロードしなければなりません。

## 2. ビルド成果物のレイアウト (Build Artifacts)

`pnpm build` を実行した後、 `dist/` ディレクトリは以下の構造を持つ必要があります：

```text
dist/
├── bin/
│   └── openstarry-core.js  # [絶対的に純粋] カーネル実行ファイル
├── lib/
│   ├── sdk.js              # SDK ライブラリ
│   └── shared.js           # 共有ユーティリティライブラリ
└── assets/
    └── standard-plugins/    # [分離して保管] 配布用の標準プラグイン
        ├── tool-fs/        # コンパイル済みのプラグイン A
        └── listener-stdio/ # コンパイル済みのプラグイン B
```

## 3. インストールロジック (Installation Logic)

インストールファイル（またはインストールスクリプト）は、実行時に「デジタル種」の生存環境を構築するために、以下のタスクを完了しなければなりません：

### A. システムのブートストラッピング (System Bootstrapping)
1.  `bin/openstarry-core` をシステムの実行パス（例： `/usr/local/bin` や `%ProgramFiles%` ）にコピーします。
2.  ユーザーの作業ディレクトリを初期化します： `~/.openstarry/` (Linux/macOS) または `%USERPROFILE%\.openstarry` (Windows)。

### B. プラグインの搬送 (Plugin Relocation)
`assets/standard-plugins/*` 内のすべてのプラグインを、グローバルプラグインディレクトリに **移動またはコピー** します：
*   **ターゲットパス：** `~/.openstarry/plugins/`

## 4. 実行時の発見メカニズム (Runtime Discovery)

エージェントコアの起動時、インストールディレクトリからプラグインを探す **べきではなく** 、以下の優先順位に従ってシステムパスをスキャンする必要があります：

1.  **プロジェクトディレクトリ：** `./.openstarry/plugins/` (存在する場合)
2.  **システムグローバルディレクトリ：** `~/.openstarry/plugins/` (これはインストーラーによって搬送された場所です)

## 5. 開発者向け同期コマンド (Dev Sync)

開発の利便性のために、上記のインストールロジックをシミュレートする `pnpm run plugins:sync` コマンドを提供します：
*   **ロジック：** すべての `plugins/` を自動的にコンパイルし、その成果物を直接 `~/.openstarry/plugins/` へシンボリックリンク (Symlink) を作成することで、「開発即インストール」を実現します。
