# 實現思路：openclaw - 協調層路由與任務圖

本文件詳細描述了為實現 `openclaw` 的多代理人路由和複雜任務執行的能力，「代理協調層」的實現思路。

## 核心設計：作為中央路由器的協調層

當系統通過多個 `UI` 插件（如 WhatsApp, Slack）接收消息時，需要一個中央樞紐來決定「哪條消息應該由哪個代理人來處理」。這正是「代理協調層」的核心職責之一。

### 1. 路由組件 (Router)

協調層內部需要一個**路由組件**。

*   **職責：** 檢查每一個傳入的請求，並根據一套預設的規則，決定將該請求分發到哪個下游的「代理人核心」實例或哪個「工作流」。
*   **請求對象：** 路由組件處理的請求對象，應包含來自 UI 插件的豐富元數據，例如：
    ```json
    {
      "channel": "whatsapp",
      "user_id": "whatsapp:+15550100000",
      "user_info": { "name": "John Doe", "is_admin": false },
      "conversation_id": "conv_123",
      "content": "幫我預定一個會議室"
    }
    ```

### 2. 路由規則 (Routing Rules)

路由的邏輯可以通過一個可配置的「路由表」或規則集來定義。

*   **示例規則表 (可以是 YAML, JSON 或數據庫表):**
    ```yaml
    - name: Route sales inquiries to Sales Agent
      if:
        channel: 'whatsapp'
        content_contains: ['價格', '購買', '折扣']
      then:
        action: 'run_agent'
        agent_id: 'sales_agent_instance_01'

    - name: Route IT support requests to IT Workflow
      if:
        channel: 'slack'
        user_group: 'employees'
        content_contains: ['密碼重置', 'VPN無法連接']
      then:
        action: 'run_workflow'
        workflow_id: 'it_support_password_reset_flow'

    - name: Default route for general chat
      if: true
      then:
        action: 'run_agent'
        agent_id: 'general_purpose_assistant'
    ```

### 3. 工作流執行 (`openclaw` 的 Task Graph)

`openclaw` 的「聲明式任務圖」完全對應我們「代理協調層」的**工作流引擎**功能。

*   當路由規則指向一個 `workflow_id` 時，協調層的**執行引擎**就會啟動。
*   它會加載對應的工作流定義（例如，一個 `it_support_password_reset_flow.yaml` 文件）。
*   然後，它會像我們在 `02_Agent_Coordination_Layer.md` 中描述的那樣，按照圖的依賴關係，依次執行各個節點（調用 API、運行腳本、甚至再次調用某個專門的代理人核心）。

### 結論

通過結合**靈活的路由規則**和**強大的工作流引擎**，「代理協調層」能夠完美地實現 `openclaw` 所描述的複雜任務處理和多代理人協同能力。UI 插件負責「接入」，協調層負責「分發和處理」，代理人核心負責「具體執行和對話」，三者各司其職，構成了一個完整的系統。
