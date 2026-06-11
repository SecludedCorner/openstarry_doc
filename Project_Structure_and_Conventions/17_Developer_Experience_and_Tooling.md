# 17. 開發者體驗與工具鏈 (Developer Experience and Tooling)

為了讓開發者能快速構建、測試並分享 Agent 與插件，我們提供了一套完整的 CLI 工具鏈與開發環境。

## 1. OpenStarry CLI (`openstarry`)

`openstarry` 是用戶與開發者的唯一入口。相關基礎指令定義請參閱 **[09. CLI 設計與管理指令](./09_CLI_Design_and_Management_Commands.md)**。

### 常用開發指令 (Alias / Extensions)
*   `openstarry init <name>`: 創建一個新的 Agent 專案 (包含 `agent.json`, `config/`, `.env`)。
*   `openstarry create-plugin <name>`: 使用模板創建一個新的插件專案。
*   `openstarry dev`: 啟動本地開發模式 (Hot Reload)。
*   `openstarry plugin add`: 安裝依賴插件。
*   `openstarry start`: 啟動 Agent。

## 2. 插件腳手架 (Scaffolding)

執行 `openstarry create-plugin my-tool` 將會生成以下標準結構：

```text
my-tool/
├── src/
│   ├── index.ts        # 入口點
│   └── tools.ts        # 工具邏輯
├── tests/
│   └── index.test.ts   # 預置的單元測試
├── package.json        # 依賴配置
├── tsconfig.json       # TypeScript 配置
└── README.md           # 文檔模板
```

**模板特色：**
*   **Pre-configured Build:** 內置 `tsup` 或 `rollup` 配置，一鍵打包。
*   **Type Safety:** 自動引入 `@openstarry/sdk` 類型定義。
*   **Linting:** 預設 ESLint/Prettier 規則。

## 3. MockHost: 實驗室環境 (The Lab)

開發插件時，啟動一個完整的 Agent (包含 LLM 連接) 既昂貴又緩慢。我們提供 `MockHost` 測試環境。

### 功能
*   **模擬 Core 行為:** 開發者可以在測試腳本中模擬 Core 發出的指令 (如 `callTool`)。
*   **虛擬上下文:** 無需真實 LLM，直接注入假造的 Context 數據。
*   **快速反饋:** 支持 Watch 模式，修改代碼後立即重跑測試。

### 使用範例 (在插件測試中)

```typescript
import { TestAgentHost } from '@openstarry/sdk/testing';
import { MyPlugin } from '../src';

test('MyPlugin loads correctly', async () => {
  const host = new TestAgentHost();
  const plugin = new MyPlugin();
  
  await host.load(plugin);
  
  // 模擬 Core 調用工具
  const result = await host.executeTool('my_tool_function', { arg: 'value' });
  
  expect(result).toBe('success');
});
```

## 4. 調試與可視化 (Debugging)

*   **Inspector:** `openstarry dev` 啟動時會開啟一個本地 WebSocket 端口。開發者可以使用 `OpenStarry DevTools` (Web UI) 連接，實時查看 Agent 的思考過程、Context 變化與工具調用日誌。
*   **VS Code Extension:** (未來規劃) 直接在編輯器中語法高亮 Agent DSL，並提供插件代碼補全。