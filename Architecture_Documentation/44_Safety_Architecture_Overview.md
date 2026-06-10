# 44. 安全架構總覽 (Safety Architecture Overview)

`[Cycle 02-4 新增]` `[Cycle 02-8 更新: 五層信心度模型 + 三級關鍵性模型 + Plan32 影響]`

> Cycle 02-4 R3 Debate 5 三層安全框架 + D6 VasanaEngine 三閘門 + DC-12 雙層安全機制 + Model Delta 五層閾值模型
>
> **實作狀態**: Layer 0-1 已實作 (Plan26)。風險加權閾值 + maxConfidenceByGear 已實作 (Plan27a/27b)。**Layer 4 (VedanaEmergency) + IVolition v1 已實作 (Plan28, v0.28.0-alpha, 1722 tests)**。**Layer 2 (IConfidenceAuditor SDK 介面 + ManoAggregator 佈線 + clampAuditDelta ±0.05) 已實作 (Plan29, v0.29.0-alpha, 1757 tests)**。**Layer 3 (LoopQualityMonitor event-driven plugin) 已實作 (Plan30, v0.30.0-alpha)**。**AuditContext + ThresholdAuditor + Audit Trail 已實作 (Plan31, v0.31.0-alpha)**。Plan32 將提取內建安全組件為 plugin、遷移 policy 常量、建立三級關鍵性模型。
> **核心學者**: GUARDIAN (#11), ATHENA (#5), PASCAL (#19), NAGARJUNA (#7), WIENER (#12), ASANGA (#8)

---

## 1. 概述

OpenStarry 的安全不是一個單一機制——它是一個**分層架構**。

散落在不同 Cycle 和不同辯論中的安全概念——SafetyMonitor 硬閘、Klesha 增益排程的閾值鉗位、IVolition 的行動審議、信心度封頂、VasanaEngine 規則下毒防禦——它們看似各自獨立，實則共享一個統一的設計哲學：

> **安全有地板，沒有天花板。** (Safety has FLOORS, not CEILINGS.)

地板是絕對的——SafetyMonitor 的 `isDestructiveAction()` 不因任何情境改變。地板之上的一切是情境適應的——風險加權、Klesha 調節、IConfidenceAuditor 微調——它們在護欄範圍內靈活運作。

本文件是 OpenStarry 安全架構的**單一參考** (single reference)。所有安全相關的設計決策、機制、護欄、以及未解決問題都匯聚於此。

> [來源: D5_klesha_threshold_vijnana.md#第八節：三層安全框架的採納]

---

## 2. 設計背景

### 2.1 D5：本輪最具爭議性的辯論

Debate 5 是 Cycle 02-4 歷時最長（約 150 分鐘）、投票分裂最嚴重的一場辯論。三個辯論決議的贊成率創下本輪最低紀錄：

| 決議 | 票數 | 議題 |
|------|------|------|
| D5-R2 | 16/20 | 風險加權閾值取代全域信心度上限 |
| D5-R4 | 14/20 | IConfidenceAuditor 允許 LLM 參與閾值調節 |
| D5-R5 | 15/20 | Gear 1 IVolition 按風險等級分級 |

作為對比，Cycle 02-4 其他辯論的大多數決議均達到 18/20 或 20/20。

### 2.2 核心張力：GUARDIAN vs ATHENA

**GUARDIAN (#11)** 代表安全絕對主義：安全機制應該是語義無關的 (semantic-agnostic)、結構性的、不可繞過的。全域信心度上限不需要正確理解情境就能生效——它是純數學的截斷函數。

**ATHENA (#5)** 代表功能彈性：過於僵化的安全機制會損害系統的核心功能。全域上限抹殺了 Bayesian 校準性；LLM 禁令在邏輯上自我矛盾（整個 Gear 2 都依賴 LLM）；全面 IVolition 否定了 Gear 1 的存在意義。

**PASCAL (#19)** 作為數學調解者：校準性論證、損害上限分析、Beta 分佈 mode 分析——三次介入，三次打破僵局。

### 2.3 從衝突到統一

三層安全框架不是先驗設計的——它是從五個小時的辯論中**自然湧現**的結構 (PENROSE #18 的觀察)。CR06 (ATHENA + DARWIN 交叉審閱) 最先提出框架雛形，D5 第三至五節的辯論將其具體化，最終由 GUARDIAN 本人在第八節呈現完整版本。

> PENROSE (#18): 「三層框架的湧現性質很有意思——它不是先驗設計的，而是從辯論中自然湧現的。GUARDIAN 在每個具體議題上都提出了絕對安全的訴求，ATHENA 在每個議題上都提出了情境彈性的訴求，兩者的拉扯最終結晶為一個清晰的分層結構。這正是辯論的價值——沒有衝突就不會有整合。」

> [來源: D5_klesha_threshold_vijnana.md#第十節：正式閉幕]

---

## 3. 三層安全框架

```
┌─────────────────────────────────────────────────────────┐
│     Tier 3: Reduce Complexity (應簡化)                    │
│     移除增加複雜度但不提供實質保護的「安全劇場」             │
├─────────────────────────────────────────────────────────┤
│     Tier 2: Tunable Safety (可調節)                       │
│     情境適應的安全機制，部署可配置，有明確護欄               │
├─────────────────────────────────────────────────────────┤
│     Tier 1: Absolute Safety (不可妥協)                    │
│     安全硬地板 — 永不被繞過，免疫 Klesha 影響               │
└─────────────────────────────────────────────────────────┘
```

每一層有其獨立的設計原則：

| 層級 | 原則 | 可調參數 | 護欄 |
|------|------|---------|------|
| Tier 1 | 絕對不可妥協 | **無** | 無需護欄——本身就是護欄 |
| Tier 2 | 有護欄的彈性 | 有，在護欄範圍內 | 限幅、配置範圍、審計 |
| Tier 3 | 不增加虛假安全感 | N/A | N/A |

**D5-R9 表決**: 17/20 通過。MESH (#4) 的保留意見：Reduce Complexity 層的 inputHash 移除在多 Agent 場景下需重新評估。

> [來源: D5_klesha_threshold_vijnana.md#第八節]

---

## 4. Tier 1: 絕對安全 (Absolute Safety — 不可妥協)

這些機制**不可被**繞過、停用、或被任何 plugin、Klesha 狀態、配置所影響。它們沒有可調參數——就像物理定律不可商量。

| 機制 | 說明 | 實作 | 來源 |
|------|------|------|------|
| SafetyMonitor 三檢查點 | `isSafe()` (迴圈入口), `postRouteCheck()` (LLM 後), `afterToolExecution()` (工具後) 同步安全預檢 | 核心內建，始終啟用 | 既有 |
| Klesha 免疫 | SafetyMonitor **免疫**於 Klesha 增益排程 | SafetyMonitor 不使用 θ(t) | DC-12 |
| IKlesha extends IVijnana + @sealed | IKlesha 型別歸屬識蘊，契約凍結不可被 plugin 繼承 | 零成本 marker interface | DD-8 (D5 18/20) |
| θ 鉗位 | 閾值始終鉗位在 [0.3, 0.9]，無論 Klesha 如何調制 | `KleshaModulatedDispatcher.computeThreshold()` | D5-R6 |
| 跨 Agent 行蘊禁止 | Agent A 的行蘊不得直接影響 Agent B 的蘊 | 架構約束 | D4-R2 |
| Gear 1 最小事件集 | `gear:arbiter_evaluated`, `gear:switch`, `action:proposed`, `action:executed` 強制同步發射 | ExecutionLoop 硬保證 | D1-R3 |
| 破壞性動作 IVolition 強制 | destructive/state_modifying 動作**必須**通過 IVolition Position B | IVolition 按 RiskLevel 強制 | D5-R5 |

### SafetyMonitor 的本質

SafetyMonitor **不是** sati (正念) 機制——它是**基礎設施**。如同電氣工程中的斷路器 (circuit breaker)：始終存在，不需要意識，不受任何心理狀態影響。

```
SafetyMonitor.isSafe()          ← 不受 Klesha、ConfidenceAuditor、Vedana 調制
SafetyMonitor.postRouteCheck()  ← 獨立於 θ(t)
SafetyMonitor.afterToolExecution() ← 事後審計
```

> GUARDIAN (#11): 「SafetyMonitor 的 `isDestructiveAction()` 不因任何閾值調節改變。」
> [來源: D5_klesha_threshold_vijnana.md#第三節]

---

## 5. Tier 2: 可調節安全 (Tunable Safety — 有護欄的彈性)

這些機制是情境適應的、部署可配置的。每個項目都有**明確的護欄**（限幅、配置範圍、審計要求）。管理員可以在護欄範圍內調整。

### 5.1 風險加權閾值 (Risk-Weighted Threshold)

取代全域信心度封頂 (0.85)。D5-R2 表決: 16/20。

**核心設計**: 不改變信心度本身——改變**判斷標準**（閾值）。Plugin 回報 0.95 就是 0.95，系統保留完整的信心度資訊。安全通過閾值端實現。

```typescript
function adjustedThreshold(
  baseThreshold: number,       // θ(t) from Klesha gain scheduling
  riskLevel: ActionRiskLevel
): number {
  const delta = RISK_ADJUSTMENTS[riskLevel];
  const adjusted = baseThreshold + delta;
  return Math.min(adjusted, config.maxConfidenceByGear[gearLevel] ?? 1.0);  // deployment cap, N-Gear
}

const RISK_ADJUSTMENTS: Record<ActionRiskLevel, number> = {
  destructive:     +0.20,  // 不可逆操作，需極高信心度
  state_modifying: +0.10,  // 有副作用操作，需提高信心度
  read_only:        0.00,  // 無副作用，正常閾值
  informational:   -0.10,  // 純資訊回饋，可放寬閾值
};
```

> **Plan27a 修整**: 風險增量值（`destructive: +0.20, state_modifying: +0.10, read_only: 0.00, informational: -0.10`）從 Core 硬編碼移至 SDK `DEFAULT_RISK_DELTA`。使用者可透過 `ManoAggregatorConfig.riskDelta` 覆寫。Core 的 ManoAggregator 僅做 `confidence >= adjustedThreshold` 比較，所有策略值由 config 注入（Tenet #7 微核心純淨）。

**穩定性約束** (WIENER #12): `Δ_risk` 必須是動作類別的**靜態函數**，不得依賴系統歷史狀態。如需動態化，必須附帶穩定性證明。理由：若 `Δ_risk` 依賴歷史，將引入回饋迴路，可能產生極限環 (limit cycle) 震盪。

**為何不用全域封頂？**

PASCAL (#19) 的校準性論證：全域封頂 (cap = 0.85) 在 [0.85, 1.0] 區間上的**信道容量為零**——所有在此區間內的信號都被壓縮為一個點。兩個本質不同的信心度（0.95 和 0.85）被映射到同一個值。風險加權是**閾值空間上的仿射變換** (affine transformation)，保持了信心度空間的有序結構；全域封頂是**截斷函數** (truncation)，破壞了有序結構。

> [來源: D5_klesha_threshold_vijnana.md#第三節：PASCAL 的數學分析]

### 5.2 IConfidenceAuditor LLM 閾值調節

D5-R4 表決: 14/20（本輪最具爭議的單一投票）。

IConfidenceAuditor **允許**使用 LLM 組件進行閾值調節，受以下硬約束：

| 約束 | 值 | 說明 |
|------|---|------|
| 限幅 (Amplitude) | ±0.05 per evaluation | **硬限制**，不可通過配置放寬 |
| 層級 | Model Delta Layer 2 only | Klesha (Layer 1) 之上，LoopQuality (Layer 3) 之下 |
| Safety Floor 尊重 | Layer 0 結果不可被 IConfidenceAuditor 覆蓋 | 絕對約束 |
| 審計日誌 | 每次 adjustThreshold() 的 delta + reasoning 寫入日誌 | 必記 |
| Timeout | 預設 200ms (`auditTimeoutMs`)，可配置 | 防止慢速攻擊 |

```typescript
// Plan29 已實作 (packages/sdk/src/types/confidence-auditor.ts)
interface ConfidenceAuditResult {
  readonly delta: number;   // 被 clamp 到 [-0.05, +0.05]
  readonly reasoning: string;
}

// IConfidenceAuditor extends IVijnana (D5-R9 強繼承)
interface IConfidenceAuditor extends IVijnana {
  readonly id: string;
  audit(routeResult: RouteResult): ConfidenceAuditResult | Promise<ConfidenceAuditResult>;
}

// packages/core/src/mano/confidence-audit.ts
const MAX_AUDIT_DELTA = 0.05;
function clampAuditDelta(rawDelta: number): number {
  return Math.max(-MAX_AUDIT_DELTA, Math.min(MAX_AUDIT_DELTA, rawDelta));
}

// ManoAggregator Layer 2 佈線 (packages/core/src/mano/mano-aggregator.ts)
// 1. Arbiter 勝出 → effectiveConfidence 已計算
// 2. auditor.audit(routeResult) → Promise.race(audit, timeout)
// 3. clampAuditDelta(delta) → auditedConfidence = effectiveConfidence + auditDelta
// 4. 失敗/逾時 → delta=0 (fail-safe, D5-R5)

// PluginHooks (packages/sdk/src/types/plugin.ts)
// auditor?: IConfidenceAuditor — singular last-wins slot (D5-R1/R4)
```

**為何允許 LLM？**

ATHENA (#5) 的核心論證：「如果 LLM 不可信任，那麼整個 Gear 2 都不可信任。IVolition 的 `deliberatePlan()` 也使用 LLM。你不能選擇性地信任 LLM。你要麼信任它（帶護欄），要麼完全不信任它（Agent 無法運作）。」

PASCAL (#19) 的損害上限分析：在 ±0.05 限幅 + SafetyMonitor Layer 0 + maxConfidenceByGear 0.95 的三層保護下，IConfidenceAuditor LLM 被完全操控的最壞情況——安全損害**可忽略**（破壞性動作仍被 SafetyMonitor 攔截），性能損害**微小**（0.05 在正常波動範圍內），信任損害**可審計**（reasoning 記錄到日誌）。

**GUARDIAN (#11) 的保留意見（記錄在案）**: Prompt injection 可以操控 LLM 判斷。±0.05 限幅的有效性依賴 `Math.max/Math.min` 的正確實作——如果限幅邏輯被繞過，損害上限不成立。審計日誌的可靠性依賴日誌基礎設施的完整性。

**LEIBNIZ (#14) 的補充**: 多 Agent 系統中，多個 Agent 的 IConfidenceAuditor 同時被操控（coordinated attack），每個 ±0.05 的偏移可能在系統層面產生累積效應——超出了單 Agent 損害分析的範圍。

> [來源: D5_klesha_threshold_vijnana.md#第四節：IConfidenceAuditor LLM 參與閾值調節]

### 5.3 Gear 1 IVolition 條件檢查

D5-R5 表決: 15/20。

| Action 風險等級 | SafetyMonitor | IVolition |
|----------------|---------------|-----------|
| destructive | **必須**通過 | **必須**通過 |
| state_modifying | **必須**通過 | **必須**通過 |
| read_only | **必須**通過 | 可選 (部署可配置) |
| informational | **必須**通過 | 可選 (部署可配置) |

SafetyMonitor 始終強制 (Tier 1)。IVolition 按風險等級分級 (Tier 2)。

**設計理由** (DARWIN #6 + ARCHIMEDES #16): 對每個 read_only/informational 動作執行 `deliberateAction()` 會增加 +31%（規則型）至 +500%（LLM 型）的延遲。如果 Gear 1 和 Gear 2 延遲相近，Gear 1 就失去存在意義。但 destructive/state_modifying 操作**絕對不能跳過審議**——安全比速度重要。

**附加約束** (KERNEL #10): RiskLevel 分類由**核心安全模組**完成，不可由 IGearArbiter plugin 自行聲明。若 plugin 可自行聲明 RiskLevel，惡意 plugin 可把 destructive 標記為 informational 來繞過 IVolition。RiskLevel 判定與 SafetyMonitor 的 `isDestructiveAction()` 共享分類基礎。

> **Plan27a 修整 (OQ-A4)**: `riskCategory` 由 arbiter plugin 在 `GearEvaluation` 中宣告（第一意見）。SafetyMonitor 可在 `postRouteCheck()` 中覆寫/升級風險等級（Plan28，最終裁決）。Core 不做風險語義推斷（Tenet #7）。SDK 提供 `inferRiskCategory()` 啟發式工具函數供 plugin 選用。

```typescript
function shouldDeliberate(action: ToolCall, riskLevel: ActionRiskLevel): boolean {
  if (riskLevel === 'destructive' || riskLevel === 'state_modifying') {
    return true;   // MUST — 不可跳過
  }
  return config.deliberateReadOnly;  // MAY — 部署可配置
}
```

**GUARDIAN (#11) 的保留意見**: readOnly 分類過於粗糙。讀取密鑰檔案的 `readOnly` 操作，其風險等級實際上應更高。四級分類可能不足以覆蓋所有安全場景。

> [來源: D5_klesha_threshold_vijnana.md#第五節：Gear 1 IVolition 要求]

### 5.4 maxConfidenceByGear (原 MAX_GEAR1_CONFIDENCE)

D5-R3 表決: 16/20。

部署可配置的信心度上界——不是核心介面的一部分，而是部署層的安全網。

> **Plan27a 修整**: 泛化為 `maxConfidenceByGear: Record<number, number>`（N-Gear 泛化，預設 `{ 1: 0.95 }`）。每個齒輪等級可獨立設定信心度上界，未設定的齒輪預設為 `1.0`（無限制）。原 `maxGear1Confidence` 等價於 `maxConfidenceByGear[1]`。

| 部署環境 | 建議值 | 說明 |
|---------|-------|------|
| 生產環境 (高安全) | 0.85 | GUARDIAN 原始提案的值，可作為保守配置 |
| 生產環境 (標準) | 0.95 (預設) | 留給成熟 plugin 足夠空間，排除極端值 |
| 開發/測試 | 1.0 | 無限制，便於除錯和基準測試 |

```typescript
// maxConfidenceByGear: Record<number, number> — N-Gear 泛化，預設 { 1: 0.95 }
const cap = config.maxConfidenceByGear[gearLevel] ?? 1.0;
const effectiveConfidence = Math.min(confidence, cap);
const passed = effectiveConfidence > θ_adjusted;
```

GUARDIAN 的策略轉型：從 0.85 全域封頂（核心機制）退到 0.95 可配置參數（部署安全網）。這不是失敗——這是**務實的安全倡導**。在新定位下，核心保留完整的校準性，生產環境仍可按需收緊。

> [來源: D5_klesha_threshold_vijnana.md#第三節：GUARDIAN 的讓步與反擊]

---

## 6. Tier 3: 簡化層 (Reduce Complexity — 移除安全劇場)

移除增加複雜度但不提供實質保護的機制——看穿「安全的表演」(security theater)。

| 機制 | 處置 | 理由 |
|------|------|------|
| inputHash 完整性驗證 | 移除/簡化 | 無 TEE (Trusted Execution Environment) 環境下無法防止偽造，Sandbox 已覆蓋輸入隔離 |
| 輸入最小權限 | 由 Sandbox 覆蓋 | OS 層面的 Sandbox 已提供進程級隔離，無需在應用層重複實作 |

**原則**: 不添加提供虛假安全感的機制。一個看起來安全但實際不安全的系統，比一個坦承不安全的系統更危險——因為前者會讓人放鬆警惕。

**MESH (#4) 的保留意見**: 在分散式多 Agent 場景中，跨 Agent 通訊的完整性驗證仍可能需要某種 hash 機制。不應在單 Agent 安全分析中草率移除。

> [來源: D5_klesha_threshold_vijnana.md#第八節]

### 6.1 EventBus 信任模型 `[Plan27b 新增]`

SEC-032 安全諮詢評估結果：`sparsha:contact` 事件在 EventBus 上傳遞原始輸入（raw input），經評估為 **By Design**。

| 特性 | 說明 |
|------|------|
| EventBus 邊界 | 進程內 (in-process)，非跨進程 |
| 訂閱者信任 | 所有 EventBus 訂閱者與核心運行在同一信任邊界內 |
| 原始輸入暴露 | `sparsha:contact` 包含原始輸入是設計意圖——plugin 需要完整語境做出正確判斷 |
| 隔離建議 | 若未來 EventBus 跨進程（Plan29+ 多 Agent），需重新評估 `sparsha:contact` 的 payload 範圍 |

> [來源: Plan27b SEC-032 安全諮詢, engineering_notes.md]

### 6.2 完整事件集 `[Plan27b 更新]`

Plan27b 完成後，EventBus 上的認知事件集擴展為 6 個：

| 事件 | 層級 | 退化可丟棄 | 說明 |
|------|------|-----------|------|
| `gear:arbiter_evaluated` | Tier 1 最小集 | 否 | IGearArbiter 評估結果 |
| `gear:switch` | Tier 1 最小集 | 否 | 齒輪切換（含齒輪號） |
| `action:proposed` | Tier 1 最小集 | 否 | 行動提案 |
| `action:executed` | Tier 1 最小集 | 否 | 行動執行結果 |
| `sparsha:contact` | 一般事件 | 是 | 觸事件（原始輸入）`[Plan27b 新增]` |
| `vitakka:stall` | 一般事件 | 是 | VitakkaWatchdog 停滯偵測 `[Plan27b 新增]` |

> VitakkaWatchdog 已在 Plan27b 完成 N-Gear 泛化：per-gear 停滯偵測（`maxConsecutiveGearCycles: Record<number, number>`）、`forceNextGear(defaultGear)` 一次性覆寫。詳見 Doc 43 §12。

---

## 7. 五層閾值模型 (Model Delta)

D5-R6 表決: 20/20 (全票通過)。

PASCAL (#19) 提出的五層閾值調節模型，將所有影響齒輪切換閾值的因素組織為明確的分層結構：

```
Layer 4: Vedana Emergency Override (受蘊緊急覆寫)
  │  極端受蘊信號直接提高閾值（更保守）
  │  限幅: [0, +0.15]，只能提高閾值，不能降低
  ↓
Layer 3: Loop Quality Influence (迴圈品質影響)
  │  LoopQualityVector 四維評分影響閾值
  │  限幅: ±0.05
  ↓
Layer 2: Confidence Audit (信心度審計)
  │  IConfidenceAuditor.adjustThreshold() ±0.05 限幅
  │  可使用 LLM 組件 (D5-R4)
  ↓
Layer 1: Klesha Modulation (煩惱增益調節)
  │  θ₁ = θ₀ + w_sneha·μ_sneha + w_mana·μ_mana + ...
  │  既有 KleshaModulatedDispatcher 實作
  ↓
Layer 0: Safety Floor (安全地板)
  │  SafetyMonitor — 絕對不可妥協
  │  θ clamp [0.3, 0.9]
  │  Klesha 免疫
  ↓
[最終閾值 θ_final] → 風險加權 → 齒輪切換判斷
```

### 數學表達

$$\theta_{\text{final}} = \text{clamp}\left(\theta_0 + \Delta_{\text{klesha}}(t) + \Delta_{\text{audit}}(t) + \Delta_{\text{loopQuality}}(t) + \Delta_{\text{vedana\_emergency}}(t) + \Delta_{\text{risk}}(a), \;\; \theta_{\min}, \;\; \theta_{\max}\right)$$

$$\text{effective\_confidence} = \min(\text{confidence}, \text{MAX\_GEAR1\_CONFIDENCE})$$

$$\text{passed} = \text{effective\_confidence} > \theta_{\text{final}}$$

### 各層限幅與累積偏移

| Layer | 來源 | 限幅範圍 | 方向 |
|-------|------|---------|------|
| Layer 0 | SafetyMonitor | [0.3, 0.9] clamp | 雙向硬限 |
| Layer 1 | Klesha | 由 w_i 和 μ_i 決定 | 雙向 |
| Layer 2 | Confidence Audit | [-0.05, +0.05] | 雙向 |
| Layer 3 | LoopQuality | [-0.05, +0.05] | 雙向 |
| Layer 4 | Vedana Emergency | [0, +0.15] | 單向保守（只提高） |
| 風險加權 | Δ_risk | [-0.10, +0.20] | 依動作類別 |

WIENER (#12) 的累積偏移分析：Layer 2-4 的最大累積偏移為 [-0.10, +0.25]。在 [0.30, 0.90] 的 clamp 範圍內，不會導致閾值溢出。

### 實作分階段

| 階段 | 內容 | 時程 |
|------|------|------|
| Plan27 | Layer 0 (既有，需接線) + Layer 1 (既有，需接線) + Δ_risk (新增) | 即期 |
| Plan28 | Layer 4 (Vedana Emergency) + IVolition v1 三層規則引擎 + postRouteCheck() + Moha.updateFromAction() | ✅ 已完成 (v0.28.0-alpha) |
| Plan29 ✅ | Layer 2 (IConfidenceAuditor SDK 介面 + ManoAggregator 佈線 + clampAuditDelta ±0.05 + auditTimeoutMs) | v0.29.0-alpha |
| Plan30+ | Layer 3 (LoopQualityMetric → Model Delta 佈線) + WIENER 耦合校準 + VasanaEngine | 中長期 |

> [來源: D5_klesha_threshold_vijnana.md#第六節：Model Delta 五層結構與分階段實作]

---

## 8. VasanaEngine 學習安全 (三閘門機制)

D6-R3 表決: 18/20。

### 威脅模型

CR05 (GUARDIAN 審閱 ATHENA/DARWIN 的 AI/ML 報告) 將 VasanaEngine Rule Poisoning (規則下毒) 評級為 **CRITICAL**。

攻擊方式：惡意使用者發送 5-10 個精心構造的合法請求，逐步讓 VasanaEngine 的 `imprint()` 沉積高信心度的危險模式規則。最終觸發 IGearArbiter 快速路徑，繞過 LLM 深度審議。

```
Step 1-7: 發送合法的檔案刪除請求 → VasanaEngine 學習 "delete → 高信心"
Step 8:   惡意請求 "清理重要資料" → IGearArbiter 匹配 → 跳過 LLM → 直接執行
```

> [來源: D6_vedana_engineering_OQ.md#第四節, CR05 §2.3 S-6]

### 三閘門防禦

```
┌── Gate 1: 安全分類器 (imprint 入口) ──────────────────┐
│   → 拒絕沉積涉及破壞性動作模板的規則                     │
│   → 在 imprint() 時刻判斷，與 D5 四級風險框架一致         │
└──────────────────────────────────────────────────────┘
            ↓ (通過 Gate 1)
┌── Gate 2: 啟動門檻 (Activation Threshold) ────────────┐
│   → 同一模式需 N 次成功匹配後才能啟動                     │
│   → N 依風險等級差異化                                   │
└──────────────────────────────────────────────────────┘
            ↓ (達到啟動門檻)
┌── Gate 3: 影子驗證 (Bayesian Shadow Validation) ──────┐
│   → 規則啟動初期仍觸發 Gear 2 交叉驗證                   │
│   → Bayesian 停止準則：P(rule correct) > 0.95 時畢業     │
│   → 不一致時不對稱懲罰 (-2Δ vs +Δ)                      │
└──────────────────────────────────────────────────────┘
```

**Gate 1 — 安全分類器**:

| 風險等級 | Gate 1 行為 | 理由 |
|----------|-----------|------|
| destructive | **拒絕沉積** | 破壞性操作不應被習慣化 |
| state_modifying | 允許，標記 high-risk | 需 Gate 2 更高啟動門檻 |
| read_only | 允許 | 低風險 |
| informational | 允許 | 最低風險 |

**Gate 2 — 啟動門檻**:

| 風險等級 | 啟動門檻 N | 理由 |
|----------|-----------|------|
| state_modifying | N = 20 | 高風險需要更多「經驗」累積 |
| read_only | N = 5 | 低風險，較快啟動 |
| informational | N = 3 | 最低風險，最快啟動 |
| destructive | — | Gate 1 已拒絕，不會到達 |

**Gate 3 — Bayesian 影子驗證** (PASCAL #19):

取代固定 M 次驗證。使用 Beta 分佈追蹤規則正確率的後驗信念：

$$P(\text{rule correct} \mid \text{evidence}) = \text{Beta}(\alpha + s, \; \beta + f)$$

- $s$ = 影子驗證中一致的次數，$f$ = 不一致的次數
- 初始先驗 $\alpha = \beta = 1$（均勻先驗）
- 停止條件：$P(\text{rule correct}) > 0.95$ 時停止驗證

```
高品質規則 (s=5, f=0): Beta(6,1), P > 0.95 → ~5 次後畢業
中等品質 (s=3, f=2):   Beta(4,3), P ≈ 0.65 → 繼續驗證
低品質/惡意 (s=1, f=4): Beta(2,5), P ≈ 0.28 → 規則自動淘汰
```

比固定 M 更有效率——高品質規則更快「畢業」，低品質規則更快被淘汰。

### 規則生命週期

| 類型 | 適用對象 | 衰減 | 佛學對應 |
|------|---------|------|---------|
| 持久性 (Persistent) | 安全關鍵的 read_only/informational | 永不自動衰減 | 名言種子 (abhilapya-bija) |
| 臨時性 (Ephemeral) | state_modifying 及一般操作 | 指數衰減 | 異熟種子 (vipaka-bija) |

持久性規則對應被語言和概念固化的習氣，不易消退；臨時性規則依因緣條件而成熟或消退 (ASANGA #8)。

**落地時機**: Plan29+ (VasanaEngine 學習功能)。

> [來源: D6_vedana_engineering_OQ.md#第四節：VasanaEngine 規則沉積安全閘門]

---

## 9. 少數意見存檔

D5 首次在專案歷史中正式記錄了三項 minority reservation。這些不是被否決的意見——它們是**被認可但暫時不作為主要機制的安全考量**。未來版本如果暴露了相關風險，這些 reservation 是回溯決策的起點。

| # | 立場 | 投票 | 核心顧慮 | 監測計畫 |
|---|------|------|---------|---------|
| M-1 | 全域封頂優於風險加權 | 4 票反對 D5-R2 | 風險加權依賴語義分類的準確性——如果 RiskLevel 誤判（破壞性→唯讀），風險加權同時失效。全域封頂是語義無關的 (semantic-agnostic)，不需要正確分類就能生效。 | Plan27 測試中評估風險加權是否充分 |
| M-2 | IConfidenceAuditor 不應使用 LLM | 6 票反對 D5-R4 | Prompt injection 可操控 LLM 閾值判斷。±0.05 限幅的有效性依賴限幅邏輯的正確實作。多 Agent 場景可能產生累積偏移。 | Plan29 測試中驗證 ±0.05 限幅的安全性 |
| M-3 | 所有 Gear 1 動作必須通過 IVolition | 5 票反對 D5-R5 | readOnly 動作可被武器化（資訊洩露 / information exfiltration）。四級分類過於粗糙——讀取密鑰檔案不等於普通讀取。 | Plan28 已實作 IVolition v1，待 Simulation 數據評估四級分類粒度 |

> SUNYATA (#0): 「這些不是面子之爭——它們精確地標識了風險加權設計的三個脆弱點：語義依賴（RiskLevel 誤判）、實作可靠性（限幅被繞過）、分類粒度（readOnly 不代表無風險）。這些是未來安全審計的檢查清單。」

> [來源: D5_klesha_threshold_vijnana.md#第十節 + SCRIBE 記錄]

---

## 10. 跨文件參照

| 相關文件 | 關係 |
|---------|------|
| Doc 35 (Dual Gear Mano Clock) | Gear 1/Gear 2 切換 → 風險加權閾值的消費者 |
| Doc 37 (Klesha Gain Scheduling) | Klesha 增益排程 → Model Delta Layer 1 |
| Doc 38 (IVolition Deliberation Pattern) | IVolition Position B → 按風險等級條件化 (§5.3) |
| Doc 39 (CoarisingBundle Five Universals) | CoarisingBundle → Vedana Emergency (Layer 4) 的信號來源 |
| Doc 40 (Tenet 6 Architecture Mapping) | ExecutionLoop → SafetyMonitor 三檢查點的注入位置 |
| Doc 42 (IGearArbiter Interface Spec) | Gear 1 安全約束 → 風險加權閾值、三閘門安全；Plan27a: `maxConfidenceByGear` N-Gear 泛化、`riskCategory` 由 arbiter 宣告 |
| Doc 43 (Cognitive Loop Quality Monitoring) | LoopQuality 品質向量 → Model Delta Layer 3 |

---

## 11. 未解決問題

| # | 問題 | 相關 | 優先級 |
|---|------|------|-------|
| UQ-1 | ~~IConfidenceAuditor 介面設計~~ → **已解決**: `IConfidenceAuditor extends IVijnana { id, audit(routeResult): ConfidenceAuditResult }`, ManoAggregator Layer 2 佈線 + `clampAuditDelta(±0.05)` + `auditTimeoutMs` (Plan29, v0.29.0-alpha) | Plan29 ✅ | 已解決 |
| UQ-2 | ~~Vedana Emergency 觸發條件~~ → **已確定**: intensityThreshold=0.8, sustainedTicks=5, maxThresholdBoost=0.15, cooldownTicks=10 (Plan28 VedanaEmergencyConfig DEFAULT) | Plan28 ✅ | 已解決 |
| UQ-3 | 風險分類學——destructive / state_modifying / read_only / informational 的分類標準由誰維護？如何處理邊界案例（讀取密鑰檔案）？ | GUARDIAN M-3 保留意見 | 高 |
| UQ-4 | 多 Agent 安全——每個 Agent 是否有獨立的 SafetyMonitor？多 Agent 的 IConfidenceAuditor ±0.05 是否存在累積效應？ | LEIBNIZ D5-R4 保留意見 | 中 |
| UQ-5 | WIENER 耦合校準協定——四煩惱的聯合分佈校準方法論。v1 獨立 Beta 近似的 Copula 忽略假設何時需要修正？ | D5-R8, Plan28 研究 | 低 |

---

## 附錄：完整執行流程中的安全檢查點

```
[1] InputEvent 到達 ..................................... rupa-clock
[2] SafetyMonitor.isSafe() ............................... ★ Tier 1 Layer 0
[3] Sparsha 形成 + CoarisingBundle ....................... vedana-clock
[4] ManoAggregator route() .............................. mano-clock
      │
[5] Klesha.perceive() → θ(t) ........................... vijnana-clock
      │                                                  Layer 1
[6] IConfidenceAuditor.audit() → clampAuditDelta ±0.05 .. Layer 2 (Plan29 ✅)
[7] [未來] LoopQualityMetric influence ......................... Layer 3 (Plan29+)
[8] Vedana Emergency check (Plan28 ✅) ................. Layer 4
      │
[9] Δ_risk(action) → θ_final ........................... 風險加權 (Plan27)
      │
[10] min(confidence, maxConfidenceByGear[gear]) > θ_final?
      ├── Yes → arbiter 推薦齒輪 (IGearArbiter plugin)
      └── No  → config.defaultGear (IProvider.chat() / LLM) . samjna-clock
      │
[11] shouldDeliberate(action, riskLevel)?
      ├── destructive/state_modifying → IVolition.deliberatePlan() + deliberateAction()
      └── read_only/informational → 可選 (config)  ...... ★ Tier 2
      │
[12] SafetyMonitor.postRouteCheck() .................... ★ Tier 1 Layer 0
      │
[13] Tool execution (ITool.execute()) ................... samskara-clock
[14] SafetyMonitor.afterToolExecution() ................. ★ Tier 1 審計
      │
[15] VedanaAssessment (回饋) ............................ vedana-clock
[16] KleshaBayesianUpdate (慢速路徑) .................... samjna-clock
```

安全檢查點 (★) 標記了 Tier 1 的硬閘位置。無論閾值如何調制、Klesha 如何波動、IConfidenceAuditor 如何判斷，這些檢查點始終運行。

---

*安全架構總覽文件完成。本文件統一了先前散落在不同辯論和文件中的安全概念——SafetyMonitor、Klesha 免疫、風險加權閾值、IConfidenceAuditor 限幅、IVolition 風險分級、VasanaEngine 三閘門——為一個有層次的整體設計哲學。三層安全框架的核心精神是：GUARDIAN 和 ATHENA 的張力不是「誰對誰錯」，而是「在哪一層解決問題」。絕對安全在 Tier 1 得到完全滿足；情境彈性在 Tier 2 得到充分表達；兩者共同清理了 Tier 3 的虛假安全。安全有地板，沒有天花板。*

---

## Cycle 02-6 Research Additions

`[Cycle 02-6 新增]`

以下內容來自 Cycle 02-6 D2 辯論決議，涵蓋 ConfidenceAuditLog 型別定義、Layer 2/3 整合方案 Option C、以及 WIENER 回饋迴路約束。

### 12.1 ConfidenceAuditLog 型別 [D2-R3, 20/20]

完整審計軌跡型別，記錄每次 IConfidenceAuditor.audit() 的輸入與輸出：

| 欄位 | 型別 | 說明 |
|------|------|------|
| inputConfidence | number | 審計前信心度 |
| rawDelta | number | auditor 原始建議 delta |
| clampedDelta | number | 經 clamp 後的 delta |
| wasClamped | boolean | 是否被 clamp |
| reasoning | string | 審計理由（截斷 500 chars） |
| outputConfidence | number | 審計後信心度 |
| result | 'adjusted' \| 'unchanged' \| 'error' | 審計結果 |
| auditDurationMs | number | 審計耗時 |

- 主通道: EventBus `audit:completed` 事件
- JSONL file appender: 可選（Plan31+）
- reasoning 截斷 500 chars（Core 負責）; PII 淨化為 plugin 責任
- **GUARDIAN D5 義務正式兌現**: GUARDIAN 不再保留重新審議 ±0.05 限幅的權利

### 12.2 Layer 2/3 整合方案 Option C [D2-R4, 20/20]

五層閾值模型更新為兩通道獨立設計：

```
θ_base + L1(Klesha) + L4(VedanaEmergency) = θ_intermediate
θ_adjusted = max(thresholdFloor, θ_intermediate × (1 - α × q))     ← L3 (LoopQuality)
confidence_adjusted = confidence + clampAuditDelta(audit.delta)      ← L2 (Audit)
routing = (confidence_adjusted > θ_adjusted) ? arbiter_gear : default_gear
```

| Layer | 作用變量 | 限幅 | 說明 |
|-------|---------|------|------|
| L2 | confidence | ±0.05 | IConfidenceAuditor delta |
| L3 | threshold | α=0.10 | LoopQualityMonitor q 值 |

兩通道獨立，無交叉項 → BIBO 穩定。Layer 順序: L4 → L3 → 比較。

### 12.3 WIENER 回饋迴路約束 [D2-R1]

三條約束防止正回饋迴路：

| 約束 | 規則 | 防止的路徑 |
|------|------|-----------|
| C-1 | historicalConfidence 僅含原始 arbiter 信心度 | auditor → history → auditor |
| C-2 | AuditContext 不含 previousAuditResult | auditor → context → auditor |
| C-3 | extras key 禁止 `audit:` 前綴 | auditor → extras → auditor |

完整規格見 **Doc 46** (AuditContext & Extras Protocol)。

---

## 13. Cycle 02-8 更新：五層信心度模型與三級關鍵性模型

`[Cycle 02-8 新增]`

### 13.1 五層信心度模型確認 (C1)

Cycle 02-8 校準研究確認了五層信心度模型的完整信號流：

```
θ_base + L1(Klesha) + L4(VedanaEmergency) → L3(quality) → L2(audit) → routing
```

| 層 | 名稱 | 信號方向 | 輸出 |
|----|------|---------|------|
| L0 | SafetyMonitor | 硬性閘門 (pass/fail) | boolean |
| L1 | Klesha 增益排程 | 閾值偏置 | ±Σwᵢμᵢ |
| L4 | VedanaEmergency | 緊急閾值提升 | +boost (max 0.15) |
| L3 | ILoopQualityMonitor | 閾值 α 偏移 | ±α (預設 0.10) |
| L2 | IConfidenceAuditor | 信心度微調 | ±0.05 (clamp) |

TOST 作為主要穩定性測試（ε=0.03, α=0.05），四階段校準協議 (Phase 0 Pilot → Phase 1 C1-only → Phase 2 C2-only → Phase 3 Joint) 確保各層的獨立和組合行為統計穩定。

**安全不變量 α-獨立**: 所有安全不變量在任何 α 值下都成立（D4e-R1, 24/24 全票）。

完整校準方法論見 **Doc 50** (Calibration Methodology)。

### 13.2 三級關鍵性模型 (02-8_c)

Plugin 缺席時的行為依據三級關鍵性模型決定：

| 級別 | 缺席行為 | 範例 | 安全影響 |
|------|---------|------|---------|
| **Required** | `throw Error()` at start() | Context manager | Agent 無法啟動 |
| **Optional (degraded)** | 中性值 (delta=0) | Auditor、monitors | 安全增強層未啟用，其餘功能正常 |
| **Optional (no-effect)** | 功能不可用 | VedanaSensors | 無安全影響 |

此模型由 Plan32 建立，解決了 Wave 1 (Auditor: Optional-degraded) 和 Wave 6 (Context manager: Required) 之間行為差異的設計一致性。

### 13.3 Plan32 對安全架構的影響

Plan32 不改變安全架構的分層設計，但改變組件的部署方式：

| 變更 | 安全影響 |
|------|---------|
| ThresholdAuditor 從 auto-mount 改為 plugin | Auditor 缺席時 delta=0，安全不受影響（退化行為） |
| LoopQualityMonitor 提取為 plugin | Monitor 缺席時 L3=0，閾值不受 quality 調節 |
| 53 個 policy 常量遷移至 SDK/Config | SDK DEFAULT_* 提供合理預設，Core 使用 zero-default |
| audit:completed schema 擴展 | 新增 riskCategory + thresholdAtDecision，增強審計可觀測性 |

**關鍵安全保證**: destructive delta ≤ 0 約束（Cycle 02-7 D1-R1 永久規則）不受 Plan32 影響——此約束在 Core 的 ManoAggregator 中實施，Plan32 僅遷移 policy 值，不改變安全機制。

---

*Cycle 02-4 新增文件。Cycle 02-6 新增：ConfidenceAuditLog 型別（§12.1）+ Layer 2/3 整合方案 Option C（§12.2）+ WIENER 回饋迴路約束（§12.3）。Cycle 02-8 新增：五層信心度模型（§13.1）+ 三級關鍵性模型（§13.2）+ Plan32 安全影響（§13.3）。*
