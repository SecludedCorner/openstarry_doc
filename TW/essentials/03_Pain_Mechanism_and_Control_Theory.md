# OpenStarry Agent 如何學習：痛覺機制與控制理論

> *"錯誤是學習的機會。"*

## 錯誤處理的問題

大多數 AI Agent 框架把錯誤當成崩潰來處理。某個東西失敗了，Agent 就死掉或盲目重試，使用者重新啟動。沒有學習，沒有適應，沒有對出錯的記憶。

OpenStarry 採取了一種受生物學啟發的根本性不同做法：**錯誤即是痛覺信號**。

## 痛覺機制

### 生物痛覺如何運作

當你碰到燙爐子時，你不會當機。你的神經系統發送痛覺信號，你的大腦解讀它（「那很燙，那很痛」），然後你把手抽回來。下次，你會用不同的方式靠近爐子。痛覺不是 Bug——它是生物學中最重要的回饋機制。

### OpenStarry 痛覺如何運作

三個層次，對應生物神經系統：

```
1. Pain Sensing (Core)       — SafetyMonitor 捕捉工具執行失敗
        ↓
2. Pain Conduction (Plugin)  — Guide 插件將失敗轉化為有意義的回饋
        ↓
3. Pain Response (LLM)       — Agent「感受」痛覺，反思，調整策略
```

### 具體情境

Agent 嘗試讀取受限的檔案：

**第一步——行動：** Agent 對 `/root/secret.txt` 呼叫 `fs.read`

**第二步——失敗：** 系統回傳 `Error: EPERM (Permission Denied)`

**第三步——痛覺注入：** Core 呼叫 Guide 插件的 `interpretPain()`：

```typescript
interpretPain: (error) => {
  const severity = calculateSeverity(error);
  return `
【System Pain Alert】
Execution anomaly detected!
Source: ${error.source}
Message: ${error.message}
Pain Level: ${severity}
Status: Your action has been blocked. This causes "discomfort."

Stop repeating this attempt. Analyze the pain source.
Adjust your strategy in the next tick.
  `;
};

calculateSeverity(error) {
  if (error.code === 'EPERM') return '🔥🔥🔥 Critical Pain';
  if (error.code === 'ENOENT') return '⚡ Medium Pain';
  return '💧 Low Pain';
}
```

**第四步——作為工具結果訊息注入上下文：**

> 【System Pain Alert】
> Source: fs.read
> Message: EPERM (Permission Denied)
> Pain Level: 🔥🔥🔥 Critical Pain
> Status: Your action has been blocked.

**第五步——自我修正：** LLM 處理這個痛覺信號並回應：

> 「讀取失敗，觸發嚴重痛覺警報。這表示直接路徑被禁止，可能觸發安全斷路器。我必須停止撞擊權限牆。我應該先檢查我的許可路徑，並尋找替代方案。」

### 三種痛覺類型

| 類型 | 範例 | Agent 回應 | 系統回應 |
|------|------|-----------|---------|
| **執行痛覺** | 參數錯誤、檔案不存在、權限被拒 | 反思，調整做法 | 痛覺注入上下文 |
| **暫態痛覺** | 網路逾時、API 頻率限制 | 等待並重試 | 指數退避自動重試 |
| **致命痛覺** | 記憶體不足、安全違規、錯誤級聯 | 無法自我修正 | 程序終止 + Daemon 警報 |

### 挫折計數器

如果 Agent 忽視痛覺，持續犯相同的錯誤呢？

SafetyMonitor 使用 **SHA-256 指紋辨識**來偵測重複的失敗：

```typescript
// 工具名稱 + 參數的雜湊 = 唯一指紋
function fingerprint(toolName: string, argsJson: string): string {
  return createHash("sha256")
    .update(`${toolName}:${argsJson}`)
    .digest("hex")
    .slice(0, 16);
}
```

**升級階梯：**

| 連續失敗次數 | 回應 |
|-------------|------|
| 1-2 | 正常痛覺回饋——Agent 有機會自我修正 |
| 3（相同指紋） | **系統警報注入**："SYSTEM ALERT: You are repeating a failed action. STOP and analyze why." |
| 5 | **挫折閾值**——系統強制暫停命令 |
| 10 次操作中有 8 次錯誤 | **錯誤級聯偵測**——`EMERGENCY_HALT`，狀態 → `ERROR_PAUSED` |

這對應了生物的挫折感：重複的痛覺信號從「哎呀」升級到「停下來想想」再到「你需要幫助」。

### 事實與意義的分離

一個關鍵的設計原則：**Core 提供失敗的事實。Guide 插件提供意義。**

Core 說：`Tool "fs.write" failed with EPERM at /etc/passwd.`
Guide 解讀：「你嘗試寫入一個系統檔案。這造成了嚴重痛覺。停下來重新考慮。」

這種分離意味著不同的 Guide 插件可以對相同的錯誤給出不同的解讀：
- **安全導向**的 Agent 將 EPERM 視為違規——「你不應該嘗試這個。」
- **學習型** Agent 將其視為探索邊界——「這條路不通。讓我們找另一條。」
- **除錯型** Agent 將其視為診斷資料——「偵測到權限問題。檢查檔案所有權。」

