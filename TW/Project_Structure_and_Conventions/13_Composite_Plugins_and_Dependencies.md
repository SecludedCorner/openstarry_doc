# 13. 複合體與依賴機制 (Composites & Dependencies)

本文件定義了如何構建 **「複合體 (Composites)」** —— 即那些不生產底層能力，而是組合現有 Provider (想) 與 Tool (行)，並加上自定義 Listener (受) 來創造新應用場景的硬代碼模組。

## 1. 核心概念：能力依賴 (Capability Dependency)

為了保持解耦，複合體 (Composite) **不應** 直接依賴 Plugin A 的代碼。它應該依賴 Plugin A 所提供的 **「能力標籤 (Capability Tag)」**。

*   ❌ **錯誤依賴:** `import { GoogleSearch } from 'openstarry-plugin-google'`
*   ✅ **正確依賴:** `dependencies: { "tools": ["search"] }`

這樣，無論用戶安裝的是 Google Search 還是 Bing Search，只要它提供了 `search` 能力，Plugin C 都能運作。

## 2. 依賴聲明 (Manifest Declaration)

在 `plugin.json` 或 `package.json` 中，新增 `dependencies` 字段，指向其他**插件 ID**：

```json
{
  "id": "workflow-engine",
  "version": "1.0.0",
  "openstarry": {
    "type": "composite",
    "dependencies": {
      "plugins": ["@openstarry-plugin/standard-function-skill"] // 硬依賴
    }
  }
}
```

## 3. 基礎設施依賴 (Infrastructure Dependency)

某些高階插件（如工作流引擎）依賴於系統的基礎解析能力。

*   **場景：** 需要解析 Markdown 技能文件。
*   **開發實作：**
    ```typescript
    initialize(context: IPluginContext) {
      // 從協調層注入的依賴中獲取 Parser
      const parser = context.dependencies['@openstarry-plugin/standard-function-skill'].parser;
      
      if (!parser) throw new Error("Missing Skill Parser dependency");
    }
    ```

## 4. 協調層的職責 (Validation)

當 `PluginLoader` 加載 Plugin C 時，它會執行 **依賴檢查 (Dependency Check)**：

1.  檢查 Core 的 `ProviderRegistry` 是否已註冊了 LLM。
2.  檢查 Core 的 `ToolRegistry` 是否存在帶有 `web-search` 和 `fs-write` 標籤的工具。
3.  **結果:** 如果檢查失敗，Plugin C 將**拒絕啟動**，並報錯提示用戶安裝缺失的插件。

## 4. 開發實作：如何調用能力？

在插件代碼中，我們通過 `AgentContext` (注意：不是初始化時的 `PluginContext`，而是運行時事件帶來的 `AgentContext`) 來獲取能力。

```typescript
// plugins/experimental/research-assistant/index.ts

export default class ResearchPlugin implements IPlugin {
  async initialize(context: IPluginContext) {
    // 註冊一個監聽器，等待任務
    context.registerListener({
      id: 'research-trigger',
      onEvent: async (event, agentOps: IAgentOperations) => {
        
        // 1. 獲取大腦 (想)
        const llm = agentOps.getLLM();
        
        // 2. 獲取工具 (行)
        // 系統會自動查找標籤為 'web-search' 的工具
        const searchTool = agentOps.findToolByTag('web-search');
        
        if (!searchTool) {
          throw new Error("Missing required capability: web-search");
        }

        // 3. 執行邏輯
        const query = await llm.generate(`Extract keywords from: ${event.content}`);
        const results = await searchTool.call({ query });
        
        // 4. 反饋
        agentOps.reply(`Search results: ${results}`);
      }
    });
  }
}
```

## 5. 總結：生態系的層級

這種機制自然形成了生態系的層級：

*   **Level 1 (Foundation):** 提供原子能力的插件 (`fs`, `http`, `gemini`).
*   **Level 2 (Composites):** 組合 L1 能力的複合插件 (`researcher`, `coder`, `writer`).
*   **Level 3 (Agents):** 組合 L2 Composites 的完整數位物種。

這讓開發者可以像搭積木一樣構建複雜應用。
