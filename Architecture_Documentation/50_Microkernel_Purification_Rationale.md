<!-- Status: CURRENT -->
<!-- Layer: 2-Philosophy -->
<!-- Applies to: v0.34.0-alpha -->
<!-- Last verified: 2026-03-16 -->

# 52. 微核心純化原理 (Microkernel Purification Rationale)

`[Cycle 02-9 新增]`

> 本文件記錄 OpenStarry 選擇微核心架構的工程理由、mechanism/policy 分類準則、純化邊界判定方法、以及 Plan32 實施成果。回答三個核心問題：為什麼選微核心、如何判斷什麼該留在核心、純化後的工程成果為何。

---

## 1. 概述

OpenStarry 是一個 AI Agent 微核心作業系統。其核心架構選擇——微核心 (Microkernel) 而非單體核心 (Monolithic Kernel)——並非基於傳統 OS 設計的理想主義，而是基於 AI Agent 場景的具體工程分析。

本文件涵蓋以下主題：

| # | 主題 | 說明 |
|---|------|------|
| 1 | 為什麼選微核心 | AI Agent OS 場景下，微核心的傳統缺點不成立，而優勢全部成立 |
| 2 | Mechanism vs Policy 分類 | BABBAGE 連續性測試提供形式化的分類準則 |
| 3 | SUSSMAN 三層配置 | Policy 值移出核心後的配置架構 |
| 4 | 純化邊界準則 | 何時移出、何時不移 |
| 5 | Plan32 實施成果 | 四個新 Plugin、53 個 policy 常量遷移 |

安全面向的完整分析見 **Technical_Specifications/10** (Microkernel Security Analysis)。

---

## 2. 為什麼 OpenStarry 選擇微核心架構

### 2.1 傳統微核心的歷史爭議

在傳統作業系統領域，微核心架構 (Minix、QNX、seL4) 長期面臨一個核心批評：**IPC (進程間通訊) 的效能開銷**。Linux 選擇 monolithic kernel 的最重要原因，就是在硬體層級，context switch 的代價過高。頻繁的 IPC 導致微核心在吞吐量密集的場景下表現不佳。

這個批評在傳統 OS 語境下是成立的。

### 2.2 AI Agent OS 場景的根本差異：IPC 開銷可忽略

OpenStarry 的情境與傳統 OS 完全不同。關鍵數據：

| 操作 | 延遲量級 | 說明 |
|------|---------|------|
| LLM 單次呼叫 | 數百毫秒 ~ 數秒 | Agent 的「進程」核心操作 |
| Plugin 間訊息傳遞 (JS/TS) | 微秒級 | IPC 等效操作 |
| **比例** | **~1:10,000** | IPC 開銷完全可忽略 |

在 AI Agent OS 中，「進程」是 AI Agent，其主要計算瓶頸是 LLM 推理。Plugin 之間的訊息傳遞 (在 JS/TS 層級) 相較之下快了四個數量級。**傳統微核心最大的反對理由，在 AI Agent OS 場景下不成立。**

這是一個結構性的場景差異，而非微小的效能最佳化。傳統 OS 的 IPC vs 函數呼叫比例可能是 1:10 或 1:100，足以構成架構否決理由；但 1:10,000 的比例意味著 IPC 開銷在統計上不可見。

### 2.3 Agent 天生不可信——隔離是必要的

LLM 的輸出具有本質上的不確定性。AI Agent 不是確定性程式——給定相同輸入，不保證產生相同輸出。這使得 Agent 在信任模型上類似於「不可完全信任的第三方進程」。

在確定性程式的世界裡，monolithic kernel 的信任模型 (所有模組共享核心空間) 是可接受的——因為程式行為可預測、可驗證。但面對 AI Agent：

- Agent 可能受到 prompt injection 攻擊，產生非預期行為
- Agent 的決策空間遠大於傳統程式，邊界行為難以窮舉測試
- Agent 操作使用者敏感資料 (API key、個人資訊等)，一旦被攻破後果嚴重

微核心的隔離設計正是為這類不可信進程而生。

