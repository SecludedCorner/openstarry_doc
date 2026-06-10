<!-- Status: CURRENT -->
<!-- Layer: 1-Engineering -->
<!-- Applies to: v0.34.0-alpha -->
<!-- Last verified: 2026-03-16 -->

# 53. 微核心安全分析 (Microkernel Security Analysis)

`[Cycle 02-9 新增]`

> 本文件記錄 OpenStarry 微核心架構的安全特性分析。涵蓋安全優勢、安全限制、威脅模型分階段評估、架構隔離與契約隔離的區分、必要配套措施、以及 AI Agent 特有的安全考量。
> 本文件與 **Doc 50** (Microkernel Purification Rationale) 互補——Doc 50 回答「為什麼選微核心」和「如何純化」，本文件聚焦於安全面向。

---

## 1. 核心結論

**微核心不是無條件更安全，而是「攻擊面更小、損害範圍更可控」。**

實際安全性取決於 IPC 安全設計、加密機制、權限配置等配套措施。微核心提供的是更好的安全架構基礎，而非安全保證。

---

## 2. 微核心的安全優勢

### 2.1 最小權限原則天然成立

每個 Plugin/服務只有自己需要的權限。不像 monolithic 裡所有模組共享核心權限。一個 Plugin 被攻破，它只能存取自己的資料，拿不到其他 Plugin 的資料。

### 2.2 攻擊面縮小

核心程式碼越少，可被利用的漏洞越少。seL4 微核心已通過形式化驗證——程式碼少到可以數學證明無漏洞。OpenStarry 的 Core 在 Plan32 後僅保留 8 個 mechanism 值，大幅減少了需要審計的核心程式碼量。

### 2.3 故障隔離 = 洩漏隔離

Plugin A 的機密不會因為 Plugin B 出問題而暴露。在 monolithic 架構中，一個模組的漏洞可能讓攻擊者存取整個核心空間的資料。

### 2.4 AI Agent 特殊適用性

Agent 會接觸使用者的敏感資料 (API key、個人資訊等)。微核心架構讓每個 Agent 的 Plugin 互相隔離——一個 Agent 被 prompt injection 攻破不會波及其他 Agent 的資料。LLM 的行為不可完全預測，隔離的價值在 AI Agent 場景下比傳統 OS 更大。

---

## 3. 微核心的安全限制

### 3.1 IPC 通道本身是攻擊面

Plugin 之間的訊息傳遞如果沒有加密或驗證，資料在傳輸中可能被截取。Monolithic 裡函數直接呼叫反而沒有這個問題。OpenStarry 需要在 IPC 層設計適當的驗證與加密機制。

### 3.2 隔離不等於加密

微核心保證「你碰不到別人的空間」，但不保證「你自己的資料是加密的」。資料安全還需要額外的加密機制，這跟架構選擇無關。

**具體範例**: 一個 Agent 的 `IAgentConfig` 可能包含 LLM API key。微核心隔離確保 Plugin B 無法*直接存取* Plugin A 的配置空間。但如果 Plugin A 將 API key 以明文儲存於其本地狀態，而 Plugin A 的審計日誌透過 `audit:completed` 事件發送該狀態，則 API key 可能出現在 auditor plugin 可讀取的日誌中。隔離防止*直接存取*，但無法防止*間接洩漏* (透過共享通道如 event bus、logs、audit trails)。

### 3.3 配置錯誤風險

Plugin 數量增加後 (Plan32 後 Plugin Catalog 從 22 增至 28 個)，權限配置的複雜度也隨之增加。配置疏漏可能引入安全缺口。

此外，SUSSMAN 三層配置架構將信任委派給 SDK 層——11 個新的 `DEFAULT_*` 檔案和 5+ 個新的 `IAgentConfig` 欄位構成了新的配置攻擊面。例如，若 SDK 預設值被設為不安全的值 (如 `DEFAULT_SAFETY_MONITOR_CONFIG.maxDestructiveDelta = 1.0`)，將實質上停用安全不變量。SDK 層必須被視為安全敏感程式碼。

---

## 4. 架構隔離 vs 契約隔離

### 4.1 關鍵區分

隔離有兩種本質不同的類型，其安全屬性截然不同：

| 類型 | 定義 | 強度 | 攻破方式 |
|------|------|------|---------|
| **架構隔離 (Architectural)** | 由 process boundary 或硬體機制強制執行 | 強 — 需要利用 OS/硬體漏洞 | kernel exploit, side-channel |
| **契約隔離 (Contractual)** | 由程式碼慣例、API 邊界、型別系統執行 | 中 — 靠遵循約定 | 直接記憶體存取、無視 API 邊界 |

### 4.2 OpenStarry 現狀：契約隔離

OpenStarry v0.32 在單一 Node.js process 中運行所有 Plugin。Plugin 之間的隔離是**契約隔離**——靠 TypeScript 型別系統、API 邊界設計、及 PluginLoader 的 `skandha` 宣告機制維持。

