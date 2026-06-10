# 38. IVolition 雙階段審議模式 (Two-Phase Deliberation Pattern)

`[Cycle 02-4 修整]`

**來源**: R3 Debate 4 共識 (20/20 無異議) + Cycle 02-2 A-2 修正 + Cycle 02-4 VasanaEngine 外部化
**日期**: 2026-02-25
**實作狀態**: ✅ 已實作 (Plan26 介面 + Plan28 策略, v0.28.0-alpha, 1722 tests) — `sdk/types/volition.ts` (IVolition 介面) + `core/execution/loop.ts` (Position B 雙階段審議) + `@openstarry-plugin/volition-rule-engine` (Plan28: 三層規則引擎)。
**核心學者**: ASANGA (#8), KERNEL (#10), TURING (#17), WIENER (#12), GUARDIAN (#11), BABBAGE (#9)
**依賴文件**: #29 五蘊連結, #30 觸→俱生模型, #31 八識俱轉

---

## 一、概述

IVolition.deliberate() 是識蘊 (vijnana) 架構中的**意志審議介面**，映射佛學「思」(cetana)。
它不是單一方法，而是**雙階段架構**：

| 階段 | 方法 | 粒度 | 頻率 |
|------|------|------|------|
| Phase 1 | `deliberatePlan()` | 整體計畫 (全部 ToolCall) | 每個 mano-cycle 一次 |
| Phase 2 | `deliberateAction()` | 個別行動 (單一 ToolCall) | 每個工具調用一次 |

> [來源: R3 Debate 4, R4.1-R4.2]

---

## 二、Position B：正規位置 (Canonical Location)

IVolition.deliberate() 在 ExecutionLoop 的 **Position B** 執行：
行動提案 (IGearArbiter plugin 或 IProvider plugin) **之後**、工具執行 **之前**。

TURING 的 v0.24.0 分析中，Position B 對應 **Step [5e] 與 Step [5g] 之間**：

```
[5e] PROCESSING_RESPONSE (LLM 串流回應, L303-412)
[5f] Build assistant message (L414-436)
>>> Position B: IVolition.deliberate() 注入點
[5g] EXECUTING_TOOLS (工具調度+執行, L439-503)
```

完整呼叫鏈中的定位：

```
Klesha.perceive() ................ 上游 (vijnana-clock)
IGearArbiter / IProvider ......... 行動提案 (plugin)
IVolition.deliberatePlan() ....... 計畫審議 (vijnana-clock)    ← Position B
IVolition.deliberateAction() ..... 行動審議 (vijnana-clock)    ← Position B
SafetyMonitor.postRouteCheck() ........ 下游 (Layer 0 硬閘)
```

Position B 在 Klesha（軟影響）之後、SafetyMonitor（硬約束）之前，
確保意志決策**受煩惱狀態調制**但**不能繞過安全閘門**。

> [來源: GUARDIAN (#11) + TURING (#17), R3 D4]

---

## 三、認知序列與 ExecutionLoop 對應 (Cognitive Sequence Mapping) — ASANGA

> 「觸為緣而受；所受即所知覺；所知覺即所尋思。」— MN 18

ASANGA 映射：

```
Sparsha (觸) → Vedana (受) → Samjna (想) → Vitakka (尋) → Cetana (思) → Samskara (行)
                                             LLM [5d-5e]   IVolition     ITool [5g]
```

| 佛學階段 | ExecutionLoop 對應 | Step |
|---------|-------------------|------|
| Sparsha (觸) | InputEvent + 觸處理 | [3]→[5a] |
| Vedana (受) | CoarisingBundle.vedana | vedana-clock |
| Samjna (想) | provider.chat() | [5d]-[5e] |
| **Cetana (思)** | **IVolition.deliberate()** | **[5e]→[5g]** |
| Samskara (行) | executeTool() | [5g] |

**為何 Position A 不適合**: Cetana 作用於 samjna **之後**。在 LLM 前修改 prompt 是「意志先於知覺」，哲學上不融貫。Position A 屬於 IGuide (作意 manasikara)。

> [來源: ASANGA (#8), R3 D4]

> **注**: 此序列的原始佛學出處為 MN 18:「觸為緣而受；所受即所知覺；所知覺即所尋思。」本文件僅取其工程對應價值。

---

## 四、雙階段審議架構 (Two-Phase Deliberation)

**Phase 1: deliberatePlan()** — cetana as intention (整體意志方向)
- 輸入: ToolCall[] + KleshaSignalBundle + VedanaAssessment
- 輸出: 修改後計畫 (重排/過濾/取消) 或 null (接受原計畫)
- 時間預算: 1-3ms (vijnana-clock)

**Phase 2: deliberateAction()** — cetana as action (具體行動承諾)
- 輸入: 單一 ToolCall + KleshaSignalBundle + Phase 1 上下文
- 輸出: veto (否決) + alternative 或通過
- 時間預算: 0.5-1ms per action (vijnana-clock)

```
典型場景: LLM 提出 3 個工具調用
  Phase 1: deliberatePlan()        ~2ms   (1 次)
  Phase 2: deliberateAction() × 3  ~2ms   (每次 ~0.7ms)
  總計審議時間                      ~4ms   (< vijnana-clock 5ms 上限)
```

---

## 五、IVolition 完整介面 (Complete Interface)

```typescript
/** IVolition -- 識蘊·意志動機（思 Cetana）
 *  Position B: 行動提案後，承諾/否決行動。
 *  @skandha vijnana | @clock vijnana-clock (1-5ms)
 *  R3 D4 決議: 雙階段審議 -- plan-level + action-level */
export interface IVolition extends IVijnana {
  deliberatePlan(input: PlanDeliberationInput): Promise<PlanDeliberationResult>;
  deliberateAction(input: ActionDeliberationInput): Promise<ActionDeliberationResult>;
}

interface PlanDeliberationInput {
  readonly proposedActions: readonly ToolCall[];   // 來自 IGearArbiter 或 IProvider plugin
  readonly kleshaSignals: KleshaSignalBundle;      // 來自 Klesha.perceive()
  readonly vedanaAssessment: VedanaAssessment;     // 來自 vedana-clock
  readonly context: SessionContext;
}

interface PlanDeliberationResult {
  readonly modifiedPlan: readonly ToolCall[] | null;  // null = accept as-is
  readonly reasoning: string;                          // 審計軌跡
}

interface ActionDeliberationInput {
  readonly proposedAction: ToolCall;
  readonly kleshaSignals: KleshaSignalBundle;
  readonly vedanaAssessment: VedanaAssessment;
  readonly planContext: PlanDeliberationResult;     // Phase 1 上下文
}

interface ActionDeliberationResult {
  readonly veto: boolean;
  readonly alternative: ToolCall | null;            // 否決時的替代建議
  readonly reasoning: string;
}
```

> [來源: BABBAGE (#9), R3 D4 + 02_type_system_changes.md Sec 1.5]

---

## 六、完整執行流程圖 (Complete Execution Flow)

```
SafetyMonitor.isSafe() ........................... Layer 0 硬閘
  │
Sparsha 形成 (觸) ................................ rupa-clock
  │
CoarisingBundle (vedana→samjna→cetana→manasikara) . vedana-clock (<1ms)
  │
ManoAggregator (歸依意) ........................... mano-clock
  │
Klesha.perceive() → KleshaSignalBundle ............ vijnana-clock
  │
IGearArbiter.evaluate()? ........................... vijnana-clock
  ├── 匹配 (Gear 1) → GearAction
  └── 無匹配 (Gear 2) → IProvider.chat() (LLM) ... samjna-clock
  │
┌─ Phase 1: deliberatePlan() ─────────────────┐ .. vijnana-clock
│  整體計畫審議 (1-3ms), 可重排/過濾/取消       │
└──────────────────────────────────────────────┘
  │
  For each action:
  ├─ Phase 2: deliberateAction() ──────────────┐ . vijnana-clock
  │  個別行動審議 (0.5-1ms), 可否決/修改         │
  └────────────────────────────────────────────┘
  │
  SafetyMonitor.postRouteCheck() ..................... Layer 0 硬閘
  │
  Tool execution (ITool.execute()) .............. samskara-clock
  │
  SafetyMonitor.afterToolExecution() ............ Layer 0 審計
  │
VedanaAssessment (回饋) .......................... vedana-clock
  │
KleshaBayesianUpdate (慢速路徑) .................. samjna-clock
  │
新環境 → 新觸 → 「beginning the cycle anew」 .... rupa-clock (Tenet #6)
```

> [來源: KERNEL (#10), R3 D4 正規呼叫順序圖]

---

## 七、TypeScript 實作虛擬碼 (Implementation Pseudocode)

```typescript
// Enhanced Step [5g] -- 注入點: loop.ts L436→L439

// Phase 1: 整體計畫審議
const planResult = await volition?.deliberatePlan({
  proposedActions: pendingToolCalls,
  kleshaSignals: currentKleshaBundle,
  vedanaAssessment: currentVedana,
  context: currentSessionContext,
});
if (planResult?.modifiedPlan) {
  pendingToolCalls = planResult.modifiedPlan;
}

// Phase 2: 逐一行動審議
for (const tc of pendingToolCalls) {
  const actionResult = await volition?.deliberateAction({
    proposedAction: tc,
    kleshaSignals: currentKleshaBundle,
    vedanaAssessment: currentVedana,
    planContext: planResult!,
  });

  if (actionResult?.veto) {
    auditTrail.logVeto(tc, actionResult.reasoning);
    feedbackQueue.push({             // 回饋給 LLM 下一輪
      type: 'volition_veto', action: tc.name,
      reason: actionResult.reasoning,
      alternative: actionResult.alternative,
    });
    continue;
  }

  const safetyResult = safetyMonitor.postRouteCheck(tc);
  if (!safetyResult.allowed) { continue; }

  const result = await executeTool(tc);
  safetyMonitor.afterToolExecution(result);
}
```

> [來源: TURING (#17) + ASANGA (#8), R3 D4]

---

## 八、IGuide/IVolition Bookend 模式

```
Position A (LLM 前)                 Position B (LLM 後)
IGuide (作意 Manasikara)            IVolition (思 Cetana)
  塑造注意力/修改 prompt              承諾/否決 tool calls
  想蘊的輸入端 [5b]                  行蘊的輸入端 [5e→5g]
```

| 屬性 | IGuide | IVolition |
|------|--------|-----------|
| 位置 | Position A | Position B |
| 蘊角色 | 作意 (manasikara) | 思 (cetana) |
| 功能 | 注意力塑造 | 行動承諾/否決 |
| 操作對象 | system prompt | tool calls |
| 現有實作 | guide.getSystemPrompt() | 新增 (Plan26) |

> ASANGA: 「Two different functions, two different positions, two different skandha roles.」

---

## 九、雙路徑強制性 (Mandatory for Both Paths)

**IGearArbiter 路徑 (Gear 1)**: 行動由 plugin 規則匹配產生，無反思性評估。
IVolition 是快速路徑中**唯一的審議機制**。若跳過，Agent 基於純粹習慣行動。

**IProvider 路徑 (Gear 2)**: 即使 LLM 已深層推理，IVolition 仍必須運行。
LLM (samjna) 產生認知結果，但 cetana (思) 是獨立心所 — 「知道該做什麼」不等同於「決定去做」。

> KERNEL: 「IVolition.deliberate() must be a mandatory checkpoint regardless
> of whether the action came from IGearArbiter (fast) or IProvider (slow).」
> [來源: R3 D4, R4.3]

---

## 九之一、識蘊雙判斷點 (Vijnana Dual Judgment Points) — DC-11 Master 修正

DC-11 確認：識蘊在執行流程中有**兩個判斷點**，不僅是 Position B 的審議。

### 判斷點 1：齒輪選擇 (Gear Selection)

識蘊決定當前情境該走哪條路：Gear 1（熟悉情境，想蘊快速辨認）或 Gear 2（陌生/複雜情境，想蘊深度辨認 via LLM）。

> Master: 「要不要經過 LLM，也可以讓識蘊來做。」

這意味著 IGearArbiter plugin 的信心度匹配不是齒輪選擇的唯一機制。識蘊可以基於更高層判斷直接覆寫齒輪選擇——例如，即使 IGearArbiter 匹配成功（Gear 1），識蘊仍可判斷「這個情境需要深思」而升級到 Gear 2。

### 判斷點 2：deliberatePlan 審議

在 Gear 2 路徑上，想蘊（LLM）產出行動提案後，識蘊在 Position B 執行 `deliberatePlan()` + `deliberateAction()` 雙階段審議（本文件主題）。

### 架構圖：識蘊雙判斷點

```
Layer 2: 歸依意路由 (ManoAggregator — 純路由)
     ━━ 識蘊·判斷點 1：走哪條路？━━
      Gear 1 (熟悉)              Gear 2 (陌生/複雜)
      想蘊快速辨認                想蘊深度辨認 (LLM)
         │                          │
         │               ━━ 識蘊·判斷點 2：deliberatePlan ━━
         │                          │
         ▼                          ▼
Layer 3: 行蘊執行 (SafetyMonitor → ITool.execute())
```

### 與現有架構的整合

| 判斷點 | 位置 | 時鐘域 | 功能 | 現有對應 |
|--------|------|--------|------|---------|
| 判斷點 1 | IGearArbiter 評估前/後 | vijnana-clock | 齒輪選擇覆寫 | 新增 (DC-11) |
| 判斷點 2 | Position B | vijnana-clock | 行動審議 | IVolition.deliberate() (本文件) |

判斷點 1 可透過 IVijnana 子介面體系實現——例如新增 `IGearSelector` 或將齒輪選擇邏輯整合到 EgoFramework 中。**此細節列入未來 Cycle 設計。**

> [來源: DC-11 Master 確認會議]

---

## 十、串級控制結構 (Cascade Control) — WIENER

```
┌─ Phase 1: 外部控制器 (Supervisory) ─┐
│  deliberatePlan() — 設定整體軌跡      │
└─────────────────┬────────────────────┘
                  │ setpoint
┌─ Phase 2: 內部控制器 (Local PID) ────┐
│  deliberateAction() — 局部調整執行    │
└─────────────────┬────────────────────┘
                  │ control signal
             工具執行 (Plant)
```

| 控制理論概念 | IVolition 對應 |
|-------------|---------------|
| 外部控制器 | Phase 1: deliberatePlan() |
| 內部控制器 | Phase 2: deliberateAction() |
| 設定點 | modifiedPlan (Phase 1 輸出) |
| 控制信號 | veto/pass (Phase 2 輸出) |
| 回饋信號 | VedanaAssessment → KleshaBayesianUpdate |

> WIENER: 「The cascade structure provides both strategic coherence (Phase 1)
> and tactical flexibility (Phase 2).」

---

## 十一、IVijnana 子介面架構 (Sub-Interface Architecture)

來源: Cycle 02-2 A-2 修正。IIdentity 不再等於整個識蘊，改為 IVijnana 根介面下的四子介面。

```typescript
/** 識蘊根介面 */
export interface IVijnana { readonly skandha: 'vijnana'; }

/** 自我認同 — 被動數據實體 (末那識恆審) */
export interface IIdentity extends IVijnana {
  readonly agentName: string;
  readonly agentDescription: string;
}

/** 行為引導 — 作意 (Manasikara), Position A */
export interface IGuide extends IVijnana {
  readonly name: string;
  readonly content: string;
}

/** 意志動機 — 思 (Cetana), Position B */
export interface IVolition extends IVijnana {
  deliberatePlan(input: PlanDeliberationInput): Promise<PlanDeliberationResult>;
  deliberateAction(input: ActionDeliberationInput): Promise<ActionDeliberationResult>;
}

/** 自省 — 尋伺 (Vitakka/Vicara), Pattern C Observer 基礎 (預留) */
export interface IReflection extends IVijnana {
  reflect(context: ReflectionContext): ReflectionResult;
}
```

| 子介面 | 佛學心所 | 功能 | 實作者 |
|--------|---------|------|--------|
| IIdentity | 末那識恆審 | 自我認同數據 | AgentConfig |
| IGuide | 作意 | 注意力/上下文 | GuidePlugin |
| IVolition | 思 | 審議/承諾/否決 | EgoFramework |
| IReflection | 尋伺 | 後設審議 (未來) | Pattern C Observer |

> [來源: 01_philosophical_corrections.md A-2 + 02_type_system_changes.md Sec 1.5]

---

## 十二、多 Agent 協調 (Multi-Agent) — LEIBNIZ

跨 Agent 行動時，IVolition.deliberateAction() 應諮詢 Coordination Daemon：

```typescript
interface CoordinationAwareDeliberation {
  readonly crossAgentImpact: boolean;           // 影響其他 Agent?
  readonly coordinationApproved: boolean | null; // Daemon 是否批准?
  readonly harmonyScore: number;                // 0.0=破壞, 1.0=和諧
}
```

預立和諧模型：Agent 不應單方面採取破壞集體的行動。crossAgentImpact=true 時，先查詢 Coordination Daemon。

> [來源: LEIBNIZ (#14), R3 D4, R4.6]

---

## 十三、審計軌跡 (Audit Trail)

PlanDeliberationResult 和 ActionDeliberationResult 的 `reasoning: string` 欄位用於：

1. **透明性**: 開發者/管理員檢視否決或修改原因
2. **LLM 回饋**: 否決理由回傳 LLM，使其調整下一輪提案
3. **永久日誌**: 所有審議決策記錄

否決回饋迴路: IVolition 否決 → reasoning 傳入 LLM 下一輪上下文 → LLM 調整提案 → IVolition 再審議。

> BABBAGE: 「The reasoning field closes the feedback loop between samjna and cetana.」

---

## 十四、安全分層 (Security Layering) — GUARDIAN

```
Layer 2 (觀察):  Vedana → CoarisingBundle → 受蘊評估    「觀察但不干預」
Layer 1 (審議):  Klesha → IVolition → proposed action   「軟影響」
Layer 0 (硬閘):  SafetyMonitor.postRouteCheck() → 允許/封鎖  「不可繞過」
```

IVolition 受煩惱狀態調制（哲學上正確），但無論其決定為何，最終行動必須通過 SafetyMonitor 硬檢查。

> GUARDIAN: 「This preserves the three-layer security model while giving
> IVolition genuine decision-making power within the safety envelope.」

---

## 十五、程式碼注入點 (Code Injection Points) — TURING

v0.24.0 ExecutionLoop 結構與注入位置：

| 注入 | 位置 | 行號 | 說明 |
|------|------|------|------|
| IGuide (Position A) | [5b] ASSEMBLING_CONTEXT | L239-253 | 已存在: guide.getSystemPrompt() |
| IVolition Phase 1 | [5f]→[5g] | L436→L439 | 新增: deliberatePlan() |
| IVolition Phase 2 | [5g] 內 for-loop | L439-503 | 新增: 每個 tc 前 deliberateAction() |
| SafetyMonitor.postCheck | [5g] 內 | 現有 | 在 deliberateAction() 之後 |

```typescript
// loop.ts — ExecutionLoopDeps 新增依賴
export interface ExecutionLoopDeps {
  // ... 現有依賴 ...
  volitionResolver?: () => IVolition | undefined;  // 識蘊·意志
}
```

> [程式碼: TURING Sec 4.2-4.5, loop.ts L45-66]

---

## 十六、階層式規劃平行 (Hierarchical Planning) — DARWIN

| 生物層級 | 審議能力 | OpenStarry 對應 |
|---------|---------|----------------|
| 昆蟲 | 行動層級反射 (stimulus→response) | Phase 2 only |
| 哺乳類 | 計畫+行動審議 | Phase 1 + Phase 2 (當前) |
| 靈長類 | 後設審議 (反思計畫品質) | IReflection (未來) |

目前架構實現**哺乳類層級**。未來 IReflection 將實現**靈長類層級**的後設審議。

> DARWIN: 「The two-phase deliberation mirrors hierarchical planning in advanced nervous systems.」

---

## 十七、Cycle 02-4 IVolition 風險分級與 Volition v1 規則架構 `[Cycle 02-4 新增]`

### 17.1 IVolition 按風險等級分級 (D5 決議)

Cycle 02-4 D5 辯論決議：IVolition 審議的必要性應按 action 風險等級區分，而非一律強制。

| 風險等級 | IVolition 審議 | 說明 |
|---------|---------------|------|
| **destructive** | **必經** (MUST) | 刪除檔案、格式化磁碟等不可逆操作，必須經過 deliberateAction() |
| **state_modifying** | **必經** (MUST) | 修改配置、寫入資料庫等有副作用操作 |
| **read_only** | **可選** (MAY) | 讀取操作，無副作用；可由部署策略決定是否跳過審議 |
| **informational** | **可選** (MAY) | 純資訊回饋（如回覆使用者問題），風險最低 |

**設計理由**: 對每個 read_only/informational action 都執行 deliberateAction() 會產生不必要的延遲（每次 0.5-1ms），且這些操作本身不改變系統狀態。但 destructive 和 state_modifying 操作**絕對不能跳過審議**，即使來自已驗證的 IGearArbiter plugin。

```typescript
// 風險分級判斷 (D5 決議)
function shouldDeliberate(action: ToolCall, riskLevel: RiskLevel): boolean {
  if (riskLevel === 'destructive' || riskLevel === 'state_modifying') {
    return true;  // MUST — 不可跳過
  }
  return config.deliberateReadOnly;  // MAY — 部署可配置
}
```

### 17.2 Volition v1: hardRules + softRules (D6 OQ-1)

D6 辯論的 OQ-1 確立了 Volition v1 的雙規則架構：

| 規則類型 | 等級 | 來源 | 可覆寫？ | 說明 |
|---------|------|------|---------|------|
| **hardRules** | P0 | SafetyMonitor action-level equivalent | **否** | 絕對安全規則，等同 SafetyMonitor 在 action 層級的投射 |
| **softRules** | P1 | 部署策略 (admin-configurable) | **是** (by admin) | 組織策略、合規要求等，管理員可調整 |
| ~~heuristicRules~~ | ~~P2~~ | ~~學習規則~~ | ~~延後~~ | ~~延後至 Plan29+ 設計，當前不實作~~ |

**hardRules** = SafetyMonitor 的 action-level 等價物：
- SafetyMonitor.postRouteCheck() 是在 IVolition 之後的**獨立硬閘**
- hardRules 是 IVolition 內部的**安全投射**，提供第一道防線
- 兩者形成 defense-in-depth：hardRules 先攔截，SafetyMonitor 再確認

**softRules** = 部署策略 (deployment policy)：
- 管理員可配置的組織級規則（如「不得存取外部 API」、「不得修改生產環境」）
- 與安全無關但與合規有關的約束
- 可透過配置檔調整，無需修改程式碼

```typescript
// Volition v1 規則架構 (D6 OQ-1)
interface VolitionV1 extends IVolition {
  readonly hardRules: readonly VolitionRule[];   // P0, non-overridable
  readonly softRules: readonly VolitionRule[];   // P1, admin-configurable
  // heuristicRules: deferred to Plan29+
}

interface VolitionRule {
  readonly name: string;
  readonly priority: 'P0' | 'P1';
  evaluate(action: ToolCall, context: SessionContext): VolitionRuleResult;
}
```

> [來源: Cycle 02-4 D5 (16/20) + D6 OQ-1]

---

## 十八、Plan28 實作：volition-rule-engine 三層規則引擎 `[Plan28 新增]`

### 18.1 概述

Plan28 完成了 IVolition v1 的第一個 plugin 實作：`volition-rule-engine`，採用三層規則引擎架構。

### 18.2 三層規則引擎

| 層級 | 優先序 | 可覆寫？ | 說明 |
|------|--------|---------|------|
| Hard | P0 (最高) | 否 | 絕對安全規則，等同 SafetyMonitor action-level 投射 |
| Soft | P1 | 是 (admin) | 部署策略、組織合規，管理員可配置 |
| Heuristic | P2 (最低) | 是 | 重複偵測、模式分析（Plan28 實作基礎版） |

**三層優先序**: Hard > Soft > Heuristic。高層規則的結果不可被低層覆寫。

### 18.3 defaultRiskCategory 回退

當 arbiter 未提供 `riskCategory` 時，volition-rule-engine 使用可配置的 `defaultRiskCategory`（SDK DEFAULT: `'read_only'`）。這確保所有動作都有風險分類，使三層規則的風險判斷始終有效。

### 18.4 DeliberationContext 型別

Plan28 新增 `DeliberationContext`，作為 IVolition 審議的 optional 上下文：

```typescript
/**
 * DeliberationContext — 路由結果 + 行動歷史。
 *
 * 提供給 IVolition plugin，使其能做風險感知的審議決策。
 * Optional field — 向後相容（無 Plan28 佈線時為 undefined）。
 *
 * @since Plan28 (v0.28.0-alpha)
 */
export interface DeliberationContext {
  readonly routeResult: RouteResult;
  readonly actionHistory: readonly ActionRecord[];
}
```

佈線路徑：loop.ts 構建 `DeliberationContext`（含 `routeResult` 和 `actionHistory`），傳入 `PlanDeliberationInput` / `ActionDeliberationInput`。

### 18.5 與既有架構的關係

```
[Plan26] IVolition 介面定義 + Position B 注入
  ↓
[Plan28] volition-rule-engine (三層規則引擎)
  │
  ├── Hard rules (P0) = SafetyMonitor action-level 投射
  │     → defense-in-depth: hardRules 先攔截，SafetyMonitor 再確認
  │
  ├── Soft rules (P1) = 部署策略
  │     → 管理員配置，無需修改程式碼
  │
  └── Heuristic rules (P2) = 重複偵測
        → 基礎版（Plan28），完整版需三閘門安全（Plan29+）
```

> [來源: Plan28 W5 volition-rule-engine; D6 OQ-1; Cycle 02-4 D5-R5]

---

## 附錄 A: Debate 4 決議摘要

| 編號 | 決議 | 投票 |
|------|------|------|
| R4.1 | Position B (行動提案後，工具執行前) | 20/20 |
| R4.2 | 雙階段: deliberatePlan() + deliberateAction() | 20/20 |
| R4.3 | IGearArbiter/IProvider 雙路徑均強制執行 | 20/20 |
| R4.4 | 正規呼叫順序圖 (KERNEL 版本) | 20/20 |
| R4.5 | IGuide(A) / IVolition(B) Bookend 模式 | 20/20 |
| R4.6 | 跨 Agent 行動需諮詢 Coordination Daemon | 20/20 |

**異議**: 無。

---

*IVolition 雙階段審議模式規格完成。涵蓋 Position B 定位、認知序列對應、
雙階段架構、IVijnana 子介面體系、串級控制、安全分層、程式碼注入點、
以及 Cycle 02-4 風險分級審議與 Volition v1 雙規則架構。*
