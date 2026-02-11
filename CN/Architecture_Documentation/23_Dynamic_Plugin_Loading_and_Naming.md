# 23. 动态插件载入与命名规范 (Dynamic Plugin Loading & Naming Convention)

本文档说明 OpenStarry 的插件解析机制与套件命名规范。

---

## 1. 设计原则

Runner（`apps/runner`）是一个**纯启动器**，不认识任何具体插件。所有插件——包括 Provider、Tool、Listener、Guide——都通过 `agent.json` 配置并在运行时动态载入。

这意味着：
- Runner 的 `package.json` **不依赖**任何 `@openstarry-plugin/*` 套件
- Runner 的源码**不 import** 任何插件模块
- 新增插件不需要修改 Runner 的任何代码

---

## 2. 套件命名规范

所有官方插件统一使用 `@openstarry-plugin/` scope：

```
@openstarry-plugin/{plugin-name}
```

| 完整套件名 | 类型 | 说明 |
|-----------|------|------|
| `@openstarry-plugin/provider-gemini-oauth` | Provider (想) | Gemini LLM + OAuth 认证 |
| `@openstarry-plugin/standard-function-fs` | Tool (行) | 文件系统操作 |
| `@openstarry-plugin/standard-function-stdio` | Listener + Guide (受 + 识) | CLI 终端 I/O + 默认人设 |
| `@openstarry-plugin/standard-function-skill` | Guide (识) | Markdown 技能文件载入 |

### 在 agent.json 中的使用

```json
{
  "plugins": [
    { "name": "@openstarry-plugin/provider-gemini-oauth" },
    { "name": "@openstarry-plugin/standard-function-fs" },
    { "name": "@openstarry-plugin/standard-function-stdio" },
    {
      "name": "@openstarry-plugin/standard-function-skill",
      "config": { "skillPath": "./skills/coder.md" }
    }
  ]
}
```

---

## 3. 插件解析策略（两层）

Runner 的 `resolvePlugins()` 对 `agent.json` 中的每个 plugin entry 按顺序尝试：

### Strategy 1：文件路径载入

当 `ref.path` 有值时，直接 `import()` 该文件路径。

```json
{
  "name": "my-custom-plugin",
  "path": "../my-plugins/custom-tool/dist/index.js"
}
```

**适用场景：**
- 本地开发中的插件（还没发布到 npm）
- 第三方插件不在 pnpm workspace 中
- 别人给你的单一 `.js` 插件文件

### Strategy 2：套件名动态载入

尝试 `import(ref.name)`，由 Node.js 的模块解析机制找到套件。

```json
{
  "name": "@openstarry-plugin/standard-function-fs"
}
```

**解析路径：**
- pnpm workspace 链接（开发环境）
- `node_modules/`（生产环境、npm install 后）

---

## 4. 插件工厂模式

每个插件套件必须 export 一个工厂函数（factory function）：

```typescript
// 具名 export
export function createMyPlugin(): IPlugin { ... }

// 或 default export
export default function createMyPlugin(): IPlugin { ... }
```

Runner 会尝试 `mod.default ?? mod.createPlugin` 来取得工厂函数。

工厂函数不接受外部参数。插件的配置通过 `IPluginContext.config`（来自 `agent.json` 的 `plugins[].config`）在 factory 阶段注入。

---

## 5. 历史沿革

| 版本 | 载入方式 |
|------|---------|
| Plan01 | `BUILTIN_FACTORIES` 写死在 CLI 中 + 短名 |
| Plan02 | 新增 `ref.path` 动态载入（三层策略） |
| Plan03 Phase C | 移除 `BUILTIN_FACTORIES`，全面动态载入，统一完整套件名 |