這不是缺陷。在單一 process 模型下，契約隔離加上 TypeScript 的型別系統提供了合理的保護層級。但文件讀者不應誤以為存在 process-level 的架構隔離。

### 4.3 隔離演化路徑

| 階段 | 隔離類型 | 說明 |
|------|---------|------|
| v0.32 (現狀) | 契約隔離 | TypeScript 型別 + API 邊界 + PluginLoader 管理 |
| 未來 (multi-process) | 架構隔離 | Plugin 跨 process/container，由 OS 強制隔離 |

微核心架構的設計使得從契約隔離升級到架構隔離成為可能——Plugin 已經透過 IPC 介面通訊，遷移到跨 process 模型不需要重新設計通訊協議。

---

## 5. 威脅模型：分階段評估

`[來源: GUARDIAN R2 supplement]`

### 5.1 Current Threat Level (v0.32)

| 面向 | 風險 | 說明 |
|------|------|------|
| IPC 攔截 | **低** | 單一 Node.js process，in-memory dispatch，無網路層攔截風險 |
| Plugin 互相存取 | **中** | 契約隔離可被 JS runtime 層繞過（共享記憶體空間） |
| 配置竄改 | **低** | SDK DEFAULT_* 在 monorepo 中，受版本控制保護 |
| Prompt injection | **高** | LLM 本質風險，與架構選擇無關 |

**現階段優先措施** (已完成或已配置):
- Audit mechanism: `audit:completed` event (Plan32 Wave 5)
- Sandbox resource limits: `DEFAULT_SANDBOX_MANAGER_CONFIG` (已配置)

### 5.2 Near-term Threat Level (v0.33-v0.35)

| 面向 | 風險 | 說明 |
|------|------|------|
| IPC 攔截 | **低** | 仍為單一 process |
| 配置攻擊面 | **低->中** | Plugin 數量增加，IAgentConfig 欄位增加 |
| Plugin 供應鏈 | **低** | Plugin 在 monorepo 中，但隨 Plugin 生態擴展風險上升 |

**近期優先措施**:
- Permission hardening: Plan34 `.openstarry/` project-level scope
- SDK DEFAULT_* 安全審閱流程

### 5.3 Future Threat Level (multi-process)

| 面向 | 風險 | 說明 |
|------|------|------|
| IPC 攔截 | **高** | Plugin 跨 process/container，IPC 經過網路層 |
| 配置竄改 | **中** | 分散式配置管理引入新的攻擊向量 |
| Plugin 身份偽造 | **中->高** | 遠程 Plugin 需要身份驗證 |

**未來優先措施**:
- IPC encryption + message authentication (HMAC)
- Plugin identity verification (cryptographic signing)

> **OPEN ITEM: Plan36 scope** — Plugin 身份驗證 (cryptographic signing / integrity check) 列為 Plan36 (Advanced Security) 研究範圍。目前 Plugin 透過 PluginLoader 以 `skandha` 宣告載入，無密碼學簽章或完整性檢查。

---

## 6. AI Agent 特有的安全考量

### 6.1 Prompt Injection 隔離

Prompt injection 是 AI Agent 系統最顯著的安全威脅。攻擊者透過精心設計的輸入，操控 LLM 產生非預期行為。

微核心架構對 prompt injection 的防護：

| 面向 | 微核心效果 | 說明 |
|------|-----------|------|
| **損害範圍限制** | 有效 | 被注入的 Agent 只能操作自己的 Plugin 範圍，無法存取其他 Agent 的資料或工具 |
| **防止注入本身** | 無效 | 微核心架構不處理 LLM 輸入層面的防護——需要 IGuide prompt shaping、IVolition deliberation 等機制 |
| **事後偵測** | 間接有效 | Audit mechanism (`audit:completed`) 記錄所有決策鏈，可事後分析異常行為模式 |

### 6.2 Agent 間資料隔離

在多 Agent 場景下，不同 Agent 處理不同使用者的敏感資料。微核心架構確保：

- 每個 Agent 的 Plugin 組合獨立
- Agent A 的 context (對話歷史、API key 等) 不可被 Agent B 的 Plugin 存取
- Agent 間協調必須透過正式的 Coordination Daemon 通道，不能直接存取彼此狀態

### 6.3 不確定性行為的安全邊界

LLM 的輸出具有本質不確定性。微核心的三層安全防護處理此問題：

| 層級 | 機制 | 說明 |
|------|------|------|
| **Layer 0** | SafetyMonitor | 硬性安全閘門，獨立於 Plugin 層，任何 Alpha 下安全成立 (D4e-R1) |
| **Layer 1** | IVolition deliberation | Plugin 層的審議機制，可 veto 危險行動 |
| **Layer 2** | Risk-weighted threshold | 依 action 危險等級動態調節 (DD-9) |

SafetyMonitor 在 Core 內且不可被 Plugin 覆寫——即使 Plugin 被完全攻破，Layer 0 的安全不變量仍然成立。這是微核心架構的重要安全屬性：安全機制 (mechanism) 在 Core 中，安全策略 (policy) 在 Plugin 中，但策略無法覆寫機制。

---

