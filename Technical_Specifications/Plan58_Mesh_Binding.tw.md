# Plan58 — Mesh BINDING 規格（TW sibling）

> ✅ **[實作狀態 — v0.59.6]** 本規格**已落地出貨**。mesh plugin 為真實 `@openstarry-plugin/mesh`（package.json `version` 0.1.0-alpha），活在生產路徑：
> - `openstarry_plugin/mesh/src/plugin.ts:12` `createMeshPlugin()` → `IPlugin`（manifest `skandha: 'rupa'` 色蘊通訊基質）
> - `openstarry_plugin/mesh/src/broker.ts:84` `createMeshBroker()`：HMAC-SHA256 verify（`verifyMessageHmac` broker.ts:69）+ nonce 重放防禦（`msh:` 前綴 `MESH_REPLAY_CACHE_PREFIX` broker.ts:113）+ fan-out 投遞（broker.ts:124-131，**僅 fan-out**符合 §6 D-§1-R2-E）
> - `openstarry_plugin/mesh/src/routing.ts:38` `compileRoutingTable()`：boot 階段編譯 + Kahn 拓撲排序循環偵測（routing.ts:90-109）；`computeManifestIntegrityHash()`（routing.ts:119）= §2.4 verification 7 的 SHA-256 manifest attestation
>
> 測試覆蓋 **23 例**（真跑 broker/routing，非樁）：`__tests__/routing.test.ts`（12）+ `__tests__/broker.test.ts`（11）。
> 下方 §5 LOC「indicative」、§6「本輪僅 fan-out / in-process single-host」與 §8 合規表為 cycle 03-21 規劃期快照；fan-out-only 與 in-process 約束在已出貨代碼中仍成立（cross-process / aggregation 仍為 Phase 7 未建，誠實標記為大未來）。原 BINDING 規劃文字保留於下供歷史對照。

**Status**: BINDING + SHIPPED（cycle 03-21 R3 D-§1 ratified 22/1 super-majority；Master Ratification Batch 18 #2 已 APPROVED 2026-05-02；代碼於 `@openstarry-plugin/mesh` 出貨，broker.ts/routing.ts/plugin.ts + 23 測試——狀態於 v0.59.6 文件對代碼複審回填）
**Authority**: Master Ratification（Batch 18 dispatch 2026-05-02 / `deliver/Master_Ratification/Master_Confirmation.md` 10/10 APPROVED）
**Cycle**: 03-21（Phase 6 第五棒；5/7 functional landing）
**Release**: v0.55.0-alpha minor-bump
**Plan number**: 58
**Subject**: Mesh — 透過中央化 hub 實現分散式 agent 通訊
**TW sibling 義務**: per Rule #78 §78.5 BINDING-tier reflexive same-PR；對應 EN 主檔 `Plan58_Mesh_Binding.md`

**Inheritance chain**: Plan52 pushInput → Plan54 AC-9 → Plan56 D-30-4 → Plan57 D-30-5 → **Plan58 Mesh**（Phase 6 連續第五棒 functional landing）

---

## §1 背景

Mesh 在 OpenStarry 五蘊架構內提供分散式 agent 通訊能力。**架構選擇**：Option B Centralized Hub（in-process broker 實現 publisher-subscriber；routing-table 於 boot 階段透過 plugin manifest 宣告式組裝）。

**已否決替代方案**：
- Option A Gossip protocol — 對 in-process 場景過度設計；潛在 MR-6 違規；isomorph 契合度低（DSS-CY21-§1-A LEIBNIZ 少數派 per MR-11 逐字保留）
- Option C Hybrid — cycle 03-21 out-of-scope

## §2 架構：Option B Centralized Hub

### §2.1 Plan52/54/56/57/58 isomorph 10 維度逐字繼承

| 維度 | Plan58 Mesh |
|-----|-------------|
| Plugin layer | mesh broker plugin |
| Core surface delta | 0 fields/const |
| ε-surface check | 7-sub-check ci_check（cycle 03-17 起逐字繼承） |
| Tri-party MR-6 audit | TANENBAUM + KERNEL + GUARDIAN |
| HMAC-SHA256 + nonce | 是 |
| Replay cache | 5-contributor（Plan52+54+56+57+58） |
| Cross-OS CI | Linux + Windows + Tier δ extension |
| F-13/F-14/F-15 v3 reflexive | PASS |
| TW sibling Rule #78 BINDING-tier | 是（即本檔） |
| LOC ceiling | 600-900 prod / 400-600 test |

