# Coordinator Quick-Reference Checklist

Operational checklist for each iteration. The Coordinator (main session) executes these items sequentially.

---

## Starting a New Iteration

```
Cycle ID: {YYYYMMDD}_cycle{N}
Target Plan: Plan__________
```

### Phase 0: Planning
- [ ] Confirm User instruction and identify the target Plan.
- [ ] Create the cycle directory:
  ```bash
  CYCLE="{YYYYMMDD}_cycle{N}"
  mkdir -p share/test/reports/arch_reviews/$CYCLE
  mkdir -p share/test/reports/dev_logs/$CYCLE
  mkdir -p share/test/reports/qa_results/$CYCLE
  mkdir -p share/test/reports/research/$CYCLE
  mkdir -p share/test/reports/sys_summary/$CYCLE
  ```
- [ ] Dispatch **researcher** for pre-research → Wait for research report.
- [ ] Dispatch **doc-keeper** to record the iteration plan in `Iteration_Log.md`.
- [ ] ✅ Exit: Research report exists + Iteration plan recorded.

### Phase 1: Design
- [ ] Notify **architect** with: cycle_id + Plan path + research report path.
- [ ] Wait for Architecture_Spec production.
- [ ] Confirm the Spec contains **Frozen Interface Definitions**.
- [ ] Confirm if the Spec notes **SDK Interface Changes** (determines Phase 2 sequence).
- [ ] Dispatch **doc-keeper** to record design decisions.
- [ ] ✅ Exit: Spec exists + Interface frozen + Decisions recorded.

### Phase 1.5: Baseline (Safety Net)
- [ ] Establish the pre-Phase-2 baseline:
  ```bash
  bash scripts/baseline.sh $CYCLE
  ```
- [ ] Confirm the output "Baseline Saved."
- [ ] ✅ If Phase 2 fails and requires rollback: `bash scripts/restore.sh $CYCLE baseline`

### Phase 2: Implementation
- [ ] **Are there SDK Interface changes?**
  - YES →
    1. Dispatch **dev-core** for Step 2a, with prompt explicitly specifying:
       "Implement ONLY SDK Interface changes (packages/sdk/) from the Architecture_Spec. Verify with pnpm build. Do not implement other parts."
    2. Wait for dev-core Step 2a completion (build pass).
    3. Dispatch in parallel:
       - **dev-core** Step 2b, with prompt explicitly specifying:
         "SDK Interface complete in Step 2a. Implement remaining changes in packages/core, packages/shared, and apps/runner."
       - **dev-plugin**, with prompt specifying the Spec path normally.
  - NO → Dispatch **dev-core** + **dev-plugin** in parallel directly.
- [ ] Wait for both `pnpm build` PASS + dev log entries.
- [ ] (Optional) Simultaneously dispatch **researcher** for next-cycle pre-research.
- [ ] ⚠️ If `INTERFACE CHANGE NEEDED` is received:
  - Suspend Phase 2.
  - Notify architect to evaluate a Spec Addendum.
  - Notify dev-plugin to re-read once the Addendum is published.
- [ ] ✅ Exit: All builds PASS + Dev logs exist.

### Phase 2.5: Sync
- [ ] Execute the sync script:
  ```bash
  bash scripts/sync-to-test.sh
  ```
- [ ] Confirm sync completion (last output "Sync Complete").
- [ ] ⚠️ If build fails after sync → Return to Phase 2.
- [ ] ✅ Exit: agent_test build PASS.

### Phase 3: Verification
- [ ] Dispatch **qa** to run tests (in `agent_test/`) → Wait for QA Report.
- [ ] Dispatch **architect** for Code Review (reading `agent_dev/`) → Wait for Code Review Report.
- [ ] (qa and architect can proceed in parallel).
- [ ] ✅ Exit: Both reports produced.

### Phase 4: Convergence
- [ ] Read QA Report + Code Review Report.
- [ ] Determine outcome:

**→ If all PASS:**
- [ ] Snapshot (exclude node_modules/dist):
  ```bash
  bash scripts/snapshot.sh $CYCLE
  ```
- [ ] Dispatch **doc-keeper** to update Plan (✅) + Iteration_Log.
- [ ] Write Summary Report to `share/test/reports/sys_summary/$CYCLE/`.
- [ ] **Lessons Learned Review**:
  - What issues were encountered? (Technical, process, communication).
  - What worked well? What needs improvement?
  - Does the Risk Register need updating? (New risks, status changes).
  - Dispatch **doc-keeper** to append to `Lessons_Learned.md`.
- [ ] Report results to User.

**→ If any FAIL:**
- [ ] Categorize each FAIL item:
  - `Code Fix` → Return to Phase 2 (partial fix only).
  - `Design Fix` → Return to Phase 1 (architect revises Spec).
  - `Plan Fix` → Return to Phase 0 (re-planning).
- [ ] Increment Rework count (Cycle reworks: ___/2).
- [ ] If reworks > 2 → Escalate to User.
- [ ] Write Rework tasks into the Summary Report.
- [ ] Dispatch **doc-keeper** to record FAIL reasons in `Iteration_Log` + `Lessons_Learned`.

---

## Escalation Checklist

When escalating to the User:
- [ ] Collect opinions from all parties.
- [ ] Organize: Problem description + party positions + Coordinator recommendation.
- [ ] Present to User for verdict.
- [ ] Dispatch **doc-keeper** to record the verdict.

---

## Fault Recovery Manual

### Scenario A: Phase 2 implementation corrupted `agent_dev` (build failure, logic errors)
```bash
# Restore to the state before Phase 2 began
bash scripts/restore.sh $CYCLE baseline
```
Restart from Phase 2 after restoration.

### Scenario B: Sync script interrupted (incomplete `agent_test`)
```bash
# Simply re-execute (uses staging strategy; agent_test remains in old state if interrupted)
bash scripts/sync-to-test.sh
```
If `agent_dev` itself is problematic → Restore `agent_dev` first (Scenario A), then re-sync.

### Scenario C: Phase 4 incorrectly PASSED, bad code snapshotted
```bash
# Delete the bad snapshot
rm -rf share/openstarry_code_iteration/$CYCLE

# Restore agent_dev to the last good snapshot
bash scripts/restore.sh {Previous_Cycle_ID} snapshot
```
Then rerun the entire iteration cycle.

### Scenario D: Agent session crashed mid-operation (file partially written)
1. Check file status in `agent_dev`.
2. If code is inconsistent → `bash scripts/restore.sh $CYCLE baseline`.
3. If only a report was cut short → simply re-dispatch the corresponding agent.

### Scenario E: Desire to rollback to an earlier version
```bash
# List all available snapshots
ls share/openstarry_code_iteration/

# Restore to specified version
bash scripts/restore.sh {Any_Cycle_ID} snapshot
```

### Preventive Measures
- **Phase 1.5 Baseline is mandatory**; do not skip.
- Snapshots after Phase 4 PASS are permanent (unless manually deleted).
- If unsure about `agent_dev` state, run `cd agent_dev/openstarry && pnpm build` to verify.
