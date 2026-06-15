# Plan57 D-30-5 VasanaEngine — Cycle 03-21 修訂（Refactor → Plugin Form；TW sibling）

**Status**: BINDING amendment（cycle 03-21 R3 D-§0 ratified 23/0 UNANIMOUS 6 items；Master Ratification Batch 18 #3 已 APPROVED 2026-05-02）
**Authority**: Master Ratification（Batch 18 dispatch 2026-05-02 / `deliver/Master_Ratification/Master_Confirmation.md` 10/10 APPROVED）
**Cycle**: 03-21（Phase 7 elevation 先驅範例）
**Source amendment**: cycle 03-21 R3 D-§0 + R2 §0（SUSSMAN+KERNEL+DARWIN tri-lens）
**Target document**: `Technical_Specifications/Plan57_D30_5_VasanaEngine_Binding.md`（cycle 03-19 ratified BINDING canonical）
**MR-12 forward-only**: 原 Plan57 binding 文本逐字保留；本修訂為 forward addendum（form change only；0 function change）
**TW sibling 義務**: per Rule #78 §78.5 BINDING-tier reflexive same-PR；對應 EN 主檔 `Plan57_D30_5_VasanaEngine_Binding_amendment_cycle03-21.md`

---

## §1 架構契合度合理性說明

### §1.1 Refactor 前狀態（cycle 03-19 v0.54.0-alpha）

VasanaEngine 在 cycle 03-19 ratified 落於 `apps/runner/src/vasana-engine/`（runner host 應用層）。MR-6 Core 零 letter 合規（不在 `packages/core`），但**架構契合度斷裂**：
- 依 `openstarry_eco/CLAUDE.md` 五蘊架構：行蘊 (Samskara) → ITool plugin
- VasanaEngine = 行蘊 種子化痕跡 → SHOULD = ITool plugin
- 實作在 runner-level → 不一致

### §1.2 Refactor 後狀態（cycle 03-21 v0.55.0-alpha）

VasanaEngine 移至 `openstarry_plugin/vasana-engine/`：
- **Form change**：runner-level → plugin layer（五蘊 行蘊 → ITool plugin pattern 嚴格對齊）
- **Function 保留**：4-method API surface 不變（deposit / verify_chain / count / latest_hash；SICP-canonical）
- **Phase 7 elevation 先驅範例**：Plan52/54/56/58 batch elevate 規劃於 cycle 03-25+ post-novel S-4

## §2 6 Items Ratified（D-§0-A 至 D-§0-F；ALL UNANIMOUS 23/0）

### D-§0-A：架構選擇 — Refactor → Plugin Form

`apps/runner/src/vasana-engine/` → `openstarry_plugin/vasana-engine/`。Factory pattern：`createVasanaEnginePlugin(manifest, factory(ctx))`，對齊 OpenStarry plugin convention。

### D-§0-B：8 AMEND 精修 bundle（per R2 SUSSMAN+KERNEL+DARWIN）

1. **Dual-barrier disambiguation**（外層 4-method consumer surface vs 內層 container-plugin lifecycle protocol）
2. **ci_check attestation log** 新增 `implementation_locus` 欄位（追蹤 refactor 落地位置）
3. **Plugin loader onBoot fail-fast** codification（不允許 soft-fail；reject-on-startup）
4. **HMAC secret key derivation path** 明文承諾不變
5. **SHA-256 manifest integrity attestation**（擴展 boot-time verification）
6. **4-property R/S/C/G template**（Refactor / SICP / Compatibility / Greenfield）作為 Phase 7 batch elevation 引用範式
7. **6-verification baseline**（在 Plan57 5 項基礎上加 append-only re-verification）
8. **§四 Plan55 plugin-loader 終評** cross-link，作為首個 emergent test case

### D-§0-C：MR-12 既有不破壞 strict

| 既有 v0.54.0-alpha | v0.55.0-alpha plugin form | Δ |
|---|---|:---:|
| 4-method API surface | 4-method API surface | 0 |
| HMAC-SHA256 signed-token | HMAC-SHA256 signed-token | 0 |
| Boot-time refuse-to-start | Boot-time refuse-to-start | 0 |
| Replay cache 4-contributor | **5-contributor**（依 cycle 03-19 模式擴展；Plan58 Mesh 加 `msh:`） | EXTEND only |
| ε-surface delta vs Plan52 | 0 fields / 0 const | 0 |
| 既有 deposit log entries | 逐字保留 | 0 |

### D-§0-D：ε-surface 0-delta strict（7-sub-check inheritance）

依 cycle 03-17 D-§1-R2-B 7-sub-check 逐字繼承。KERNEL R2 獨立驗證 PASS（7/7）。

### D-§0-E：HMAC-SHA256 + Tri-party MR-6 + Replay cache 5-contributor

`vsn:` prefix 從 runner-level 遷移到 plugin layer 經由 factory pattern；資安保證悉數保留。

### D-§0-F：Plan57 spec amendment dispatch + TW sibling

本修訂主檔 + 本 TW sibling（即本檔）per Rule #78 §78.5 BINDING-tier reflexive。

## §3 4-property R/S/C/G template（per D-§0-B AMEND-6；Phase 7 forward）

供 Phase 7 batch elevation Plan52/54/56/58 → plugin form 引用：

- **R（Refactor）**：form change only；0 function change
- **S（SICP-canonical）**：API surface 保留（Black-box invariant）
- **C（Compatibility）**：MR-12 既有不破壞；既有 data + entries 保留
- **G（Greenfield）**：新 factory + manifest + container-plugin lifecycle 整合；對 consumer 0 干擾

## §4 Cycle 03-22+ Forward

- 本修訂自 cycle 03-22+ 起 effective forward
- 原 Plan57 binding 文本依 MR-12 forward-only 保留
- Phase 7 batch elevation（Plan52/54/56/58）規劃 cycle 03-25+ post-novel S-4
- Coordinator G5 sync 階段同步 amendment 並建立本 TW sibling 至 canonical

## §5 合規

| Constraint | Status |
|------------|:------:|
| MR-12 forward-only | ✅ amendment 屬 forward addendum |
| MR-11 dissent | ✅ 修訂 ratification 0 dissent（全 23/0 UNANIMOUS） |
| MR-6 鐵律 | ✅ plugin layer（非 Core） |
| Rule #78 §78.5 TW sibling | ✅ amendment.tw sibling 即本檔 |
| F-15 v3 schema | ✅ |
| Phase 6 strict 7-list anchor | ✅ Plan57 = 4/7（form change 不影響計數） |

---

*Plan57 D-30-5 VasanaEngine Cycle 03-21 修訂 — Refactor → Plugin Form（TW sibling）— 2026-05-02*
*cycle 03-21 R3 D-§0 ratified 23/0 UNANIMOUS 6 items*
*Master Ratification Batch 18 #3 dispatch ready / Master_Confirmation 10/10 APPROVED*
*Phase 7 elevation 先驅範例（Plan52/54/56/58 batch elevate cycle 03-25+ forward）*
*EN 主檔同步：`Plan57_D30_5_VasanaEngine_Binding_amendment_cycle03-21.md`*
