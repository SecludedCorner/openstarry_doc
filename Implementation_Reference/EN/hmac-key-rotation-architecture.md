# HMAC Key Rotation — Architecture (design-spec only, per R3 D-17a)

**Plan48 scope**: this document is the **design spec only**. Runtime key
rotation is **NOT implemented** in Plan48 and is deferred to a future
cycle. Per R3 D-17(a) UNANIMOUS (23/0).

**Scope qualifier (D-12b)**: within-process scope applies to rotation
too — rotated keys inherit the in-process-memory threat-model assumption
stated in `./hmac-compliance.md`.

---

## 1. Key lifecycle states

### 1.1 Current state (Plan48)

```
    [init]
      ↓
    [bound-to-closure]        ← captureHmacKey()
      ↓
    [consumed-for-shutdown-sign]
      ↓
    [cleared]                 ← binding.clear()
```

Single transition; no rotation.

### 1.2 Future state (rotation design)

```
    [init]
      ↓
    [bound-to-closure-N]
      ↓ ── [sign N events] ──
      ↓
    [rotate-trigger-fires]
      ↓
    [bound-to-closure-N+1]    ← new key captured via the same
      ↓                          captureHmacKey() surface
    ...
      ↓
    [cleared-all]
```

Rotation adds a `[rotate-trigger-fires]` transition that swaps the
closure reference without releasing control to any other subsystem.

---

## 2. Rotation triggers (considered)

| Trigger            | Description                                       | Trade-off                      |
|--------------------|---------------------------------------------------|--------------------------------|
| Wall-clock TTL     | Rotate every `N` hours.                           | Simple; may over-rotate.       |
| Usage-count        | Rotate after `K` signings.                        | Bounded-leakage window.        |
| Externally signalled | Rotate on explicit admin command / signal.      | Human-in-the-loop; safest.     |
| Process-restart    | Current state — rotation = process restart.       | Minimal; Plan48 default.       |

A future implementation MAY combine triggers (e.g., "wall-clock OR
usage-count, whichever first").

---

## 3. Storage requirements

- **No plain-text persistence** of any rotated key outside the configured
  secure-store root (`OPENSTARRY_SECURE_STORE`).
- Rotation MUST use the same secure-store integration contract that
  `captureHmacKey()` would consume — a rotation design does not introduce
  a second key-source code path.
- Envelope storage format is deliberately unspecified here; the
  implementation Plan will select (e.g., sealed box, KMS reference).

---

## 4. Audit requirements

Every rotate event MUST emit a journal entry via the audit-sink
(`capability_denied` / `ws_connection_denied` are not the right channel;
rotation adds a new `hmac_key_rotated` event type when implemented).
The audit-trail file must record:

- `timestamp`
- `trigger` (`wallclock_ttl` | `usage_count` | `external` | `restart`)
- `reason` (short free-text)
- `previous_key_id` (short hash, not the key itself)
- `new_key_id` (short hash)

---

## 5. Threat-model qualifier (D-12b, reiterated)

Within-process scope applies to the rotated key just as it does to the
initial key. Out-of-process adversaries are explicitly out of scope.

---

## 6. Forward-compatibility constraints

- **MR-6 preservation**: rotation design must not require Core policy
  changes. All rotation logic lives at runner layer, in the same module
  family as `hmac-cleanup/`.
- **Interface stability**: `captureHmacKey()` return type
  (`HmacCleanupBinding`) must remain the primary handle; rotation may
  add an optional `rotate(): Promise<HmacCleanupBinding>` method on the
  binding, but `sign` / `clear` must not break.

---

## 7. Out-of-scope for Plan48 (explicit)

- No runtime rotation code.
- No scheduler / timer integration for rotation.
- No key-store connector or adapter.
- No tests for rotation (tests for existing cleanup cover C48-M3 only).

Plan48 aggregate PASS is independent of future rotation work.

---

## 8. Cross-references

- `./hmac-compliance.md` — C48-M3c standards mapping (ASVS + NIST).
- `apps/runner/src/hmac-cleanup/README.md` — C48-M3 implementation surface.
- `share/research_team_suggestion/cycle03-12/deliver/O5_Plan48_scope_and_engineering_spec.md` §4 — binding design-spec scope.