### 2.4 演化速度的需求

AI 領域的變化速度遠超傳統軟體。Agent 的能力和行為模式持續在變——新的 LLM 架構、新的工具介面、新的安全機制不斷出現。

Plugin 架構讓系統可以：

- 新增功能 = 新增 Plugin，不改核心
- 替換任何一個模組而不影響核心穩定性
- 按需載入/卸載功能，資源使用更彈性
- 不同 Agent 選擇不同的 Plugin 組合，形成差異化能力

Monolithic 架構在這種演化速度下會成為負擔——每次功能變更都可能影響核心穩定性。

### 2.5 核心零策略常數的合理性

OpenStarry Tenet #7 要求「微內核與絕對純淨」。Master 裁定 MR-6 明確要求 Core 必須零 policy 常量。策略放進核心意味著每次策略調整都要改核心，這是不必要的風險。核心只做機制 (mechanism)，策略全在 Plugin——這在 Agent OS 場景下不是理想主義，是工程上合理的選擇。

---

## 3. Mechanism vs Policy 分類原則

### 3.1 BABBAGE 連續性測試

`[來源: DD-14, Cycle 02-8_a D3-Q1-R1, 24/24 全票]`

BABBAGE 提出的連續性測試是 OpenStarry 正式的 mechanism/policy 分類準則：

> **定義**: 若移除一個常量後，系統在數學上失去定義 (除以零、NaN 傳播、型別不完整等)，則該常量是 **mechanism**；否則是 **policy**。

這個測試提供了客觀的、可重複驗證的分類標準，取代了主觀的「這感覺像是核心功能」判斷。

### 3.2 分類結果

Plan32 對 Core 中 61 個常量進行 BABBAGE 測試：

| 分類 | 數量 | 處置 |
|------|------|------|
| Mechanism | 8 | 保留在 Core |
| Policy | 53 | 遷移至 SDK/Config |

8 個 mechanism 值 (保留在 Core)：

| # | 值 | 理由 |
|---|-----|------|
| 1 | NaN guard (confidence clamping) | 移除後 NaN 傳播，系統數學失定義 |
| 2 | clamp01 utility | 移除後數值可超出 [0,1] 邊界 |
| 3 | AuditContext version discriminant | 移除後型別不完整 |
| 4 | SparshEvent version discriminant | 移除後型別不完整 |
| 5 | Cold-start confidence = 0.5 | 移除後初始狀態未定義 |
| 6 | Cold-start gear = 3 | 移除後初始齒輪未定義 |
| 7 | coherence = 1.0 (乘法恆等元) | 移除後乘法運算失定義 |
| 8 | efficiency = 1.0 (乘法恆等元) | 移除後乘法運算失定義 |

53 個 policy 值分三優先級遷移：

| 優先級 | 數量 | 範圍 |
|--------|------|------|
| P0 | ~20 | 校準 + 安全 (circuit breaker, confidence audit, mano aggregator) |
| P1 | ~19 | 核心操作 (execution, klesha filter) |
| P2 | ~15 | 沙盒 + 可觀測性 (sandbox, audit logger) |

### 3.3 SUSSMAN 三層配置架構

`[來源: DD-15, Cycle 02-8_a D3-Q3-R1, 23/24]`

SUSSMAN 設計的三層配置架構解決了「policy 值移出核心後放哪裡」的問題：

```
Layer 1: IAgentConfig        — 使用者/部署配置（最高優先）
Layer 2: SDK DEFAULT_*       — SDK 提供的合理預設值
Layer 3: Core zero-default   — Core 退化為零預設（0、空字串、空陣列）
```

**解析順序**: 使用者配置 > SDK 預設 > Core 零預設

這個設計的工程意義：

| # | 特性 | 說明 |
|---|------|------|
| 1 | Core 零污染 | Core 不包含任何 policy 常量，完全符合 Tenet #7 |
| 2 | SDK 提供合理預設 | 開發者不需要手動設定每個值，SDK DEFAULT_* 提供經過校準的預設 |
| 3 | 使用者可覆蓋 | IAgentConfig 允許部署時自訂所有 policy 值 |
| 4 | 關注點分離 | Core 只做機制，SDK 定義預設策略，使用者決定最終策略 |

