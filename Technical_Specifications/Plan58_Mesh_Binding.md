# Plan58 — Mesh BINDING Specification

> ✅ **[實作狀態 — v0.59.6] Mesh plugin 已完整實作並掛載（SHIPPED）。** 本規格的核心面已落地為實際代碼，不再是「pending Batch 18」的待批准 spec：
> - **集中式 hub broker**：`openstarry_plugin/mesh/src/broker.ts` `createMeshBroker()`（HMAC-SHA256 驗證 `verifyMessageHmac` L69-82；`msh:` nonce replay cache L113-116；fan-out 投遞 L124-131，源插件不回送 L128）。
> - **路由表 boot-time 編譯 + 環檢測**：`openstarry_plugin/mesh/src/routing.ts` `compileRoutingTable()` 用 Kahn topological sort 偵測環（L90-109）＋ self-cycle 拒絕（L79-81）＋ SHA-256 manifest integrity attestation `computeManifestIntegrityHash()`（L119-133，Plan58 §2.4 verification 7）。
> - **plugin factory**：`openstarry_plugin/mesh/src/plugin.ts` `createMeshPlugin()`（skandha `rupa`；FIX-2026-06-11 boot 硬化 L21-46）。
> - **SDK 型別**（API 權威）：`packages/sdk/src/types/mesh.ts`（`MeshRoutingRuleSchema`/`MeshMessageSchema`/`MeshPublishResultSchema`/`MESH_REPLAY_CACHE_PREFIX = 'msh:'`），經 `packages/sdk/src/index.ts` L221-224 匯出。
> - **23 個測試**：`__tests__/broker.test.ts`（11）＋ `__tests__/routing.test.ts`（12）。
> - **掛載活線**：`@openstarry-plugin/mesh` 依賴 `@openstarry/sdk: workspace:*`，隨 workspace 一次 build/test 涵蓋。
>
> §6 forward constraints（fan-out only、in-process single-host）為**誠實且仍生效的範圍界線**——與代碼一致（broker.ts L8-12 + L124），保留。下文 §1-§8 原文為 cycle 03-21 R3 設計時的歷史紀錄，狀態行已就地更正。

**Status**: BINDING + SHIPPED（cycle 03-21 R3 D-§1 ratified 22/1 super-majority；~~pending Master Ratification Batch 18 #2~~ → **已實作落地並掛載活線，見上方 v0.59.6 實作狀態牌**）
**Authority**: Master Ratification (Batch 18 dispatch 2026-05-02)
**Cycle**: 03-21 (Phase 6 第五棒; 5/7 functional landing)
**Release**: v0.55.0-alpha minor-bump
**Plan number**: 58
**Subject**: Mesh — distributed agent communication via centralized hub

**Inheritance chain**: Plan52 pushInput → Plan54 AC-9 → Plan56 D-30-4 → Plan57 D-30-5 → **Plan58 Mesh** (5th consecutive Phase 6 functional landing)

---

## §1 Background

Mesh enables distributed agent communication within OpenStarry's五蘊 architecture. **Architectural choice**: Option B Centralized Hub (publisher-subscriber via in-process broker; routing-table declarative at boot via plugin manifests).

**Rejected alternatives**:
- Option A Gossip protocol — over-engineered for in-process; potential MR-6 violation; poor isomorph fit (DSS-CY21-§1-A LEIBNIZ minority preserved verbatim per MR-11)
- Option C Hybrid — out-of-scope cycle 03-21

## §2 Architecture: Option B Centralized Hub

### §2.1 Plan52/54/56/57/58 isomorph 10-dimension verbatim

| Dimension | Plan58 Mesh |
|-----------|-------------|
| Plugin layer | mesh broker plugin |
| Core surface delta | 0 fields/const |
| ε-surface check | 7-sub-check ci_check (verbatim inheritance cycle 03-17) |
| Tri-party MR-6 audit | TANENBAUM + KERNEL + GUARDIAN |
| HMAC-SHA256 + nonce | yes |
| Replay cache | 5-contributor (Plan52+54+56+57+58) |
| Cross-OS CI | Linux + Windows + Tier δ extension |
| F-13/F-14/F-15 v3 reflexive | PASS |
| TW sibling Rule #78 BINDING-tier | yes |
| LOC ceiling | 600-900 prod / 400-600 test |

### §2.2 Replay cache 5-contributor structured prefix

