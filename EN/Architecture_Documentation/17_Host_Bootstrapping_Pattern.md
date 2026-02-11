# 17. Host Bootstrapping Pattern

This document resolves a classic architectural paradox: **"If the Agent Core is absolutely pure and lacks I/O capabilities, how does it read configurations from the disk and load plugins?"**

The answer is: **It doesn't. These are the responsibilities of the "Host."**

## 1. The Problem: The Bootstrap Paradox

We require `packages/core` to be:
1.  **Headless:** Unaware of the execution environment.
2.  **Pure:** Independent of Node.js `fs` or `module` modules.
3.  **Microkernel:** All capabilities (including file reading) are provided by plugins.

But if it cannot even read a file, how can it read `agent.json` to know that it needs to load the `fs-plugin`?

## 2. The Solution: The Host Layer

We explicitly distinguish between two runtime roles in the architecture:

### Role A: Host (Host / Coordinator)
*   **Entity:** `apps/runner`, `apps/daemon`, `apps/web-server`.
*   **Privileges:** Full operating system privileges (file I/O, network requests, process management).
*   **Responsibility:** **The Pioneer**. It is responsible for preparing the environment, reading configurations, and transporting plugins from the disk into memory.

### Role B: Kernel (Core)
*   **Entity:** An instance of `packages/core`.
*   **Privileges:** Zero.
*   **Responsibility:** **The Inheritor**. It passively receives resources prepared by the Host and begins logical operations.

## 3. Detailed Sequence

### Step 1. Host Awakening
When a user executes `node apps/runner/dist/bin.js`, it is the **Host (Runner)** that starts.
At this point, the Host uses the native `fs` module to read `./agent.json`.

> **Host Perspective:** "I've read the configuration; this Agent requires `fs` and `http` plugins."

### Step 2. Physical Loading
Based on the configuration, the Host attempts two resolution strategies for each plugin: (1) a file path specified by `ref.path`, or (2) dynamic loading of a package name via `import(ref.name)`. It uses ESM's `import()` to load them into memory.

> **Host Perspective:** "I have loaded `fs-plugin.js` into RAM; it is now an object."

### Step 3. Dependency Injection
The Host instantiates the Core and passes in these **pre-loaded objects**.

```typescript
// Host Code (Pseudo-code)
const loadedPlugins = [ fsPluginObject, httpPluginObject ];
const core = new AgentCore({ plugins: loadedPlugins });
```

### Step 4. Kernel Awakening
The Core starts. It doesn't need to look for plugins on the disk because they are already in its possession. It only needs to call `plugin.initialize()`.

> **Core Perspective:** "I am awake. I have tools in my hands. I don't know where they came from, but I can use them."

## 4. Summary

**The Coordinating Layer (Host) is responsible for "Survival," while the Kernel (Core) is responsible for "Life."**

*   **What plugins to load?** -> Requires looking at folder contents -> This is the **Host's** job.
*   **How to use the plugins?** -> This is the **Core's** job.

This pattern allows us to maintain the absolute purity of the Core while resolving the issue of physical loading.