# 16. OpenStarry 標準協議 (OpenStarry Standard Protocol)

本文件定義了 OpenStarry 生態系中，Agent Core 與外部插件（特別是 Provider 與 Tools）溝通的技術標準。所有官方與第三方插件都必須遵守此協議，以確保跨組件的互操作性。

---

## 1. 指令路徑對比：斜線指令 vs. 自主行為

在 OpenStarry 中，同樣的一個工具（如 `fs.read`）可以透過兩條截然不同的路徑被觸發：

| 特性 | 斜線指令 (Slash Commands) | 自主行為 (Autonomous Actions) |
| :--- | :--- | :--- |
| **觸發源** | 用戶 (User) | 代理人大腦 (LLM/Provider) |
| **觸發方式** | 輸入 `/read filename.txt` | LLM 決定調用工具並回傳 `tool_call` |
| **繞過思考** | 是 (直接執行) | 否 (經過 OODA 迴圈決策) |
| **權限模型** | 用戶授權 (User Authorized) | 代理人自律 (Agent Autonomy) |
| **處理邏輯** | 由 `ExecutionLoop` 前置解析 | 由 `ExecutionLoop` 解析 `ProviderResponse` |

---

## 2. 轉譯指南：從異質 API 到標準格式

Provider 的核心價值在於「消除差異」。以下是以 Google Gemini API 為例的轉譯虛擬代碼：

### A. 上行轉譯 (將 Core 工具轉為 API 格式)
```typescript
// Core 的工具定義
const coreTool: ITool = { name: "read", description: "...", parameters: { ... } };

// 轉譯為 Gemini 格式
const geminiFunction = {
  name: coreTool.name,
  description: coreTool.description,
  parameters: coreTool.parameters // Gemini 接受 JSON Schema
};
```

### B. 下行轉譯 (將 API 回應轉為 Core 協議)
```typescript
// LLM API 的原始回傳 (Raw Content)
const geminiPart = { functionCall: { name: "read", args: { path: "a.txt" } } };

// 轉譯為 OpenStarry 標準協議
const response: ProviderResponse = {
  segments: [{
    type: 'tool_call',
    toolCall: {
      name: geminiPart.functionCall.name,
      args: geminiPart.functionCall.args
    }
  }]
};
```

---

## 3. 類型定義 (Type Definitions)

這些定義位於 `@openstarry/sdk` 的 `interfaces.ts` 中。

### A. 工具調用結構 (ToolCall)
這是 Agent 意圖執行某個動作的標準載體。

```typescript
export interface ToolCall {
  /**
   * 調用的唯一識別碼 (用於追蹤與回調匹配)
   * 某些 LLM (如 OpenAI) 會提供此 ID，若 LLM 未提供，Provider 應自動生成一個 UUID。
   */
  id?: string;

  /**
   * 要調用的工具名稱 (必須與註冊的 tool.name 完全匹配)
   */
  name: string;

  /**
   * 傳遞給工具的參數物件
   */
  args: any;
}
```

### B. Provider 回應結構 (ProviderResponse)
這是 `IProvider.generate()` 方法必須回傳的標準結果。為了支援多模態 (VLLM, Audio, UI)，我們採用 **分段式內容 (Segmented Content)** 設計。

```typescript
export type ContentType = 'text' | 'image' | 'audio' | 'video' | 'ui' | 'tool_call';

export interface ContentSegment {
  type: ContentType;
  
  /**
   * 用於 type='text'
   */
  text?: string;
  
  /**
   * 用於 type='image' | 'audio' | 'video'
   * 通常是 Base64 編碼的數據或 URL
   */
  data?: string;
  
  /**
   * 媒體類型 (MIME Type), e.g., "image/png", "audio/mp3"
   */
  mimeType?: string;

  /**
   * 用於 type='tool_call'
   */
  toolCall?: ToolCall;

  /**
   * 用於 type='ui'
   * 描述前端應渲染的組件 (JSON)
   */
  ui?: any;
}

export interface ProviderResponse {
  /**
   * 有序的內容段落列表
   * Agent 可以同時說話、展示圖片並執行工具。
   */
  segments: ContentSegment[];

  /**
   * 額外的元數據 (如 Token 使用量、模型延遲等)
   */
  metadata?: any;
}
```

