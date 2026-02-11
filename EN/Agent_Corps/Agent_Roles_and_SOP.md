# OpenStarry Agent Corps — Roles & Standard Operating Procedures

Established: 2026-02-09
Last Updated: 2026-02-09 (PMP Audit v2)

---

## 1. Agent Roster

| Agent | Role | Model | Working Directory | Output Directory |
|-------|------|-------|-------------------|------------------|
| `architect` | Architecture guardian, design specs, code review, security audit | sonnet | share/openstarry_doc/, agent_dev/ (read-only) | share/test/reports/arch_reviews/ |
| `dev-core` | Core monorepo developer (sdk, core, shared, runner) | sonnet | agent_dev/openstarry/ | share/test/reports/dev_logs/ |
| `dev-plugin` | Plugin ecosystem developer | sonnet | agent_dev/openstarry_plugin/ | share/test/reports/dev_logs/ |
| `qa` | Quality assurance — build, test, purity verification | sonnet | agent_test/ | share/test/reports/qa_results/ |
| `doc-keeper` | Documentation, decision persistence, plan tracking | haiku | share/openstarry_doc/ | share/openstarry_doc/ |
| `researcher` | Technical pre-research, reference project analysis | sonnet | share/ref/ | share/test/reports/research/ |
| **Coordinator** | Main session — orchestration, sync, convergence, escalation | opus | all | share/test/reports/sys_summary/ |

---

## 2. RACI Matrix

**R** = Responsible, **A** = Accountable, **C** = Consulted, **I** = Informed

| Phase | Coordinator | architect | dev-core | dev-plugin | qa | doc-keeper | researcher | User |
|-------|:-----------:|:---------:|:--------:|:----------:|:--:|:----------:|:----------:|:----:|
| **Phase 0: Planning** | **R/A** | C | I | I | I | **R** (Record) | **R** (Research) | **A** (Approve) |
| **Phase 1: Design** | I | **R/A** | C | C | I | **R** (Record Design Decisions) | C (Provide Research) | I |
| **Phase 1.5: Baseline** | **R/A** | I | I | I | I | I | I | I |
| **Phase 2: Implement** | I | C (Clarify spec) | **R/A** (core) | **R/A** (plugin) | I | I | R (Research next) | I |
| **Phase 2.5: Sync** | **R/A** | I | I | I | I | I | I | I |
| **Phase 3: Verify** | I | **R** (Code Review) | I | I | **R/A** (Test) | I | I | I |
| **Phase 4: Converge** | **R/A** | I | I | I | I | **R** (Update Docs) | I | **A** (Verdict PASS/FAIL) |
| **Rework** | **R** (Assign) | C/R (If design issue) | R (If core issue) | R (If plugin issue) | I | I | I | I |
| **Escalation** | **R** (Report) | C | C | C | C | I | I | **A** (Verdict) |

---

## 3. Standard Iteration Cycle

### Naming Convention

ID format for each iteration: `{YYYYMMDD}_cycle{N}`

Example: `20260210_cycle1`, `20260210_cycle2` (second cycle on the same day)

All reports for a given cycle are placed in the corresponding cycle subdirectory:
```
share/test/reports/arch_reviews/20260210_cycle1/Architecture_Spec_Plan05.1.md
share/test/reports/dev_logs/20260210_cycle1/dev-core_Plan05.1.md
share/test/reports/qa_results/20260210_cycle1/QA_Report_Plan05.1.md
share/test/reports/research/20260210_cycle1/Research_SessionIsolation.md
share/test/reports/sys_summary/20260210_cycle1/Summary.md
```

---

### Phase 0: Planning

**Trigger**: User instruction, or Coordinator proposal for the next cycle after a Phase 4 PASS.

**Participants**: Coordinator (R/A), researcher (R), doc-keeper (R), User (A)

