# 59. AC-7 分散式阿賴耶運行時 (AC-7 Distributed Alaya Runtime)

**Status**: Cycle 20260404_cycle03-3 PASS  
**Version**: v0.39.0-alpha  
**Plan**: Plan39 Engineering Specification  
**Author**: architect  
**Date**: 2026-04-04

---

## 1. 概述

AC-7 分散式阿賴耶運行時（Distributed Alaya Runtime）是 Tenet #6（八識俱轉與功能性代理）的實現核心，將第八識（阿賴耶識）從單 Agent 本地儲存提升至跨 Agent 分散式狀態共識。本架構透過以下機制實現：

1. **IBijaStore** — 本地種子儲存 (Agent 級別)
2. **SeedSignatureService** — HMAC-SHA256 簽名驗證
3. **向量時鐘 (Vector Clock)** — 因果排序
4. **分散式共識協議** — 1-Agent 容錯
5. **Plant/Propagate 生命週期** — 顯式傳播（非自動化）

---

## 2. 架構原理

### 2.1 種子的本體論

在 OpenStarry 哲學框架中：

- **種子 (ISeed)** 是代理「意識的種子」，代表意識的潛力狀態
- **種子的三不變性質**:
  - `skandha`: 五蘊分類（Rupa, Vedana, Samjna, Samskara, Vijnana）
  - `agentId`: 所有者代理（不可轉移）
  - `id`: 全局唯一識別符

### 2.2 八識映射

| 識別 | 梵文名 | 對應 | AC-7 實現 |
|------|--------|------|----------|
| 前五識 | Pancha-Vijnana | 五個感官意識 | IListener + 五蘊聽眾 |
| 第六識 | Mano-Vijnana | 意識心智 | IProvider (推理引擎) |
| 第七識 | Klitta-Mano | 末那識 (我執) | IGuide (身份框架) |
| 第八識 | Alaya-Vijnana | **阿賴耶識 (藏識)** | **AC-7 Distributed Alaya** ← 本文件範圍 |

### 2.3 分散性的必要性

在多 Agent 系統中，單 Agent 的本地「意識儲存」不足以支撐整個網絡的狀態同步。AC-7 透過：

1. **顯式傳播** — 種子從一個 Agent 傳播至他人時，接收方必須主動驗證簽名、檢驗時間戳、更新向量時鐘
2. **共識而非同步** — 目標非完全一致性，而是在 1-Agent 容錯下的可用共識
3. **不可逆傳播** — 種子一經植入，無法撤回；但可被更新（透過 SeedPatch 的不可變欄位限制）

---

## 3. 核心介面定義

### 3.1 IBijaStore 介面

**檔案**: `agent_dev/openstarry/packages/sdk/src/types/distributed-alaya.ts:299`

```typescript
export interface IBijaStore {
  /** 植入新種子。驗證 agentId === 所有者代理 (F-8)。 */
  plant(seed: ISeed): Promise<void>;
  
  /** 查詢符合篩選條件的種子。傳回副本，不傳回引用。 */
  query(filter: SeedFilter): Promise<ISeed[]>;
  
  /** 僅更新可變欄位 (SeedPatch 強制執行不可變欄位排除)。 */
  update(seedId: string, patch: SeedPatch): Promise<void>;
  
  /** 從本地儲存移除種子 (不移除已傳播副本)。 */
  remove(seedId: string): Promise<void>;
  
  /** 取得此 Agent 流的目前向量時鐘。 */
  getVectorClock(): VectorClock;
  
  /** 合併傳入的向量時鐘 (在 exchangeSeeds 後)。 */
  mergeVectorClock(incoming: VectorClock): void;
  
  /** 本地儲存中種子的計數。 */
  size(): number;
}
```

