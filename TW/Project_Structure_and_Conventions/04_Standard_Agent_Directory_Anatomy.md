# 03. 標準代理人目錄解剖學與層級規範 (Standard Agent Directory Anatomy & Hierarchy)

本文件定義了 OpenStarry 系統中「代理人工作空間」的標準佈局。此規範同時適用於 **系統級 (System-level)** 與 **專案級 (Project-level)** 的環境。

## 核心哲學：自相似性 (Self-Similarity)

在 OpenStarry 中，系統管理員（System Agent）與具體的執行者（Project Agent）在物理結構上是完全對稱的。這種「分形」設計確保了任何一個專案在未來都有可能演化成另一個子系統的管理器。

---

## 標準目錄結構 (The Anatomy)

無論是在系統路徑還是專案路徑，都必須遵循以下佈局：

```text
[Agent_Root]/
├── plugins/           # 聚合體插件存放區 (Aggregates)
│   ├── tool-A/        # 插件 A
│   └── listener-B/    # 插件 B
├── configs/           # 代理人藍圖與配置 (Manifests)
│   └── agent.json     # 定義該 Agent 加載哪些插件
├── states/            # 狀態與持久化 (States/Snapshots)
│   └── memory.db      # 識蘊 (Vijnana) 的物理存儲
└── logs/              # 運行時日誌 (受蘊的歷史記錄)
```

---

## 插件發現與繼承機制

Agent 啟動時，其 `PluginLoader` 會掃描多個層級的 `plugins/` 目錄：

1.  **優先級 1 (專案級):** `[Current_Project_Root]/plugins/`
2.  **優先級 2 (系統級):** `[openstarry_system_root]/plugins/`

**規則：**
*   專案 Agent **可以** 調用系統級插件，實現資源共享（如使用系統統一提供的 `git-tool`）。
*   如果兩處存在同名插件，**專案級優先**，允許專案覆蓋系統默認行為。

---

## 角色權限與隔離 (Security Policy)

雖然結構相同，但不同層級的 Agent 受到嚴格的職責限制：

### 1. 系統代理人 (System Agent / Manager)
*   **駐留位置：** `openstarry_system_root`
*   **核心任務：** 管理專案 Agent 的生命週期（創建、監控、銷毀）。
*   **嚴格限制：** **嚴禁加載任何業務插件**（如 WhatsApp, 數據庫操作工具）。
*   **操作邏輯：** 如果系統 Agent 需要執行業務操作，它必須啟動一個特定的「專案 Agent」來代為執行。它本身只擁有 `AgentManagerTool`。

### 2. 專案代理人 (Project Agent / Worker)
*   **駐留位置：** 具體的專案目錄。
*   **核心任務：** 執行具體業務。
*   **操作邏輯：** 擁有其目錄下 `plugins/` 內所有工具的完整操作權限。

---

## 強制關閉機制 (The Kill Switch)

系統 Agent 與人類管理員擁有對專案 Agent 的最高控制權：

1.  **進程控制：** 系統 Agent 通過 `Orchestrator Daemon` 維護專案 Agent 的 PID。
2.  **強制終止：** 當專案 Agent 失控或任務完成時，系統 Agent 可以發送 `SIGKILL` 訊號。
3.  **哲學意義：** 這代表了「識蘊 (主體意識)」對「行蘊 (行動)」的絕對控制。當一個行動不再符合系統目標時，管理中心必須能物理性地切斷其能量供應（結束進程）。
