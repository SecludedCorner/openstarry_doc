# 16. 插件註冊表與分發機制 (Plugin Registry and Distribution)

OpenStarry 採用 **「去中心化 Git 倉庫」** 作為核心分發模型，而非依賴單一的中央伺服器。

## 1. 插建增加支援

### 1.1 官方生態系 (The Ecosystem Repo)
*   **倉庫：** `https://github.com/SecludedCorner/openstarry_plugin`
*   **角色：** 這是官方維護的「插件大賣場」。
*   **機制：** 所有的標準插件 (Standard) 與審核過的社群插件都存放在此 Monorepo 中。
*   **獲取：** 用戶將此倉庫 Clone 下來，通過 `openstarry plugin sync` 保持更新。

### 1.2 私有/第三方分發
*   **機制：** 開發者可以建立自己的 Git 倉庫（結構需符合 `openstarry_plugin` 規範）。
*   **獲取：** 用戶 Clone 該倉庫後，同樣使用 `openstarry plugin add --all` 進行批量安裝。

### 1.3 基於 NPM 的分發 (推薦)
*   **機制：** 插件本身是一個標準的 NPM Package。
*   **命名規範：** `@scope/openstarry-plugin-<name>` 或 `openstarry-plugin-<name>`。
*   **優勢：** 利用現有的 NPM 基礎設施 (CDN, 版本管理, 依賴解決)。
*   **安裝：** `openstarry plugin add openstarry-plugin-weather`。

### 1.4 基於 Git 的分發 (去中心化)
*   **機制：** 直接指向一個 Git Repository URL。
*   **場景：** 企業內部私有插件、開發中插件。
*   **安裝：** `openstarry plugin add git+https://github.com/user/my-plugin.git`。

## 2. 插件註冊表 (The Registry)

雖然分發是去中心化的，但為了便於發現，我們維護一個**元數據註冊表 (Metadata Registry)**。

### 結構
Daemon 維護一個內存資料庫 (`~/.openstarry/registry.db`)，記錄了：
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
### 查詢流程
當 Agent 啟動並請求 `standard/interaction` 時，Core 向 Daemon 查詢 Local Registry，Daemon 返回該插件在硬碟上的物理路徑。

### 2.2 提交流程
1.  開發者發布插件到 NPM。
2.  開發者向 `openstarry/registry` 提交 Pull Request，新增插件信息。
3.  CI 自動驗證插件是否符合 `manifest.json` 規範。
4.  合併後，`openstarry` CLI 工具即可搜索到該插件。

## 3. 安裝與依賴管理

### 3.1 插件清單 (Agent Manifest)
每個 Agent 實例目錄下的 `agent.json` 定義了其依賴：

```json
{
  "name": "coding-assistant",
  "plugins": {
    "openstarry-plugin-fs": "^1.2.0",
    "openstarry-plugin-gemini": "latest"
  }
}
```

### 3.2 安裝過程 (`openstarry plugin add`)
當用戶在 Agent 目錄執行 `openstarry plugin add` 時：
1.  **讀取** `agent.json`。
2.  **解析** 依賴來源 (NPM 或 Git)。
3.  **下載** 插件包至 `~/.openstarry/plugins/` (全局緩存) 或 `./node_modules`。
4.  **驗證** 插件簽名 (見安全章節)。
5.  **生成** 鎖定文件 `openstarry-lock.json` 以確保環境可重現。

## 4. 插件版本控制

*   **官方插件：** 跟隨 `openstarry_plugin` 倉庫的 Git Tag 版本。
*   **鎖定機制：** 專案的 `openstarry-lock.json` 會記錄當前使用的插件版本 (Git Commit Hash)，確保在不同機器上執行的一致性。

遵循 **Semantic Versioning (SemVer)**。
*   **Major:** 破壞性變更 (如修改了 Tool 的參數結構，導致舊的 Prompt 無法正確調用)。
*   **Minor:** 新增功能 (如新增了一個 Tool)。
*   **Patch:** Bug 修復。

Agent Core 會在加載時檢查插件的 `engines` 欄位，確保插件兼容當前運行的 Core 版本。