**重點**:
- `plant()` 強制執行 F-8 所有者代理驗證：發出 plant 的 Agent 的 agentId 必須與種子的 agentId 匹配
- `query()` 傳回**副本**，防止外部程式碼直接修改內部儲存
- `update()` 接受 `SeedPatch`（排除 `skandha` 和 `agentId` 的 Partial<ISeed>），在編譯時強制執行不可變性

### 3.2 ISeedSignatureService 介面

**檔案**: `agent_dev/openstarry/packages/sdk/src/types/distributed-alaya.ts:329`

```typescript
export interface ISeedSignatureService {
  /** 簽名種子的內容，傳回 HMAC-SHA256 十六進制字串。 */
  sign(seed: ISeed): Promise<string>;
  
  /** 驗證種子的簽名。簽名不符時傳回 false (故障關閉)。 */
  verify(seed: ISeed): Promise<boolean>;
}
```

**簽名機制**:
- 使用 HMAC-SHA256 + Agent 本地密鑰（從未傳送）
- 簽名驗證發生在接收端，拒絕無效簽名 (fail-closed)
- 簽名不提供加密，僅提供完整性保證

### 3.3 向量時鐘類型

**檔案**: `agent_dev/openstarry/packages/sdk/src/types/distributed-alaya.ts:272`

```typescript
/**
 * VectorClock — 分散式種子排序的邏輯時鐘。
 * 每個 Agent 維護自己的計數器；exchangeSeeds 使用元素級最大值合併時鐘
 * (標準向量時鐘合併)。
 *
 * 實作注意：鍵為 agentId，值為單調遞增計數器。計數器不會遞減。
 */
export type VectorClock = Readonly<Record<string, number>>;
```

**向量時鐘合併演算法**:
```typescript
// 偽代碼：合併兩個向量時鐘
function mergeVectorClocks(local: VectorClock, incoming: VectorClock): VectorClock {
  const merged: Record<string, number> = { ...local };
  for (const [agentId, incomingCounter] of Object.entries(incoming)) {
    merged[agentId] = Math.max(merged[agentId] ?? 0, incomingCounter);
  }
  return merged;
}
```

---

## 4. IDistributedAlaya 介面 & 生命週期

### 4.1 公開介面

**檔案**: `agent_dev/openstarry/packages/sdk/src/types/distributed-alaya.ts`

```typescript
export interface IDistributedAlaya {
  /**
   * 植入種子到此 Agent 的阿賴耶流。
   * - 驗證種子.agentId === 呼叫 Agent 身份 (F-8)
   * - 簽名種子並儲存
   */
  plant(seed: ISeed): Promise<void>;
  
  /**
   * 查詢種子。可選篩選器支援skandha/狀態/年齡範圍查詢。
   */
  query(filter?: SeedFilter): Promise<ISeed[]>;
  
  /**
   * 更新種子的可變欄位 (類型安全: SeedPatch)。
   */
  update(seedId: string, patch: SeedPatch): Promise<void>;
  
  /**
   * 移除種子 (本地操作，不影響遠程副本)。
   */
  remove(seedId: string): Promise<void>;
  
  /**
   * 與另一個 Agent 交換種子 (雙向傳播)。
   * - 驗證簽名
   * - 合併向量時鐘
   * - 解決衝突 (CRDT: 時間戳最新者贏)
   */
  exchangeSeeds(peerId: string): Promise<ExchangeResult>;
  
  /**
   * 查詢目前狀態快照 (用於系統監視)。
   */
  getState(): Promise<IAlayaState>;
}
```

### 4.2 Plant 生命週期

```
應用程式呼叫 plant(seed)
  ↓
[F-8 驗證] agentId 檢查
  ↓ (成功)
[簽名] ISeedSignatureService.sign()
  ↓
[IBijaStore.plant()]
  ├─ 檢查重複 seedId (可選) → 拒絕或更新
  └─ 儲存到本地記憶體/持久化層
  ↓
[向量時鐘] 增加此 Agent 的計數器
  ↓
✅ Promise<void> 已完成
```

