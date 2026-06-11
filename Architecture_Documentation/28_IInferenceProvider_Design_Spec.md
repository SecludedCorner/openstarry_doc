# 28 — IInferenceProvider Design Spec

## 1. Problem Statement

目前 `IProvider` 介面專為 LLM 對話設計（`chat()` 接收 `ChatRequest` 含 `messages[]`），無法自然地承載 CNN/DNN 推論任務：

- CNN 影像分類需要的是 **image + label schema**，不是 chat messages
- DNN 特徵提取需要的是 **tensor input → vector output**，不是 streaming text
- 若將 CNN/DNN 掛為 ITool（行蘊），**哲學錯位** — 推論本質是「認知/想」，不是「行動/行」
- Workflow Engine 的 `llm` step 預設 `ChatRequest`，無法直接調用 inference-only 模型

## 2. Design Direction (方向 D)

**核心原則：CNN/DNN 是想蘊（Provider），不是行蘊（Tool）**

| 層 | 改動 | 說明 |
|----|------|------|
| SDK | 新增 `IInferenceProvider` | extends `IProvider`，加 `infer()` 方法 |
| Workflow Engine | 新增 `inference` step type | 直接調用 `IInferenceProvider.infer()` |
| Core | **零改動** | 微內核純淨（宣言 #7） |
| ProviderRegistry | **零改動** | `IInferenceProvider extends IProvider` → 自然註冊 |

## 3. SDK Layer

### 3.1 New File: `packages/sdk/src/types/inference.ts`

```typescript
import type { IProvider, ModelInfo } from "./provider.js";

// ─── Input Types ────────────────────────────────────────

/** Discriminated union of inference input formats. */
export type InferenceInput =
  | { type: "image"; data: Uint8Array; mimeType: string }
  | { type: "tensor"; shape: number[]; data: number[] }
  | { type: "text"; text: string }
  | { type: "raw"; data: unknown };

/** A request to an inference provider. */
export interface InferenceRequest {
  /** Model ID to use for inference. */
  model: string;

  /** Structured input data. */
  input: InferenceInput;

  /** Provider-specific options (e.g., confidence threshold). */
  options?: Record<string, unknown>;

  /** Optional abort signal. */
  signal?: AbortSignal;
}

// ─── Output Types ───────────────────────────────────────

/** A single classification label with confidence score. */
export interface ClassificationLabel {
  label: string;
  score: number;
}

/** A detected object with bounding box. */
export interface DetectedObject {
  label: string;
  score: number;
  /** [x, y, width, height] normalized to [0, 1]. */
  bbox: [number, number, number, number];
}

/** Discriminated union of inference output formats. */
export type InferenceOutput =
  | { type: "classification"; labels: ClassificationLabel[] }
  | { type: "features"; vector: number[] }
  | { type: "detection"; objects: DetectedObject[] }
  | { type: "text"; text: string }
  | { type: "raw"; data: unknown };

/** The result of an inference invocation. */
export interface InferenceResult {
  /** Model ID that produced the result. */
  model: string;

  /** Structured output data. */
  output: InferenceOutput;

  /** Provider-specific metadata (e.g., latency, confidence). */
  metadata?: Record<string, unknown>;
}

// ─── Interface ──────────────────────────────────────────

/**
 * Inference provider interface — extends IProvider for non-LLM models.
 *
 * CNN/DNN plugins implement this interface to provide structured inference
 * while remaining compatible with the standard ProviderRegistry (想蘊).
 *
 * The inherited `chat()` method serves as an adapter: converts the last
 * user message to text InferenceInput, runs inference, yields result as
 * text_delta. This allows inference providers to be used in LLM steps
 * as a fallback, though `inference` steps are preferred.
 */
export interface IInferenceProvider extends IProvider {
  /** Perform structured inference on the given input. */
  infer(request: InferenceRequest): Promise<InferenceResult>;
}

// ─── Type Guard ─────────────────────────────────────────

/**
 * Runtime type guard to check if a provider supports inference.
 *
 * Usage:
 * ```typescript
 * const provider = ctx.providers?.get("my-cnn");
 * if (isInferenceProvider(provider)) {
 *   const result = await provider.infer(request);
 * }
 * ```
 */
export function isInferenceProvider(
  provider: IProvider | undefined
): provider is IInferenceProvider {
  return provider !== undefined && typeof (provider as IInferenceProvider).infer === "function";
}
```

### 3.2 SDK Export: `packages/sdk/src/index.ts`

新增 export：

```typescript
export type {
  IInferenceProvider,
  InferenceRequest,
  InferenceResult,
  InferenceInput,
  InferenceOutput,
  ClassificationLabel,
  DetectedObject,
} from "./types/inference.js";

export { isInferenceProvider } from "./types/inference.js";
```

