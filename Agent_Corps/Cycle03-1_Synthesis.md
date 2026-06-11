# Cycle 03-1 研究統合報告 — Phase 6 啟動

**日期**: 2026-03-24
**統合者**: SYNTHESIST (#1)
**審批**: SUNYATA (#0)
**基準版本**: v0.36.1-alpha (8/1/1)
**Plan 起始**: Plan37 (v0.37.0-alpha)

---

## 執行摘要

Cycle 03-1 是 Phase 6（Multi-Agent）的啟動輪。本輪完成 22 項決議（D1:5, D2:10, D3:3, D4:4），零宣言違反。研究涵蓋四大軌道：

1. **BUG-3 修復**（D1）：確認雙重根因（RC-1: 批次級 route() + RC-2: first-match 遮蔽），採用混合修復策略（worst-risk route + per-tool audit events），擴展 Rule #29 增加 must-invoke 第三安全保證類別，Tenet #8 維持 COMPLIANT with remediation。
2. **多代理通訊架構**（D2）：確立 Pipeline-first 實作順序，ICommChannel 統一介面從 Plan37 即定義於 SDK，Core 零修改（Tenet #7 保證），seL4 啟發式 capability model，五層故障隔離 + 優雅關機協定。
3. **哲學-佛學多代理延伸**（D3）：八識分配採混合模型（前七識 per-agent，第八識共享為世俗諦慣例指定），AC-6 先於 AC-7，ICompositeAgent 遞迴介面 + 權限格 + bisimulation。
4. **工程規劃**（D4）：Plan37 四波交付（~600-800 LOC），架構文件更新優先序，研究團隊四組轉型提案（延後啟動），W2-R1 協定定案（SC-1~SC-6）。

合規狀態維持 8/1/1，Tenet #6 和 #10 朝升級方向推進。新增/修改基線規則 10 條（#29 修改 + #31~#39 新增），總計 39 條 Current Baseline。

---

## 一、BUG-3 修復（D1, 5 項決議）

**軌道**: FAST TRACK (P0)
**參與者**: TURING (#17), KNUTH (#21), WIENER (#12), GUARDIAN (#11), BABBAGE (#9), ARCHIMEDES (#16), KERNEL (#10)

### BUG-3 根因確認

**狀態**: CONFIRMED。雙重根因：
- **RC-1**: `loop.ts` 每輪僅呼叫一次 `route()`，而非每個 tool 一次 [程式碼: packages/core/src/execution/loop.ts#processEvent, L475-486]
- **RC-2**: `gear-arbiter-static` 使用 `Array.some()` first-match，遮蔽較高風險 tool [程式碼: openstarry_plugin/gear-arbiter-static/src/index.ts#createStaticRuleEvaluator, L50-67]

### D1-R1: 混合修復策略

採用 Position B（Hybrid）：
1. **RC-2 修復（關鍵路徑）**：gear-arbiter-static evaluate() 改為 worst-risk 語義 — 掃描所有 proposedToolCalls 對所有 rules，回傳最高風險 riskCategory。風險排序：destructive(3) > state_modifying(2) > read_only(1) > informational(0)。~15 LOC。
2. **RC-1 修復（完整性路徑）**：loop.ts 工具執行迴圈中，每次工具執行後發出 `audit:tool_audited` 事件（toolName, inferredRiskCategory, executionResult, batchIndex, batchSize）。Non-blocking，不影響 delta 計算。~25 LOC。

**分類**: worst-risk = Mechanism；per-tool events = Policy。

### D1-R2: SC-3b CONDITIONAL PASS

SC-3b（4/4 category coverage）= CONDITIONAL PASS with cross-reference verification：
- **PASS**: 四類別皆出現於 audit records，OR 缺失類別確認未出現於工具執行日誌
- **FAIL**: 工具執行日誌顯示某類別工具已執行但無對應 audit entry（BUG-3 症狀）
- 自動化交叉比對腳本納入 W2-R1 測試指令

### D1-R3: 對稱累積夾鉗

維持對稱 ceiling/floor：正向 +0.05（Rule #26，Master 指令）、負向 -0.05（對稱，暫定）。GUARDIAN 的非對稱提案（+0.05/-0.10）正式登記至 Cycle 03-2 評估，需三項證據：WIENER 穩定性分析、KNUTH W2-R1 實證分佈、BABBAGE mechanism/policy 分類。

### D1-R4: Must-Invoke 第三安全保證類別

Rule #29 擴展為三類別：
1. **fail-open**: 觀察型元件（VedanaSensor, AuditTrailWriter）— 錯誤不阻塞
2. **fail-closed**: 強制型元件（SecurityLayer, timeoutAction='deny'）— 錯誤阻塞/拒絕
3. **must-invoke**: 安全監控元件 — 所有相關事件在所有執行路徑上必須觸發。CTL 形式化：`AF invoke(c)`

**實作機制**: 由呼叫端（loop.ts per-tool events）保證，非自我報告。外部驗證透過 `audit:tool_audited` 事件計數匹配工具執行計數。

### D1-R5: Tenet #8 維持 COMPLIANT

BUG-3 歸類為實作缺陷（plugin logic + architecture wiring），非設計層級控制缺口。條件：
1. 修復必須在 Plan37 交付
2. W2-R1 必須確認 4/4 category coverage（SC-3b）
3. 修復後 sigma > 0

**證據**: ThresholdAuditor 正確定義四類 delta 規則；sigma=0.000245 證明迴路在功能類別上存活；WIENER 預測修復後 sigma ~0.012。

---

## 二、多代理通訊架構（D2, 10 項決議）

**軌道**: CORE DEBATE（Phase 6 基礎決議）
**參與者**: MESH (#4), ATHENA (#5), KERNEL (#10), RUSSELL (#23), TANENBAUM (#20), SUSSMAN (#22), VITRUVIUS (#3), DARWIN (#6), HERACLITUS (#15), GUARDIAN (#11), BABBAGE (#9), LEIBNIZ (#14), ARCHIMEDES (#16)

### D2-R1: 通訊模式實作順序

**Pipeline (Plan37) -> Hub-and-Spoke (Plan38) -> API Runtime (Plan39+) -> Mesh (Phase 6 late)**

- Pipeline 零 Core 修改，利用現有 Daemon spawnAgent() + 循序編排，~150-200 LOC
- Hub-and-Spoke：Daemon 作為自然 Hub（QNX 模型），~400-500 LOC
- API Runtime：完整生命週期控制，IAgentDriver 介面早期設計但延後實作，~800+ LOC
- Mesh：最高複雜度（O(n^2) 連線），需 API Runtime 作為先決條件

全場一致通過。

### D2-R2: ICommChannel 從 Plan37 即定義

SDK 層定義 `ICommChannel` 介面（comm-channel.ts），僅實作 `PipelineChannel`：
- `CommCapability = 'messaging' | 'streaming' | 'rpc' | 'composable'`
- `CommMessage` 含 `traceId` + `traceDepth`（遞迴防護）
- `PluginHooks.commChannels?: ICommChannel[]`（array slot）
- `CommChannelRegistry` 於 PluginLoader（平行 VedanaRegistry 模式）

TANENBAUM/SUSSMAN「一次設計正確」論證勝出 ATHENA 的快速 MVP 方案（差異僅 ~80 LOC）。

### D2-R3: Claude Code Channels 定位

Channels = optional transport adapter plugin。Plan37 無依賴。所有訊息即使使用 Channels 傳輸仍必須經 Hub 進行 capability verification。

### D2-R4: Core 零修改

**Core 新增零 IPC 原語。EventBus 不變。所有多代理通訊透過 Daemon + Plugin。**

| 層 | 多代理職責 | 修改 |
|---|-----------|-----|
| Core | 無。Per-Agent EventBus/StateManager/ExecutionLoop | **零修改** |
| SDK Types | ICommChannel, CommMessage, CommCapability | 新型別定義 |
| Plugin | PipelineChannel, comm-proxy sidecar | 新插件 |
| Daemon | MessageRouter, EventBridge, Global ServiceRegistry, ProcessTree | Daemon 擴充 |

### D2-R5: seL4 啟發式 Capability Model

agent.json 宣告通訊 capabilities（canSendTo, canReceiveFrom, exposedTools）：
- **預設零 capability** — 無 `communication` 區段的 Agent 無法收發多代理訊息
- **Capability checking = Mechanism** — Daemon MessageRouter 必須驗證，不可繞過
- **Permission lists = Policy** — agent.json 可設定
- **Child subset of parent**: `C_child.canSendTo ⊆ C_parent.canSendTo`，spawn 時強制

### D2-R6: 三原語 + publish 為 streaming capability

最小 messaging：`send(target, message)` + `onMessage(handler)` + `reply(msgId, response)`。`publish/subscribe` 為 'streaming' capability，由宣告該 capability 的 channel 實作。PipelineChannel（Plan37）僅 messaging；HubSpokeChannel（Plan38）加 streaming。

### D2-R7: EventBridge + L2 Global ServiceRegistry

- EventBridge 為 Daemon 層服務，非 Core。Agent 透過 comm plugin 選擇性轉發事件。Per-agent event type whitelist。
- L2 Global ServiceRegistry 於 Daemon CoordinationLayer。L1 local 優先（fast path），L2 global via IPC（DNS 類比）。
- 與未來 Blackboard/Alaya 分離 — 統一評估延後至 Plan39+。

### D2-R8: 五層故障隔離 + 優雅關機

**五層**：L1 Process Isolation（既有）-> L2 Circuit Breaker -> L3 Bulkhead -> L4 Supervisor Strategy -> L5 Timeout Hierarchy。

**優雅關機**: RUNNING -> DRAINING -> TERMINATED。Grace period SDK default 30s，hard max 300s（mechanism ceiling）。Force-kill after timeout = mechanism。DRAINING Agent 禁止 spawn 新子代（防止 drain evasion）。

### D2-R9: 文件更新優先序

P0（本輪）：Doc 09（Communication Protocol Strategy）+ Doc 19（Agent Coordination Layer）+ Doc 57（NEW: Multi-Agent Communication Interface Spec）。P1：Doc 13 + NEW Multi-Agent Security Model。P2+：Doc 06/22 + Unified Comm Interface Spec + State Consistency。

### D2-R10: 研究團隊四組轉型

四組結構（A: Architecture, B: Philosophy, C: Verification, D: Engineering）原則接受，實際切換延後至 Cycle 03-4+ pilot。組內 Pipeline，組間 Hub-and-Spoke。12/13 通過（MESH 保留組別分配意見）。

---

## 三、哲學-佛學多代理延伸（D3, 3 項決議）

**參與者**: ASANGA (#8), NAGARJUNA (#7), PENROSE (#18), LINNAEUS (#13), LEIBNIZ (#14), BABBAGE (#9), PASCAL (#19), RUSSELL (#23)

### D3-R1: 八識多代理分配（混合模型）

| 識 | 分配 | 理據 |
|---|------|-----|
| 前五識 (1-5) | **Per-agent** | 各 Agent 有獨立感覺通道（IListener），不同 Agent 面對不同 indriya/vishaya |
| 意識 (6) | **Per-agent** | 各 Agent 需獨立 ExecutionLoop + IProvider 認知循環 |
| 末那識 (7) | **Per-agent + 階層覆寫** | 各 Agent 有自己的 EgoFramework；父 Agent 末那可覆寫子 Agent 自我觀 |
| 阿賴耶識 (8) | **共享（世俗諦慣例指定）** | 單一種子庫綁定所有 Agent。Management Zone 作為協調基礎。**二諦宣言必須標註** |

**佛學共識**: 瑜伽行派（ASANGA）與中觀派（NAGARJUNA）於世俗諦層面達成共識。共享阿賴耶為假名施設（prajnapti），非存有論主張。

**形式模型**: BABBAGE pi-calculus 公式化：`System = (nu alaya)(Agent_1(alaya) | ... | Agent_N(alaya))`。向後相容 bisimulation：N=1 退化為單代理行為。

### D3-R2: Tenet #6 升級路徑

**AC-6 先於 AC-7**：
- **AC-6（前五識型別化）**：~150-200 LOC，介面層變更。定義 5 個 typed IListener 子介面（IVisualListener 等），senseType 辨別欄位。Union type 向後相容。Target: Plan37。
- **AC-7（阿賴耶分散式）**：~800+ LOC，多個新插件 + 分散式協定。Dependencies: Process Tree + ICommChannel。Target: Plan38+。需 GUARDIAN 安全審查。

### D3-R3: ICompositeAgent 遞迴介面

Tenet #10 fractal composition 正式模式：
- **ICompositeAgent** 遞迴 `children: ReadonlyArray<ICompositeAgent | ILeafAgent>`
- **權限格**: `child.allowedPaths ⊆ parent.allowedPaths`（Mechanism，spawn 時強制）
- **資源預算**: `reserve_ratio` = Policy（SDK default 0.3），分配策略 = Policy
- **Fractal depth limit**: 3（Policy，可設定）
- **Backward-compatible bisimulation**: 單子代 CompositeAgent 必須與獨立 Agent 弱 bisimilar
- **哲學映射**: 佛教僧伽模型（個人 -> 社群 -> 社群之社群）

---

## 四、工程規劃（D4, 4 項決議）

**參與者**: ARCHIMEDES (#16), TURING (#17), VITRUVIUS (#3), WIENER (#12), PASCAL (#19), KNUTH (#21), SUNYATA (#0), LEIBNIZ (#14)

### D4-R1: Plan37 範圍

四波交付，~600-800 LOC，v0.37.0-alpha：

| Wave | Priority | 範圍 | LOC |
|------|----------|------|-----|
| W1 | P0 | BUG-3 修復（worst-risk + per-tool audit）+ 累積夾鉗 +0.05 驗證 | ~50 |
| W2 | P1 | ICommChannel SDK type + PipelineChannel plugin + Process Tree（parentAgentId, spawnChildAgent, permission lattice） | ~300 |
| W3 | P1 | AC-6 前五識 typed IListener 子介面 + agent.json 通訊 capability | ~200 |
| W4 | P2, conditional | EventBridge Daemon service + L2 Global ServiceRegistry | ~150 |

**約束**: W1 獨立驗證後才進 W2；W2 Process Tree 須通過 BABBAGE continuity test（零子代 bisimulation）；W3 AC-6 須 union-compatible with existing IListener；W4 conditional on timeline。

### D4-R2: 架構文件更新

本輪（P0）：Doc 19（Agent Coordination Layer）+ Doc 09（Communication Protocol Strategy）+ Doc 57（NEW: Multi-Agent Communication Interface Spec）。下輪（P1）：Doc 13 + NEW Multi-Agent Security Model。延後：Doc 06/22 + Unified Comm Interface Spec + State Consistency。

### D4-R3: 研究團隊轉型

四組結構原則接受（A: Architecture, B: Philosophy, C: Verification, D: Engineering）。實際切換延後至 Cycle 03-4+，需滿足三條件：Plan37 W2 實作且驗證、至少完成一輪有多代理 codebase 的研究循環、組間通訊協定已文件化。

### D4-R4: W2-R1 協定定案

**前置條件**: BUG-3 修復獨立驗證通過 + 累積夾鉗 +0.05 代碼確認 + LLM >= 70B。

**協定**: 50 cycles, 5 blocks（category-targeted），參數從新鮮資料重新校準（不沿用 W2 v2）。

**成功準則**:

| SC | 準則 | 閾值 |
|----|------|-----|
| SC-1 | Audit rate | >= 50% rounds 產出 audit entries |
| SC-2 | Non-zero delta | >= 20% entries delta != 0 |
| SC-3 | Risk category count | >= 3 distinct categories |
| SC-4 | Sigma | > 0 |
| SC-5 | Category representation | 每個觀察到的類別至少出現 1 次（NEW） |
| SC-6 | Bidirectional delta | 至少 1 正向 AND 1 非正向 delta（NEW） |

**三層判定**: GO（全 SC pass）/ INFORMATIVE FAILURE（>= 4/6 pass + operational failures）/ REDESIGN（< 4/6 pass OR structural failure）。

---

## 五、決議總覽表

| ID | 主題 | 決議摘要 | 分類 | 新規則? |
|----|------|---------|------|---------|
| D1-R1 | BUG-3 修復策略 | Hybrid: worst-risk route() + per-tool audit events | Mech + Policy | No |
| D1-R2 | SC-3b 定位 | CONDITIONAL PASS with cross-reference | Policy | No |
| D1-R3 | 累積夾鉗對稱性 | 對稱 +0.05/-0.05；非對稱延後 03-2 | Policy | Provisional |
| D1-R4 | Must-invoke | 第三安全保證類別（CTL AF phi） | Mechanism | **#29 修改** |
| D1-R5 | Tenet #8 評估 | COMPLIANT with remediation | Policy | No |
| D2-R1 | 通訊模式順序 | Pipeline -> Hub-Spoke -> API Runtime -> Mesh | Policy | No |
| D2-R2 | ICommChannel | Plan37 SDK 定義，僅 PipelineChannel | Mechanism | **#31** |
| D2-R3 | Channels 定位 | Optional transport adapter，無依賴 | Policy | No |
| D2-R4 | Core 純淨 | 零 Core 修改 | Mechanism | **#32** |
| D2-R5 | Capability model | seL4 啟發式，Daemon 強制，default zero | Mechanism | **#33** |
| D2-R6 | IPC 原語 | Three primitives + publish as capability | Mechanism | No |
| D2-R7 | EventBridge | Daemon 層，與 Blackboard 分離 | Mechanism | No |
| D2-R8 | 故障隔離 + 關機 | 五層隔離 + 優雅關機 + force-kill | Mechanism | **#34, #35** |
| D2-R9 | 文件優先序 | Doc 09/19 P0 + Doc 57 new | Policy | No |
| D2-R10 | 團隊轉型 | 四組原則接受，延後 03-4+ | Policy | No |
| D3-R1 | 八識分配 | 混合模型（per-agent 1-7, shared 8） | Architecture | **#36** |
| D3-R2 | Tenet #6 路徑 | AC-6 先，AC-7 後 | Engineering | **#37** |
| D3-R3 | ICompositeAgent | 權限格 + bisimulation + depth=3 | Architecture | **#38** |
| D4-R1 | Plan37 範圍 | 四波 ~600-800 LOC | Engineering | No |
| D4-R2 | 文件更新 | Doc 09/19/57 本輪 | Policy | No |
| D4-R3 | 團隊轉型時程 | 延後至 03-4+ | Policy | No |
| D4-R4 | W2-R1 協定 | SC-1~SC-6，50 cycles | Policy | **#39** |

**Total: 22 decisions, zero tenet violations.**

---

## 六、新基線規則

### 修改規則

**Rule #29 (MODIFIED)**: 三類安全保證 — fail-open（觀察型）, fail-closed（強制型）, **must-invoke（安全監控型）**。Must-invoke 形式化為 CTL `AF invoke(c)`，由呼叫端保證（非自我報告），外部驗證透過事件計數匹配。
[來源: D1-R4]

### 新增規則

**Rule #31**: ICommChannel 統一通訊介面 — 所有多代理 channel 實作 ICommChannel with capability declaration (messaging/streaming/rpc/composable)。SDK 型別定義，PluginHooks.commChannels? array slot。
[來源: D2-R2]

**Rule #32**: 多代理零 Core 修改 — 所有通訊邏輯於 Daemon + Plugin 層，Core 不變。EventBus 維持 per-Agent 範疇，不加入 sendTo/onRemote。
[來源: D2-R4]

**Rule #33**: seL4 啟發式 capability model — capability checking = mechanism（Daemon 強制，不可繞過），permission rules = policy（agent.json 可設定）。預設零 capability。Child capability subset of parent（格約束，spawn 時強制）。
[來源: D2-R5]

**Rule #34**: 五層故障隔離 — L1 Process Isolation（既有）-> L2 Circuit Breaker -> L3 Bulkhead Isolation -> L4 Supervisor Strategy -> L5 Timeout Hierarchy。L1 = Required，L2-L5 = optional-degraded。
[來源: D2-R8]

**Rule #35**: 優雅關機協定 — RUNNING -> DRAINING -> TERMINATED。Force-kill after grace period（mechanism）。Grace period default 30s（policy），hard max 300s（mechanism ceiling）。DRAINING 禁止 spawn 新子代。
[來源: D2-R8]

**Rule #36**: 八識多代理分配混合模型 — 前五識 (1-5) 及意識 (6) per-agent；末那識 (7) per-agent with hierarchical override；阿賴耶識 (8) 共享為 samvriti-satya 慣例指定，二諦宣言 REQUIRED。
[來源: D3-R1, ASANGA-NAGARJUNA Buddhist consensus]

**Rule #37**: Tenet #6 升級順序 — AC-6（前五識型別化，~150-200 LOC）先於 AC-7（阿賴耶分散式，~800+ LOC）。AC-6 為介面層變更；AC-7 需分散式狀態基礎設施。
[來源: D3-R2]

**Rule #38**: ICompositeAgent 權限格 + bisimulation — child.allowedPaths subset_of parent.allowedPaths（mechanism，always enforced）。資源預算 reserve_ratio = policy（SDK default 0.3）。Fractal depth limit = 3（policy，可設定）。單子代 CompositeAgent 必須與獨立 Agent 弱 bisimilar（backward-compatible bisimulation）。
[來源: D3-R3, LEIBNIZ + BABBAGE + PASCAL consensus]

**Rule #39**: W2-R1 增強成功準則 — SC-1~SC-4 沿用 W2 v2 + SC-5（每個類別至少出現 1 次）+ SC-6（雙向 delta 確認）。參數從新鮮資料重新校準，不沿用 W2 v2。BUG-3 修復驗證為前置條件。
[來源: D4-R4, WIENER + PASCAL + KNUTH consensus]

### 暫定規則

- 累積負向 floor = -0.05/session（對稱，暫定）。Subject to Cycle 03-2 review with W2-R1 empirical data（D1-R3）。

### 規則總計

Current Baseline: #1-#30（既有）+ #29 修改 + #31-#39 新增 = **39 條 Current Baseline**
Provisional: 累積負向 floor -0.05 + PROC-SPEC-3（既有）= 2 條 PROVISIONAL

---

## 七、合規影響分析

### 當前狀態 (v0.36.1-alpha): 8 COMPLIANT / 1 CONDITIONAL / 1 NON-COMPLIANT

| Tenet | 狀態 | Cycle 03-1 影響 |
|-------|------|----------------|
| #1 Agent as OS Process | COMPLIANT | 無變化。每個 Agent 維持獨立進程。 |
| #2 Everything Plugin | COMPLIANT | 強化：ICommChannel 為 plugin array slot。 |
| #3 Five Aggregates | COMPLIANT | 無變化。Per-agent 五蘊保留。 |
| #4 Directory as Protocol | COMPLIANT | 無變化。 |
| #5 Directory as Permission | COMPLIANT | 強化：capability model 延伸至多代理。 |
| #6 Eight Consciousnesses | CONDITIONAL | 推進中：AC-6 in Plan37, AC-7 in Plan38+。 |
| #7 Microkernel Purity | COMPLIANT | 強化：D2-R4 零 Core 修改保證。 |
| #8 Control Theory | COMPLIANT | With remediation（BUG-3 fix in Plan37）。 |
| #9 Pluggable Memory | COMPLIANT | 無變化。 |
| #10 Fractal Social | NON-COMPLIANT | 推進中：ICompositeAgent + Process Tree in Plan37。 |

**Post-Plan37 預測**: 8/1/1（深化，無狀態變更）
**Post-Plan38 預測**: 9/1/0 或 9/0/1（潛在 #6 升級）
**Cycle 03 目標**: 10/0/0

詳見 `compliance_impact_analysis.md`。

---

## 八、Plan37 概要

**版本**: v0.37.0-alpha
**LOC**: ~600-800（W4 conditional）
**交付**: 四波

| Wave | Priority | 內容 | LOC | 關鍵約束 |
|------|----------|------|-----|---------|
| W1 | P0 | BUG-3 修復 + 累積夾鉗驗證 | ~50 | 須獨立驗證後才進 W2 |
| W2 | P1 | ICommChannel + PipelineChannel + Process Tree | ~300 | BABBAGE continuity test |
| W3 | P1 | AC-6 typed IListener + agent.json comm capability | ~200 | Union-compatible with existing IListener |
| W4 | P2 | EventBridge Daemon + L2 ServiceRegistry | ~150 | Conditional on timeline |

**文件交付**: Doc 09 更新 + Doc 19 更新 + Doc 57（NEW）
**測試交付**: BUG-3 fix verification + W2-R1 protocol + test instructions

詳見 `plan37_spec.md`（deliver/）。

---

## 九、未解決議題與延後項目

| 項目 | 狀態 | 目標 |
|------|------|------|
| 非對稱累積夾鉗 (+0.05/-0.10) | 登記待評估 | Cycle 03-2（需 WIENER/KNUTH/BABBAGE 三項證據） |
| Must-invoke 廣泛範疇（VedanaSensor, ConfirmationGate, 多代理延伸） | 延後 | 後續 cycles |
| AC-7 阿賴耶分散式 | 延後 | Plan38+（需 Process Tree + ICommChannel + GUARDIAN 安全審查） |
| L2 Registry 與 Blackboard 統一評估 | 延後 | Plan39+（需兩者皆存在後再評估） |
| 研究團隊四組轉型 | 延後 | Cycle 03-4+ pilot（需 Plan37 W2 實作 + 一輪多代理經驗 + 組間協定文件化） |
| Hub-and-Spoke 通訊 | 延後 | Plan38 |
| API Runtime | 延後 | Plan39+ |
| Mesh 通訊 | 延後 | Phase 6 late |
| Tool-level capability filtering | 延後 | Plan39+（GUARDIAN 建議 Agent-level 足夠 MVP） |
| Circuit Breaker + Bulkhead + Timeout Hierarchy | 延後 | Plan38（L4 Supervisor in Plan37） |
| GlobalManoAggregator | 延後 | 未來 plugin spec（PENROSE 撤回 Core-level 提案） |
| Doc 06/22/07 更新 | 延後 | Plan39-40+ |

---

## 十、下一步

1. **R4 交付物完成**:
   - `plan37_spec.md` — Plan37 完整工程規格（ARCHIMEDES 主筆）
   - `compliance_impact_analysis.md` — 合規影響分析
   - `prior_research/cycle03-1/` — decisions_summary + research_roadmap
   - `plan_roadmap.md` 更新

2. **Plan37 工程交付**:
   - W1: BUG-3 修復 + 獨立驗證
   - W2: ICommChannel + PipelineChannel + Process Tree
   - W3: AC-6 typed IListener + agent.json comm
   - W4: EventBridge + ServiceRegistry (conditional)

3. **W2-R1 Lyapunov 測試**:
   - BUG-3 修復驗證通過後執行
   - 50 cycles, 5 blocks, SC-1~SC-6
   - 參數重新校準

4. **Cycle 03-2 研究範疇**:
   - Plan37 驗證
   - 非對稱夾鉗評估（D1-R3 三項證據）
   - W2-R1 結果分析
   - Hub-and-Spoke 設計（Plan38 預備）

---

*SYNTHESIST (#1), Cycle 03-1 R4 Synthesis*
*Reviewed by: SUNYATA (#0)*
*2026-03-24*