## 7. 必要的配套安全措施

微核心架構基礎需要以下配套才能達成完整的安全保障：

| # | 層面 | 需求 | 現狀 | 優先級 |
|---|------|------|------|--------|
| 1 | IPC 安全 | 訊息驗證與加密 | 未實作 (單一 process 下非必要) | 未來 (multi-process 時) |
| 2 | 權限管控 | 最小權限配置 | 部分 (PluginLoader skandha 宣告) | Plan34 強化 |
| 3 | 審計機制 | 操作日誌追蹤 | 已完成 (`audit:completed`, Plan32 Wave 5) | - |
| 4 | 沙盒隔離 | 資源限制 | 已配置 (`DEFAULT_SANDBOX_MANAGER_CONFIG`) | - |
| 5 | 安全閘門 | SafetyMonitor | 已實作 (Layer 0, Core 內) | - |
| 6 | Plugin 身份驗證 | 密碼學簽章 | 未實作 | Plan36 (OPEN ITEM) |
| 7 | SDK 安全審閱 | DEFAULT_* 值安全性檢查 | 未建立流程 | 近期建議 |

### 7.1 優先次序

依據威脅模型的分階段評估，配套措施的優先次序為：

1. **已完成**: Audit mechanism、sandbox limits、SafetyMonitor (v0.32)
2. **近期**: Permission hardening (Plan34)、SDK DEFAULT_* 安全審閱流程
3. **未來**: IPC encryption、Plugin identity verification (Plan36+)

此優先次序反映一個務實原則：在單一 process 模型下，IPC 加密不是必要的——真正的攻擊面在配置錯誤和 prompt injection，而非 IPC 攔截。

---

## 8. 微核心 vs 單體核心：安全面向比較

| 安全面向 | 微核心 (OpenStarry) | Monolithic |
|---------|---------------------|------------|
| **攻擊面大小** | 小 — Core 僅 8 個 mechanism 值 | 大 — 所有策略邏輯在核心空間 |
| **損害範圍** | 可控 — 單一 Plugin 被攻破不擴散 | 可能全面擴散 |
| **最小權限** | 天然成立 — Plugin 只有自己的權限 | 需額外機制 |
| **故障隔離** | 天然成立 — Plugin 獨立 | 不成立 — 共享核心空間 |
| **IPC 安全** | 需額外設計 — IPC 通道是攻擊面 | 無此問題 — 直接函數呼叫 |
| **配置複雜度** | 較高 — 多 Plugin 權限管理 | 較低 — 統一管理 |
| **安全審計範圍** | 小 — Core 程式碼量少 | 大 — 核心程式碼量大 |
| **Prompt injection 隔離** | 有效 — 損害限制在被攻破的 Agent 範圍 | 困難 — Agent 共享核心空間 |
| **安全機制可驗證性** | 高 — Core 小到可逐行審計 | 低 — Core 大，審計成本高 |
| **重複實作風險** | 存在 — 多 Plugin 可能各自實作類似安全邏輯 | 較低 — 統一實作 |

---

## 9. 相關決議索引

| 決議 | 來源 | 安全相關性 |
|------|------|-----------|
| DD-14: BABBAGE 連續性測試 | Cycle 02-8_a D3-Q1, 24/24 | 確保安全相關的 mechanism 值不被移出 Core |
| DD-16: 三級關鍵性模型 | Cycle 02-8_c, Master 裁定 | 定義 Plugin 缺席時的安全退化行為 |
| DD-9: 風險加權閾值 | Cycle 02-4 D5, 16/20 | 安全性與學習天花板的平衡 |
| DD-13: 三層規則優先序 | Plan28 D6, OQ-1 | Hard > Soft > Heuristic，安全底線不可覆寫 |
| Master MR-6 | Cycle 02-8_a | Core 零 policy — 安全機制留在 Core，安全策略在 Plugin |
| D4e-R1: 安全不變量 alpha-獨立 | Cycle 02-8, 24/24 | 任何 alpha 下安全成立 |

---

## 10. 相關文件

| 文件 | 說明 |
|------|------|
| **Doc 50** (Microkernel_Purification_Rationale) | 微核心純化原理 (架構選擇、BABBAGE 測試、Plan32) |
| **Doc 44** (Safety_Architecture_Overview) | 安全架構概述 (SafetyMonitor、三層安全框架) |
| **Doc 41** (Design_Decisions_and_Rationale) | 設計決策記錄 (DD-9, DD-13, DD-14, DD-16) |
| **Research_Methodology/04** (Ten_Tenets_Compliance_Report) | 十大宣言合規報告 |
| **Doc 04** (Plugin_Infrastructure) | Plugin 基礎設施 |

---

*本文件由 KERNEL (#10) 撰寫，整合 GUARDIAN (#11) 在 R2 交叉審閱中提出的三項安全補充 (IPC threat model 分階段、architectural vs contractual isolation、isolation != encryption concrete example)，以及 R3 D4-1 決議的分拆要求。*
*作為 OpenStarry 微核心架構安全面向的完整分析文件。*
