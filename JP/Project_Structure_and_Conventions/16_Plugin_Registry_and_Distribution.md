# 16. プラグインレジストリと分かち合いメカニズム (Plugin Registry and Distribution)

OpenStarry は、単一の中央サーバーに依存するのではなく、 **「非中央集権的な Git リポジトリ」** を核心的な分かち合い（配信）モデルとして採用しています。

## 1. プラグイン配信のサポート

### 1.1 公式エコシステム (The Ecosystem Repo)
*   **リポジトリ：** `https://github.com/SecludedCorner/openstarry_plugin`
*   **役割：** これは公式に維持されている「プラグイン・マーケットプレイス」です。
*   **メカニズム：** すべての標準プラグイン (Standard) と、審査済みのコミュニティプラグインがこの Monorepo に保管されています。
*   **取得方法：** ユーザーはこのリポジトリを Clone し、 `openstarry plugin sync` を通じて最新の状態に保ちます。

### 1.2 プライベート/サードパーティ配信
*   **メカニズム：** 開発者は独自に Git リポジトリ（構造は `openstarry_plugin` 仕様に準拠する必要あり）を構築できます。
*   **取得方法：** ユーザーはそのリポジトリを Clone した後、同様に `openstarry plugin add --all` を使用して一括インストールを行います。

### 1.3 NPM ベースの配信 (推奨)
*   **メカニズム：** プラグイン自体が標準的な NPM パッケージです。
*   **命名規則：** `@scope/openstarry-plugin-<name>` または `openstarry-plugin-<name>` 。
*   **利点：** 既存の NPM インフラストラクチャ（ CDN 、バージョン管理、依存関係の解決）を活用できます。
*   **インストール：** `openstarry plugin add openstarry-plugin-weather` 。

### 1.4 Git ベースの配信 (非中央集権)
*   **メカニズム：** Git リポジトリの URL を直接指定します。
*   **シナリオ：** 企業内部のプライベートプラグイン、開発中のプラグイン。
*   **インストール：** `openstarry plugin add git+https://github.com/user/my-plugin.git` 。

## 2. プラグインレジストリ (The Registry)

配信は非中央集権的ですが、発見を容易にするために、 **メタデータレジストリ (Metadata Registry)** を維持しています。

### 構造
Daemon はメモリ内データベース ( `~/.openstarry/registry.db` ) を維持し、以下を記録します：
*   **Plugin ID:** `standard-function-stdio`
*   **Source:** `/Users/me/openstarry_plugin/plugins/standard-function-stdio`
*   **Capabilities:** `[stdio, cli-ui]`
*   **Installed At:** `2026-02-02 10:00:00`
```json
{
  "plugins": {
    "openstarry-plugin-fs": {
      "type": "tool",
      "description": "Safe file system operations",
      "versions": {
        "1.0.0": {
          "url": "npm:openstarry-plugin-fs@1.0.0",
          "checksum": "sha256:..."
        }
      }
    }
  }
}
```
### 照会フロー
エージェントが起動し `standard/interaction` を要求すると、コアは Daemon を介してローカルレジストリを照会し、 Daemon はそのプラグインのディスク上の物理パスを返します。

### 2.2 申請フロー
1.  開発者がプラグインを NPM に公開します。
2.  開発者が `openstarry/registry` に対して Pull Request を送信し、プラグイン情報を追加します。
3.  CI が、プラグインが `manifest.json` 仕様に準拠しているか自動検証します。
4.  マージされると、 `openstarry` CLI ツールでそのプラグインを検索できるようになります。

## 3. インストールと依存関係管理

### 3.1 エージェント・マニフェスト (Agent Manifest)
各エージェントインスタンスのディレクトリにある `agent.json` で、その依存関係を定義します：

```json
{
  "name": "coding-assistant",
  "plugins": {
    "openstarry-plugin-fs": "^1.2.0",
    "openstarry-plugin-gemini": "latest"
  }
}
```

### 3.2 インストールプロセス ( `openstarry plugin add` )
ユーザーがエージェントディレクトリで `openstarry plugin add` を実行すると：
1.  `agent.json` を **読み込み** ます。
2.  依存関係のソース（ NPM または Git ）を **解析** します。
3.  プラグインパッケージを `~/.openstarry/plugins/` （グローバルキャッシュ）または `./node_modules` に **ダウンロード** します。
4.  プラグインの署名を **検証** します（セキュリティの章を参照）。
5.  環境の再現性を確保するために、ロックファイル `openstarry-lock.json` を **生成** します。

## 4. プラグインのバージョン管理

*   **公式プラグイン：** `openstarry_plugin` リポジトリの Git Tag バージョンに従います。
*   **ロックメカニズム：** プロジェクトの `openstarry-lock.json` に、現在使用しているプラグインのバージョン ( Git Commit Hash ) が記録され、異なるマシン間での実行の一致を保証します。

**セマンティック・バージョニング (SemVer)** に準拠します：
*   **Major：** 破壊的変更（ツールのパラメータ構造の変更などにより、古いプロンプトが正しく呼び出せなくなる場合）。
*   **Minor：** 機能の追加（新しいツールの追加など）。
*   **Patch：** バグ修正。

エージェントコアは、ロード時にプラグインの `engines` フィールドをチェックし、現在動作しているコアのバージョンと互換性があることを確認します。
