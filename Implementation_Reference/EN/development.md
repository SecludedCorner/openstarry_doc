# Development Guide

## Testing

```bash
# Run all tests (core + plugins)
pnpm test

# Watch mode (re-run on file changes)
pnpm test:watch

# Microkernel purity check (ensure core has no plugin dependencies)
pnpm test:purity

# Run specific tests
pnpm test -- packages/core/src/bus/event-bus.test.ts
pnpm test -- -t "FIFO"
```

## Creating a New Plugin

### Using the Scaffolding Tool

```bash
node apps/runner/dist/bin.js create-plugin
```

Follow the interactive prompts to enter the plugin name, description, etc. The tool will generate a plugin template automatically.

### Manual Creation

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

### Plugin Structure

```
my-plugin/
├── package.json
├── tsconfig.json
├── src/
│   ├── index.ts          # Plugin entry, exports createMyPlugin()
│   └── index.test.ts     # Tests
└── dist/                 # Build output
```

### Key Conventions

- Factory pattern: export `createXxxPlugin()` → returns `IPlugin`
- `factory(ctx)` receives `IPluginContext`, returns `PluginHooks`
- Use `ctx.pushInput()` to communicate with core — **never call core APIs directly**
- Plugin package names: `@openstarry-plugin/xxx`
- vitest version must match the root project (`^4.0.18`)
- tsconfig must exclude test files: `"exclude": ["src/**/*.test.ts"]`

## Building

```bash
# Build all packages (core + plugins)
pnpm build

# Build a single package
pnpm --filter @openstarry/core build
```

## Further Reading

- `packages/sdk/README.md` — All interface definitions
- `packages/core/README.md` — Core architecture
- `apps/runner/README.md` — CLI command details