**Steps**:
1. **Coordinator** receives User instructions and identifies the corresponding Implementation Plan.
2. **Coordinator** creates the current cycle directory (under all reports subdirectories).
3. **researcher** performs pre-research on the technical solution for the Plan (Shift-Left) and produces a research report.
4. **doc-keeper** records the current iteration plan into `share/openstarry_doc/Agent_Corps/Iteration_Log.md`.
5. **Coordinator** confirms the research report is ready and notifies the architect to enter Phase 1.

**Deliverables**:
| Output | Path | Responsible |
|------|------|--------|
| Research Report | `share/test/reports/research/{cycle_id}/Research_{Topic}.md` | researcher |
| Iteration Log Record | `share/openstarry_doc/Agent_Corps/Iteration_Log.md` (Append) | doc-keeper |

**Exit Criteria**:
- [ ] Research report exists and content is complete.
- [ ] Iteration plan is recorded.
- [ ] Coordinator confirms readiness for Phase 1.

---

### Phase 1: Design

**Entry Criteria**: All Phase 0 Exit Criteria met.

**Participants**: architect (R/A), doc-keeper (R), researcher (C)

**Steps**:
1. **architect** reads:
   - Implementation Plan: `share/openstarry_doc/Implementation_Plans/Plan{XX}_{Name}.md`
   - Research Report: `share/test/reports/research/{cycle_id}/Research_{Topic}.md`
   - Existing architecture code: `agent_dev/` (read-only)
2. **architect** produces Architecture_Spec, which must include:
   - **Frozen Interface Definitions** (TypeScript types) — the common basis for dev-core and dev-plugin.
   - Technical constraints and design decision rationales.
   - Five Aggregates mapping.
   - Security considerations.
3. **doc-keeper** records design decision rationales into `share/openstarry_doc/Agent_Corps/Iteration_Log.md`.

**Deliverables**:
| Output | Path | Responsible |
|------|------|--------|
| Architecture_Spec | `share/test/reports/arch_reviews/{cycle_id}/Architecture_Spec_{PlanName}.md` | architect |
| Design Decision Record | `share/openstarry_doc/Agent_Corps/Iteration_Log.md` (Append) | doc-keeper |

**Exit Criteria**:
- [ ] Architecture_Spec exists.
- [ ] Spec includes frozen Interface definitions.
- [ ] Design decisions are recorded.
- [ ] Coordinator confirms Spec is complete and ready for Phase 2.

**Important**: Interface definitions in the Architecture_Spec are **frozen** upon publication. Any modifications during Phase 2 must follow the escalation process (see Section 6).

---

### Phase 1.5: Baseline (Safety Net)

**Entry Criteria**: All Phase 1 Exit Criteria met.

**Participants**: Coordinator (R/A)

**Purpose**: Back up the current state of `agent_dev` before Phase 2 implementation begins. If Phase 2 fails and is unfixable, a quick rollback can be performed.

**Execution**: `bash scripts/baseline.sh {cycle_id}`

**Output**: `share/openstarry_code_iteration/{cycle_id}_baseline/`

**Restore Command**: `bash scripts/restore.sh {cycle_id} baseline`

---

### Phase 2: Implementation

**Entry Criteria**: Phase 1.5 Baseline established.

**Participants**: dev-core (R/A), dev-plugin (R/A), architect (C), researcher (R - pre-research next cycle)

**Execution Order**:

```
Step 2a: dev-core implements SDK Interface changes (if any)
         → verify via pnpm build
         ↓
Step 2b: dev-core (remaining implementation) + dev-plugin (plugin implementation) in parallel
         → respective verification via pnpm build
```

If the Architecture_Spec does not change the SDK Interface, Step 2a is skipped, and dev-core and dev-plugin proceed in parallel.

**Steps (dev-core)**:
1. Read Spec: `share/test/reports/arch_reviews/{cycle_id}/Architecture_Spec_{PlanName}.md`
2. If there are Interface changes: modify `packages/sdk/` first and run `pnpm build` to confirm compilation.
3. Implement related changes in `packages/core/`, `packages/shared/`, and `apps/runner/`.
4. Run `pnpm build` for verification.
5. Write dev log.

