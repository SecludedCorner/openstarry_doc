# 16. 插件注册表与分发机制 (Plugin Registry and Distribution)

OpenStarry 采用 **「去中心化 Git 仓库」** 作为核心分发模型，而非依赖单一的中央服务器。

## 1. 插件增加支持

### 1.1 官方生态系 (The Ecosystem Repo)
*   **仓库：** `https://github.com/SecludedCorner/openstarry_plugin`
*   **角色：** 这是官方维护的「插件大卖场」。
*   **机制：** 所有的标准插件 (Standard) 与审核过的社区插件都存放在此 Monorepo 中。
*   **获取：** 用户将此仓库 Clone 下来，通过 `openstarry plugin sync` 保持更新。

### 1.2 私有/第三方分发
*   **机制：** 开发者可以建立自己的 Git 仓库（结构需符合 `openstarry_plugin` 规范）。
*   **获取：** 用户 Clone 该仓库后，同样使用 `openstarry plugin add --all` 进行批量安装。

### 1.3 基于 NPM 的分发 (推荐)
*   **机制：** 插件本身是一个标准的 NPM Package。
*   **命名规范：** `@scope/openstarry-plugin-<name>` 或 `openstarry-plugin-<name>`。
*   **优势：** 利用现有的 NPM 基础设施 (CDN, 版本管理, 依赖解决)。
*   **安装：** `openstarry plugin add openstarry-plugin-weather`。

### 1.4 基于 Git 的分发 (去中心化)
*   **机制：** 直接指向一个 Git Repository URL。
*   **场景：** 企业内部私有插件、开发中插件。
*   **安装：** `openstarry plugin add git+https://github.com/user/my-plugin.git`。

## 2. 插件注册表 (The Registry)

虽然分发是去中心化的，但为了便于发现，我们维护一个**元数据注册表 (Metadata Registry)**。

### 结构
Daemon 维护一个内存数据库 (`~/.openstarry/registry.db`)，记录了：
*   **Plugin ID:** `standard-function-stdio`
*   **Source:** `/Users/me/openstarry_plugin/plugins/standard-function-stdio`
*   **Capabilities:** `[stdio, cli-ui]`
*   **Installed At:** `2026-02-02 10:00:00`
```json
{
  "plugins": {
    "openstarry-plugin-fs": {
      "type": "tool",
      "description": "Safe file system operations",
      "versions": {
        "1.0.0": {
          "url": "npm:openstarry-plugin-fs@1.0.0",
          "checksum": "sha256:..."
        }
      }
    }
  }
}
```
### 查询流程
当 Agent 启动并请求 `standard/interaction` 时，Core 向 Daemon 查询 Local Registry，Daemon 返回该插件在硬盘上的物理路径。

### 2.2 提交流程
1.  开发者发布插件到 NPM。
2.  开发者向 `openstarry/registry` 提交 Pull Request，新增插件信息。
3.  CI 自动验证插件是否符合 `manifest.json` 规范。
4.  合并后，`openstarry` CLI 工具即可搜索到该插件。

## 3. 安装与依赖管理

### 3.1 插件清单 (Agent Manifest)
每个 Agent 实例目录下的 `agent.json` 定义了其依赖：

```json
{
  "name": "coding-assistant",
  "plugins": {
    "openstarry-plugin-fs": "^1.2.0",
    "openstarry-plugin-gemini": "latest"
  }
}
```

### 3.2 安装过程 (`openstarry plugin add`)
当用户在 Agent 目录执行 `openstarry plugin add` 时：
1.  **读取** `agent.json`。
2.  **解析** 依赖来源 (NPM 或 Git)。
3.  **下载** 插件包至 `~/.openstarry/plugins/` (全局缓存) 或 `./node_modules`。
4.  **验证** 插件签名 (见安全章节)。
5.  **生成** 锁定文件 `openstarry-lock.json` 以确保环境可重现。

## 4. 插件版本控制

*   **官方插件：** 跟随 `openstarry_plugin` 仓库的 Git Tag 版本。
*   **锁定机制：** 项目的 `openstarry-lock.json` 会记录当前使用的插件版本 (Git Commit Hash)，确保在不同机器上执行的一致性。

遵循 **Semantic Versioning (SemVer)**。
*   **Major:** 破坏性变更 (如修改了 Tool 的参数结构，导致旧的 Prompt 无法正确调用)。
*   **Minor:** 新增功能 (如新增了一个 Tool)。
*   **Patch:** Bug 修复。

Agent Core 会在加载时检查插件的 `engines` 字段，确保插件兼容当前运行的 Core 版本。
    *   **Major:** 破坏性变更 (如修改了 Tool 的参数结构，导致旧的 Prompt 无法正确调用)。
*   **Minor:** 新增功能 (如新增了一个 Tool)。
*   **Patch:** Bug 修复。

Agent Core 会在加载时检查插件的 `engines` 字段，确保插件兼容当前运行的 Core 版本。
