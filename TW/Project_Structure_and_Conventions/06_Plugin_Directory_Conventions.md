# 06. 插件目錄結構規範 (Plugin Directory Conventions)

本文件落實了 **"Directory as Protocol"** 的核心設計哲學。它定義了 OpenStarry 系統如何識別、加載與驗證一個插件。

## 1. 插件的物理形態

一個標準的 OpenStarry 插件就是一個 **NPM Package**。它可以發布到 npm registry，也可以直接存在於 `plugins/` 目錄下的文件夾中。

### 標準目錄佈局

```text
my-awesome-plugin/
├── src/
│   ├── index.ts            # 入口文件 (必須導出插件類)
│   ├── tools/              # 工具邏輯 (如果有的話)
│   └── listeners/          # 監聽器邏輯 (如果有的話)
├── tests/                  # 單元測試
├── package.json            # NPM 配置 (定義依賴與元數據)
├── plugin.json             # [可選] OpenStarry 專用元數據 (或合併入 package.json)
├── README.md               # 說明文檔
└── tsconfig.json           # TypeScript 配置
```

## 2. 插件入口 (Entry Point)

系統加載器會尋找 `package.json` 中 `main` 字段指向的文件。該文件 **必須** 導出一個實現了 `IPlugin` 接口的類別作為默認導出 (Default Export) 或命名導出 (Named Export)。

```typescript
// src/index.ts
import { IPlugin, IPluginContext } from '@openstarry/sdk';

export default class MyAwesomePlugin implements IPlugin {
  id = 'my-awesome-plugin';
  name = 'My Awesome Plugin';
  
  async initialize(context: IPluginContext) {
    // 1. 獲取 Core 注入的依賴
    const apiKey = context.config.apiKey; // 從 agent.json 獲取配置
    const logger = context.logger;
    
    // 2. 獲取跨插件服務 (如果有的話)
    const mdParser = context.dependencies.markdownParser;

    logger.info('Initializing MyAwesomePlugin...');

    // 3. 註冊五蘊成分
    context.registerTool(new MyTool(apiKey));
    context.registerListener(new MyListener());
  }
}
```

## 3. 元數據規範 (Manifest)

為了讓 `Agent Design Service` 能夠發現並管理插件，我們需要在 `package.json` 中添加 `openstarry` 字段，或者提供獨立的 `plugin.json`。

**推薦方式：直接在 `package.json` 中擴展**

```json
{
  "name": "@openstarry-plugin/weather",
  "version": "1.0.0",
  "main": "dist/index.js",
  "dependencies": {
    "@openstarry/sdk": "^0.1.0"
  },
  "openstarry": {
    "type": "tool", 
    "capabilities": ["weather-lookup", "forecast"],
    "permissions": {
      "network": ["api.weather.com"]
    }
  }
}
```

*   **type**: 插件類型 (`tool` | `provider` | `listener` | `bundle`)。
*   **capabilities**: 插件提供的能力標籤 (用於 AI 搜索)。
*   **permissions**: 插件請求的權限 (Core 將據此生成安全策略)。

## 4. 插件分類目錄

在 **Ecosystem Repo (`openstarry_plugin`)** 中，我們採用**極簡扁平化結構**，所有功能聚合包直接存放於倉庫根目錄：

```text
openstarry_plugin/
├── @openstarry-plugin/standard-function-fs/      # 系統工具包
├── @openstarry-plugin/standard-function-stdio/   # 核心互動包
├── @openstarry-plugin/provider-gemini-oauth/     # 認知適配包
├── @openstarry-plugin/guide-mcp/                 # MCP 協議包
├── @openstarry-plugin/standard-function-skill/   # 技能加載包 (解析 Markdown)
└── @openstarry-plugin/experimental-weather-tool/ # 實驗性插件
```

> **命名規範：** 所有官方插件均使用 `@openstarry-plugin/` 命名空間發布。在 `agent.json` 的 `plugins` 欄位中引用時，必須使用完整包名（例如 `@openstarry-plugin/standard-function-stdio`），不再支持短名稱。

這種結構確保了核心代碼庫 (`openstarry`) 的絕對純淨，所有具體能力都以外部模組形式存在。