**Steps (dev-plugin)**:
1. Read Spec (same path as above).
2. **Wait for Step 2a to complete** (if SDK Interface changes exist).
3. Implement plugin changes.
4. Run `pnpm build` for verification.
5. Write dev log.

**Steps (researcher, parallel)**:
- Pre-research technical solutions for the next iteration (if known).

**Deliverables**:
| Output | Path | Responsible |
|------|------|--------|
| dev-core Log | `share/test/reports/dev_logs/{cycle_id}/dev-core_{PlanName}.md` | dev-core |
| dev-plugin Log | `share/test/reports/dev_logs/{cycle_id}/dev-plugin_{PlanName}.md` | dev-plugin |

**Exit Criteria**:
- [ ] dev-core `pnpm build` PASS.
- [ ] dev-plugin `pnpm build` PASS (if plugin changes exist).
- [ ] Dev logs are written.
- [ ] Coordinator confirms all builds pass and is ready for Phase 2.5.

---

### Phase 2.5: Sync to Test Environment

**Entry Criteria**: All Phase 2 Exit Criteria met.

**Participants**: Coordinator (R/A)

**Purpose**: Synchronize the latest code from `agent_dev/` to `agent_test/` to ensure QA tests the latest version.

**Quick Execution**: `bash scripts/sync-to-test.sh` (executed from openstarry_eco root).

**Sync SOP (Script Content)**:

```bash
# Step 1: Clear old source code in agent_test (retain node_modules for speed)
rm -rf agent_test/openstarry/packages agent_test/openstarry/apps
rm -rf agent_test/openstarry/dist agent_test/openstarry/.turbo

# Step 2: Copy source code (exclude node_modules, dist, .turbo)
cp -r agent_dev/openstarry/packages agent_test/openstarry/packages
cp -r agent_dev/openstarry/apps agent_test/openstarry/apps

# Copy root-level config files (if changed)
cp agent_dev/openstarry/package.json agent_test/openstarry/package.json
cp agent_dev/openstarry/tsconfig*.json agent_test/openstarry/
cp agent_dev/openstarry/vitest*.* agent_test/openstarry/ 2>/dev/null
cp agent_dev/openstarry/pnpm-workspace.yaml agent_test/openstarry/ 2>/dev/null

# Step 3: Synchronize plugins (same strategy)
for dir in agent_dev/openstarry_plugin/*/; do
  plugin=$(basename "$dir")
  rm -rf "agent_test/openstarry_plugin/$plugin/src" "agent_test/openstarry_plugin/$plugin/dist"
  cp -r "agent_dev/openstarry_plugin/$plugin/src" "agent_test/openstarry_plugin/$plugin/src"
  cp "agent_dev/openstarry_plugin/$plugin/package.json" "agent_test/openstarry_plugin/$plugin/package.json"
  cp "agent_dev/openstarry_plugin/$plugin/tsconfig.json" "agent_test/openstarry_plugin/$plugin/tsconfig.json" 2>/dev/null
done
cp agent_dev/openstarry_plugin/package.json agent_test/openstarry_plugin/package.json 2>/dev/null
cp agent_dev/openstarry_plugin/pnpm-workspace.yaml agent_test/openstarry_plugin/pnpm-workspace.yaml 2>/dev/null

# Step 4: Install dependencies
cd agent_test/openstarry && pnpm install
cd agent_test/openstarry_plugin && pnpm install

# Step 5: Verify basic build
cd agent_test/openstarry && pnpm build
```

**Exit Criteria**:
- [ ] agent_test source code synchronized.
- [ ] `pnpm install` successful.
- [ ] `pnpm build` successful.
- [ ] Coordinator confirms readiness for Phase 3.

**Failure Handling**: If the build fails after sync, the issue is in Phase 2; return to Phase 2 for correction (do not enter Phase 3).

---

### Phase 3: Verification

**Entry Criteria**: All Phase 2.5 Exit Criteria met.

