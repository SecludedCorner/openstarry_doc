# 11. 第三方插件安装指南 (Third-Party Plugin Installation)

本文档定义了 OpenStarry 系统如何从官方仓库 (`openstarry_plugin`) 或其他来源安装插件。

## 1. 支持的安装来源 (Installation Sources)

OpenStarry 的安装器 (`openstarry plugin add`) 支持以下三种来源：

1.  **本地路径 (Local Path):** 您已经下载或解压好的插件文件夹。
2.  **Git 仓库 (Git Repository):** 直接指向 GitHub/GitLab 的 URL。
3.  **NPM Registry:** 发布到 npm 的标准包。

## 1.1 标准安装流程：仓库同步 (Repository Sync)

这是 OpenStarry **最推荐** 的插件管理方式。系统的标准插件库并非逐一下载，而是通过与官方生态系统仓库同步来获取。

### 操作指令
```bash
# 同步官方仓库 (需先将 openstarry_plugin clone 到本地)
$ openstarry plugin sync <path-to-openstarry_plugin-repo>
```

### 执行逻辑
1.  **来源扫描：** 系统扫描指定的 `openstarry_plugin` 本地仓库目录。
2.  **增量更新：** 比对 `~/.openstarry/plugins/` 中的版本，将新版插件复制过去。
3.  **依赖检查 (Install-time Check):** 
    *   系统会立即检查新安装插件的 `dependencies` 列表。
    *   **若有缺失：** CLI 会立即输出黄色警告：
        > "⚠️ Installed [Workflow], but it requires [Skill]. Please install [Skill] to enable full functionality."
4.  **自动注册：** 协调层 (Daemon) 自动索引所有同步进来的插件。

---

## 2. 开发者模式：批量注册 (`add --all`)

如果您开发了一个包含多个插件的「插件包 (Plugin Bundle)」或私有仓库，可以使用此命令一次性注册。

### 操作指令
```bash
# 批量注册当前目录下的所有插件
$ cd my-private-plugins-repo
$ openstarry plugin add --all .
```

### 执行逻辑
1.  **递归扫描：** 系统递归扫描目标目录，寻找所有含有 `plugin.json` 或 `package.json` (含 `openstarry` 字段) 的子目录。
2.  **原地链接 (Symlink) 或 复制：**
    *   默认情况下，系统会尝试建立 **符号链接 (Symlink)** 到 `~/.openstarry/plugins/`，方便开发调试。
    *   若加上 `--copy` 参数，则执行物理复制。
3.  **批量索引与检查：** Daemon 更新索引并对整批插件执行依赖完整性检查。

## 2.1 CLI 安装流程 (The CLI Way)

这是推荐的安装方式，系统会自动处理依赖与路径。

### A. 安装本地文件夹
假设您下载了一堆插件在 `C:\Downloads\my-plugins`：

```bash
# 语法：openstarry plugin add <path>
$ openstarry plugin add C:\Downloads\my-plugins\super-search-tool
```

**系统执行步骤：**
1.  **验证:** 检查目标文件夹是否有有效 `package.json` 和 `plugin.json` (或 `openstarry` 字段)。
2.  **复制:** 将文件夹复制到系统插件目录 `~/.openstarry/plugins/super-search-tool`。
3.  **依赖:** 进入该目录执行 `npm install --production` 安装其执行所需的库。
4.  **构建:** 如果检测到 `tsconfig.json` 且没有 `dist/`，尝试执行 `npm run build`。
5.  **注册:** 通知 Daemon 刷新插件列表。

### B. 直接从 Git 安装

```bash
$ openstarry plugin add https://github.com/username/weather-plugin.git
```

**系统执行步骤：**
1.  **Clone:** 将仓库 clone 到临时目录。
2.  **处理:** 执行上述的依赖安装与构建步骤。
3.  **移动:** 将产物移入系统目录。

---

## 3. 安装流程 (The Manual Way)

针对单独的 NPM 包或 Git URL，仍保留传统安装方式。

```bash
$ openstarry plugin add https://github.com/user/weather-tool.git
```
*   **即时检查：** 安装完成后，CLI 同样会检查并报告任何缺失的依赖。

如果您希望完全手动控制（例如批量复制），请遵循以下规范：

### 目标路径
将插件文件夹放置于：`~/.openstarry/plugins/<plugin-id>/`

### 必须执行的操作
仅仅复制文件是不够的，因为大多数插件依赖 Node.js 模块。

1.  **复制文件:** 
    将插件源码复制到目标路径。
    
2.  **安装依赖 (关键一步):** 
    您必须进入该目录并安装依赖，否则 Agent 执行时会报错 (`Module not found`)。
    ```bash
    cd ~/.openstarry/plugins/<plugin-id>
    npm install --production
    ```

3.  **编译 (如果是 TypeScript):** 
    如果插件是 TS 写的且没有附带 `dist`，您需要编译它。
    ```bash
    npm run build
    ```

4.  **重启 Daemon:** 
    ```bash
    openstarry daemon restart
    ```

---

## 4. 插件权限与隔离

无论是 Sync 还是 Add，当新插件首次被 Agent **启用 (Enabled)** 时，系统都会根据 `agent.json` 的权限设定进行拦截检查，确保没有未授权的高风险操作。

安装第三方插件时，系统会扫描其 `manifest` 声明的权限。

*   **沙盒机制:** 默认情况下，第三方插件运行在受限环境中。
*   **权限提示:** 当您执行 `openstarry plugin add` 时，CLI 会列出该插件请求的权限（如：`network`, `fs:write`），并要求您确认。
*   **手动审查:** 对于手动安装的插件，首次运行时 Daemon 会拦截并要求管理员授权。
