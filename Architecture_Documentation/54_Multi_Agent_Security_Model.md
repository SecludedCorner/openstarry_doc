# 58. 多代理安全模型 (Multi-Agent Security Model)

> [Cycle 03-2 D4-R5] 本文件定義 OpenStarry 多代理環境的安全模型，包含攻擊分類、信任等級、防禦機制、和覆蓋率分析。

---

## 1. 安全哲學：信任即空性 (Trust is Sunyata)

> Lead: NAGARJUNA (#7)

信任不具有自性（svabhava）。信任因六種條件之聚合而生起（pratityasamutpada）：

1. **身份驗證** (Identity Verification) — 代理的 PID-to-agentId 映射已確認
2. **能力授予** (Capability Assignment) — 代理擁有執行操作的 capability
3. **協議遵循** (Protocol Adherence) — 代理遵守通訊協議和驗證鏈
4. **時間有效性** (Temporal Validity) — 代理的 session 和 token 尚未過期
5. **情境適當性** (Context Appropriateness) — 操作在當前系統狀態下合理
6. **雙方同意** (Mutual Consent) — sender 和 receiver 均有相應的 capability

移除任一條件即移除信任。此為零信任架構的中觀學派 (Madhyamaka) 基礎。

---

## 2. 信任等級 (Trust Levels)

> Lead: LINNAEUS (#13)

| Level | Name | Description | Capabilities |
|-------|------|------------|-------------|
| T0 | Untrusted | 未註冊的進程 | 無。僅能嘗試 register_agent |
| T1 | Registered | 已完成 register_agent | 可被發現 (list_agents)；可接收 broadcast |
| T2 | Capable | 已被授予 comm capabilities | 可主動 send_message；可查詢 get_agent_status |
| T3 | Privileged | 擁有管理能力的代理 | 可 spawn/stop 子代理；可修改 capability |

**Trust 轉換**：
- T0 → T1: register_agent 成功（5 步驗證通過）
- T1 → T2: 父代理授予 comm capabilities（Rule #37: default zero）
- T2 → T3: 明確的特權授予（spawnChildAgent 權限）
- 任何 → T0: deregister 或 crash handling

**Trust 不可繼承**: 子代理的 trust level 必須獨立建立。父代理的 T3 不自動傳遞給子代理（Rule #33: child ⊆ parent，但須獨立驗證）。

---

## 3. 攻擊分類 (Attack Taxonomy)

> Lead: LINNAEUS (#13) + PENROSE (#18)

### AT-1: Message Injection（訊息注入）

| Sub-type | Description | Current Defense | Coverage |
|----------|-------------|----------------|----------|
| AT-1a | Source Spoofing — 偽造 sender identity | SEC-002 PID-to-agentId (Plan38 W0) | PARTIAL → FULL (Plan38) |
| AT-1b | Replay Attack — 重放有效訊息 | MessageRouter id-dedup + timestamp 新鮮度窗（`MAX_MESSAGE_AGE_MS`/`MAX_CLOCK_SKEW_MS`，v0.59.6 `message-router.ts` `checkReplay`） | FULL |
| AT-1c | EventBridge Injection — 注入惡意事件 | EventBridge sender validation | PARTIAL |

### AT-2: Capability Escalation（能力提升）

| Sub-type | Description | Current Defense | Coverage |
|----------|-------------|----------------|----------|
| AT-2a | Initial Over-Grant — 初始 capability 過度授予 | Rule #37 zero-default | FULL |
| AT-2b | Runtime Expansion — 運行時非法擴展 capability | Type contract (ICompositeAgent) | PARTIAL |
| AT-2c | Service Impersonation — 冒充其他代理 | SEC-002 identity verification (Plan38 W0) | PARTIAL → FULL |
| AT-2d | Path Traversal — 利用路徑穿越存取受限資源 | SEC-003 isPathSafe() (Plan38 W0) | PARTIAL → FULL |

### AT-3: Denial of Service（阻斷服務）

| Sub-type | Description | Current Defense | Coverage |
|----------|-------------|----------------|----------|
| AT-3a | Message Flood — 訊息洪泛 | Rate limiting (Plan38 W5: per-agent 100/s) | NONE → FULL |
| AT-3b | EventBridge Flood — 事件洪泛 | Rate limiting (Plan38 W5: per-target 20/s) | NONE → FULL |
| AT-3c | Registry Pollution — 大量虛假註冊 | Message size limit + registration cap | PARTIAL |

### AT-4: Information Disclosure（資訊洩露）

| Sub-type | Description | Current Defense | Coverage |
|----------|-------------|----------------|----------|
| AT-4a | Message Eavesdrop — 竊聽訊息 | IPC isolation (Unix sockets) | PARTIAL |
| AT-4b | Registry Enumeration — 列舉所有代理 | list_agents: basic only; get_agent_status: capability-gated | PARTIAL |
| AT-4c | Audit Log Access — 存取審計日誌 | File system permissions | PARTIAL |
| AT-4d | Service Discovery Leak — 服務發現資訊外洩 | GlobalServiceRegistry capability check | PARTIAL |

### AT-5: Replay Attacks（重放攻擊）

| Sub-type | Description | Current Defense | Coverage |
|----------|-------------|----------------|----------|
| AT-5a | Message Replay — 重放有效訊息 | MessageRouter 訊息 id 去重 + timestamp 新鮮度窗（已接受的 id 入快取、過窗剪除；重放/陳舊/未來時戳 fail-closed 拒收，v0.59.6 `checkReplay`，`message-router.test.ts` 25 測試） | FULL |
| AT-5b | Credential Replay — 重放認證資訊 | Session-bound registration | PARTIAL |

### AT-6: Observation Interference（觀測干擾）

> PENROSE (#18) 提出

| Sub-type | Description | Current Defense | Coverage |
|----------|-------------|----------------|----------|
| AT-6a | Audit Overhead DoS — 審計系統本身成為瓶頸 | Fire-and-forget + JSONL rotation | PARTIAL |
| AT-6b | Feedback Loop Attack — 操縱觸發 audit → auto-response → 行為改變 | No feedback path from audit to delta (by design) | FULL |
| AT-6c | Audit Log Tampering — 篡改審計日誌 | SHA-256 hash chain (Rule #30) | FULL |

---

## 4. 防禦機制清單 (Defense Mechanism Inventory)

> Lead: GUARDIAN (#11)

### Existing (v0.37.0-alpha)

| Mechanism | Layer | Type | Protects Against |
|-----------|-------|------|-----------------|
| MessageRouter 6-layer chain | Daemon | Mechanism | AT-1a, AT-2b, AT-3c |
| Rule #37 zero-capability default | Daemon | Mechanism | AT-2a |
| SEC-001 drain-evasion guard | Daemon | Mechanism | AT-3 (spawn variant) |
| SEC-004 broadcast sender check | Daemon | Mechanism | AT-1a (broadcast) |
| SHA-256 audit hash chain (Rule #30) | Plugin | Mechanism | AT-6c |
| ICommChannel FROZEN interface | SDK | Mechanism | AT-2b (type safety) |

### Plan38 Additions

| Mechanism | Wave | Type | Protects Against | LOC |
|-----------|------|------|-----------------|-----|
| SEC-002 PID-to-agentId identity | W0 | Mechanism | AT-1a, AT-2c | ~15 |
| SEC-003 isPathSafe() for configPath | W0 | Mechanism | AT-2d | ~20 |
| SEC-005 traceDepth input validation | W0 | Mechanism | AT-1c (indirect) | ~5 |
| Per-agent rate limiting (100 msg/s) | W5 | Mechanism+Policy | AT-3a | ~15 |
| Per-target rate limiting (20 msg/s) | W5 | Mechanism+Policy | AT-3b | ~15 |
| Message size limit (Zod) | W5 | Mechanism+Policy | AT-3c | ~5 |
| L2 Circuit Breaker | W2 | Mechanism+Policy | AT-3a (cascade prevention) | ~100 |
| L3 Bulkhead | W2 | Mechanism+Policy | AT-3a (resource isolation) | ~80 |
| F-5 permission lattice | W3 | Mechanism | AT-2b (runtime enforcement) | ~125 |
| get_agent_status capability gate | W1 | Mechanism | AT-4b, AT-4d | ~20 |

---

## 5. 覆蓋率矩陣 (Coverage Matrix)

> Lead: LINNAEUS (#13)

### Current (v0.37.0-alpha): 5/21 FULL (23.8%)

| Category | FULL | PARTIAL | NONE |
|----------|------|---------|------|
| AT-1 (3) | 0 | 2 | 1 |
| AT-2 (4) | 1 | 2 | 1 |
| AT-3 (3) | 0 | 1 | 2 |
| AT-4 (4) | 0 | 4 | 0 |
| AT-5 (2) | 0 | 1 | 1 |
| AT-6 (3) | 2 | 1 | 0 |
| **Total (21)** | **5** | **11** | **5** |

### Post-Plan38 Target: >= 11/21 FULL (>= 52.4%)

Plan38 closes:
- AT-1a: PARTIAL → **FULL** (SEC-002)
- AT-2c: PARTIAL → **FULL** (SEC-002)
- AT-2d: PARTIAL → **FULL** (SEC-003)
- AT-3a: NONE → **FULL** (rate limiting)
- AT-3b: NONE → **FULL** (rate limiting)
- AT-3c: PARTIAL → **FULL** (message size limit)

Predicted post-Plan38: **11/21 FULL** (52.4%), meeting the >= 50% target.

---

## 6. 已知缺口與修復路徑 (Known Gaps + Remediation Roadmap)

> Lead: LINNAEUS (#13) + DARWIN (#6)

> ⚠️ **[v0.59.6 更新]** 此表 §5 覆蓋矩陣的 `Current 5/21 (v0.37.0-alpha)` 為**陳舊基線**——多數 Plan38 防禦（SEC-002/005/008、MessageRouter 鏈、DualRateLimiter、PermissionLattice、SpawnValidator）其實已落地接線（見各 §AT 表 `→ FULL` 列＋對應測試）。「Message replay prevention（AT-1b/5a）」已於 v0.59.6 由 MessageRouter `checkReplay` 補上（id 去重＋timestamp 新鮮度窗），不再屬未竟。其餘列（IPC 加密 AT-4a、credential rotation AT-5b、event signing AT-1c）仍為誠實未竟。

| Gap | Attack Type | Target Plan | Approach |
|-----|-----------|-------------|----------|
| ~~Message replay prevention~~ ✅ 已做 (v0.59.6) | AT-1b, AT-5a | ~~Plan39~~ | MessageRouter id-dedup + timestamp window (`checkReplay`) |
| EventBridge injection hardening | AT-1c | Plan39 | Event source signing |
| Runtime capability expansion | AT-2b | Plan38 W3 (F-5) | Permission lattice runtime enforcement |
| Audit overhead rate limiting | AT-6a | Plan39 | Sampled auditing under high load |
| IPC encryption | AT-4a | Plan40+ | TLS for cross-machine communication |
| Credential replay | AT-5b | Plan39 | Session token rotation |

---

## 7. 觀測者效應與審計設計 (Observer Effect and Audit Design)

> Lead: PENROSE (#18)

### 設計原則

1. **審計不影響行為**: audit trail 的寫入不觸發 delta 計算或任何 feedback loop（by design）。AT-6b 因此被 FULLY defended。

2. **審計不成為瓶頸**: fire-and-forget 模式 + JSONL rotation。若訊息速率超過閾值，切換為 sampled auditing：
   ```
   sample_rate = min(1.0, audit_capacity / actual_rate)
   ```
   Sampling threshold = **Policy** (configurable)。

3. **審計不可篡改**: per-entry SHA-256 hash chain (Rule #30)。每條 audit entry 包含前一條的 hash，形成不可變鏈。chain-reset on rotation。

### 機制/策略分類

| Item | Classification |
|------|---------------|
| Hash chain algorithm | Mechanism |
| Rotation size/period | Policy |
| Sampling threshold | Policy |
| fire-and-forget pattern | Mechanism |

### 7.5 拒絕審計（denial audit，v0.59.7-alpha LANDED）

上述 fail-closed 防禦（AT-3a/3b rate limiting、SEC-001 drain-evasion、SEC-003 path traversal、capability/depth/budget/ceiling lattice）在拒絕一個請求時，**過去只回 RPC 錯誤、不留審計軌跡**——攻擊者的探測在守護行程側不可見。v0.59.7 起，守護行程把每次拒絕經 Plan48 audit-sink 落成一筆 `agent_request_denied` 事件（與 §7 的信心度 audit hash-chain 是不同機制；此處是安全拒絕可見性）：

| 拒絕來源 | `reason` | `detail` 範例 |
|---|---|---|
| `agent.input` 超速（AT-3a/3b，-32005） | `rate_limited` | `per_agent:sessionId` |
| `agent.spawnChild` 被拒（SEC-001/003、capability、depth/budget/ceiling） | `spawn_constraint` | `DRAINING`／`PATH_TRAVERSAL`／`CEILING_EXCEEDED`… |

- opt-in（`OPENSTARRY_AUDIT=1`／`AUDIT_SINK_PATH`，env 未設＝no-op，零行為變更）；經 `(timestamp, event_hash)` 去重；關機 flush 落盤。
- 實作：`apps/runner/src/daemon/daemon-entry.ts`（rate-limit catch + `handleSpawnChild` 各拒絕點 `obs.publishAgentRequestDenied`）、事件型別 `apps/runner/src/audit-sink/audit-bus.ts`。
- e2e（真實 daemon）：`apps/runner/__tests__/e2e/daemon-observability.e2e.test.ts`；生命週期記錄見 [Tech Spec 18](../Technical_Specifications/18_Structured_Log.md)。
- **AT-4c（Audit Log Access）維持 PARTIAL**：審計檔仍僅靠檔案系統權限保護，未加密／未簽章——不誇大。

---

## 8. 機制 vs 策略分類 (Mechanism vs Policy Classification)

> Lead: BABBAGE (#9) + NAGARJUNA (#7)

| Defense | Mechanism (Non-bypassable) | Policy (Configurable) |
|---------|---------------------------|----------------------|
| MessageRouter validation chain | 6-layer sequence, fail-closed | — |
| Rate limiting | Token bucket algorithm | Threshold values (100/s, 20/s) |
| Circuit breaker | 3-state machine, ordering | failureThreshold, cooldownMs |
| Bulkhead | Pool management, reject on full | maxConcurrent, maxQueue |
| Identity verification | PID-to-agentId binding | — |
| Path validation | isPathSafe() + symlink defense | allowedPaths list |
| Trust levels | T0-T3 definitions, transition rules | — |
| Capability model | Zero-default, child ⊆ parent | Specific capability grants |
| Audit hash chain | SHA-256 per-entry chaining | Rotation policy |
| Message size limit | Zod schema validation | MAX_MESSAGE_SIZE value |

**哲學基礎** (NAGARJUNA): 機制如金剛不壞——不因條件改變而改變。策略如方便善巧——隨情境調整以達最佳效果。兩者分離確保安全基礎不被配置侵蝕。

---

*Doc 54: Multi-Agent Security Model*
*Version: v0.37.0-alpha (design for Plan38)*
*Authors: LINNAEUS (#13), GUARDIAN (#11), PENROSE (#18), NAGARJUNA (#7), BABBAGE (#9), DARWIN (#6)*
*[Source: Cycle 03-2 D4-R5]*
