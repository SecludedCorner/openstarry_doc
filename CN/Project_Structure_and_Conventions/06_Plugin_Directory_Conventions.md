# 06. 插件目录结构规范 (Plugin Directory Conventions)

本文档落实了 **"Directory as Protocol"** 的核心设计哲学。它定义了 OpenStarry 系统如何识别、加载与验证一个插件。

## 1. 插件的物理形态

一个标准的 OpenStarry 插件就是一个 **NPM Package**。它可以发布到 npm registry，也可以直接存在于 `plugins/` 目录下的文件夹中。

### 标准目录布局

```text
my-awesome-plugin/
├── src/
│   ├── index.ts            # 入口文件 (必须导出插件类)
│   ├── tools/              # 工具逻辑 (如果有的话)
│   └── listeners/          # 监听器逻辑 (如果有的话)
├── tests/                  # 单元测试
├── package.json            # NPM 配置 (定义依赖与元数据)
├── plugin.json             # [可选] OpenStarry 专用元数据 (或合并入 package.json)
├── README.md               # 说明文档
└── tsconfig.json           # TypeScript 配置
```

## 2. 插件入口 (Entry Point)

系统加载器会寻找 `package.json` 中 `main` 字段指向的文件。该文件 **必须** 导出一个实现了 `IPlugin` 接口的类作为默认导出 (Default Export) 或命名导出 (Named Export)。

```typescript
// src/index.ts
import { IPlugin, IPluginContext } from '@openstarry/sdk';

export default class MyAwesomePlugin implements IPlugin {
  id = 'my-awesome-plugin';
  name = 'My Awesome Plugin';
  
  async initialize(context: IPluginContext) {
    // 1. 获取 Core 注入的依赖
    const apiKey = context.config.apiKey; // 从 agent.json 获取配置
    const logger = context.logger;
    
    // 2. 获取跨插件服务 (如果有的话)
    const mdParser = context.dependencies.markdownParser;

    logger.info('Initializing MyAwesomePlugin...');

    // 3. 注册五蕴成分
    context.registerTool(new MyTool(apiKey));
    context.registerListener(new MyListener());
  }
}
```

## 3. 元数据规范 (Manifest)

为了让 `Agent Design Service` 能够发现并管理插件，我们需要在 `package.json` 中添加 `openstarry` 字段，或者提供独立的 `plugin.json`。

**推荐方式：直接在 `package.json` 中扩展**

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

*   **type**: 插件类型 (`tool` | `provider` | `listener` | `bundle`)。
*   **capabilities**: 插件提供的能力标签 (用于 AI 搜索)。
*   **permissions**: 插件请求的权限 (Core 将据此生成安全策略)。

## 4. 插件分类目录

在 **Ecosystem Repo (`openstarry_plugin`)** 中，我们采用**极简扁平化结构**，所有功能聚合包直接存放于仓库根目录：

```text
openstarry_plugin/
├── @openstarry-plugin/standard-function-fs/      # 系统工具包
├── @openstarry-plugin/standard-function-stdio/   # 核心互动包
├── @openstarry-plugin/provider-gemini-oauth/     # 认知适配包
├── @openstarry-plugin/guide-mcp/                 # MCP 协议包
├── @openstarry-plugin/standard-function-skill/   # 技能加载包 (解析 Markdown)
└── @openstarry-plugin/experimental-weather-tool/ # 实验性插件
```

> **命名规范：** 所有官方插件均使用 `@openstarry-plugin/` 命名空间发布。在 `agent.json` 的 `plugins` 字段中引用时，必须使用完整包名（例如 `@openstarry-plugin/standard-function-stdio`），不再支持短名称。

这种结构确保了核心代码库 (`openstarry`) 的绝对纯净，所有具体能力都以外部模块形式存在。
