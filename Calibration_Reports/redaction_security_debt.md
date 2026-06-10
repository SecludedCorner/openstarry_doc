# Redaction Security Debt Register

**Status**: BINDING canonical (cycle 03-19 R3 D-§4 ratified 23/0 UNANIMOUS; pending Master Ratification Batch 16 #10)
**Authority**: SCRIBE-internal authority (Reference/15 Security Retroactive Precedent first concrete application)
**Cycle of origin**: 03-19
**Reference**: `Reference/15_Security_Retroactive_Precedent.md` (cycle 03-18 Master directive 2026-04-30)

---

## §1 Purpose

Per cycle 03-18 Master directive 2026-04-30 (γ) γ retrofit + Reference/15 Security Retroactive Precedent: ~15-18 actionable plugins MUST retrofit to uniform redaction format (per cycle 03-18 D-§1-R2-B `<redacted-volition-payload len:NN first4:abcd>` N=4 + category-aware) by **cycle 03-19 R4 close hard sunset**.

This register **transparently lists** retrofit progress and known security debt per Master directive 2026-04-30 §4.2.

---

## §2 Register Schema (13 fields per entry)

| Field | Description |
|-------|-------------|
| `plugin_id` | Plugin package identifier |
| `family` | Family group (e.g. provider / context / guide / comm / auditor) |
| `sensitivity_class` | HIGH/MED/LOW per category-aware redaction |
| `pre_format` | Format before retrofit |
| `post_format` | Format after retrofit (uniform target) |
| `retrofit_cycle` | Cycle when retrofit applied |
| `retrofit_pr` | PR number / commit hash |
| `tri_party_audit` | TANENBAUM + KERNEL + GUARDIAN sign-off status |
| `f15_lint_status` | F-15 v3 linter pass/fail |
| `cross_os_ci` | Linux + Windows CI status |
| `w2r22_regression` | W2-R22 regression test status |
| `dissent_inheritance` | DSS preservation tracking (e.g. DSS-CY18-02 KERNEL N=8) |
| `notes` | Free-form context |

**Append-only discipline**: entries are added; existing entries amended via subsequent rows with timestamp annotation. NO ENTRY REMOVAL (preserves audit trail per Reference/15).

---

## §3 Plugin Retrofit Status (~15-18 actionable)

### §3.1 IN-SCOPE plugins (10 families)

| # | plugin_id | family | sensitivity | retrofit_cycle | status |
|:-:|-----------|--------|:-----------:|:--------------:|:------:|
| 1 | provider-openai-chat | provider-* | HIGH | 03-19 | PENDING DEV |
| 2 | provider-anthropic-chat | provider-* | HIGH | 03-19 | PENDING DEV |
| 3 | provider-google-chat | provider-* | HIGH | 03-19 | PENDING DEV |
| 4 | provider-cohere-chat | provider-* | HIGH | 03-19 | PENDING DEV |
| 5 | provider-mistral-chat | provider-* | HIGH | 03-19 | PENDING DEV |
| 6 | provider-deepseek-chat | provider-* | HIGH | 03-19 | PENDING DEV |
| 7 | provider-local-llm | provider-* | HIGH | 03-19 | PENDING DEV |
| 8 | provider-test-mock | provider-* | LOW | 03-19 | PENDING DEV |
| 9 | guide-character-init | guide-* | MED | 03-19 | PENDING DEV |
| 10 | context-sliding-window | context-* | MED | 03-19 | PENDING DEV |
| 11 | context-summarization | context-* | MED | 03-19 | PENDING DEV |
| 12 | context-state-extraction | context-* | MED | 03-19 | PENDING DEV |
| 13 | context-vector-store | context-* | HIGH | 03-19 | PENDING DEV |
| 14 | comm-mcp-bridge | comm-* | MED | 03-19 | PENDING DEV |
| 15 | comm-mesh-protocol | comm-* | MED | 03-19 | PENDING DEV |
| 16 | auditor-trail | auditor-* | HIGH | 03-19 | PENDING DEV |
| 17 | spc-monitor-control / spc-monitor-westgard / spc-monitor-l2-l3 | spc-monitor (3) | LOW | 03-19 | PENDING DEV |
| 18 | confirmation-gate-standard | confirmation-gate | HIGH | 03-19 | PENDING DEV |

**Total IN-SCOPE**: 18 actionable plugins (deduplicated to ~15-18 via shared `redact_volition_payload` helper per BABBAGE R2 §2.4).

### §3.2 Shared helper deduplication

Per cycle 03-19 R3 D-§4 + BABBAGE R2: shared helper `redact_volition_payload` lives at Plan52 `sourceContext` utility module. Provider-* family (8) + context-* family (4) all consume helper centrally → effective retrofit touched-plugin count = 15-18 (not 24).

### §3.3 OUT-of-scope plugins (~20 plugins; no sensitive data)

| family | plugins | rationale |
|--------|---------|-----------|
| `static-*` | static-fs-loader / static-config-loader | reads structured config; no payload redaction needed |
| `fs-*` | fs-readwrite / fs-snapshot | path-metadata only; not via Plan52 sourceContext |
| `mcp-*` | mcp-server / mcp-client | protocol-level; redaction at app layer |
| `devtools-*` | devtools-inspector / devtools-tracer | dev-only; no production runtime |
| `policy-*` | policy-evaluator | rule evaluation; no payload data |
| `lifecycle-*` | lifecycle-monitor | metadata only |

---

## §4 6 道關卡 Non-bypassable Enforcement

Per cycle 03-19 R3 D-§4-R2-06 23/0 UNANIMOUS:
1. **Rule #75 §75.X gate** — `pnpm install --frozen-lockfile && pnpm build && pnpm test && pnpm test:purity` 全 0 exit
2. **Tri-party MR-6 audit** — TANENBAUM + KERNEL + GUARDIAN sign-off
3. **F-15 v3 linter** — false-positive 0% / false-negative 0% target
4. **Cross-OS CI matrix** — Linux + Windows persistent
5. **W2-R22 verification** — regression test
6. **R3 review** — cycle 03-19 R3 D-§4 (this round)

**Any waive request escalates to Master prerogative**.

---

## §5 Sunset Mechanism (cycle 03-19 R4 close hard)

Per Master directive 2026-04-30 §4.3 BINDING:
- **Cycle 03-19 R4 close** (2026-05-01) = hard sunset deadline
- Any IN-SCOPE plugin not retrofitted → **DEPRECATED label** + **WARN log** + **replacement schedule**
- DEPRECATED entries require `capability_orphan_status` attestation (per RUSSELL R2 amendment) to prevent silent capability degradation

---

## §6 Scope-strict Reject-on-violation

Per cycle 03-19 R3 D-§4-R2-08 23/0 UNANIMOUS:

**Strictly bounded to redaction format**. Plugin logic changes OUT-OF-SCOPE. Any retrofit PR introducing logic changes → **REJECT at R2/R3** + **return to Dev for scope split**.

---

## §7 Dissent Preservation per MR-11

DSS preservation tracking:
- **DSS-CY18-02** (KERNEL N=8 hex preference; cycle 03-18 carryover) — preserved verbatim across all 18 retrofit entries; non-blocking; N=4 alphanumeric ratified per cycle 03-18 D-§1-R2-B 22/1
- **DSS-CY19-§1-C** (KERNEL N=8 carryover; cycle 03-19 D-§1-R2-B reaffirm 22/1) — preserved

---

*Redaction Security Debt Register — BINDING canonical — cycle 03-19 R3 D-§4 ratified 23/0 — 2026-05-01*
*First concrete application of Reference/15 Security Retroactive Precedent*
*Master Ratification Batch 16 #10 dispatch ready*
