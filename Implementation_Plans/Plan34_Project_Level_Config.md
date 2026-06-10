# Plan34: .openstarry/ Project-Level Configuration

**Version**: v0.34.0-alpha
**Cycle**: 20260316_cycle02-10
**SOP**: Standard + Release + Simulation + DOC
**Status**: COMPLETE

---

## Objective

Implement `.openstarry/` project-level directory support, upgrading Tenet #5 "Directory as Permission" from CONDITIONAL to COMPLIANT.

## Design Decisions

| Key Decision | Choice | Rationale |
|-------------|--------|-----------|
| KD-1 | Restrict-only model | Project config can narrow but never expand permissions |
| KD-2 | Plugin list: complete override | Simplicity; project declares exactly what plugins it needs |
| D2-R1 | Three-file separation | config.json / permissions.json / plugins.json |
| D2-R3 | isPathSafe() with symlink resolution | Cross-platform path validation, SEC-003 hardened |
| D2-R6 | Ten-step validation | Steps 1-4=SecurityError, Step 5=graceful skip, Steps 6-9=ConfigError, Step 10=empty-set fail-fast |
| D2-R8 | Load-once at startup | No hot-reload; TOCTOU risk accepted |

## Architecture

### Field Classification (Merge Rules)

| Classification | Merge Rule | Fields |
|---------------|------------|--------|
| Security ceiling | Intersection / min | allowedPaths, allowedTools, maxConcurrentTools, maxTokens |
| Security floor | Max (more restrictive) | confidenceFloor, safetyMinimumGear |
| Denied set | Union | deniedTools |
| Neutral | Project overrides system | identity.name, identity.description, cognition.temperature, cognition.maxRetries |
| Plugin list | Complete override | plugins |

### Microkernel Compliance

All project-level logic resides in `apps/runner/`, not `packages/core/`. Core remains pure.

## Implementation Waves

### Wave 1: SDK + Config Merger

**Deliverables**:
- `packages/sdk/src/types/project.ts` — 5 frozen interfaces (IProjectConfig, IProjectConfigFile, IProjectPermissions, IProjectPlugins, IProjectCognition)
- `apps/runner/src/utils/config-merger.ts` — restrict-only merge logic implementing KD-1
- Interface definitions locked; no further changes without Spec Addendum

### Wave 2: Zod Schemas + Validation

**Deliverables**:
- `packages/shared/src/utils/project-schema.ts` — 3 schemas with `.strict()` (projectConfigSchema, projectPermissionsSchema, projectPluginsSchema)
  - `.strict()` prevents field injection (SEC-008)
  - Validates field types and required/optional constraints
- `apps/runner/src/utils/permission-validator.ts` — Ten-step validation + loadProjectContext()
  - `isPathSafe()` implementation with realpathSync for symlink defense (SEC-003)
  - Steps 1-4 raise SecurityError; Steps 6-9 raise ConfigError
- `apps/runner/src/utils/project-detector.ts` — `findProjectRoot()` recursive upward directory traversal

### Wave 3: CLI Integration

**Deliverables**:
- `apps/runner/src/commands/start.ts` — project detection + merge + `--no-project-dir` flag
  - Auto-detects `.openstarry/` at startup
  - Applies restrict-only merge when project config found
  - `--no-project-dir` flag forces skip of project detection
- `apps/runner/src/commands/init.ts` — `--project` flag + template generation
  - `openstarry init --project` creates `.openstarry/` directory with three empty template files
- `apps/runner/src/commands/version.ts` — `--verbose` project config display
  - Shows both system and project configuration details when requested

## Test Coverage

- 28 new tests across 3 test files
- `project-detector.test.ts` — 4 tests
- `config-merger.test.ts` — 11 tests
- `permission-validator.test.ts` — 13 tests

## Security & Code Review Fixes

### Security Hardening
- **SEC-003** (BLOCKING): isPathSafe() enhanced with realpathSync on candidate path
  - Prevents symlink-based directory traversal attacks
  - Cross-platform path resolution before validation
- **SEC-008**: Inner plugin object `.strict()` added to prevent field injection
  - Zod schema validation rejects unexpected fields in plugins.json

### Code Review Corrections
- **Criticality Field**: `IPlugin.criticality` field type corrected (string literal union vs. enum)
- **plugins.json Template**: Removed incorrect template structure; now schema-validated only
- **JSDoc Fix**: Added missing JSDoc comments for `PluginCriticality` enum and `findProjectRoot()` function
- **Spec Addendum #01**: Approved relocation of `loadProjectContext()` function and SDK error imports to `apps/runner/`

## Metrics

- **LOC**: ~620 (spec estimated ~618)
- **Tests**: 1848 → 1876 (+28)
  - project-detector.test.ts: 4 tests
  - config-merger.test.ts: 11 tests
  - permission-validator.test.ts: 13 tests
- **Build Status**: PASS (34 workspace projects)
- **Rework Cycles**: 0