### 4.3 Propagate 生命週期

當另一個 Agent 呼叫 `exchangeSeeds(this.agentId)` 時：

```
遠程 Agent 呼叫 exchangeSeeds(localAgentId)
  ↓
[本地簽名] 簽署所有待傳播種子
  ↓
[傳輸] 透過 ICommChannel 傳送 SeedPropagationRequest
  ↓
[遠程接收] 驗證簽名
  ├─ 若無效 → 拒絕此種子 (fail-closed)
  └─ 若有效 → 繼續
  ↓
[版本衝突解決] 
  ├─ 本地已有此 seedId
  │  ├─ 遠程時間戳更新 → 接受遠程
  │  └─ 遠程時間戳舊 → 保留本地
  └─ 本地無此 seedId → 接受遠程
  ↓
[向量時鐘合併] mergeVectorClocks()
  ↓
✅ 回傳 ExchangeResult { seedsExchanged, conflictsResolved, peerId, timestamp }
```

---

## 5. 共識協議

### 5.1 共識不變性

在 N 個 Agent 的網絡中，任意 1 個 Agent 可能在任何時間點故障。AC-7 共識協議保證：

1. **可用性**: 故障 Agent < N 時，系統繼續運作
2. **一致性 (最終)**: 所有活躍 Agent 最終會聚合到同一狀態（在沒有新種子加入的情況下）
3. **不可逆性**: 已植入/傳播的種子不可被移除（僅可更新）

### 5.2 故障情景

| 情景 | 結果 |
|------|------|
| 1 個 Agent 故障，3 個存活 | ✅ 存活 Agent 達成共識；故障 Agent 重新啟動時同步回追 |
| 2 個 Agent 故障，3 個存活 | ⚠️ 存活者共識；故障者重啟後可能需數輪同步 |
| N 個 Agent 全部故障 | ✅ 持久層恢復後，每個 Agent 獨立重新啟動，再次 exchangeSeeds |

### 5.3 CRDT 策略（衝突解決）

當兩個 Agent 各自植入相同 ID 的種子（不應發生，但 AC-7 容許邊界情況）：

```typescript
function resolveConflict(local: ISeed, remote: ISeed): ISeed {
  // 規則 1: 所有者代理必須相同，否則視為不同種子
  if (local.agentId !== remote.agentId) {
    throw new Error('AgentId mismatch: treat as separate seeds');
  }
  
  // 規則 2: 時間戳最新者贏
  if (remote.timestamp > local.timestamp) {
    return remote;
  }
  return local;
}
```

---

## 6. 實作詳情 (W1 交付)

### 6.1 BijaStore 實作

**檔案**: `openstarry_plugin/distributed-alaya/src/bija-store.ts`（`BijaStoreImpl`）

- 記憶體存儲 (Map<seedId, ISeed>)
- 向量時鐘追蹤
- 查詢篩選 (skandha, 狀態, 年齡範圍)
- 副本傳回保證

### 6.2 SeedSignatureService 實作

**檔案**: `openstarry_plugin/distributed-alaya/src/seed-signature.ts`（`SeedSignatureServiceImpl`）

- HMAC-SHA256 签名 (使用 crypto 模組)
- Agent 本地密鑰管理
- 簽名快取 (可選最佳化)

### 6.3 DistributedAlaya 實作

**檔案**: `openstarry_plugin/distributed-alaya/src/distributed-alaya-impl.ts`（`DistributedAlayaImpl`）

- IDistributedAlaya 完整實現 (~750 LOC，含評論和日誌)
- plant/query/update/remove/exchangeSeeds 生命週期
- 共識協議驗證
- 審計追蹤事件發射

---

## 7. 合規性對應

### 7.1 Tenet #6 映射

