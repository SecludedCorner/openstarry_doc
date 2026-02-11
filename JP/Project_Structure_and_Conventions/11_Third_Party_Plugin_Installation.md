# 11. サードパーティプラグインインストールガイド (Third-Party Plugin Installation)

このドキュメントでは、 OpenStarry システムが公式リポジトリ ( `openstarry_plugin` ) やその他のソースからプラグインをどのようにインストールするかを定義します。

## 1. サポートされるインストールソース (Installation Sources)

OpenStarry のインストーラー ( `openstarry plugin add` ) は、以下の3つのソースをサポートしています：

1.  **ローカルパス (Local Path)：** すでにダウンロードまたは解凍済みのプラグインフォルダ。
2.  **Git リポジトリ (Git Repository)：** GitHub/GitLab などを直接指す URL 。
3.  **NPM レジストリ (NPM Registry)：** npm に公開されている標準パッケージ。

## 1.1 標準的なインストールフロー：リポジトリ同期 (Repository Sync)

これは OpenStarry が **最も推奨する** プラグイン管理方法です。システムの標準プラグインライブラリは個別にダウンロードするのではなく、公式のエコシステムリポジトリと同期することで取得します。

### 操作コマンド
```bash
# 公式リポジトリを同期する (あらかじめ openstarry_plugin をローカルに clone しておく必要があります)
$ openstarry plugin sync <path-to-openstarry_plugin-repo>
```

### 実行ロジック
1.  **ソーススキャン：** 指定された `openstarry_plugin` ローカルリポジトリディレクトリをスキャンします。
2.  **増分更新：** `~/.openstarry/plugins/` 内のバージョンと比較し、新しいバージョンのプラグインをコピーします。
3.  **依存関係チェック (Install-time Check)：** 
    *   新しくインストールされたプラグインの `dependencies` リストを即座にチェックします。
    *   **不足がある場合：** CLI が直ちに黄色い警告を出力します：
        > "⚠️ Installed [Workflow], but it requires [Skill]. Please install [Skill] to enable full functionality."
4.  **自動登録：** 調整レイヤー (Daemon) が、同期されたすべてのプラグインを自動的にインデックス登録します。

---

## 2. 開発者モード：一括登録 ( `add --all` )

複数のプラグインを含む「プラグインバンドル (Plugin Bundle)」やプライベートリポジトリを開発した場合、このコマンドを使用して一度に登録できます。

### 操作コマンド
```bash
# 現在のディレクトリ以下のすべてのプラグインを一括登録する
$ cd my-private-plugins-repo
$ openstarry plugin add --all .
```

### 実行ロジック
1.  **再帰スキャン：** ターゲットディレクトリを再帰的にスキャンし、 `plugin.json` または `package.json` ( `openstarry` フィールドを含む) を持つすべてのサブディレクトリを探します。
2.  **インプレースリンク (Symlink) またはコピー：**
    *   デフォルトでは、開発デバッグを容易にするために、 `~/.openstarry/plugins/` への **シンボリックリンク (Symlink)** の作成を試みます。
    *   `--copy` パラメータを付けると、物理的なコピーを実行します。
3.  **一括インデックスとチェック：** Daemon がインデックスを更新し、一連のプラグインに対して依存関係の整合性チェックを実行します。

## 2.1 CLI によるインストールフロー (The CLI Way)

これは推奨されるインストール方法であり、システムが依存関係とパスを自動的に処理します。

### A. ローカルフォルダのインストール
`C:\Downloads\my-plugins` にいくつかのプラグインをダウンロードしたと仮定します：

```bash
# 構文： openstarry plugin add <path>
$ openstarry plugin add C:\Downloads\my-plugins\super-search-tool
```

**システムの実行ステップ：**
1.  **検証：** ターゲットフォルダに有効な `package.json` および `plugin.json` (または `openstarry` フィールド) があるかチェックします。
2.  **コピー：** フォルダをシステムプラグインディレクトリ `~/.openstarry/plugins/super-search-tool` へコピーします。
3.  **依存関係：** そのディレクトリに入り、 `npm install --production` を実行して実行に必要なライブラリをインストールします。
4.  **ビルド：** `tsconfig.json` が検知され、 `dist/` が存在しない場合、 `npm run build` の実行を試みます。
5.  **登録：** Daemon にプラグインリストの更新を通知します。

### B. Git から直接インストール

```bash
$ openstarry plugin add https://github.com/username/weather-plugin.git
```

**システムの実行ステップ：**
1.  **Clone：** リポジトリをテンポラリディレクトリに clone します。
2.  **処理：** 前述の依存関係のインストールとビルドステップを実行します。
3.  **移動：** 成果物をシステムディレクトリへ移動します。

---

## 3. インストールフロー (The Manual Way)

個別の NPM パッケージや Git URL に対しては、従来のインストール方法も残されています。

```bash
$ openstarry plugin add https://github.com/user/weather-tool.git
```
*   **即時チェック：** インストール完了後、 CLI は同様に不足している依存関係をチェックして報告します。

完全に手動で制御したい場合（例：一括コピーなど）は、以下の規範に従ってください：

### ターゲットパス
プラグインフォルダを `~/.openstarry/plugins/<plugin-id>/` に配置します。

### 必須の操作
ほとんどのプラグインは Node.js モジュールに依存しているため、単にファイルをコピーするだけでは不十分です。

1.  **ファイルのコピー：** 
    プラグインのソースコードをターゲットパスにコピーします。
    
2.  **依存関係のインストール (重要なステップ)：** 
    そのディレクトリに入り、依存関係をインストールしなければなりません。そうしないと、エージェントの実行時にエラー ( `Module not found` ) が発生します。
    ```bash
    cd ~/.openstarry/plugins/<plugin-id>
    npm install --production
    ```

3.  **コンパイル ( TypeScript の場合)：** 
    プラグインが TS で書かれており、 `dist` が付属していない場合は、コンパイルが必要です。
    ```bash
    npm run build
    ```

4.  **Daemon の再起動：** 
    ```bash
    openstarry daemon restart
    ```

---

## 4. プラグインの権限と隔離

同期 (Sync) であれ追加 (Add) であれ、新しいプラグインが初めてエージェントによって **有効化 (Enabled)** された際、システムは `agent.json` の権限設定に基づいて傍受チェックを行い、未認可の高リスク操作が行われないことを保証します。

サードパーティプラグインをインストールする際、システムはその `manifest` で宣言された権限をスキャンします。

*   **サンドボックスメカニズム：** デフォルトでは、サードパーティプラグインは制限された環境で動作します。
*   **権限の提示：** `openstarry plugin add` を実行すると、 CLI はそのプラグインが要求する権限（例： `network`, `fs:write` ）をリストアップし、ユーザーに確認を求めます。
*   **手動審査：** 手動でインストールされたプラグインについては、初回実行時に Daemon が動作を遮断し、管理者の認可を要求します。
