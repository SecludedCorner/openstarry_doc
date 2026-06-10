# 11. 第三方插件安裝指南 (Third-Party Plugin Installation)

本文件定義了 OpenStarry 系統如何從官方倉庫 (`openstarry_plugin`) 或其他來源安裝插件。

## 1. 支援的安裝來源 (Installation Sources)

OpenStarry 的安裝器 (`openstarry plugin add`) 支援以下三種來源：

1.  **本地路徑 (Local Path):** 您已經下載或解壓好的插件資料夾。
2.  **Git 倉庫 (Git Repository):** 直接指向 GitHub/GitLab 的 URL。
3.  **NPM Registry:** 發布到 npm 的標準包。

## 1.1 標準安裝流程：倉庫同步 (Repository Sync)

這是 OpenStarry **最推薦** 的插件管理方式。系統的標準插件庫並非逐一下載，而是透過與官方生態系倉庫同步來獲取。

### 操作指令
```bash
# 同步官方倉庫 (需先將 openstarry_plugin clone 到本地)
$ openstarry plugin sync <path-to-openstarry_plugin-repo>
```

### 執行邏輯
1.  **來源掃描：** 系統掃描指定的 `openstarry_plugin` 本地倉庫目錄。
2.  **增量更新：** 比對 `~/.openstarry/plugins/` 中的版本，將新版插件複製過去。
3.  **依賴檢查 (Install-time Check):** 
    *   系統會立即檢查新安裝插件的 `dependencies` 列表。
    *   **若有缺失：** CLI 會立即輸出黃色警告：
        > "⚠️ Installed [Workflow], but it requires [Skill]. Please install [Skill] to enable full functionality."
4.  **自動註冊：** 協調層 (Daemon) 自動索引所有同步進來的插件。

---

## 2. 開發者模式：批量註冊 (`add --all`)

如果您開發了一個包含多個插件的「插件包 (Plugin Bundle)」或私有倉庫，可以使用此指令一次性註冊。

### 操作指令
```bash
# 批量註冊當前目錄下的所有插件
$ cd my-private-plugins-repo
$ openstarry plugin add --all .
```

### 執行邏輯
1.  **遞歸掃描：** 系統遞歸掃描目標目錄，尋找所有含有 `plugin.json` 或 `package.json` (含 `openstarry` 欄位) 的子目錄。
2.  **原地鏈接 (Symlink) 或 複製：**
    *   預設情況下，系統會嘗試建立 **符號連結 (Symlink)** 到 `~/.openstarry/plugins/`，方便開發調試。
    *   若加上 `--copy` 參數，則執行物理複製。
3.  **批量索引與檢查：** Daemon 更新索引並對整批插件執行依賴完整性檢查。

## 2.1 CLI 安裝流程 (The CLI Way)

這是推薦的安裝方式，系統會自動處理依賴與路徑。

### A. 安裝本地資料夾
假設您下載了一堆插件在 `C:\Downloads\my-plugins\`：

```bash
# 語法：openstarry plugin add <path>
$ openstarry plugin add C:\Downloads\my-plugins\super-search-tool
```

**系統執行步驟：**
1.  **驗證:** 檢查目標資料夾是否有有效 `package.json` 和 `plugin.json` (或 `openstarry` 字段)。
2.  **複製:** 將資料夾複製到系統插件目錄 `~/.openstarry/plugins/super-search-tool`。
3.  **依賴:** 進入該目錄執行 `npm install --production` 安裝其執行所需的庫。
4.  **構建:** 如果檢測到 `tsconfig.json` 且沒有 `dist/`，嘗試執行 `npm run build`。
5.  **註冊:** 通知 Daemon 刷新插件列表。

### B. 直接從 Git 安裝

```bash
$ openstarry plugin add https://github.com/username/weather-plugin.git
```

**系統執行步驟：**
1.  **Clone:** 將倉庫 clone 到臨時目錄。
2.  **處理:** 執行上述的依賴安裝與構建步驟。
3.  **移動:** 將產物移入系統目錄。

---

## 3. 安裝流程 (The Manual Way)

針對單獨的 NPM 包或 Git URL，仍保留傳統安裝方式。

```bash
$ openstarry plugin add https://github.com/user/weather-tool.git
```
*   **即時檢查：** 安裝完成後，CLI 同樣會檢查並報告任何缺失的依賴。

如果您希望完全手動控制（例如批量複製），請遵循以下規範：

### 目標路徑
將插件資料夾放置於：`~/.openstarry/plugins/<plugin-id>/`

### 必須執行的操作
僅僅複製文件是不夠的，因為大多數插件依賴 Node.js 模組。

1.  **複製文件:** 
    將插件源碼複製到目標路徑。
    
2.  **安裝依賴 (關鍵一步):** 
    您必須進入該目錄並安裝依賴，否則 Agent 執行時會報錯 (`Module not found`)。
    ```bash
    cd ~/.openstarry/plugins/<plugin-id>
    npm install --production
    ```

3.  **編譯 (如果是 TypeScript):** 
    如果插件是 TS 寫的且沒有附帶 `dist`，您需要編譯它。
    ```bash
    npm run build
    ```

4.  **重啟 Daemon:** 
    ```bash
    openstarry daemon restart
    ```

---

## 4. 插件權限與隔離

無論是 Sync 還是 Add，當新插件首次被 Agent **啟用 (Enabled)** 時，系統都會根據 `agent.json` 的權限設定進行攔截檢查，確保沒有未授權的高風險操作。

安裝第三方插件時，系統會掃描其 `manifest` 聲明的權限。

*   **沙盒機制:** 默認情況下，第三方插件運行在受限環境中。
*   **權限提示:** 當您執行 `openstarry plugin add` 時，CLI 會列出該插件請求的權限（如：`network`, `fs:write`），並要求您確認。
*   **手動審查:** 對於手動安裝的插件，首次運行時 Daemon 會攔截並要求管理員授權。

```