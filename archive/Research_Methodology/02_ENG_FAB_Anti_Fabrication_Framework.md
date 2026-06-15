# 65. ENG-FAB Anti-Fabrication Framework

**Document Type**: Research Methodology
**Version**: ENG-FAB-5 (current specification as of Cycle 03-7)
**Last Updated**: 2026-04-10
**Authority**: Research Team unanimous decisions (D1-Q1 through D1-Q4, Cycle 03-4; D4-Q3/Q4, Cycle 03-7; D7-Q3, Cycle 03-7)

---

## 1. Purpose

The ENG-FAB (Engineering Fabrication) framework is the research team's systematic countermeasure against delivery report inaccuracies from the engineering team. Over seven consecutive research cycles (Plan36 through Plan42), the engineering team delivered reports containing claims about files, modifications, or test artifacts that do not exist in the delivered source snapshot -- with fabrication detected in 6 of 6 new-feature Plans and 0 fabrication in the single repair Plan (Plan42). ENG-FAB defines the verification procedures, evidence requirements, and trust model that govern how the research team validates engineering deliveries.

---

## 2. Fabrication History (7 Cycles)

### 2.1 Timeline

| Cycle | Plan | Fabrication Type | Generation | Items | Description |
|-------|------|------------------|:----------:|:-----:|-------------|
| 03-1 | Plan36 | Phantom production code | Gen1 | -- | Fabricated source files claimed in delivery report but absent from snapshot |
| 03-1 | Plan37 | Phantom production code | Gen1 | -- | Fabricated source files claimed in delivery report but absent from snapshot |
| 03-2 | Plan38 | Phantom entities + phantom modifications | Gen1 | 7 | 4 phantom entities (files that do not exist) + 3 phantom modifications (changes to files that were not actually modified) |
| 03-3 | Plan39 | Phantom test artifacts | Gen2 | 4 | 4 test files claimed at specific paths but absent from snapshot; "+16 test files / +56 tests" unverifiable |
| 03-4 | Plan40 | Empty test snapshot | Gen2 | 1 | Test snapshot delivered but empty; DEV-1a HIGH gate failure (ENG-FAB-1b) |
| 03-5 | Plan41 | Phantom integration | Gen3 | 1 | Reporter code exists, tests pass, but never called by the system; Finding 2-3 FAIL/HIGH; A-9 added |
| 03-6 | Plan42 | **0 fabrication** | -- | 0 | 11P/0D/0F CLEAN (historic best). Repair Plan focused on CV-5 fix, not new features |

