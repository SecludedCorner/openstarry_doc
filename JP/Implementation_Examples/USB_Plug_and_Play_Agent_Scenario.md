# 実装例： USB プラグアンドプレイ・エージェント (USB Plug-and-Play Agent)

この例では、 OpenStarry アーキテクチャの究極の柔軟性を示します。それは、完全なエージェントを USB メモリ内にカプセル化することです。 USB がホストマシンに挿入されると、システムは自動的にそれを感知してエージェントを呼び起こし、特定のタスク（自動バックアップ、システム診断など）を実行させ、タスク終了後には安全な取り外しを許可します。

---

## 1. シナリオの説明

*   **目標：** 「写真バックアップ・スティック」を作成する。
*   **動作：**
    1.  USB を挿入する。
    2.  システムが「写真バックアップ助手を検出しました」と提示する。
    3.  ユーザーが承認すると、そのエージェントは自動的にホストマシンのピクチャフォルダをスキャンし、新しい写真を USB 内にバックアップする。
    4.  完了後、ユーザーに通知し、自身を終了させる。

---

## 2. USB 内部のディレクトリ構造

これは標準的な OpenStarry **プロジェクト級エージェント** のディレクトリであり、物理的な媒体が USB ドライブ（例： `E:/` ）になっただけです。

```text
E:/ (USB Root)
├── configs/
│   └── agent.json       # アイデンティティと権限の定義
├── plugins/
│   └── backup-utils/    # 専用プラグイン：増分バックアップアルゴリズムを担当
├── logs/                # 実行ログ
└── backup_data/         # バックアップファイルの保存先
```

### `configs/agent.json`

```json
{
  "identity": {
    "id": "usb-photo-backup-01",
    "name": "USB Photo Backup",
    "role": "backup_worker"
  },
  "capabilities": {
    "plugins": [
      "std-fs-tools",       // システムが提供するファイルツールを継承
      "backup-utils"        // USB 自体が持つバックアップアルゴリズムを使用
    ],
    "permissions": {
      "fs_allow_paths": ["C:/Users/Public/Pictures", "E:/backup_data"], // 読み書き範囲を厳格に制限
      "network_allow_hosts": [] // プライバシー確保のため、ネットワーク接続を禁止
    }
  }
}
```

---

## 3. システム側の実装 (Host Side)

ホストマシン上の **システム・エージェント (System Agent)** は、「ハードウェアの変更を感知する」能力を備えている必要があります。

### ステップ A: USB 監視プラグインのインストール
システム・エージェントは `std-hardware-listener` プラグイン（受蘊に属する）をロードします。

```javascript
// plugins/std-hardware-listener/index.js (イメージコード)
const usb = require('usb-detection');

class HardwareListener {
  start() {
    usb.on('add', (device) => {
      // 1. デバイスの挿入を検知
      const mountPoint = this.getMountPoint(device);
      // 2. agent.json が存在するかチェック
      if (fs.existsSync(path.join(mountPoint, 'configs/agent.json'))) {
        // 3. システム・エージェントに通知
        this.core.submitSensoryInput({
          sense: 'environmental',
          type: 'USB_AGENT_DETECTED',
          payload: { path: mountPoint, deviceId: device.serialNumber }
        });
      }
    });
  }
}
```

### ステップ B: システム・エージェントの応答ロジック
システム・エージェントの LLM が `USB_AGENT_DETECTED` イベントを受信すると、 System Prompt に基づいて以下の意思決定を実行します。

1.  **検証：** `agent:validate_manifest` を呼び出して、 USB エージェントの権限リクエストをチェックします（過大な権限を要求していないか？）。
2.  **問い合わせ：** UI を通じてユーザーに「 USB バックアップ助手を検出しました。起動を許可しますか？ピクチャフォルダへのアクセスを要求しています」と尋ねます。
3.  **生成 (Spawn)：** ユーザーが同意すると、 `agent:spawn` を呼び出します：
    ```javascript
    agent_manager.spawn({
      manifest: "E:/configs/agent.json",
      work_dir: "E:/"
    });
    ```

---

## 4. 実行時の協調 (Runtime Coordination)

1.  **起動：** Orchestrator Daemon が新しい Node.js プロセスを起動し、 `E:/` 下のコードをロードします。
2.  **実行：** USB エージェント（プロジェクト・エージェント）が動作を開始します。システムから提供された `std-fs-tools` を使用してホストのハードディスクを読み取り、自身が持つ `backup-utils` を使用して USB に書き込みます。
3.  **通信：** USB エージェントは MCP プロトコルを通じてシステム・エージェントに進捗を報告します。「50枚の写真をバックアップしました...」。
4.  **終了：** タスクが完了すると、 USB エージェントは `TASK_COMPLETE` 信号を送信します。
5.  **破棄：** システム・エージェントは信号を受信すると、 `agent:kill` を呼び出してプロセスを終了させ、ユーザーに「安全に取り外せます」と通知します。

---

## 5. アーキテクチャのまとめ

このケースは、 OpenStarry の **階層的な継承** と **権限の隔離** メカニズムを完璧に示しています：
*   **ポータビリティ：** エージェントの魂 (Prompt) と肉体 (Plugins) がすべて USB の中にあり、プラグアンドプレイが可能です。
*   **セキュリティ：** ホストのシステム・エージェントが門番として機能し、 USB エージェントの権限を厳格に審査することで、悪意のあるソフトウェアが機密データを読み取るのを防ぎます。
