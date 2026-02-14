# OpenStarry Agent Corps — Risk Register

Established: 2026-02-09

**Scoring Criteria**: Probability (L/M/H) × Impact (L/M/H) = Risk Level (Low/Medium/High/Critical)

---

## Technical Risks

### R-T01: Agent Context Window Overflow
- **Probability**: H (Nearly inevitable during implementation of large features)
- **Impact**: M (Agent output may be incomplete or miss details)
- **Risk Level**: High
- **Trigger Scenario**: dev-core needs to read massive existing code + Spec + write numerous changes
- **Mitigation Strategies**:
  - Coordinator splits large features into multiple small tasks for incremental dispatch
  - Agent prompt explicitly specifies only necessary files to be read
  - Complex changes are divided into Step 2a / 2b / 2c
- **Contingency Plan**: If output is incomplete, the Coordinator reviews the completed portion and dispatches a new agent session to continue

### R-T02: pnpm install or build failure in agent_test
- **Probability**: M
- **Impact**: M (Stalls Phase 2.5/3)
- **Risk Level**: Medium
- **Trigger Scenario**: Dependency conflicts, workspace protocol issues, Windows path issues
- **Status**: **Materialized** (Cycle 18 Phase 2.5) — stale node_modules (yaml/vitest module resolution failures)
- **Mitigation Strategies**:
  - Sync script copies `pnpm-lock.yaml` to ensure consistent dependencies
  - Phase 2 exit criteria require build pass; theoretically, it should pass after sync
  - **NEW (Cycle 18 lesson)**: Establish node_modules cleanup infrastructure (Plan16+ task)
- **Contingency Plan**:
  - If sync build fails while agent_dev build passes → Check if the sync script missed any files
  - Or perform manual node_modules cleanup and pnpm install
  - Plan16: Implement CI hooks for automatic cleanup of old node_modules (cross-iteration environment hygiene)

### R-T03: Windows Environment Compatibility
- **Probability**: M
- **Impact**: M (Scripts or tools may not operate as expected)
- **Risk Level**: Medium
- **Trigger Scenario**: Bash script behavior differences in Windows Git Bash, path separators, lack of symlink support
- **Mitigation Strategies**:
  - Scripts use `$(cd ... && pwd)` to obtain absolute paths
  - Avoid symlinks; use `cp` for copying
  - Scripts use `2>/dev/null || true` to handle non-critical errors
- **Contingency Plan**: Manually execute failed steps and record issues for future script correction

### R-T04: Plugin Dependency on Out-of-sync SDK Version
- **Probability**: M
- **Impact**: H (Plugin build failure, runtime type mismatch)
- **Risk Level**: High
- **Trigger Scenario**: dev-core modifies SDK interface while dev-plugin uses an old version
- **Mitigation Strategies**:
  - Interface Freeze mechanism ensures Spec is frozen
  - Phase 2 Step 2a completes SDK first → build → then dispatch dev-plugin
  - Workspace protocol (`workspace:*`) automatically links local versions
- **Contingency Plan**: Use the Spec Addendum process to handle necessary interface changes

---

## Process Risks

### R-P01: Coordinator Main Session Crash
- **Probability**: M (Long-running operations, token exhaustion)
- **Impact**: H (Iteration interruption, potential state inconsistency)
- **Risk Level**: High
- **Mitigation Strategies**:
  - All states persisted via files (reports act as implicit state markers)
  - SOP 8.3 defines state determination methods
  - Outputs of each Phase are independent files, not dependent on session memory
- **Contingency Plan**: New session reads SOP + checks cycle directory → determines Phase → continues

### R-P02: Rework Infinite Loop
- **Probability**: L
- **Impact**: H (Iteration never finishes)
- **Risk Level**: Medium
- **Trigger Scenario**: Design flaws lead to repeated FAILs; each rework introduces new issues
- **Mitigation Strategies**:
  - Rework limit of 2; exceeding this triggers mandatory escalation to User
  - Rebuild baseline before each rework
- **Contingency Plan**: User decides whether to abandon the Plan or perform a major redesign

### R-P03: Inconsistent Agent Output Quality
- **Probability**: M
- **Impact**: M (Garbage-in garbage-out, discovered only in Phase 3)
- **Trigger Scenario**: Vague agent prompts, agent misunderstanding the Spec
- **Mitigation Strategies**:
  - Communication Protocol defines necessary prompt fields
  - Phase 3 double verification (QA + architect)
  - Dev logs record all changes for review
- **Contingency Plan**: Handled via the Rework process

