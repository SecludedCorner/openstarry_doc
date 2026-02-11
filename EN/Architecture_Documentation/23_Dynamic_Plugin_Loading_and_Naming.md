# 23. Dynamic Plugin Loading & Naming Convention

This document describes OpenStarry's plugin resolution mechanism and package naming convention.

---

## 1. Design Principles

The Runner (`apps/runner`) is a **pure launcher**; it does not recognize any specific plugins. All plugins—including Providers, Tools, Listeners, and Guides—are configured via `agent.json` and dynamically loaded at runtime.

This means:
- The Runner's `package.json` **does not depend** on any `@openstarry-plugin/*` packages.
- The Runner's source code **does not import** any plugin modules.
- Adding a plugin does not require modifying any Runner code.

---

## 2. Package Naming Convention

All official plugins use the `@openstarry-plugin/` scope uniformly:

```
@openstarry-plugin/{plugin-name}
```

| Full Package Name | Type | Description |
|-----------|------|------|
| `@openstarry-plugin/provider-gemini-oauth` | Provider (Perception) | Gemini LLM + OAuth Authentication |
| `@openstarry-plugin/standard-function-fs` | Tool (Volition) | File System Operations |
| `@openstarry-plugin/standard-function-stdio` | Listener + Guide (Sensation + Consciousness) | CLI Terminal I/O + Default Persona |
| `@openstarry-plugin/standard-function-skill` | Guide (Consciousness) | Markdown Skill File Loading |

### Usage in agent.json

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

## 3. Plugin Resolution Strategy (Two-Tier)

The Runner's `resolvePlugins()` attempts the following for each plugin entry in `agent.json`, in order:

### Strategy 1: File Path Loading

When `ref.path` has a value, it directly `import()`s that file path.

```json
{
  "name": "my-custom-plugin",
  "path": "../my-plugins/custom-tool/dist/index.js"
}
```

**Applicable Scenarios:**
- Plugins in local development (not yet published to npm).
- Third-party plugins not in the pnpm workspace.
- Single `.js` plugin files provided by others.

### Strategy 2: Dynamic Package Name Loading

Attempts `import(ref.name)`, allowing the Node.js module resolution mechanism to find the package.

```json
{
  "name": "@openstarry-plugin/standard-function-fs"
}
```

**Resolution Paths:**
- pnpm workspace links (development environment).
- `node_modules/` (production environment, after npm install).

---

## 4. Plugin Factory Pattern

Each plugin package must export a factory function:

```typescript
// Named export
export function createMyPlugin(): IPlugin { ... }

// or default export
export default function createMyPlugin(): IPlugin { ... }
```

The Runner attempts `mod.default ?? mod.createPlugin` to obtain the factory function.

Factory functions do not accept external parameters. Plugin configuration is injected during the factory phase via `IPluginContext.config` (sourced from `plugins[].config` in `agent.json`).

---

## 5. Historical Context

| Version | Loading Method |
|------|---------|
| Plan01 | `BUILTIN_FACTORIES` hardcoded in CLI + Short Names |
| Plan02 | Added `ref.path` dynamic loading (Three-tier strategy) |
| Plan03 Phase C | Removed `BUILTIN_FACTORIES`, fully dynamic loading, unified full package names |
