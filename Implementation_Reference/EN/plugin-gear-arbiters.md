# Plugin Gear Arbiters — identity, priority, and null-output contract

**Status**: Plan49 C49-M7e (O4 doc artefact; D-11 UNANIMOUS).
**Effective from**: v0.49.0-alpha (2026-04-24).
**Canonical peer**: `share/openstarry_doc/Architecture_Documentation/42_IGearArbiter_Interface_Spec.md` (Plan42 interface freeze).

## 1. Purpose

Clarify the by-design behaviour of the registered gear-arbiter plugins, specifically `@openstarry-plugin/gear-arbiter-dynamic`, to prevent misreading its intended null-output as a silent plugin failure. Companion to Plan49 C49-M7a (`riskCategory` propagation via `audit:completed`) and the Plan50 rename scope (D-01 20/3).

## 2. Identity — what each arbiter is

| Plugin | Role | `riskCategory` output |
|--------|------|----------------------|
| `@openstarry-plugin/gear-arbiter-static` | Static rule-based routing | populated per matched rule |
| `@openstarry-plugin/gear-arbiter-dynamic` | **WIENER control-theory adaptive routing** (Plan41 CV-5; "dynamic" = runtime-dynamic delta-based math, NOT LLM-dynamic) | **by design `undefined`** — see §3 |

## 3. `gear-arbiter-dynamic` null-output contract

`gear-arbiter-dynamic` is a **pure-WIENER-math arbiter**: its outputs are a function of the StateTracker observation history and the calibration bridge deltas, with no category classification logic. It emits `riskCategory === undefined` on every `GearEvaluation` it produces.

This is the **intended null output**. It is NOT:

- A silent plugin failure.
- A bug to fix.
- A sign the plugin is uninstalled or misconfigured.
- A violation of any `audit:completed` schema contract (the field is optional — `riskCategory?: RiskCategory` on `GearEvaluation`, `riskCategory?: string` on `ConfidenceAuditEntry`).

Downstream consumers **must** treat `undefined` as a first-class value equivalent to "no category claim" and fall back to:

- Rule-based routing from a sibling arbiter (e.g., `gear-arbiter-static`) if composed in a chain.
- `gearEvaluation.routeResult`'s `riskCategory` if it has been populated at a later pipeline stage.
- Omitting risk-category-dependent audit columns (leaving them `null`).

## 4. Priority and chain composition

Per `42_IGearArbiter_Interface_Spec.md`, arbiters are evaluated in declared priority order; the first arbiter whose confidence clears the risk-weighted threshold wins. `gear-arbiter-dynamic` is typically installed alongside `gear-arbiter-static` and placed at a lower priority so static rules short-circuit before the dynamic pass engages — but this is deployment-configurable, not a hard constraint.

## 5. Plan49 — Plan50 sequencing (D-01 20/3)

Plan49 preserves the name `gear-arbiter-dynamic`. The rename `→ gear-arbiter-wiener` is **tabled to Plan50** per D-01 20/3, with 3 dissent entries (WIENER, LEIBNIZ, TANENBAUM) preferring in-Plan49 rename preserved per MR-11.

Plan50 will ship: (a) atomic rename, (b) dual-name support window, (c) migration guide, (d) deprecation-warning schedule. No operational change is expected until Plan50.

## 6. `audit:completed` consumer forward-compatibility (C49-M7f)

Plan49 C49-M7a attests that the `riskCategory` field on the `audit:completed` event payload is optional and pre-existing (Plan32 Wave 5 P0). TURING-style consumer audit confirms all in-tree consumers ignore-unknown or treat missing `riskCategory` as null, satisfying MR-9 forward-compat guarantee.

Known consumers (as of v0.49.0-alpha):

- `packages/core/src/observability/audit-trail-writer.ts` (writes optional field to JSONL; treats `undefined` as absent).
- `packages/core/src/mano/mano-aggregator.ts` (threshold computation; `riskCategory` absent → default base threshold).
- Downstream plugins (WebSocket transport, structured-log forwarders): receive envelope wholesale, do not gate on `riskCategory` presence.

## 7. References

- Plan49 engineering spec §2.7 (C49-M7): `share/research_team_suggestion/cycle03-13/deliver/O2_plan49_engineering_spec.md`
- Plan49 dev spec §1.2 C49-M7: `share/research_team_suggestion/cycle03-13/todo/Plan49_dev_spec.md`
- D-11 UNANIMOUS (O3+O4 composite): `share/research_team_suggestion/cycle03-13/deliver/R3_decision_log.md §4.5`
- D-01 20/3 (Plan50 rename tabled): same file; dissent preservation via MR-11
- IGearArbiter interface: `share/openstarry_doc/Architecture_Documentation/42_IGearArbiter_Interface_Spec.md`
- gear-arbiter-dynamic source: `openstarry_plugin/gear-arbiter-dynamic/src/index.ts`
- audit-completed type: `packages/sdk/src/types/confidence-audit-log.ts`
