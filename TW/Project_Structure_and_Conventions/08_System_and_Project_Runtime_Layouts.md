# 08. 系統級與專案級運行時佈局 (System & Project Runtime Layouts)

本文件落實了 **"Directory as Permission"** 的核心哲學。OpenStarry 區分了「全域系統空間」與「局部專案空間」，以實現權限隔離與環境可攜性。

---

## 1. 系統全域空間 (System-wide Space)

這是 Orchestrator Daemon 的主目錄，通常位於用戶的主資料夾下。

*   **路徑 (Windows):** `%USERPROFILE%\.openstarry\`
*   **路徑 (Linux/macOS):** `~/.openstarry/`

### 目錄結構

```text
~/.openstarry/
├── daemon.pid              # Daemon 進程鎖定文件
├── config.json             # Daemon 全域配置文件 (API Port, 存儲路徑等)
├── agents/                 # [系統級 Agent] 受 Daemon 託管的所有實體
│   ├── agent-001/          # 符合 04 號文檔定義的標準目錄
│   └── agent-002/ 
│   
├── plugins/                # [系統級插件] 全域可用的工具與 Provider
│   
├── storage/                # 全域持久化數據
│   └── main.db             # SQLite/LevelDB (存儲 Agent 狀態索引)
│   
└── logs/                   # Daemon 系統日誌
```

---

## 2. 專案局部空間 (Project-specific Space)

當開發者在特定專案目錄下啟動 OpenStarry 時，系統會自動識別專案空間。

*   **路徑:** `[Your-Project-Path]/.openstarry/`

### 目錄結構

```text
.openstarry/
├── agent.json              # 專案專屬 Agent 配置
├── plugins/                # [專案插件] 僅對本專案可見的能力
│   └── custom-tools/       # 專案私有工具
│   
├── data/                   # 專案產出的本地數據
│   
└── logs/                   # 專案運行日誌
```

---

## 3. 加載優先級與覆蓋機制 (Overlay Mechanism)

OpenStarry 採用類似於 Linux UnionFS 的加載邏輯。當 Agent 尋找一個插件或配置時，搜尋順序如下：

1.  **專案空間 (`./.openstarry/`)**: 最高優先級，用於定制化與調試。
2.  **系統空間 (`~/.openstarry/`)**: 標準能力提供者。
3.  **內置空間 (Built-in)**: 隨二進制文件發布的原始默認配置。

### 權限安全原則

*   **隔離性**: 專案 Agent 默認無法訪問系統空間的其他 Agent 數據，除非明確獲得權限。
*   **唯讀性**: 系統級插件對專案 Agent 來說通常是唯讀的，防止專案級誤操作破壞系統穩定。

---

## 4. 哲學映射

*   **系統空間 = 「大環境」**: 數位物種生存的共同生態系統。
*   **專案空間 = 「棲息地」**: 數位物種具體的執行領地。
*   **目錄即權限**: 插件被放置在 `~/.openstarry` 還是 `./.openstarry`，直接決定了它的信任等級與可見範圍。
