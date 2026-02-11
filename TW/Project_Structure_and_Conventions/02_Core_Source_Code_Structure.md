# 02. Core 源碼目錄結構規範 (Core Source Code Structure)

本文件定義了 `packages/core/src/` 內部的代碼組織方式。此結構旨在實現「高度解耦、可測試、且符合擬人化架構」的開發目標。

## 源碼目錄樹狀圖

```text
src/
├── agents/             # 代理人主體類別 (Agent Entities)
├── execution/          # 執行、排程與事件流 (The Loop)
├── memory/             # 上下文管理與記憶策略 (Working Memory)
├── infrastructure/     # 插件加載、註冊與隔離 (Plugin Infra)
├── security/           # 安全攔截與斷路器 (Guardrails)
├── state/              # 狀態快照與持久化 (State Manager)
├── transport/          # 核心通訊橋接層 (Communication Bridge)
├── types/              # TypeScript 強類型定義
└── index.ts            # 庫入口文件
```

---

## 模組職責詳解 (含五蘊哲學映射)

### 1. `execution/` (魂魄：執行系統) —— [映射：行蘊 Samskara / 識蘊 Vijnana]
這是 Agent 的生命中樞，實現了「執行迴圈」與「控制理論」模型。
*   **`loop/`**: 實現 `tick()` 機制，負責從事件隊列取件、調用 LLM 並觸發行動（意志與造作）。
*   **`queue/`**: 管理異步事件優先級。

### 2. `memory/` (記憶：上下文策略) —— [映射：想蘊 Samjna]
實現 `10_Context_Management_Strategy` 中定義的邏輯。
*   **`context/`**: 負責對接收到的訊息進行識別、關聯與壓縮（認知與模式識別）。

### 3. `infrastructure/` (器官：插件管理) —— [映射：色蘊 Rupa]
實現「一切皆插件」的物理加載。
*   **`loader/` & `registry/`**: 負責掃描並存儲 Agent 的「身體部件」。當插件加載時，它將其具體功能（色）拆解並註冊到系統中。

### 4. `security/` (超我：安全層) —— [映射：受蘊 Vedana 的防禦面]
夾在「決策」與「執行」之間的防火牆。
*   **`guardrails/`**: 負責捕獲異常並將其轉化為 Agent 的「負面感受（痛覺）」，從而觸發自我修正。

### 5. `transport/` (感官：通訊橋接) —— [映射：受蘊 Vedana 的感知面]
實現 `02_Communication_Interface` 定義的 API。
*   **`bridge/`**: 負責對接外部刺激（眼、耳、身等監聽器），將外界資訊轉化為內部感受。

---

## 命名慣例 (Naming Conventions)

1.  **類別 (Classes):** 使用大駝峰命名，如 `AgentCore`, `ExecutionLoop`。
2.  **介面 (Interfaces):** 以 `I` 開頭，如 `IPlugin`, `ITool`。
3.  **文件後綴:** 
    *   邏輯實現：`.ts`
    *   單元測試：`.test.ts` (放在與源文件同級的目錄)
4.  **入口:** 每個目錄應包含一個 `index.ts` 用於導出公共 API，保持外部引用的簡潔。

---

## 代碼演進原則

*   **無副作用 (No Side Effects):** 除了 `infrastructure/` 和 `transport/` 之外，其餘模組應盡量保持純粹邏輯，不直接操作網絡或文件系統。
*   **依賴注入 (Dependency Injection):** `AgentCore` 類在初始化時，應接收 `IContextManager`、`IStateManager` 的實例，以便在測試時可以輕易替換為 Mock 版本。
