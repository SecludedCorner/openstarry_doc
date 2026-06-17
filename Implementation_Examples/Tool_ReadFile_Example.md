# 實作範例：Tool - 讀取文件

> ⚠️ **[漂移更正 — v0.59.6 stale-API；下方原文為早期概念草圖，已被實際 SDK 取代]**
>
> 下方原始 `javascript` 範例描述的 Tool 介面**與實際 shipped 程式碼不符**，請勿照抄。實證 API 如下：
>
> **真正的 `ITool` 介面**（`packages/sdk/src/types/tool.ts:52-64`）—— `extends ISamskara`：
> - `id: string`（**不是** `name`）
> - `description: string`
> - `parameters: z.ZodType<TInput>`（Zod schema，**不是**手寫 plain-object 的 `args`）
> - `execute(input: TInput, ctx: ToolContext): Promise<string>`（**兩個參數**：`input` + `ctx`，**不是**單一 `args`）
> - `metadata?: IToolMetadata`（選用；`hasSideEffects` / `riskCategory` / `requiresConfirmation`）
>
> `ToolContext`（同檔 `:10-16`）提供 `workingDirectory` / `allowedPaths` / `signal?` / `bus`，工具用它做路徑沙箱與取消。
>
> **真正的交付方式**：工具是**plain object 字面量**（非 class、非 instance），透過**工廠函式**回傳的 `PluginHooks.tools` 陣列註冊——**不是** `module.exports = new Class()`。
> 權威參考實作見 `standard-function-fs/src/index.ts`：`fsReadTool`（`:46-64`，`id: "fs.read"`）與 `createFsPlugin()`（`:157-172`，`factory(ctx)` 回傳 `{ tools: [fsReadTool, ...] }`）。
>
> **最小正確骨架**（對照下方陳舊版本）：
>
> ```typescript
> import { z } from "zod";
> import type { IPlugin, IPluginContext, PluginHooks, ITool, ToolContext } from "@openstarry/sdk";
>
> const fsReadTool: ITool<{ path: string; encoding?: string }> = {
>   skandha: "samskara" as const,
>   id: "fs.read",
>   description: "Read the contents of a file. Returns the file content as a string.",
>   parameters: z.object({
>     path: z.string().describe("The file path to read (relative or absolute)"),
>     encoding: z.string().optional().describe("File encoding (default: utf-8)"),
>   }),
>   async execute(input, ctx: ToolContext) {
>     // 用 ctx.workingDirectory / ctx.allowedPaths 做路徑沙箱驗證後再讀檔
>     const content = await readFile(validatePath(input.path, ctx), input.encoding ?? "utf-8");
>     return content;
>   },
> };
>
> export function createFsPlugin(): IPlugin {
>   return {
>     manifest: { name: "standard-function-fs", version: "0.1.0-alpha", skandha: "samskara" as const },
>     async factory(_ctx: IPluginContext): Promise<PluginHooks> {
>       return { tools: [fsReadTool] };
>     },
>   };
> }
> ```
>
> 以下原始 class 範例保留作歷史紀錄。

本文件提供一個簡單的 Tool 插件如何實現其接口的具體程式碼範例。

---

> 🕰️ **以下為陳舊草圖（historical）—— `name` / plain-object `args` / `execute(args)` / `module.exports = new Class()` 皆已不再是實際 API。以上方骨架為準。**

```javascript
// 示例：一個用於讀取文件的 Tool 插件
class ReadFile_Tool {
  // name: 工具的唯一標識符，供 LLM 調用
  name = "file_system:read_file";

  // description: 工具功能的詳細描述，供 LLM 理解其用途
  description = "Reads the entire content of a specified file and returns it as a string.";

  // args: 描述工具需要的參數，供 LLM 構造請求和核心進行驗證
  // 這裡可以使用 JSON Schema 等格式來進行更嚴格的定義
  args = {
    path: {
      type: "string",
      description: "The absolute or relative path to the file."
    }
  };

  /**
   * 執行工具的核心邏輯
   * @param {object} args - 經過驗證的參數對象，例如 { path: "/path/to/file.txt" }
   * @returns {Promise<string>} - 工具執行的結果
   */
  async execute(args) {
    const fs = require('fs/promises');
    try {
      const content = await fs.readFile(args.path, 'utf-8');
      return content;
    } catch (error) {
      // 最好返回一個結構化的錯誤信息
      return `Error reading file: ${error.message}`;
    }
  }
}

// 插件入口點，導出工具實例或類
module.exports = new ReadFile_Tool();
```