**Participants**: qa (R/A), architect (R)

**Parallelizable**: qa and architect can proceed simultaneously and independently.

**Steps (qa)**:
1. In `agent_test/openstarry/`, execute:
   - `pnpm build` — Build verification.
   - `pnpm test` — Unit tests (Baseline: 82 tests).
   - `pnpm test:purity` — Microkernel purity check.
2. In `agent_test/openstarry_plugin/`, execute:
   - `pnpm build` — Plugin build verification.
   - `pnpm test` — Plugin tests (if any).
3. Produce QA Report.

**Steps (architect)**:
1. Read Architecture_Spec: `share/test/reports/arch_reviews/{cycle_id}/Architecture_Spec_{PlanName}.md`
2. Read implementation code: `agent_dev/` (read-only).
3. Verify item by item:
   - Interface implementation adheres to the frozen Spec.
   - Five Aggregates compliance.
   - Microkernel purity.
   - pushInput pattern usage.
   - Security vulnerabilities.
4. Produce Code Review Report (PASS / FAIL / CONDITIONAL).

**Deliverables**:
| Output | Path | Responsible |
|------|------|--------|
| QA Report | `share/test/reports/qa_results/{cycle_id}/QA_Report_{PlanName}.md` | qa |
| Code Review | `share/test/reports/arch_reviews/{cycle_id}/Code_Review_{PlanName}.md` | architect |

**Exit Criteria**:
- [ ] QA Report exists.
- [ ] Code Review Report exists.
- [ ] Coordinator has read both reports; enter Phase 4.

---

### Phase 4: Convergence

**Entry Criteria**: All Phase 3 Exit Criteria met.

**Participants**: Coordinator (R/A), doc-keeper (R), User (A)

**Steps**:
1. **Coordinator** reads QA Report and Code Review Report.
2. **Coordinator** aggregates them into a Summary Report.
3. **Coordinator** makes a determination:

   **→ If all PASS:**
   - Snapshot: `bash scripts/snapshot.sh {cycle_id}` (exclude node_modules/dist).
   - **doc-keeper** updates:
     - Check off completed items in Implementation Plan (✅).
     - Append cycle results to `Iteration_Log.md`.
   - **Coordinator** reports results to User and asks for next steps.

   **→ If any FAIL:** Enter Rework process (see Section 4).

**Deliverables**:
| Output | Path | Responsible |
|------|------|--------|
| Summary Report | `share/test/reports/sys_summary/{cycle_id}/Summary.md` | Coordinator |
| Updated Iteration Log | `share/openstarry_doc/Agent_Corps/Iteration_Log.md` | doc-keeper |
| Updated Plan (✅) | `share/openstarry_doc/Implementation_Plans/` | doc-keeper |

---

## 4. Rework Process (Phase 4 FAIL)

### 4.1 Problem Classification

| Level | Description | Return To | Example |
|------|------|--------|------|
| **Code Fix** | Implementation bug, test failure, build error. | Phase 2 (partial fix only) | Type error, test assertion failure. |
| **Design Fix** | Spec itself is flawed or incomplete. | Phase 1 (architect revises Spec) | Ill-conceived interface design, missing edge cases. |
| **Plan Fix** | Plan requirements themselves are problematic. | Phase 0 (re-planning) | Infeasible technical solution, contradictory requirements. |

### 4.2 Rework SOP

1. **Coordinator** reads FAIL items from QA Report and Code Review Report.
2. **Coordinator** classifies each FAIL item (Code Fix / Design Fix / Plan Fix).
3. **Coordinator** produces Rework Tasks and writes them into the Summary Report:
   ```
   ## Rework Tasks
   - [ ] [FAIL-1] (Code Fix) Description → assigned to dev-core/dev-plugin
   - [ ] [FAIL-2] (Design Fix) Description → assigned to architect
   ```
