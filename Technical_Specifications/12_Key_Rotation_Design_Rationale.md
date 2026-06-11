# 12. GUARDIAN Key Rotation Design Rationale

**Plan**: Plan42 W3-1
**Status**: Design complete — Implementation deferred to Plan43 (~40-60 LOC)
**Date**: 2026-04-07

---

## 1. Purpose

This document records the design rationale and operating assumptions for GUARDIAN's key rotation mechanism. It provides the formal basis for the Plan43 implementation. All assumptions are explicitly stated so future implementors can identify when a change in deployment model requires revisiting this design.

---

## 2. Foundational Assumptions (A1–A4)

### A1: Single-Host Deployment

**Assumption**: All agents sharing a GUARDIAN-managed session run on the same host process (or at minimum within a single trust boundary with shared IPC access, e.g., a single machine with Unix domain sockets).

**Rationale**: GUARDIAN does not implement network-level key distribution (e.g., TLS mutual auth, KMS integration). Key material is passed via in-process function calls or local IPC. If this assumption breaks (multi-host deployment), a network-capable key distribution layer must be added before Plan43's rotation logic applies.

**Implication**: Key rotation does not require cryptographic key encapsulation or transport encryption. The channel is trusted.

---

### A2: In-Memory Key Store

**Assumption**: HMAC keys are held exclusively in process memory. No key is written to disk, a database, or any persistent store.

**Rationale**: Persistence would introduce a separate threat surface (file permissions, database ACLs, encryption-at-rest). For the current threat model (Plan40/Plan42), the risk of in-memory compromise is accepted as lower than the complexity of secure persistent storage.

**Implication**: Keys are lost on process restart. Any session relying on GUARDIAN-issued HMAC keys is invalidated on restart. Agents must re-register and receive fresh keys. This is by design, not a deficiency.

---

### A3: Short Session Lifetime

**Assumption**: Sessions are bounded to hours, not days. A session lasting more than 24 hours is outside the design envelope.

**Rationale**: The rotation window is sized assuming bounded session duration. If sessions could last days, accumulated replay-window risk from a compromised key would grow unacceptably. Long-running sessions would require additional controls (e.g., rolling session IDs, external key refresh) not present in Plan43.

**Implication**: Key rotation intervals of 30–60 minutes are appropriate. A maximum of ~48 rotation events per session is expected, keeping the key version counter (uint16) well within bounds for the foreseeable lifetime.

---

### A4: Trusted Daemon

**Assumption**: The Daemon process is the root of trust. It generates all HMAC keys, performs rotation, and distributes keys to agents. No agent generates its own HMAC key material.

**Rationale**: Centralised key generation eliminates the coordination problem of distributed key agreement (Diffie-Hellman, etc.) within a single-host environment. The Daemon's privilege is appropriate because it already gates plugin load/unload and holds the plugin manifest authority.

**Implication**: Compromise of the Daemon process is equivalent to compromise of the entire session. This is accepted under the single-host trust model (A1). Daemon isolation (process sandboxing, minimal surface area) is the primary defence.

---

## 3. Key Version and Nonce

Each HMAC key carries a monotonically increasing version number (`keyVersion: number`, uint16 range, wraps at 65535 with an alert).

- The version is incremented by exactly 1 on each successful rotation.
- HMAC-signed payloads include the `keyVersion` field in the signed body (not just the header), binding the signature to a specific key generation.
- Verifiers MUST reject payloads whose `keyVersion` is more than 1 behind the current version (grace window = 1 prior version only).

---

## 4. Edge Cases

### 4.1 Late-Joiner During Rotation

**Scenario**: A new agent connects to GUARDIAN while a key rotation is in progress.

**Handling**:
1. GUARDIAN completes distributing the new key to already-registered agents first.
2. The late-joiner receives the new key (post-rotation) as its initial key.
3. The late-joiner starts at the current `keyVersion` and is never exposed to the old key.
4. In-flight seeds signed with the old key that arrive at the late-joiner are rejected (it has no old key). The originating agent must retry after the late-joiner's registration is acknowledged.

