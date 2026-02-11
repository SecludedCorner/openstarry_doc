# 01. Monorepo Top-Level Structure Specification

This document defines the top-level physical layout of the OpenStarry project. We adopt the **Monorepo (Single Giant Repository)** pattern and recommend using `pnpm` as the package manager to leverage its highly efficient `workspace` functionality.

## Top-Level Directory Overview

```text
openstarry/ (Core Repo)
├── apps/                # Executable applications
├── packages/            # Reusable core components and libraries
├── tools/               # Developer tools and internal scripts
├── .geminiignore        # Guidelines for Agent operations
├── package.json         # Workspace Root configuration
├── pnpm-workspace.yaml  # pnpm virtual workspace configuration
└── README.md            # Project overview
```

---

## Detailed Directory Description

### 1. `apps/` (Application Layer)
This directory houses projects that can be directly executed or deployed.
*   **`apps/daemon/`**: The Orchestrator Daemon (background process).
*   **`apps/runner/`**: A pure bootstrap runner responsible for parsing `agent.json` and launching the Core. It is not limited to the CLI; it can be replaced by the Daemon or other hosts in the future.
*   **`apps/dashboard/`**: A Web-based management interface—the **visual console for the coordination layer**.
*   **`apps/installer/`**: Installation and deployment tools—the **resource scheduling frontend for the coordination layer**, responsible for transporting standard plugins to `~/.openstarry`.

### 2. `packages/` (Core Library Layer)
This directory contains the system's core logic, organized as NPM packages.
*   **`packages/core/`**: **(Core)** An absolutely pure execution engine, free of any plugin logic.
*   **`packages/sdk/`**: The plugin development contract, defining interfaces and specifications.
*   **`packages/shared/`**: A library of globally common utilities.

### 3. `tools/` (Engineering Support)
*   Scripts used for CI/CD, code generation, and documentation automation.

---

## Ecosystem Layering & Code Isolation

To ensure the Core remains absolutely pure, we physically separate the "System Body" from the "Functional Content":

### 1. The System Body (`openstarry`)
*   **Positioning:** This is the primary Monorepo containing the system kernel, daemon, and infrastructure.
*   **Principle:** **Strictly forbidden from containing any specific Tool or Provider code.** This repository does not include a `plugins/` directory. The system dynamically scans the following at runtime:
    *   **System Path:** `~/.openstarry/plugins/` (synchronized from external repositories).
    *   **Project Path:** `<Project>/.openstarry/plugins/` (private plugins).

### 2. Official Plugin Library (`openstarry_plugin`)
*   **Positioning:** An independent external repository serving as the official source for standard plugins (similar to a Linux package repository).
*   **Operational Mechanism:**
    *   It does not participate in the compilation or building of the Core.
    *   Users use the `openstarry plugin sync` command to "download and install" content from this repository into the system path.

---

## Key Configuration File Example

### `pnpm-workspace.yaml`
```yaml
packages:
  - 'apps/*'
  - 'packages/*'
  # Note: plugins are not in this workspace; they are external dependencies
```

---

## Why this design? (Design Philosophy)

1.  **Physical Isolation = Logical Decoupling**: Separating `core` from `daemon` ensures the "Headless Core" philosophy. The Core is unaware of the Daemon's existence; it is simply a library launched by the Daemon.
2.  **Absolutely Pure Kernel**: `packages/core` is completely isolated from plugins during the compilation phase. This ensures the stability and portability of the system core.
3.  **Detached Installation**: The installation process handles environment initialization and plugin transport rather than hardcoding plugins into the kernel. This allows OpenStarry's "Digital Species" to possess different "senses and limbs" in different environments.
4.  **Single Version Management**: As a developer, you only need to run `pnpm install` once in the root directory, and dependencies for all sub-projects will be handled correctly.
