# OpenStarry Implementation Plan 02 — 架構補齊與品質提升

> **狀態**: ✅ 已完成 (2026-02-05)

## 背景

Plan01 完成了 MVP v0.1 Alpha 的基礎骨架（可編譯、可啟動、可接收指令）。
但經由測試人員的驗證報告（`test/20260204/`），以及與 `openstarry_doc` 設計文件的比對，
識別出以下幾類**實作與設計之間的落差 (Gaps)**。

本計畫定義 Plan02 的改進範圍，目標是**補齊 Phase 2.3 / Phase 3.1 的缺失，並達成設計文件的要求**。

---

## 一、測試報告識別的問題

| # | 來源 | 問題 | 嚴重度 |
|---|------|------|--------|
| G1 | Developer_Handoff_Report / Engineering_Implementation_Gaps | CLI 插件載入被硬編碼 (`switch-case`)，無法動態 `import()` 第三方插件 | 🔴 高 |
| G2 | Engineering_Implementation_Gaps | `ExecutionLoop.run(string)` 接受字串，非事件驅動；與文件「核心唯一輸入源是事件隊列」不符 | 🔴 高 |
| G3 | QA_MultiInput_Verification_Report | 工具執行期間的異步競爭風險 — 新輸入與當前 loop 可能狀態衝突 | 🟡 中 |
| G4 | Developer_Handoff_Report / QA_Report | 缺少併發單元測試，`EventQueue` FIFO 穩定度未驗證 | 🟡 中 |

---

## 二、設計文件比對 — 未實作功能

| # | 文件來源 | 缺失功能 | 屬於 Phase |
|---|----------|----------|-----------|
| D1 | `02_Headless_Agent_Core.md` | ExecutionLoop 應為 `WAITING_FOR_EVENT` 狀態機，從 EventQueue pull 事件，而非直接接收 string | 2.1 |
| D2 | `12_Capabilities_Injection_Mechanism.md` | PluginLoader 應支援動態 `import()` 載入，根據 `agent.json` 路徑解析 | 3.1 |
| D3 | `07_Safety_Circuit_Breakers.md` | 資源級熔斷：Token 預算上限、循環次數上限 (Loop Cap) | 2.3 |
| D4 | `07_Safety_Circuit_Breakers.md` | 行為級熔斷：重複工具調用偵測、錯誤級聯熔斷 | 2.3 |
| D5 | `12_Error_Handling_and_Self_Correction.md` | 「錯誤即痛覺」機制：工具錯誤標準化 + 挫折計數器 + 強制求助 | 2.3 |
| D6 | `09_Observability_and_Tracing.md` | 結構化 JSON 日誌（含 agent_id, trace_id, module）| 3.1 |
| D7 | `15_Testing_Strategy_and_Infrastructure.md` | 單元測試基建 (Vitest)、核心純淨性檢查、MockHost | 1.3 |
| D8 | `01_Execution_Loop.md` | 輸出路由：根據事件 `source` 回傳至正確渠道 | 2.1 |

---

## 三、Plan02 實作步驟

### Phase A：事件驅動轉型（對應 G1, G2, D1, D2, D8）

**目標**：ExecutionLoop 不再直接接收 string，改為監聽 EventQueue；CLI 支援動態插件載入。

#### A1. ExecutionLoop 事件化重構
- `ExecutionLoop` 新增 `start()` 方法：持續從 `EventQueue.pull()` 取出事件
- 事件 payload 標準化：`{ source: string, type: string, data: unknown }`
- `run(userInput)` 降級為內部方法，僅由事件觸發器調用
- 加入 `isProcessing` 鎖防止同時處理多事件（解決 G3 競爭問題）
- 輸出路由：根據 `event.source` 決定回覆渠道

#### A2. AgentCore 串接
- `AgentCore.processInput()` 改為推入 EventQueue 而非直接呼叫 loop
- Listener 插件推入事件至 EventQueue（而非直接呼叫 AgentCore）
- 保留 slash command 的快速路徑（不進入 LLM 循環）

#### A3. CLI 動態插件載入
- 新增 `DynamicPluginLoader`：根據 `agent.json` 的 `plugins[].path` 使用 `import()` 動態載入
- 保留 builtin 插件的快速路徑（無 path 時使用 switch-case fallback）
- 支援 `node_modules` 包名解析（`import(pluginName)`）

#### A4. 事件 Payload 標準化
- 定義 `InputEvent` 類型：
  ```typescript
  interface InputEvent {
    source: string;      // "cli", "webhook", "mcp"
    type: string;        // "user_input", "system_command"
    data: unknown;
    replyTo?: string;    // 回覆渠道標識
  }
  ```

---

### Phase B：安全熔斷與自我修正（對應 D3, D4, D5）

**目標**：實作 `SafetyMonitor` 模組，防止失控與無限循環。

