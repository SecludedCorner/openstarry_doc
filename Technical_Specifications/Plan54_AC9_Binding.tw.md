---
title: Plan54 — AC-9 子代理組合 Candidate A BINDING 工程規範
author: LEIBNIZ (#14) + RUSSELL (#23) + TANENBAUM (#20) + ARCHIMEDES (#16) + SUNYATA (#0) + SYNTHESIST (#1)
date: 2026-04-28
cycle: 03-16
status: 程式庫已移除於 v0.58.0-alpha（2026-06-11 修復稽核）— v0.52.0-alpha 交付且有測試，但從未接線（0 個 production import；MAX_SPAWN_DEPTH=4 與現役 daemon COMPOSITE_AGENT_MAX_DEPTH=3 衝突）。本設計保留作未來工作參考；現役組合實作 = daemon spawn 系統（Plan37/38）。
authority: research-team (R4 final draft); Master (ratification)
supersedes: null
language: tw
cross_refs:
  - research record/cycle03-16/openstarry_doc/Technical_Specifications/Plan54_AC9_Binding.md (EN sibling)
  - research record/cycle03-16/deliver/O1_AC9_Plan54_final.md
  - research record/cycle03-16/deliver/Master_Ratification/Batch_13_Request.md §3.1
  - research record/cycle03-14/openstarry_doc/Technical_Specifications/Plan52_pushInput_CandidateB.md (Plan52 同構基準)
  - research record/cycle03-15/openstarry_doc/Reference/11_Rule_78_TW_Translation.md
binding_until: Master Ratification Batch 13 close
prefix_discipline: verified | inferred | speculative
---

# Plan54 — AC-9 子代理組合 Candidate A BINDING 工程規範

## 1. 狀態

**CANDIDATE**（cycle 03-16 R4 close）。經 Master Ratification Batch 13 Item #1 後升為 **BINDING**。**前向有效**：自 cycle 03-16 R4 close 起；cycle 03-16 之前的 Core 表面**保留不修改**，依 MR-12 既有不破壞與 MR-6 Core 零原則。

**落地形式**：Phase 6 第二棒 — AC-9 子代理組合首份正式 spec；Plan52 同構（pushInput Candidate B v0.50.0-alpha 於 cycle 03-14 R4 close 出貨）。

**TW 配對 sibling**：本檔即為 EN sibling `Technical_Specifications/Plan54_AC9_Binding.md` 的 TW sibling，依 Rule #78 §78.5 forward-only + §78.8 reflexive + cycle 03-16 R3 D-09 UNANIMOUS 採取 same-PR-strict 同 PR 落地。

## 2. R3 出處

- **D-01（Round A 主案；AC-9 候選方案投票）** — Stage 1（推進 vs D 延後）= 19/4 否決 D；Stage 2 多數決 A=14、B=4、C=5；Stage 2 決選（A vs 非 A 合併）= **16/7 — A 以 ⅔ 超多數通過**。DSS-CY16-01 共 9 票少數意見依 MR-11 逐字保留（4 票 D 延後 + 5 票 C orchestrator + Stage 2 4 票 B hybrid 結轉）。
- **D-04（Round A；MAX_SPAWN_DEPTH 預設）** — `MAX_SPAWN_DEPTH = 4` 配合 operator 覆寫，**17/6**。DSS-CY16-03 6 票（KERNEL + TANENBAUM + GUARDIAN + 其他 3 位）偏好 3-cap 之少數意見保留。
- **D-11（Round B；編號）** — Plan54 整數編號 vs Plan52a 子編號，**19/4**。DSS-CY16-NUM 4 票偏好鏈式一致性少數意見保留。
- **D-13（Round C；LOC 上限）** — 800 prod + 500 test，**全體一致 23/0**。
- **D-18（Round D；哲學註釋）** — NAGARJUNA Madhyamaka + PENROSE 意識註釋採用為 non-binding doc-only，**18/5**。DSS-CY16-06 5 票哲學漂移少數意見保留。
- **MRB-§1-01 RESOLVED** — §一（AC-9）必須先於 §四（plugin-loader）的 sequencing pin 鎖定。
- **MRB-§1-02 RESOLVED** — Plan54 編號傳播由 SCRIBE + coordinator G5 4-mirror sync 執行。

## 3. 架構摘要（Candidate A — Full Plugin、Plan52 同構）

### 3.1 ε-Surface 重述（繼承 Plan52）

Plan54 實作回合的 Core 表面差異：

```
+ 0 個新欄位（Plan52 sourceContext?.parentAgentId 已存在）
+ 0 個 Zod passthrough 增添（Plan52 z.record(z.unknown()).optional() 已存在）
不新增（否則違反 MR-6 FAIL）：
  - ISpawner interface
  - SubAgent type
  - spawn(): SubAgentHandle method
  - {SPAWNED, ACTIVE, COMPLETED, ABORTED, ORPHANED} enum
  - MAX_SPAWN_DEPTH 常數於 Core（Candidate B 已 REJECTED）
  - lifecycle hook registry 於 Core
相對 Plan52 baseline 的 Core 總差異：0 欄位、0 const → ε-surface PASS（嚴格相等）。
```

### 3.2 Plugin 即組合 runtime

AC-9 子代理組合的權威 runtime 為 **`agent-composition` plugin**（暫名；最終命名於 cycle 03-17 R0 確定）。Plugin 位於 `apps/runner/src/agent-composition/`（peripheral；**非** `@openstarry/core` 之下）。Plugin：

1. 透過 `@openstarry/sdk` **匯出** SDK 層工廠：
   - `spawnChild(req: SpawnChildRequest): SpawnChildResponse`
   - `registerLifecycleHook(state: LifecycleState, handler: LifecycleHandler): void`
   - `enforceSpawnDepth(parentDepth: number, maxDepth: number): void`
2. 使用 **Plan52 HMAC-SHA256 簽章 token 紀律**（CV-02/04/05 重申；不引入新 crypto）**驗證**父代理身分。
3. 透過既有 pushInput pipe **建構**子 InputEvent 發送 — **無新增 pipe**。
4. 透過 Plan52 既有 Core 轉發 pipe **發送**。**Core 轉發 bytes；Core 不讀 `sourceContext`。**

### 3.3 Plan52 同構（逐字重用對照）

| Plan52 contract | AC-9 Plan54 重用方式 |
|-----------------|--------------------|
| `InputEvent.sourceContext?: Record<string, unknown>` | 逐字重用；子代理繼承 |
| `RecommendedSourceContextKeys`（SDK 匯出） | 前向擴展（MR-12）：`parentAgentId`、`spawnDepth`、`spawnId` |
| HMAC-SHA256 預設 tokenSig | 逐字重用（CV-02） |
| 演算法前綴強制（如 `hmac-sha256:`） | 逐字重用（CV-04） |
| Local-CLI MAY-omit tokenSig | 逐字重用（CV-05） |
| Nonce ≥ 16 bytes 熵 | 逐字重用（CV-03） |
| Replay cache TTL ≥ 金鑰輪替重疊 | 逐字重用（CV-06） |
| deepFreeze 遞迴不變性 | 逐字重用 |
| Tri-party MR-6 audit（TANENBAUM + KERNEL + GUARDIAN） | 擴展至 AC-9 |
| F-13 hook dispatch 可驗證性 | 擴展至 AC-9 spawn / lifecycle hooks |
| F-14 外部資源基準 | 擴展至 AC-9 LLM-token-per-child + spawn quota |
| F-15 v3 reflexive scope | 擴展至本 spec 文件 + plugin 程式碼 |

## 4. 組合 primitives（Plugin 層；非 Core）

### 4.1 Spawn primitive

`SpawnChildRequest` Zod schema 定義於 `@openstarry/sdk`：
- `parentAgentId: string` — 依 Plan52 §3.4 簽章 token 驗證
- `parentTokenSig: string` — 演算法前綴強制（CV-04）
- `childAgentSpec: { capability: string; config: Record<string, unknown> }`
- `spawnDepth: number` — 子 = 父 + 1
- `spawnId?: string` — UUID；若省略則由 AC-9 plugin 產生
- `nonce: string` — ≥ 16 bytes 熵（CV-03）

`SpawnChildResponse`：`{ success, childAgentId, childTokenSig, state: 'spawned', reason? }`。

### 4.2 Lifecycle primitive

狀態機（plugin 字串 literal；**非** Core enum）：
```
spawned → active → {completed, aborted, orphaned}
```

Lifecycle hooks：`onSpawned` / `onActive` / `onCompleted` / `onAborted` / `onOrphaned`（預設 30 秒 grace window）。透過 Plan51 hook-registry 派發（cycle 03-15 ratified 模組；首次出貨 cycle 03-16 W2-R16 verification 依 MRB-§1-01 sequencing pin）。F-13 hook dispatch 可驗證性適用。

### 4.3 Boundary primitive

1. **加密邊界** — 每次 spawn 重新簽 `tokenSig`（HMAC-SHA256、新 nonce、子代理特定 subject）；父 tokenSig **不**轉發。
2. **語義邊界** — 子代理在 `sourceContext` 繼承 `parentAgentId`，供下游政策 plugins 使用。
3. **血統完整性** — 遞迴走訪 `sourceContext.parentAgentId`（深度 ≤ MAX_SPAWN_DEPTH）；對 Core 不透明（MR-6 invariant）。
4. **能力封裝** — 由 plugin 政策決定；AC-9 Plan54 僅提供機制。

### 4.4 IPC contract

重用 Plan52 既有 pushInput pipe。**無新 pipe、無新 transport、無新事件類型。**

## 5. ε-Surface MR-6 Reflexive Audit（Tri-Party 簽核）

於 v0.52.0-alpha tag（cycle 03-17 實作關閉）：
- **TANENBAUM (#20)** — 驗證 Core file（`packages/core/src/**`）未引入 `agent`、`spawn`、`subagent`、`parent`、`child`、`lifecycle`、`orchestrator` 詞彙超出 Plan52 baseline。
- **KERNEL (#10)** — 驗證 Core 表面相對 `v0.50.0-alpha` 之 diff 對 AC-9 相關變更必為 0 行。
- **GUARDIAN (#11)** — 驗證 HMAC-SHA256 簽章 token 紀律端對端保持；replay cache 正確繼承 Plan52 TTL ≥ 金鑰輪替重疊（CV-06）；log 中無 plaintext parentAgentId 洩漏。

每位人格 (persona) 提交簽核裁決於 `discussions/cycle03-17/MR6_audit_log.md`。自動化 CI gate（Plan52 §5 繼承擴展）：Core 表面 diff > 0 即 block PR + escalate 至 tri-party 人工檢視。

失敗處置：任一 tri-party FAIL → Plan54 v0.52.0-alpha tag **不**發行；cycle 03-17 R3 重啟 AC-9 架構檢視；依 Rule #75 §75.X 等價 halt policy 觸發 Master letter §五 escalation。

## 6. HMAC-SHA256 + 演算法前綴 + Local-CLI MAY-Omit（CV-02/04/05）

- **預設演算法** HMAC-SHA256（CV-02 重申）；Ed25519 已記錄但**非** AC-9 Plan54 預設（continuity discipline；ZT-1）。
- **演算法前綴強制**（CV-04）：`tokenSig = "<algo>:<encoding>:<value>"`，例如 `"hmac-sha256:hex:7a3b4c2e..."`。Plugin 內驗證（**非** Core）；前綴缺失 / 不識別 → `reason: "tokenSig_algo_prefix_missing"`。
- **Local-CLI MAY-omit**（CV-05）：同 OS process 內子代理；plugin SHOULD 依靠 process-local UID + 記憶體位址空間隔離。**不**延伸至跨 process 或 capability-restrict plugins。
- **Replay cache TTL** ≥ 金鑰輪替重疊（CV-06；預設 24h；可設定）。於 v0.52.0-alpha，AC-9 plugin **必須**與 Plan52 transport plugins 共用 replay cache（單 process 部署）**或**記載協調 cache 拓樸（多 process 部署）。前向（MR-12）— 既有 Plan52 部署**不**回溯改裝。
- **Nonce ≥ 16 bytes 熵**，來自 CSPRNG（CV-03 重申）。

## 7. MAX_SPAWN_DEPTH = 4 + Operator 覆寫（D-04）

### 7.1 常數位置

```
apps/runner/src/agent-composition/config.ts:
  export const MAX_SPAWN_DEPTH_DEFAULT = 4;
```

Plugin 內部 const，**非** Core。Candidate B（Core const）已於 R3 D-01 REJECTED。

### 7.2 Operator 覆寫

優先序：per-spawn > config 檔 > env var > 預設。
- `OPENSTARRY_MAX_SPAWN_DEPTH=<int>`（範圍 1..16；超出範圍回到預設 + WARN log）。
- `agent-composition.config.json` 欄位 `maxSpawnDepth: number`。
- `SpawnChildRequest.overrideMaxDepth?: number`（延至 Plan55+，需 operator 核發 grant token）。

### 7.3 Audit 日誌

每次覆寫發出結構化 INFO log（時戳 / 來源 / 覆寫值 / 預設值 / operator UID）。F-15 v3 reflexive audit 適用。

### 7.4 失敗模式

`spawnDepth + 1 > MAX_SPAWN_DEPTH` →
```
SpawnChildResponse:
  success: false
  reason: "max_spawn_depth_exceeded"
  state: 'aborted'
```
F-16 SHOULD-initial 結構化錯誤（CV-09 重申；FORBIDDEN-phrasings 6 patterns binding — 不得用 "race condition" / "weird state" / "shouldn't happen" / "TODO fix later" / "transient" / "harmless"）。

## 8. Quota / 資源限制（Plugin 層）

- `MAX_ACTIVE_SUBAGENTS_GLOBAL = 64`（env 覆寫 1..1024）。
- `MAX_ACTIVE_SUBAGENTS_PER_PARENT = 8`。
- `DEFAULT_LLM_TOKEN_BUDGET_PER_SPAWN = 32_000` tokens（僅 telemetry hook；下游 plugin 強制執行依 F-14）。
- `ORPHAN_GRACE_WINDOW_MS = 30_000`（父代理終止 → 子進入 orphaned 狀態；30 秒 flush 後強制清理）。
- 背壓：`success: false`，`reason ∈ {spawn_capacity_exhausted, parent_quota_exhausted, global_quota_exhausted}`。重試語意 = plugin 政策（**非** AC-9 spec 強制）。

## 9. LOC 預算（D-13 全體一致 23/0）

- **Production code：800 LOC 上限**（涵蓋 `apps/runner/src/agent-composition/**` + `@openstarry/sdk` AC-9 schema 增添）。
- **Test code：500 LOC 上限**（unit + integration + cross-OS CI fixtures）。

指示性配額（R1 §1.3 baseline + R2 refinement）：

| 模組 | 指示 prod LOC | 指示 test LOC |
|------|--------------:|--------------:|
| `agent-composition/spawn.ts` | ~150 | ~120 |
| `agent-composition/lifecycle.ts` | ~120 | ~100 |
| `agent-composition/boundary.ts` | ~80 | ~60 |
| `agent-composition/quota.ts` | ~80 | ~60 |
| `agent-composition/config.ts` | ~40 | ~20 |
| SDK schema 增添 | ~80 | ~80 |
| Plan51 hook-registry 整合 | ~60 | ~40 |
| 雜項（log、errors、types） | ~70 | ~20 |
| **合計** | **~680** | **~500** |

上限留 ~120 prod buffer（15%）給 cycle 03-17 R0/R1 refinement。Test 上限為精確值（500 LOC）— 嚴格。

LOC accounting 依 cycle 03-16 D-12：主要量度 = Δ vs 中位數和指示量；次要量度 = Δ vs Master letter 上限 800。Cycle 03-17 delivery_report.md 必須回報兩者。

## 10. F-13 / F-14 / F-15 v3 + Rule #74 + Rule #75 Reflexive

- **F-13 Hook Dispatch 可驗證性** — v0.52.0-alpha smoke test runtime probe 觸發 spawn → 驗證所有 5 個 lifecycle hooks 派發；`cat=undefined` auto-FAIL（零容忍）。
- **F-14 外部資源基準** — 子代理 LLM 呼叫**必須**讓 provider token counter 每個活動 spawn 增加 ≥ 1（zero-baseline auto-FAIL）；`active_subagent_count` gauge 增/減；`spawnDepth` gauge。
- **F-15 v3 Reflexive Scope** — front-matter 7 欄位 ✓；cross_refs 至 R1+R2+R3+Plan52 ✓；epistemic 前綴 `BINDING (R4 final; pending Master Ratification Batch 13 #1)` ✓。
- **ENG-FAB v1.8（48 items）all-MUST PASS 前置條件**於 v0.52.0-alpha tag。依 Rule #75 §75.X cross-OS partial-PASS halt policy（D-05 cycle 03-16）：cross-OS 不對稱 PASS → tag **不**發行。
- **Rule #74 L1'（Code + Doc 對等）** — 5 個子檢查於 cycle 03-17 delivery_report.md（CV-07 重申）。
- **Rule #78 + F-15 v3** — TW sibling MANDATORY same-PR-strict 依 D-09 UNANIMOUS。

## 11. plugin-loader 5-Criterion 依賴對照（D-02）

Plan54 spec 穩定 = **C1 GREEN** 給 plugin-loader cycle 03-17 評估（`Reference/13_Plugin_Loader_Cycle03_17_Evaluation_Criteria.md`）。C2-C5 於 cycle 03-17 R0 依矩陣 Row 1-5 + DSS-A4 honour clause 衡量。**§一（AC-9）必須先於 §四（plugin-loader）的 sequencing pin 鎖定**依 MRB-§1-01。

## 12. 實作順序（cycle 03-17 Dev Kickoff）

| 順序 | Plan | 進入 cycle 03-17 時的狀態 |
|:---:|------|--------------------------|
| 1 | Plan49 gear-arbiter retrofit | 已落地（cycle 03-13） |
| 2 | Plan50 σ_regime in-place revise | 已落地（cycle 03-13） |
| 3 | Plan52 pushInput Candidate B | 已出貨 v0.50.0-alpha |
| 4 | Plan51 4 模組 | shipping 中（cycle 03-16 W2-R16 verification） |
| 5 | **Plan54 AC-9（本 SPEC）** | **R4 BINDING；cycle 03-17 Dev kickoff** |

Plan54 實作 tag target：`v0.52.0-alpha`。

Plan53 保留（依 cycle 03-14 D-§1-02 DSS-1 結轉；Plan52 B 路線成功 → 永不啟用 → 維持保留）。**Plan54 整數編號依 D-11 19/4**（子編號暗示 errata-like 緊急插入，依 cycle 03-7 Plan-32.5 先例；AC-9 為全新 Plan）。

Phase 6 前向 roadmap（informational；無 MUST 主張，依 MR-5 hard）：Plan55 Multi-IVolition（cycle 03-18 spec 目標）/ Plan56 VasanaEngine（cycle 03-20）/ Plan57 Mesh（cycle 03-22）/ Plan58 API Runtime（cycle 03-23）/ Plan59 Blackboard-Alaya（cycle 03-24）。7/7 functional 落地目標 cycle 03-25 保守估計。

## 13. 哲學註釋（D-18 ADOPT non-binding doc-only，18/5）

### 13.1 NAGARJUNA Madhyamaka — 世俗諦 / 勝義諦

Lifecycle 狀態 `{spawned, active, completed, aborted, orphaned}` 為**世俗諦**（saṃvṛti-satya） — 使 plugin 協調得以進行的有用虛構。**非**勝義諦（paramārtha-satya）：「spawn」無 svabhāva（自性）— 存在的是處理結點上瞬時的訊息事件，依約定標籤化。狀態機是協調便利，**非**關於代理的形上主張。

操作上：doc-only。Plugin 作者可以、亦可以不內化此觀點；AC-9 plugin 行為的正確性**不**依賴哲學吸收。F-15 v3 third-tier prefix discipline 認可。

（*DSS-5-philosophical 結轉依 MR-11 保留：勝義諦批評持續存在；本註釋僅吸收世俗諦半邊。*）

### 13.2 PENROSE 意識免責

AC-9 Plan54 對子代理內在經驗、qualia、現象意識**不作**形上主張。組合 primitives **純粹功能性** — 訊息流 + 資源控制，**非**主觀經驗。子代理是否「有意識」於任何現象學意義上，**屬工程範圍之外**。F-15 v3 epistemic level：`DOC-ONLY-PHILOSOPHY`。

### 13.3 Disposition

兩項註釋落於 `Reference/`（非 `Technical_Specifications/`）。本 spec 引用但**不**綁定實作。Cycle 03-17 Dev **不得**將其視為架構約束。

## 14. Compliance Audit（R4 快照）

| 約束 | 狀態 | 證據 |
|------|:---:|------|
| MR-2 + MR-4（Tenet 措辭不改） | PASS | 0 Tenet 措辭變更 |
| MR-5 hard（Tenet #10 status 不變） | PASS | §1 明確聲明；§12 7/7 框架僅 informational |
| MR-6（Core 零） | PASS | §3.1 ε-surface 相對 Plan52 baseline = 0 行 |
| MR-7 | PASS | 無預先 COMPLIANT 主張 |
| MR-9（無預先 MUST） | PASS | F-16 SHOULD-initial 不變；CV-09 重申 |
| MR-10 + MR-12（前向） | PASS | §3.3 前向；§6 既有 Plan52 部署不回溯 |
| MR-11（保留少數意見） | PASS | §16 — 24 entries（10 cycle 03-16 NEW + 9 carryover + 5 absorbed/internal） |
| MR-13 standby | PASS | (A) Full retained；未啟動 |
| ZT-1（十宣言） | PASS | 無 Tenet 文字或狀態修改 |
| ZT-2（endpoint 10/0/0★） | PASS | 9/0/1★ ACTIVE 保留 |
| ZT-3（控制範圍） | PASS | σ_regime 依 Rule #77 不變；W2-R16 N=5 < 10 不觸發 SPC |
| Rule #74 L1' | PASS | §10 — 5 子檢查 delivery_report 義務 |
| Rule #75 §75.X | PASS | §10 cross-OS partial-PASS halt 引用（D-05） |
| Rule #78 TW parity | PASS | §1 — TW sibling MANDATORY same-PR-strict（D-09 + MRB-§5-§78-A） |
| ENG-FAB v1.8（48）MUST | 前置 | §10 — 全 48 個 MUST PASS 於 v0.52.0-alpha |
| F-13 / F-14 / F-15 v3 reflexive | PASS | §10 |
| F-16 SHOULD initial | PASS | §7.4 結構化錯誤；無預先 MUST 升級（CV-09）；FORBIDDEN-phrasings 0 hits |
| Wording sweep | PASS | σ / V11 / "CONDITIONAL PASS" 僅於 MR-6 audit context |
| GP-coherence gate | PASS | front-matter 完備；sibling cross_refs 雙向 |

## 15. 實作 Handoff（cycle 03-17 R0）

R0 dispatch inputs：本 spec + plugin-loader 5-criterion（`Reference/13`）+ Plan51 first-shipping retrospective + W2-R16/R17 裁決 + F-15 v3 sub-mech 2/3/4-table/5 Plan-spec scope（D-03 ratified）+ MR6 audit_log.md scaffold + TW siblings MANDATORY same-PR-strict 於 v0.52.0-alpha（D-09 + MRB-§5-§78-A；Rule #78 §78.5 + §78.8）。

R0 dispatch outputs：AC-9 plugin Plan-spec（Wave breakdown ≤ 4 Waves 依 S-1）+ cycle 03-17 R0 orientation doc + subagent dispatch matrix。

Risk register（cycle 03-17 entry）：MR-6 ε-surface drift HIGH（緩解：tri-party + automated CI）/ Plan51 hook-registry W2-R16 FAIL MED（MRB-§1-01 暫停 cycle 03-17 Dev）/ LOC overshoot LOW（15% buffer）/ Cross-OS divergence MED（Rule #75 §75.X halt）/ ENG-FAB v1.8 PARTIAL HIGH（F-13/14/15 reflexive checks）/ Sub-agent quota race MED（property tests + 30s grace）。

## 16. 少數意見保留（依 MR-11 逐字）

24 條 entries 依 MR-11 逐字保留。Cycle 03-16 R3 NEW = 10（DSS-CY16-01..09 + DSS-CY16-NUM + DSS-CY16-CHAIR-1）+ carryover = 9 + absorbed/closures = 5。**S6 ≥ 20 floor PASS**。

重點（完整文字見 `research record/cycle03-16/deliver/O1_AC9_Plan54_final.md` §16）：

- **DSS-CY16-01**（D-01；共 9 票） — 「Candidate A full plugin Plan52 同構在工程風險最小；C orchestrator-as-plugin 提供 LEIBNIZ 多代理對稱；D 延後保留為 DSS-1 結轉（LEIBNIZ + RUSSELL B+B' 時序不對稱顧慮）」
- **DSS-CY16-03**（D-04；6 票） — 「MAX_SPAWN_DEPTH=3 配合記載 use-case scope 較佳；資源隔離較緊；4 可能引發非預期深層遞迴」
- **DSS-CY16-NUM**（D-11；4 票） — 「Plan52a 子編號保留鏈式一致性；Plan54 跳過 Plan53 保留」
- **DSS-CY16-06**（D-18；5 票） — 「哲學漂移顧慮（cycle-03-13 D-20 6-vote 先例）；採用邊際價值 vs 文件成本」
- **DSS-CY16-CHAIR-1**（D-01 Stage 2 決選；4 票 B 結轉） — 「Candidate B hybrid + 1-2 Core consts 提供 slippery-slope 風險平衡；A 之 MR-6 Core 零原始而 B 顯式 at-bounds-but-controlled」
- **DSS-1**（cycle 03-14 carryover） — LEIBNIZ + RUSSELL B+B' 時序不對稱；逐字保留；AC-9 Plan54 通過 ≠ DSS-1 關閉（一般顧慮對 Phase 6 5/7 sequencing 持續）。

---

*Plan54 — AC-9 子代理組合 Candidate A — CANDIDATE pending Master Ratification Batch 13 #1*
*作者：LEIBNIZ + RUSSELL + TANENBAUM + ARCHIMEDES + SUNYATA + SYNTHESIST*
*7 R3 D-items 吸收（D-01 / D-02 / D-03 / D-04 / D-11 / D-13 / D-18）+ 5 MRB resolved + 20 CV reaffirmed + 24 dissent preserved（S6 ≥20 PASS）*
*MR-6 Core 零 ε-surface（Plan52 isomorph；0 new fields beyond Plan52 baseline）/ MR-5 hard Tenet #10 status 不變 / endpoint 10/0/0★ 不變依 ZT-2*
*Phase 6 進度：2/7 functional spec（cycle 03-17 實作後）；7/7 目標 cycle 03-25 保守*
*TW sibling：本檔依 Rule #78 §78.5 + §78.8 + D-09 UNANIMOUS 採取 MANDATORY same-PR-strict*