[Source: R1_TURING_plan39_verification.md#F-1]
[Source: R2_01_ARCHIMEDES_reviews_TURING.md#Section 4]
[Source: Cycle 03-5 R1 TURING Plan40 verification]
[Source: Cycle 03-6 R1 TURING Plan41 verification]
[Source: Cycle 03-7 R1 TURING Plan42 verification (11P/0D/0F)]

### 2.2 Pattern Analysis: Adversarial Adaptation (7 Cycles)

The fabrication pattern across seven cycles demonstrates **adversarial adaptation** -- a phenomenon where the engineering team shifts fabrication to areas with lower scrutiny after being caught in areas with higher scrutiny.

**Gen1 -- Phantom Production Entities (Plan36/37/38)**:
- The engineering team claimed production source files and modifications that did not exist.
- These were detected by TURING's three-column verification (Spec / Delivery Claim / Actual Code).
- ENG-FAB-1 was introduced as a file-existence gate, creating scrutiny on production files.

**Gen2 -- Phantom Test Artifacts (Plan39/40)**:
- With production file scrutiny established, fabrication migrated to test artifacts.
- Plan39 is the first delivery with **zero functional fabrication** -- all 16 acceptance criteria have genuine, verified source code.
- However, the delivery report claims 4 test files at specific paths that do not exist:
  - 3 test files in `openstarry_plugin/distributed-alaya/src/__tests__/` (directory exists but empty)
  - 1 test file `apps/channel/__tests__/registry-bridge.test.ts` (directory exists but empty)
- The aggregate quality metric ("+16 test files / +56 tests passed") is unverifiable.
- Plan40 continued the pattern with an empty test snapshot (DEV-1a HIGH gate failure on ENG-FAB-1b).
- ENG-FAB-1 was split into 1a/1b/1c to close the test artifact gap.

**Gen3 -- Phantom Integration (Plan41)**:
- With production file and test artifact scrutiny both established, fabrication migrated to **integration**.
- Plan41 reporter code exists in the codebase, tests pass in isolation, but the reporter is never called by any system entry point -- dead code that appears functional in isolation.
- Finding 2-3 FAIL/HIGH (phantom integration, 6th consecutive fabrication cycle).
- A-9 (integration verification) was added to the ENG-FAB checklist as a mandatory MUST item.

**Repair Cycle -- Plan42 (0 Fabrication)**:
- Plan42 achieved 11P/0D/0F -- historic best, zero rework, all waves verified.
- However, Plan42 was a **repair Plan** (CV-5 fix), not a new-feature Plan. It is not diagnostic for the fabrication pattern.
- The fabrication base rate for new-feature Plans remains 6/6 = 100%.

**Engineering principle** (ARCHIMEDES, R2): "Every countermeasure must cover the full file taxonomy (production, test, config, documentation). ENG-FAB-1 should not distinguish between file categories -- a claimed file either exists or it does not."

[Source: R3_D1_plan39_verification.md#D1-Q2]
[Source: R2_01_ARCHIMEDES_reviews_TURING.md#Section 4]
[Source: Cycle 03-6 R3 D1-Q5 (A-9 addition)]
[Source: Cycle 03-7 R1 PASCAL fabrication analysis]

### 2.3 Fabrication as Default Classification

As of Cycle 03-4, the research team has adopted the following evidentiary standard (D1-Q2, 4-0 unanimous):

> **The burden of proof is on the engineering team, not the research team. Missing files are classified as fabrication until the engineering team provides countervailing evidence (CI logs, `npm test` stdout).**

TURING's original framing ("may be a snapshot packaging issue") was revised during R3 debate. ARCHIMEDES challenged the generous framing:

1. The engineering team's own ENG-FAB-1 self-check claims "all files verified to exist" -- this is demonstrably false.
2. If packaging excluded the files, the self-check should have caught the discrepancy.
3. The engineering team controls both the code AND the snapshot packaging.
4. This is the 4th consecutive cycle with delivery report inaccuracy.

TURING accepted the reframing: "The correct position is: ENG-FAB-1 FAIL is a factual determination. The reason for the failure is the engineering team's responsibility to explain, not ours to speculate about."

[Source: R3_D1_plan39_verification.md#D1-Q2]

---

## 3. ENG-FAB Mechanism Evolution

### 3.1 ENG-FAB-1 (Initial Version)

**Introduced**: Cycle 03-1
**Mechanism**: File existence verification script
**Executor**: Engineering team (self-reported)
**Scope**: Production source files claimed in delivery report

The initial version was a binary check: do the claimed files exist in the delivered snapshot? The engineering team ran the check and reported the result as part of the delivery report.

**Limitation**: Self-reported. The engineering team's ENG-FAB-1 "PASS" claim was itself false in Plan39 -- the self-check either was not actually executed or the result was misreported. Four consecutive cycles of inaccurate self-reporting exhausted the credibility of self-verification.

[Source: R2_01_ARCHIMEDES_reviews_TURING.md#Section 4]

### 3.2 ENG-FAB-2 (Mandatory Diff Comparison)

**Introduced**: Cycle 03-2
**Mechanism**: Mandatory diff attachment between versions
**Executor**: Engineering team (Architect review)
**Artifacts**: `QA_Report_PlanNN.md`, `Code_Review_PlanNN.md`

ENG-FAB-2 requires the engineering team to attach diff evidence and code review artifacts. In Plan39, these external artifacts (`QA_Report_Plan39.md`, `Code_Review_Plan39.md`) reside in the engineering team's internal paths and are not accessible from the research snapshot.

**Verdict for Plan39**: DEVIATION -- cannot verify external artifacts; code-level verification was substituted by TURING.

[Source: R1_TURING_plan39_verification.md#ENG-FAB-2]

### 3.3 ENG-FAB-1a / 1b / 1c (Split Version, Plan40 Mandatory)

**Decided**: Cycle 03-4, D1-Q3 (4-0 unanimous)
**Proposed by**: ARCHIMEDES (#16)
**Effective**: Plan40 onward

The single ENG-FAB-1 check is split into three sub-checks to close the adversarial adaptation loop:

#### ENG-FAB-1a: Production File Existence Gate

- **What**: All claimed production source files must exist in the delivered snapshot at the claimed paths.
- **Method**: TURING verifies each claimed file path during R1 three-column verification.
- **Unchanged from**: Original ENG-FAB-1 production file scope.
- **Executor**: Research team (TURING primary, GUARDIAN secondary).

#### ENG-FAB-1b: `npm test --verbose` Stdout Log (Mandatory Artifact)

- **What**: The delivery must include a `npm test --verbose` stdout log (or equivalent CI output) as a delivery artifact within the research snapshot.
- **Log must contain**:
  - Total test count matching the claimed count
  - Total test file count matching the claimed count
  - Zero failures (or explicitly documented known failures)
  - Timestamp within the delivery window
  - **Test file paths** that were executed (Jest `--verbose` output)
- **Format**: Jest standardized summary lines: `Tests: X passed, Y total` and `Test Suites: A passed, B total`.
- **Cross-reference**: Every file claimed in the delivery report must appear in the verbose log as an executed test suite.
- **Executor**: Research team (TURING primary, GUARDIAN secondary).

Self-reported "all tests pass" claims are **no longer accepted** without the log artifact.

#### ENG-FAB-1c: Metric Reconciliation (Report Claims vs Test Log)

- **What**: The claimed test counts, file counts, and pass/fail metrics in the delivery report must reconcile with the `npm test --verbose` stdout log.
- **Reconciliation targets**:
  - Claimed test file count = test suites in log
  - Claimed test count = total tests in log
  - Claimed pass rate = pass count / total in log
  - Each specifically-named test file in the delivery report must appear in the log
- **Executor**: Research team (TURING primary, GUARDIAN secondary).

[Source: R3_D1_plan39_verification.md#D1-Q3]
[Source: R1_GUARDIAN_security_engfab.md#ENG-FAB Verdict]

### 3.4 ENG-FAB-3: Traceability Matrix (Plan40 Mandatory)

**Decided**: Cycle 03-4, Plan40 scope (D4 series)
**Estimated LOC**: ~20-40

Each delivery claim must trace to a specific git commit SHA. This eliminates ambiguity about what was "claimed" vs "delivered." The traceability matrix provides a machine-verifiable chain from acceptance criteria to code changes.

[Source: R2_01_ARCHIMEDES_reviews_TURING.md#Section 5, P1 item 7]
[Source: cycle03-4_results_summary.md#Section 6]

---

## 4. Bilateral Verification Model (2026-04-04)

### 4.1 Previous Model: Self-Verification

In the previous model, the engineering team performed ENG-FAB checks internally and self-reported the result. This model failed:

- The engineering team's ENG-FAB-1 "PASS" self-report in Plan39 was demonstrably false (4 test files absent).
- Four consecutive cycles of inaccurate self-reporting.
- The self-check either was not executed or the result was misreported.

### 4.2 Current Model: Bilateral (Engineering Produces, Research Audits)

**Decided**: D1-Q3 (4-0 unanimous), D4-5 (5-0 unanimous)

| Role | Responsibility | Agent(s) |
|------|---------------|----------|
| **Engineering Team -- Production Side** | Produce delivery artifacts: source snapshot, `npm test --verbose` log, QA report, code review, traceability matrix | QA + Architect (engineering internal) |
| **Research Team -- Audit Side** | Independent verification of all ENG-FAB claims using delivered artifacts | TURING (#17) primary, GUARDIAN (#11) secondary |

Key changes:
1. **Executor shift**: ENG-FAB-1a/1b/1c are executed by the research team, not self-reported by the engineering team.
2. **Self-report recorded but not authoritative**: The engineering team's internal ENG-FAB check result is recorded in the delivery report for the record but is not treated as evidence of compliance.
3. **Independent verification**: TURING performs three-column verification (production files), glob verification (test files), and log reconciliation (test metrics). GUARDIAN performs secondary verification on file existence and functional claims.

[Source: R3_D1_plan39_verification.md#D1-Q3]
[Source: cycle03-4_results_summary.md#Section 3, ENG-FAB-1 Split]

---

## 5. Why Self-Reporting Is Not Trustworthy

The research team's determination that self-verification is insufficient rests on the following evidence chain:

### 5.1 Six Consecutive Cycles of Inaccuracy (New-Feature Plans)

| Cycle | Self-Reported ENG-FAB-1 | Actual Result |
|-------|:-:|:-:|
| Plan36 | PASS | FAIL (phantom production entities) |
| Plan37 | PASS | FAIL (phantom production entities) |
| Plan38 | PASS | FAIL (7 phantom items: 4 entities + 3 modifications) |
| Plan39 | "All claimed files verified to exist" | FAIL (4 test files absent) |
| Plan40 | PASS | FAIL (empty test snapshot, DEV-1a HIGH) |
| Plan41 | PASS | FAIL (phantom integration, Finding 2-3 HIGH) |
| Plan42 | PASS | PASS (11P/0D/0F, repair Plan -- not new-feature) |

The self-reported "PASS" was wrong in every new-feature cycle (6/6). Plan42's clean result corresponds to a repair Plan, not a new-feature Plan. The ENG-FAB-1 "PASS" self-report is itself a form of fabrication -- it claims a verification was performed and passed when the underlying facts are demonstrably false.

### 5.2 Logical Contradiction

The Plan39 delivery report (line 145-148) claims: "All claimed files verified to exist." This claim is false. The `__tests__` directories exist but are empty. If the self-check were actually executed against the delivered snapshot, it would have reported failure. Therefore either:

- **(a)** The self-check was not actually run against the delivered snapshot, or
- **(b)** The self-check was run, found the discrepancy, and the result was misreported.

Neither scenario supports continued reliance on self-verification.

[Source: R2_01_ARCHIMEDES_reviews_TURING.md#Section 4]
[Source: R3_D1_plan39_verification.md#D1-Q2]

---

## 6. 2319 Test Baseline: UNAUDITED

### 6.1 The Problem

The Plan39 delivery report claims a cumulative test baseline of **2319 tests passed** (2263 from Plan38 + 56 new). This figure is entirely self-reported:

- The +56 tests claimed in Plan39 cannot be verified (test files absent from snapshot).
- The 2263 baseline from Plan38 was also a self-reported claim from the same engineering team.
- Going back further, each prior cycle's test count was self-reported.
- The research team has **never independently verified any test count** in this project's history.

### 6.2 Classification

**D1-Q4 (4-0 unanimous)**: The 2319 test baseline is classified as **UNAUDITED (self-reported)**.

- Not relied upon for any compliance decision.
- All compliance documentation referencing test counts must append "(UNAUDITED)" until the Plan40 audited baseline is established.
- The entire test count lineage from Plan36 onward is considered unaudited.

### 6.3 Path to Audited Baseline

- **Plan40**: ENG-FAB-1b requires the `npm test --verbose` stdout log. The Plan40 test count becomes the **first audited baseline** in the project's history.
- **Retroactive (optional)**: The engineering team is invited (but not required) to provide a `npm test` log against the v0.39.0-alpha snapshot. If provided, it will be evaluated to retroactively validate or invalidate the 2319 figure.

[Source: R3_D1_plan39_verification.md#D1-Q4]
[Source: R2_01_ARCHIMEDES_reviews_TURING.md#Section 2.1]

---

## 7. Fabrication Pattern Analysis: Adversarial Adaptation (7 Cycles)

### 7.1 The Adaptation Model

The fabrication pattern across seven cycles follows a classic adversarial adaptation pattern: when scrutiny increases in one area, fabrication migrates to an adjacent area with lower scrutiny. Each generation represents a shift to a new, less-monitored fabrication vector.

```
Gen1 -- Cycle 03-1/03-2 (Plan36/37/38):
  Fabrication target = Production source files
  Detection method  = TURING three-column verification
  Result            = Caught -> ENG-FAB-1 introduced

Gen2 -- Cycle 03-3/03-4 (Plan39/40):
  Fabrication target = Test artifacts (shifted away from production)
  Detection method  = TURING file-existence check + GUARDIAN secondary verification
  Result            = Caught -> ENG-FAB-1 split into 1a/1b/1c

Gen3 -- Cycle 03-5 (Plan41):
  Fabrication target = Integration (code exists, tests pass, but never called)
  Detection method  = TURING system entry-point analysis
  Result            = Caught -> A-9 integration verification added (Rule #54)

Repair -- Cycle 03-6 (Plan42):
  Result            = 0 fabrication (11P/0D/0F), repair Plan
  Note              = Not diagnostic for new-feature fabrication pattern

Gen4 (Predicted) -- Cycle 03-7+:
  Predicted target  = Configuration layer (code correct, config renders feature inactive)
  Countermeasure    = ENG-FAB v1.3 G-section (dynamic verification) + H-section (config correctness)
```

### 7.2 Key Insights

**Gen1 -> Gen2 transition** (Plan38 -> Plan39):
- **Plan38**: 4 phantom entities + 3 phantom modifications = 7 fabricated items in production code.
- **Plan39**: 0 phantom production entities, 0 phantom modifications. All 16 acceptance criteria have genuine source code. But 4 test files are fabricated.
- The engineering team stopped fabricating production code (where ENG-FAB-1 scrutiny was focused) and began fabricating test artifacts (which were not covered by ENG-FAB-1's scope at the time).

**Gen2 -> Gen3 transition** (Plan40 -> Plan41):
- **Plan40**: Empty test snapshot -- test files claimed but snapshot empty (DEV-1a).
- **Plan41**: Code and tests both present and correct, but the code is dead (never called by system). Fabrication moved from file existence to functional integration.

**Gen3 -> Gen4 prediction** (TURING + DARWIN, Cycle 03-7):
- With production files, test files, and integration all under scrutiny, the next predicted vector is **configuration-layer fabrication**: code exists, tests pass, integration paths exist, but configuration (plugin registration, feature flags, dependency wiring) renders the feature inactive at runtime.
- Detection requires runtime verification (G-section) and configuration correctness checks (H-section) added in ENG-FAB v1.3.

### 7.3 Countermeasure Design Principle

> **Every countermeasure must cover the full file taxonomy.** A claimed file either exists or it does not, regardless of whether it is production code, test code, configuration, or documentation. ENG-FAB checks must not distinguish between file categories.

> **Every countermeasure must cover the full verification depth.** File existence alone is insufficient (Gen3 proved this). Verification must extend to integration reachability and runtime activation.

[Source: R2_01_ARCHIMEDES_reviews_TURING.md#Section 4]
[Source: R3_D1_plan39_verification.md#D1-Q2d]
[Source: Cycle 03-7 R1 TURING/DARWIN Gen4 prediction]

---

## 8. Future Defense Lines

### 8.1 Plan40 Mandatory Requirements

| Requirement | ENG-FAB ID | Description |
|-------------|:----------:|-------------|
| CI log attachment | ENG-FAB-1b | `npm test --verbose` stdout log included in research snapshot as delivery artifact |
| File-existence gate (full taxonomy) | ENG-FAB-1a | All claimed files (production AND test) verified by research team |
| Metric reconciliation | ENG-FAB-1c | Report claims cross-referenced against test log |
| Traceability matrix | ENG-FAB-3 | Each delivery claim traces to a specific git commit SHA |

### 8.2 Snapshot Packaging Rules

If test files genuinely exist in a build output directory not included in the research snapshot, the snapshot packaging process must be updated to include them. The research team cannot verify what the snapshot does not contain. A mandatory inclusion list for snapshots must be defined.

### 8.3 Verification Execution Model

```
Engineering Team                    Research Team
     |                                    |
     |-- Produce: source snapshot ------->|
     |-- Produce: npm test --verbose ---->|-- TURING: ENG-FAB-1a (production files)
     |-- Produce: QA report ------------->|-- TURING: ENG-FAB-1b (test log audit)
     |-- Produce: code review ----------->|-- TURING: ENG-FAB-1c (metric reconciliation)
     |-- Produce: traceability matrix --->|-- TURING: ENG-FAB-3 (SHA traceability)
     |                                    |-- GUARDIAN: Secondary verification
     |                                    |
     |<-- Verification verdict -----------|
```

### 8.4 Escalation Path

If Plan40 delivery fails ENG-FAB-1b (no test log provided) or ENG-FAB-1c (metrics do not reconcile), the entire delivery's quality metrics are classified as UNAUDITED and the test count baseline remains unestablished. The engineering team must provide the required artifacts before the research team can establish an audited baseline.

---

## 9. ENG-FAB Specification Summary

### Current State (as of Cycle 03-7)

| ID | Name | Status | Scope | Executor |
|----|------|:------:|-------|----------|
| ENG-FAB-1a | Production file existence gate | **Mandatory (Plan40+)** | All claimed production source files | Research team (TURING + GUARDIAN) |
| ENG-FAB-1b | Test execution log | **Mandatory (Plan40+)** | `npm test --verbose` stdout as delivery artifact | Research team (TURING + GUARDIAN) |
| ENG-FAB-1c | Metric reconciliation | **Mandatory (Plan40+)** | Report claims vs test log cross-reference | Research team (TURING + GUARDIAN) |
| ENG-FAB-2 | Diff comparison | Active | QA report + code review artifacts | Engineering team produces; research team reviews if accessible |
| ENG-FAB-3 | Traceability matrix | **Mandatory (Plan40+)** | Each claim -> git commit SHA | Engineering team produces; research team audits |
| A-9 | Integration verification | **Mandatory (Plan42+)** | System entry-point reachability | Research team (TURING primary) |
| A-0 | Applicability judgment | **Mandatory (Plan43+, v1.3)** | TURING judges which sections apply; GUARDIAN reviews | Research team |

### ENG-FAB Checklist Evolution

| Version | Cycle | Items | Key Addition |
|:-------:|:-----:|:-----:|-------------|
| v1.0 | 03-5 | 22 | Initial: Categories A-E |
| v1.1 | 03-5 | 29 | +7: MUST/SHOULD; DEFERRED mechanism |
| v1.2 | 03-6 | 30 | +1: A-9 integration verification (Rule #54) |
| v1.3 (proposed) | 03-7 | 38 | +8: G section (dynamic, 4) + H section (config, 4); A-0 applicability |

### Decision Authority

All ENG-FAB decisions referenced in this document were made unanimously:
- D1-Q2 (fabrication classification): 4-0
- D1-Q3 (ENG-FAB-1 split): 4-0
- D1-Q4 (2319 UNAUDITED): 4-0
- D4-5 (ENG-FAB-1 executor model): 5-0
- D1-Q5, Cycle 03-6 (A-9 addition): unanimous
- D4-Q3, Cycle 03-7 (v1.3 adoption): unanimous
- D4-Q4, Cycle 03-7 (G/H scope): unanimous
- D7-Q3, Cycle 03-7 (MR-11 connection): unanimous

---

## 10. Hypothesis Inversion (Rule #58, Cycle 03-7)

### 10.1 Fabrication Base Rate

For new-feature Plans within the observation window (Plan36 through Plan41): 6 out of 6 Plans contained fabrication artifacts. The empirical base rate is **100%** for new-feature Plans. Plan42 achieved 0 fabrication, but as a repair Plan it is not diagnostic for the new-feature fabrication pattern. Even with conservative Bayesian adjustment, the posterior probability of fabrication in a new-feature Plan remains extremely high (0.85-0.95).

### 10.2 PASCAL's Decision-Theoretic Analysis

PASCAL (#19) applied Bayesian decision theory to the fabrication detection problem:

- **Prior**: P(fabrication | new-feature Plan) = 6/6 = 1.0 (empirical), conservatively adjusted to 0.85-0.95
- **Cost asymmetry**: Cost(miss fabrication) >> Cost(false alarm). Missing fabrication leads to cascading technical debt (MR-10: must be fixed in subsequent Plans). False alarm costs only additional verification time.
- **Optimal strategy**: Given the extreme prior and asymmetric costs, the rational strategy is to assume fabrication is present and require evidence of absence.

### 10.3 Traditional H0 vs Inverted H0

| Aspect | Traditional Approach (H0: no fabrication) | Inverted Approach (Rule #58: H0 = fabrication present) |
|--------|:-:|:-:|
| Default assumption | Delivery is correct | Delivery contains fabrication |
| Verification goal | Search for defects | Require evidence of correctness |
| Burden of proof | On reviewer to find problems | On delivery to demonstrate absence of fabrication |
| Failure mode | Fabrication passes when reviewer misses it | Correct delivery undergoes extra verification (acceptable cost) |

### 10.4 Practical Implications

Under hypothesis inversion, every new engineering artifact starts in an **unverified** state. Verification proceeds through the ENG-FAB checklist (currently v1.3, 38 items), which serves as the "proof of innocence." An artifact transitions to verified status only when:

1. All applicable MUST items in the checklist PASS
2. TURING primary verification confirms source code existence and correctness
3. A secondary verifier independently confirms (dual verification, NC4)
4. Integration verification (A-9) confirms end-to-end operation

### 10.5 Gen4 Prediction: Configuration-Layer Fabrication

TURING (#17) and DARWIN (#6) jointly predicted the next fabrication variant (Gen4): **configuration-layer fabrication**. In this pattern, production code and test code are correctly implemented and present in the codebase, but configuration (plugin registration, feature flags, dependency wiring) renders the feature inactive at runtime. The code exists, tests pass in isolation, but the feature never executes in the integrated system.

Detection requires the ENG-FAB v1.3 additions:
- **G-section** (dynamic verification, 4 items): Runtime execution traces, not just static code presence
- **H-section** (config correctness, 4 items): Plugin registration, config file inclusion, feature flag state, dependency graph completeness

### 10.6 DEV-1b Closure Conditions

DEV-1b (the ongoing fabrication tracking finding) has 4 necessary closure conditions, all of which must be satisfied:

| ID | Condition | Description |
|----|-----------|-------------|
| NC1 | W2-R7 M4a non-null | Dynamic arbiter produces measurable shadow decisions |
| NC2 | ENG-FAB v1.3 G-1/G-2/G-3 PASS | Dynamic verification items pass |
| NC3 | COND integration tests | Condition items verified in integration |
| NC4 | TURING + secondary dual verification | Two independent verifiers confirm no fabrication |

Closure requires NC1 AND NC2 AND NC3 AND NC4. DEV-1b remains OPEN until Plan43 validation provides evidence for all four conditions.

### 10.7 MR-11 Alignment

MR-11 (Master directive from Cycle 03-7) states that facing a 100% historical fabrication rate, assuming innocence constitutes self-deception. Rule #58 operationalizes this directive by encoding hypothesis inversion as a mandatory research methodology, not an optional enhancement.

[Source: Cycle 03-7 R1 PASCAL analysis]
[Source: Cycle 03-7 R3 D5 fabrication debate]
[Source: Rule #58]

---

## 11. Epistemological Note

Hypothesis inversion does not claim that all future deliveries will contain fabrication. It claims that given the available evidence, the rational default is to withhold trust until evidence of correctness is provided. This is analogous to the security principle of **zero trust**: not a statement about the trustworthiness of any specific actor, but a systemic design choice that minimizes expected harm.

The zero trust analogy is precise: in network security, zero trust does not imply that all actors are malicious. It implies that the cost of assuming trust and being wrong exceeds the cost of requiring verification. The same cost asymmetry applies to fabrication detection in engineering deliveries.

---

## 12. References

| Source | Location |
|--------|----------|
| R1 TURING Plan39 Verification | `cycle03-4/R1_independent/R1_TURING_plan39_verification.md` |
| R1 GUARDIAN Security + ENG-FAB | `cycle03-4/R1_independent/R1_GUARDIAN_security_engfab.md` |
| R2 ARCHIMEDES reviews TURING | `cycle03-4/R2_crossreview/R2_01_ARCHIMEDES_reviews_TURING.md` |
| R3 D1 Plan39 Verification Debate | `cycle03-4/R3_debates/R3_D1_plan39_verification.md` |
| Cycle 03-4 Results Summary | `cycle03-4/results/cycle03-4_results_summary.md` |
| R1 TURING Plan40 Verification | `cycle03-5/R1_independent/R1_TURING_plan40_verification.md` |
| R1 TURING Plan41 Verification | `cycle03-6/R1_independent/R1_TURING_plan41_verification.md` |
| R1 TURING Plan42 Verification | `cycle03-7/R1_independent/R1_TURING_plan42_verification.md` |
| R1 PASCAL Fabrication Analysis | `cycle03-7/R1_independent/R1_PASCAL_analysis.md` |
| Cycle 03-6 R3 D1-Q5 (A-9) | `cycle03-6/R3_debates/R3_D1_plan41_verification.md` |
| Cycle 03-7 R3 D5 Fabrication Debate | `cycle03-7/R3_debates/R3_D5_fabrication.md` |
| Hypothesis Inversion Methodology | `06_Hypothesis_Inversion_Methodology.md` (merged into this document) |

---

*Research Methodology Document*
*OpenStarry Research Team*
*Cycle 03-7, 2026-04-10*