| 要求 | 實現 | 狀態 |
|------|------|------|
| 八識理論框架 | Doc 31 (八識識別表) | ✅ 參考 |
| 第八識行為 | AC-7 plant/exchangeSeeds/共識 | ✅ **COMPLIANT** (Plan39) |
| 跨 Agent 狀態同步 | 向量時鐘 + CRDT 衝突解決 | ✅ **COMPLIANT** |
| N>=1 消費者 | distributed-alaya 插件自動啟動 (self-activating) | ✅ **COMPLIANT** |

**評估**: 透過 Plan39 W1，Tenet #6 從 CONDITIONAL → **COMPLIANT**

### 7.2 Tenet #8 (控制理論) 關聯

AC-7 與信心審計整合（Plan31/38）：

- 種子交換事件記錄到審計追蹤 (SeedExchangeAuditEntry)
- 向量時鐘時間戳通知審計系統進行因果排序
- 與 IConfidenceAuditor 協作，追蹤多 Agent 狀態轉移

---

## 8. F-8 所有者代理強制執行

### 8.1 編譯時強制

透過 `SeedPatch` 類型（W0 修正 3）:
```typescript
export type SeedPatch = Omit<Partial<ISeed>, 'skandha' | 'agentId'>;
```

- TypeScript 拒絕任何試圖修改 `agentId` 的程式碼

### 8.2 執行時強制

在 `plant()` 和 `propagate()`:
```typescript
async plant(seed: ISeed): Promise<void> {
  // F-8: 驗證所有者
  const callingAgentId = this.ctx.agentId;
  if (seed.agentId !== callingAgentId) {
    throw new Error(
      `[F-8] Seed agentId mismatch: seed.agentId=${seed.agentId}, caller=${callingAgentId}`
    );
  }
  // ... 繼續 plant
}
```

---

## 9. 已知限制與未來改進

### 9.1 當前限制

| 限制 | 原因 | 優先級 |
|------|------|--------|
| 無跨 Agent 加密傳輸 | openstarry-channel 使用本地 IPC | P2 (生產部署需求) |
| 無持久化層 | AC-7 運行時使用記憶體儲存 | P1 (長期執行需求) |
| 向量時鐘無時間界限 | 長期運行可能導致溢位 | P2 (防禦措施) |

### 9.2 未來改進方向

- **Plan40**: AC-8（跨 Agent 持久化層）
- **Plan41**: 加密傳播通道 (AgentId-based TLS)
- **Plan42**: 故障復原與檢查點機制

---

## 10. 審計與監視

### 10.1 SeedExchangeAuditEntry

所有 exchangeSeeds 操作記錄到審計追蹤:

```typescript
export interface SeedExchangeAuditEntry extends AuditTrailEntryBase {
  readonly type: 'seed_exchanged';
  readonly seedId: string;
  readonly fromAgentId: string;
  readonly toAgentIds: readonly string[];
  readonly seedsExchanged: number;
  readonly conflictsResolved: number;
}
```

### 10.2 可觀測性

- 每次 plant/exchangeSeeds 發射 `alaya:seed_planted` / `alaya:seeds_exchanged` 事件
- 向量時鐘狀態透過 `getState()` 可查詢
- 審計 JSONL 記錄完整傳播路徑

---

## 參考文件

| 文件 | 涵蓋 |
|------|------|
| Architecture_Spec_Plan39.md | 凍結介面定義 + 波形依賴 |
| Research_Methodology/04_Ten_Tenets_Compliance_Report.md | Tenet #6 評估 (CONDITIONAL → COMPLIANT) |
| 31_Eight_Consciousnesses_Framework.md | 八識哲學背景 |
| 36_Vedana_Measurement_Model.md | 三受反饋與種子評估整合 |
| Engineering_Delivery_Plan39.md | 實作完成報告 + 約束驗證 |

---

**Status**: ✅ FROZEN (2026-04-04)  
**Version**: v0.39.0-alpha  
**Changes from v0.38.0-alpha**: +W1 AC-7 full runtime (~750 LOC)
