# 01. Monorepo 顶层目录结构规范

本文档定义了 OpenStarry 项目的顶层物理布局。我们采用 **Monorepo (单一巨型仓库)** 模式，并建议使用 `pnpm` 作为包管理器，以利用其高效的 `workspace` 功能。

## 顶层目录概览

```text
openstarry/ (Core Repo)
├── apps/                # 可执行的应用程序 (Applications)
├── packages/            # 可重用的核心组件与库 (Packages/Libraries)
├── tools/               # 开发者工具与脚本 (Internal Tools)
├── .geminiignore        # 代理人操作忽略规范
├── package.json         # 根目录配置 (Workspace Root)
├── pnpm-workspace.yaml  # pnpm 虚拟工作空间配置
└── README.md            # 项目总览
```

---

## 目录详细说明

### 1. `apps/` (应用层)
这里存放可以直接运行、部署的项目。
*   **`apps/daemon/`**: Orchestrator Daemon (守护进程)。
*   **`apps/runner/`**: 纯启动引导程序 (Bootstrap Runner)，负责解析 `agent.json` 并拉起 Core。非 CLI 专属，未来可由 Daemon 或其他宿主取代。
*   **`apps/dashboard/`**: 基于 Web 的管理界面，**协调层的可视化控制台**。
*   **`apps/installer/`**: 安装与部署工具，**协调层的资源调度前端**，负责将标准插件搬运至 `~/.openstarry`。

### 2. `packages/` (核心库层)
这里存放系统的核心逻辑，以 NPM 包的形式存在。
*   **`packages/core/`**: **(核心)** 绝对纯净的执行引擎，不含任何插件逻辑。
*   **`packages/sdk/`**: 插件开发契约，定义接口与规范。
*   **`packages/shared/`**: 全局通用的工具库。

### 3. `tools/` (工程配套)
*   用于 CI/CD、代码生成、文档自动化的脚本。

---

## 生态系分层与代码隔离 (Ecosystem Separation & Code Isolation)

为了确保核心绝对纯净，我们将「系统本体」与「功能内容」在物理上完全隔离：

### 1. 系统本体 (`openstarry`)
*   **定位：** 这是唯一的 Monorepo，包含系统内核、守护进程与基础设施。
*   **原则：** **严禁包含任何具体的 Tool 或 Provider 代码。** 此仓库中不包含 `plugins/` 目录。系统运行时会动态扫描：
    *   **系统路径：** `~/.openstarry/plugins/` (由外部仓库同步而来)。
    *   **项目路径：** `<Project>/.openstarry/plugins/` (私有插件)。

### 2. 官方插件库 (`openstarry_plugin`)
*   **定位：** 这是一个独立的外部仓库，作为官方维护的标准插件来源（类似 Linux 的软件包源）。
*   **运作机制：** 
    *   它不参与 Core 的编译或构建。
    *   用户通过 `openstarry plugin sync` 命令将此仓库的内容「下载并安装」到系统路径中。

---

## 关键配置文件示例

### `pnpm-workspace.yaml`
```yaml
packages:
  - 'apps/*'
  - 'packages/*'
  # 注意：plugins 不在此 workspace 中，它们是外部依赖
```

---

## 为什么这样设计？ (设计哲学)

1.  **物理隔离 = 逻辑解耦**：将 `core` 与 `daemon` 分开，确保了我们之前提到的「Core 是无头的」这一哲学。Core 甚至不知道 Daemon 的存在，它只是一个被 Daemon 拉起来运行的库。
2.  **内核绝对纯净**：`packages/core` 在编译阶段与插件完全分离。这确保了系统核心的稳定性与可移植性。
3.  **分离式安装 (Detached Installation)**：安装过程负责环境初始化与插件搬运，而非将插件硬编码进内核。这使得 OpenStarry 的「数字物种」可以在不同的环境中拥有不同的「感官与肢体」。
4.  **单一版本管理**：作为开发者，您只需在根目录执行一次 `pnpm install`，所有子项目的依赖都会被正确处理。
    *   **物理隔离 = 逻辑解耦**：将 `core` 与 `daemon` 分开，确保了我们之前提到的「Core 是无头的」这一哲学。Core 甚至不知道 Daemon 的存在，它只是一个被 Daemon 拉起来运行的库。
2.  **内核绝对纯净**：`packages/core` 在编译阶段与插件完全分离。这确保了系统核心的稳定性与可移植性。
3.  **分离式安装 (Detached Installation)**：安装过程负责环境初始化与插件搬运，而非将插件硬编码进内核。这使得 OpenStarry 的「数字物种」可以在不同的环境中拥有不同的「感官与肢体」。
4.  **单一版本管理**：作为开发者，您只需在根目录执行一次 `pnpm install`，所有子项目的依赖都会被正确处理。
