# 74 — Plan47 K-3 Wire-In：Factory Bridge + Composite Snapshot + HMAC Sync

**Status**: ACTIVE (since v0.47.0-alpha, 2026-04-19)
**Plan**: Plan47
**Cycle**: 03-11 Dev
**Author**: Dev Team
**Reviewers**: Binding constraint per Master Ratification (2026-04-18)
**Related**: Doc 73 (Plan46 Tool Filtering + K-3 SDK hook), Plan46 `Implementation_Plans/Plan46_Implementation_Plan.md`

---

## 1. Context

Plan46 introduced the SDK-level K-3 framework hook (`PluginHooks.onCheckpoint` / `onRestore`) and the runner-level `CheckpointManager` + `capturePluginHooks` wrapper. The hook surface was delivered, but the three stateful components inside spc-monitor (SafetyGate, ShewhartChart, EscalationMonitor) and the StateTracker inside gear-arbiter-dynamic exposed their `onCheckpoint`/`onRestore` at the component level — not the factory level. `capturePluginHooks` in `apps/runner/src/utils/tool-filter-proxy.ts:33` guards on `hooks.onCheckpoint || hooks.onRestore`, so those plugins ended up absent from the framework `hookMap`, leaving runtime checkpoint/restore inexercisable end-to-end.

Plan47 closes this gap with a **binding constraint** clause set: 5 MUST (C47-K3-M1..M5) that Dev cannot drop, plus two hard limits (single-machine only, HMAC + nonce in the same Plan).

## 2. Decision

### 2.1 Factory Bridge (C47-K3-M1 + M2)

Each stateful plugin returns a single factory-level `PluginSnapshot` whose `pluginName` equals the manifest name. Inside that snapshot, sub-components serialize their own state into named sub-sections. The bridge owns the composite envelope; the runner-side `capturePluginHooks` + `CheckpointManager` stays untouched.

### 2.2 Composite Schema (C47-K3-M4)

`spc-monitor` owns three stateful components with independent sub-schemas:

```
PluginSnapshot (Plan47 composite, schemaVersion = 1, FROZEN per Rule #45 K-3 reauth)
├── pluginName   = '@openstarry-plugin/spc-monitor'
├── schemaVersion = 1
├── state
│   ├── safetyGate?        { schemaVersion:1, lastTriggerMs, shadowDecisionsSinceTrigger }
│   ├── shewhartChart?     { schemaVersion:1, windows: <JSON string> }     // SEC-003 preserved
│   └── escalationMonitor? { schemaVersion:1, states: [[cat, { anomalyTimestamps, level, monitoringOnly }]] }
└── timestamp
```

Each sub-section carries its own `schemaVersion` so one sub-component can evolve independently without bumping the composite envelope version. Missing sub-sections are forward-compatible no-ops.

`gear-arbiter-dynamic` follows the same pattern with a single sub-section (`state.stateTracker`) — the composite envelope is retained so the schema is symmetric across plugins and leaves room for future additions (e.g., M4a aggregator snapshot).

### 2.3 Dual-Path Resolution (C47-K3-M5)

Two restore paths exist:
1. **K-3 onRestore** (Plan46) — authoritative, called by the framework after factory returns.
2. **`config.safetyGate.snapshot`** (Plan45 cold-start) — deprecated.

Resolution rule: **K-3 onRestore wins**. When both paths supply state, K-3 overwrites factory-initialised state. Plan47 adds a `console.warn` when the cold-start path is used; Plan48 removes the cold-start path (per KERNEL §Q-K3-OQ-4).

### 2.4 HMAC + Nonce (Master hard limit 2)

Checkpoint blobs are signed with HMAC-SHA256 over `nonce || ":" || signedAt || ":" || payloadJSON`. The `NonceRegistry` is a process-local `Set<string>` that rejects duplicate nonces within one runner lifetime (replay defence). HMAC keys enter via CLI flag (`--checkpoint-hmac-key`) or env (`OPENSTARRY_CHECKPOINT_HMAC_KEY`); neither the SDK nor Core references them. Minimum key length: 32 bytes.

R3 D2.5 (BABBAGE 翻轉) established that staging HMAC after wire-in is MORE dangerous than delaying wire-in, because a wired-in-but-unsigned path can be exploited before the signature lands. Plan47 therefore packages them together.

### 2.5 Consumer Wire-In (C47-K3-M3)

Two complementary consumer paths:
- **Daemon lifecycle (Option B)** — `apps/runner/src/commands/start.ts` on startup detects `--checkpoint-path`, reads + verifies + restores; on shutdown captures + signs + writes.
- **Offline CLI (Option C)** — `runner checkpoint verify <path>` does full HMAC verify without a live daemon; `runner checkpoint inspect <path>` prints envelope metadata for operational debugging.

Dead code `void _checkpointMgr;` in `start.ts` (Plan46 placeholder) is removed — the manager is now held by a real reference.

## 3. Consequences

### 3.1 Positive
- `hookMap.size > 0` for spc-monitor + gear-arbiter-dynamic in runtime (was 0 in Plan46 per KERNEL §4.2).
- End-to-end checkpoint/restore is testable with the shipped plugins (not just the synthetic Counter).
- Checkpoint files are tamper-evident and replay-resistant within a session.
- MR-6 preserved — zero new Core modifications (purity check: 0 new violations).

### 3.2 Trade-offs
- File-system backend couples `OPENSTARRY_CHECKPOINT_PATH` to a single directory. Plugin-level backends (S3, DB, etc.) are deferred to Plan48+.
- `NonceRegistry` is process-local — cross-process replay defence requires a persistent registry (deferred).
- `config.safetyGate.snapshot` path kept through v0.47 for one release cycle; any plugin consumers must migrate before Plan48.

### 3.3 Known carry-forward
- SHOULD/MAY items from todo_README §4 (F-L3-SEC-2/3 audit-gap补回, plugin-install flaky, pre-existing purity violations) — status tracked in `delivery_report.md §Carry-Forward`.
- Per-Plan47 `config.safetyGate.snapshot` deprecation → Plan48 removal target.

## 4. Alternatives Considered

1. **Wrap CheckpointManager inside Core** — rejected (violates MR-6; Core must not know about signing policy or file paths).
2. **Per-component PluginSnapshot (three separate entries in the snapshot Map for spc-monitor)** — rejected (violates `capturePluginHooks` 1:1 keying; would require framework changes).
3. **Delay HMAC to Plan48** — rejected (R3 D2.5 BABBAGE 翻轉：wire-in without signature is more dangerous than no wire-in).
4. **Daemon IPC RPC for `runner checkpoint`** — deferred; offline file ops are sufficient for Plan47 scope.

## 5. References

- SDK contract: `packages/sdk/src/types/plugin.ts:270-373`
- Runner capture: `apps/runner/src/utils/tool-filter-proxy.ts:25-39`
- Runner manager: `apps/runner/src/utils/checkpoint-manager.ts`
- Plan46 doc: Doc 73
- Master Ratification: `discussions/Master_Ratification_Plan47_K3_5MUST_APPROVED.md`
- Rule #74 (L1' code+doc sync): `discussions/Master_Ratification_Rule_74.md`
- ENG-FAB v1.6 (42 items): `discussions/Master_Ratification_ENG_FAB_v1_6.md`
