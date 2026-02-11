# OpenStarry 是如何建造的：多 Agent 開發

> OpenStarry 不僅僅是一個 AI Agent 框架。它是一個**由 AI Agent 建造的** AI Agent 框架。

## 6-Agent 團隊

OpenStarry 由一個 6 名專業 AI Agent 組成的團隊開發，遵循正式的標準作業程序（SOP）。每個 Agent 都有明確的角色、專屬的工作區，以及結構化的產出要求。這不是一個玩具展示——而是一個真正的軟體工程流程，產出的是經過架構審查、品質驗證、有完整文件的程式碼。

### 團隊成員

| Agent | 角色 | 模型 | 工作區 | 產出 |
|-------|------|------|--------|------|
| **Architect** | 架構守護者 | Sonnet | 文件 + 程式碼（唯讀） | 設計規格、程式碼審查、安全稽核 |
| **Dev-Core** | 核心開發者 | Sonnet | `agent_dev/openstarry/` | SDK、Core、Shared、Runner 實作 |
| **Dev-Plugin** | 插件開發者 | Sonnet | `agent_dev/openstarry_plugin/` | Transport、Provider、Tool、Guide 插件 |
| **QA** | 品質保證 | Sonnet | `agent_test/`（隔離副本） | 建置驗證、測試報告、純度檢查 |
| **Doc-Keeper** | 文件管理者 | Haiku | `share/openstarry_doc/` | 決策記錄、迭代日誌、文件更新 |
| **Researcher** | 技術研究員 | Sonnet | `share/ref/` | 前期研究報告、參考專案分析 |

一位人類**協調者**使用 Claude Opus 來指揮團隊、做最終決策、解決爭議，並管理週期轉換。

### 為什麼是這樣的結構？

每個角色的存在都有其理由：

- **Architect** 防止架構侵蝕。沒有它，五蘊的純度會隨著捷徑的累積而逐漸退化。Architect 對每次提交都強制執行合規性檢查。
- **獨立的 Dev-Core 和 Dev-Plugin** Agent 可以平行工作，因為 SDK 介面已被凍結。Dev-Core 建造框架，Dev-Plugin 基於框架開發——同步進行。
- **QA 在隔離副本中工作**（`agent_test/`），不是在開發目錄中。這防止了經典的「在我的機器上可以跑」問題——如果在測試環境中通過，那就是真的通過了。
- **Doc-Keeper 使用較輕量的模型**（Haiku），因為文件更新不需要程式碼生成的推理能力，但需要保持一致性和完整性。
- **Researcher** 在實作開始前進行前期研究，研究參考專案（OpenClaw 用於 Agent 路由、OpenCode 用於 TUI 模式、OpenOctopus 用於 MCP 整合），以此為設計決策提供依據。

## 迭代週期

每個功能都經歷一個嚴謹的 8 階段週期，搭配品質關卡：

### 第 0 階段：規劃
```
Coordinator: "Cycle 1 範圍：Session 隔離 + HTTP SSE + 健康檢查"
Researcher: 研究 Transport 模式、SSE 最佳實踐、Session 管理策略
Doc-Keeper: 在 Iteration_Log.md 中記錄計畫，附上週期 ID（20260210_cycle1）
```

### 第 1 階段：設計
```
Architect: 產出 Architecture_Spec_Cycle1.md，包含：
  - 新增/修改的介面（ISession、ISessionManager、InputEvent.sessionId）
  - Session 生命週期的序列圖
  - 安全考量（Session 隔離防止跨 Session 干擾）

⚠️ 介面凍結：一旦發佈，介面不可更改，除非透過正式的 Spec Addendum
```

### 第 1.5 階段：基線備份
```
Coordinator: 執行 scripts/baseline.sh → 儲存實作前的快照
  └─ 安全網：如果出了問題，可以復原到這個時間點
```

### 第 2 階段：實作
```
Phase 2a: Dev-Core 先實作 SDK 介面變更
  └─ ISession、ISessionManager 加入 @openstarry/sdk
  └─ InputEvent 新增選用的 sessionId 欄位（向下相容）
  └─ pnpm build 必須通過才能進入 Phase 2b

Phase 2b: Dev-Core + Dev-Plugin 平行工作
  └─ Dev-Core: Session 管理器、執行迴圈的 Session 路由
  └─ Dev-Plugin: WebSocket Session 綁定、HTTP SSE 端點
  └─ 各自產出結構化的 DevLog，記錄決策、挑戰與解決方案
```

