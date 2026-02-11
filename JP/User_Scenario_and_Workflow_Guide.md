# OpenStarry ユーザーシナリオとワークフローガイド (User Scenario and Workflow Guide)

本ドキュメントでは、OpenStarry の主要な操作シナリオを深く体験していただきます。システムの起動からエージェントの設計、実際の対話インタラクション、そしてエージェントをシステムの自動化管理フローに組み込む方法までをカバーします。

---

## 1. システムエントリ：起動と監視 (`openstarry`)

ターミナルで `openstarry` と入力すると、このマルチエージェントオペレーティングシステムの**「統合コントロールパネル (Runtime Dashboard)」**が起動します。

### 操作シナリオ
```bash
$ openstarry
```

### システムの動作
1.  **デーモン (Daemon) への自動接続/起動:** バックグラウンドで `Orchestrator Daemon` が実行中かどうかをチェックします。実行されていない場合は自動的に起動します。
2.  **ダッシュボード表示:** リアルタイム更新される TUI (Text User Interface) ダッシュボードが表示されます。

### ビジュアルイメージ (Dashboard)
```text
  ___                   _____ _
 / _ \ _ __   ___ _ __ /  ___| |_ __ _ _ __ _ __ _   _
| | | | '_ \ / _ \ '_ \\ `--.| __/ _` | '__| '__| | | |
| |_| | |_) |  __/ | | |`--. \ || (_| | |  | |  | |_| |
 \___/| .__/ \___|_| |_/\____/\__\__,_|_|  |_|   \__, |
      | |                                         __/' |
      |_|                                        |___/

[SYSTEM DASHBOARD] - Connected to Daemon (PID: 14023)
---------------------------------------------------------------------
● System Status:  HEALTHY   |   ● Uptime: 2d 4h 12m
● CPU Usage:      12%       |   ● Memory: 402MB / 8GB
---------------------------------------------------------------------

[RUNNING AGENTS]
ID       NAME            TYPE          STATUS    THOUGHTS/SEC   LAST ACTIVITY
sys-01   Master-Mind     Orchestrator  RUNNING   0.5            Just now
dev-01   Code-Helper     Worker        IDLE      0.0            5m ago
web-01   Search-Bot      Worker        BUSY      1.2            Processing...

[CONTROLS]
(q)uit view  (r)estart daemon  (k)ill agent  (d)esign mode
(a)ttach to agent (select via arrows)
```

---

## 2. デザインモード：創造と編集 (`openstarry design`)

新しい生命を創造したり、既存のエージェントを修正する必要がある場合は、デザインモードを使用します。このコマンドは**コンテキスト認識**機能を備えています。

### シナリオ A：グローバルデザインモード (Global Design Mode)
*   **トリガー条件：** プロジェクトディレクトリ以外で `openstarry design` を実行。
*   **機能：** これは「創造主の視点」です。
    *   **Create New Agent Project:** 標準テンプレート（Coding, Writing, Analysis）から新しいエージェントプロジェクトを生成します。
    *   **Manage Templates:** エージェントのテンプレートライブラリをダウンロード・管理します。
    *   **System Configuration:** グローバル API Key の設定やデーモンのリソース制限を調整します。

### シナリオ B：プロジェクトデザインモード (Project Design Mode)
*   **トリガー条件：** エージェントのプロジェクトディレクトリ内で `openstarry design` を実行。
*   **機能：** これは「手術台の視点」です。システムがエージェントの**五蘊（ごうん）**の定義をガイドします：
    1.  **識 (Guide):** 魂を注入します。`system_prompt` を修正し、**ペルソナ**、**核心目標**、**記憶戦略**を設定します。
    2.  **色 (UI/Body):** 肉体を形成します。エージェントの**名称**、**外観**、**インタラクションインターフェース (UI)** を設定します。
    3.  **受 (Listeners):** 感覚器官を構成します。メッセージを受信するチャンネル（`stdio`、`webhook` など）を設定します。
    4.  **行 (Tools):** 四肢を装備します。ツールプラグイン（`fs`、`gemini-search` など）をインストールします。
    5.  **想 (Providers):** 脳を選択します。特定の LLM モデル（Gemini 1.5 Pro など）に接続します。

---

## 3. 対話インタラクション：エージェントへの接続 (`openstarry attach`)

エージェントとの対話を開始するには、`attach` コマンドを使用します。これがインタラクションの**唯一のエントリポイント**です。

### シナリオ A：ダッシュボードからの接続
1.  `openstarry` ダッシュボードで、方向キーを使って対象エージェント（例：`sys-01`）を選択します。
2.  キーボードの **`a`** キーを押します。
3.  画面がそのエージェントの対話ウィンドウに即座に切り替わります。
4.  `Ctrl+D`（Detach）を押すとダッシュボードに戻り、エージェントはバックグラウンドで実行を継続します。

### シナリオ B：プロジェクトディレクトリからの接続（最も一般的）
開発ディレクトリで直接入力します：
```bash
$ openstarry attach
```
*   **エージェントが未実行の場合：** システムが自動的にエージェントを起動し、対話インターフェースに直接入ります。
*   **エージェントがバックグラウンドで実行中の場合：** そのまま接続します。

### シナリオ C：任意のバックグラウンドエージェントへの接続
```bash
$ openstarry attach web-01
```

### インタラクションテクニック：スラッシュコマンド
対話中に `/` を使って、LLM を介さずにツールを直接呼び出すことができます。
*   `/login` -> `google-login` ツールを呼び出し（エイリアスが登録されている場合）。
*   `/tool google-login` -> ツールを直接実行。
*   `/clear` -> 現在のコンテキストメモリをクリア。

---

## 4. デプロイと自動起動フロー (Deployment Workflow)

システム起動時にエージェントを自動的にオンラインにするにはどうすればよいでしょうか？

### ステップ 1：標準プラグインの同期 (Sync)
システムが最新の標準能力ライブラリを持っていることを確認します。
```bash
# openstarry_plugin リポジトリをローカルに clone 済みと仮定
$ openstarry plugin sync ./openstarry_plugin
```

### ステップ 2：登録 (Register)
まず、デーモン（Daemon）にエージェントの場所を伝えます。プロジェクトディレクトリで実行します：
```bash
$ openstarry register
```
これにより、プロジェクトパスが `~/.openstarry/registry.json` に書き込まれ、「管理対象エージェント」となります。

### ステップ 3：自動起動リストへの追加 (Autostart)
現在、2つの方法があります：

**方法 A (CLI):**
ダッシュボードの「システム設定」メニュー、または将来予定されているコマンドを使用します：
```bash
$ openstarry system config --autostart add <project-id>
```

**方法 B (手動設定):**
`~/.openstarry/daemon.json` を編集します：
```json
{
  "autostart": [
    "master-agent",
    "my-coding-bot",
    "daily-news-reporter"
  ]
}
```

### ステップ 4：検証 (Verification)
デーモンを再起動（`openstarry daemon restart`）するか、`openstarry` を再入力してダッシュボードに入ります。エージェントが **[RUNNING AGENTS]** リストに自動的にオンラインになっているのが確認できるはずです。

---

## 5. 高度な操作：ワークフローとマルチエージェント連携 (Advanced Orchestration)

OpenStarry の強力さは、一連のエージェントを連携して実行できる点にあります。

### ステップ 1：ワークフローの定義 (Define Workflow)
プロジェクトまたはシステムディレクトリに `workflow.yaml` を作成します：
```yaml
name: Daily Research
steps:
  - id: research
    agent: research-agent
    instruction: "今日のAIニュースを収集"
  - id: summary
    agent: writer-agent
    input_from: research
    instruction: "3つのポイントに要約して日本語に翻訳"
```

### ステップ 2：ワークフローの実行 (Execute)
CLI でトリガーします：
```bash
$ openstarry run ./workflow.yaml
```

### ステップ 3：連携の観察 (Observe)
ダッシュボード（`openstarry`）に戻ると、以下が確認できます：
1.  **動的生成：** `research-agent` がリストに突然現れ、`BUSY` と表示されます。
2.  **タスク引き継ぎ：** 数秒後、`research-agent` が消え（タスク完了で破棄）、`writer-agent` が現れて執筆を開始します。
3.  **結果の配信：** 最終結果がターミナルまたは指定されたファイルに出力されます。

---

## 6. トラブルシューティングとログの解読 (Troubleshooting & Logs)

エージェントの動作が期待通りでない場合、どのように診断すればよいでしょうか？

### リアルタイムの思考を確認 (Live Thoughts)
`attach` でエージェントに入ると、その「内なる独白」が確認できます：
```text
[THOUGHT] User asked for file search.
[PLAN] I need to use 'fs' tool to list directory.
[CALL] fs.list_dir(path=".")
[RESULT] Success. Found 3 files.
[RESPONSE] I found 3 files...
```
もし停滞している場合、通常は `[CALL]` の失敗か `[THOUGHT]` のループが原因です。

### システムログの確認 (System Logs)
エージェントがそもそも起動しない場合は、デーモンのログを確認してください：
```bash
$ openstarry system logs
```
よくあるエラー：
*   **Template Not Found:** レジストリのパスが正しくありません。`register` をやり直してください。
*   **Permission Denied:** プラグインの権限が不足しています。`manifest.json` を確認するか、`openstarry plugin grant` を使用してください。
