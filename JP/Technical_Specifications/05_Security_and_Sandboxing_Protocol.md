# 05. セキュリティとサンドボックスプロトコル技術仕様 (Security & Sandboxing Protocol)

本ドキュメントは、OpenStarry の多層防御体系を定義します。実行能力を持つ Agent システムとして、セキュリティは妥協できない基盤です。

## 1. ファイルシステムサンドボックス (FileSystem Sandboxing)

すべての IO 関連操作（読み取り、書き込み、一覧表示）は、`PathGuard` による厳格な監視を受ける必要があります。

### 1.1 ジェイルモード (Chroot-like Jail)
Agent はデフォルトでワーキングディレクトリ (`workspace_root`) のみにアクセスできます。親ディレクトリへのアクセスを試みる操作はすべてインターセプトされます。

*   **許可:** `./data/file.txt`, `src/index.ts`
*   **禁止:** `../secret.key`, `/etc/passwd`, `C:\Windows\System32`

### 1.2 パス正規化
パーミッションチェックの前に、すべてのパスを解決および正規化する必要があります。これにより、シンボリックリンク (Symlink) や相対パストラバーサル攻撃 (`/app/workspace/../../etc/passwd`) によるバイパスを防止します。

---

## 2. コマンド実行ホワイトリスト (Command Execution Whitelist)

`shell:exec` のような高リスクツールに対しては、ホワイトリスト戦略を実施する必要があります。

### セキュリティポリシー設定 (`agent.json`)

```json
"security": {
  "allow_shell": true,
  "blocked_commands": ["rm -rf", "mkfs", ":(){:|:&};:", "format"],
  "allowed_commands": ["ls", "cat", "grep", "npm test", "git status"],
  "confirm_dangerous_actions": true
}
```

`confirm_dangerous_actions` が `true` の場合、`allowed_commands` に含まれていないコマンドに対して、コアは実行を一時停止し `EVENT:REQUEST_CONFIRMATION` を発行して、人間の承認を待ってから実行を続行します。

---

## 3. リソースクォータとサーキットブレーカー (Resource Quotas & Circuit Breakers)

DoS 攻撃や無限ループを防止するため、コアは以下の制限を強制的に適用します：

| リソース次元 | 制限しきい値（デフォルト） | トリガー後の結果 |
| :--- | :--- | :--- |
| **Max Loop Ticks** | 50 回/タスク | タスクの強制一時停止 (Safety Halt) |
| **Token Budget** | 100k / セッション | LLM リクエストの拒否 |
| **Tool Timeout** | 30 秒 | ツールプロセスの強制終了 (SIGKILL) |
| **Output Size** | 1 MB | 出力の切り捨て |

---

## 4. プラグインパーミッションモデル (Plugin Permission Model)

プラグインはロード時に必要なパーミッションを宣言する必要があります（マニフェスト宣言）。

```json
// plugin.json
{
  "permissions": [
    "fs:read",
    "network:http",
    "core:state:write"
  ]
}
```

コアはプラグイン初期化時に、`agent.json` の認可リストと照合します。プラグインが未認可の機密パーミッションを要求した場合、ロードは拒否されます。
