# 開發指南

## 測試

```bash
# 執行所有測試（核心 + 插件）
pnpm test

# 監聽模式（檔案修改時自動重跑）
pnpm test:watch

# 微核心純淨性檢查（確保 core 不依賴插件）
pnpm test:purity

# 只跑特定測試
pnpm test -- packages/core/src/bus/event-bus.test.ts
pnpm test -- -t "FIFO"
```

## 開發新插件

### 使用腳手架工具

```bash
node apps/runner/dist/bin.js create-plugin
```

依照互動式提示填寫插件名稱、描述等資訊，工具會自動產生插件模板。

### 手動建立

```typescript
// my-plugin/src/index.ts
import type { IPlugin, IPluginContext, PluginHooks } from "@openstarry/sdk";

export function createMyPlugin(): IPlugin {
  return {
    manifest: {
      name: "my-plugin",
      version: "0.1.0",
      description: "My custom plugin",
    },
    async factory(ctx: IPluginContext): Promise<PluginHooks> {
      return {
        tools: [/* ITool instances */],
        listeners: [/* IListener instances */],
        ui: [/* IUI instances */],
      };
    },
  };
}

export default createMyPlugin;
```

### 插件結構

```
my-plugin/
├── package.json
├── tsconfig.json
├── src/
│   ├── index.ts          # 插件入口，export createMyPlugin()
│   └── index.test.ts     # 測試
└── dist/                 # 編譯輸出
```

### 重要規範

- 工廠模式：export `createXxxPlugin()` → 回傳 `IPlugin`
- `factory(ctx)` 接收 `IPluginContext`，回傳 `PluginHooks`
- 使用 `ctx.pushInput()` 與核心通訊，**不直接調用核心 API**
- 插件套件名：`@openstarry-plugin/xxx`
- vitest 版本需與根專案一致（`^4.0.18`）
- tsconfig 需排除測試檔案：`"exclude": ["src/**/*.test.ts"]`

## 編譯

```bash
# 編譯所有套件（核心 + 插件）
pnpm build

# 編譯單一套件
pnpm --filter @openstarry/core build
```

## 延伸閱讀

- `packages/sdk/README.md` — 所有介面定義
- `packages/core/README.md` — 核心架構
- `apps/runner/README.md` — CLI 指令細節