**具體實作** (Plan32 交付):

```
packages/sdk/src/defaults/          — 11 個 DEFAULT_* 常量檔案
packages/sdk/src/types/agent.ts     — IAgentConfig 新增 mano, safety, execution,
                                      kleshaFilter, sandbox, confidenceAudit 等欄位
```

---

## 4. 純化邊界準則：何時移出，何時不移

### 4.1 移出核心的條件

當以下條件皆成立時，功能應從 Core 移出為 Plugin：

1. **BABBAGE 測試判定為 policy**: 移除後系統仍有數學定義
2. **介面契約不比功能本身更複雜**: 拆分後的 IPC/hook 介面設計合理
3. **存在合理的缺席行為**: Plugin 不存在時，系統有明確的退化方案

### 4.2 不應移出核心的情況

> 避免「為純化而純化」。

以下情況功能應保留在 Core：

1. **BABBAGE 測試判定為 mechanism**: 移除後系統數學失定義 (NaN、除零、型別不完整)
2. **介面契約比功能本身還複雜**: 拆分後的通訊協議開銷超過功能本身的複雜度
3. **純化的判斷標準是「機制還是策略」**: 而非「核心能不能再少一行」

### 4.3 三級關鍵性模型

`[來源: DD-16, Cycle 02-8_c, Master 裁定]`

Plugin 缺席時的行為依據三級模型分類：

| 級別 | 缺席行為 | Core 反應 | 範例 |
|------|---------|-----------|------|
| **Required** | 系統無法運行 | `throw Error()` at `start()` | Context manager (`@openstarry-plugin/context-sliding-window`) |
| **Optional (degraded)** | 功能降級但系統運行 | 中性值 (delta=0) | Auditor (`@openstarry-plugin/auditor-threshold`)、monitors |
| **Optional (no-effect)** | 功能不可用但無影響 | 靜默跳過 | VedanaSensors |

此模型確保純化不會導致系統脆弱化。Required 級別的 Plugin 必須安裝才能啟動；Optional 級別的 Plugin 缺席時系統自然退化到安全狀態。

---

## 5. Plan32 純化實施成果

Plan32 是 Tenet #7「微內核與絕對純淨」的工程實施，分六波完成。

### 5.1 六波結構

| 波次 | 內容 | 成果 |
|------|------|------|
| Wave 1 | Config 外部化 + Auto-mount 移除 | ThresholdAuditor 不再自動掛載，無 auditor 時 Core 正常運作 (delta=0) |
| Wave 2 | 內建 Plugin 抽出 (3 個新套件) | `@openstarry-plugin/auditor-threshold`、`auditor-passthrough`、`monitor-loop-quality` |
| Wave 3 | P0 Config 遷移 (安全+信心度) | 11 個 SDK DEFAULT_* 常量，IAgentConfig 新增 5 個欄位 |
| Wave 4 | P1+P2 Config 遷移 (執行+Klesha+沙盒+審計) | 7 個額外 DEFAULT_* 常量，~34 個 policy 值遷移 |
| Wave 5 | Audit Schema 擴展 | `audit:completed` 新增 `riskCategory` + `thresholdAtDecision` |
| Wave 6 | Context Manager 抽出 | `@openstarry-plugin/context-sliding-window`，Core 缺席時 throw Error |

### 5.2 新增 Plugin 套件

| 套件 | 關鍵性 | 說明 |
|------|--------|------|
| `@openstarry-plugin/context-sliding-window` | Required | 滑動視窗上下文管理，Core `start()` 缺席時 throw Error |
| `@openstarry-plugin/auditor-threshold` | Optional (degraded) | ThresholdAuditor，缺席時 delta=0 |
| `@openstarry-plugin/auditor-passthrough` | Optional (degraded) | PassthroughAuditor，用於 Phase 0 校準 |
| `@openstarry-plugin/monitor-loop-quality` | Optional (degraded) | 迴圈品質監控 |

