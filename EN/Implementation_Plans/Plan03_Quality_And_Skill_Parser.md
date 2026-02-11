# OpenStarry Implementation Plan 03 — Quality Reinforcement & Architectural Purification

> **Status**: ✅ Completed (2026-02-05)

## Background

Plan01 completed the MVP skeleton, and Plan02 completed the event-driven architecture and safety circuit breakers.
The testing team confirmed all tests passed on 2026-02-05 and recommended the next steps.
Based on the gap analysis of the design documents, this plan focuses on **quality upgrades and architectural purification from v0.1 to v0.2**.

---

## Objectives

1. **Quality Reinforcement**: Make the existing system more robust, avoiding crashes due to configuration errors, tool freezes, and debugging difficulties.
2. **Capability Expansion**: Implement Markdown skill parsing, allowing Agents to dynamically load personas and behavioral guidelines.
3. **Architectural Purification**: Fully decouple plugins from the Runner, enable dynamic loading for all plugins, and unify full package names.

---

## Phase A: Quality Reinforcement (v0.2 Foundation) ✅ Completed

### A1. agent.json Zod Runtime Validation ✅

- Path: `packages/shared/src/utils/config-schema.ts`
- Defined `AgentConfigSchema` using Zod, covering all configuration fields.
- Used the schema for validation within `loadConfig()` in `apps/runner/src/bin.ts`.
- Outputs clear error messages upon validation failure.

### A2. Tool Call Timeout ✅

- Path: `executeTool()` in `packages/core/src/execution/loop.ts`.
- Wrapped with `Promise.race([tool.execute(...), timeoutPromise])`.
- Timeout value retrieved from `config.policy.toolTimeout` (default 30000ms).

### A3. TraceID Mechanism ✅

- Generates `traceId` at the start of `processEvent()`.
- Injected into the logger and event payload to string together a complete processing cycle.

### A4. CI Purity Check ✅

- `scripts/check-purity.sh` + `pnpm test:purity`.
- Ensures `core` does not reference `plugin/apps`, and `sdk` does not reference `core/shared`.

---

## Phase B: Capability Expansion ✅ Completed

### B1. standard-function-skill Plugin ✅

- Path: `openstarry_plugin/standard-function-skill/`.
- Reads `.md` skill files and parses YAML Frontmatter + Markdown Body.
- Registered as a Guide plugin in the GuideRegistry.
- **Dynamic Loading**: Not hardcoded in the Runner; configured via `agent.json`:
  ```json
  {
    "name": "@openstarry-plugin/standard-function-skill",
    "config": { "skillPath": "./skills/my-agent.md" }
  }
  ```
- Example skill file located at `openstarry_plugin/standard-function-skill/examples/coder.md`.
- 7 unit tests cover frontmatter parsing.

### B2. system_prompt External File Reference ✅

- Added `guideFile?: string` field to `IAgentConfig`.
- AgentCore reads the external `.md` file upon startup and registers it as a FileGuide.
- Coexistence of two methods: `guide` (Guide ID) or `guideFile` (file path).

---

## Phase C: Architectural Purification ✅ Completed

### C1. IPluginContext.pushInput ✅

**Problem:** The stdio plugin relied on an `onInput` callback injected by the host to push user input. This forced the CLI to recognize the stdio plugin and perform special handling, violating the goal of full dynamic loading.

**Solution:**
- Added `pushInput(event: InputEvent)` method to the SDK's `IPluginContext`.
- The Core's `getPluginContext()` automatically injects `pushInput → core.pushInput`.
- All Listener plugins push input via the standardized context, removing dependency on host callbacks.

### C2. Removal of BUILTIN_FACTORIES ✅

**Problem:** The Runner's `BUILTIN_FACTORIES` hardcoded imports for three plugins, requiring modifications to the Runner whenever a new plugin was added.

**Solution:**
- Completely removed `BUILTIN_FACTORIES` and all `@openstarry-plugin/*` imports.
- Plugin resolution now has only two tiers:
  1. `ref.path` → Direct loading from file path.
  2. `import(ref.name)` → Dynamic loading from workspace / node_modules using full package names.
- The Runner's `package.json` no longer depends on any plugin packages.

### C3. Unified Full Package Names ✅

- All plugins in `agent.json` use full package names: `@openstarry-plugin/xxx`.
- The plugins list in `defaultConfig` also uses full package names.
- Eliminated the mix of short names and full names.

### C4. apps/cli → apps/runner ✅

- Renamed to `apps/runner`, with the package name changed to `@openstarry/runner`.
- Reflects its role as a "pure launcher": read config → create core → dynamically load plugins → start.
- Does not recognize any specific plugins.

---

## Implementation Sequence (All Completed)

```
A1 agent.json Zod Validation          ✅
A2 Tool Call Timeout                 ✅
A3 TraceID Mechanism                 ✅
A4 CI Purity Check                   ✅
B1 standard-function-skill           ✅
B2 system_prompt External Reference  ✅
C1 IPluginContext.pushInput          ✅
C2 Remove BUILTIN_FACTORIES          ✅
C3 Unified Full Package Names        ✅
C4 apps/cli → apps/runner            ✅
```

---

## Verification Results

1. `pnpm build` — ✅ All 8 workspace projects compiled successfully.
2. `pnpm test` — ✅ 56 tests passed (across 7 test files).
3. `pnpm test:purity` — ✅ Purity check passed.
4. Zero Plugin Dependency for Runner — ✅ `apps/runner/package.json` depends only on core/sdk/shared.