## 4. Workflow Engine Layer

### 4.1 New Step Type: `IInferenceStep`

**File**: `workflow-engine/src/types/workflow.ts`

```typescript
/**
 * Inference step — invokes an IInferenceProvider directly.
 */
export interface IInferenceStep extends IWorkflowStepBase {
  type: "inference";

  /** Provider ID (must implement IInferenceProvider). */
  provider: string;

  /** Optional model override (default: first model in provider). */
  model?: string;

  /** Inference input definition (supports Mustache templates in values). */
  input: {
    type: "image" | "tensor" | "text" | "raw";
    [key: string]: unknown;  // type-specific fields, Mustache-interpolated
  };

  /** Optional provider-specific options. */
  options?: Record<string, unknown>;
}
```

**Changes to existing types**:

- `IWorkflowStepBase.type` → add `"inference"` to union: `"tool" | "service" | "llm" | "command" | "inference"`
- `IWorkflowStep` union → add `| IInferenceStep`

### 4.2 New Executor: `inference-executor.ts`

**File**: `workflow-engine/src/engine/executors/inference-executor.ts`

```typescript
import type { IPluginContext, InferenceRequest, InferenceInput } from "@openstarry/sdk";
import { isInferenceProvider } from "@openstarry/sdk";
import type { IInferenceStep } from "../../types/workflow.js";
import { WorkflowExecutionError } from "../../errors.js";
import { interpolate } from "../interpolate.js";

export async function executeInferenceStep(
  step: IInferenceStep,
  context: Record<string, unknown>,
  pluginCtx: IPluginContext,
  executionId: string
): Promise<unknown> {
  // 1. Resolve provider
  const provider = pluginCtx.providers?.get(step.provider);
  if (!provider) {
    const available = pluginCtx.providers?.list().map(p => p.id) || [];
    throw new WorkflowExecutionError(
      "unknown", executionId,
      `Provider "${step.provider}" not found. Available: ${available.join(", ") || "none"}`,
      step.name
    );
  }

  // 2. Type-check: must be IInferenceProvider
  if (!isInferenceProvider(provider)) {
    throw new WorkflowExecutionError(
      "unknown", executionId,
      `Provider "${step.provider}" does not support inference (missing infer() method). Use an "llm" step for LLM providers.`,
      step.name
    );
  }

  // 3. Interpolate input fields
  const interpolatedInput = interpolate(step.input, context) as Record<string, unknown>;
  const input = buildInferenceInput(interpolatedInput);

  // 4. Build request
  const request: InferenceRequest = {
    model: step.model || provider.models?.[0]?.id || "default",
    input,
    ...(step.options && { options: interpolate(step.options, context) as Record<string, unknown> }),
  };

  // 5. Execute inference
  try {
    const result = await provider.infer(request);
    return result;  // Full InferenceResult stored in context.steps.<name>
  } catch (error) {
    throw new WorkflowExecutionError(
      "unknown", executionId,
      `Inference provider failed: ${error instanceof Error ? error.message : String(error)}`,
      step.name, error
    );
  }
}

function buildInferenceInput(raw: Record<string, unknown>): InferenceInput {
  const type = raw.type as string;
  switch (type) {
    case "text":
      return { type: "text", text: raw.text as string };
    case "image":
      return {
        type: "image",
        data: raw.data as Uint8Array,
        mimeType: (raw.mimeType as string) || "image/png",
      };
    case "tensor":
      return {
        type: "tensor",
        shape: raw.shape as number[],
        data: raw.data as number[],
      };
    case "raw":
      return { type: "raw", data: raw.data };
    default:
      return { type: "raw", data: raw };
  }
}
```

### 4.3 WorkflowEngine Integration

**File**: `workflow-engine/src/engine/workflow-engine.ts`

`executeStep()` switch 加一個 case：

```typescript
case "inference":
  return executeInferenceStep(step, context, this.ctx, executionId);
```

### 4.4 Zod Schema Extension

**File**: `workflow-engine/src/schema/workflow-schema.ts`

新增 `InferenceStepSchema`：

```typescript
const InferenceStepSchema = BaseStepSchema.extend({
  type: z.literal("inference"),
  provider: z.string().min(1),
  model: z.string().optional(),
  input: z.object({
    type: z.enum(["image", "tensor", "text", "raw"]),
  }).passthrough(),  // Allow additional fields based on type
  options: z.record(z.unknown()).optional(),
});
```

`StepSchema` union 加入：

```typescript
const StepSchema = z.discriminatedUnion("type", [
  ToolStepSchema,
  ServiceStepSchema,
  LLMStepSchema,
  CommandStepSchema,
  InferenceStepSchema,  // NEW
]);
```