**Rationale**: Simplicity over completeness. A late-joiner that cannot verify old-key messages is safe — it simply cannot process them, and the sending agent will detect the failure via the acknowledgement protocol.

---

### 4.2 Partial Rotation Failure

**Scenario**: GUARDIAN begins distributing the new key but fails to deliver to one or more agents (agent crash, IPC timeout).

**Handling**:
1. GUARDIAN tracks delivery acknowledgement per agent.
2. If any agent fails to acknowledge within the rotation timeout, GUARDIAN aborts the rotation and retains the old key as current.
3. The partially notified agents (those that received the new key before the abort) are sent a rollback message instructing them to revert to the old key.
4. The `keyVersion` is NOT incremented; the failed rotation attempt is treated as if it never occurred.

**Rationale**: An inconsistent key state (some agents on new key, others on old key) is worse than staying on the old key. Atomic commit-or-abort semantics are mandatory for rotation correctness.

---

### 4.3 Backward Compatibility — Versioned Keys and Grace Period

**Scenario**: Seeds signed with old key `v_n` are in-flight when rotation advances to `v_{n+1}`.

**Handling**:
- GUARDIAN maintains the immediately prior key (`v_n`) in a "grace slot" for a configurable window (default: 30 seconds after rotation completes, or until all in-flight seeds are acknowledged, whichever comes first).
- Agents receiving a seed with `keyVersion == currentVersion - 1` during the grace period accept and verify it using the grace-slot key.
- Agents receiving a seed with `keyVersion < currentVersion - 1` reject it immediately (replay or stale message, not covered by grace).
- After the grace window expires, the prior key is zeroed from memory.

**Grace period configuration** (Plan43 implementation detail):
```
GUARDIAN_KEY_ROTATION_GRACE_MS = 30_000   // default
```

---

### 4.4 Key Version Counter Wrap

**Scenario**: `keyVersion` reaches the uint16 maximum (65535) and wraps to 0.

**Handling**:
- GUARDIAN emits a warning log at version 65000 (535 rotations remaining).
- At version 65535, the next rotation wraps to version 1 (not 0; 0 is reserved as "unversioned/legacy").
- Wrap is treated as a normal rotation event; the grace period and partial-failure protections apply identically.

**Rationale**: At a rotation interval of 30 minutes, reaching 65535 would take ~3.4 years of continuous session uptime, far beyond assumption A3. The wrap handling is a safety net, not an expected code path.

---

## 5. Implementation Scope (Plan43 Preview)

The following are NOT implemented in Plan42 and are deferred to Plan43:

| Item | Estimated LOC | Notes |
|---|---|---|
| `KeyRotationManager` class | ~20 | Holds current + grace-slot keys, handles version increment |
| Rotation protocol messages | ~10 | `KeyRotateNotify`, `KeyRotateAck`, `KeyRotateRollback` |
| Grace period timer | ~10 | setTimeout-based, zeroes grace-slot key on expiry |
| Agent delivery tracking | ~10 | Map<agentId, acked: boolean> per rotation attempt |
| Integration with GUARDIAN | ~10 | Wire rotation trigger into existing GUARDIAN lifecycle |

Total Plan43 estimate: 40–60 LOC (excluding tests).

---

## 6. Out of Scope

The following are explicitly excluded from Plan42 and Plan43:

- Multi-host key distribution (network transport, TLS, KMS)
- Persistent key storage (disk, DB, HSM)
- Key derivation functions (KDF) or key wrapping
- Certificate-based identity for agents
- Forward secrecy (ephemeral keys per message)

These would be required for a production multi-host deployment and are tracked as future work in the Risk Register.

---

## 7. References

- `09_HMAC_Seed_Authentication_Architecture.md` — HMAC seed auth design (Plan40)
- `10_Microkernel_Security_Analysis.md` — Threat model context
- `share/openstarry_doc/Agent_Corps/Risk_Register.md` — Multi-host risk item
- Plan42 Architecture_Spec (Cycle 20260407_cycle03-5)
