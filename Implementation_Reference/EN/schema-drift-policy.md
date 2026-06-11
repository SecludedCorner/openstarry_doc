# Schema-Drift Policy

**Status**: Plan49 Follow-on A (C49-M3a / M3c / M3e / M3f / M3g) — central module + call-site migration.
**Effective from**: v0.49.0-alpha (2026-04-24).
**Module**: `apps/runner/src/schema-drift-policy/index.ts`.

## 1. Purpose

A single process-wide policy module for handling unknown or invalid fields in inbound data at Zod `safeParse` boundaries. Centralises the decision that call-sites previously made ad-hoc: accept silently, throw loudly, or emit a structured-log audit event.

## 2. Modes (`SchemaDriftMode`)

| Mode | Success path | Failure path | Use case |
|------|-------------|--------------|----------|
| `tolerant` (default) | `{ ok: true, data }` | `{ ok: false, error, issues }` — no throw; caller logs or falls back | Backward-compatible behaviour pre-Plan49. |
| `strict` | `{ ok: true, data }` | `throw SchemaDriftError` | Fail-fast during CI and integration tests to surface unexpected drift. |
| `audited` | `{ ok: true, data }` | `{ ok: false, error, issues }` + `schema_drift_audit` event via wired sink | Compliance-sensitive deployments that need a tamper-resistant record of drift without hard-failing. |

## 3. Resolution order (C49-M3g process-global)

`resolveSchemaDriftMode()` reads `SCHEMA_DRIFT_MODE` env var exactly once at first call and caches the result for the lifetime of the process. Call-sites **must not** pass an `overrideMode` in production code — the override parameter exists for unit tests only. This satisfies C49-M3g (single boot-resolved mode; integration test asserts uniformity).

```
process.env.SCHEMA_DRIFT_MODE ∈ { "tolerant", "strict", "audited" }  — default "tolerant"
unknown value → falls back silently to "tolerant"
```

## 4. Call-site migration status (v0.49.0-alpha)

The Plan49 spec enumerated an aspirational 8-site migration target (event-bus ×3, checkpoint-store ×2, IAgentConfig ×1, WebSocket ×1, hook-registry ×1). An audit of the v0.48.0-alpha baseline finds **safeParse boundaries exist only where Zod schemas are explicitly invoked today**. Current reality:

| Call-site | Status | Module |
|-----------|--------|--------|
| IAgentConfig | ✅ migrated | `apps/runner/src/utils/config-validator.ts` |
| ProjectPermissions | ✅ migrated | `apps/runner/src/utils/permission-validator.ts` `validatePermissionsFile` |
| ProjectPlugins | ✅ migrated | `apps/runner/src/utils/permission-validator.ts` `validatePluginsFile` |
| ProjectConfig (strict + unknown-keys-tolerate hybrid) | ◼ kept as-is | `apps/runner/src/utils/permission-validator.ts` `validateConfigFile` — has deliberate site-specific semantics (warn on unknown keys, retry with neutral fields) that are incompatible with uniform policy. Documented as intentional divergence. |
| `packages/shared/src/utils/validation.ts` generic helper | ◼ kept as-is | Architectural: `packages/shared` is more foundational than `apps/runner`; importing policy from runner up to shared would invert the dependency graph. Leaving the generic helper un-migrated. Consumers opt into the policy at the call-site layer. |
| event-bus ×3, checkpoint-store ×2, WebSocket ×1, hook-registry ×1 (spec-enumerated) | ❌ not present | These modules do not currently invoke Zod `safeParse`; the spec enumeration was aspirational. No migration possible until a Zod boundary is introduced at those points (a separate refactor not in Plan49 scope). |

**Effective migrated sites**: 3 call-sites in apps/runner.

## 5. API

```typescript
import {
  applySchemaDriftPolicy,
  resolveSchemaDriftMode,
  setSchemaDriftAuditSink,
  SchemaDriftError,
  type SchemaDriftResult,
  type SchemaDriftMode,
  type SchemaDriftIssue,
  type SchemaDriftAuditEvent,
  type SchemaDriftAuditSink,
} from "./schema-drift-policy/index.js";

// Simple usage — let the policy decide based on global mode.
const result = applySchemaDriftPolicy(MySchema, rawInput, "my-context");
if (result.ok) {
  use(result.data);
} else {
  log("schema drift", result.error, result.issues);
}

// Boot-time: optionally wire the audited-mode sink to your log infrastructure.
setSchemaDriftAuditSink((event) => structuredLogger.emit(event));
```

## 6. Error type

`SchemaDriftError extends Error` is thrown only in `strict` mode. It carries `context` and `zodIssues` fields for operator diagnostics. Callers expecting the previous behaviour of `throw new ConfigError(...)` on any Zod failure continue to work because the migrated sites wrap `applySchemaDriftPolicy`'s non-ok return in `ConfigError` (and the thrown `SchemaDriftError` in strict mode propagates past the wrap, preserving the "invalid → throw" contract in either mode).

## 7. MR-6 posture (C49-M3f)

Module lives under `apps/runner/src/schema-drift-policy/`. Zero Core policy constants added; zero Core import edges. Verified at delivery:

```bash
grep -rn "schema-drift-policy" packages/core/
# → 0 matches
```

## 8. Rule #74 L1' 5 sub-checks

| # | Sub-check | Evidence |
|---|-----------|----------|
| i | Code file exists | `apps/runner/src/schema-drift-policy/index.ts` |
| ii | Test file exists | `apps/runner/__tests__/schema-drift-policy/index.test.ts` (11 tests PASS) |
| iii | Doc exists | this file + `docs/TW/schema-drift-policy.md` |
| iv | CHANGELOG | v0.49.0-alpha Plan49 Follow-on A entry |
| v | Cross-refs bidirectional | module JSDoc references this doc; this doc cross-refs migrated call-sites + `docs/EN/plugin-install-reliability.md` + `wiener-thresholds.md` |

## 9. References

- Plan49 spec §2.3 (C49-M3): `share/research_team_suggestion/cycle03-13/deliver/O2_plan49_engineering_spec.md`
- Plan49 dev spec §1.2 C49-M3: `share/research_team_suggestion/cycle03-13/todo/Plan49_dev_spec.md`
- D-12 UNANIMOUS (SHOULD per-site + process-global C49-M3g): `share/research_team_suggestion/cycle03-13/deliver/R3_decision_log.md §4.5`
- MR-6 conditional gates: `share/research_team_suggestion/cycle03-13/openstarry_doc/Technical_Specifications/Plan49_MR6_Conditional_Gates.md`