### R-P04: Report or Document Content Conflicts
- **Probability**: L
- **Impact**: L (Confusion but not fatal)
- **Trigger Scenario**: Multiple agents writing to adjacent files simultaneously; doc-keeper and Coordinator updating Iteration_Log concurrently
- **Mitigation Strategies**:
  - Each agent has a distinct output directory; no overlap
  - doc-keeper is the sole writer for Iteration_Log
- **Contingency Plan**: Coordinator manually merges conflicts if detected

### R-P05: Integration Point Missing
- **Probability**: M (Easy to miss during new feature integration)
- **Impact**: M (Feature fails to work, discovered only in Phase 3)
- **Trigger Scenario**: Phase 2 implementation writes standalone modules but fails to integrate them into the main flow; or uses TODO comments instead of actual code
- **Status**: Discovered in Cycle 7 FAIL-1 (import analysis not integrated into sandbox-manager)
- **Mitigation Strategies**:
  - Phase 1 Spec explicitly lists all integration points
  - Phase 2 Code Review checklist: Check for TODO/FIXME comments at integration points
  - Phase 2 establishes "integration tests" specifically verifying each integration point is called
- **Contingency Plan**: All future new features must include explicit "integration verification tests"; cannot be skipped

### R-P06: Spec Omission
- **Probability**: M (Complex Specs are prone to omissions)
- **Impact**: M (Implementation complete but components defined in Spec were not created, Phase 3 FAIL)
- **Trigger Scenario**: Architecture Spec defines a file/component list, but Phase 2 does not create all items; e.g., Cycle 7 plugin-signer package
- **Status**: Discovered in Cycle 7 FAIL-2 (plugin-signer package in spec but not implemented)
- **Mitigation Strategies**:
  - Before Phase 2 begins, Coordinator builds a "Spec file list checklist"
  - At Phase 2 completion, the checklist is mandatorily verified item by item
  - Dev logs record "list of created files" for verification
- **Contingency Plan**: Future new Plans must prepare checklists before starting

### R-P07: Unvalidated Assumptions
- **Probability**: M (Complex systems often contain implicit assumptions)
- **Impact**: M (Code path crashes under specific conditions, e.g., package-name plugins)
- **Trigger Scenario**: Code assumes a condition holds (e.g., "all plugins have file paths") but does not confirm it via input validation
- **Status**: Discovered in Cycle 7 FAIL-3 (signature verification assumed filePath exists)
- **Mitigation Strategies**:
  - Phase 1 Spec enumerates all boundary conditions
  - Phase 2: every assumption requires an explicit guard (if check)
  - Tests cover edge cases (package-name plugins, null values, etc.)
- **Contingency Plan**: Code Review checklist adds "assumption validation" item

---

## External Risks

### R-E01: Claude Pro Token Quota Exhaustion
- **Probability**: M (Large iterations may consume massive tokens)
- **Impact**: H (Forced suspension of the entire iteration)
- **Mitigation Strategies**:
  - doc-keeper uses haiku (low cost)
  - Dispatch agents only when necessary; avoid unnecessary calls
  - Split large features into multiple smaller iteration cycles
- **Contingency Plan**: Suspend iteration, wait for quota recovery, and resume based on SOP 8.3 state determination

### R-E02: Outdated Reference Projects
- **Probability**: L
- **Impact**: L (Researcher reports based on old information)
- **Trigger Scenario**: `share/ref/` versions of openclaw/opencode/openoctopus are no longer updated
- **Mitigation Strategies**: Researcher notes reference version and date in reports
- **Contingency Plan**: Manually update `share/ref/` as needed

---

## Risk Overview

| ID | Risk | Level | Status |
|----|------|:----:|:----:|
| R-T01 | Context Window Overflow | High | Monitoring |
| R-T02 | agent_test build Failure | Medium | Materialized (Cycle 18), pending mitigation (infrastructure) |
| R-T03 | Windows Compatibility | Medium | Monitoring |
| R-T04 | SDK Version Out-of-sync | High | Mitigated (Interface Freeze) |
| R-P01 | Main Session Crash | High | Mitigated (State Persistence) |
| R-P02 | Rework Infinite Loop | Medium | Mitigated (Limit of 2) |
| R-P03 | Inconsistent Output Quality | Medium | Mitigated (Communication Protocol) |
| R-P04 | Content Conflicts | Low | Mitigated (Directory Separation) |
| R-P05 | Integration Point Missing | Medium | Discovered (Cycle 7), pending mitigation (checklist) |
| R-P06 | Spec Omission | Medium | Discovered (Cycle 7), pending mitigation (checklist) |
| R-P07 | Unvalidated Assumptions | Medium | Discovered (Cycle 7), pending mitigation (guard checks) |
| R-E01 | Token Quota Exhaustion | Medium | Monitoring |
| R-E02 | Outdated References | Low | Accepted |
