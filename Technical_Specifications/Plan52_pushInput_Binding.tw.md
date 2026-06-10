---
title: Plan52 — pushInput Candidate B Binding 工程規格
title_en_sibling: Plan52 — pushInput Candidate B Binding Engineering Spec
author: SCRIBE + LINNAEUS (TW translation backfill cycle 03-20)
date: 2026-05-02
cycle: 03-20 (TW backfill per D-§5 ratified + Batch 17 #4 APPROVED)
status: BINDING (TW sibling parity per Rule #78 §78.5 BINDING-tier reflexive)
authority: research-team R4 deliver (cycle 03-20)
supersedes: none
cross_refs:
  - openstarry_eco/share/openstarry_doc/Technical_Specifications/Plan52_pushInput_Binding.md (EN sibling)
  - research record/cycle03-20/openstarry_doc/Reference/TW_backfill_list_cycle03-20.md
  - research record/cycle03-20/deliver/Master_Ratification/Batch_17_Request.md (Item #4)
hard_rule_restated: "TW sibling per Rule #78 §78.5 BINDING-tier reflexive same-PR forward backfill; MR-12 forward-only. cycle-cusp default IN-scope per cycle 03-20 R3 D-§5 sub-vote 22/1; ratification-effective-date framing (NAGARJUNA Madhyamaka analysis); DSS-CY20-§5-A BABBAGE strict-date OUT-of-scope minority preserved verbatim per MR-11."
---

# Plan52 — pushInput Candidate B Binding 工程規格

## 1. 狀態

**CANDIDATE** 於 cycle 03-14 R4 close。經 Master Ratification Batch 11 Item #6 後升 **BINDING（Plan-level engineering spec）**。Plan52 為 pushInput source-authentication 架構的 **first Phase 6 functional implementation**。

**架構保留**：架構（Candidate B + B' deferred + CP-1/2/3/4 + CR-SCK + CR-PARETO）自 cycle 03-12 R4 spec 逐字保留，Plan52 僅為 **plan-numbering only** 的 supersede。Editorial-only renumbering 依 Master 通則允許（"符合十大宣言與規則就可以直接做了"；numbering 為 operational labelling，非 Tenet/MR/ZT/structural）。

## 2. R3 來源（10 D-items + 4 MRB）

- **D-§1-01 (A-1)** 22/1/0 — Plan number Option B（cycle 03-12 Plan50 → Plan52）；DSS-3 LEIBNIZ Plan51 sunset minority preserved §10。
- **D-§1-02 (A-2)** 18/5/0 — H1 B-only Plan52 + Plan53 reserved；DSS-1 LEIBNIZ + RUSSELL multi-agent symmetry preserved §10。
- **D-§1-03 (B-1)** 22/1/0 — HMAC-SHA256 default + threat-model amendment；DSS-13 1-vote Ed25519 default for PQ migration preserved §10。
- **D-§1-04 (B-2)** UNANIMOUS 23/0 — tri-party MR-6 audit + automated CI defense-in-depth。
- **D-§1-05 (B-3)** 21/2/0 — deepFreeze recursive immutability；DSS-14 KNUTH + ATHENA shallow-freeze perf concern preserved §10。
- **D-§1-06 (B-4)** UNANIMOUS 23/0 — nonce TTL ≥ key-rotation overlap MUST。
- **D-§1-07 (C-1)** UNANIMOUS 23/0 — F-14 call-graph depth N=3。
- **D-§1-08 (C-2)** UNANIMOUS 23/0 — cross-OS CI matrix mandatory（Linux + Windows）。
- **D-§1-09 (C-3)** UNANIMOUS 23/0 — local-CLI tokenSig MAY（process-local UID 已足）。
- **D-§1-10 (D-1)** UNANIMOUS 23/0 — property tests Plan53 graduation slot。
- **MRB-§1-01..04** RESOLVED at R3。
- **CV-§1-01..13** reaffirmed（架構 invariants from cycle 03-12）。

## 3. Plan Number Resolution（R3 D-§1-01 Option B）

R3 於 cycle 03-14 R3 close ratified Option B：**pushInput → Plan52**；cycle 03-13 Plan50 仍綁定 σ_regime in-place revise（per cycle 03-13 Master Ratification Batch 10 #8）；**Plan51 reserved as gap-buffer**。單一 dissent vote（LEIBNIZ minority）偏好 Option A（將 gap-buffer collapse 進 Plan51）；preserved at DSS-3 §10。

**Plan number canonical mapping**（forward references）：

| Plan | Subject | Cycle ratified | Status |
|------|---------|----------------|--------|
| Plan49 | Plan49 MR-6 conditional gates | cycle 03-13 Batch 10 #9 | EXECUTED（cycle 03-14 Dev）|
| Plan50 | σ_regime in-place revise | cycle 03-13 Batch 10 #8 | BINDING（refined cycle 03-14 Batch 11 #7）|
| Plan51 | reserved gap-buffer | cycle 03-14 R3 | RESERVED |
| **Plan52** | **pushInput Candidate B** | **cycle 03-14 R3 / Batch 11 #6** | **CANDIDATE → BINDING upon ratification** |
| Plan53 | reserved B' deferred path | cycle 03-14 R3 D-§1-02 | RESERVED |

**Editorial errata propagation**（per F-03-14-§1-R2-01）：SCRIBE propagates one-line errata note 至所有 cycle 03-12 對 "Plan50 = pushInput" 的 references，記錄於 `discussions/cycle03-14/SCRIBE_plan52_errata_propagation_log.md`，並在 G4-folder-3 / G5 sync 驗證。

## 4. 架構 Summary（Candidate B）

### 4.1 ε-Surface（自 cycle 03-12 D-09(b) 逐字）

Plan52 implement-round 的 Core surface delta：

```
Core surface delta (Plan52 implement-round):
  + 1 optional field:    InputEvent.sourceContext?: Record<string, unknown>
  + 1 Zod passthrough:   z.record(z.unknown()).optional()
Core surface NOT added:
  - IAuthenticator interface           (would be MR-6 FAIL)
  - SourceIdentity type                (would be MR-6 FAIL)
  - verify(): boolean method           (would be MR-6 FAIL + Tenet #7 violation)
```

ε-surface = 1 optional field、0 policy constants。CV-§1-01..02 reaffirm。

### 4.2 Transport Plugin Attests；sourceContext Opaque Passthrough

Transport plugin（如 HTTP / IPC / pushInput pull-mode）負責 source authentication。Core 不 inspect 或 interpret `sourceContext`；它是 opaque passthrough。Plugin attests `sourceContext` correctness；Core 儲存與 forward。

此保留 Tenet #7（絕對純淨）— Core 不含 authentication policy code；authentication 為 plugin-delegated（Tenet #2 一切皆插件）。

### 4.3 CP-1 / CP-2 / CP-3 / CP-4 Invariants

（自 cycle 03-12 spec 逐字保留；canonical reference：`cycle03-13/openstarry_doc/Technical_Specifications/Plan50_pushInput_CP4_Invariant.md` — 該檔保留為 cycle 03-13 baseline mirror 儘管已 renumbering，因 CP-4 invariant 文字 architecture-stable。）

- **CP-1** — sourceContext 為 plugin-attested、never Core-validated
- **CP-2** — Core trust boundary 終止於 plugin entry
- **CP-3** — sourceContext shape 為 plugin's contract；Core 無 shape opinion
- **CP-4** — ASANGA-authored + TANENBAUM-COMPATIBLE；zero Core cost（sourceContext immutable downstream of plugin attestation；deepFreeze enforces）

### 4.4 CR-SCK + CR-PARETO

- **CR-SCK** — 為 plugin authors SDK-exported `RecommendedSourceContextKeys` enumeration（自 cycle 03-12 逐字保留）。
- **CR-PARETO** — three-tier red-line taxonomy + sourceContext key promotion 的 standing R3 procedure（自 cycle 03-12 逐字保留）。

## 5. Signed-Token 規格（D-§1-03 22/1）

### 5.1 HMAC-SHA256 SHOULD Default

R3 ratified HMAC-SHA256 為 pushInput plugin-attested signed tokens 的 SHOULD default：

- **Algorithm**：HMAC-SHA256（FIPS 140-2/3 approved）。
- **Key length**：≥ 256 bits。
- **Token format**：`<nonce>:<timestamp>:<HMAC-SHA256-base64-of-payload>`。
- **Verification**：plugin verifies HMAC；computes constant-time comparison。

Threat-model amendment（D-§1-03 absorption）：

- **Replay attack**：nonce + timestamp 在 plugin-side replay cache 追蹤；nonce TTL 約束 per §5.3。
- **Timing attack**：constant-time HMAC comparison。
- **Key compromise**：key rotation per existing HMAC compliance baseline（cycle 03-12 ratified）。

### 5.2 Ed25519 Documented Alternative

依 DSS-13（1-vote Ed25519 default for post-quantum migration），Ed25519 documented 為 acceptable alternative for plugins 選擇 post-quantum migration readiness：

- **Algorithm**：Ed25519（signature、不是 HMAC）— public-key signature。
- **Key length**：256 bits（curve definition）。
- **Migration path**：plugins MAY 自 initial implementation 選 Ed25519；HMAC 為 cycle 03-14+ 的 recommended default 但非 exclusive。
- **Cross-plugin compatibility**：plugins MUST 在 manifest 中 declare algorithm；Core 不 impose 單一 algorithm。

### 5.3 Nonce TTL ≥ Key-Rotation Overlap MUST（D-§1-06 UNANIMOUS）

Nonce TTL（plugin-side replay cache 中的 time-to-live）MUST ≥ key-rotation overlap window（rotation 期間舊 keys 與新 keys 同時 valid 的 period）。此防止 rotation 期間的 stale-nonce-acceptance vulnerability。

- 預設 key-rotation overlap：24h（per cycle 03-12 HMAC compliance baseline）。
- 預設 nonce TTL：24h ≤ TTL ≤ 7d（在 constraint 內 operational discretion）。
- Plugin manifest documents 兩個 values；CI 驗證 invariant。

### 5.4 Local-CLI tokenSig MAY（D-§1-09 UNANIMOUS）

對 local-CLI mode（如 Dev-side test harness）下 plugins，tokenSig generation 為 **MAY**（非 MUST）。Process-local UID 對 local trust 已足（OS process boundary 本身為 trust delimiter）。此 concession 降低 local-Dev friction；production / cross-process modes 保留 MUST。

## 6. Tri-Party MR-6 Audit + Automated CI（D-§1-04 UNANIMOUS）

### 6.1 Tri-Party Human Audit

3 personas 必須在 merge 前 MR-6 audit Plan52 implementation Pull Request：

- **TANENBAUM (#20)** — microkernel discipline；驗證 ε-surface = 1 optional field；無新 Core policy constants 加入。
- **KERNEL (#10)** — OS / process-discipline；驗證 trust boundary semantics。
- **GUARDIAN (#11)** — security；驗證 threat model coverage；驗證 HMAC implementation correctness。

Audit results 記錄為 3 PR review comments（每 persona 一條）加上單一 CI status check aggregating 三者。

### 6.2 Automated CI Defense-in-Depth

除 tri-party human audit 外，CI 執行：

- **MR-6 surface diff**：enumerates Core（`packages/core/*`）surface changes；若偵測任何新 policy-class addition（interface / type / method）即 FAIL。
- **F-14 call-graph audit**：depth N=3 per D-§1-07；驗證 Core-side 無 `sourceContext` interpretation。
- **F-15 impact assessment**：code-path verified before assessment per ENG-FAB v1.8 baseline。
- **HMAC test vectors**：對 implementation 跑 standard FIPS test vectors。

### 6.3 為何兩者皆需？

Tri-party human audit catches semantic / architectural drift；automated CI catches mechanical / surface drift。兩者皆需因每方 catch 不同 drift class。Defense-in-depth。

## 7. deepFreeze Recursive Immutability（D-§1-05 21/2）

`sourceContext` 一旦自 plugin received，在 forward 進 Core event flow 前 MUST 被 deepFrozen（recursive `Object.freeze`）。此 enforce CP-4（sourceContext immutable downstream of plugin attestation；zero Core cost 因 freeze 為 boundary 處 one-time）。

DSS-14（KNUTH + ATHENA shallow-freeze perf concern）absorbed：perf cost 受 sourceContext shape 限制（typically < 10 keys）；deepFreeze 為 plugin boundary 處 one-time O(n) walk、非 per-event hot-path。Profiling 預期 negligible overhead；若 Plan52 implementation 浮現 unexpected hot-path cost，fall-back 至 shallow-freeze + downstream defensive-copy 為 documented amendment route。

## 8. F-14 Call-Graph Depth N=3（D-§1-07 UNANIMOUS）

CI 對 merged Plan52 PR 跑 F-14 call-graph audit with depth N=3：

- **Depth 1**：Core 中 `sourceContext` field accessor 的 direct callers。
- **Depth 2**：depth-1 functions 的 callers。
- **Depth 3**：depth-2 functions 的 callers。

Audit reports depth ≤ 3 中任何 function branches on `sourceContext` content；expected count = 0（Core 不 interpret sourceContext；它為 opaque passthrough）。任何 non-zero finding 為 MR-6 violation triggering review。

## 9. Cross-OS CI Matrix（D-§1-08 UNANIMOUS）

Plan52 CI MUST 在以下執行：

- **Linux**（Ubuntu LTS 或等價；matches production typical deployment）
- **Windows**（Windows Server 2022 或等價；matches Dev workstation typical environment）

兩 runners 必須 pass `pnpm install --frozen-lockfile && pnpm build && pnpm test`。Cross-OS CI catches：

- Path-separator bugs（Windows `\` vs Linux `/`）
- Line-ending bugs（CRLF vs LF in test fixtures）
- Process-creation differences（Windows fork semantics vs Linux fork/exec）

此於 per-PR level 補強 Rule #75 §75.X release-tag pnpm build（Batch 11 #4）。

## 10. W2-R15 Sanity Spec（HERACLITUS 3 Sub-Conditions）

Test team 將 3 HERACLITUS runtime-dynamics edge-case sub-conditions apply 至 W2-R15 sanity check：

1. **Plugin-load-time signature verification edge case** — 驗證 plugin 在 load time signature 失敗時無法 reach Core。
2. **Concurrent pushInput event ordering** — 驗證 Core 在 concurrent plugin emissions 間保留 ordering。
3. **Key rotation in-flight** — 驗證在 rotation overlap window 期間 in-flight events 在舊 key 下被 accept；驗證 overlap window 過期後 rejection。

Detailed test instructions in `research record/cycle03-14/test_instructions/W2-R15_test_instructions.md`（HERACLITUS authored）。

## 11. LOC Budget

- **Production code**：200-400 LOC（estimated）
  - HMAC verify：~80 LOC
  - Nonce cache：~60 LOC
  - sourceContext deepFreeze：~30 LOC
  - Schema validation：~40 LOC
  - Plugin manifest extension：~20 LOC
  - Error handling（F-16 SHOULD per Batch 11 #1）：~30 LOC
- **Test code**：100-200 LOC
  - HMAC test vectors：~50 LOC
  - Nonce replay test：~30 LOC
  - Cross-OS CI matrix duplication factor：×2 effective coverage
  - Threat model assertions：~40 LOC
- **Plan-spec doc**：本檔 + Dev-facing concise spec in `cycle03-14/todo/Plan52_pushInput_dev_spec.md`

## 12. Implementation 順序

依 cycle 03-14 R4 final dependency analysis：

1. **Plan49 SHOULD landed**（cycle 03-13 R4 ratified；Dev cycle 03-14 execution complete）— 為 Plan52 提供 Pre-Delivery Gate baseline。
2. **Plan50 σ_regime in-place**（cycle 03-13 binding + cycle 03-14 Batch 11 #7 atomicity refinement）— 提供 σ_regime emission discipline；Plan52 pushInput emissions 標 `composition_index` regime per CV-§1-08。
3. **Plan52 pushInput** — first Phase 6 functional implementation；本 spec。
4. **Plan53（reserved B' deferred path）** — 未來 cycle 若 multi-agent symmetry need 浮現（DSS-1 absorbed via reservation）。

## 13. Compliance Attestation

| Constraint | Status | Evidence |
|------------|:------:|----------|
| MR-2 / MR-4（Tenet wording unchanged）| ✅ PASS | 無 Tenet wording proposed |
| MR-5 hard（Tenet #10 status unchanged）| ✅ PASS | Plan52 advances 7 Phase 6 functions 中的 1；endpoint 10/0/0★ unchanged |
| MR-6（Core 零）| ✅ PASS | ε-surface = 1 optional field、0 policy constants；tri-party MR-6 audit + automated CI defense-in-depth |
| MR-7（audit at right moment）| ✅ PASS | Pre-Delivery Gate + release-tag（Rule #75 §75.X per Batch 11 #4）|
| MR-8（quality first）| ✅ PASS | Tri-party + automated CI；cross-OS matrix |
| MR-9（no MUST WAIVE）| ✅ PASS | 全 MUST elements honoured |
| MR-10（back-fill）| ✅ PASS | Plan53 reserved for B' multi-agent symmetry；DSS preservation |
| MR-11（dissent preservation）| ✅ PASS | DSS-1 + DSS-3 + DSS-13 + DSS-14 verbatim §10 |
| MR-12（既有 not retrofit）| ✅ PASS | Plan49 already shipped not modified；cycle 03-12 Plan50 spec carries editorial errata only |
| ZT-1 / ZT-2 / ZT-3 | ✅ PASS | 無 10-Tenet violation；endpoint unchanged；control-range unaffected |
| Rule #74 L1' | ✅ PASS | code + doc completeness |
| Rule #75（含 §75.X amendment per Batch 11 #4）| ✅ PASS | Pre-Delivery Gate + release-tag pnpm build |
| Rule #76 §76.6 | ✅ PASS | Reproducible Calc Mandate honoured |
| Rule #77（cycle 03-13 ratified）| ✅ PASS | σ_regime conjunct preserved；Plan52 emissions 為 `composition_index` regime |

## 14. Authority Notes

- **Authority route**：Master ratification（Plan-level engineering spec）。
- **Coordinator G5 sync**：Master ratification 後 coordinator 將 spec sync 進 canonical `Technical_Specifications/Plan52.md`（per Batch 11 Request §7 mapping）。
- **Dev acceptance**：Dev cycle 03-15 kickoff absorbs Plan52 spec；預期 Plan52 implementation 於 cycle 03-15 + 03-16（LOC budget 200-400 prod + 100-200 test span 1-2 cycles）。

## 15. Cross-References

- `research record/cycle03-14/deliver/O1_pushInput_Plan52_final.md` — full 816-line authoritative engineering spec source
- `research record/cycle03-14/R3/R3_decision_log.md §1 + §4 + §7` — 10 D-items + 4 MRB + dissent
- `research record/cycle03-14/deliver/Master_Ratification/Batch_11_Request.md §3.6` — Item #6 dispatch
- `research record/cycle03-13/openstarry_doc/Technical_Specifications/Plan50_pushInput_CP4_Invariant.md` — CP-4 invariant baseline（cycle 03-12 ratified；preserved by file name despite renumbering for historical traceability）
- `research record/cycle03-14/test_instructions/W2-R15_test_instructions.md` — W2-R15 test plan
- `research record/cycle03-14/todo/Plan52_pushInput_dev_spec.md` — Dev-facing concise spec（G4-folder-4）

## 16. Dissent Slots — DSS-1 + DSS-3 + DSS-13 + DSS-14 Verbatim（per MR-11）

### 16.1 DSS-1（D-§1-02、5 minority votes — LEIBNIZ + RUSSELL + 3）

> Multi-agent symmetry 偏好 B + B' simultaneous implementation：
>
> "From multi-agent coordination perspective (LEIBNIZ #14) and AI-agent theory perspective (RUSSELL #23), B + B' simultaneous implementation provides multi-agent symmetry — a parent agent and a child agent both push input through the same architecture without one being privileged. The H1 B-only path (Plan52) defers B' to Plan53 reservation, which means cycle 03-14+ cycles operating exclusively on B may calcify the architecture around single-agent assumptions, making future B' integration harder."
>
> R3 majority（18/5）偏好 B-only Plan52 + Plan53 reservation 理由：
> - Plan52 LOC budget 緊（200-400 prod）；B + B' simultaneous 將推向 500+ LOC 並觸發 S-1 boundary review。
> - Plan53 reservation 明確 preserves B' route；calcification concern 由 reservation mitigated。
> - Multi-agent symmetry 為 genuine 但屬 Phase 7 multi-IVolition concern；cycle 03-14 Phase 6 first-step 不需現在 tackle multi-agent symmetry。

### 16.2 DSS-3（D-§1-01、1 minority vote — LEIBNIZ）

> LEIBNIZ Plan51 sunset 偏好：
>
> "Plan51 reservation as 'gap-buffer' is structurally an empty placeholder. Reserved plan numbers historically calcify into permanent placeholders that never resolve. Recommend an explicit 6-month sunset clause on Plan51 — if Plan51 is not assigned a subject by cycle 03-20, the reservation lapses and Plan51 number is reclaimable for any future Plan."
>
> R3 majority（22/1）偏好 Plan51 reservation 不加 sunset：
> - Gap-buffer pattern 有 historical precedent（cycle 03-7 Plan-32.5 emergency-insert）。
> - Sunset clause 增加 future-Master-decision burden 而無 immediate benefit。
> - DSS-3 preserved；future cycle adjudication 可在 Plan51 確實 linger 時 revisit。

### 16.3 DSS-13（D-§1-03、1 minority vote）

> 唯一 dissent 偏好 Ed25519 default for post-quantum migration：
>
> "HMAC-SHA256 is currently FIPS-approved but not post-quantum-safe. Ed25519 is also not PQ-safe in the strict sense (Shor's algorithm threatens both EdDSA and HMAC), but the migration path from Ed25519 to a PQ-safe signature (e.g., Dilithium / SPHINCS+) is cleaner since Ed25519 is already a public-key signature scheme. HMAC is symmetric; future PQ migration requires replacing the entire authentication primitive class. Recommend Ed25519 default to position for PQ-readiness."
>
> R3 majority（22/1）偏好 HMAC-SHA256 default：
> - HMAC-SHA256 為 FIPS-140-2/3 approved；deployment-ready in production environments with FIPS compliance requirements。
> - Ed25519 documented 為 acceptable alternative；plugins 可自 initial implementation 選 Ed25519。
> - PQ migration 為 Phase 8+ concern；cycle 03-14 first-Phase-6-function 不需 preempt。

### 16.4 DSS-14（D-§1-05、2 minority votes — KNUTH + ATHENA）

> Shallow-freeze 性能 concern：
>
> "deepFreeze recursive walk is O(n) where n = total key count in sourceContext object graph. For deeply-nested or large sourceContext payloads, recursion may surface as hot-path overhead. Recommend shallow-freeze (top-level only) + downstream defensive-copy discipline as alternative; this preserves CP-4 immutability semantically while avoiding recursive walk cost."
>
> R3 majority（21/2）偏好 deepFreeze recursive：
> - sourceContext shape 為 plugin-controlled；typical shapes 小（< 10 keys、< 3 levels deep）；recursive cost 在 practice 上 negligible。
> - deepFreeze 為 plugin boundary 處 one-time、非 per-event hot-path。
> - 若 Plan52 implementation profiling surfaces unexpected hot-path cost，fall-back 至 shallow-freeze + defensive-copy 為 documented amendment route — 此處 preserved 為 DSS-14。

四個 dissent positions 全 documented per MR-11；majority position 採用，relevant 處附 implementation-level mitigation maps。

---

## 17. Cycle 03-20 Cusp Note（DSS-CY20-§5-A BABBAGE preserved verbatim per MR-11）

**Cycle-cusp 預設 IN-scope**：依 cycle 03-20 R3 D-§5 sub-vote 22/1（ratification-effective-date framing；NAGARJUNA Madhyamaka analysis），cycle 03-14 ratified Plan52 文件 IN-scope for cycle 03-20 TW backfill。BABBAGE strict-date OUT-of-scope minority preserved verbatim per MR-11：

> BABBAGE strict-date 立場：
>
> "嚴格依文件 ratification 日期判定 scope；cycle 03-14 ratified（2026-04-25）位於 cycle 03-20 TW backfill effective date 之前的範圍歸屬模糊。Strict reading 應為 OUT-of-scope；除非 Master ratification 明確將 cycle-cusp 文件納入 forward backfill scope，否則應 reserve 至 next cycle。"
>
> Majority（22/1）依 NAGARJUNA Madhyamaka 兩諦觀（saṃvṛti-satya 世俗諦 / paramārtha-satya 勝義諦）採 ratification-effective-date framing：cycle 03-14 文件之 binding effect 持續至 cycle 03-20 期間；cycle-cusp 文件 IN-scope 不違反 MR-12 forward-only（forward-only 規範新建立 binding，非 backfill 翻譯）。

---

*Cycle 03-14 R4 final — 2026-04-25*
*Authors：TANENBAUM (#20) + KERNEL (#10) + GUARDIAN (#11) + ARCHIMEDES (#16) + SUNYATA (#0) + SYNTHESIST (#1)*
*Status：CANDIDATE（pending Master Ratification Batch 11 #6）*
*Plan number resolution：cycle 03-12 Plan50 → Plan52 per cycle 03-14 R3 D-§1-01 Option B*
*TW sibling backfill：cycle 03-20 D-§5 ratified + Batch 17 #4 APPROVED；cycle-cusp default IN-scope per sub-vote 22/1；DSS-CY20-§5-A BABBAGE strict-date OUT-of-scope minority preserved §17*