### §2.2 Replay cache 5-contributor structured prefix

| Contributor | Plugin | Prefix |
|:-----------:|--------|:------:|
| 1 | Plan52 pushInput | `psh:` |
| 2 | Plan54 AC-9 | `ac9:` |
| 3 | Plan56 D-30-4 | `mvq:` |
| 4 | Plan57 D-30-5（refactor 後 plugin form） | `vsn:` |
| **5** | **Plan58 Mesh** | **`msh:`** |

### §2.3 Routing-rule schema（依 D-§1-R2-A 釐清項 a+b+c）

a. Source plugin manifest 宣告 `mesh_routes: [{topic, target_plugins[]}]`
b. Mesh broker 於 boot 階段編譯 routing-table
c. 循環偵測採 Kahn 拓撲排序（D-§1-R2-B）

### §2.4 6-verification baseline（在 Plan57 5 項基礎上擴展）

1-5：Plan57 既有逐字繼承
6. Routing-rule cycle-detection（Kahn 拓撲排序）
7. **新增** Manifest integrity SHA-256 attestation（D-§1-R2-D；T1 mitigation per GUARDIAN R2）

### §2.5 5-layer 抽象屏障擴展

L1 plugin manifests → L2 broker compile → L3 routing dispatcher → L4 NEG/POS verification → L5 Mesh-specific active routing dispatcher（新增）

## §3 ε-surface 0-delta 嚴格相等性（7-sub-check 逐字繼承）

依 cycle 03-17 D-§1-R2-B 與 Plan57 §3 逐字繼承。

## §4 Tri-party MR-6 sign-off

TANENBAUM（Plan-spec authority）+ KERNEL（Core surface）+ GUARDIAN（資安；Plan58 專屬證據：routing-table SHA-256 attestation + REJECT-by-default collision per D-§1-R2-C）。

## §5 LOC 軌跡

| Checkpoint | 時機 | prod | test |
|-----------|------|------|------|
| CP-1 R4 close | NOW | indicative ~720 / ceiling 900 | indicative ~510 / ceiling 600 |
| CP-2 Dev mid | 2026-05-03 | 漂移 > 10% 重評 | 漂移 > 10% 重評 |
| CP-3 Pre-tag v0.55.0-alpha | 2026-05-04 | hard ≤ 900 | hard ≤ 600 |

**CP-1 緩衝**：prod ~25% / test ~18%。

## §6 前向約束

- **本輪僅 fan-out**（依 D-§1-R2-E；aggregation 延後 Phase 7 — DSS-CY21-§1-D LEIBNIZ aggregation-now 保留）
- **本輪僅 in-process single-host**（依 D-§1-clarify；cross-process Mesh forward-binding Phase 7）

## §7 異議保留 per MR-11

3 項 NEW DSS-CY21：
- **DSS-CY21-§1-A**（LEIBNIZ，1 票）：「Gossip protocol B' minimal-routing 變體在多 agent 去中心語意上更強；本輪 cycle 03-21 first-shipping 接受 Option B；cycle 03-22+ 若出現 multi-arbiter pattern 則重新檢視。」
- **DSS-CY21-§1-B**（KERNEL，1 票）：「N=8 hex 在鑑識平衡上優於 N=4 alphanumeric；繼承自 DSS-CY18-02/CY19-§1-C。」
- **DSS-CY21-§1-D**（LEIBNIZ，1 票）：「Mesh aggregation-mode capability 同輪 ship 可獲更高初值；接受延 Phase 7 但留紀錄。」

## §8 合規

| Constraint | Status |
|------------|:------:|
| MR-5 hard / MR-6 鐵律 / MR-9 / MR-11 / MR-12 | ✅ PASS |
| ZT-1/2/3 | ✅ PASS |
| Rule #74/75/76/77/78 | ✅ PASS（Rule #75 §75.X 第 8 次 enforce） |
| F-13/14/15 v3 reflexive | ✅ PASS |
| Phase 6 strict 7-list anchor | ✅ Plan58 = 5/7 |

---

*Plan58 Mesh BINDING 規格（TW sibling）— cycle 03-21 R3 D-§1 ratified 22/1 super-majority — 2026-05-02*
*Master Ratification Batch 18 #2 dispatch ready / Master_Confirmation 10/10 APPROVED*
*Inheritance：Plan52 → Plan54 → Plan56 → Plan57 → Plan58（5/7 Phase 6 functional）*
*EN 主檔同步：`Plan58_Mesh_Binding.md`*
