# 01. Monorepo 頂層目錄結構規範

本文件定義了 OpenStarry 專案的頂層物理佈局。我們採用 **Monorepo (單一巨型倉庫)** 模式，並建議使用 `pnpm` 作為包管理器，以利用其高效的 `workspace` 功能。

## 頂層目錄概覽

```text
openstarry/ (Core Repo)
├── apps/                # 可執行的應用程式 (Applications)
├── packages/            # 可重用的核心組件與庫 (Packages/Libraries)
├── tools/               # 開發者工具與腳本 (Internal Tools)
├── .geminiignore        # 代理人操作忽略規範
├── package.json         # 根目錄配置 (Workspace Root)
├── pnpm-workspace.yaml  # pnpm 虛擬工作空間配置
└── README.md            # 專案總覽
```

---

## 目錄詳細說明

### 1. `apps/` (應用層)
這裡存放可以直接運行、部署的專案。
*   **`apps/daemon/`**: Orchestrator Daemon (守護進程)。
*   **`apps/runner/`**: 純啟動引導程式 (Bootstrap Runner)，負責解析 `agent.json` 並拉起 Core。非 CLI 專屬，未來可由 Daemon 或其他宿主取代。
*   **`apps/dashboard/`**: 基於 Web 的管理介面，**協調層的可視化控制台**。
*   **`apps/installer/`**: 安裝與佈署工具，**協調層的資源調度前端**，負責將標準插件搬運至 `~/.openstarry`。

### 2. `packages/` (核心庫層)
這裡存放系統的核心邏輯，以 NPM 包的形式存在。
*   **`packages/core/`**: **(核心)** 絕對純淨的執行引擎，不含任何插件邏輯。
*   **`packages/sdk/`**: 插件開發契約，定義介面與規範。
*   **`packages/shared/`**: 全域通用的工具庫。

### 3. `tools/` (工程配套)
*   用於 CI/CD、代碼生成、文檔自動化的腳本。

---

## 生態系分層與代碼隔離 (Ecosystem Separation & Code Isolation)

為了確保核心絕對純淨，我們將「系統本體」與「功能內容」在物理上完全隔離：

### 1. 系統本體 (`openstarry`)
*   **定位：** 這是唯一的 Monorepo，包含系統內核、守護進程與基礎架構。
*   **原則：** **嚴禁包含任何具體的 Tool 或 Provider 代碼。** 此倉庫中不包含 `plugins/` 目錄。系統運行時會動態掃描：
    *   **系統路徑：** `~/.openstarry/plugins/` (由外部倉庫同步而來)。
    *   **專案路徑：** `<Project>/.openstarry/plugins/` (私有插件)。

### 2. 官方插件庫 (`openstarry_plugin`)
*   **定位：** 這是一個獨立的外部倉庫，作為官方維護的標準插件來源（類似 Linux 的套件源）。
*   **運作機制：** 
    *   它不參與 Core 的編譯或構建。
    *   用戶透過 `openstarry plugin sync` 指令將此倉庫的內容「下載並安裝」到系統路徑中。

---

## 關鍵配置文件示例

### `pnpm-workspace.yaml`
```yaml
packages:
  - 'apps/*'
  - 'packages/*'
  # 注意：plugins 不在此 workspace 中，它們是外部依賴
```

---

## 為什麼這樣設計？ (設計哲學)

1.  **物理隔離 = 邏輯解耦**：將 `core` 與 `daemon` 分開，確保了我們之前提到的「Core 是無頭的」這一哲學。Core 甚至不知道 Daemon 的存在，它只是一個被 Daemon 拉起來運行的庫。
2.  **內核絕對純淨**：`packages/core` 在編譯階段與插件完全分離。這確保了系統核心的穩定性與可移植性。
3.  **分離式安裝 (Detached Installation)**：安裝過程負責環境初始化與插件搬運，而非將插件硬編碼進內核。這使得 OpenStarry 的「數位物種」可以在不同的環境中擁有不同的「感官與肢體」。
4.  **單一版本管理**：作為開發者，您只需在根目錄執行一次 `pnpm install`，所有子專案的依賴都會被正確處理。