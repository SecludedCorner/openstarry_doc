# 実装の考え方： openclaw - 調整レイヤーのルーティングとタスクグラフ

このドキュメントでは、 `openclaw` のマルチエージェント・ルーティングおよび複雑なタスク実行能力を実現するための「エージェント調整レイヤー」の実装の考え方を詳細に説明します。

## 核心的な設計：中央ルーターとしての調整レイヤー

システムが複数の `UI` プラグイン（ WhatsApp, Slack など）を介してメッセージを受信する場合、「どのメッセージをどのエージェントが処理すべきか」を決定するための中央ハブが必要になります。これはまさに「エージェント調整レイヤー」の核心的な責務の1つです。

### 1. ルーティングコンポーネント (Router)

調整レイヤーの内部には **ルーティングコンポーネント** が必要です。

*   **責務：** 入ってくるすべてのリクエストを検査し、あらかじめ設定されたルールセットに基づいて、そのリクエストをどの下流の「エージェントコア」インスタンスまたは「ワークフロー」に振り分けるかを決定します。
*   **リクエストオブジェクト：** ルーターが処理するリクエストオブジェクトには、 UI プラグインからの豊富なメタデータが含まれている必要があります。例：
    ```json
    {
      "channel": "whatsapp",
      "user_id": "whatsapp:+15550100000",
      "user_info": { "name": "John Doe", "is_admin": false },
      "conversation_id": "conv_123",
      "content": "会議室の予約を手伝って"
    }
    ```

### 2. ルーティングルール (Routing Rules)

ルーティングのロジックは、設定可能な「ルーティングテーブル」またはルールセットを通じて定義できます。

*   **ルールテーブルの例（ YAML, JSON またはデータベースのテーブルなど）：**
    ```yaml
    - name: Route sales inquiries to Sales Agent
      if:
        channel: 'whatsapp'
        content_contains: ['価格', '購入', '割引']
      then:
        action: 'run_agent'
        agent_id: 'sales_agent_instance_01'

    - name: Route IT support requests to IT Workflow
      if:
        channel: 'slack'
        user_group: 'employees'
        content_contains: ['パスワードリセット', 'VPNに接続できない']
      then:
        action: 'run_workflow'
        workflow_id: 'it_support_password_reset_flow'

    - name: Default route for general chat
      if: true
      then:
        action: 'run_agent'
        agent_id: 'general_purpose_assistant'
    ```

### 3. ワークフローの実行（ `openclaw` のタスクグラフ）

`openclaw` の「宣言的タスクグラフ」は、私たちの「エージェント調整レイヤー」の **ワークフローエンジン** 機能に完全に対応します。

*   ルーティングルールが `workflow_id` を指している場合、調整レイヤーの **実行エンジン** が起動します。
*   対応するワークフロー定義（例： `it_support_password_reset_flow.yaml` ファイル）をロードします。
*   その後、 `02_Agent_Coordination_Layer.md` で説明したように、グラフの依存関係に従って、各ノード（ API の呼び出し、スクリプトの実行、あるいは特定の専門エージェントコアの再呼び出しなど）を順番に実行します。

### 結論

**柔軟なルーティングルール** と **強力なワークフローエンジン** を組み合わせることで、「エージェント調整レイヤー」は `openclaw` が記述する複雑なタスク処理とマルチエージェント協調能力を完璧に実現できます。 UI プラグインが「アクセス」を担当し、調整レイヤーが「振り分けと処理」を担当し、エージェントコアが「具体的な実行と対話」を担当するというように、3者がそれぞれの役割を果たすことで、1つの完全なシステムが構成されます。