#### B1. SafetyMonitor 模組
- **位置**：`packages/core/src/security/safety-monitor.ts`
- **職責**：
  - `beforeLLMCall()`: Token 預算檢查
  - `afterToolExecution()`: 重複調用偵測 + 錯誤率計算
  - `onLoopTick()`: 循環次數上限檢查

#### B2. 資源級熔斷
- `MAX_LOOP_TICKS`：單次任務最大循環次數（預設 50，可在 `policy` 配置）
- `MAX_TOKEN_USAGE`：Token 累計消耗上限（依賴 Provider 回傳的 `usage`）
- 觸發後狀態切換為 `SAFETY_LOCKOUT`

#### B3. 行為級熔斷
- `ToolCallFingerprint` 歷史隊列：hash(toolName + args)
- 連續 N 次相同失敗指紋 → 強制注入系統提示：「STOP and analyze why」
- 滑動視窗錯誤率（最近 10 次中 8 次失敗 → `EMERGENCY_HALT`）

#### B4. 挫折計數器與自我修正
- 連續失敗計數器
- 超過閾值（預設 5）→ 強制注入 System Prompt：「ask the user for help」
- 工具錯誤標準化為 `{ code, message, suggestion }` 結構

---

### Phase C：測試基建（對應 G4, D7）

**目標**：建立 Vitest 測試環境，覆蓋核心邏輯。

#### C1. 測試框架配置
- 根目錄加入 Vitest 配置
- 各 package 的 `test` script

#### C2. 核心單元測試
- `EventBus.test.ts`：多 handler、wildcard、錯誤隔離
- `EventQueue.test.ts`：FIFO 順序、併發 push/pull、壓力測試
- `StateManager.test.ts`：add/clear/snapshot/restore
- `ContextManager.test.ts`：滑動視窗截斷邏輯
- `SecurityLayer.test.ts`：路徑驗證正確性
- `SafetyMonitor.test.ts`：各級熔斷觸發條件

#### C3. 整合測試
- `MultiSourceEvent.test.ts`：模擬多來源事件同時注入
- `ExecutionLoop.test.ts`：mock provider，驗證完整 loop 流程
- `PluginLoader.test.ts`：動態載入驗證

#### C4. 核心純淨性檢查
- 加入 `dependency-cruiser` 或 ESLint 規則
- 確保 `packages/core` 不 import `plugins/*` 或 `apps/*`

---

### Phase D：可觀測性提升（對應 D6）

**目標**：結構化 JSON 日誌。

#### D1. Logger 升級
- `packages/shared/src/logger` 輸出 JSON 格式
- 新增 `agent_id`, `trace_id` 欄位
- 支援日誌等級過濾（環境變數 `LOG_LEVEL`）

---

## 四、實作優先順序

```
Phase A (事件驅動轉型)     ← 🔴 最高優先級，測試報告的核心問題
  └→ A1 ExecutionLoop 事件化
  └→ A2 AgentCore 串接
  └→ A3 動態插件載入
  └→ A4 事件 Payload 標準化

Phase B (安全熔斷)          ← 🔴 高，Phase 2.3 未完成項
  └→ B1 SafetyMonitor
  └→ B2 資源級熔斷
  └→ B3 行為級熔斷
  └→ B4 挫折計數器

Phase C (測試基建)          ← 🟡 中，Phase 1.3 未完成項
  └→ C1 Vitest 配置
  └→ C2 核心單元測試
  └→ C3 整合測試
  └→ C4 純淨性檢查

Phase D (可觀測性)          ← 🟢 低，可與其他 phase 並行
  └→ D1 Logger 升級
```

---

## 五、完成後的預期狀態

完成 Plan02 後，專案將達到：

- **Phase 1.3** ✅ 測試基建 + CI 規則
- **Phase 2** ✅ 完整的意識內核（含安全熔斷、事件驅動、自我修正）
- **Phase 3.1** ✅ 動態插件載入 + 事件標準化
- `openstarry_doc` 中 Phase 1~3.1 的所有設計要求均已落地

**距離 Phase 3 完成還差**：
- 3.2 `openstarry plugin sync` 指令
- 3.2 `guide-mcp` + `standard-function-skill` 插件
- 3.4 `openstarry create-plugin` 腳手架 + `MockHost`

**距離 Phase 4 完成還差**：
- 4.2 端對端 LLM 通話驗證（需 OAuth 登入後手動測試）

---

## 六、驗證方式

1. `pnpm install && pnpm build` — 編譯通過
2. `pnpm test` — 所有單元測試 + 整合測試通過
3. EventQueue 壓力測試通過（1000 事件 FIFO 正確）
4. 動態載入：`agent.json` 配置自訂 plugin path → CLI 成功載入
5. 安全熔斷：模擬連續失敗 → SafetyMonitor 正確觸發 halt
6. 多輸入模擬：兩個事件源同時注入 → 逐一處理，無衝突