4. **Coordinator** recreates Baseline: `bash scripts/baseline.sh {cycle_id}_rework{N}` (preserves pre-rework state for secondary rollback).
5. Return to corresponding Phase based on classification; **only FAIL parts require rework**.
6. After correction, proceed downward from the subsequent phase (Phase 2.5 Sync must be rerun).

### 4.3 Rework Limit

- Maximum of **2 Reworks** per iteration cycle.
- Exceeding 2 → **Escalate to User** for verdict (see Section 6).

---

## 5. Phase 2 Parallel Strategy: Interface Freeze Mechanism

### 5.1 Principle

**TypeScript Interfaces** defined in the Architecture_Spec are **frozen** once Phase 1 is complete. Both dev-core and dev-plugin develop based on this frozen Interface.

### 5.2 Execution Order

```
Does Architecture_Spec include SDK Interface changes?
│
├── Yes → dev-core implements SDK Interface first (Step 2a)
│        → after pnpm build passes
│        → dev-core (remaining) + dev-plugin proceed in parallel (Step 2b)
│
└── No → dev-core + dev-plugin proceed directly in parallel
```

### 5.3 Exception Handling during Freeze

If dev-core discovers during implementation that the Interface must be modified:
1. dev-core **suspends implementation**.
2. dev-core records the issue in the dev log.
3. **Coordinator** notifies the architect.
4. architect evaluates → publishes a **Spec Addendum** to the same cycle directory.
5. dev-plugin adjusts after reading the Addendum.
6. If the impact is too large → Escalate to User.

---

## 6. Escalation and Dispute Resolution

### 6.1 Escalation Conditions

| Scenario | Escalation Path |
|------|---------|
| Disagreement between Agents on Spec interpretation. | → Coordinator arbitration. |
| Coordinator cannot arbitrate (design direction issue). | → User verdict. |
| Rework exceeds 2 times. | → User verdict. |
| Phase 2 reveals Interface requires major modification. | → User verdict. |
| Technical solution found infeasible (by researcher). | → User verdict. |

### 6.2 Escalation Process

1. Discovering party clearly marks `⚠️ ESCALATION NEEDED` in their report.
2. **Coordinator** collects opinions from all parties and aggregates them into a decision summary.
3. **Coordinator** presents the following to the User:
   - Problem description.
   - Positions of each party.
   - Coordinator recommendation (if any).
4. **User** delivers a verdict.
5. **doc-keeper** records the verdict in `Iteration_Log.md`.

---

## 7. doc-keeper Participation Schedule

| Phase | doc-keeper Responsibilities |
|-------|----------------|
| Phase 0 | Record: Current cycle goals, involved Plan, research directions. |
| Phase 1 | Record: architect's design decision rationales, frozen Interface content. |
| Phase 1.5 | No active tasks (Coordinator builds baseline). |
| Phase 2 | Standby: Record changes upon receiving a Spec Addendum. |
| Phase 3 | Standby: No active tasks. |
| Phase 4 PASS | Update Plan (✅), write iteration results, update Roadmap. |
| Phase 4 FAIL | Record FAIL reasons and Rework decisions. |

---

## 8. Communication Protocol

When the Coordinator dispatches an agent, the prompt must include the following information to avoid guesswork or working in the wrong context.

### 8.1 Required Fields (All Dispatches)

Each agent dispatch prompt **must include**:

| Field | Description | Example |
|------|------|------|
| `cycle_id` | Current iteration ID. | `20260210_cycle1` |
| `plan` | Target Plan name. | `Plan05.1 Session Isolation` |
| `task` | Specifically what to do (one sentence). | "Pre-research technical solution for Session Isolation." |
| `output_path` | Where reports/outputs should be written. | `share/test/reports/research/20260210_cycle1/` |

### 8.2 Phase-Specific Fields

