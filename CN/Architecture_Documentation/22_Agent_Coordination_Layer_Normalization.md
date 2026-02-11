# 22. 代理人协调层：归一化与适配 (Agent Coordination Layer: Normalization & Adaptation)

## 1. 核心哲学

在多变且破碎的 AI 生态系中，**OpenStarry 代理人协调层 (Agent Coordination Layer)** 的核心价值在于扮演「万能转接头」的角色。它通过 **归一化 (Normalization)** 与 **适配 (Adaptation)** 机制，让 Agent Core (内核) 能够保持纯净与稳定，同时灵活对接任何外部 LLM 或工具。

我们拒绝让内核去适配外部世界；我们强迫外部世界适配内核的标准协议。

---

## 2. 挑战：巴别塔效应 (The Tower of Babel Effect)

目前的 LLM 生态系处于「战国时代」，每家厂商对「工具调用 (Function Calling)」的实现各不相同：

*   **OpenAI:** 使用 `tools` 字段，回传 `tool_calls` 数组。
*   **Google Gemini:** 使用 `function_declarations`，回传结构化的 `functionCall` 对象或嵌套的 `parts`。
*   **Anthropic:** 使用 XML 标签或特定的 `tools` 结构。
*   **Local LLMs (Llama/Mistral):** 可能需要特定的 Prompt 格式 (如 `[INST]`) 才能触发工具。

如果直接在 `Agent Core` 中处理这些差异，内核将变得臃肿不堪且难以维护。每当新模型发布，内核就需要升级，这违反了微内核架构的「稳定性」原则。

---

## 3. 解决方案：协调层的双向翻译

协调层引入了一套 **OpenStarry 标准协议 (Standard Protocol)**，作为系统内部的唯一通用语言。

### A. 归一化 (Normalization) - 内核视角
对于 `Agent Core` 而言，世界是标准化的：
*   **工具定义:** 永远是标准的 JSON Schema (基于 `ITool` 接口)。
*   **思考结果:** 永远是标准的 `ProviderResponse` 对象，包含 `content` (文字) 和 `toolCalls` (行动)。

Core 不需要知道它背后接的是 GPT-4 还是 Gemini 1.5，它只看得到标准协议。

### B. 适配 (Adaptation) - 插件视角
复杂性被推向了边缘 (Edge)，即 **Provider Plugin**。每个 Provider 插件本质上是一个 **适配器 (Adapter)**：

1.  **上行适配 (Upstream Adaptation):**
    *   **输入:** Core 提供的标准 `ITool[]` 清单。
    *   **转换:** Provider 将其翻译成目标 LLM 理解的格式 (例如 Gemini 的 `v1beta.FunctionDeclaration`)。
    *   **传送:** 发送给 LLM API。

2.  **下行适配 (Downstream Adaptation):**
    *   **输入:** LLM API 回传的异质原始数据 (Raw Response)。
    *   **转换:** 解析原始数据 (处理 XML、JSON、特殊字段)，将其映射回 OpenStarry 的标准 `ProviderResponse`。
    *   **回传:** 交给 Core 处理。

---

## 4. 架构优势

### 1. 内核极简化 (Microkernel Purity)
Core 的 `Execution Loop` 逻辑变得非常简单：
```typescript
// Core 的逻辑永远不变，无论模型是谁
const response = await provider.generate(prompt, context);
if (response.toolCalls) {
    executeTools(response.toolCalls);
}
```

### 2. 插件热插拔 (Hot-Swappable Providers)
你可以随时将 `provider-openai` 换成 `provider-gemini`，甚至换成 `provider-local-llama`。只要该插件遵守 `IProvider` 接口并完成了适配工作，Agent 的行为逻辑完全不需要修改。

### 3. 未来兼容性 (Future Proofing)
当新的 AI 模型或新的互动模式出现时，我们只需要开发一个新的适配器插件，而不需要重写操作系统内核。

---

## 5. 结论

OpenStarry 的协调层不仅仅是连接器，它是 **熵减 (Entropy Reduction)** 的机制。它将外部世界的混乱有序化，转化为内部系统的秩序。这就是为什么我们的 Agent 能够在变动的环境中保持稳定的「心智」结构。
