# Structured Log — Plan48 F-L3-SEC-2 (Doc 78 candidate)

**Binding sub-item**: C48-M1g (L1' doc sync).
**Layer**: Runner (NOT Core; MR-6 preserved).

Self-built, zero-external-dep structured-log facility used across the
OpenStarry runner for policy-path observability. See
`apps/runner/src/structured-log/README.md` for the implementation-facing
surface.

## JSON-line schema

Every emitted line is a single JSON object:

| Field      | Type   | Notes                                                    |
|------------|--------|----------------------------------------------------------|
| `timestamp`| string | ISO-8601, generated via `isoTimestamp()` on emit.        |
| `level`    | string | One of `DEBUG` / `INFO` / `WARN` / `ERROR` / `FATAL`.    |
| `event`    | string | Short dotted identifier (e.g., `runner.boot`).            |
| `payload`  | any \| null | Safe-stringified (circular / BigInt / long-string guarded). |

## Configuration

| Env var                        | Default    | Effect                              |
|--------------------------------|------------|-------------------------------------|
| `LOG_LEVEL`                    | `INFO`     | Lines with level below are dropped. |
| `OPENSTARRY_LOG_PATH`          | *(stderr)* | Absolute JSONL path.                |
| `OPENSTARRY_LOG_BUFFER_MAX`    | `1024`     | Ring buffer size.                   |

## Back-pressure contract (C48-M1d)

On overflow the oldest entry is dropped (FIFO). A one-shot
`W_AUDIT_OVERFLOW` WARN record is emitted directly to the sink so the
back-pressure signal itself cannot be dropped. `resetOverflowReported()`
re-arms the signal for subsequent overflow windows.

## Shutdown flush (C48-M1e)

`registerStructuredLogShutdown()` attaches a hook at
`SHUTDOWN_ORDER.FLUSH_STRUCTURED_LOG` (200). Each hook in the shutdown
registry is awaited serially, so structured-log is fully drained before
audit-sink flush (300) and HMAC clear (400).

## Safe stringify edge cases

`safe-stringify.ts` covers the HERACLITUS MRB-12 §12.2 JSON edge-case
matrix:

| Case              | Behaviour                                        |
|-------------------|--------------------------------------------------|
| Circular reference| Replaced with `"[Circular ~]"`.                  |
| BigInt            | Coerced to `"<digits>n"` string.                 |
| Very long string  | Truncated at 16 KiB with `"...[truncated]"` tail.|
| Error instance    | `{ name, message, stack }`.                      |
| Symbol interaction| Caught → `{ "__serialization_error": ... }`.     |

## Relation to Plan47 snapshot-hmac logging

Plan47 `snapshot-hmac.ts` did not depend on a structured-log facility;
Plan48 retrofits the writer for use in runner bootstrap, plugin-install
flows, and the SIGTERM cascade. Existing callers (e.g., checkpoint
commands) continue to use `console.*` for human-facing CLI output and
are not forced onto the structured log.
