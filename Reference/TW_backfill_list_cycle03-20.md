# TW Sibling Backfill List — Cycle 03-20 R4

**Status**: BINDING (cycle 03-20 R3 D-§5 ratified 23/0; pending Master Ratification Batch 17 #4)
**Authority**: Master Ratification (Batch 17 dispatch 2026-05-02)
**Cycle**: 03-20 (consolidation lighter; v0.54.1-alpha doc-only patch)
**Purpose**: Per Rule #78 §78.5 BINDING-tier reflexive same-PR forward backfill; canonical 265 → 274
**Source**: cycle 03-20 R3 D-§5 + R2 §5 (LINNAEUS + SCRIBE + ATHENA + NAGARJUNA + ASANGA + KERNEL)
**MR-12 forward-only**: 既有 TW siblings 不 retrofit; only ADD missing siblings via same-PR forward

---

## §1 9 Missing TW Siblings (8 MUST + 1 SHOULD 24h grace)

### Tier-A: Reference/ (5 MUST same-PR)

| # | Source EN doc | TW sibling target | Tier | Cycle ratified |
|:-:|---------------|-------------------|:----:|:--------------:|
| 1 | `Reference/12_*.md` | `Reference/12_*.tw.md` | MUST | cycle 03-15+ |
| 2 | `Reference/13_Plugin_Loader_Cycle03_17_Evaluation_Criteria.md` | `Reference/13_Plugin_Loader_Cycle03_17_Evaluation_Criteria.tw.md` | MUST | cycle 03-16 R3 |
| 3 | `Reference/14_*.md` | `Reference/14_*.tw.md` | MUST | cycle 03-17 R3 |
| 4 | `Reference/15_Security_Retroactive_Precedent.md` | `Reference/15_Security_Retroactive_Precedent.tw.md` | MUST | cycle 03-18 R3 + Master directive 2026-04-30 |
| 5 | `Reference/16_R4_Folder_Convention_Discipline.md` | `Reference/16_R4_Folder_Convention_Discipline.tw.md` | MUST | Master directive 2026-05-01 |

### Tier-B: Technical_Specifications/ (2 MUST same-PR)

| # | Source EN doc | TW sibling target | Tier | Cycle ratified |
|:-:|---------------|-------------------|:----:|:--------------:|
| 6 | `Technical_Specifications/Plan51_*_Binding.md` | `Technical_Specifications/Plan51_*_Binding.tw.md` | MUST | cycle 03-15 R3 |
| 7 | `Technical_Specifications/Plan52_*_Binding.md` | `Technical_Specifications/Plan52_*_Binding.tw.md` | MUST (cycle-cusp default IN-scope per cycle 03-20 R3 D-§5 sub-vote 22/1) | cycle 03-15 R3 (cycle-cusp; ratification-effective-date framing per NAGARJUNA Madhyamaka analysis) |

### Tier-C: Research_Methodology/ (1 SHOULD 24h grace)

| # | Source EN doc | TW sibling target | Tier | Cycle ratified |
|:-:|---------------|-------------------|:----:|:--------------:|
| 8 | `Research_Methodology/15_ENG_FAB_v1.9_F_16_StructuredError.md` | `Research_Methodology/15_ENG_FAB_v1.9_F_16_StructuredError.tw.md` | SHOULD 24h grace (CANDIDATE status; per F-15 v3 §3.2 SHOULD-tier discipline) | cycle 03-15+ candidate |

### Tier-D: Calibration_Reports/ (1 MUST same-PR)

| # | Source EN doc | TW sibling target | Tier | Cycle ratified |
|:-:|---------------|-------------------|:----:|:--------------:|
| 9 | `Calibration_Reports/redaction_security_debt.md` | `Calibration_Reports/redaction_security_debt.tw.md` | MUST | cycle 03-19 R3 D-§4 + Reference/15 first concrete application |

**Total**: 9 backfill items (8 MUST + 1 SHOULD); ~2,615-3,030 TW LOC backfill (per ATHENA spot-check + KERNEL technical-term verbatim preservation discipline)

---

## §2 Translation Discipline

Per Rule #78 §78.4 + cycle 03-20 R3 LINNAEUS+ASANGA+KERNEL R2 cross-review (UNANIMOUS endorsement):

- **Three-layer translation strategy** (Sanskrit anchor + 漢字 canonical + technical-term verbatim):
  - Sanskrit anchors: vasanā / vijñāna / saṃvṛti-satya / paramārtha-satya / pratītyasamutpāda preserved
  - 漢字 canonical: 習氣 / 識 / 世俗諦 / 勝義諦 / 緣起
  - Technical terms (HMAC-SHA256 / Ed25519 / sourceContext / ε-surface / nonce / Wilson CI-LB / σ_regime / Plan52/54/56/57 etc): VERBATIM preserved (not translated)

- **Sibling-naming convention**: `<basename>.tw.md` (per cycle 03-15 §6.3 forward-only)

- **Front-matter parity**: 5/5 fields match EN sibling (title 中譯 / author 同 / date 同 / cycle 同 / status 同)

- **Cross-reference parity**: cross_refs anchor to BOTH EN and TW siblings (paired references)

---

## §3 Compliance

| Constraint | Status |
|------------|:------:|
| MR-12 forward-only | ✅ same-PR forward; 既有 not retrofit |
| MR-11 dissent preservation | ✅ DSS-CY20-§5-A (BABBAGE OUT-of-scope strict-date) preserved verbatim non-blocking |
| Rule #78 §78.5 BINDING-tier reflexive | ✅ all 8 MUST same-PR + 1 SHOULD 24h grace |
| F-15 v3 schema | ✅ |
| ATHENA spot-check exemplar | ✅ Plan57 4/4 quality PASS confirmed |

---

*TW Sibling Backfill List — Cycle 03-20 R4 — 2026-05-02*
*Master Ratification Batch 17 #4 dispatch ready*
*Coordinator G5 sync stage executes actual backfill with EN-anchored content + ATHENA review*
*Canonical 265 → 274 (+9 TW siblings)*
