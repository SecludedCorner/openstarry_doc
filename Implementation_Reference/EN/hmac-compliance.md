# HMAC Compliance — Plan48 E-5 MUST (within-process scope)

**Binding**: Plan48 sub-item `C48-M3c`.
**Scope qualifier (per R3 D-12b)**: HMAC signing in OpenStarry operates
with **within-process scope**; the adversary threat model assumes
**absence of in-process memory read**. Out-of-process / cross-process
HMAC authentication is **NOT in scope** for Plan48 or its predecessors.

---

## 1. Standards mapping

### 1.1 OWASP ASVS V2.10.1 — Secret management

> "Verify that secrets are not kept in resident memory longer than
> necessary."

**OpenStarry compliance**:
- `hmac-cleanup/index.ts` `captureHmacKey()` reads the env var once and
  zeroes the env-var value immediately (`process.env[name] = ''` then
  `delete`).
- The key lives in a single closure reference (`closureKey`) bound only
  to the shutdown-signing callback. No module-level variable retains it.
- On shutdown, `binding.clear()` overwrites the closure reference and
  nulls it, completing the ASVS V2.10.1 obligation.

### 1.2 NIST SP 800-57 Part 1 §8.2.2 — Key destruction

> "When a cryptographic key is no longer needed, it shall be destroyed."

**OpenStarry compliance**:
- `registerHmacCleanupShutdown()` attaches the cleanup hook at
  `SHUTDOWN_ORDER.HMAC_CLEAR_AND_SIGN` (400) — after audit-sink flush,
  before process exit. The hook runs `binding.clear()` unconditionally
  in a `finally` block so a failing `onBeforeClear` callback cannot
  leak the key past the shutdown cascade.
- `binding.clear()` overwrites the string before dropping the reference;
  JavaScript has no guaranteed zero-fill for primitive strings, so the
  overwrite + drop combination is the maximal-available-defence.
- C48-M3d W2-R13 runtime evidence verifies `binding.cleared === true`
  after the shutdown cascade completes.

---

## 2. Threat model and non-goals

### 2.1 In-scope (Plan48)

- Adversary cannot read OpenStarry process memory directly.
- Adversary may gain read-only access to the filesystem after process
  exit (→ no plaintext key on disk outside the configured secure store).
- Adversary may spawn child processes or sibling processes that read
  environment variables (→ env var is zeroed immediately after capture).

### 2.2 Out-of-scope (NOT in Plan48)

- **In-process memory read adversary.** The closure key is subject to
  the normal V8 / Node runtime behaviour; an attacker with heap-dump or
  debugger-attach capability is outside the scope declared by D-12b.
- **Cross-process HMAC authentication.** OpenStarry's HMAC usage is
  confined to a single runner process signing / verifying its own
  checkpoint blobs. Multi-process HMAC handshakes are not part of the
  Plan47 / Plan48 delta and are explicitly excluded from this document.
- **HMAC key rotation runtime.** See
  `./hmac-key-rotation-architecture.md` — Plan48 ships the design only;
  runtime rotation is deferred per R3 D-17(a).

---

## 3. Per-sub-item mapping

| Sub-item | Standard clause | Implementation artefact |
|----------|-----------------|-------------------------|
| C48-M3a  | ASVS V2.10.1 + NIST §8.2.2 | `hmac-cleanup/index.ts` `captureHmacKey` + `clear` |
| C48-M3b  | NIST §8.2 (ephemeral sources) | `hmac-cleanup/policy.ts` `resolveSecureStoreRoot` + `isPathInsideSecureStore` |
| C48-M3c  | — (this document)            | `docs/EN/hmac-compliance.md` + TW translation |
| C48-M3d  | NIST §8.2.2 (destruction evidence) | W2-R13 shutdown scenario + `binding.cleared` assertion |

---

## 4. Verification notes

- **L1 grep**: `captureHmacKey`, `registerHmacCleanupShutdown`,
  `isPathInsideSecureStore` — all present under
  `apps/runner/src/hmac-cleanup/`.
- **L1' doc**: this file + `hmac-cleanup/README.md` + CHANGELOG entry +
  bidirectional cross-reference to `hmac-key-rotation-architecture.md`.
- **L2 unit tests**: `apps/runner/__tests__/hmac-cleanup/*.test.ts`.
- **L3 integration**: `plan48-integration.test.ts` verifies end-to-end
  capture → zero env → sign → shutdown → clear sequence.
- **W2-R13 runtime**: shutdown scenario asserts `binding.cleared` AND
  heap-state post-shutdown contains no plaintext key (best-effort:
  grep on dumped string pool).
