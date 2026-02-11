# 18. エージェントランタイム構成ロジック (Agent Runtime Configuration)

本ドキュメントは `05_Agent_Manifest_Specification.md` を補足するもので、**ランタイム (Runtime)** が `agent.json` の構成をどのように解析、検証、適用するかに焦点を当てています。特にプラグインの有効化とパラメータインジェクションについて説明します。

## 1. 構成のロードフロー

`openstarry start` または `Daemon` が Agent を起動する際：

1.  **読み取り：** `configs/agent.json` から生の JSON を読み取ります。
2.  **マージ (Merger):**
    *   `agent.json` が `extends: "generic-assistant-v1"` を定義している場合、システムはまずそのテンプレートの構成をロードし、次に現在のプロジェクトの構成を上書きします（Deep Merge）。
    *   これにより、プロジェクトは差分のみを定義できます（例：Prompt の変更や Tool の追加のみ）。
3.  **環境変数インジェクション：** `${ENV_VAR}` 構文を解析し、API Key などの機密情報を注入します。

## 2. プラグインの有効化とパラメータインジェクション

これは構成の中で最も重要な部分です：Agent が自身の利用可能なケイパビリティをどのように認識するかを定めます。

### 構成の構造

```json
"capabilities": {
  "plugins": {
    "@openstarry-plugin/fs": {
      "enabled": true,
      "config": { "root": "./workspace" } // initialize(context) に注入される
    },
    "@openstarry-plugin/weather": {
      "enabled": false // 明示的に無効化
    }
  }
}
```

### ランタイムロジック

1.  **フィルタリング：** Loader が `plugins` リストを走査し、`enabled: false` のエントリを除外します。
2.  **コンテキストの構築：**
    *   有効化された各プラグイン（例：`@openstarry-plugin/fs`）について、Loader がその `config` オブジェクト（例：`{ "root": "./workspace" }`）を抽出します。
    *   このオブジェクトは `IPluginContext.config` に配置されます。
3.  **初期化：** プラグインの `initialize(context)` メソッドが呼び出されます。
    *   プラグインは内部で `context.config.root` を通じてパラメータを取得し、それに基づいて `fs` ツールのパス制限を設定します。

## 3. 五蘊の構成マッピング

`agent.json` の `cognition` と `identity` セクションは、実質的に **Guide（識）** タイプのプラグインを直接構成します。

*   **System Prompt:** `agent.json` が `system_prompt` を定義している場合、Core はそれを一時的な `AnonymousGuidePlugin` としてラップし、最高の優先度を設定します。
*   **Provider:** `cognition.provider` の構成は、対応する Provider プラグイン（例：`gemini`）に直接渡されます。

## 4. エラーハンドリングとデフォルト動作

*   **ゼロプラグイン状態 (Zero Plugin State):**
    *   システム起動時にプラグインが検出されない場合（または `~/.openstarry/plugins` が空の場合）、Agent Core は**クラッシュしてはなりません**。
    *   システムは **「アイドルモード (Idle Mode)」** に入り、ターミナルまたはダッシュボードに目立つ通知を表示する必要があります：
        > "⚠️ No plugins found. Please run `openstarry plugin sync` to install standard capabilities."
*   **依存関係の欠如：** 構成で `plugin-x` が有効化されているがシステムが見つけられない場合、Agent はエラーを報告して起動を終了する必要があります（Fail Fast）。ただし、そのプラグインが `optional: true` とマークされている場合は除きます。
*   **無効な構成：** プラグインは `initialize` フェーズで受け取った `config` を検証し、無効な場合は `ConfigurationError` をスローする必要があります。
