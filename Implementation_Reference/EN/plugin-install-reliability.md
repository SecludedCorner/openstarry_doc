# Plugin Install Reliability

**Status**: Plan49 C49-M1 — root cause identified and fixed.
**Effective from**: v0.49.0-alpha (2026-04-24).

## 1. Symptom

Prior to Plan49, `pnpm test apps/runner/__tests__/commands/plugin-install.test.ts`
and `pnpm test apps/runner/__tests__/utils/plugin-installer.test.ts` intermittently
failed on Windows with filesystem errors (EBUSY / EPERM / EEXIST) when run
concurrently, which vitest does by default (one thread per test file).

## 2. Root cause

Both `plugin-installer.ts` and `plugin-lock.ts` exported **module-level constants**
pointing to a single user-global filesystem location:

- `INSTALLED_DIR` → `~/.openstarry/plugins/installed/`
- `LOCK_FILE_PATH` → `~/.openstarry/plugins/lock.json`

When vitest parallelised across test files, two threads simultaneously called
`installPlugin()` on the same plugin name and raced on the same install target
directory:

- Thread A: `rm(targetDir, …)` then `cp(packageDir, targetDir, …)`
- Thread B: `rm(targetDir, …)` then `cp(packageDir, targetDir, …)`

On Windows, the `rm` / `cp` combination is not race-safe — an open handle from
one thread can block the other's delete or copy, producing intermittent EBUSY /
EPERM errors. The `syncPlugin()` path (`plugin-scanner.ts`) hits the same
target-directory race.

Secondary contributor: the npm-pack fallback tempdir name was
`openstarry-install-${Date.now()}` (millisecond resolution); two parallel
threads entering the npm path in the same millisecond got identical tempdir
paths.

## 3. Fix (Plan49 C49-M1b)

### 3.1 Test-injectable install dir

`InstallOptions` gained an `installedDir?: string` field. `installPlugin()`,
`uninstallPlugin()`, and `installAll()` now resolve the effective install
directory in this order:

1. `options.installedDir` if provided.
2. `process.env.OPENSTARRY_INSTALL_DIR` if set.
3. `DEFAULT_INSTALLED_DIR` (the module-level constant; unchanged default
   behaviour for production runs).

The env var exists so CLI-layer tests can isolate state without threading an
`installedDir` option through every CLI call.

`lockPath` already supported per-call override; the same
env-var-then-module-default fallback pattern was added
(`OPENSTARRY_LOCK_PATH`).

### 3.2 Collision-proof tmpdir naming

The npm-pack fallback now names its tempdir
`openstarry-install-${process.pid}-${Date.now()}-${random6}`. PID alone
eliminates cross-process collisions; the random suffix covers intra-process
same-millisecond parallel calls.

### 3.3 Test changes

- `apps/runner/__tests__/utils/plugin-installer.test.ts` creates a
  PID-scoped `tempDir` in `beforeEach` and passes
  `{ lockPath, installedDir }` on every call.
- `apps/runner/__tests__/commands/plugin-install.test.ts` sets
  `OPENSTARRY_INSTALL_DIR` and `OPENSTARRY_LOCK_PATH` in `beforeAll` and
  restores them in `afterAll`, isolating the CLI layer's transitive install
  calls.

## 4. Hypothesis elimination (Plan49 §2R1 §2.2.1)

The Plan49 research spec enumerated eight hypotheses (H1–H8). Observed
evidence supports H1 (parallel-test shared-FS race) as the root cause and
eliminates the others:

| ID | Hypothesis | Verdict |
|----|-----------|---------|
| H1 | Parallel test files share `INSTALLED_DIR` / `LOCK_FILE_PATH` | **CONFIRMED — root cause** |
| H2 | `Date.now()` collision in tmpdir naming (secondary) | Contributing; addressed alongside H1 |
| H3 | `require.resolve` child-process timeout instability | Not observed; timeout raised only on slow machines |
| H4 | `npm pack` retry flake | Not observed in reproduction (workspace path used) |
| H5 | `plugin-lock.ts` non-atomic write | Not observed; write is whole-file with atomic rename semantics on Unix, serialised per-test on Windows |
| H6 | Worker-thread race inside a single test file | Not applicable (tests are sync-awaited) |
| H7 | Filesystem journaling delay | Not reproducible under isolated-dir fix |
| H8 | npm hoist vs nested-import | Not observed; Plan49 did not restructure imports |

## 5. Operational notes for CI and plugin authors

- **Production callers**: no API break. `installPlugin`, `uninstallPlugin`,
  and `installAll` continue to default to `~/.openstarry/plugins/installed/`
  when no option or env var is set.
- **Test authors**: always pass `installedDir` + `lockPath` (or set
  `OPENSTARRY_INSTALL_DIR` + `OPENSTARRY_LOCK_PATH`) for any test that
  calls `installPlugin` / `installAll`. Do not share these paths across
  parallel test files.
- **CI flake gating (C49-M1d SHOULD)** — delivered by Plan49 Follow-on B:
  run `pnpm test:flake-gate` (default 50 iterations, configurable:
  `bash scripts/flake-gate.sh <N>`). The script runs both plugin-install
  test files per iteration and bails on the first failure with tail-30
  diagnostics. Zero-tolerance semantics. Initial post-fix local Windows
  smoke: 5/5 iter × 17 tests PASS (Task #65) + 2/2 iter flake-gate smoke
  (Task #68). Full 50-iter gate should be wired in CI when a pipeline is
  added (none exists in the repo today).

## 6. Rule #74 L1' sub-checks

| # | Sub-check | Evidence |
|---|-----------|----------|
| i | Code file exists | `apps/runner/src/utils/plugin-installer.ts` (`installedDir` option + env-var fallback + collision-proof tmpdir) |
| ii | Test file exists | `apps/runner/__tests__/utils/plugin-installer.test.ts` (`C49-M1c concurrent install isolation (regression)` block + updated existing cases) + `apps/runner/__tests__/commands/plugin-install.test.ts` (env-var isolation) |
| iii | Doc exists | This file + `docs/TW/plugin-install-reliability.md` |
| iv | CHANGELOG references it | `CHANGELOG.md` v0.49.0-alpha entry under Plan49 C49-M1 |
| v | Cross-refs bidirectional | `plugin-installer.ts` option JSDoc references Plan49 C49-M1; this doc cross-refs `plugin-installer.ts`, `plugin-lock.ts`, and the regression test |

## 7. References

- Plan49 engineering spec (full): `share/research_team_suggestion/cycle03-13/deliver/O2_plan49_engineering_spec.md`
- Plan49 dev spec (concise): `share/research_team_suggestion/cycle03-13/todo/Plan49_dev_spec.md`
- Plan49 C49-M1 sub-items: M1a root cause, M1b fix, M1c regression, M1d CI flake gating, M1e this doc
