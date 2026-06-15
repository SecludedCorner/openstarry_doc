# Plan57 D-30-5 VasanaEngine — Amendment Cycle 03-21 (Refactor → Plugin Form)

**Status**: BINDING amendment (cycle 03-21 R3 D-§0 ratified 23/0 UNANIMOUS 6 items; pending Master Ratification Batch 18 #3)
**Authority**: Master Ratification (Batch 18 dispatch 2026-05-02)
**Cycle**: 03-21 (Phase 7 elevation 先驅範例)
**Source amendment**: cycle 03-21 R3 D-§0 + R2 §0 (SUSSMAN+KERNEL+DARWIN tri-lens)
**Target document**: `Technical_Specifications/Plan57_D30_5_VasanaEngine_Binding.md` (cycle 03-19 ratified BINDING canonical)
**MR-12 forward-only**: original Plan57 binding text preserved; this amendment is forward addendum (form change only; 0 function change)

---

## §1 Architectural Coherence Rationale

### §1.1 Pre-Refactor State (cycle 03-19 v0.54.0-alpha)

VasanaEngine ratified at `apps/runner/src/vasana-engine/` (runner host application level). MR-6 Core 零 letter compliant (NOT in `packages/core`), but **architectural coherence break**:
- Per `openstarry_eco/CLAUDE.md` 五蘊 architecture: 行蘊 (Samskara) → ITool plugin
- VasanaEngine = 行蘊 種子化痕跡 → SHOULD = ITool plugin
- Implemented as runner-level → NOT consistent

### §1.2 Post-Refactor State (cycle 03-21 v0.55.0-alpha)

VasanaEngine moved to `openstarry_plugin/vasana-engine/`:
- **Form change**: runner-level → plugin layer (五蘊 行蘊 → ITool plugin pattern strict alignment)
- **Function preserved**: 4-method API surface 不變 (deposit / verify_chain / count / latest_hash; SICP-canonical)
- **Phase 7 elevation 先驅範例**: Plan52/54/56/58 batch elevate cycle 03-25+ post-novel S-4

## §2 6 Items Ratified (D-§0-A through D-§0-F; ALL UNANIMOUS 23/0)

### D-§0-A: Architecture Choice — Refactor → Plugin Form

`apps/runner/src/vasana-engine/` → `openstarry_plugin/vasana-engine/`. Factory pattern: `createVasanaEnginePlugin(manifest, factory(ctx))` per OpenStarry plugin convention.

### D-§0-B: 8 AMEND Refinements Bundle (per R2 SUSSMAN+KERNEL+DARWIN)

1. **Dual-barrier disambiguation** (outer 4-method consumer surface vs inner container-plugin lifecycle protocol)
2. **ci_check attestation log** `implementation_locus` field (track refactor implementation location)
3. **Plugin loader onBoot fail-fast** codification (no soft-fail; reject-on-startup)
4. **HMAC secret key derivation path** explicit unchanged commitment
5. **SHA-256 manifest integrity attestation** (extends boot-time verification)
6. **4-property R/S/C/G template** (Refactor / SICP / Compatibility / Greenfield) for Phase 7 batch elevation reference
7. **6-verification baseline** (extends Plan57 5 with append-only re-verification)
8. **Cross-link to §四 Plan55 plugin-loader 終評** as first emergent test case

### D-§0-C: MR-12 既有不破壞 strict

| 既有 v0.54.0-alpha | v0.55.0-alpha plugin form | Δ |
|---|---|:---:|
| 4-method API surface | 4-method API surface | 0 |
| HMAC-SHA256 signed-token | HMAC-SHA256 signed-token | 0 |
| Boot-time refuse-to-start | Boot-time refuse-to-start | 0 |
| Replay cache 4-contributor | **5-contributor** (extends per cycle 03-19 pattern; Plan58 Mesh adds `msh:`) | EXTEND only |
| ε-surface delta vs Plan52 | 0 fields / 0 const | 0 |
| Existing deposit log entries | Preserved verbatim | 0 |

### D-§0-D: ε-surface 0-delta strict (7-sub-check inheritance)

Per cycle 03-17 D-§1-R2-B 7-sub-check verbatim. KERNEL R2 independent verification PASS (7/7).

### D-§0-E: HMAC-SHA256 + Tri-party MR-6 + Replay cache 5-contributor

`vsn:` prefix moves from runner-level to plugin layer via factory pattern; security guarantees preserved.

### D-§0-F: Plan57 Spec Amendment Dispatch + TW Sibling

This amendment doc + `Plan57_D30_5_VasanaEngine_Binding_amendment_cycle03-21.tw.md` per Rule #78 §78.5 BINDING-tier reflexive.

## §3 4-Property R/S/C/G Template (per D-§0-B AMEND-6; Phase 7 Forward)

For Phase 7 batch elevation of Plan52/54/56/58 to plugin form:

- **R (Refactor)**: form change only; 0 function change
- **S (SICP-canonical)**: API surface preservation (Black-box invariant)
- **C (Compatibility)**: MR-12 既有不破壞; existing data + entries preserved
- **G (Greenfield)**: new factory + manifest + container-plugin lifecycle integration; 0 disruption to consumers

## §4 Cycle 03-22+ Forward

- This amendment effective forward cycle 03-22+
- Original Plan57 binding text preserved per MR-12 forward-only
- Phase 7 batch elevation (Plan52/54/56/58) cycle 03-25+ post-novel S-4
- Coordinator G5 sync stage applies amendment + creates `Plan57_D30_5_VasanaEngine_Binding_amendment_cycle03-21.tw.md` TW sibling

## §5 Compliance

| Constraint | Status |
|------------|:------:|
| MR-12 forward-only | ✅ amendment is forward addendum |
| MR-11 dissent | ✅ 0 dissent at amendment ratification (all 23/0 UNANIMOUS) |
| MR-6 鐵律 | ✅ plugin layer (not Core) |
| Rule #78 §78.5 TW sibling | ✅ amendment.tw sibling required |
| F-15 v3 schema | ✅ |
| Phase 6 strict 7-list anchor | ✅ Plan57 = 4/7 (form change preserves count) |

---

*Plan57 D-30-5 VasanaEngine Amendment Cycle 03-21 — Refactor → Plugin Form — 2026-05-02*
*cycle 03-21 R3 D-§0 ratified 23/0 UNANIMOUS 6 items*
*Master Ratification Batch 18 #3 dispatch ready*
*Phase 7 elevation 先驅範例 (Plan52/54/56/58 batch elevate cycle 03-25+ forward)*
