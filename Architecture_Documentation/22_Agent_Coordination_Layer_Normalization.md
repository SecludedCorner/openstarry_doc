# 22. 代理人協調層：歸一化與適配 (Agent Coordination Layer: Normalization & Adaptation)

## 1. 核心哲學

在多變且破碎的 AI 生態系中，**OpenStarry 代理人協調層 (Agent Coordination Layer)** 的核心價值在於扮演「萬能轉接頭」的角色。它通過 **歸一化 (Normalization)** 與 **適配 (Adaptation)** 機制，讓 Agent Core (內核) 能夠保持純淨與穩定，同時靈活對接任何外部 LLM 或工具。

我們拒絕讓內核去適配外部世界；我們強迫外部世界適配內核的標準協議。

---

## 2. 挑戰：巴別塔效應 (The Tower of Babel Effect)

目前的 LLM 生態系處於「戰國時代」，每家廠商對「工具調用 (Function Calling)」的實作各不相同：

*   **OpenAI:** 使用 `tools` 欄位，回傳 `tool_calls` 陣列。
*   **Google Gemini:** 使用 `function_declarations`，回傳結構化的 `functionCall` 物件或嵌套的 `parts`。
*   **Anthropic:** 使用 XML 標籤或特定的 `tools` 結構。
*   **Local LLMs (Llama/Mistral):** 可能需要特定的 Prompt 格式 (如 `[INST]`) 才能觸發工具。

如果直接在 `Agent Core` 中處理這些差異，內核將變得臃腫不堪且難以維護。每當新模型發布，內核就需要升級，這違反了微內核架構的「穩定性」原則。

---

## 3. 解決方案：協調層的雙向翻譯

協調層引入了一套 **OpenStarry 標準協議 (Standard Protocol)**，作為系統內部的唯一通用語言。

### A. 歸一化 (Normalization) - 內核視角
對於 `Agent Core` 而言，世界是標準化的：
*   **工具定義:** 永遠是標準的 JSON Schema (基於 `ITool` 介面)。
*   **思考結果:** 永遠是標準的 `ProviderResponse` 物件，包含 `content` (文字) 和 `toolCalls` (行動)。

Core 不需要知道它背後接的是 GPT-4 還是 Gemini 1.5，它只看得到標準協議。

### B. 適配 (Adaptation) - 插件視角
複雜性被推向了邊緣 (Edge)，即 **Provider Plugin**。每個 Provider 插件本質上是一個 **適配器 (Adapter)**：

1.  **上行適配 (Upstream Adaptation):**
    *   **輸入:** Core 提供的標準 `ITool[]` 清單。
    *   **轉換:** Provider 將其翻譯成目標 LLM 理解的格式 (例如 Gemini 的 `v1beta.FunctionDeclaration`)。
    *   **傳送:** 發送給 LLM API。

2.  **下行適配 (Downstream Adaptation):**
    *   **輸入:** LLM API 回傳的異質原始數據 (Raw Response)。
    *   **轉換:** 解析原始數據 (處理 XML、JSON、特殊欄位)，將其映射回 OpenStarry 的標準 `ProviderResponse`。
    *   **回傳:** 交給 Core 處理。

---

## 4. 架構優勢

### 1. 內核極簡化 (Microkernel Purity)
Core 的 `Execution Loop` 邏輯變得非常簡單：
```typescript
// Core 的邏輯永遠不變，無論模型是誰
const response = await provider.generate(prompt, context);
if (response.toolCalls) {
    executeTools(response.toolCalls);
}
```

### 2. 插件熱插拔 (Hot-Swappable Providers)
你可以隨時將 `provider-openai` 換成 `provider-gemini`，甚至換成 `provider-local-llama`。只要該插件遵守 `IProvider` 介面並完成了適配工作，Agent 的行為邏輯完全不需要修改。

### 3. 未來兼容性 (Future Proofing)
當新的 AI 模型或新的互動模式出現時，我們只需要開發一個新的適配器插件，而不需要重寫作業系統內核。

---

## 5. 結論

OpenStarry 的協調層不僅僅是連接器，它是 **熵減 (Entropy Reduction)** 的機制。它將外部世界的混亂有序化，轉化為內部系統的秩序。這就是為什麼我們的 Agent 能夠在變動的環境中保持穩定的「心智」結構。