---

## 2. 介面行為規範 (Interface Behavior Specifications)

### IProvider (想蘊)

Provider 插件必須實作 `generate` 方法，並遵循以下行為：

1.  **能力感知:** Provider 應從 `context.tools` 讀取當前可用的工具列表。
2.  **協議轉換:** 將 `context.tools` 轉換為目標 LLM API 所需的 Schema 格式。
3.  **結果解析:** 接收 LLM API 的回應，並將其標準化為 `ProviderResponse`。
    *   **純文字:** 回傳 `[{ type: 'text', text: '...' }]`
    *   **工具調用:** 回傳 `[{ type: 'tool_call', toolCall: { ... } }]`
    *   **混合模式:** 回傳多個 Segment。例如 Gemini 可能先回傳一段解釋 (text)，再回傳一個工具調用 (tool_call)。

```typescript
export interface IProvider {
  name: string;
  generate(prompt: string, context: IAgentContext): Promise<ProviderResponse>;
}
```

### IAgentContext (環境上下文)

Core 必須將自身的狀態與能力，透過 Context 暴露給 Provider。

```typescript
export interface IAgentContext {
  // ... 其他屬性
  
  /**
   * 能力反射 (Capability Reflection)
   * 讓 Provider 知道 Agent 當前擁有什麼工具 (行蘊)。
   * Map key 為工具名稱。
   */
  tools: Map<string, ITool>;
}
```

---

## 3. 交互時序範例 (Sequence Example)

以下是一個標準的「思考-行動」迴圈的協議流：

1.  **Input:** 用戶輸入 "查詢台北天氣"。
2.  **Core -> Provider:** 調用 `generate("查詢台北天氣", context)`。
    *   *Context 包含 `get_weather` 工具定義。*
3.  **Provider -> LLM API:** 發送 Prompt + Function Definition。
4.  **LLM API -> Provider:** 回傳原始 JSON (例如 Gemini 的 `functionCall` 結構)。
5.  **Provider (適配):** 將原始 JSON 轉換為 `ProviderResponse`:
    ```json
    {
      "content": "好的，我來查詢。",
      "toolCalls": [{ "name": "get_weather", "args": { "location": "Taipei" } }]
    }
    ```
6.  **Provider -> Core:** 回傳上述物件。
7.  **Core (執行):** 檢測到 `toolCalls`，根據 `name` 查找 `context.tools` 並執行。
8.  **Core (回饋):** 將工具執行結果 (如 "25°C, Sunny") 存入記憶，並再次調用 Provider (進入下一輪)。

---

## 4. 進階模式：大規模工具管理 (Advanced Pattern)

目前的標準協議採用「即時注入 (Just-in-Time Injection)」模式，適合中小型 Agent。對於擁有數百個工具的大型系統，建議採用以下優化策略：

### A. 靜態註冊與快取 (Registry & Caching)
針對支援 Context Caching 的 LLM (如 OpenAI Assistants 或 Gemini 1.5)，Provider 應在初始化階段一次性上傳工具 Schema，獲取 `tools_id`。後續對話僅需傳遞 ID，大幅降低 Token 消耗與延遲。

### B. 動態上下文過濾 (Dynamic Context Filtering)
為了避免 Context Window 被大量工具定義塞滿，Core 可引入「工具檢索層 (Tool Retrieval Layer)」：
1.  **意圖識別:** 分析用戶當前輸入的意圖 (Intent)。
2.  **向量檢索:** 從工具庫中檢索出 Top-N 最相關的工具。
3.  **精準注入:** 僅將這 N 個工具注入給 Provider，而非全部。

這確保了 Agent 即使擁有成千上萬的技能，也能保持輕量與專注。

---

## 5. 錯誤處理規範

*   **解析失敗:** 如果 Provider 無法解析 LLM 的輸出 (例如無效的 JSON)，應捕捉錯誤並在 `content` 中回傳錯誤描述，或拋出標準 `AgentPainException`。
*   **工具不存在:** 如果 Provider 回傳了一個不存在的 `toolCall.name`，Core 應捕捉此錯誤並將錯誤訊息作為「系統觀察」回饋給下一輪對話，讓 LLM 有機會自我修正。
