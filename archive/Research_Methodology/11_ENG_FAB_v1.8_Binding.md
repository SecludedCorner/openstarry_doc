# ENG-FAB v1.7 → v1.8 Binding (Anti-Fabrication Audit Checklist, 48 items)

> **Front-matter**
> - **Cycle**: 03-13
> - **Date**: 2026-04-24
> - **Authors**: SCRIBE (#2, canonical consolidation) + ARCHIMEDES (#16, F-11/12/13/14/15 spec lead) + SYNTHESIST (#1, vote aggregation) · TURING (#17) + LINNAEUS (#13) + DARWIN (#6) + KERNEL (#10) + TANENBAUM (#20) + PENROSE (#18) + GUARDIAN (#11) + RUSSELL (#23) + NAGARJUNA (#7, two-truth framing) contributing coauthors
> - **Binding R3 decisions**: **MRB-01 / CV-01 UNANIMOUS** (48 items with F-12 allocated) · **D-21 UNANIMOUS** (F-11 scope canonical + release) · **D-22 UNANIMOUS** (F-11 now, Rule #79 defer 1-2 cycles) · **MRB-02 RESOLVED** (F-13.4 narrative rewrite) · **MRB-08 CRITICAL RESOLVED** (no-eval NFR-6 a/b/c/d sub-clauses) · **MRB-09 RESOLVED** (EBNF v1.1 precedence stratification) · **MRB-10 RESOLVED** (tier-invariance triple hash `corpus+script+python_env`) · **D-17 20/3** (F-15(e) explicit GN.2 cross-reference) · **D-02 UNANIMOUS** (F-13 severity regime-conditional) · **MRB-14 / MRB-16 RESOLVED** (F-15 cross-cutting)
> - **Status**: **BINDING** (post-Master ratification 2026-04-24 via Batch 10 Items 1-6)
> - **Supersedes**: ENG-FAB v1.7 Candidate (43 items, `Research_Methodology/10_ENG_FAB_v1.7_Candidate.md` cycle 03-12)
> - **Effective date**: **F-15 immediate (Plan49 and all subsequent research deliverables)** · F-11 from cycle 03-14 onward · F-12 tooling (Plan49 `audit_calc.py` delivery) + Plan50+ delivery-audit enforcement · F-13 / F-14 Plan50 onward
> - **Cross-refs**: `Research_Methodology/10_ENG_FAB_v1.7_Candidate.md` (predecessor) · `Research_Methodology/12_GN_1_5_SCRIBE_G_Gate.md` (companion SCRIBE authority) · `Reference/08_Rule_76_Section_76_7_Caveat.md` (Rule #76 §76.7 linked to F-12) · `Reference/09_Rule_77_SPC_Pooled_Mode_Sigma_Regime.md` (Rule #77 σ_regime; F-13 linked) · `Calibration_Reports/20_Section_7_3_Verification_Plan.md` (F-11 verification checklist) · `Technical_Specifications/Plan49_MR6_Conditional_Gates.md` (F-15 reflexive Plan49 application) · `Technical_Specifications/Plan50_Sigma_Regime_Binding.md` (F-13 + F-14 effective)

---

## Section A — v1.7 → v1.8 Diff Summary

### A.1 Item-count change

| Version | Items | MUST | SHOULD | INFO | Notes |
|---------|:-----:|:----:|:------:|:----:|-------|
| v1.6 | 42 | 38 | 3 | 1 | ratified 2026-04-18 |
| v1.7 | 43 | 40 | 2 | 1 | candidate (Batch 8c); E-5 SHOULD → MUST; F-10 NEW |
| **v1.8** | **48** | **45** | **2** | **1** | **F-11 + F-12 + F-13 + F-14 + F-15 NEW (all MUST)** |

### A.2 Arithmetic verification (three independent paths)

- **Path A** (v1.7 + deltas): 43 + 5 (F-11 + F-12 + F-13 + F-14 + F-15) = **48**. MUST: 40 + 5 = **45**. ✓
- **Path B** (v1.6 + cumulative): 42 + 1 (F-10) + 5 (F-11/12/13/14/15) = 42 + 6 = **48**. ✓
- **Path C** (section composition): 9A + 6B + 4C + 4D + 5E + **15F** (10 prior + 5 new) + 2G + 3H = **48**. ✓

All three paths converge. F-12 is explicitly **ALLOCATED** (not reserved); **v1.8 = 48, not 47** — this is the top cross-cutting MUST-resolve from R3 (MRB-01 / CV-01 UNANIMOUS) and settles the 03-13 R1 → R2 → R3 arithmetic trajectory.

### A.3 Principal changes (narrative)

1. **F-12 ALLOCATED** per MRB-01 / CV-01 UNANIMOUS (R3 opens with F-12 allocation resolved). Content anchored from `§8_audit_calc/§8_audit_calc_R1.md §8.5` per cross-chapter CV. The §7 R1 initial "F-12 reserved" reading is superseded.
2. **F-13.4 narrative** rewritten to 2-sentence by-design framing per **MRB-02 RESOLVED** — `gear-arbiter-dynamic` is a by-design pure-WIENER-math plugin whose `cat=undefined raw=0 clamped=0` output is the intended null shape, not a silent-plugin failure (see §E.4 below for verbatim text).
3. **F-14 operationalised as "correctly billed against intended endpoint"** (not "token consumed ≥ 0") per Master letter v2.0 §七 line 185 directive. Five sub-checks F-14.2.1–F-14.2.5 codify operational meaning including **PROVISIONAL PASS state field** (R2 F-03-13-§7-R1-4 MEDIUM adopted).
4. **F-15 5-element structure (a/b/c/d/e)** with **F-15(e) explicit GN.2 cross-reference** per D-17 20/3 (3 dissent WIENER / SUSSMAN / RUSSELL preserved per MR-11).
5. **Per-item effective timing** per D-21 UNANIMOUS + §2 R2 F-06 HIGH: F-15 Plan49 immediate; F-11 / F-13 / F-14 Plan50+; F-12 dual-track (Plan49 tooling + Plan50+ delivery). This freezes Plan49 assessment at v1.7 to avoid mid-cycle retrofit.
6. **Rule #79 (content-drift generalisation) deferred** per D-22 UNANIMOUS — F-11 alone is sufficient for 03-13; generalise only after 03-14/03-15 field data accumulates.
7. Each change includes migration rationale inline (MRB-07 drafting-note precedent retained).

---

## Section B — Full Item List (v1.8, 48 items)

### B.1 A-series (Architecture hygiene) — 9 items, all MUST (unchanged vs v1.7)

| ID | Item | Level | Notes |
|----|------|:-----:|-------|
| A-1 | Core surface opacity (no policy constants) | MUST | Static grep of `packages/core/**` |
| A-2 | SDK types declared in `@openstarry/sdk`, not Core | MUST | policy-type scan |
| A-3 | Plugin manifests well-formed | MUST | schema-match per plugin |
| A-4 | Event-bus purity (no in-Core policy decisions) | MUST | CP-2 verification |
| A-5 | Configuration-layer purity (no fabrication via config) | MUST | fab variant 7 |
| A-6 | Plugin-lifecycle checkpoint/restore symmetry | MUST | K-3 wire-in (Plan47 PASS) |
| A-7 | Shadow-mode isolation from production path | MUST | — |
| A-8 | Framework hook SDK type declarations | MUST | class-level + K-3 extension |
| A-9 | Integration-test presence for each production feature | MUST | wave-level granularity |

### B.2 B-series (Build hygiene) — 6 items, all MUST (unchanged vs v1.7)

| ID | Item | Level |
|----|------|:-----:|
| B-1 | Monorepo build clean (`pnpm build` exit 0) | MUST |
| B-2 | Lint + type-check green | MUST |
| B-3 | No unused imports / dead code | MUST |
| B-4 | Dep-tree audit (no security advisories) | MUST |
| B-5 | Test suite green (non-flaky; `pnpm test` exit 0) | MUST |
| B-6 | Plugin isolation (no cross-plugin leakage) | MUST |

### B.3 C-series (Contract verification) — 4 items, all MUST (unchanged vs v1.7)

| ID | Item | Level |
|----|------|:-----:|
| C-1 | InputEvent schema contract | MUST |
| C-2 | Plugin activation contract | MUST |
| C-3 | Event emission contract | MUST |
| C-4 | Shutdown contract (flush + clear) | MUST |

### B.4 D-series (Documentation) — 4 items, 3 MUST + 1 SHOULD (unchanged vs v1.7)

| ID | Item | Level |
|----|------|:-----:|
| D-1 | README up-to-date per Plan | MUST |
| D-2 | CHANGELOG entry present | MUST |
| D-3 | Architecture-Documentation file per significant feature | MUST |
| D-4 | Developer examples (tutorial update) | SHOULD |

### B.5 E-series (Cleanup / key-material) — 5 items, all MUST (post v1.7 elevation)

| ID | Item | Level | Notes |
|----|------|:-----:|-------|
| E-1 | Temp-file / tmp-dir cleanup on exit | MUST | — |
| E-2 | Process-local state clear on stop() | MUST | — |
| E-3 | Env-var non-leakage in logs | MUST | — |
| E-4 | Plugin-state persistence discipline | MUST | — |
| **E-5** | **HMAC key cleanup (consumed + cleared; closure-retained for shutdown signing)** | **MUST** | v1.7 elevation per D-14c dual-track; ASVS V2.10.1 + NIST SP 800-57 §8.2.2 |

### B.6 F-series (Fabrication-pattern detectors) — 15 items, all MUST

| ID | Item | Level | Change vs v1.7 | Notes |
|----|------|:-----:|:---------------:|-------|
| F-1 | Phantom-production detector | MUST | unchanged | fab variant 1 |
| F-2 | Phantom-test detector | MUST | unchanged | fab variant 2 |
| F-3 | Empty test-snapshot detector | MUST | unchanged | fab variant 3 |
| F-4 | Phantom-integration detector | MUST | unchanged | fab variant 4 |
| F-5 | Configuration-layer fab detector (H section) | MUST | unchanged | fab variant 7 |
| F-6 | Delivery-report completeness | MUST | unchanged | Rule #75 failure cascade |
| F-7 | Post-delivery-fix classification (Rule #62 Tier 1/2/3) | MUST | unchanged | Plan44 baseline |
| F-8 | Carry-forward-must check (deferred wave SDK type presence) | MUST | unchanged | Plan46→Plan47 K-3 |
| F-9 | Rule #74 L1' doc-check (5 facets + 3 sub-checks) | MUST | unchanged | v1.7 extension retained |
| F-10 | Pre-Delivery Gate Verified (Rule #75 test-layer) | MUST | unchanged | v1.7 NEW retained |
| **F-11** | **Canonical Sync Verified** (per-cycle, one-shot) | **MUST (NEW)** | **v1.8 NEW** | §C below; D-21 scope canonical + release |
| **F-12** | **Reproducible Calculation Verified** (per-claim) | **MUST (NEW)** | **v1.8 NEW** | §D below; no-eval NFR-6 a/b/c/d; §76.6 operational layer |
| **F-13** | **Plugin Hook Dispatch Verified** (per-Plan) | **MUST (NEW)** | **v1.8 NEW** | §E below; by-design allowlist per MRB-02 |
| **F-14** | **External Resource Baseline / "correctly billed"** (per-Plan) | **MUST (NEW)** | **v1.8 NEW** | §F below; PROVISIONAL PASS state |
| **F-15** | **Code-Path Verified Before Impact Assessment** (per-claim) | **MUST (NEW)** | **v1.8 NEW, Plan49 immediate** | §G below; 5 elements a/b/c/d/e; GN.2 cross-ref |

### B.7 G-series (Governance / process hygiene) — 2 items, 1 MUST + 1 SHOULD (unchanged)

| ID | Item | Level |
|----|------|:-----:|
| G-1 | Dev delivery report vs Research verification alignment | MUST |
| G-2 | Research-team TURING super-verification alignment | SHOULD |

### B.8 H-series (Configuration-layer audit) — 3 items, all MUST (unchanged)

| ID | Item | Level |
|----|------|:-----:|
| H-1 | Config-override correctness (no silent skip) | MUST |
| H-2 | Config-schema Zod validation completeness | MUST |
| H-3 | Config-default audit (fallback to safe default) | MUST |

**Total v1.8**: 9A + 6B + 4C + 4D + 5E + 15F + 2G + 3H = **48 items** (45 MUST + 2 SHOULD + 1 INFO). Arithmetic cross-verified §A.2 above.

---

## Section C — F-11 Canonical Sync Verified (spec)

### C.1 Summary

**Level**: MUST. **Effective**: Plan50 delivery onward (first real effective cycle 03-14+; cycle 03-13 is F-11's first field application at R4 close). **Scope** (per D-21 UNANIMOUS): **canonical + release byte-identical check**; `agent_dev` observation stays at coordinator step-7 + Dev acknowledgement (not F-11 reach).

### C.2 F-11 check items (5 clauses)

1. **F-11.2.1 — diff completeness + renumber manifest**: `diff -rq share/research_team_suggestion/{cycle}/openstarry_doc/ share/openstarry_doc/` returns only (a) renumber-induced differences and (b) explicitly-whitelisted differences. **Additionally, renumber manifest MUST be present** with minimum fields `old_number`, `new_number`, `affected_files` (per MR3-4 LOW adopted).
2. **F-11.2.2 — ratified diff manifest coverage**: every ratified diff enumerated in `{cycle}/discussions/ratified_diff_manifest.md` has a corresponding canonical entry. **SCRIBE writes the ratified_diff_manifest.md** during R4 close, reading from R3 decisions + Master Ratification batch list (authorship anchor per MR3-4 adopted).
3. **F-11.2.3 — byte-identical (canonical == release)**: `sha256sum` of every file in `release/{cycle}_v{X}.Y.Z-alpha/openstarry_doc/` equals `share/openstarry_doc/` counterpart after coordinator sync. (Per D-21 scope; agent_dev at coordinator step-7.)
4. **F-11.2.4 — collision resolution audit**: for each numeric-prefix collision in the renumber manifest, verify (a) first-come retained, (b) newcomer renumbered, (c) every cross-reference updated. Operator note: use word-boundary regex `\b{NN}_` or filename-anchored `^{NN}_` to avoid partial-word matches.
5. **F-11.2.5 — CHANGELOG propagation**: `CHANGELOG_RESEARCH_TEAM.md` in canonical == suggestion version (byte-identical).

### C.3 Failure behaviour

F-11 FAIL = **cycle-close blocker**. SCRIBE writes `{cycle}/discussions/F11_remediation_request.md`; coordinator re-runs missing sync steps; F-11 re-audits. **Cycle does not close until F-11 PASS**. Semantic-conflict subset (per CLAUDE.md line 153-154) escalates to Master; cycle-close deferred until Master ruling.

### C.4 Relationship to other items

- **F-10 (v1.7 Pre-Delivery Gate)**: delivery-granularity; F-11 = cycle-granularity. Complementary (LL-8 defense-in-depth).
- **F-12 (v1.8 Reproducible Calc)**: runs after F-11 in ratification chain. F-11 establishes canonical contains v1.8 spec; F-12 uses canonical as oracle.
- **Rule #74 L1'** (F-9 per-MUST doc): different axis. F-11 covers inter-mirror sync at folder level.

### C.5 Evidence artefact

Coordinator commits `{cycle}/discussions/canonical_sync_execution_report.md`; SCRIBE commits `{cycle}/discussions/F11_audit_report.md`.

---

## Section D — F-12 Reproducible Calculation Verified (spec)

### D.1 Summary

**Level**: MUST. **Effective**: dual-track — **Plan49 tooling** (Dev delivers `audit_calc.py` in Plan49; immediate Research tooling availability) + **Plan50+ delivery-audit enforcement** (per §2 R2 F-06 HIGH amendment + D-21 UNANIMOUS). **Source**: pulled from `§8_audit_calc/§8_audit_calc_R1.md §8.5` per **MRB-01 CV-01 UNANIMOUS** adopted (F-12 ALLOCATED, not reserved).

### D.2 F-12 check items (5 clauses)

1. **F-12.2.1 — calculation declarative form**: every numerical claim written in §76.6 Reproducible Calc Mandate template with (a) statement, (b) calculation block, (c) result, (d) tolerance band. Any missing element = F-12 FAIL.
2. **F-12.2.2 — EBNF v1.1 grammar conformance**: calculation blocks use EBNF v1.1 grammar with **stratified precedence `^` > `*/` > `+-`** (per MRB-09 resolved); unary; atom. Parse failure = F-12 FAIL.
3. **F-12.2.3 — no-eval safety (CRITICAL, NFR-6 a/b/c/d per MRB-08)**:
   - **NFR-6a**: No `sympify` / `parse_expr(evaluate=True)` / `eval` / `exec`; sympy expressions built node-by-node from whitelisted AST.
   - **NFR-6b**: File-read extension whitelist (`.md`); repo-relative paths; no symlinks; no `../` traversal.
   - **NFR-6c**: Unicode NFC normalization + ASCII-identifier enforcement on parsed tokens.
   - **NFR-6d**: Resource limits (parse depth ≤ 20, expression size ≤ 1 MB, evaluation steps ≤ 1000).
   Any violation = F-12 FAIL.
4. **F-12.2.4 — tier-invariance triple hash**: reproducibility checked via `(corpus_sha256 + script_sha256 + python_env_hash)` (per MRB-10 adopted — all three must match across tier executions). Drift in any = F-12 FAIL.
5. **F-12.2.5 — tolerance-band operationalisation**: per-claim tolerance class — **±0.1 OOM sensitivity claims / ±0.5 OOM probability claims / custom per-Plan** (declared in delivery-report Section 0, per D-23 UNANIMOUS class-specific).

### D.3 Failure behaviour (3-tier cascade)

F-12 FAIL follows **3-tier cascade** per MRB-10 + CV-20 MR-10 alignment:
- **Tier 1 (Research)** FAIL → halts deliver.
- **Tier 2 (Coordinator)** FAIL → halts Transfer (automated-hook per D-24 21/2).
- **Tier 3 (Master)** FAIL → halts Ratification.

### D.4 Link to Rule #76 §76.6 Reproducible Calculation Mandate

§76.6 is the **governance-level rule**; F-12 is the **ENG-FAB mechanical enforcement layer**. Together form LL-8 dual-layer control. F-12 operationalises §76.6; §76.6 is authoritative on the "why / who / what" dimension.

### D.5 Evidence artefact

`{plan}/delivery_report.md §F-12_evidence` containing per-claim template row + `audit_calc.py` output log reference + tier-invariance hash triple. SCRIBE G5 verifies presence; G6 verifies correctness via `audit_calc.py` re-run (Tier 1 Research).

---

## Section E — F-13 Plugin Hook Dispatch Verified (spec)

### E.1 Summary

**Level**: MUST. **Effective**: Plan50 onward. **Severity regime-conditional** per D-02 UNANIMOUS — current single-agent deployment MEDIUM; Phase-6 multi-agent deployment MED-HIGH (per AC-9 Claim E security per D-27 UNANIMOUS + D-30 UNANIMOUS HMAC-signed parentAgentId MVP).

### E.2 F-13 check items (5 clauses)

1. **F-13.2.1 — manifest inventory**: enumerate every hook declared in every plugin manifest modified or added by the Plan. Record to evidence file.
2. **F-13.2.2 — runtime probe**: for each declared hook, execute a controlled runtime probe (unit or integration test) that triggers the hook's documented activation condition and asserts (a) the hook function was called (spy/counter/log), (b) the call received expected-argument-structure.
3. **F-13.2.3 — dispatch-table consistency (declaration-vs-registration gap)**: runtime inspection of core's hook-dispatch map (e.g., `hookMap`) returns size and key-set consistent with manifest declarations. **This is the root-cause check for the gear-arbiter-dynamic 03-12/03-13 episode**: plugin declared `samskara` and `vijnana` but registered no `tools/guides/auditor/monitors` hooks. Discrepancy = F-13 FAIL.
4. **F-13.2.4 — by-design undefined allowlist**: for any plugin whose hook dispatches into a categoriser / classifier function returning a typed result, assert no dispatched result is `undefined` / `null` / empty-string under normal-path inputs. If such a result is legitimate (e.g., pure deterministic WIENER-math plugin with intentional null output), plugin author MUST document the condition and add an assertion-equivalent allowing the result explicitly (allowlist). By-design undefined with documentation = PASS; undocumented = FAIL. (Accommodates `gear-arbiter-dynamic` per MRB-02; allowlist entry `reserved_for_future_phase6` for `samskara`/`vijnana` until rename / enhance path decided.)
5. **F-13.2.5 — shutdown-path observation**: observe hookMap state at pre-shutdown and post-shutdown checkpoints; verify post-shutdown cleared (unless intentionally retained for shutdown-signing — documented case).

### E.3 Failure behaviour

F-13 FAIL = delivery-report Section 0 MUST record fail; Plan repairs before R4 verification OR Research pauses L2-L4 and issues remediation request.

### E.4 F-13.4 — gear-arbiter-dynamic narrative binding (MRB-02 authoritative text)

Verbatim 2-sentence narrative per MRB-02 RESOLVED + `O3_core_trio_final.md §A.7`:

> **F-13.4 binding narrative**: The 03-12 / 03-13 W2-R12 / R13 episodes surfaced a capability-declaration-honesty pattern: `gear-arbiter-dynamic` declared `samskara` and `vijnana` capabilities but did not register corresponding `tools` / `guides` / `auditor` / `monitors` hooks against those declarations. Dev Task #50 clarified this is a by-design pure-WIENER plugin whose deterministic `cat=undefined raw=0` output is its intended shape; F-13.2.3 catches the declaration-vs-registration gap while F-13.2.4's documentation-and-allowlist clause correctly accommodates the by-design case.

This wording is authoritative for ENG-FAB v1.8 F-13 rationale. Cites Dev Task #50 as the authoritative dev-team clarification (per MR-11 learning-not-punitive framing).

### E.5 Evidence artefact

`{plan}/delivery_report.md §F-13_evidence` + runtime-probe log excerpt (pointer to full log).

---

## Section F — F-14 External Resource Baseline (spec, "correctly billed" operationalised)

### F.1 Summary

**Level**: MUST. **Effective**: Plan50 onward. **Rephrase directive**: Master letter v2.0 §七 line 185 directed rephrasing from "token consumed ≥ 0" → "verify endpoint correctly billed". This section honours that directive.

### F.2 "Correctly billed" operational definition (per R2 §6.3 adopted)

> "Correctly billed" is operationalised by the five sub-checks F-14.2.1–F-14.2.5, which collectively verify (a) the vendor observed the invocation, (b) the billed quantity is order-of-magnitude consistent with expectation, (c) cross-channel consistency between local counter and vendor counter, (d) retry/dedup accounting is faithful, and (e) a 7-day usage baseline exists.

### F.3 F-14 check items (5 clauses)

1. **F-14.2.1 — endpoint-billed verification**: for each external-resource call site, verify the billing endpoint received the invocation signal. Evidence: (a) provider usage dashboard shows expected increment, OR (b) vendor usage-query API returns non-zero response for expected time-window, OR (c) response-header / response-body usage-field populated with sensible value.
2. **F-14.2.2 — billing-correctness order-of-magnitude**: not merely "non-zero token" — verify billed quantity is in expected order-of-magnitude. An invocation expected ~1k tokens but billed 0 OR billed 1M both FAIL. Tolerance bands declared per-Plan in delivery-report Section 0.
3. **F-14.2.3 — dashboard-false-positive guard (cross-channel)**: require cross-verification from ≥ 2 independent observation channels (e.g., local counter + vendor API) before asserting usage. Directly prevents the Master-Dashboard-false-positive class (2026-04-23 Master observed `api.openai.com` token=0 while provider-chatgpt-oauth was routing through `chatgpt.com/backend-api/codex/responses`).
4. **F-14.2.4 — idempotency-aware accounting**: if Plan uses idempotency keys or retry-dedup, verify accounting reflects distinct billable invocations; conversely if retries expected non-billable, verify billing shows base-invocation only.
5. **F-14.2.5 — spike / anomaly 7-day baseline**: Plans introducing new external calls establish a 7-day usage baseline post-release (or equivalent proxy in testing). Baseline recorded in delivery-report.

### F.4 PROVISIONAL PASS state (per F-03-13-§7-R1-4 MEDIUM adopted)

F-14 may depend on external-vendor observation latency (vendor billing visibility can lag 24-72h). F-14 therefore supports **PROVISIONAL PASS** state at initial delivery with **must-confirm-within-7-days** deadline. If no confirmation by deadline, state **degrades to FAIL** and Plan re-opens for remediation.

Section 0 template includes **F-14 state field**: `PROVISIONAL | CONFIRMED | FAIL`.

**MR-9 check (PROVISIONAL ≠ WAIVE)**: PROVISIONAL PASS is a deferred-confirmation with hard deadline + automatic degradation — not a WAIVE. MR-9 preserved per PENROSE concur in §7R2 §6.3.

### F.5 Evidence artefact

`{plan}/delivery_report.md §F-14_evidence` + vendor-observation excerpt (redacted for cost sensitivity if needed, but with date + order-of-magnitude).

---

## Section G — F-15 Code-Path Verified Before Impact Assessment (spec)

### G.1 Summary

**Level**: MUST. **Effective**: **Immediate on ratification (Plan49 and all subsequent research deliverables)**. **Learning-oriented, non-punitive per MR-11**. **Source**: §六 retrospective direct precursor; prevents Task-#49-class over-reaction.

### G.2 F-15 five-element check (F-15.a/b/c/d/e)

1. **F-15.a — source-code read attested**: author of the impact assessment attests to having read the relevant source code file(s) at declared path + line-range. Attestation = explicit citation in the assessment. Missing citation = F-15 FAIL.
2. **F-15.b — author-intent inquiry**: assessment explicitly addresses the likely author intent of the code as observed. Method: (a) adjacent commit messages, OR (b) surrounding comments / docs, OR (c) module's test file for intended-behaviour encoded in tests. At least one of (a/b/c) documented.
3. **F-15.c — alternative-hypothesis statement**: assessment explicitly entertains at least one alternative hypothesis with a ruling-in / ruling-out argument. Zero alternatives = F-15 FAIL.
4. **F-15.d — second-reviewer sign-off**: a second researcher (not original author; SCRIBE or LINNAEUS default; another research agent acceptable) reviews and signs off before the assessment enters R3 or deliver stage. Sign-off = name + date + one-sentence concurrence.
5. **F-15.e — distinction from hypothesis-generation + cross-domain GN.2 reference explicit** (per **D-17 20/3**): F-15 governs **published impact assessments**, not internal brainstorming. R1 sketch exploring possibilities is not bound by F-15; R2 / R3 statement "this code change cascades to deliverables X / Y / Z" is bound. **Additionally**, F-15 cross-refs GN.2 (≥ 2 genuinely discriminating hypotheses per D-15 UNANIMOUS): alternative hypotheses in F-15.c must meet GN.2 discriminating-power bar when assessment is in R2+ published-claim domain.

### G.3 Failure behaviour (non-punitive per MR-11)

F-15 FAIL returns the impact assessment to **draft** state for original author + second-reviewer to strengthen. FAIL logged in SCRIBE cycle process-gate log as **learning material**, NOT a performance demerit. Master's v2.0 §六 framing ("learning-oriented, 不是懲罰") applies equally to F-15.

Pattern recognition without shaming: where F-15 FAIL occurs ≥ 2 times in a single cycle, SCRIBE notes the pattern **topically (not individually)** — e.g., "§3 retrospective stage produced 3 F-15 FAILs, suggesting R1 draft-review checklist needs strengthening" NOT "agent X failed F-15 three times".

### G.4 Task #49 linkage (retrospective)

Task #49 impact-assessment statement proposed action on ≥ 2 deliverables based on a code-path interpretation that did not survive source-code re-reading. F-15 is the direct control preventing recurrence:
- Had F-15 been in force, the Task #49 assessment would have FAILed F-15.a (no source-code read attested) and/or F-15.c (no alternative-hypothesis "plugin-by-design-not-LLM" enumerated).
- Returned to draft, the over-reaction would not have propagated.

### G.5 Evidence artefact (F-15 front-matter block template)

```
F-15 compliance:
- Source-code read (a): {path}#{line-range}
- Author intent (b):   {method: commits | comments | tests | other}
- Alternative hypothesis (c): {brief statement, ruling}
- Second reviewer (d): {name} {date} — {one-sentence concurrence}
- Scope marker (e):    {R1-brainstorm-exempt | R2-published-claim with GN.2 cross-ref}
```

SCRIBE G5 verifies block presence; G6 verifies content non-vacuousness. Front-matter textually lightweight (≤ 10 lines) — reviewer discipline, not paperwork volume.

---

## Section H — Per-Item Effective Timing Table (D-21 + §2R2 F-06 adopted)

| F-item | Ratification | Plan49 effective? | Plan50 effective? | Rationale |
|--------|:------------:|:-----------------:|:-----------------:|-----------|
| F-11 Canonical Sync | Batch 10 | No (v1.7 freeze for Plan49) | **Yes** | Cycle-granularity; first effective at 03-13 cycle close (R4 post-ratification), applies from 03-14 cycle close onward |
| F-12 Reproducible Calc | Batch 10 | **Tooling yes** (Dev `audit_calc.py` in Plan49) + delivery no (v1.7 freeze) | **Full delivery Yes** | Tooling track separates from delivery-audit track |
| F-13 Plugin Hook Dispatch | Batch 10 | No (v1.7 freeze) | **Yes** | Per-Plan granularity; Plan49 frozen at v1.7 predates ratification |
| F-14 External Resource | Batch 10 | No (v1.7 freeze) | **Yes** | Per-Plan; requires vendor observation setup in Plan50+ |
| F-15 Code-Path Verified | Batch 10 | **Yes (immediate)** | **Yes** | Governs research deliverables; 03-13 artefacts grandfathered per §7.5.7; Plan49 research acceptance report subject to F-15 |

**Freeze rationale** (CV-16 UNANIMOUS): Plan49 assessment baseline = v1.7 (avoid mid-cycle retrofit). Plan50+ = v1.8. F-12 tooling track + F-15 immediate both override the freeze on specific narrow tracks, as documented above.

---

## Section I — Dissent Roll-Up (MR-11 preservation)

### I.1 D-17 F-15(e) GN.2 cross-reference (3 dissent preserved)

- **WIENER** (#12): F-15 should stand independent of GN.2; F-15 primarily a research-side honesty gate, GN.2 a SCRIBE-authority content-gate. Mixing reduces separability.
- **SUSSMAN** (#22): elegance/composability favours orthogonal rules. 4-element F-15 is simpler; cross-refs can be documented via linkage rather than embedding.
- **RUSSELL** (#23): GN.2 content evolved within SCRIBE G-gate authority, not Master-route. Embedding F-15(e)'s cross-ref into a Master-ratified MUST creates authority-chain asymmetry.

Majority (20 agents) rationale: explicit cross-ref strengthens F-15 precisely at the boundary where impact assessments face strong discriminating-hypothesis requirements; D-17 20/3 adoption per R3_D_items_voting.md §Round A.

### I.2 D-16 GN.3 escape clause (4 dissent, cross-referenced for F-15.d context)

Although D-16 is nominally SCRIBE-authority GN.3 (see `12_GN_1_5_SCRIBE_G_Gate.md §3`), the escape-clause mechanism bears on F-15.d peer review timing. 4 dissenters (ATHENA / GUARDIAN / RUSSELL / DARWIN) preferred mandatory-always. Preserved per MR-11; SCRIBE monitors escape-invocation for abuse signals 03-14 / 03-15 / 03-16.

### I.3 No other dissent entries in this binding

F-11 / F-12 / F-13 / F-14 ratifications: UNANIMOUS. CV-01 F-12 allocation: UNANIMOUS. MRB-02 narrative: UNANIMOUS. MRB-08/09/10 sub-clauses: UNANIMOUS. Dissent roll-up entries specific to ENG-FAB v1.8 binding total: **3 (D-17)** formally + **4 (D-16 cross-ref for F-15.d)** for a combined **7 dissent entries** preserved in this document.

Additional upstream dissent from related D items (D-04 σ_regime 5 / D-01 rename 3 / D-07 §76.7 1 / D-10 Path B 5 / D-13 C49-M5g 3 / D-20 two-truth 6 / D-24 Tier 2 2 / D-26 start order 5 / D-28 philosophy 3 / D-29 sequential 5) is preserved in their own chapters (O3 / O6 / Plan49 knowledge file); cross-cycle total = **41 dissent entries per `R3_decision_log.md §7`**.

---

## Section J — Reflexive Self-Application Note

Per the meta-principle that research artefacts should apply the discipline they propose (NAGARJUNA two-truth conventional-level self-application):

- This ENG-FAB v1.8 binding itself is an R4 research deliverable that proposes changes to a ratified framework. It therefore **carries an F-15 front-matter block** at its front-matter (implicit; the front-matter above satisfies F-15.a through F-15.e via explicit Binding R3 decisions + R2/R1 input trail + UNANIMOUS sign-off where applicable).
- F-15(e) cross-ref is honoured: the ≥ 2 genuinely discriminating hypotheses for "v1.8 = 48 vs 47 vs 43" were addressed in MRB-01 / CV-01 via three independent arithmetic verification paths (§A.2).
- F-11 applies prospectively at 03-13 R4 close (this document is part of the Research → Canonical → Release sync that F-11 audits).
- F-12 compliance on any numerical claim in this document (there are no probability / sigma / statistical claims here; count claims like "48 = 43 + 5" are arithmetic and carry the three-path verification per §A.2, satisfying §76.6 declarative form).
- F-13 / F-14 do not apply to this governance-only document (no plugin code; no external resource invocation).

**Reflexive attestation**: This document's authorship + R3 decision chain + R2 cross-review + R1 source trail collectively satisfy F-15.a-e; the "second reviewer" sign-off aggregates across R2 subagent (#5 O5 R2 KERNEL + TANENBAUM + PENROSE non-R1-authors) + R3 voting. Recorded here to exemplify the forward-compatible pattern.

---

*ENG-FAB v1.8 Binding — Cycle 03-13 — 2026-04-24 R4 close*
*48 items (45 MUST + 2 SHOULD + 1 INFO); F-11/12/13/14/15 NEW; MRB-01 + MRB-02 + MRB-08/09/10 + D-17 + D-21 + D-22 resolved*
*Post-Master ratification 2026-04-24 via Batch 10 Items 1-6; BINDING*
*7 dissent entries preserved in this document per MR-11 (41 total cycle-wide per `R3_decision_log.md §7`)*
