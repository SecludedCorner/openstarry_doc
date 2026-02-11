# 10. 构建与分发策略 (Build & Distribution Strategy)

本文档定义了 OpenStarry 的「分离式构建」规范。核心目标是保持 **Agent Core 与协调层的绝对纯净**，确保内核不包含任何插件代码，并在安装时实现物理分离。

## 1. 核心纯净原则 (The Purity Principle)

*   **内核零捆绑 (Zero Bundling):** `packages/core` 的编译产物中 **严禁** 包含任何 `plugins/` 目录下的代码。内核仅依赖于 `packages/sdk` 的接口定义。
*   **动态载入:** 内核必须在运行时通过插件加载器 (PluginLoader) 从外部路径动态载入 `.js` 文件。

## 2. 构建产物布局 (Build Artifacts)

执行 `pnpm build` 后，`dist/` 目录应呈现以下结构：

```text
dist/
├── bin/
│   └── openstarry-core.js  # [绝对纯净] 内核执行档
├── lib/
│   ├── sdk.js              # SDK 库
│   └── shared.js           # 共享工具库
└── assets/
    └── standard-plugins/    # [分离存放] 待分发的标准插件
        ├── tool-fs/        # 编译后的插件 A
        └── listener-stdio/ # 编译后的插件 B
```

## 3. 安装逻辑 (Installation Logic)

安装档（或安装脚本）在执行时必须完成以下任务，以建立「数字物种」的生存环境：

### A. 系统定位 (System Bootstrapping)
1.  将 `bin/openstarry-core` 复制到系统执行路径 (如 `/usr/local/bin` 或 `%ProgramFiles%`)。
2.  初始化用户工作目录：`~/.openstarry/` (Linux/macOS) 或 `%USERPROFILE%\.openstarry` (Windows)。

### B. 插件搬运 (Plugin Relocation)
将 `assets/standard-plugins/*` 中的所有插件 **移动或复制** 到全局插件目录中：
*   **目标路径:** `~/.openstarry/plugins/`

## 4. 运行时发现机制 (Runtime Discovery)

当 Agent Core 启动时，它**不应**从其安装目录寻找插件，而是根据以下优先级扫描系统路径：

1.  **项目目录:** `./.openstarry/plugins/` (若存在)
2.  **系统全局目录:** `~/.openstarry/plugins/` (这是由安装程序搬运过去的位置)

## 5. 开发者同步指令 (Dev Sync)

为了开发便利，我们提供 `pnpm run plugins:sync` 指令，模拟上述安装逻辑：
*   **逻辑:** 自动编译所有 `plugins/` 并将产物直接建立软链接 (Symlink) 到 `~/.openstarry/plugins/`，实现「开发即安装」。