Core 保持純淨。意義是插件的職責。

## 多層級斷路器

除了痛覺之外，OpenStarry 還實作了受工業斷路器啟發的三層安全系統：

### 第一級：資源限制（資源級）

在每次操作前強制執行的硬性限制：

| 資源 | 預設限制 | 觸發後果 |
|------|---------|---------|
| Token 預算 | 每個 Agent 100,000 tokens | 強制終止迴圈，進入 `STOPPED` 狀態 |
| 迴圈迭代次數 | 每個任務 50 ticks | 「偵測到無限迴圈」→ 暫停等待人為介入 |
| 工具逾時 | 每次執行 30 秒 | `Promise.race()` 終止工具呼叫 |

```typescript
// 每次 LLM 呼叫前：
const tokenCheck = safetyMonitor.beforeLLMCall();
if (tokenCheck.halt) {
  setState("SAFETY_LOCKOUT");
  return; // 預算耗盡——不再思考
}
```

### 第二級：行為分析（行為級）

啟發式偵測問題行為模式：

- **重複工具呼叫**：相同指紋 + 失敗 × 3 = 注入警報
- **錯誤級聯**：滑動視窗 10 次操作中 80% 錯誤率 = `EMERGENCY_HALT`
- **輸出異常**：連續無效 JSON 或呼叫不存在的工具 = 升級處理

### 第三級：人為覆寫（指令級）

終極開關。`SYSTEM_HALT` 事件被標記為 **Priority 0**（最高優先級）。即使佇列中有 100 個待處理任務，暫停命令也會在下一次迴圈迭代開始時立即被處理：

```
EXECUTING --[limit reached / anomaly]--> SAFETY_LOCKOUT
SAFETY_LOCKOUT --["admin:unlock"]--> WAITING_FOR_EVENT
```

Agent 只能透過明確的人為介入才能解鎖。

## Agent 即控制系統

除了痛覺和安全機制之外，OpenStarry 將整個 Agent 建模為一個**回饋控制系統**——與設計自動駕駛、恆溫器和工業機器人所用的同一套數學框架。

### 控制迴圈

```
                    ┌───────────────────────────────────┐
                    │                                   │
  User Goal ──────► │  Error = Goal - Current State     │
  (reference)       │          ↓                        │
                    │     Controller (LLM)              │
                    │          ↓                        │
                    │     Control Input (Tool Calls)    │
                    │          ↓                        │
                    │     Plant (External World)        │
                    │          ↓                        │
                    │     Sensor (Tool Results)   ──────┘
                    │          ↓
                    │     Measured Output (current state)
                    └───────────────────────────────────┘
```

### 對應關係

| 控制理論 | OpenStarry 組件 | 具體範例 |
|---------|----------------|---------|
| 參考輸入 (r) | System Prompt + 使用者訊息 | 「找出並修復 auth.ts 中的 Bug」 |
| 控制器 (C) | LLM | Gemini 2.0 Flash 分析問題 |
| 控制輸入 (u) | 工具呼叫 | `fs.read("auth.ts")`、`fs.write("auth.ts", fixedCode)` |
| 受控對象 (P) | 外部世界 | 檔案系統、程式碼庫 |
| 感測器 (H) | 工具結果 | 檔案內容、錯誤訊息、測試輸出 |
| 誤差信號 (e) | 上下文差距 | 「Bug 仍然存在」→ 繼續迭代 |

### 三個穩定性問題（與解方）

**1. 振盪**——Agent 在兩個狀態之間來回跳動（復原/重做循環）
- *成因：*過度反應或誤導性的感測器資料
- *解方：*上下文歷史作為**積分項**——Agent 記住過去的嘗試，避免重蹈覆轍。滑動視窗保持近期歷史可見。

**2. 發散**——Agent 偏離原始目標
- *成因：*上下文充滿雜訊，原始意圖被淹沒
- *解方：***上下文錨定**——System Prompt 在上下文組裝中擁有最高權重，即使滑動視窗丟棄較舊的訊息，System Prompt 也永遠不會被修剪。

**3. 穩態誤差**——Agent 自認為完成了，但實際上沒有
- *成因：*驗證不足（感測器盲點）
- *解方：***驗證步驟**——在宣告完成前強制進行檢查。如同 PID 的微分項，測量的是誤差的*變化率*，而非僅僅是誤差本身。

### 關鍵洞察

> **智慧不僅僅在於擁有強大的 LLM。它在於回饋迴路的品質。**

一個搭配優秀回饋（詳細的工具結果、準確的痛覺信號、適當的上下文管理）的普通 LLM，會勝過搭配糟糕回饋（被吞掉的錯誤、沒有上下文、沒有驗證）的頂尖 LLM。

這就是為什麼 OpenStarry 在以下方面投入大量心力：
- **錯誤標準化**——每次失敗都產生一致的、可解析的輸出
- **痛覺解讀**——Guide 插件賦予錯誤意義
- **上下文管理**——可插拔的策略決定 LLM 看到什麼
- **安全監控**——行為分析防止病態迴圈

LLM 只是其中一個組件——控制器。回饋迴路才是整個系統。

> *"當工具調用失敗，Core 不會崩潰，而是將錯誤訊息作為「感官輸入」回饋給 LLM。"*