| Phase | Agent | Additional Required Fields |
|-------|-------|-------------|
| Phase 0 | researcher | `research_topic`, `reference_projects` (directories in share/ref/ to scan). |
| Phase 0 | doc-keeper | `iteration_goals` (summary of current goals). |
| Phase 1 | architect | `plan_path`, `research_report_path`. |
| Phase 1 | doc-keeper | `spec_summary` (summary of key design decisions in the Spec). |
| Phase 2a | dev-core | `spec_path`, explicit instruction "Implement SDK Interface changes only." |
| Phase 2b | dev-core | `spec_path`, explicit instruction "SDK Interface complete, implement remaining parts." |
| Phase 2 | dev-plugin | `spec_path`, instruction "SDK Interface is ready" if SDK changes exist. |
| Phase 3 | qa | `cycle_id` (qa works in `agent_test/`, no extra path needed). |
| Phase 3 | architect | `spec_path` (path to Architecture_Spec for code review comparison). |
| Phase 4 | doc-keeper | `qa_result` (PASS/FAIL), `review_result` (PASS/FAIL), `lessons_learned`. |

---

## 9. Lessons Learned

### 9.1 Timing

The Coordinator performs a review **after Phase 4 concludes** (regardless of PASS or FAIL).

### 9.2 Review Process

1. **Coordinator** reviews the iteration cycle, answering:
   - What went well? (Process, collaboration, quality).
   - What went wrong? (Sticking points, rework reasons, communication issues).
   - How can we improve next time? (Process adjustments, prompt optimization, new check items).
2. **Coordinator** writes the review into the `## Lessons Learned` section of the Summary Report.
3. **doc-keeper** appends persistent improvement items to `share/openstarry_doc/Agent_Corps/Lessons_Learned.md`.
4. If modifications to the SOP/Agent definitions/Checklists are needed → Coordinator executes in the next Phase 0.

---

## 10. Fault Recovery

### 10.1 Recovery Scripts

| Script | Purpose | Command |
|------|------|------|
| `scripts/baseline.sh` | Create safety net before Phase 2. | `bash scripts/baseline.sh {cycle_id}` |
| `scripts/restore.sh` | Restore `agent_dev` from baseline or snapshot. | `bash scripts/restore.sh {cycle_id} baseline\|snapshot` |
| `scripts/sync-to-test.sh` | Sync `agent_dev` → `agent_test`. | `bash scripts/sync-to-test.sh` |
| `scripts/snapshot.sh` | Archive after Phase 4 PASS. | `bash scripts/snapshot.sh {cycle_id}` |

### 10.2 Recovery Scenarios

| Scenario | Recovery Method |
|------|---------|
| Phase 2 corrupted `agent_dev`. | `restore.sh {cycle_id} baseline` → Rerun Phase 2. |
| Sync interrupted halfway. | Rerun `sync-to-test.sh`. |
| Incorrect PASS + bad snapshot. | Delete bad snapshot → `restore.sh {PreviousCycle} snapshot`. |
| Agent session crashed. | Check files → restore baseline if inconsistent → redispatch agent. |
| Failure again after Rework. | `restore.sh {cycle_id}_rework{N} baseline`. |

---

## 11. Design Principles

1. **Roles ≠ Tasks** — Each Agent is a permanent role, not built for a single task.
2. **All Tier 1** — All 6 Agents are core team members, no hierarchy.
3. **Extensible** — Agents naturally take on extended tasks after defined Plans are complete.
4. **Decision Persistence** — All decisions must be written to files; nothing exists only in memory.
5. **Quality Gates** — Clear Entry/Exit Criteria for each Phase; no progression without fulfillment.
6. **Interface Freeze** — Interfaces are frozen upon Spec publication; changes require escalation.

---

## 12. Extended Mode

Trigger condition: All Plans in `share/openstarry_doc/Implementation_Plans/` are completed ✅.

Coordinator evaluates and asks User whether to continue:
- **researcher** → Deeply analyze reference projects, produce new plugin concepts + feasibility reports.
- **architect** → Comprehensive security audit + sandbox hardening solutions.
- **dev-core/dev-plugin** → Implement new features (Web UI, new plugins, etc.).
- **qa** → Verify extended features.
- **doc-keeper** → Record all decisions and results from extended work.
