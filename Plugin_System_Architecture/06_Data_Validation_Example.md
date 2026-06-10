# 插件範例：數據驗證 (Zod)

本文件闡述了像 `Zod` 這樣的數據驗證庫，如何在我們的架構中發揮作用。

## 定位：庫/依賴，而非插件

首先，`Zod` 本身**不是一個插件**，而是一個可以被多個組件引用的**第三方庫 (Library / Dependency)**。它的作用是為系統的各個部分提供運行時的數據類型檢查和驗證，從而極大地增強系統的健壯性。

---

## 應用場景

`Zod` 可以在我們架構的以下幾個關鍵位置被集成和使用：

### 1. 在 `Tool` 插件的定義中

這是 `Zod` 最直觀的應用場景。

*   **目標：** 確保 LLM 生成的工具調用參數，在傳遞給 `execute` 方法之前是類型安全且符合預期的。
*   **實現思路：**
    1.  在 `Tool` 插件的定義中，`args` 字段可以直接是一個 `Zod` schema 對象。
        ```javascript
        import { z } from 'zod';

        class MyTool {
          name = "create_user";
          description = "Creates a new user in the system.";
          
          args = z.object({
            username: z.string().min(3),
            email: z.string().email(),
            age: z.number().optional()
          });

          async execute(args) {
            // 此處的 'args' 已經是被 Zod 驗證和轉換過的對象
            // ...
          }
        }
        ```
    2.  在「代理人核心」的「執行循環」中，當它準備調用一個工具時，它會先從 `ToolRegistry` 獲取工具的 `args` schema。
    3.  然後，它使用 `schema.safeParse(llm_generated_args)` 來驗證 LLM 提供的參數。
    4.  如果驗證失敗，它會將 `Zod` 生成的詳細錯誤信息作為「工具執行失敗」的結果，反饋給 LLM，讓 LLM 自我糾正並生成正確的參數。
    5.  如果驗證成功，才調用工具的 `execute` 方法，並傳入經過 `Zod` 清洗和類型轉換後的參數。

### 2. 在 `MCP` 消息總線中

*   **目標：** 確保在「消息總線」上流動的所有消息封包都符合預定義的格式。
*   **實現思路：**
    *   在「消息總線引擎」中，定義一個 `MCP_Packet_Schema` 的 Zod schema。
    *   當消息總線從任何來源（如 `mcp:send` 工具）接收到一個消息時，它會首先使用 `MCP_Packet_Schema.safeParse(message)` 進行驗證。
    *   只有驗證通過的消息才會被路由到目標組件，格式錯誤的消息會被拒絕並記錄日誌。

### 3. 在 `Listener` 插件中

*   **目標：** 驗證來自不可信外部事件源的數據。
*   **實現思路：**
    *   一個 `Webhook_Listener` 在接收到外部的 HTTP `POST` 請求時，其 `body` 內容是不可信的。
    *   在將請求內容打包成核心事件對象之前，`Listener` 應該使用一個預定義的 Zod schema 來驗證 `body` 的結構和類型。
    *   這確保了推入核心「內部事件隊列」的 `payload` 始終是乾淨和結構化的。

通過在這些關鍵數據交換的「邊界」上使用 `Zod`，我們可以構建一個防禦性的、類型安全的代理人系統，有效避免因 LLM 的「幻覺」或外部髒數據導致的運行時錯誤。
