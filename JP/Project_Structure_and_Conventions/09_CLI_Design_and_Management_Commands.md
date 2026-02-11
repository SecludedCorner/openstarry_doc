# 09. CLI 設計と管理コマンド (CLI Design & Management Commands)

OpenStarry CLI ( `openstarry` ) は、ユーザーがエージェントシステムと対話するための統合された入り口です。コンテキスト認識能力を備えており、実行場所に基づいて「プロジェクトモード」と「システムモード」を自動的に切り替えます。

## 1. 核心的な設計原則

### コンテキスト認識 (Context Awareness)
CLI の「設計モード」 ( `openstarry design` ) は強力なコンテキスト認識能力を備えており、その挙動は **「どこで実行するか」** に依存します。
*   **グローバル設計モード (Global Design Mode)：** プロジェクトディレクトリ以外で実行した場合、システムレベルの管理機能（新規プロジェクトの作成、グローバルテンプレートの管理など）を提供します。
*   **プロジェクト設計モード (Project Design Mode)：** `agent.json` を含むディレクトリで実行した場合、そのエージェントの編集および設定インターフェースに自動的に入ります。

### 実行時と設計時の分離 (Runtime vs Design)
*   **`openstarry` (Runtime)：** **実行と監視** に特化しています。どこで実行しても守護プロセス (Daemon) に接続し、リアルタイムのシステムダッシュボード (Dashboard) を提供します。
*   **`openstarry design` (Design)：** **定義と設定** に特化しています。五蘊を構築し、エージェントを組み立てるためのワーク台となります。

### フォアグラウンドとバックグラウンド (Foreground vs Background)
*   **プロジェクトモード (Foreground)：** エージェントは現在のシェルの子プロセスとして動作します。ターミナルを閉じるとエージェントは停止します。
*   **守護モード (Background)：** Daemon を介して起動され、ターミナルを閉じてもエージェントはバックグラウンドで動作し続けます。

---

## 2. コマンドセットの詳細

### A. 核心的なエントリポイント (Core Entry Points)

```bash
# OpenStarry ランタイム環境を起動
$ openstarry
# -> Orchestrator Daemon が動作しているかチェック。
# -> 動作していない場合は、 Daemon とマスターエージェントを自動起動 (System Mode)。
# -> 動作している場合は、対話型コンソール (Console) に入り、現在動作中のエージェントの状態とシステム指標を表示。
# -> [Console Feature] コンソールで特定のエージェントを選択し、 'a' キーを押すと直接接続 (Attach) して対話が可能。

# -> [Context Aware] 現在のディレクトリに基づいて自動切り替え：
#    - グローバルモード：「新規プロジェクトの作成」「テンプレートのダウンロード」「グローバル設定の管理」メニューを表示。
#    - プロジェクトモード：現在のエージェントの構造を表示し、ユーザーによる **五蘊** (色受想行識) の定義をガイド：
#      * 人格設定と記憶の設定 (識)
#      * インターフェースと名称の設定 (色)
#      * ツールプラグインの追加 (行)
#      * 感覚リスナーの選択 (受)
#      * AI モデルの指定 (想)
# -> 対話型 TUI (Text User Interface) を起動。
# -> agent.json とディレクトリ構造を自動生成または更新。
```

### B. プロジェクトのライフサイクル (Project Lifecycle)

これらのコマンドは、現在のディレクトリにある「プロジェクト級エージェント」を管理するために使用されます。

```bash
# 新しいエージェントプロジェクトを初期化
$ openstarry init [template-name]
# -> agent.json, package.json, .openstarry/ 構造を作成

# [対話モード] 起動して対話に入る
$ openstarry attach
# -> プロジェクトディレクトリで実行した場合：
#    - エージェントがまだ動作していない場合：エージェントを自動起動し、直ちに対話インターフェースに入る。
#    - エージェントがすでにバックグラウンドで動作している場合：そのエージェントに直接接続 (Attach) する。
# -> エージェントとの対話を開始するための、最も一般的なコマンドです。

# [起動モード] プロセスをバックグラウンドでのみ起動
$ openstarry start
# -> ./agent.json を読み取り、エージェントをバックグラウンド守護プロセスとして起動。
# -> ./openstarry/agent.pid を書き込み、ログをファイルに記録。
# -> 現在のターミナルを占有しません。

# ツールの直接実行 (Manual Tool Invocation)
$ openstarry run-tool <tool-name> [args...]
# -> LLM を介さず、指定された ITool を直接呼び出す。
# -> 主な用途：ログイン (google-login) 、初期設定、またはツールの機能テスト。
# -> 例： openstarry run-tool google-login

# プロジェクトのランタイムデータをクリーンアップ
$ openstarry clean
# -> ./.openstarry/ 内の一時ファイル（ログ、状態キャッシュ）を削除
```

### B. システム守護管理 (Daemon Management)

これらのコマンドは、 `~/.openstarry` 内の守護プロセスと対話するために使用されます。

```bash
# システム守護プロセスを起動 (まだ動作していない場合)
$ openstarry daemon start

# システム状態の確認 (ホストされているすべてのエージェントをリストアップ)
$ openstarry ps
# OUTPUT:
# ID      NAME        STATUS    PID    MODE      LOCATION
# sys-01  Scheduler   RUNNING   1024   System    ~/.openstarry/agents/sys-01
# prj-01  MyBot       RUNNING   4521   Project   D:/Code/MyBot (Registered)

# 守護プロセス (およびすべての小エージェント) を停止
$ openstarry daemon stop
```

### C. 登録とホスティング (Registration & Hosting)

「プロジェクト級エージェント」を「システム級サービス」にアップグレードします。

```bash
# 現在のディレクトリのエージェントを Daemon に登録
$ openstarry register
# -> Daemon はパス "D:/Code/MyBot" を ~/.openstarry/registry.json に記録。
# -> 以降、 Daemon がそのエージェントの再起動と監視を担当。

# グローバルまたは特定のエージェントに接続 (Attach)
$ openstarry attach [agent-id]
# -> agent-id が指定されている場合：指定されたバックグラウンドエージェントに接続。
# -> プロジェクトディレクトリ内で agent-id 指定なしの場合：現在のプロジェクトエージェントに接続。
# -> Ctrl+C で接続を解除 (Detach) しますが、エージェントはバックグラウンドで動作し続けます。

# プラグインレジストリの更新 (Manual Refresh)
$ openstarry plugin refresh
# -> Daemon にシステムプラグインディレクトリの再スキャンを強制。
# -> 手動でコピーした新しいプラグインを検出するために使用。
```

## 3. データフローのまとめ

| モード | コマンド例 | 設定の読み込み場所 | 実行状態 (PID/State) | ログの場所 | ライフサイクル |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **プロジェクト (フォアグラウンド)** | `openstarry start` | `./agent.json` | `./.openstarry/` | `./.openstarry/logs/` | シェルの終了まで |
| **プロジェクト (ホスト)** | `openstarry start -d` | `./agent.json` | `~/.openstarry/state/` | `~/.openstarry/logs/` | Daemon が管理 |
| **システム (内蔵)** | `openstarry system start` | `~/.openstarry/agents/` | `~/.openstarry/state/` | `~/.openstarry/logs/` | 常駐サービス |

この設計により、開発時の柔軟性（プロジェクトモード）とデプロイ時の安定性（ホストモード）が完璧に共存します。