## 5. Core Layer

**零改動。**

- `ProviderRegistry` 存 `Map<string, IProvider>`，`IInferenceProvider extends IProvider` → 自然 fit
- `agent-core.ts` 不碰推論邏輯
- `resolveProvider()` / `resolveModel()` 不需修改

## 6. Plugin Implementation Pattern

### 6.1 CNN/DNN Plugin 範例結構

```
openstarry_plugin/
  provider-image-classifier/
    package.json
    tsconfig.json
    README.md
    src/
      index.ts          # createImageClassifierPlugin()
      classifier.ts     # IInferenceProvider 實作
      index.test.ts
```

### 6.2 Plugin Factory Pattern

```typescript
import type { IPlugin, IPluginContext, PluginHooks, ChatRequest, ProviderStreamEvent } from "@openstarry/sdk";
import type { IInferenceProvider, InferenceRequest, InferenceResult } from "@openstarry/sdk";

export function createImageClassifierPlugin(): IPlugin {
  return {
    manifest: {
      name: "@openstarry-plugin/provider-image-classifier",
      version: "0.1.0-alpha",
      description: "CNN image classifier (想蘊 inference provider)",
    },
    async factory(ctx: IPluginContext): Promise<PluginHooks> {
      const provider: IInferenceProvider = {
        id: "image-classifier",
        name: "Image Classifier (CNN)",
        models: [
          { id: "mobilenet-v3", name: "MobileNet V3" },
          { id: "resnet-50", name: "ResNet-50" },
        ],

        // Adapter: chat() converts last message to text inference
        async *chat(request: ChatRequest): AsyncGenerator<ProviderStreamEvent> {
          const lastMsg = request.messages[request.messages.length - 1];
          const text = lastMsg?.content
            .filter(s => s.type === "text")
            .map(s => (s as { type: "text"; text: string }).text)
            .join(" ") || "";

          const result = await this.infer({
            model: request.model,
            input: { type: "text", text },
            signal: request.signal,
          });

          // Yield result as text
          const outputText = JSON.stringify(result.output, null, 2);
          yield { type: "text_delta", text: outputText };
          yield { type: "finish", stopReason: "end_turn", usage: { totalTokens: 0 } };
        },

        // Core inference method
        async infer(request: InferenceRequest): Promise<InferenceResult> {
          // ... actual CNN inference logic ...
          return {
            model: request.model,
            output: {
              type: "classification",
              labels: [
                { label: "cat", score: 0.95 },
                { label: "dog", score: 0.03 },
              ],
            },
          };
        },
      };

      return {
        providers: [provider],
      };
    },
  };
}
```

### 6.3 chat() Adapter 行為

| 場景 | 行為 |
|------|------|
| LLM step 呼叫 inference provider | chat() adapter 自動轉換 → infer() → JSON text output |
| Inference step 呼叫 inference provider | 直接 infer() → structured InferenceResult |
| LLM step 呼叫 LLM provider | 正常 chat() → streaming text |
| Inference step 呼叫 LLM provider | 報錯：「Provider does not support inference」 |

## 7. Workflow YAML Example

### 7.1 Image Classification Pipeline

```yaml
name: image-classification
version: 1.0.0
description: Classify image then describe with VLM

inputs:
  image_path:
    type: string
    required: true
    description: Path to the image file

steps:
  - name: read-image
    type: tool
    tool: "fs:read-binary"
    arguments:
      path: "{{inputs.image_path}}"

  - name: classify
    type: inference
    provider: image-classifier
    model: mobilenet-v3
    input:
      type: image
      data: "{{steps.read-image}}"
      mimeType: image/png

  - name: describe
    type: llm
    provider: claude
    prompt: |
      Based on the classification result: {{steps.classify.output}}
      Please describe what you think this image contains.

outputs:
  classification: "{{steps.classify.output}}"
  description: "{{steps.describe}}"
```

### 7.2 Multi-stage Pipeline (CNN → DNN → VLM)

```yaml
name: image-analysis-pipeline
version: 1.0.0

inputs:
  image_data:
    type: object
    required: true

steps:
  - name: detect-objects
    type: inference
    provider: object-detector
    model: yolov8
    input:
      type: image
      data: "{{inputs.image_data.buffer}}"
      mimeType: "{{inputs.image_data.mimeType}}"

  - name: extract-features
    type: inference
    provider: feature-extractor
    model: resnet-50
    input:
      type: image
      data: "{{inputs.image_data.buffer}}"
      mimeType: "{{inputs.image_data.mimeType}}"

  - name: vlm-analysis
    type: llm
    provider: gemini
    model: gemini-2.0-flash
    prompt: |
      I detected these objects: {{steps.detect-objects.output.objects}}
      Feature vector: {{steps.extract-features.output.vector}}
      Please provide a comprehensive analysis.

outputs:
  detections: "{{steps.detect-objects.output}}"
  features: "{{steps.extract-features.output}}"
  analysis: "{{steps.vlm-analysis}}"
```

