# 33. Plugin Skandha 多值設計 (Multi-value Skandha Design)

> Master 文件需求 #5: Plugin skandha 多值支援
> M-7 裁定 + Cycle 02-3 整合版

---

## 1. Master 原文

> 「型別系統會拒絕沒有 skandha 欄位的 Plugin。目前 Plugin 的 skandha 欄位不會僅僅具備某個蘊，可能受想行，都在此 plugin 裡面。」
>
> 「每個 Plugin 聲明自己的蘊歸屬，但如果此 plugin 有包含不同的蘊怎麼辦？plugin 有可能同時具備兩種或是多種的 skandha。」

---

## 2. 設計方案：Manifest 多值 + Interface 單值

### 2.1 核心原則

**兩層分離**：
- **Interface 層**：每個介面物件仍是單值 — `ITool` 的 `skandha` 永遠是 `'samskara'`
- **Manifest 層**：Plugin 聲明包含哪些蘊 — 可以是多值

這樣不破壞型別守衛，又允許 Plugin 跨蘊。

### 2.2 型別定義

```typescript
/** 五蘊 (M-1 梵文命名) */
type Skandha = 'rupa' | 'vedana' | 'samjna' | 'samskara' | 'vijnana';

/** Plugin Manifest — skandha 必填，支援多值 */
interface PluginManifest {
  readonly name: string;
  readonly version: string;
  readonly skandha: Skandha | readonly Skandha[];  // 單值或多值
  readonly description?: string;
}

/** 五蘊根介面 — 各自的 skandha 是固定的 */
interface IRupa     { readonly skandha: 'rupa'; }
interface IVedana   { readonly skandha: 'vedana'; }
interface ISamjna   { readonly skandha: 'samjna'; }
interface ISamskara { readonly skandha: 'samskara'; }
interface IVijnana  { readonly skandha: 'vijnana'; }
```

### 2.3 Manifest 層型別守衛

```typescript
/** 檢查 Plugin 是否包含某個蘊 */
function hasSkandha(manifest: PluginManifest, skandha: Skandha): boolean {
  return Array.isArray(manifest.skandha)
    ? manifest.skandha.includes(skandha)
    : manifest.skandha === skandha;
}

/** 物件層 isSkandha — 不需改動 */
function isSkandha<S extends Skandha>(
  obj: unknown, skandha: S
): obj is { skandha: S } {
  return typeof obj === 'object' && obj !== null
    && 'skandha' in obj && (obj as any).skandha === skandha;
}
```

---

## 3. 一致性檢查

Plugin 載入時驗證 declared vs actual：

```typescript
function validatePluginSkandha(
  manifest: PluginManifest,
  hooks: PluginHooks
): void {
  const declared = Array.isArray(manifest.skandha)
    ? manifest.skandha
    : [manifest.skandha];
  const actual = inferSkandha(hooks);

  // 實際提供的蘊必須是聲明蘊的子集
  for (const s of actual) {
    if (!declared.includes(s)) {
      throw new Error(
        `Plugin ${manifest.name} provides ${s} hooks ` +
        `but does not declare skandha: ${s}`
      );
    }
  }

  // 聲明的蘊應至少有一個 hook（警告，不阻斷）
  for (const s of declared) {
    if (!actual.includes(s)) {
      logger.warn(
        `Plugin ${manifest.name} declares skandha: ${s} ` +
        `but provides no corresponding hooks`
      );
    }
  }
}

function inferSkandha(hooks: PluginHooks): Skandha[] {
  const result: Skandha[] = [];
  if (hooks.listeners?.length || hooks.ui?.length) result.push('rupa');
  // vedana: future VedanaPlugin hooks
  if (hooks.providers?.length) result.push('samjna');
  if (hooks.tools?.length) result.push('samskara');
  if (hooks.guides?.length) result.push('vijnana');
  return result;
}
```

---

## 4. 22 個 Plugin 的 skandha 值

| # | Plugin | 現行 | 更新後 | 多值? |
|---|--------|------|--------|-------|
| 1 | devtools | samskara | `['samskara']` | |
| 2 | guide-character-init | vijnana | `['vijnana']` | |
| 3 | http-static | rupa | `['rupa']` | |
| 4 | mcp-client | rupa | `['rupa']` | |
| 5 | mcp-server | rupa | `['rupa']` | |
| 6 | provider-chatgpt | samjna | `['samjna']` | |
| 7 | provider-claude | samjna | `['samjna']` | |
| 8 | provider-gemini | samjna | `['samjna']` | |
| 9 | provider-gemini-oauth | samjna | `['samjna']` | |
| 10 | provider-lmstudio | samjna | `['samjna']` | |
| 11 | provider-local-llama | samjna | `['samjna']` | |
| 12 | standard-core-commands | samskara | `['samskara']` | |
| 13 | standard-function-fs | samskara | `['samskara']` | |
| 14 | **standard-function-skill** | samskara | **`['samskara', 'vijnana']`** | **Yes** |
| 15 | standard-function-stdio | samskara | `['samskara']` | |
| 16 | standard-model-selector | samskara | `['samskara']` | |
| 17 | transport-http | rupa | `['rupa']` | |
| 18 | transport-websocket | rupa | `['rupa']` | |
| 19 | tui-dashboard | rupa | `['rupa']` | |
| 20 | web-ui | rupa | `['rupa']` | |
| 21 | workflow-engine | samskara | `['samskara']` | |
| 22 | marketplace | samskara | `['samskara']` | |

> 目前僅 **standard-function-skill** 為多值（同時提供 Tool + Guide）。
> 未來 VedanaPlugin 可能為 `['vedana', 'samjna']`（受蘊感測 + 想蘊觀察能力）。

---

## 5. 遷移策略

### 5.1 Manifest 欄位遷移

所有 22 個 Plugin 的 manifest 需從字串改為陣列：

```typescript
// 遷移前
export const manifest: PluginManifest = {
  name: '@openstarry-plugin/devtools',
  skandha: 'samskara',  // string
};

// 遷移後
export const manifest: PluginManifest = {
  name: '@openstarry-plugin/devtools',
  skandha: ['samskara'],  // array
};
```

### 5.2 SDK 型別更新

```typescript
// packages/sdk/src/plugin.ts
export interface PluginManifest {
  readonly name: string;
  readonly version: string;
  readonly skandha: Skandha | readonly Skandha[];  // 允許多值
  readonly description?: string;
}
```

### 5.3 PluginLoader 更新

```typescript
// packages/core/src/plugin-loader.ts
// 載入時進行一致性檢查
private validatePlugin(plugin: IPlugin, hooks: PluginHooks): void {
  validatePluginSkandha(plugin.manifest, hooks);
}
```

### 5.4 @skandha JSDoc 標記

```typescript
/**
 * @openstarry-plugin/standard-function-skill
 * @skandha samskara (行蘊·工具), vijnana (識蘊·引導)
 */
```

---

## 6. 向後相容性

- `skandha: 'samskara'`（字串）仍然有效 — 型別是 `Skandha | Skandha[]`
- `isSkandha()` 不需改動 — 它檢查的是介面物件，不是 manifest
- 新增 `hasSkandha()` 用於 manifest 查詢
- 載入時一致性檢查是額外的安全網，不破壞現有功能

---

*Master 文件需求 #5 完成。M-7 裁定方案 B+D 混合策略。*