### 5.3 驗證結果

| 指標 | 數值 |
|------|------|
| 測試數量 | 1803 (新增 33 個) |
| 測試檔案 | 167 個 |
| 失敗數 | 0 |
| 微核心純淨度檢查 | 通過 (僅 2 個已知 false positive) |

---

## 6. 微核心 vs 單體核心：AI Agent OS 場景比較

| 面向 | 微核心 (OpenStarry) | Monolithic |
|------|---------------------|------------|
| **IPC 效能開銷** | 可忽略 (LLM 延遲 >> IPC 延遲，比例 ~1:10,000) | 無 IPC (直接函數呼叫) |
| **隔離性** | 天然隔離，Plugin 間互不干擾 | 所有模組共享核心空間 |
| **攻擊面** | 小 (Core 僅 8 個 mechanism 值) | 大 (所有策略邏輯在核心空間) |
| **損害範圍** | 可控 (單一 Plugin 被攻破不擴散) | 可能全面擴散 |
| **Agent 不可信處理** | 天然適合 (隔離不可預測行為) | 難以隔離 (需額外機制) |
| **演化彈性** | 高 (新功能 = 新 Plugin) | 低 (功能變更影響核心) |
| **核心穩定性** | 高 (Core 小，易驗證) | 低 (Core 大，變更風險高) |
| **配置複雜度** | 較高 (多 Plugin 權限管理) | 較低 (統一管理) |
| **除錯複雜度** | 較高 (跨 Plugin 追蹤) | 較低 (單一空間除錯) |
| **Plugin 可組合性** | 高 (不同 Agent 選擇不同組合) | 低 (功能固定在核心) |
| **Tenet #7 合規** | 天然合規 (mechanism/policy 分離) | 難以合規 |

**結論**: 傳統 OS 微核心是「理想很好但代價太高」。OpenStarry 的場景剛好反過來——**代價幾乎為零 (IPC 不可見)，而好處全部成立 (隔離、演化、安全)**。微核心純化在 AI Agent OS 的語境下，是有充分工程根據的正確架構選擇。

---

## 7. 相關決議索引

| 決議 | 來源 | 說明 |
|------|------|------|
| DD-14: BABBAGE 連續性測試 | Cycle 02-8_a D3-Q1, 24/24 | Mechanism/Policy 分類準則 |
| DD-15: SUSSMAN 三層配置 | Cycle 02-8_a D3-Q3, 23/24 | IAgentConfig > SDK DEFAULT_* > Core zero-default |
| DD-16: 三級關鍵性模型 | Cycle 02-8_c, Master 裁定 | Required / Optional-degraded / Optional-no-effect |
| Master MR-4 | Cycle 02-8_a | Tenet #7 「絕對純淨」措辭不可修改 |
| Master MR-6 | Cycle 02-8_a | Core 必須零 policy 常量 |
| Plan32 | Cycle 02-8 | 六波實施，1803 tests，全部通過 |

---

## 8. 相關文件

| 文件 | 說明 |
|------|------|
| **Technical_Specifications/10** (Microkernel_Security_Analysis) | 微核心安全性完整分析 |
| **Doc 41** (Design_Decisions_and_Rationale) | DD-14, DD-15, DD-16 詳細記錄 |
| **Doc 44** (Safety_Architecture_Overview) | 安全架構概述 |
| **Doc 04** (Plugin_Infrastructure) | Plugin 基礎設施 |
| **Research_Methodology/04** (Ten_Tenets_Compliance_Report) | 十大宣言合規報告 |

---

*本文件由 KERNEL (#10) 撰寫。基於 Cycle 02-8 系列 Master 微核心純化討論、BABBAGE 連續性測試、SUSSMAN 三層配置架構、Plan32 工程交付成果。作為 OpenStarry 微核心架構選擇的技術論述文件。*
*安全面向的完整分析見 Technical_Specifications/10 (Microkernel Security Analysis)。*