## 8. Interface Compatibility Matrix

| 現有 Provider | IProvider | IInferenceProvider | chat() | infer() |
|---|---|---|---|---|
| provider-claude | ✅ | ❌ | ✅ Native | N/A |
| provider-chatgpt | ✅ | ❌ | ✅ Native | N/A |
| provider-gemini | ✅ | ❌ | ✅ Native | N/A |
| provider-local-llama | ✅ | ❌ | ✅ Native | N/A |
| provider-image-classifier (新) | ✅ | ✅ | ✅ Adapter | ✅ Native |
| provider-object-detector (新) | ✅ | ✅ | ✅ Adapter | ✅ Native |

## 9. Five Aggregates Alignment

| 蘊 | 介面 | 作用 | 推論 Provider 歸屬 |
|----|------|------|---------------------|
| 色（IUI） | UI 介面 | 感知呈現 | — |
| 受（IListener） | 事件監聽 | 接收刺激 | — |
| **想（IProvider）** | **認知提供者** | **產生認知** | **✅ CNN/DNN 在此** |
| 行（ITool） | 工具執行 | 對外行動 | ❌ 不在此 |
| 識（IGuide） | 系統指引 | 引導方向 | — |

CNN/DNN 的本質是「從輸入產生認知」（分類、特徵提取、物件偵測），這正是想蘊的語義。將其掛為 IProvider（想）而非 ITool（行）是哲學正確的。

## 10. Backward Compatibility

| 面向 | 影響 |
|------|------|
| 現有 IProvider 實作 | **零影響** — IInferenceProvider 是 extends，不改原型 |
| 現有 LLM Step | **零影響** — 仍使用 `chat()` |
| 現有 ProviderRegistry | **零影響** — IInferenceProvider 是 IProvider 子型 |
| 現有 Workflow YAML | **零影響** — `inference` 是新 step type |
| Core | **零改動** |
| SDK | **純增量** — 新檔 + 新 exports |
| Workflow Engine | **增量** — 新 step type + executor |

## 11. Implementation Scope

### Phase 1: SDK + Workflow Engine（本 Cycle）

| 改動 | 檔案 | 類型 |
|------|------|------|
| InferenceInput/Output/Request/Result types | `packages/sdk/src/types/inference.ts` | NEW |
| IInferenceProvider interface + isInferenceProvider guard | 同上 | NEW |
| SDK exports | `packages/sdk/src/index.ts` | EDIT |
| IInferenceStep type | `workflow-engine/src/types/workflow.ts` | EDIT |
| Inference executor | `workflow-engine/src/engine/executors/inference-executor.ts` | NEW |
| WorkflowEngine.executeStep() | `workflow-engine/src/engine/workflow-engine.ts` | EDIT |
| Zod schema | `workflow-engine/src/schema/workflow-schema.ts` | EDIT |
| Tests | `packages/sdk/src/types/__tests__/inference.test.ts` | NEW |
| Tests | `workflow-engine/src/engine/executors/__tests__/inference-executor.test.ts` | NEW |
| Tests | `workflow-engine/src/schema/__tests__/inference-schema.test.ts` | NEW |

### Phase 2: Reference Plugin（Future）

- `provider-image-classifier` — MobileNet/ResNet CNN 分類
- `provider-object-detector` — YOLO 物件偵測
- `provider-feature-extractor` — DNN 特徵提取

## 12. Frozen Interfaces

以下介面一旦 Merge 即凍結，變更需 Spec Addendum：

- `IInferenceProvider`
- `InferenceRequest`
- `InferenceResult`
- `InferenceInput` (discriminated union)
- `InferenceOutput` (discriminated union)
- `IInferenceStep`
- `isInferenceProvider()`

## 13. Verification Criteria

1. `pnpm build` — 全 workspace 通過
2. `pnpm test` — 全測試通過（新增 ≥ 15 tests）
3. 現有 1418 tests 全不受影響
4. `isInferenceProvider()` type guard 正確區分 IProvider vs IInferenceProvider
5. Workflow `inference` step 能正確調用 IInferenceProvider.infer()
6. Workflow `inference` step 呼叫非 IInferenceProvider 時報錯
7. Workflow `llm` step 仍然正常（backward compat）
8. Core 零改動確認
