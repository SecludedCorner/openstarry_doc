# 04. エージェント・マニフェスト仕様 (Agent Manifest Specification)

このドキュメントでは、 `agent.json` のデータ構造を定義します。これは各エージェントインスタンスの核心的な設定ファイルであり、 `[Agent_Root]/configs/agent.json` に配置されます。

## 核心的な哲学：アイデンティティと能力の静的な定義

`agent.json` はエージェントの起動時にコアによって読み込まれます。これは **不変 (Immutable)** であり（ランタイム時に変更不可）、エージェントの挙動の予測可能性と監査可能性を保証します。

---

## 構造定義

```json
{
  "$schema": "./schemas/agent-manifest.schema.json",
  "identity": {
    "id": "log-analyst-01",
    "name": "Log Analysis Bot",
    "role": "data_engineer",
    "description": "サーバーログの分析とレポート生成を担当"
  },
  
  "cognition": {
    "system_prompt": "configs/prompts/system.md", // 外部ファイルへのポインタ、または直接インライン文字列
    "provider": {
      "id": "gemini-pro-adapter",
      "config": {
        "model": "gemini-1.5-pro",
        "temperature": 0.2
      }
    },
    "context_strategy": {
      "type": "summarizer", // 戦略プラグイン ID を参照
      "config": {
        "threshold": 20
      }
    }
  },

  "capabilities": {
    "plugins": [
      // プラグインの完全なパッケージ名を参照。コアは継承ルール（プロジェクト優先 -> システム）に従って検索する
      "@openstarry-plugin/standard-function-fs",
      "@openstarry-plugin/standard-function-stdio",
      "project-log-parser"
    ],
    "permissions": {
      "fs_allow_paths": ["./logs", "./reports"], // サンドボックスのパス制限
      "network_allow_hosts": ["github.com"]
    }
  },

  "policy": {
    "max_steps": 50, // 無限ループ防止
    "circuit_breaker": {
      "error_threshold": 5
    }
  }
}
```

---

## フィールド詳細

### 1. `identity`
*   **id:** ログ追跡やデーモン管理に使用される、グローバルにユニークな識別子。
*   **role:** マルチエージェント協調時のアドレッシングに使用されます（例：「 `data_engineer` を探して手伝わせる」）。

### 2. `cognition` (想蘊設定)
*   **system_prompt:** エージェントの核心となる人格と職務を定義します。長文の作成に便利なように、外部の Markdown ファイルの参照をサポートしています。
*   **provider:** どの LLM を脳として使用するかを指定します。

## 3. プラグイン設定の詳細 (Plugin Configuration)

エージェントがロードするプラグインには複数の成分が含まれている場合があり、それらを `agent.json` 内で細かく設定できます。

```json
"plugins": {
  "@openstarry-plugin/standard-function-fs": {
    "tools": { "read": true, "write": false } // 行蘊を制限
  },
  "@openstarry-plugin/standard-function-skill": {
    "guides": { "autoload": true } // 識蘊を設定
  }
}
```

### コンポーネントタイプ (Component Types)

*   **tools (行):** 具体的に実行される関数。
*   **listeners (受):** イベントリスナー。
*   **providers (想):** AI モデルのバックエンド。
*   **ui (色):** インターフェースコンポーネント。
*   **guides (識):** スキル、プロセス、およびプロトコルロジック。

### 4. `policy` (超我設定)
*   **max_steps:** リソース枯渇を防ぐための、単一タスクの最大実行ステップ数。
*   **circuit_breaker:** 「強制冷却」や「助けを求める」タイミングを定義します。
