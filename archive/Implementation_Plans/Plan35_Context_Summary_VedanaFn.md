# Plan35: Context Summary Plugin + VedanaFn Wiring

**Version**: v0.35.0-alpha
**Cycle**: 20260316_cycle02-11
**SOP**: Standard
**Status**: IN PROGRESS

---

## Objective

Deliver a second IContextManager implementation (context-summary) to upgrade Tenet #9 "Pluggable Context Strategy" from CONDITIONAL to COMPLIANT, and wire VedanaRegistry aggregate into ManoAggregator to establish the vedana feedback interface.

**Compliance target**: 7/2/1 → **8/1/1** (Tenet #9 CONDITIONAL → COMPLIANT)

## Design Decisions

| Key Decision | Choice | Rationale |
|-------------|--------|-----------|
| KD-1 | Dir B (識蘊深化) over Dir A (受蘊) | LEIBNIZ weighted score 4.45/5 vs 2.35/5 (D3-R1) |
| KD-2 | context-summary as IContextManager plugin | Tenet #9 requires ≥2 IContextManager implementations (D3-R2) |
| KD-3 | vedanaFn wiring via VedanaRegistry.aggregate() | Establishes vedana feedback interface; zero-sensor backward compat required (D3-R4) |
| KD-4 | W3 guide-persistent deferred to Plan36 | Engineering resource allocation; W1+W2 core = ~285 LOC sufficient for compliance goal |
| KD-5 | BABBAGE continuity test mandatory for W2 | agent-core.ts Core-layer change must preserve identical behavior when VedanaRegistry is empty (D3-R4) |

## Architecture

### Tenet #9 Upgrade Path

Current state: only `@openstarry-plugin/context-sliding-window` implements IContextManager.

After Plan35 W1: `@openstarry-plugin/context-sliding-window` + `@openstarry-plugin/context-summary` → IContextManager ≥ 2 implementations → Tenet #9 **COMPLIANT**.

Both strategies are selectable via PluginHooks.contextManager (last-wins policy, configured by which plugin is loaded).

### VedanaFn Wiring Architecture

```
VedanaRegistry (Core)
  └── sensors: Map<string, IVedanaSensor>  (currently empty — no sensor plugins yet)
  └── aggregate(): ChannelVedana           (neutral when empty)
        │
        ▼
agent-core.ts start()
  └── createManoAggregator(..., vedanaFn, ...)
        │                         ▲
        │                    () => vedanaRegistry.aggregate()
        ▼
ManoAggregator
  └── checkVedanaEmergency(vedanaFn())
        └── sensors=[] → neutral → no boost (backward compat ✓)
```

## Implementation Waves

### Wave 1: context-summary plugin (~255 LOC) — P0 必要

**Location**: `agent_dev/openstarry_plugin/context-summary/`

**Design**: Summary-based context management strategy that compresses older conversation history into summaries while preserving recent turns verbatim.

| File | Description | LOC |
|------|-------------|-----|
| `src/index.ts` | Plugin factory `createContextSummaryPlugin()` + manifest | ~30 |
| `src/context.ts` | IContextManager implementation: `assembleContext()` with summary compression | ~120 |
| `src/summarizer.ts` | Message summarization logic (template-based, no LLM dependency) | ~60 |
| `src/__tests__/context-summary.test.ts` | Unit tests | ~45 |
| `package.json` + `tsconfig.json` | Package config | — |

**Plugin manifest**:
```typescript
{
  name: '@openstarry-plugin/context-summary',
  version: '0.1.0-alpha',
  description: 'Summary-based context manager — compresses older turns into summaries',
  skandha: 'samjna',  // 想蘊 — cognitive strategy
}
```

**Key behavior**:
- `assembleContext(messages, maxTurns)`: keeps last N turns verbatim, summarizes older turns into a single system message
- Summary threshold configurable via `SummaryContextConfig.summaryThreshold` (default: when messages exceed 2x maxTurns)
- Preserves system messages unconditionally
- Falls back to sliding-window behavior when message count is below threshold

### Wave 2: vedanaFn wiring (~30 LOC) — P1 強烈建議

**Location**: `agent_dev/openstarry/packages/core/src/agents/agent-core.ts`

**Change**: Wire VedanaRegistry's aggregate function as vedanaFn parameter to createManoAggregator.

**Current code** (line 423-428):
```typescript
_manoAggregator = createManoAggregator(
  bus, resolvedManoConfig, undefined, undefined, undefined,
  pluginAuditor ?? undefined,
  loopQualityFn,
  resolvedConfidenceAuditConfig,
);
```

**Target code**:
```typescript
// Plan35 W2: Wire vedanaFn — VedanaRegistry aggregate provides vedana feedback
// BABBAGE continuity: when registry has no sensors, aggregate() returns neutral vedana
// which produces identical behavior to undefined vedanaFn (zero boost)
const vedanaFn = vedanaRegistry.list().length > 0
  ? () => vedanaRegistry.aggregate()
  : undefined;

_manoAggregator = createManoAggregator(
  bus, resolvedManoConfig, undefined, vedanaFn, undefined,
  pluginAuditor ?? undefined,
  loopQualityFn,
  resolvedConfidenceAuditConfig,
);
```

**BABBAGE continuity test requirements** (D3-R4):
1. VedanaRegistry empty → vedanaFn = undefined → behavior identical to pre-wiring
2. `checkVedanaEmergency()` with no sensors → no threshold change
3. All existing agent-core test suite passes without modification

### Wave 3: guide-persistent plugin (~150 LOC) — P2 可選，deferred to Plan36

Deferred per KD-4. If engineering resources allow, may be pulled into Plan35 scope. Requires:
- 二諦聲明 (Two Truths Declaration) — Layer 2 standard
- IPersistentGuide maps to 前六識動態分別功能 (mano-vijnana), not 第七末那識 (manas)
- `clearDirectives()` excludability confirms non-permanence (D3-R5)

## Test Plan

| Test | Description | Wave |
|------|-------------|------|
| context-summary unit tests | assembleContext with various message counts, summary threshold, system message preservation | W1 |
| IContextManager contract test | Verify context-summary satisfies same interface as sliding-window | W1 |
| Plugin loading test | Verify context-summary loads via PluginHooks.contextManager | W1 |
| Existing agent-core tests | All 1876+ tests pass unchanged after vedanaFn wiring | W2 |
| vedanaFn backward compat | Empty VedanaRegistry → no behavioral change (BABBAGE continuity) | W2 |
| Purity check | `pnpm test:purity` — Core remains pure after W2 change | W2 |

## PROC Rule Compliance

| Rule | Action |
|------|--------|
| PROC-SKANDHA-1 | context-summary: `skandha: 'samjna'` (想蘊) — verified correct |
| PROC-DOC-1 | Four-file sync: Iteration_Log + Roadmap + README + openstarry_doc/README.md |
| PROC-DOC-5 | Any hotfix during Plan35 → record in Iteration_Log |

## Metrics (Expected)

| Metric | Before | After |
|--------|--------|-------|
| Tenet compliance | 7/2/1 | **8/1/1** |
| Total tests | 1876 | ~1920+ |
| Production LOC | — | +~285 (W1+W2) |
| Remaining CONDITIONAL | #6 (並發識) | #6 only |
| Remaining NON-COMPLIANT | #10 (Phase 6) | #10 only |

## Dependencies

- Plan34 complete (v0.34.0-alpha) ✓
- `@openstarry-plugin/context-sliding-window` exists as reference implementation ✓
- VedanaRegistry + createManoAggregator vedanaFn parameter exist ✓
- IVedanaSensor interface defined in SDK ✓

## Risk Register

| Risk | Mitigation |
|------|-----------|
| Summary compression loses critical context | Conservative threshold (2x maxTurns); system messages always preserved |
| vedanaFn wiring changes Core behavior | BABBAGE continuity test; conditional wiring only when sensors registered |
| W3 scope creep | Explicitly deferred to Plan36; KD-4 decision recorded |

---

*Plan35 Architecture Spec*
*Standard SOP Phase 0*
*2026-03-16*
