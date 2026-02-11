# 16. OpenStarry Standard Protocol

This document defines the technical standards for communication between the Agent Core and external plugins (specifically Providers and Tools) within the OpenStarry ecosystem. All official and third-party plugins must adhere to this protocol to ensure interoperability across components.

---

## 1. Command Path Comparison: Slash Commands vs. Autonomous Actions

In OpenStarry, the same tool (e.g., `fs.read`) can be triggered through two distinct paths:

| Feature | Slash Commands | Autonomous Actions |
| :--- | :--- | :--- |
| **Trigger Source** | User | Agent's Brain (LLM/Provider) |
| **Trigger Method** | Input `/read filename.txt` | LLM decides to invoke a tool and returns `tool_call` |
| **Bypass Thought** | Yes (Direct execution) | No (Via OODA loop decision) |
| **Permission Model** | User Authorized | Agent Autonomy |
| **Processing Logic** | Pre-parsed by `ExecutionLoop` | `ProviderResponse` parsed by `ExecutionLoop` |

---

## 2. Translation Guide: From Heterogeneous APIs to Standard Format

The core value of a Provider lies in "eliminating discrepancies." Below is pseudo-code for translation, using the Google Gemini API as an example:

### A. Upstream Translation (Translating Core Tools to API Format)
```typescript
// Core tool definition
const coreTool: ITool = { name: "read", description: "...", parameters: { ... } };

// Translate to Gemini format
const geminiFunction = {
  name: coreTool.name,
  description: coreTool.description,
  parameters: coreTool.parameters // Gemini accepts JSON Schema
};
```

### B. Downstream Translation (Translating API Response to Core Protocol)
```typescript
// Raw content returned by LLM API
const geminiPart = { functionCall: { name: "read", args: { path: "a.txt" } } };

// Translate to OpenStarry standard protocol
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

## 3. Type Definitions

These definitions are located in `interfaces.ts` of `@openstarry/sdk`.

### A. Tool Call Structure (ToolCall)
The standard carrier for an Agent's intent to perform an action.

```typescript
export interface ToolCall {
  /**
   * Unique identifier for the call (used for tracking and callback matching).
   * Some LLMs (e.g., OpenAI) provide this ID; if not provided, the Provider should generate a UUID.
   */
  id?: string;

  /**
   * Name of the tool to be called (must exactly match the registered tool.name).
   */
  name: string;

  /**
   * Argument object passed to the tool.
   */
  args: any;
}
```

### B. Provider Response Structure (ProviderResponse)
The standard result that the `IProvider.generate()` method must return. To support multi-modality (VLLM, Audio, UI), we adopt a **Segmented Content** design.

```typescript
export type ContentType = 'text' | 'image' | 'audio' | 'video' | 'ui' | 'tool_call';

export interface ContentSegment {
  type: ContentType;
  
  /**
   * Used for type='text'
   */
  text?: string;
  
  /**
   * Used for type='image' | 'audio' | 'video'
   * Typically Base64 encoded data or a URL.
   */
  data?: string;
  
  /**
   * Media type (MIME Type), e.g., "image/png", "audio/mp3"
   */
  mimeType?: string;

  /**
   * Used for type='tool_call'
   */
  toolCall?: ToolCall;

  /**
   * Used for type='ui'
   * Describes components to be rendered by the frontend (JSON).
   */
  ui?: any;
}

export interface ProviderResponse {
  /**
   * An ordered list of content segments.
   * The Agent can simultaneously speak, display images, and execute tools.
   */
  segments: ContentSegment[];

  /**
   * Additional metadata (e.g., Token usage, model latency).
   */
  metadata?: any;
}
```

---

## 4. Interface Behavior Specifications

### IProvider (Perception)

Provider plugins must implement the `generate` method and adhere to the following behaviors:

1.  **Capability Awareness:** The Provider should read the current list of available tools from `context.tools`.
2.  **Protocol Conversion:** Translate `context.tools` into the Schema format required by the target LLM API.
3.  **Result Parsing:** Receive the LLM API response and normalize it into a `ProviderResponse`.
    *   **Plain Text:** Return `[{ type: 'text', text: '...' }]`
    *   **Tool Call:** Return `[{ type: 'tool_call', toolCall: { ... } }]`
    *   **Mixed Mode:** Return multiple segments. For example, Gemini might return an explanation (text) followed by a tool call (tool_call).

```typescript
export interface IProvider {
  name: string;
  generate(prompt: string, context: IAgentContext): Promise<ProviderResponse>;
}
```

### IAgentContext (Environmental Context)

The Core must expose its state and capabilities to the Provider via the Context.

```typescript
export interface IAgentContext {
  // ... other properties
  
  /**
   * Capability Reflection
   * Informs the Provider of the tools (Volition) currently possessed by the Agent.
   * Map key is the tool name.
   */
  tools: Map<string, ITool>;
}
```

---

## 5. Sequence Example

Below is the protocol flow for a standard "Think-Act" loop:

1.  **Input:** User enters "Query weather in Taipei."
2.  **Core -> Provider:** Calls `generate("Query weather in Taipei", context)`.
    *   *Context contains the `get_weather` tool definition.*
3.  **Provider -> LLM API:** Sends Prompt + Function Definition.
4.  **LLM API -> Provider:** Returns raw JSON (e.g., Gemini's `functionCall` structure).
5.  **Provider (Adaptation):** Transforms raw JSON into `ProviderResponse`:
    ```json
    {
      "segments": [
        { "type": "text", "text": "Sure, let me check." },
        { "type": "tool_call", "toolCall": { "name": "get_weather", "args": { "location": "Taipei" } } }
      ]
    }
    ```
6.  **Provider -> Core:** Returns the above object.
7.  **Core (Execution):** Detects `tool_call`, locates `get_weather` in `context.tools`, and executes it.
8.  **Core (Feedback):** Stores the tool execution result (e.g., "25°C, Sunny") into memory and calls the Provider again (entering the next round).

---

## 6. Advanced Pattern: Large-Scale Tool Management

The current standard protocol uses a "Just-in-Time Injection" mode, suitable for small to medium-sized Agents. For large-scale systems with hundreds of tools, the following optimization strategies are recommended:

### A. Registry & Caching
For LLMs supporting Context Caching (e.g., OpenAI Assistants or Gemini 1.5), the Provider should upload tool Schemas once during initialization to obtain a `tools_id`. Subsequent dialogues only need to pass the ID, significantly reducing Token consumption and latency.

### B. Dynamic Context Filtering
To prevent the Context Window from being overwhelmed by massive tool definitions, the Core can introduce a "Tool Retrieval Layer":
1.  **Intent Recognition:** Analyzes the intent of the user's current input.
2.  **Vector Retrieval:** Retrieves the top-N most relevant tools from the tool library.
3.  **Precise Injection:** Injects only these N tools into the Provider.

This ensures the Agent remains lightweight and focused, even with thousands of skills.

---

## 7. Error Handling Specifications

*   **Parsing Failure:** If the Provider cannot parse the LLM's output (e.g., invalid JSON), it should catch the error and return an error description in the `text` segment or throw a standard `AgentPainException`.
*   **Tool Non-existent:** If the Provider returns a non-existent `toolCall.name`, the Core should catch this error and feed the error message back as a "system observation" in the next round of dialogue, giving the LLM an opportunity to self-correct.