### 第 2.5 階段：同步
```
Coordinator: 執行 scripts/sync-to-test.sh
  └─ 從 agent_dev/ 原子性複製至 agent_test/
  └─ 確保測試環境擁有開發程式碼的完整副本
  └─ 測試環境中的 pnpm build 必須通過
```

### 第 3 階段：驗證（平行進行）
```
QA（在 agent_test/ 中）：                Architect（讀取 agent_dev/）：
├─ pnpm build → 全部 11 個套件通過     ├─ 五蘊合規性檢查
├─ pnpm test → 118+ 測試通過           ├─ 微內核純度審查
├─ pnpm test:purity → 零違規           ├─ 安全稽核
└─ 產出 QA_Report_Cycle1.md           └─ 產出 CodeReview_Cycle1.md
```

### 第 4 階段：收斂
```
PASS: Coordinator 執行 scripts/snapshot.sh → 儲存版本化快照
FAIL: 問題分類：
  ├─ Code Fix → 回到第 2 階段（實作中的 Bug）
  ├─ Design Fix → 回到第 1 階段（架構問題）
  └─ Plan Fix → 回到第 0 階段（需求問題）

最多 2 次返工循環，之後升級至人類處理
```

## 品質關卡

### 介面凍結
一旦 Architect 發佈了 Architecture Spec，介面就被凍結。Dev-Core 可以實作它們，但不能更改它們。這防止了困擾大多數專案的範圍蔓延和介面不穩定。如果介面需要變更，必須透過正式的 Spec Addendum——一個刻意的、經過審查的流程。

### 微內核純度驗證
QA 執行 `pnpm test:purity`，掃描編譯後的 Core 二進位檔是否有任何插件匯入。**不允許任何違規。**這個自動化關卡防止了隨時間推移逐漸侵蝕微內核架構的汙染。

### 測試隔離
QA 絕不在開發目錄中測試。同步腳本將程式碼複製到獨立的 `agent_test/` 環境。如果開發者留下了一個只在本地狀態下才能讓測試通過的暫時性修改，QA 會抓到它。

### 返工分類
並非所有失敗都是相同的：
- **Code Fix** → 快速修補，回到第 2 階段
- **Design Fix** → 需要介面變更，回到第 1 階段（需要 Spec Addendum）
- **Plan Fix** → 需求有誤，回到第 0 階段（罕見但嚴重）

## 結構化產出物

每個週期都產出可追溯的、結構化的報告：

```
share/test/reports/
├── research/20260210_cycle1/
│   └── Research_Transport_Enhancement.md      ← Researcher
├── arch_reviews/20260210_cycle1/
│   ├── Architecture_Spec_Cycle1.md            ← Architect（已凍結）
│   └── CodeReview_Cycle1.md                   ← Architect
├── dev_logs/20260210_cycle1/
│   ├── DevLog_Phase2a_SDK_Core.md             ← Dev-Core
│   └── DevLog_Phase2b_Transport_Plugins.md    ← Dev-Plugin
├── qa_results/20260210_cycle1/
│   └── QA_Report_Cycle1.md                    ← QA
└── sys_summary/20260210_cycle1/
    └── Convergence_Summary.md                 ← Coordinator
```

所有檔案都以週期 ID（`{YYYYMMDD}_cycle{N}`）命名，確保完整的可追溯性。六個月後，你可以追溯任何一行程式碼，回到啟發它的研究、定義它的規格、實作它的開發日誌，以及驗證它的 QA 報告。

## 這對社群意味著什麼

當 OpenStarry 發佈時，貢獻者繼承的不只是程式碼：

1. **一套完整的開發流程**——不只有程式碼，還有一套工作方法
2. **架構護欄**——自動化純度測試防止善意的貢獻侵蝕架構
3. **可追溯的決策**——每一個「為什麼」都有文件記錄，不會迷失在聊天記錄中
4. **平行安全的開發**——Dev-Core 和 Dev-Plugin 可以是同時工作的不同貢獻者，因為 SDK 介面是凍結的
5. **可重現的品質**——無論開發者是人類還是 AI，SOP 都是一樣的

> 流程本身就是產品的一部分。
