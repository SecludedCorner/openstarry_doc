# 10. Build & Distribution Strategy

This document defines the "Detached Build" specification for OpenStarry. The primary objective is to maintain **absolute purity of the Agent Core and the coordination layer**, ensuring the kernel contains zero plugin code and achieving physical separation upon installation.

## 1. The Purity Principle

*   **Zero Bundling**: The compiled artifacts of `packages/core` are **strictly forbidden** from containing any code from the `plugins/` directory. The kernel depends solely on the interface definitions within `packages/sdk`.
*   **Dynamic Loading**: The kernel must dynamically load `.js` files from external paths at runtime via the `PluginLoader`.

## 2. Build Artifact Layout

Upon executing `pnpm build`, the `dist/` directory should present the following structure:

```text
dist/
├── bin/
│   └── openstarry-core.js  # [Absolutely Pure] Kernel executable
├── lib/
│   ├── sdk.js              # SDK library
│   └── shared.js           # Shared utility library
└── assets/
    └── standard-plugins/    # [Separately Stored] Standard plugins for distribution
        ├── tool-fs/        # Compiled Plugin A
        └── listener-stdio/ # Compiled Plugin B
```

## 3. Installation Logic

The installer (or installation script) must perform the following tasks during execution to establish the environment for the "Digital Species":

### A. System Bootstrapping
1.  Copy `bin/openstarry-core` to the system execution path (e.g., `/usr/local/bin` or `%ProgramFiles%`).
2.  Initialize the user's working directory: `~/.openstarry/` (Linux/macOS) or `%USERPROFILE%\.openstarry` (Windows).

### B. Plugin Relocation
Move or copy all plugins from `assets/standard-plugins/*` into the global plugin directory:
*   **Target Path:** `~/.openstarry/plugins/`

## 4. Runtime Discovery Mechanism

When the Agent Core starts, it **should not** search for plugins within its installation directory. Instead, it must scan system paths based on the following priorities:

1.  **Project Directory**: `./.openstarry/plugins/` (if present).
2.  **System-wide Directory**: `~/.openstarry/plugins/` (the location where the installer relocated the plugins).

## 5. Developer Sync Command

For development convenience, a `pnpm run plugins:sync` command is provided to simulate the installation logic above:
*   **Logic**: Automatically compiles all packages in `plugins/` and establishes symlinks directly to `~/.openstarry/plugins/`, achieving "install as you develop."
