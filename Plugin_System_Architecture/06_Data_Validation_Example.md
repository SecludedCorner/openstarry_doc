# 插件範例：數據驗證 (Zod)

> ⚠️ **[漂移更正 — v0.59.6 設計示意，非實作對照]** 本文件混雜「概念正確但 API 已漂移」與「整段虛構層」兩類問題，已逐處插牌；原文保留為歷史設計敘述。
>
> 1. **§1 工具參數欄位名稱已漂移**：真實 `ITool` 介面（`packages/sdk/src/types/tool.ts:53-57`）的 schema 欄位叫 **`parameters: z.ZodType<TInput>`**，不是 `args`；且工具是 plain object（`id` / `description` / `parameters` / `execute(input, ctx)`），不是本文示範的 `class MyTool { name; args; execute(args) }`。`Zod` 仍確實是驗證機制——只是欄位名與形狀請以 SDK 型別為準。
> 2. **§2「MCP 消息總線」整段為虛構層**：核心程式碼**不存在**「消息總線引擎」或 `MCP_Packet_Schema`（全 `packages/` 0 grep 命中）。此段與本資料夾已隔離的 doc 01/02 屬同一虛構家族，請勿據此實作。
>
> 權威鏈：SDK 型別檔（`packages/sdk/src/types/tool.ts`）＝ API 最高權威；任何 doc 與 SDK 衝突，SDK 贏。

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
    1.  在 `Tool` 插件的定義中，schema 欄位可以直接是一個 `Zod` schema 對象。

        > ⚠️ **[漂移更正 — v0.59.6]** 下方原始示例的欄位名（`args`）與形狀（`class MyTool`）已過時。真實 `ITool`（`packages/sdk/src/types/tool.ts:53-57`）長這樣：
        > ```typescript
        > import { z } from 'zod';
        >
        > const createUser: ITool = {
        >   id: 'create_user',
        >   description: 'Creates a new user in the system.',
        >   parameters: z.object({        // ← 欄位叫 parameters，不是 args
        >     username: z.string().min(3),
        >     email: z.string().email(),
        >     age: z.number().optional(),
        >   }),
        >   async execute(input, ctx) {    // ← (input, ctx)，input 已通過 Zod 驗證
        >     // ...
        >     return 'ok';
        >   },
        > };
        > ```

        以下為原始（已漂移）敘述，保留作歷史：
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
    2.  在「代理人核心」的「執行循環」中，當它準備調用一個工具時，它會先從 `ToolRegistry` 獲取工具的 schema（真實欄位 `parameters`）。
    3.  然後，它使用 `schema.safeParse(llm_generated_args)` 來驗證 LLM 提供的參數。
    4.  如果驗證失敗，它會將 `Zod` 生成的詳細錯誤信息作為「工具執行失敗」的結果，反饋給 LLM，讓 LLM 自我糾正並生成正確的參數。
    5.  如果驗證成功，才調用工具的 `execute` 方法，並傳入經過 `Zod` 清洗和類型轉換後的參數。

### 2. 在 `MCP` 消息總線中

> ⚠️ **[虛構層 — v0.59.6]** 本節描述的「消息總線引擎」與 `MCP_Packet_Schema` **在核心程式碼中不存在**（`packages/` 全域 0 grep 命中），屬與本資料夾 doc 01/02 同一虛構家族。請勿據此實作；MCP 整合的真實路徑請參考實際出貨的 `transport-*` / `mcp-*` 插件。以下原文僅保留為歷史設計示意。

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
