# 10. 構建與分發策略 (Build & Distribution Strategy)

本文件定義了 OpenStarry 的「分離式構建」規範。核心目標是保持 **Agent Core 與協調層的絕對純淨**，確保內核不包含任何插件代碼，並在安裝時實現物理分離。

## 1. 核心純淨原則 (The Purity Principle)

*   **內核零捆綁 (Zero Bundling):** `packages/core` 的編譯產物中 **嚴禁** 包含任何 `plugins/` 目錄下的代碼。內核僅依賴於 `packages/sdk` 的介面定義。
*   **動態載入:** 內核必須在運行時通過插件加載器 (PluginLoader) 從外部路徑動態載入 `.js` 文件。

## 2. 構建產物佈局 (Build Artifacts)

執行 `pnpm build` 後，`dist/` 目錄應呈現以下結構：

```text
dist/
├── bin/
│   └── openstarry-core.js  # [絕對純淨] 內核執行檔
├── lib/
│   ├── sdk.js              # SDK 庫
│   └── shared.js           # 共享工具庫
└── assets/
    └── standard-plugins/    # [分離存放] 待分發的標準插件
        ├── tool-fs/        # 編譯後的插件 A
        └── listener-stdio/ # 編譯後的插件 B
```

## 3. 安裝邏輯 (Installation Logic)

安裝檔（或安裝腳本）在執行時必須完成以下任務，以建立「數位物種」的生存環境：

### A. 系統定位 (System Bootstrapping)
1.  將 `bin/openstarry-core` 複製到系統執行路徑 (如 `/usr/local/bin` 或 `%ProgramFiles%`)。
2.  初始化用戶工作目錄：`~/.openstarry/` (Linux/macOS) 或 `%USERPROFILE%\.openstarry\` (Windows)。

### B. 插件搬運 (Plugin Relocation)
將 `assets/standard-plugins/*` 中的所有插件 **移動或複製** 到全域插件目錄中：
*   **目標路徑:** `~/.openstarry/plugins/`

## 4. 運行時發現機制 (Runtime Discovery)

當 Agent Core 啟動時，它**不應**從其安裝目錄尋找插件，而是根據以下優先級掃描系統路徑：

1.  **專案目錄:** `./.openstarry/plugins/` (若存在)
2.  **系統全域目錄:** `~/.openstarry/plugins/` (這是由安裝程序搬運過去的位置)

## 5. 開發者同步指令 (Dev Sync)

為了開發便利，我們提供 `pnpm run plugins:sync` 指令，模擬上述安裝邏輯：
*   **邏輯:** 自動編譯所有 `plugins/` 並將產物直接建立軟連結 (Symlink) 到 `~/.openstarry/plugins/`，實現「開發即安裝」。