| Contributor | Plugin | Prefix |
|:-----------:|--------|:------:|
| 1 | Plan52 pushInput | `psh:` |
| 2 | Plan54 AC-9 | `ac9:` |
| 3 | Plan56 D-30-4 | `mvq:` |
| 4 | Plan57 D-30-5 (post-refactor plugin form) | `vsn:` |
| **5** | **Plan58 Mesh** | **`msh:`** |

### §2.3 Routing-rule schema (per D-§1-R2-A clarifications a+b+c)

a. Source plugin manifest declares `mesh_routes: [{topic, target_plugins[]}]`
b. Mesh broker compiles routing-table at boot
c. Cycle detection via Kahn's topological sort (D-§1-R2-B)

### §2.4 6-verification baseline (extends Plan57's 5)

1-5: inherited from Plan57
6. Routing-rule cycle-detection (Kahn's topological sort)
7. **NEW** Manifest integrity SHA-256 attestation (D-§1-R2-D; T1 mitigation per GUARDIAN R2)

### §2.5 5-layer abstraction barrier extension

L1 plugin manifests → L2 broker compile → L3 routing dispatcher → L4 NEG/POS verification → L5 Mesh-specific active routing dispatcher (NEW)

## §3 ε-surface 0-delta Strict Equality (7-sub-check inheritance verbatim)

Per cycle 03-17 D-§1-R2-B + Plan57 §3 inheritance.

## §4 Tri-party MR-6 Sign-off

TANENBAUM (Plan-spec authority) + KERNEL (Core surface) + GUARDIAN (security; Plan58-specific evidence: routing-table SHA-256 attestation + REJECT-by-default collision per D-§1-R2-C).

## §5 LOC Trajectory

| Checkpoint | When | prod | test |
|-----------|------|------|------|
| CP-1 R4 close | NOW | indicative ~720 / ceiling 900 | indicative ~510 / ceiling 600 |
| CP-2 Dev mid | 2026-05-03 | drift > 10% reassess | drift > 10% reassess |
| CP-3 Pre-tag v0.55.0-alpha | 2026-05-04 | hard ≤ 900 | hard ≤ 600 |

**Buffer at CP-1**: prod ~25% / test ~18%.

## §6 Forward Constraints

- **Fan-out only this cycle** (per D-§1-R2-E; aggregation deferred to Phase 7 — DSS-CY21-§1-D LEIBNIZ aggregation-now preserved)
- **In-process single-host this cycle** (per D-§1-clarify; cross-process Mesh forward-binding Phase 7)

## §7 Dissent Preservation per MR-11

3 NEW DSS-CY21 entries:
- **DSS-CY21-§1-A** (LEIBNIZ, 1 vote): "Gossip protocol B' minimal-routing variant offers stronger multi-agent decentralization semantics; defer to Option B for cycle 03-21 first-shipping; cycle 03-22+ revisit if multi-arbiter pattern emerges."
- **DSS-CY21-§1-B** (KERNEL, 1 vote): "N=8 hex preferred for forensic balance over N=4 alphanumeric; carryover from DSS-CY18-02/CY19-§1-C."
- **DSS-CY21-§1-D** (LEIBNIZ, 1 vote): "Mesh aggregation-mode capability could ship same cycle for higher initial value; defer aggregation Phase 7 acceptable but documented."

## §8 Compliance

| Constraint | Status |
|------------|:------:|
| MR-5 hard / MR-6 鐵律 / MR-9 / MR-11 / MR-12 | ✅ PASS |
| ZT-1/2/3 | ✅ PASS |
| Rule #74/75/76/77/78 | ✅ PASS (Rule #75 §75.X 8th-enforced) |
| F-13/14/15 v3 reflexive | ✅ PASS |
| Phase 6 strict 7-list anchor | ✅ Plan58 = 5th of 7 |

---

*Plan58 Mesh BINDING Specification — cycle 03-21 R3 D-§1 ratified 22/1 super-majority — 2026-05-02*
*~~Master Ratification Batch 18 #2 dispatch ready~~ → SHIPPED：mesh plugin 已實作落地並掛載活線（broker/routing/plugin + SDK types + 23 tests）；見頂部 v0.59.6 實作狀態牌 — 2026-06-16*
*Inheritance: Plan52 → Plan54 → Plan56 → Plan57 → Plan58 (5/7 Phase 6 functional)*
