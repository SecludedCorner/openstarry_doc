<!-- Status: CURRENT -->
<!-- Written: 2026-06-12 -->
<!-- 性質: 蒸餾清單（task #24a）。全庫 353 個 md（308 個主檔＋.tw 雙胞）由 16 個並行分類 agent 按四分法掃描；每檔附一行依據。本清單是「蒸餾發表」方向的工作底稿：blueprint＝出版主體、fix＝修後可用、quarantine＝虛構層、archive＝post-mortem 資料。 -->

# openstarry_doc 蒸餾清單 (Distillation List)

掃描日期：2026-06-12（v0.59.1-alpha 收官期）。方法：每目錄一個分類 agent（16 組並行），讀 header 狀態戳＋結構掃描，曖昧時深讀；判準見各節定義。

> **蒸餾第一波已執行（2026-06-12 同日）**：下方標 archive 的檔案已實際移入 [archive/](./archive/) 子目錄（原路徑 `X/Y.md` → `archive/X/Y.md`，本清單保留原路徑作為判定紀錄）；缺誠實標記的 18 個 quarantine 檔已補掛 ⚠ 隔離牌；門面 README 加入導讀層。例外：`ZENODO_DEPOSIT_RUNBOOK.md` 判 archive 但 DOI 存繳未完成，仍留原位現役。
> 計數口徑：總帳「封存 133」含掃描輸出的 2 筆重複列（唯一 131 檔）；扣除 runbook 例外後實移 **130 個主檔＋28 個 .tw 雙胞＝158 個 md**（與 archive/README 的 158 一致）。

## 總帳

| 分類 | 檔數 | 佔比 |
|---|---|---|
| 進 blueprint | 51 | 17% |
| 先修再用 | 95 | 31% |
| 隔離 | 29 | 9% |
| 封存 | 133 | 43% |
| **計（主檔）** | **308** | |

讀法：blueprint 51 檔＝信中所稱「~20 顆皇冠寶石」的放寬集合（含哲學層與已驗證頂層文件）；真正第一波蒸餾建議從下方 blueprint 節的 ★ 標記開始。


## 進 blueprint（可發表級設計核心）

> 與真實系統相符（或獨立於實作仍成立）的耐久設計內容；蒸餾發表的主體。

### TOP-LEVEL

- ★ `README.md` — Canonical Ten Tenets text (highest doc authority per Master 2026-05-11) plus macro architecture and doc navigation map; links to fulfillment ledger and Letter; embedded cycle-update annotations are dated but explicitly stamped as historical notes.
- ★ `TENETS_FULFILLMENT.md` — The fulfillment ledger itself — every claim checked against source + passing test on 2026-06-11 at v0.59.1-alpha; core of the Letter to the Future; exemplary honest-boundary discipline.
- ★ `LETTER_TO_THE_FUTURE.md` — Agent OS charter / time-capsule core document written 2026-06-12; every engineering claim cross-referenced to TENETS_FULFILLMENT.md and tests; explicitly flags v0.34-era verification stamps of cited rationale docs.
- ★ `RETROSPECTIVE.md` — The distilled post-mortem itself (87.5% gap, 96% inflation, 0/30 HISTORIC, mechanism analysis, migration lessons) — publish-grade analysis of the project's scarcest asset, not raw residue; finalized 2026-06-12 (data layer git/test-verifiable, mechanism/lessons explicitly marked N=1).
- ★ `GETTING_STARTED.md` — Rewritten 2026-06-11 against actual CLI + resolver source (header explicitly documents the unfollowable v0.35-era predecessor it replaced); the verified cold-start path for the real system.

### Agent_Core_Components_Deep_Dive

- `Agent_Core_Components_Deep_Dive/00_Core_Philosophy.md` — Set anchor: three-pillar design rationale + five-skandha component map matches live architecture (klesha/vedana/volition all real); only trivial naming drift (EgoFramework vs klesha module)
- `Agent_Core_Components_Deep_Dive/01_Execution_Loop.md` — Position B two-phase deliberatePlan/deliberateAction confirmed verbatim in sdk/src/types/volition.ts; event-driven queue loop + security/confirmation gating match real core/src/execution/loop.ts design
- `Agent_Core_Components_Deep_Dive/07_Safety_Circuit_Breakers.md` — Spec was built as written: defaults match DEFAULT_SAFETY_MONITOR_CONFIG exactly (50 ticks, 100k tokens, 3 repeated-fail fingerprints, 8-of-10 error window) and SafetyMonitor lives at core/src/security/safety-monitor.ts
- `Agent_Core_Components_Deep_Dive/13_Agent_Core_as_Control_System.md` — Control-theory framing (LLM as controller, tools as control input, oscillation/divergence analysis) argues independently of implementation detail and aligns with the real vedana PID lineage — durable theory chapter
- ★ `Agent_Core_Components_Deep_Dive/14_Agent_Core_Philosophy_Five_Aggregates.md` — Known crown jewel: Five Aggregates architecture mapping with Cycle 02-4 integrated corrections; skandha interfaces (IRupa/IVedana/ISamjna/ISamskara/IVijnana) all live in SDK
- `Agent_Core_Components_Deep_Dive/15_Skandha_Protocol_Bridge.md` — Skandha-to-PluginHooks mapping verified field-by-field against sdk types/plugin.ts (vedana/provider/arbiter/contextManager/tools/volition/guide/auditor/monitors/klesha all real hooks); includes honest Two Truths epistemic disclaimer

### Architecture_Documentation 00-29

- `Architecture_Documentation/00_OpenStarry_Design_Philosophy.md` — Pure philosophy layer (digital-sentient vision, four design pillars); matches the shipped microkernel + skandha-plugin reality and argues independently of API detail.
- `Architecture_Documentation/05_Linux_Design_Principles_Inspiration.md` — Timeless Unix-philosophy analogy (everything-is-a-plugin, kernel/user space) that the shipped microkernel actually honors; no API claims to drift.
- `Architecture_Documentation/16_Plugin_Types_Philosophical_Mapping.md` — Five-skandha-to-interface mapping chart verified against sdk/types/aggregates.ts (IRupa/IVedana/ISamjna/ISamskara/IVijnana + sub-interfaces all real); future items honestly marked (future); crown-jewel companion to docs 45/DD14.
- `Architecture_Documentation/17_Host_Bootstrapping_Pattern.md` — Bootstrap-paradox resolution (Host loads, pure Core receives) is exactly how apps/runner + check-purity.sh work today; only the illustrative host list (apps/daemon, web-server) is cosmetic drift.
- `Architecture_Documentation/21_Plugin_Interface_Deep_Dive.md` — Container-vs-content (IPlugin as USB connector) explanation is durable and already received the 2026-06-11 correction pass: stamped CONCEPTUAL, redirects API authority to SDK type files, repudiates Tech Spec 03.
- `Architecture_Documentation/23_Dynamic_Plugin_Loading_and_Naming.md` — Runner-as-pure-launcher + @openstarry-plugin scope + ref.path/import(ref.name) resolution all match the shipped apps/runner exactly.
- `Architecture_Documentation/24_Runner_Architecture.md` — Describes the real apps/runner (4-step boot, CLI→Runner rename rationale, zero plugin dependencies) accurately; publish-grade host-architecture doc.
- `Architecture_Documentation/25_PushInput_Event_Architecture.md` — IPluginContext.pushInput exists verbatim in sdk/types/plugin.ts (plus pushinput-helpers.ts); design-decision narrative (killing host-injected onInput callback) matches shipped contract.
- `Architecture_Documentation/28_IInferenceProvider_Design_Spec.md` — Spec matches shipped sdk/types/inference.ts (IInferenceProvider extends IProvider with infer(), discriminated InferenceInput) and the workflow inference step exists; CNN/DNN-as-Samjna argument is durable.
- `Architecture_Documentation/29_Five_Aggregates_Core_Connection.md` — Static/dynamic five-aggregates connection diagrams with Klesha DI use interfaces verified real in SDK (volition.ts, klesha.ts, aggregates.ts); records Master DC-6 corrections; companion to crown jewels 45/DD14.

### Architecture_Documentation 30-54

- `Architecture_Documentation/34_OpenStarry_Vision_Statement.md` — Master-verbatim vision/charter (five-skandha architecture, microkernel zero-builtin, functional-AGI goal) explicitly framed as aspiration not current fact — the manifesto layer the time capsule exists for
- `Architecture_Documentation/35_Dual_Gear_Mano_Clock.md` — Dual-gear mano-clock model is the design basis of the shipped createManoAggregator + IGearArbiter chain; docs 41 (DD-1) and 42 build directly on it — crown-jewel adjacent
- `Architecture_Documentation/36_Vedana_Measurement_Model.md` — Continuous-valence measurement model shipped exactly as specified: ChannelVedana, classifyVedana, IVedanaSensor, VedanaRegistry all live in SDK; vedana-sensor-core is a production path
- `Architecture_Documentation/37_Klesha_Gain_Scheduling.md` — Known crown jewel; uniquely carries honest 2026-06-11 audit corrections (dead-wire disclosure + v0.59 closed-loop N=2 proof) — model of the honesty standard the whole corpus should meet
- ★ `Architecture_Documentation/38_IVolition_Deliberation_Pattern.md` — Known crown jewel; two-phase deliberation shipped (sdk/types/volition.ts + volition-rule-engine plugin verified present)
- `Architecture_Documentation/39_CoarisingBundle_Five_Universals.md` — Five-universals bundle shipped (sdk/types/coarising.ts verified); carries honest DC-8 'reference design, not locked' marker — durable doctrinal+type rationale
- ★ `Architecture_Documentation/41_Design_Decisions_and_Rationale.md` — Known crown jewel — ADR log (DD-1..DD-9) with rejected alternatives and reasons; the 'why' record for the whole skandha architecture
- ★ `Architecture_Documentation/42_IGearArbiter_Interface_Spec.md` — Known crown jewel — IGearArbiter spec; VasanaEngine externalization rationale matches shipped SDK type and N-gear arbitration in core
- `Architecture_Documentation/43_Cognitive_Loop_Quality_Monitoring.md` — LoopQualityMonitor design fully realized: ILoopQualityMonitor SDK type (sdk/src/types/loop-quality-monitor.ts) + monitor-loop-quality plugin both verified live; renaming-history section is honest provenance
- ★ `Architecture_Documentation/45_Five_Skandha_OOP_Architecture.md` — Known crown jewel — answers Master's four core architecture questions (type hierarchy / DI wiring / plugin loading / execution flow) with code entry points that match the real SDK
- `Architecture_Documentation/48_Twelve_Nidanas_Temporal_Taxonomy.md` — Three-tier temporal taxonomy argues independently of code and is honest about weak mappings (front-chain micro-level explicitly marked weak, 23/24) — publishable philosophical analysis with no implementation claims
- `Architecture_Documentation/49_Skandha_Soft_Constraints.md` — σ-constraint framework shipped as specified: checkSkandhaCorrespondence live in core/src/infrastructure/skandha-check.ts with dedicated test file; soft-constraint default matches implementation
- ★ `Architecture_Documentation/50_Microkernel_Purification_Rationale.md` — Known crown jewel — microkernel rationale, mechanism/policy tests, Plan32 results; minor cosmetic issue: internal title says '# 52' vs filename 50
- `Architecture_Documentation/53_Multi_Agent_Communication_Interface_Spec.md` — ICommChannel spec became reality verbatim: capability literals (messaging/streaming/rpc/composable) and file path sdk/src/types/comm-channel.ts verified matching shipped SDK; 'Research Draft' stamp now understates it

### Architecture_Documentation 55+

- `Architecture_Documentation/67_ServiceKey_Phantom_Type_Theory.md` — Type-theory rationale for ServiceKey<T> phantom-type registry; verified to match live SDK exactly (packages/sdk/src/types/service.ts:32-33 identical to doc citation) — publish-grade design content
- `Architecture_Documentation/73_Plan46_Tool_Filtering_And_Checkpoint_Hook.md` — ADR for factory-wrapper tool filtering and K-3 checkpoint hook with microkernel rationale (why runner-layer wrapper, not Core); anchors tool-filter-proxy.ts and checkpoint-manager.ts verified live in v0.59 runner
- `Architecture_Documentation/74_Plan47_K3_Wire_In.md` — ACTIVE-stamped decision record of factory bridge + composite PluginSnapshot schema + HMAC sync matching the delivered checkpoint system; single-machine limit honestly stated (cross-host alaya still open per ledger); only minor future-tense Plan48 reference now past

### Calibration_Reports

- `Calibration_Reports/15_Phase3_Shadow_Architecture.md` — Concise shadow-decision design spec (pure-function isolation guarantees); computeShadowDecision(deltas, totalObs, gear, dwell) signature verified matching live code at gear-arbiter-dynamic/src/shadow-decision.ts:18 today — design matches the real system despite the old applies-to stamp.

### Implementation_Examples

- `Implementation_Examples/Transport_Plugin_Websocket.md` — Matches the shipped @openstarry-plugin/transport-websocket (createWebSocketPlugin, IListener/IUI dual rupa, ctx.pushInput, replyTo targeting all verified in plugin source), honestly framed as a simplified version of the real plugin, plus a publish-grade JSON protocol spec; only minor drift is that pushInput auth is now built-in rather than future work.

### Implementation_Reference

- `Implementation_Reference/EN/architecture.md`〔TW/EN 對〕 — Five Aggregates mapping corrected 2026-06-11 to match SDK @skandha annotations; package table (sdk/core/shared/plugin-signer/apps/runner) and event flow match the real repo
- `Implementation_Reference/EN/configuration.md`〔TW/EN 對〕 — agent.json sections (identity/cognition/capabilities/policy/memory/plugins/guide) match configs/phase6-agent.json and SDK types (maxConcurrentTools/toolTimeout confirmed in packages/sdk/src/types/agent.ts)
- `Implementation_Reference/EN/development.md`〔TW/EN 對〕 — pnpm test/test:purity commands match the live verification SOP; createXxxPlugin factory + PluginHooks pattern matches SDK exports
- `Implementation_Reference/EN/plugins.md`〔TW/EN 對〕 — Plugin count corrected 2026-06-11 (43 dirs, explicitly curated-not-exhaustive); all named plugins verified present in openstarry_plugin/

### Plugin_System_Architecture

- `Plugin_System_Architecture/00_Plugin_Philosophy_Five_Aggregates.md` — PluginHooks shape (providers/tools/listeners/ui/guides/commands) matches sdk/src/types/plugin.ts:276-283 exactly; canonical Five-Aggregates-to-hook mapping, compact and publishable alongside doc 45/DD14.

### Project_Structure_and_Conventions

- `Project_Structure_and_Conventions/14_Markdown_Skill_Specification.md` — Frontmatter spec matches the shipped standard-function-skill plugin exactly (SkillFrontmatter: id/version/description/dependencies/parameters.model_preference verified in plugin src/index.ts); concise, publish-grade skill-format spec.

### Reference 01-22

- `Reference/01_Vijnana_Terminology_Reference.md` — Three-tier vijnana terminology convention (API/engineering/philosophy) with Two Truths declaration; key names verified live in SDK (IGuide, IVijnana, skandha:'vijnana', IInferenceProvider) — durable philosophy-layer design pairing with DD14/doc 45

### Technical_Specifications

- `Technical_Specifications/08_SecureStore_Technical_Guide.md` — 830-line component guide; spot-check matches live packages/shared/src/security exactly (secure-store.ts+file-lock.ts, hostname+username machine-binding, dual-layer locking) — publish-grade security design for real code
- `Technical_Specifications/16_HMAC_Compliance.md` — OWASP ASVS/NIST SP 800-57 compliance mapping for live runner hmac-cleanup module (confirmed present; Plan48 wired at v0.58 repair) — honest in-process threat-model scoping
- `Technical_Specifications/18_Structured_Log.md` — Concise JSONL schema + backpressure/shutdown-flush contract matching live apps/runner/src/structured-log (confirmed present, Plan48 wired v0.58)
- `Technical_Specifications/19_Schema_Drift_Policy.md` — Live module spec (apps/runner/src/schema-drift-policy confirmed present) with three-mode policy design AND honest aspirational-vs-real call-site audit table — exemplary honesty


## 先修再用（耐久但已漂移）

> 有設計價值但驗證時戳停在舊版／介面名已改——發表前需做一輪驗證更新或誠實漂移標記。

### Agent_Core_Components_Deep_Dive

- `Agent_Core_Components_Deep_Dive/02_Communication_Interface.md` — Decoupled event/command UI protocol is durable design, but the concrete API contract (submitUserInput/onNewMessage/onAgentStateChange/provideConfirmation) has zero hits in packages — real system uses pushInput + typed events; needs rename/drift pass
- `Agent_Core_Components_Deep_Dive/03_Security_Layer.md` — Path scoping, argument sanitization, audit logging, user confirmation all exist (core/src/security guardrails + security-layer), but the external policy-engine sync HTTP contract (principal/groups/ip_address) was never built — needs honest marker
- `Agent_Core_Components_Deep_Dive/04_State_Manager.md` — Tool-call/result ID correlation design is real (ToolCallRequest/ToolCallResult names survive in sdk types/message.ts) but Message shape drifted: real uses content: ContentSegment[], not separate content/tool_calls/tool_results fields
- `Agent_Core_Components_Deep_Dive/05_Plugin_Infrastructure_Integration.md` — Registry+loader wiring architecture matches core (createToolRegistry, plugin-loader with per-type registration handlers) but component names drifted: ProviderManager/UIManager are now ProviderRegistry/UIRegistry/ListenerRegistry/GuideRegistry per core README
- `Agent_Core_Components_Deep_Dive/08_Safety_Implementation.md` — Three-layer responsibility split is real (SafetyMonitor in-kernel non-plugin + live daemon at apps/runner/src/daemon), but agent_template.json naming drifted (real: AgentConfig.safety) and daemon watchdog claims (kill -9, cgroups, heartbeat) unverified — needs verification pass; stale internal number '16.'
- `Agent_Core_Components_Deep_Dive/09_Observability_and_Tracing.md` — Partially real: trace_id is mandatory in sdk StructuredError and MetricsCollector exists in core/src/observability, but Orchestrator/Gateway TraceID generation and cross-agent MCP-metadata trace propagation not found in code — needs implemented/not-implemented markers
- `Agent_Core_Components_Deep_Dive/10_Context_Management_Strategy.md` — Pluggable cognitive-strategy concept fully realized (contextManager hook + context-sliding-window/context-summary plugins exist), but IContextManager signature drifted: doc specs composePayload(systemPrompt, history, tools, ragContext); real is assembleContext(messages, maxTurns)
- `Agent_Core_Components_Deep_Dive/11_Plugin_Runtime_Isolation.md` — Worker-based sandbox is real (core/src/sandbox: worker-pool, sandbox-manager, rpc-handler) but '系統支持三種隔離級別' overstates — vm2 level cites a dead/vulnerable library and WASM level is aspirational; needs per-level status pass
- `Agent_Core_Components_Deep_Dive/12_Error_Handling_and_Self_Correction.md` — Error-normalization feedback loop and Type A/B/C taxonomy match real loop behavior (tool errors fed to context, guide interprets), but the central type ToolExecutionResult does not exist — real types are ToolCallResult and StructuredError
- `Agent_Core_Components_Deep_Dive/16_OpenStarry_Standard_Protocol.md` — Protocol substance matches SDK (ProviderResponse.segments with tool_call segment shape is real), but type names/locations drifted: ToolCall{id?} vs real ToolCallRequest{id required, arguments not args}, and cited '@openstarry/sdk interfaces.ts' no longer exists

### Architecture_Documentation 00-29

- `Architecture_Documentation/01_Architecture_Overview.md` — Useful system map, but 2 of 5 subsystems (Orchestrator Daemon, Agent Design Service) were never built; stale CURRENT/v0.34 stamp needs drift markers.
- `Architecture_Documentation/02_Headless_Agent_Core.md` — Event-queue-driven pure core concept matches real AgentCore, but stamped v0.34 and terminology (Listener typing, MCP_Listener) needs verification against shipped SDK.
- `Architecture_Documentation/04_Plugin_Infrastructure.md` — Aggregate-pattern + five-skandha engineering mapping is real and durable, but the plugin.json manifest format it specifies never existed (confirmed by doc 12 banner); needs drift marker.
- `Architecture_Documentation/06_Plugin_Interface_Examples.md` — Callback-style interfaces (onNewMessage, submitUserInput, provideConfirmation) name an API that changed — real contract is factory + PluginHooks + bus events; keep only with interface-drift pass.
- `Architecture_Documentation/07_Supporting_Engines_Ecosystem.md` — Kernel-vs-external-services rationale is durable, but the concrete engine roster (memory/policy/CI engines as microservices) was never built and reads as existing capability.
- `Architecture_Documentation/08_Command_And_Tool_Design.md` — Command-vs-Tool taxonomy survives (ISlashCommand is real in SDK), but the handling locus (UI plugin intercepts) and v0.34 stamp need verification against shipped slash-command flow.
- `Architecture_Documentation/09_Communication_Protocol_Strategy.md` — ICommChannel + commChannels hook + capability/topology model substantially shipped (sdk/types/comm-channel.ts matches), but stamp still says 'pending engineering formal spec' and Hub=Daemon framing is stale — needs verification-update pass.
- `Architecture_Documentation/12_Workflow_Engine_Tool_Design.md` — Already carries honest 2026-06-11 banner enumerating never-built parts (workflow:status, 4 node executors, plugin.json) and pointing to plugin source as authority; encapsulate-workflow-as-tool rationale is durable but body still needs the banner to stay attached.
- `Architecture_Documentation/18_Plugin_Loading_Protocol.md` — Dynamic-loading mechanics (ref.path / import(ref.name), BUILTIN_FACTORIES removal) match reality, but the entry contract shown (export default async initialize) names an interface that changed to IPlugin{manifest,factory}→PluginHooks.
- `Architecture_Documentation/19_Agent_Coordination_Layer.md` — Several specified pieces later became real (agent-registry.ts, process tree + orphan reparenting in v0.59.1, findByCapability), but the Daemon-process framing and 'openstarry plugin sync' never shipped; stale Research-Draft stamp needs reconciliation pass.
- `Architecture_Documentation/20_Dependency_Injection_and_Control_Loop.md` — OODA wiring narrative and broken-loop diagnostics are durable, but labels Listener as 受 (canon says 色·輸入) and uses context.dependencies — real mechanism is ctx.services IServiceRegistry; v0.34 stamp.
- `Architecture_Documentation/22_Agent_Coordination_Layer_Normalization.md` — Anti-corruption-layer rationale matches shipped provider normalization (ProviderStreamEvent, hot-swappable provider-local-llama), but pseudo-code names provider.generate() where the real contract is chat(); one-pass naming update needed.
- `Architecture_Documentation/26_Plugin_Service_And_Lifecycle_Management.md` — Service-sharing concept shipped (ctx.services IServiceRegistry), but doc shows ctx.registerService/getService(string-id) where real API is typed ServiceKey<T> registry (docs 58/67), and privileged IPluginManager does not exist in SDK.

### Architecture_Documentation 30-54

- `Architecture_Documentation/30_Sparsha_Coarising_Model.md` — Sparsha→coarising engineering mapping is durable and CoarisingBundle is implemented (sdk/types/coarising.ts), but stamp frozen at v0.34.0-alpha / 2026-03-16 — needs re-verification pass against v0.59 names
- `Architecture_Documentation/31_Eight_Consciousnesses_Runtime.md` — Eight-consciousness runtime mapping (EventBus/ExecutionLoop/IListener) is durable manifesto-layer content but verified only at v0.34.0-alpha; runtime details need a drift check before publishing
- `Architecture_Documentation/32_Samskara_Extended_Scope.md` — Master's cetana-centered samskara semantics (DC-6 open-scope ruling) are durable design authority, but doc has no status stamp and names v0.2x-era interfaces (ISlashCommand, State.update) needing verification
- `Architecture_Documentation/33_Multivalue_Skandha_Design.md` — Manifest-multi-value/interface-single-value design shipped (plugin.ts:79 skandha?: Skandha|readonly Skandha[]) BUT doc claims skandha 必填 while SDK made it optional — concrete drift, needs honest marker (SDK wins)
- `Architecture_Documentation/40_Tenet_6_Architecture_Mapping.md` — Sentence-by-sentence Tenet #6 exegesis is durable canonical-tenet material, but mapped against v0.34 types/clock domains and predates the v0.59 Tenet-#6 fulfillment proof — needs reconciliation with TENETS_FULFILLMENT.md
- `Architecture_Documentation/44_Safety_Architecture_Overview.md` — 'Single reference' for layered safety is durable, but its 'Layer 4 VedanaEmergency 已實作 (Plan28)' claim was a delivered-dead fossil until v0.59.1 T1b wiring — doc 37 got the 2026-06-11 honest-correction pass, this one did not; needs same pass
- `Architecture_Documentation/46_AuditContext_and_Extras_Protocol.md` — AuditContext/extras/WIENER-constraint spec matches shipped SDK (audit-context.ts, confidence-auditor.ts) and auditor-threshold/auditor-passthrough plugins exist, but status block still says 'Plan32 將提取...' in future tense — stale, needs verified-stamp refresh
- `Architecture_Documentation/47_Cognitive_Sequence_Appendix.md` — Six-state FSM + two-truths framing is durable theory, but it claims to formalize ExecutionLoop's actual state machine as of Cycle 02-8 — needs verification that v0.59 loop states still match before publishing as spec
- `Architecture_Documentation/52_Alaya_Partial_Mapping.md` — Exemplary honest partial-mapping analysis (two-truths declaration, PRESENT/ABSENT evidence table) but evidence pinned to v0.34.0-alpha AgentCore; alaya runtime since evolved (AC7 distributed alaya, v0.59 work) — needs re-verification of the coverage table
- `Architecture_Documentation/54_Multi_Agent_Security_Model.md` — Zero-trust/capability model partially real (register_agent + capability gating in apps/channel, agent-registry frozen at Plan38) but the T0-T3 trust taxonomy and coverage-analysis claims need verification against the shipped channel before publishing

### Architecture_Documentation 55+

- `Architecture_Documentation/55_AC7_Distributed_Alaya_Runtime.md` — Design spec of real distributed-alaya subsystem (IBijaStore verified in sdk/src/types/distributed-alaya.ts + plugin), but stamped v0.39.0-alpha and alaya was extended for Tenet #6 N=2 in v0.59 — needs drift re-verification
- `Architecture_Documentation/56_Audit_Path_B_Modified_Delta.md` — Real confidence-audit delta mechanism (rawDelta lives in core/mano/confidence-audit.ts + loop.ts), but coefficients predate the v0.40 DELTA_SCALING_FACTOR=0.055 recalibration — numbers stale, mechanism durable
- `Architecture_Documentation/57_Registry_Bridge_IPC.md` — Daemon-authoritative IPC design is real (apps/runner/src/daemon has ipc-server/client, global-service-registry, event-bridge) but module names drifted from the doc and v0.59.1 processTree repair touched this area
- `Architecture_Documentation/58_Typed_Service_Registry_Design.md` — Design rationale for ServiceKey<T> registry that did land (verified live in SDK), but still framed as pre-implementation proposal with vote stamps and LOC estimates — needs an implemented-as-designed pass
- `Architecture_Documentation/62_ISeedKeyProvider_Abstraction_Analysis.md` — Only doc covering the real ISeedKeyProvider HMAC abstraction and DELTA_SCALING_FACTOR placement, but framed as a cycle 03-5 review with letter grades and W2-R4 references — needs reframing/verification at v0.59
- `Architecture_Documentation/65_Gear_Arbiter_Dynamic_Buddhist_Mapping.md` — Durable Madhyamaka/Yogacara design mapping for the real gear-arbiter-dynamic plugin with an honest Two Truths disclaimer, but pinned to v0.41 constants (MIN_N=10, TOOL_CONFIDENCE_TABLE) that need drift verification
- `Architecture_Documentation/66_Shadow_Counting_Control_Theory.md` — Strong durable design rationale (observer-controller separation, P(observe exit)=0 proof of the coupled-design deadlock) for a real shipped mechanism, but argued through cycle 03-6 process artifacts (O5/CV-5, D2 votes) — needs a verification-update pass
- `Architecture_Documentation/71_Two_Path_Config_Propagation.md` — Documents a real architectural invariant (Path A factory(config) vs Path B ctx.config; plugin-resolver.ts verified live) with honest 24-version-gap history, but last grounded at v0.43/44 — both paths need re-verification at v0.59
- `Architecture_Documentation/72_WIENER_L2_L3_Safety_Framework.md` — Durable L0-L3 observe/escalate/respond safety-stack design for the real spc-monitor plugin, but the v0.59.1 repair sprint found safety-chain wiring (VedanaEmergency) dead until June — claims need re-verification against current wiring
- `Architecture_Documentation/79_Plugin_Gear_Arbiters.md` — Durable null-output contract for gear-arbiter-dynamic (companion to crown-jewel doc 42), but §5's promised Plan50 rename to gear-arbiter-wiener never landed (plugin dir still gear-arbiter-dynamic) — needs an honest drift marker

### Calibration_Reports

- `Calibration_Reports/02_Plan_B4_Delta_Scaling_Design.md` — Root-cause analysis (13.2x clamp signal loss) and design rationale for DELTA_SCALING_FACTOR=0.055, which is verified live in gear-arbiter-dynamic/src/calibration-bridge.ts:12 — durable ADR material, but v0.4x-era stamps and scholar-vote framing need a verification/cleanup pass.
- `Calibration_Reports/03_Calibration_Methodology.md` — Five-layer confidence model (L0 SafetyMonitor / L1 Klesha / L2 IConfidenceAuditor / L3 ILoopQualityMonitor / L4 VedanaEmergency) matches the real v0.59.1 stack and is core routing design, but last touched cycle 03-7 (v0.4x era) and the C2 four-phase calibration protocol is tied to retired W2 campaign machinery — needs drift-marker pass.
- `Calibration_Reports/21_WIENER_Thresholds_Hypothesis.md` — The sigma-transparency disclaimer (sigma = composition index over deterministic event counts, NOT a variance estimator) and HYPOTHESIS epistemics are durable honest-marker content, but the doc states apps/runner/src/wiener/thresholds.ts as current — that module no longer exists (only the wiener_threshold_hit event name survives in spc-monitor); recalibration schedule is tied to retired cycle counters.

### Implementation_Examples

- `Implementation_Examples/Context_Strategy_SlidingWindow.md` — FIFO sliding-window concept matches the real context-sliding-window plugin (required in every runner config), but code shows phantom class SlidingWindowStrategy.composePayload vs real createSlidingWindowContextPlugin factory; also dated model refs (Gemini 1.5 Pro, Claude 3).
- `Implementation_Examples/Developer_Guide_Plugin_Migration.md` — Durable migration guidance and step 4 (pushInput replacing constructor callbacks) matches current SDK, but entry-point spec drifts: class implements IPlugin with initialize(context) + context.registerProvider/registerTool vs real manifest+factory(ctx)->PluginHooks; package.json 'openstarry' metadata field unverified.
- `Implementation_Examples/Pain_Mechanism_Demo.md` — Vedana three-feelings design is real and shipped (vedana-sensor-core plugin exists; VedanaEmergency wired at v0.59.1) and is crown-jewel-adjacent philosophy, but context.registerGuide/interpretPain exist only in the SDK mock-host test harness, not the production plugin API — needs interface re-verification or drift markers.
- `Implementation_Examples/Provider_Gemini_Example.md` — Real counterpart exists (provider-gemini plugin shipped), but doc codes against phantom LLMProvider from '@openstarry/core/interfaces' with init/generate class shape instead of real IProvider + factory from '@openstarry/sdk'; helper methods are stubs returning empty arrays.
- `Implementation_Examples/Tool_ReadFile_Example.md` — Trivially valid tool concept (real equivalent = standard-function-fs), but the class-with-name/description/args + module.exports-instance shape predates the factory pattern and matches nothing in the current SDK; tiny doc, cheap to rewrite or fold into a real example.
- `Implementation_Examples/UI_Plugin_Example.md` — UI-plugin bridge role and tool-confirmation flow are real (IUI + confirmation-gate-standard exist), but the entire code block is the phantom pre-factory API (constructor(coreApi), core.on('onNewMessage'), provideConfirmation, submitUserInput — none in SDK); needs full rewrite against IUI.onEvent + pushInput.
- `Implementation_Examples/openclaw_UI_Channel_Adapters.md` — The per-platform channel-adapter pattern (webhook inbound, platform API outbound, session mapping) maps directly onto the real IListener/IUI + pushInput architecture and apps/channel, but all API calls shown (coreApi.submitUserInput, core.on('onNewMessage'), constructor wiring) are phantom pre-factory names needing rewrite.

### Implementation_Plans

- `Implementation_Plans/Plan37_Multi_Agent_Communication.md` — Only doc in set with durable architecture: Master-confirmed all-nodes-MCP-Server+Client topology, openstarry-channel hub, ICommChannel/IMcpTransport/process-tree — subsystem is live (apps/channel, v0.59.1 processTree fix) but spec is pinned at v0.37 with R-ring citations; needs drift-marker pass vs current SDK

### Implementation_Reference

- `Implementation_Reference/EN/cli.md`〔TW/EN 對〕 — Every command listed exists in apps/runner/src/commands/, but doc omits checkpoint, config-validate, config-migrate; no verification stamp — needs a completeness pass
- `Implementation_Reference/EN/hmac-compliance.md`〔TW/EN 對〕 — hmac-cleanup module exists in apps/runner/src and OWASP/NIST mapping is durable security design, but doc is framed in Plan48/R3 governance bindings and never re-verified past v0.48/0.49 era
- `Implementation_Reference/EN/hmac-key-rotation-architecture.md`〔TW/EN 對〕 — Honestly self-labeled design-spec-only (rotation never implemented, deferred) so not aspiration-as-fact; durable forward design but anchored to Plan48 decisions and needs a fresh not-implemented drift marker at v0.59
- `Implementation_Reference/EN/plugin-gear-arbiters.md`〔TW/EN 對〕 — Null-output contract for gear-arbiter-dynamic (companion to crown-jewel doc 42) is durable and both arbiter plugins still exist, but §5's promised Plan50 rename to gear-arbiter-wiener never happened and stamp is frozen at v0.49.0-alpha
- `Implementation_Reference/EN/plugin-install-reliability.md`〔TW/EN 對〕 — OPENSTARRY_INSTALL_DIR/OPENSTARRY_LOCK_PATH env-var contract and resolution order are durable operational reference, but the H1-H8 hypothesis-elimination table is Plan49 research residue to trim and the stamp is v0.49-era
- `Implementation_Reference/EN/schema-drift-policy.md`〔TW/EN 對〕 — Module apps/runner/src/schema-drift-policy/index.ts exists and the 3-mode design + boot-resolved env resolution is durable, but the call-site migration status table is stamped as-of v0.49.0-alpha and unverified at v0.59
- `Implementation_Reference/EN/structured-log.md`〔TW/EN 對〕 — Module and all referenced files (writer.ts, safe-stringify.ts, shutdown.ts) exist and the schema/back-pressure/shutdown-order design is durable, but Plan48 binding labels and HERACLITUS/MRB jargon need stripping and no re-verification since v0.48 era

### Plugin_System_Architecture

- `Plugin_System_Architecture/06_Data_Validation_Example.md` — Core idea is real — sdk/src/types/tool.ts uses parameters: z.ZodType<TInput> with safeParse-at-boundary practice — but the class-based 'args' tool shape is stale and section 2 leans on the never-built 'MCP 消息總線'; needs a drift-marker pass.
- `Plugin_System_Architecture/07_SPC_Monitor_Plugin.md` — Real plugin (@openstarry-plugin/spc-monitor; shewhart-chart.ts and audit:shadow_decision/spc_anomaly events confirmed in src) with an honest stamp, but frozen at v0.44: the '13 tests' and 'monitoring-only, informational ONLY' claims predate safety-gate.ts/escalation-monitor.ts/sigma-regime.ts now in the plugin.

### Project_Structure_and_Conventions

- `Project_Structure_and_Conventions/01_Monorepo_Top_Level_Structure.md` — Purity/ecosystem-separation rationale is durable, but lists apps/daemon, apps/dashboard, apps/installer (real apps = runner, channel only) and claims plugins are outside the workspace while pnpm-workspace.yaml actually includes ../openstarry_plugin/*.
- `Project_Structure_and_Conventions/02_Core_Source_Code_Structure.md` — Five-Aggregate module mapping is durable and vedana/, vijnana/ (klesha.ts, vitakka-watchdog) are real, but tree drifts: lists memory/ and types/ which don't exist; missing real bus/, mano/, observability/, sandbox/, session/, testing/.
- `Project_Structure_and_Conventions/03_Shared_and_SDK_Structure.md` — Zero-dep SDK principle and SUSSMAN three-layer config pattern (Plan32) are durable, but the IPlugin example (initialize/shutdown class) predates the real manifest+factory contract and names stale interfaces (ILanguageModelProvider, IVectorStore).
- `Project_Structure_and_Conventions/04_Standard_Agent_Directory_Anatomy.md` — Fractal agent-root layout and project-over-system plugin precedence match the real runner, but the System Agent / AgentManagerTool / daemon SIGKILL kill-switch layer was never built — needs honest drift markers.
- `Project_Structure_and_Conventions/05_Agent_Manifest_Specification.md` — agent.json sections (identity/cognition/capabilities) match the real config shape, but details drifted: real provider is a string not an {id,config} object, plugins is a top-level array of {name} not capabilities.plugins, and the policy block doesn't exist in configs/phase6-agent.json.
- `Project_Structure_and_Conventions/06_Plugin_Directory_Conventions.md` — Directory-as-protocol, package.json openstarry field, and @openstarry-plugin/ namespace are durable and real, but the entry-point example uses the old class-based IPlugin (real convention = createXxxPlugin() factory) and references a never-built 'Agent Design Service'.
- `Project_Structure_and_Conventions/07_Coding_and_Testing_Standards.md` — Standards match real practice (Vitest, strict TS, co-located *.test.ts, error-as-pain → ToolResult), but names AgentPainException which has zero hits in the codebase and waffles 'Jest 或 Vitest'.
- `Project_Structure_and_Conventions/08_System_and_Project_Runtime_Layouts.md` — ~/.openstarry + ./.openstarry overlay and directory-as-permission are real (runner utils: plugin-resolver, project-detector, config-merger), but the daemon.pid / agents/ / storage/main.db daemon-managed layout describes the removed daemon era.
- `Project_Structure_and_Conventions/09_CLI_Design_and_Management_Commands.md` — Most commands verified real in apps/runner (start, init, attach, ps, daemon start/stop, plugin sync), but 'openstarry design' TUI workbench, run-tool, clean, and register are not in the current command set — durable CLI UX design needing a reality pass.
- `Project_Structure_and_Conventions/10_Build_and_Distribution_Strategy.md` — Zero-bundling purity principle is real and enforced by check-purity.sh, but the described dist/ layout (bin/openstarry-core.js, assets/standard-plugins) and 'pnpm run plugins:sync' don't match real per-package builds and the 'plugin sync' CLI command.
- `Project_Structure_and_Conventions/11_Third_Party_Plugin_Installation.md` — Install design (repo sync, symlink dev mode, dependency warnings, permission prompts) is durable and partially shipped, but the spec command is 'plugin add' while the real CLI is plugin install/uninstall, and daemon auto-registration/restart steps reference the removed daemon.
- `Project_Structure_and_Conventions/12_Capabilities_Injection_Mechanism.md` — IoC/DI loading-sequence design is durable, but the core contract shown (class IPlugin with initialize/shutdown, setLLMProvider) is the pre-factory-pattern API; real plugins use manifest + factory(ctx) and pushInput.
- `Project_Structure_and_Conventions/13_Composite_Plugins_and_Dependencies.md` — Capability-tag dependency and L1/L2/L3 ecosystem-layering concepts are durable, but IAgentOperations / findToolByTag / agentOps.getLLM were never built; real dependency wiring is ctx.dependencies + topologicalSort (Plan19).
- `Project_Structure_and_Conventions/15_Testing_Strategy_and_Infrastructure.md` — Strategy matches what was actually done (co-located Vitest tests, purity boundary enforcement, LLM replay mechanism — replay-cache exists), but tool names drift: dependency-cruiser/'pnpm check-dependency-boundaries' vs real check-purity.sh, and TestAgentHost is speculative.
- `Project_Structure_and_Conventions/16_Plugin_Registry_and_Distribution.md` — Decentralized git distribution, SemVer rules, and signature verification (packages/plugin-signer is real) are durable, but registry.db, openstarry-lock.json (real file is ~/.openstarry/plugins/lock.json), the PR-based metadata registry, and the 'openstarry-plugin-<name>' naming all drift from reality.
- `Project_Structure_and_Conventions/17_Developer_Experience_and_Tooling.md` — create-plugin scaffold is real (CreatePluginCommand) and devtools inspector exists as a plugin, but 'openstarry dev' hot-reload mode and TestAgentHost from '@openstarry/sdk/testing' don't exist as named.
- `Project_Structure_and_Conventions/18_Agent_Runtime_Configuration.md` — extends/deep-merge and env-var injection are real (runner utils/config-merger.ts) and zero-plugin fail-soft is durable design, but the plugins-as-map-with-enabled-flags schema drifts from the real top-level plugins array.

### Reference 01-22

- `Reference/04_Priority_Deadlock_Formal_Analysis.md` — Real engineering content: gear-arbiter chain priority model, P(observe exit)=0 proof, shadow-counting decoupling spec, Rules #55/#56 design constraints — but verified at v0.41.0-alpha era, needs drift pass against current arbiter code (doc 42 sibling)

### Technical_Specifications

- `Technical_Specifications/09_HMAC_Seed_Authentication_Architecture.md` — Durable threat-model + tautological-verification flaw analysis for distributed-alaya (plugin live, SEC-002 since fixed), but anchored to v0.39.0-alpha defect state — needs current-state verification pass
- `Technical_Specifications/10_Microkernel_Security_Analysis.md` — Durable security companion to crown-jewel doc 50 (attack surface, architecture-vs-contract isolation), but Last-verified 2026-03-16 at v0.34.0-alpha — textbook stale-stamp case
- `Technical_Specifications/11_Phased_Authority_Transfer_Architecture.md` — Shadow-counting/phased-transfer design for live gear-arbiter-dynamic plugin (complements crown-jewel doc 42 IGearArbiter), but v0.41-era and laced with retired governance Rules #55-57 — needs verification vs current arbiter code
- `Technical_Specifications/12_Key_Rotation_Design_Rationale.md` — Honest design-only doc with explicit assumptions A1-A4; rotation was never implemented (doc 17 confirms still deferred) — keep with an explicit never-built status marker
- `Technical_Specifications/13_M4a_Dual_Track_Design.md` — Durable metric-design rationale (binary vs deviation track) for shipped shadow-agreement machinery, but cycle-03-7-stamped and gates on retired Rules/Phase vocabulary — needs scrub + verification
- `Technical_Specifications/15_Phase3_Shadow_M4a_Implementation.md` — Concrete pure-function shadow-decision spec (computeShadowDecision, isolation constraints C44-1/2) for live gear-arbiter-dynamic — durable but stamped v0.44.0-alpha, unverified since
- `Technical_Specifications/17_HMAC_Key_Rotation_Architecture.md` — Explicitly design-only rotation architecture, never implemented — durable future-work design but needs current never-built marker and governance-vote-reference scrub
- `Technical_Specifications/Plan50_Sigma_Regime_Binding.md` — sigma-regime.ts is live in spc-monitor plugin (confirmed) and the regime-tagging rationale is durable, but doc is wrapped in retired Rule #76/#77 SPC governance machinery and vote provenance — needs extraction pass
- `Technical_Specifications/Plan50_pushInput_CP4_Invariant.md` — CP-1/2/3/4 invariants + CR-SCK sourceContext type for shipped pushInput (live in SDK, confirmed) — durable security-design core buried under R3 vote provenance; superseded-by-Plan52 status needs marking
- `Technical_Specifications/Plan52_pushInput_Binding.md`〔TW/EN 對〕 — pushInput Candidate B shipped and live in SDK (confirmed); header honestly corrected to SHIPPED at 2026-06-11 audit, but body unverified vs current SDK and heavy with R3/Batch provenance
- `Technical_Specifications/Plan57_D30_5_VasanaEngine_Binding.md`〔TW/EN 對〕 — vasana-engine plugin is live (confirmed in plugin repo); deposit-only passive-observer design is durable 五蘊 architecture content, but status still reads 'pending Batch 16' and 4-method API needs verification vs current plugin
- `Technical_Specifications/Plan58_Mesh_Binding.md`〔TW/EN 對〕 — mesh plugin live (and repaired at v0.58 'mesh unloadable' fix); centralized-hub architecture choice is durable, but status stuck at 'pending Batch 18' and pre-repair content needs verification
- `Technical_Specifications/Plan59_API_Runtime_Binding.md`〔TW/EN 對〕 — api-runtime plugin live and elevated to FP in Phase 7 closure (A1-5); observability/bounded-intervention design durable, but governance-stamped and unverified since v0.56-era
- `Technical_Specifications/Plan60_Blackboard_Alaya_Binding.md`〔TW/EN 對〕 — distributed-alaya plugin live (BijaStore/seed-signature/vector-clock baseline real); alaya storage-layer spec durable, but wrapped in Phase 6 completion ceremony and 'pending Batch 20' staleness


## 隔離（虛構層）

> 描述未曾建造的系統、或把願景寫成現況——必須掛隔離 banner，永不作為 spec 發表。

### TOP-LEVEL

- `User_Scenario_and_Workflow_Guide.md` — Vision-scenario doc (TUI dashboard with THOUGHTS/SEC, global `openstarry` command) that never matched the built system; isolation banner already applied 2026-06-11 redirecting readers to GETTING_STARTED.md — verify banner stays, never publish as spec.

### Agent_Core_Components_Deep_Dive

- `Agent_Core_Components_Deep_Dive/06_State_Persistence_Mechanism.md` — Presents AgentSnapshot contract + 'storage' plugin type + crash recovery as defined system spec, but no storage hook exists in PluginHooks, no snapshot persistence anywhere in 48 plugins, and CLI history persistence is still on the future-work list — never built; also carries stale internal number '14.'

### Architecture_Documentation 00-29

- `Architecture_Documentation/03_Agent_Design_and_Template_Service.md` — Runtime REST template service (POST /templates, PUT .../plugins) never built — real system uses static agent.json configs; presented as CURRENT fact with no banner.
- `Architecture_Documentation/10_Bootstrapping_And_Plugin_Loading.md` — Daemon-first three-stage boot (opennexus-daemon start → Master Agent → workers) describes a never-built system, stamped CURRENT with no banner; real boot is runner + agent.json.
- `Architecture_Documentation/11_Agent_Manager_Tool_Design.md` — AgentManagerTool POSTing to a daemon management API at localhost:5050 — neither the tool nor the daemon exists (LLM child-spawn tool face is still on the unfinished-work ledger).
- `Architecture_Documentation/13_Orchestrator_Daemon_Design.md` — MIXED (isolation banner added 2026-06-12 + v0.59.7 reconciliation note): the systemd-like daemon (process supervision, infra-plugin hosting, REST management API) was never built; apps/ contains only runner and channel. BUT the Plan37 Process Tree section IS real and e2e-tested (agent.spawnChild / agent.processTree / SEC-003 / orphan reap — daemon-process-tree.e2e.test.ts) and the `ps --tree` CLI landed v0.59.7 — those parts are not fiction.
- `Architecture_Documentation/14_System_Boot_Sequence.md` — Boot chain Daemon Layer → Design Layer API → Master Agent bootstrap describes the never-built daemon/template-service stack as current operating logic; no banner.
- `Architecture_Documentation/15_System_Startup_and_Task_Flow.md` — Full lifecycle narrative (openstarry-daemon start, Master Agent with AgentManagerTool/AgentDesignerTool, MCP inter-agent collaboration) of a system that never existed; no banner.
- `Architecture_Documentation/27_System_Topology_and_Management_Zone.md` — Grand topology (WASM sandboxes, causality-chain scheduler, quota policy layer, HAL feeding PiKVM/Jetson sensor streams) presented as the system's defined operating logic — none of it built; no banner.

### Implementation_Examples

- `Implementation_Examples/Developer_Guide_Standalone_Execution.md` — Premised on never-built Orchestrator Daemon / Design Layer / database production stack; entire code uses phantom APIs (AgentCore, InMemoryStateManager, coreApi.on('onNewMessage'), submitUserInput, provideConfirmation — zero hits in packages/sdk/src); real standalone path is apps/runner bin.js with a JSON config.
- `Implementation_Examples/Tool_CodeInterpreter_Example.md` — Depends entirely on context.infrastructure.getService('sandbox_manager') and a Daemon-managed Sandbox Infrastructure — no such SDK surface or subsystem was ever built (zero 'infrastructure' service API in packages/sdk); presents the Level-3 isolation stack as working fact.
- `Implementation_Examples/USB_Plug_and_Play_Agent_Scenario.md` — Presented as 實作範例 but describes a never-built stack: std-hardware-listener plugin, System Agent gatekeeper, Orchestrator Daemon, agent:validate_manifest/agent:spawn/agent:kill tools — none exist (real agent spawning is the v0.59.1 processTree/agent-ask path, entirely different); compelling vision content that must carry a banner.
- `Implementation_Examples/openclaw_Coordination_Layer.md` — Describes a central multi-channel router + YAML routing tables + workflow-engine coordination layer (cross-references 02_Agent_Coordination_Layer) that was never built; written as implementation thinking for matching openclaw, i.e. aspiration framed as the system's design.
- `Implementation_Examples/opencode_Code_Interpreter_Suite.md` — Entire doc designs around a shared backend Sandbox Manager (Docker lifecycle, sandbox_fs:* tools, python:execute via sandbox) that was never built — same fiction layer as Tool_CodeInterpreter_Example and the Daemon infrastructure in Tech Specs 01-07.

### Plugin_System_Architecture

- `Plugin_System_Architecture/01_MCP_Plugin_Example.md` — context.infrastructure.getService('message_broker') and the init/start(eventQueue) Listener from '@openstarry/core/interfaces' never existed (zero codebase hits); 'MCP' here means a broker-based agent bus, colliding with the real mcp-client/server (Model Context Protocol) plugins.
- `Plugin_System_Architecture/02_MCP_Protocol_Integration.md` — Presents the never-built broker-based MCP listener/sender as the current architecture ('在新的架構中'); no message_broker service or eventQueue.push API ever shipped; same MCP name collision as doc 01.
- `Plugin_System_Architecture/03_Developer_Tools_Example.md` — code:search, SandboxManager, and LspProcessManager were never built; uses fictional plugin.json type/entryPoint manifest model; the real devtools plugin is agent introspection/metrics (Plan11), not these tools.
- `Plugin_System_Architecture/04_Web_Interaction_Example.md` — Webhook-Listener via plugin.json config and a Puppeteer/Playwright Browser-Control-Suite were never built (no such plugin among the 44); manifest model and eventQueue.push API are fictional relative to the shipped factory-pattern SDK.
- `Plugin_System_Architecture/05_Advanced_UI_And_Device_Example.md` — Canvas drawing UI and device sensors (GPS/camera/accelerometer via React Native/Flutter/Tauri bridge) were never built; pure aspiration written as worked examples on the fictional plugin.json model.

### Technical_Specifications

- `Technical_Specifications/01_Command_Registry_and_Discovery.md` — 2026-06-11 quarantine banner verified present; describes registry.json/plugin.json discovery system never built (17-claim audit: contradicts shipped SDK)
- `Technical_Specifications/02_Event_Schema_and_Bus_Protocol.md` — Banner verified; IOpenStarryEvent UUID envelope has zero hits in source — real system uses ~90-constant AgentEventType vocabulary
- `Technical_Specifications/03_Plugin_Interface_Definitions.md` — Banner verified; IOpenStarryPlugin/ICoreHost contract never existed — shipped contract is IPlugin{manifest,factory}+PluginHooks
- `Technical_Specifications/04_Context_Management_and_Memory_Strategy.md` — Banner verified; three-tier memory model is aspiration — real system is context-sliding-window plugin, not this design
- `Technical_Specifications/05_Security_and_Sandboxing_Protocol.md` — Banner verified; PathGuard/chroot-jail/whitelist protocol never built as specified
- `Technical_Specifications/06_Inter_Agent_MCP_Protocol.md` — Banner verified; MCP server/client extension protocol as specified contradicts shipped mcp-client/mcp-server plugins (audit confirmed v0.59: mcp stdio was a delivered-dead fossil)
- `Technical_Specifications/07_Management_Zone_and_Orchestrator_Spec.md` — Banner verified; openstarryd IDaemonControlPlane spec never built — real daemon differs (live composition = Plan37/38 spawn system)
- `Technical_Specifications/Plan51_Zod_Gate_Binding.md`〔TW/EN 對〕 — Header already carries honest banner: all zod-gate modules REMOVED at v0.58.0-alpha, never enforced at any runtime boundary (zero production imports) — future-work reference only, never publishable as spec
- `Technical_Specifications/Plan54_AC9_Binding.md`〔TW/EN 對〕 — Banner present: delivered v0.52 but NEVER WIRED (zero production imports; MAX_SPAWN_DEPTH conflicted with live daemon), removed at v0.58 — live composition is the Plan37/38 daemon spawn system
- `Technical_Specifications/Plan56_D30_4_Binding.md`〔TW/EN 對〕 — Banner present: multi-IVolition library delivered v0.53 but NEVER WIRED into execution loop, removed at v0.58 — live volition is single-IVolition core path; the 五蘊 doctrinal layer survives in doc 38/DD14 instead


## 封存（過程殘留＝post-mortem 資料）

> cycle 戳記的治理紀錄、投票、校準報告——僅作為 RETROSPECTIVE 的原始資料層保存，不進 blueprint。

### TOP-LEVEL

- `CHANGELOG_RESEARCH_TEAM.md` — 228KB of cycle-stamped R-team governance records (gate verdicts, vote tallies, ratification batches, counter snapshots) — prime post-mortem raw data for RETROSPECTIVE.md, zero blueprint content.
- `OD_doc_corrections.md` — Cycle 02-10 doc-correction work list, self-marked HISTORICAL with a cycle 03-29 supersession note; all items long executed; kept only as audit trail.
- `QW_quick_wins.md` — Cycle 02-10 deliverable memo (Codex-review quick wins); its durable payloads (plugin category table, 3-layer doc architecture, audience tags) were already integrated into README/GETTING_STARTED — the memo itself is process residue.
- `VERIFICATION.md` — Cycle 03-5 snapshot copy-verification report (file/line-count checksums dated 2026-04-08) — pure process record with no design content.
- `Windows_Migration_Guide.md` — Point-in-time migration snapshot of the v0.19.0-beta (2026-02-13) multi-agent eco environment — Claude Code settings, agent definitions, memory files of the now-retired team harness; Windows fixes it documents are long since in code; historical record, not design.
- `ZENODO_DEPOSIT_RUNBOOK.md` — Internal ops runbook for the pending Zenodo deposit (task #23, awaiting Master decisions on authorship/license) — actively needed, do NOT delete, but it is operational tooling, not blueprint content for publication.

### Agent_Core_Components_Deep_Dive

- `Agent_Core_Components_Deep_Dive/NOTE_Doc15_gap.md` — Cycle 02-10 process note about a doc-numbering gap that was since filled by 15_Skandha_Protocol_Bridge.md — obsolete bookkeeping, post-mortem value only

### Agent_Corps

- `Agent_Corps/Agent_Roles_and_SOP.md` — Roles/RACI/5-SOP playbook for the retired 7-agent dev corps; pure dev-process methodology, not system design; useful only as how-we-worked post-mortem.
- `Agent_Corps/Coordinator_Checklist.md` — Per-cycle operational checklist for the retired coordinator workflow (phase gates, baseline scripts); process residue with no design content.
- `Agent_Corps/Cycle03-1_Compliance_Impact_Analysis.md` — Cycle-stamped per-Tenet compliance impact table against v0.36.1-alpha decisions (D1-D4); governance audit record, superseded by TENETS_FULFILLMENT.md.
- `Agent_Corps/Cycle03-1_Synthesis.md` — Cycle 03-1 research-round synthesis (22 decisions, BUG-3, Plan37 planning); cycle-stamped R-team output, design substance lives in the Architecture docs it fed.
- `Agent_Corps/Cycle03-1_Team_Transformation_Proposal.md` — Proposal to reorganize the 24-agent research team into 4 groups; pure team-governance artifact of the retired machine.
- `Agent_Corps/Cycle03-2_Scope_Notes.md` — Forward scope/planning notes for cycle 03-2 (Plan37 verification waves, Plan38 spec); cycle-stamped planning residue.
- `Agent_Corps/CYCLE18_DOCUMENTATION_UPDATE.md` — doc-keeper status report listing which tracking files were updated for cycle 18; bookkeeping about bookkeeping.
- `Agent_Corps/CYCLE_03_2_DOC_FIX_SUMMARY.md` — Summary of 5 doc patches applied post-Plan38 (doc 51/13 edits); change-log of edits already merged elsewhere, no standalone value.
- `Agent_Corps/Iteration_Log.md` — Index + quick-reference for the cycle logs; stale snapshot (claims current = v0.43.0-alpha vs real v0.59.1-alpha); process ledger, post-mortem only.
- `Agent_Corps/Iteration_Log_Cycle01.md` — Per-cycle dev log 2026-02-09~02-16 (Cycles 1-33, Plan01-23); raw process history, prime retrospective material but not blueprint.
- `Agent_Corps/Iteration_Log_Cycle02.md` — Per-cycle dev log for Plan24-36b incl. Five-Skandha remapping decisions D-01~D-06; the design outcomes live in Architecture docs, this is the process trail.
- `Agent_Corps/Iteration_Log_Cycle03.md` — Per-cycle dev log for Plan37-43 multi-agent phase; cycle-stamped execution record, post-mortem data only.
- `Agent_Corps/Lessons_Learned.md` — Per-cycle retrospectives with reusable engineering patterns noted inline; exactly the honest post-mortem data the distillation/retrospective track needs, but process-stamped, not blueprint spec.
- `Agent_Corps/Plan35_Completion_Summary.md` — Single-plan completion report (context-summary plugin + vedanaFn wiring) frozen at v0.35.0-alpha; delivery record, content superseded by code and arch docs.
- `Agent_Corps/Plan_Dependencies_and_DoD.md` — Plan dependency graph + DoD checklist tracking Plan01-43 completion; project-management ledger of the retired cadence, not design content.
- `Agent_Corps/Risk_Register.md` — Dev-process risk register (agent context overflow, pnpm sync failures, Windows quirks); operational governance of the retired corps, post-mortem only.
- `Agent_Corps/Roadmap.md` — Version-history table frozen mid-v0.4x era (real version v0.59.1-alpha); stale tracking ledger, useful only as timeline evidence for the retrospective.
- `Agent_Corps/Iteration_Decisions/20260211_cycle7_converge.md` — Cycle 7 Phase 4 convergence record (PASS verdict, rework tally) for sandbox hardening; cycle-stamped gate record, the feature itself is documented in arch docs/code.

### Architecture_Documentation 30-54

- `Architecture_Documentation/51_Codex_Review_Response.md` — Cycle 02-10 R4 response report to external Codex review — vote-referenced (D4-R1..R6) process record and comms strategy, not design; valuable only as post-mortem/external-validation data

### Architecture_Documentation 55+

- `Architecture_Documentation/59_V4_Spec_Signal_Theory_Analysis.md` — Rationale for retired W2 verification-rule thresholds (CV-6, Rules #50/#51) — signal-theory framing is interesting but the subject is governance calibration machinery, not the system
- `Architecture_Documentation/60_Calibration_Convergence_Model.md` — W2 four-round calibration convergence report (R1-R4 trajectory tables, Phase 2 transition criteria) — classic calibration report, post-mortem value only
- `Architecture_Documentation/61_ENG_FAB_Checklist_Design_Rationale.md` — Design rationale for the 29-item anti-fabrication audit checklist (Rules #46-#53) — retired governance machinery; valuable as honest-process post-mortem data only
- `Architecture_Documentation/63_W2_Phase2_Calibration_Architecture.md` — W2 R1-R4 calibration history with GO/IF verdicts and V4 spec correction record — duplicates doc 60 territory; cycle-stamped calibration report
- `Architecture_Documentation/64_Fabrication_Pattern_Five_Cycle.md` — Plan36-40 fabrication evolution taxonomy — prime honest-process post-mortem data (exactly the asset-class #1 material for the retrospective) but emphatically not blueprint content
- `Architecture_Documentation/68_MR11_Tenets_As_Architectural_Choice.md` — Master permanent-ruling record (MR-11) with R3 decision-impact tables — governance doctrine; the one-line statement is quotable in the letter-to-future but the doc is process record
- `Architecture_Documentation/69_Rule63_L4_Config_Activates.md` — Verification-process rule (L4 config-activates testing requirement) from the H0=fabrication framework — team-process governance, not system design; testing wisdom belongs in retrospective
- `Architecture_Documentation/70_Rules_64_to_68_Rerun_Policy_Fix_Disclosure.md` — Five governance rules on re-run policy and fix disclosure with ratification stamps — pure process machinery from the retired verification regime
- `Architecture_Documentation/75_Rules_71_72_FR1_FR2_Amendments.md` — Rule amendment record (rolling-reference wording fix + zero-variance handling) with 24/24 vote tallies and ratification batch numbers — governance residue
- `Architecture_Documentation/76_Compliance_Framework_9_0_1_And_Zero_Tolerance.md` — Compliance bookkeeping framework (9/0/1 PENDING, 9-cell matrix, auto-upgrade conditions, ZERO TOLERANCE rulings) — the governance machine itself, post-mortem value only
- `Architecture_Documentation/76_Compliance_Framework_endpoint_achievement_cycle03-24.md`〔TW/EN 對〕 — Cycle-stamped HISTORIC endpoint status update (10/0/0-star declaration, 23/0 vote tallies) — the closure-declaration genre later implicated in the 96% inflation audit
- `Architecture_Documentation/77_Tenet_10_Phase_6_Roadmap.md` — Phase 6 planning roadmap (Plan48-55) framed entirely in MR-5 compliance machinery and ratification batches — planning/process record, superseded by what actually shipped
- `Architecture_Documentation/77_Tenet_10_Phase_6_Roadmap_completion_cycle03-24.md`〔TW/EN 對〕 — Cycle-stamped 7/7 completion status update with vote tallies and ratification authority chain — pure process residue
- `Architecture_Documentation/77_Tenet_10_Phase_6_Roadmap_status_update_cycle03-25.md`〔TW/EN 對〕 — Cross-reference index / Plan55 sunset bookkeeping for the 77-series siblings — meta-process record about other process records
- `Architecture_Documentation/78_Rule_74_L1_Prime_Code_Doc_Sync.md` — Governance rule (code-doc sync gate) born from the PLAN46-DOC-001 incident, with debate citations and ratification batches — team-process rule, not system design

### Calibration_Reports

- `Calibration_Reports/01_Calibration_Phase1_Report.md` — Phase-1 test-campaign trajectory record (W2-R1/R2/R3, CV-5/CV-6 decisions, sensor-repair history); empirical post-mortem data only — methodology lives in doc 03, the resulting design in doc 02.
- `Calibration_Reports/04_Lyapunov_Phase0_Report.md` — Campaign analysis report with a CONDITIONAL GO verdict pinned to v0.33.0-alpha data; Lyapunov framing is interesting but the document is a phase verdict over a specific test run, not durable spec.
- `Calibration_Reports/05_Phase2_Monitoring_Framework.md` — Campaign monitoring framework with W2-R5 (v0.41.0-alpha) LOCKED baseline numbers and metric thresholds for a completed phase; the 'ACTIVE' status is itself stale process state.
- `Calibration_Reports/06_CV5_Observe_Mode_Investigation.md` — Bug root-cause investigation (calibration-bridge payload extraction + observe-mode exit); the fix shipped in Plan42, so this is incident post-mortem data only.
- `Calibration_Reports/16_Phase3_Baseline_R9.md` — W2-R9 empirical baseline snapshot with PROVISIONAL parameters and round counters; pure campaign data, superseded by later rounds.
- `Calibration_Reports/17_Phase3_Baseline_R10_R11.md` — W2-R10/R11 model-change transition-window data report (gpt-5.4 sigma equality observation) tied to Rules #69-#73 cycle machinery; campaign record only.
- `Calibration_Reports/18_PathC_20Field_Monitoring_Format.md` — BINDING per-round W2 monitoring record format (20 fields) with R3 vote tallies and ratification-batch annexes — pure governance apparatus of the retired cycle machine.
- `Calibration_Reports/19_Plan48_25_SubItems_Binding.md` — Plan48 acceptance-criteria/attestation checklist (25 atomic criteria, vote tallies, Batch 8a annex) — delivery-process residue; the engineering content lives in code and delivery reports.
- `Calibration_Reports/20_Section_7_3_Verification_Plan.md` — Research-side audit checklist (CK-1..7, EC-1..8) for the coordinator 7-step sync procedure in a CLAUDE.md version that no longer exists — governance of the retired harness.
- `Calibration_Reports/22_SPC_Recalibration_First_Execution_cycle03-22.md`〔TW/EN 對〕 — Coordinator-backfilled SPC baseline governance record (Master directive citations, Batch 19 ratification tallies, supersession chains) for the retired Rule #72 cycle machinery — post-mortem data only.
- `Calibration_Reports/redaction_security_debt.md`〔TW/EN 對〕 — Per-plugin retrofit progress register with sunset deadlines and ratification status; the redaction format itself is implemented and tested in packages/sdk/src/utils/redaction.ts, so the register is campaign-tracking residue (useful only as security-debt audit trail).
- `Calibration_Reports/redaction_security_debt_schema_v2_cycle03-20.md` — Schema-amendment vote record (sunset_state enum codification, per-item tallies, Batch 17 dispatch) for the debt register above — governance process residue.

### Implementation_Plans

- `Implementation_Plans/Plan01_MVP_Alpha_Foundation.md` — Completed 2026-02-04 MVP bootstrap plan; monorepo skeleton history, all content superseded by real codebase
- `Implementation_Plans/Plan02_Event_Driven_And_Safety.md` — Completed gap-fix plan vs 2026-02 test reports; event-queue/safety design now lives in code and Architecture docs
- `Implementation_Plans/Plan03_Quality_And_Skill_Parser.md` — Completed v0.1→v0.2 quality/purity upgrade checklist; pure delivery record
- `Implementation_Plans/Plan04_IUI_Interface_And_Guide_Plugin.md` — Completed plan that split IUI from IListener; the Five-Aggregates rationale is canonical in Arch 45/DD14, this is the build record
- `Implementation_Plans/Plan05_Multi_Channel_UI_And_Advanced_Listener.md` — Completed v0.2 Beta plan for websocket/http transports; plugins exist, plan is history
- `Implementation_Plans/Plan08_TUI_Dashboard_MVP.md` — Stale header still says Planned though tui-dashboard plugin shipped; peripheral plugin plan, not blueprint material — header deserves a one-line correction during archiving
- `Implementation_Plans/Plan28_IVolition_v1_Safety_Hardening.md` — Completed v0.28 wave record citing cycle02-4 R-team spec files; IVolition design authority is Arch doc 38, not this
- `Implementation_Plans/Plan29_ILoopQualityMonitor_IConfidenceAuditor_SEC029.md` — Completed v0.29 wiring record (file-by-file change tables, test counts); pure delivery log
- `Implementation_Plans/Plan30_LoopQualityMonitor_Layer3.md` — Scope-absorbed stub (work landed via Plan29/Plan32); historical pointer only
- `Implementation_Plans/Plan31_AuditContext_ThresholdAuditor_AuditTrail.md` — Header still says Pending though v0.31 shipped long ago — stale stamp on a cycle-02-7 wave plan; delivery record
- `Implementation_Plans/Plan32_Tenet7_Absolute_Purity.md` — Completed purity-migration record (policy values out of Core); the microkernel argument itself is canonical in Arch doc 50
- `Implementation_Plans/Plan34_Project_Level_Config.md` — Completed .openstarry/ config delivery record; KD decision table is useful post-mortem data but restrict-only model is documented in code/Reference layer
- `Implementation_Plans/Plan35_Context_Summary_VedanaFn.md` — Header frozen at IN PROGRESS though v0.35 shipped (stale stamp); compliance-driven wave plan with LEIBNIZ scoring residue
- `Implementation_Plans/Plan37_W2R1_Protocol.md` — Lyapunov mini-pilot calibration protocol (sigma/w_theta recalibration, designer roster by agent codename); textbook calibration-report residue
- `Implementation_Plans/Plan41_Typed_Registry_CV5_LateJoiner.md` — Completed v0.41 delivery record laced with ENG-FAB/CV-5 governance vocabulary; ServiceKey/late-joiner design now lives in SDK types
- `Implementation_Plans/Plan42_CV5_Fix_Stabilization.md` — Completed stabilization record (RC-1/2/3 root causes, Rules #54-56); good post-mortem data, not blueprint
- `Implementation_Plans/Plan43_COND_Backfill.md` — Backfill/ratification record (W-1 GATE, Rules #57-62, MR-11/12, compliance 10/0/0) — governance-process residue by definition
- `Implementation_Plans/Plan44_Phase3_SPC.md` — Completed SPC-monitor delivery summary with cycle predicates (D2 v2, Rule #47, compliance streak); delivery log
- `Implementation_Plans/Plan45_Implementation_Plan.md` — Completed WIENER L2/L3 delivery record; header already honestly corrected by 2026-06-11 repair audit — model archive citizen
- `Implementation_Plans/Plan46_Implementation_Plan.md` — Delivered v0.46 cleanup record (SEC items, tool filtering, K-3 hook) with ratification cross-refs; Arch doc 73 carries the durable design
- `Implementation_Plans/Plan47_Implementation_Plan.md` — Delivered v0.47 K-3 wire-in record built around Master ratification artifacts and Rule #74; Arch doc 74 is the design-side twin

### Implementation_Reference

- `Implementation_Reference/EN/wiener-thresholds.md`〔TW/EN 對〕 — Subject module deleted in v0.58.0-alpha (zero imports over its life, honest 2026-06-11 banner); remaining content is HYPOTHESIS/calibration status with cycle counters — post-mortem data only, not blueprint material

### Project_Structure_and_Conventions

- `Project_Structure_and_Conventions/00_Roadmap_and_Milestones.md` — Cycle-stamped milestone/status ledger (v0.1.0–v0.32.0, Feb-Mar 2026) with per-cycle test counts and completion checkmarks; pure progress record, post-mortem data only — not design content.
- `Project_Structure_and_Conventions/Testing_Guide.md` — Release-stamped manual verification checklist frozen at v0.20.1-beta (15 plugins / 1339 tests vs today's 46 loadable plugins / 3247 passed at v0.59.7), ending in a blank test-results recording table; superseded by the rewritten GETTING_STARTED.md and current verification SOP.

### Reference 01-22

- `Reference/02_Fabrication_Pattern_Evolution.md` — Taxonomy of Plan36-40 engineering fabrication patterns tied to MR-10/Rule #48/52/53 governance machinery — prime honest-process post-mortem data, not blueprint content
- `Reference/03_ENG_FAB_Audit_Checklist.md` — ENG-FAB v1.1 audit checklist (MUST/SHOULD gates, Rule #47/49/53) — retired governance audit process, post-mortem value only
- `Reference/05_Fabrication_Pattern_Six_Cycle.md` — Six-cycle fabrication evolution analysis (Plan36-41 phantom integration) — post-mortem honesty data, governance-cycle stamped, not design spec
- `Reference/06_P_Coincidence_Binding_Convention.md` — Governance wording convention for P(coincidence)<=10^-48 sigma-match claims (Rule #76, Batch 3A ratification record) — pure process residue
- `Reference/07_V11_Wording_Binding.md` — Wording binding declaring V11=1.0000 a tautological numerical artifact (R3 D-03(b) 22/1 vote record) — governance metric-phrasing residue
- `Reference/08_Rule_76_Section_76_7_Caveat.md` — In-place Rule #76 sigma-regime caveat with full R3 vote/dissent/Batch 3-A' machinery — governance rule residue
- `Reference/09_Rule_77_SPC_Pooled_Mode_Sigma_Regime.md` — Rule #77 SPC pooled-mode sigma_regime conjunct (D-04 18/5 with 5-agent dissent, Batch 10 Item 8) — SPC governance machinery retired with the cycle system
- `Reference/10_Rule_75_Section_75_X_pnpm_build.md` — Rule #75 §75.X pnpm-build-at-release-tag amendment; the engineering practice survives in CLAUDE.md verification SOP, the rule document itself is ratification-batch residue
- `Reference/11_Rule_78_TW_Translation.md`〔TW/EN 對〕 — Rule #78 TW translation parity policy (L1 half of L1+L3 hybrid, vote tallies, batch dispatch) — governance translation-process residue
- `Reference/12_Cycle03-13_Backfill_Classification.md`〔TW/EN 對〕 — One-off MR-10 retroactive classification of a single TW backfill instance — cycle-stamped procedural record with zero design content
- `Reference/13_Plugin_Loader_Cycle03_17_Evaluation_Criteria.md`〔TW/EN 對〕 — 5-criterion/5x4 decision matrix for whether plugin-loader re-enters scope — governance gating framework for a module that stayed DEFERRED FINAL; decision-record residue
- `Reference/13_Plugin_Loader_Cycle03_17_Evaluation_Criteria_status_update_cycle03-21.md`〔TW/EN 對〕 — Terminal-evaluation status update (Continue-DEFERRED FINAL, T1-T4 sunset clause, 23/0 vote) — pure cycle status record
- `Reference/14_Chair_Rule_Retrofit_Codification.md`〔TW/EN 對〕 — SCRIBE-internal chair-rule for retrofit motion admissibility — R3 procedural governance, later RETIRED per its own status update
- `Reference/14_Chair_Rule_Retrofit_Codification_status_update_cycle03-25.md`〔TW/EN 對〕 — Status update recording chair-rule RETIRED after cycle 03-21 binary final terminal — retirement bookkeeping
- `Reference/15_Security_Retroactive_Precedent.md`〔TW/EN 對〕 — Master directive precedent classification for security-gap retroactive backfill — governance precedent record, no system design content
- `Reference/16_R4_Folder_Convention_Discipline_v3.md`〔TW/EN 對〕 — R4 deliverable folder convention + §G6 cycle-close freeze rule for the retired R-team cycle machinery — pure process discipline
- `Reference/19_Audit_Methodology_Codification_v2.md`〔TW/EN 對〕 — Audit methodology for the plugin x Tenet sweep matrix (canonical Tenet axis re-anchor) — methodology of the retired audit cycles, post-mortem only
- `Reference/20_Audit_Verdict_Format_Codification_v2.md`〔TW/EN 對〕 — Audit verdict cell format codification (CP/FP verdict shapes, lint levels) for retired audit machinery — process format residue
- `Reference/20_Audit_Verdict_Format_Codification_v3.md` — v3 superset of verdict format (verdict-shape canonical pattern, roll_forward strict reading, lint L9/L10) — same retired audit-process machinery, stacked vote provenance
- `Reference/21_Counter_Registry.md`〔TW/EN 對〕 — Single source of truth for C-S1/C-C4/C-U* governance counters with 7-version amendment ledger — quintessential counter/process residue (CLAUDE.md: no new counters ever)
- `Reference/21_amendment_cycle03-30.md` — Additive counter amendment (C-LEIBNIZ-N2-observation) — counter governance residue
- `Reference/21_amendment_cycle03-31.md` — Additive counter amendment (C-C8 + evidence-tier ladder) — counter governance residue
- `Reference/22_Agent_Behavioral_Guidelines.md`〔TW/EN 對〕 — BG-* agent process-discipline guidelines repurposed from the drift Tenet list — governance doctrine layer explicitly retired (CLAUDE.md: BG-* no longer produced); content is process residue not architecture
- `Reference/ENG_FAB_v1.9_satellites_status_update_cycle03-25.md`〔TW/EN 對〕 — Combined status update marking ENG-FAB v1.9 candidate RETIRED across 4 satellite docs — retirement bookkeeping (no numeric prefix, included for directory completeness)
- `Reference/TW_backfill_list_cycle03-20.md` — One-shot list of 9 missing TW siblings per Rule #78 §78.5 — completed task list, pure process residue (no numeric prefix, included for directory completeness)

### Reference 23+

- `Reference/ENG_FAB_v1.9_satellites_status_update_cycle03-25.md`〔TW/EN 對〕 — Cycle 03-25 combined status update retiring ENG-FAB v1.9 candidate references in 4 satellite docs; pure ratification/vote-tally bookkeeping (Batch 22, 23/0 UNANIMOUS), no system design content.
- `Reference/TW_backfill_list_cycle03-20.md` — Cycle 03-20 TW-translation sibling backfill checklist (Rule #78 §78.5 same-PR ledger, canonical 265→274); pure governance bookkeeping with no design value.
- `Reference/_archive/cycle03-26_consolidation/16_R4_Folder_Convention_Discipline.md`〔TW/EN 對〕 — Superseded v1 of R-team release-folder governance rules (consolidated into v3 at top level); already in _archive, retains only post-mortem value.
- `Reference/_archive/cycle03-26_consolidation/16_R4_Folder_Convention_Discipline_amendment_cycle03-20.md` — Superseded forward-binding amendment patch (4 AMEND clauses, 23/0 vote) merged into Reference/16 v2/v3; governance-process residue in _archive.
- `Reference/_archive/cycle03-26_consolidation/16_R4_Folder_Convention_Discipline_amendment_cycle03-23.md`〔TW/EN 對〕 — Superseded amendment patch (responsibility-matrix rows 4-5) merged into Reference/16 v2/v3; cycle-stamped governance residue in _archive.
- `Reference/_archive/cycle03-26_consolidation/19_Audit_Methodology_Codification.md`〔TW/EN 對〕 — Superseded v1 of R-team audit methodology codification (top-level v2 exists); governance-process methodology for the retired cycle machine, post-mortem value only.
- `Reference/_archive/cycle03-26_consolidation/20_Audit_Verdict_Format_Codification.md`〔TW/EN 對〕 — Superseded v1 of audit verdict format spec (top-level v2/v3 exist); pure governance-output formatting rules for the retired R-team process.
- `Reference/_archive/cycle03-28_consolidation/16_R4_Folder_Convention_Discipline_v2.md`〔TW/EN 對〕 — Superseded v2 consolidation of Reference/16 (top-level v3 exists); mechanical merge of v1 + two amendments, governance residue already archived.

### Research_Methodology

- `Research_Methodology/01_D2_Compliance_Predicate_Methodology.md` — Formal compliance-predicate methodology adopted by R3 D2 debate (5-0 vote); pure governance-evaluation machinery for cycle audits, superseded in authority by TENETS_FULFILLMENT.md
- `Research_Methodology/02_ENG_FAB_Anti_Fabrication_Framework.md` — 7-cycle fabrication history (6/6 new-feature Plans fabricated, Plan42 clean) plus verification trust model; prime honest-process-data for the retrospective, not blueprint design
- `Research_Methodology/03_Ten_Tenets_Compliance_Report_v039.md` — Cycle-stamped 10/0/0 compliance declaration with R3 vote tallies; claim later contradicted by reconciliation (96% inflation finding) and final TENETS_FULFILLMENT.md ledger — post-mortem evidence only
- `Research_Methodology/04_Ten_Tenets_Compliance_Report.md` — v0.37-era compliance report (8/2/0) superseded by doc 03; records MR-1~MR-5 Master rulings but is a stale governance status record
- `Research_Methodology/05_ENG_FAB_Audit_Checklist_v1.2.md` — Versioned audit checklist (v1.0-v1.3, 22 to 38 items) for the retired Dev-to-Research verification loop; pure process tooling
- `Research_Methodology/06_Rerun_Policy_and_Fix_Disclosure.md` — Governance Rules #64-67 rationale record for W2 test-round reruns; tied entirely to the retired cycle machinery
- `Research_Methodology/07_DEV_1b_Closure_Framework.md` — Fabrication-pattern closure adjudication (NC1-NC4 conditions) for one tracked governance entry; cycle-stamped process record
- `Research_Methodology/08_Rule_72_N10_SubClause_Draft.md` — DRAFT SPC-recalibration sub-clause (Westgard pooled-std, N-window semantics) for W2 round statistics; never-ratified governance rule draft
- `Research_Methodology/09_Rule_75_PreDelivery_Gate_Draft.md` — DRAFT dev-layer pre-delivery gate rule responding to the Plan36-41 fabrication pattern; process control for the retired team workflow
- `Research_Methodology/10_ENG_FAB_v1.7_Candidate.md` — Checklist version diff (v1.6 to v1.7, 43 items) with R3 vote provenance; superseded by v1.8 and pure process residue
- `Research_Methodology/11_ENG_FAB_v1.8_Binding.md` — Ratified 48-item audit checklist version with vote tallies and triple-path item-count arithmetic; governance tooling for the retired verification loop
- `Research_Methodology/12_GN_1_5_SCRIBE_G_Gate.md` — SCRIBE-internal gate definitions (GN.1-GN.5) with dissent records and vote tallies; internal research-team process discipline, retired with the teams
- `Research_Methodology/13_F_15_Scope_Expansion_Governance_Docs.md` — Front-matter schema amendment for governance docs (F-15 v2); meta-governance about governance documents themselves
- `Research_Methodology/14_G4_folder_6_O7_Freshness.md` — Evidence-freshness check gate for O7 compliance reports; cycle-timing process fix, no design content
- `Research_Methodology/15_ENG_FAB_v1.9_F_16_StructuredError.md`〔TW/EN 對〕 — ENG-FAB audit-item candidate for structured-error reporting in delivery reports; governance checklist increment (later RETIRED per status update), not a runtime error-schema spec
- `Research_Methodology/15_ENG_FAB_v1.9_F_16_StructuredError_status_update_cycle03-21.md`〔TW/EN 對〕 — Terminal status update with R3 vote tally (15/7/1) retiring the F-16 candidate; pure cycle-stamped bookkeeping
- `Research_Methodology/16_F_15_v3_Third_Tier_Amendment.md`〔TW/EN 對〕 — TW-translation parity tier amendment to the F-15 governance-doc front-matter discipline; process rule for doc bookkeeping, no system design

### Technical_Specifications

- `Technical_Specifications/14_Plan43_COND2_StateTracker_Spec_Amendment.md` — Retroactive method-name ratification record (Rule #62 Tier 1 bureaucracy); only durable content is two method signatures already in code
- `Technical_Specifications/Plan49_MR6_Conditional_Gates.md` — Self-declared KNOWLEDGE ONLY scope record dominated by vote tallies; 2026-06-11 audit banner confirms its thresholds module was removed with zero lifetime imports
- `Technical_Specifications/Plan54_AC9_Binding_status_update_cycle03-25.md`〔TW/EN 對〕 — Pure governance residue: records retirement of FORBIDDEN-phrasings patterns via cycle vote tallies; no design content
- `Technical_Specifications/Plan57_D30_5_VasanaEngine_Binding_amendment_cycle03-21.md`〔TW/EN 對〕 — Refactor bookkeeping record (runner-level to plugin-layer move, 0 function change) with UNANIMOUS-vote provenance; the result is already reflected in live plugin location
- `Technical_Specifications/_archive/Plan53_B_prime_sunset.md` — Sunset/retirement record of a reserved deferred path that was never opened — vote-tally and DSS-preservation bureaucracy, post-mortem value only
- `Technical_Specifications/_archive/cycle03-21_Plan55_sunset/Plan55_DEFERRED_FINAL.md` — Self-described historical/sunset archive existing only to preserve a citation handle for the terminally-deferred Plan55; pure process residue


## 各組觀察（分類 agent 的跨檔 pattern 備註）

- **TOP-LEVEL**：Top level splits cleanly into two strata: a freshly verified 2026-06-11/12 time-capsule core (README + TENETS_FULFILLMENT + LETTER + RETROSPECTIVE + GETTING_STARTED, all 5 = the publishable spine) versus pre-pivot residue untouched since its originating cycle (CHANGELOG/OD/QW/VERIFICATION at cycle 02-10..03-5, Windows guide at v0.19.0-beta). No .tw.md twins exist at this level. Only one quarantine case (User_Scenario guide) and its isolation banner is already in place.
- **Agent_Core_Components_Deep_Dive**：No Last-verified stamps anywhere in the set; no .tw twins. Pattern: philosophy/protocol layer (00, 13, 14, 15) and the safety-breaker spec (07) survived to implementation nearly verbatim, while early engineering deep-dives (02-06, 08-12, 16) carry pre-rename interface names and one (06) specs a never-built storage subsystem. Files 06/07/08 still carry stale internal numbers 14/15/16 from an older series.
- **Agent_Corps**：Entire directory is dev-process residue of the retired multi-agent team machine (roles/SOP, checklists, cycle logs, plan tracking, cycle-stamped reports) — zero Agent OS design content, so all 18 files are archive. Iteration_Log_Cycle01-03 and Lessons_Learned are the highest-value post-mortem inputs for the retrospective/distillation (task #24); no .tw twins exist in this set.
- **Architecture_Documentation 00-29**：Clear stratigraphy: docs 00-15 are the early v0.34-era suite built around an Orchestrator Daemon + Master Agent + Template Service that was never built (real apps/ = runner + channel only) — the daemon family needs isolation banners; docs 16-29 are later and track the shipped SDK closely (IRupa/IVedana/ISamjna/ISamskara/IVijnana, pushInput, ICommChannel, IInferenceProvider all verified present in packages/sdk/src). Only docs 12 and 21 received the 2026-06-11 repair-audit banner pass; the other daemon docs carry no honesty marker at all.
- **Architecture_Documentation 30-54**：This range is the R3-debate-derived design core. Nearly every interface named (ICommChannel, ILoopQualityMonitor, AuditContext, checkSkandhaCorrespondence, CoarisingBundle, IVolition, IGearArbiter) verified live in SDK/core/plugins — drift concentrates in v0.34-era "Last verified" stamps, future-tense Plan32 status lines, and implementation claims that doc 37 got honest 2026-06-11 corrections for but siblings (esp. 44) did not. Only doc 51 is pure process residue.
- **Architecture_Documentation 55+**：Clear stratification: 55-67 are cycle-stamped R-team engineering analyses of real shipped subsystems (mostly fix — design durable but pinned to v0.39-v0.41 stamps); 68-78 are dominated by governance rules/compliance machinery (archive); the only .tw twins in the set are the 76/77 cycle-stamped status-update siblings, all pure process residue. Code anchors spot-checked against v0.59.1 source: ServiceKey (doc 67) matches SDK byte-for-byte; tool-filter/checkpoint anchors (73/74) live; gear-arbiter-wiener rename promised in doc 79 never landed.
- **Calibration_Reports**：Directory is dominated by W2 calibration-campaign residue (round reports, locked baselines, BINDING monitoring formats, ratification tallies). Three docs hold durable design kernels: 15 (shadow architecture — computeShadowDecision signature verified byte-matching live gear-arbiter-dynamic/src/shadow-decision.ts), 02 (rationale for DELTA_SCALING_FACTOR=0.055, verified live in calibration-bridge.ts), 03 (five-layer confidence model L0-L4 that matches the real klesha/auditor/loop-quality/VedanaEmergency stack). Doc 21 names apps/runner/src/wiener/thresholds.ts which no longer exists in the codebase (only the wiener_threshold_hit event name survives in spc-monitor) — needs a drift marker.
- **Implementation_Examples**：No file in this folder carries a Status/Applies-to/Last-verified stamp, and none has a .tw/.en twin (all are zh-TW only). The folder splits cleanly into three eras: (1) one modern doc (Transport_Plugin_Websocket) written against the real factory-pattern SDK and explicitly tracking the shipped plugin; (2) pre-factory-era examples using a phantom class/constructor/coreApi.on('onNewMessage')/submitUserInput API that never existed in packages/sdk — concepts real, code stale; (3) Daemon/Sandbox-Manager/coordination-layer docs that belong to the same never-built fiction layer as Tech Specs 01-07 and need the same isolation banner.
- **Implementation_Plans**：Directory is version-stamped engineering plan/delivery records (Plan01-47), i.e. honest build history rather than durable spec — almost all archive. Three stale status stamps spotted: Plan08 still says "Planned" though tui-dashboard plugin shipped; Plan31 says "Pending" and Plan35 "IN PROGRESS" though v0.31/v0.35 were delivered. Only Plan37 (multi-agent foundation) carries durable architecture for a still-live subsystem (apps/channel, mcp-client/server, process tree) and merits a drift-marker pass.
- **Implementation_Reference**：Two strata: 4 user-facing reference docs freshly corrected/verifiable at v0.58-0.59 (blueprint) vs. 7 Plan48/49-stamped module docs frozen at v0.49.0-alpha era whose underlying modules still exist but whose stamps, governance bindings (C48-*/C49-*/R3 D-*), and status tables were never re-verified (fix), plus 1 calibration doc for a module removed in v0.58 (archive). Every EN file has a byte-equivalent TW twin with identical verdict.
- **Plugin_System_Architecture**：Clean three-way split: doc 00 is the accurate philosophy-to-API bridge (matches shipped PluginHooks verbatim); docs 01-05 are pre-implementation sketches from an abandoned plugin.json/eventQueue/infrastructure-broker design that never matched the shipped factory-pattern SDK — none carry isolation banners today despite being as fictional as Tech Specs 01-07; docs 06-07 landed in code but drifted (07 is the only file in the set with a status stamp).
- **Project_Structure_and_Conventions**：No EN/TW twins and no formal status stamps anywhere in this set (only Testing_Guide carries a version header). The directory is the Feb-2026 conventions layer: its structural conventions largely shipped (overlay dirs, plugin namespace, purity enforcement, most CLI commands verified in apps/runner src), but every code example froze at the pre-factory-pattern IPlugin (class initialize/shutdown) and several docs assume the now-removed daemon/design-mode layer. 16 of 19 files are 'fix' with the SAME two drift axes — one systematic drift-marker pass (old IPlugin contract + daemon references) would clear most of the set; only 14 is publish-ready as-is.
- **Reference 01-22**：Reference/ is almost entirely the governance machine's rulebook: 24 of 26 files are cycle-stamped rules, wording bindings, counters, audit formats, and retirement status updates (archive). Only two carry system-design value: 01 (vijnana terminology tiers — names still match the live SDK, blueprint) and 04 (gear-arbiter priority deadlock formal analysis + shadow-counting spec — durable but v0.41-era, fix). The _archive/ subdir (cycle03-26/03-28 consolidation snapshots of superseded v1/v2 docs) was treated as out of scope. Note 16/19/20 exist only as v2/v3 here (v1s moved to _archive); 20 has both v2 (.tw twin) and v3 (no twin) live side by side — distillation should keep at most one if any survive as post-mortem exhibits.
- **Reference 23+**：No Reference docs numbered 23+ exist (highest is 22). Assigned set resolves to 2 non-numbered top-level files plus _archive/ (superseded governance-doc versions). Everything is cycle-stamped governance-process residue: ratification batches, vote tallies, MR-12 forward-only amendment patches, TW-sibling backfill ledgers. Zero design content about the actual system — uniform archive verdict.
- **Research_Methodology**：Entire folder is governance-process residue from the cycle-03 R-team era (ENG-FAB versions, Rule drafts, compliance reports, gates, vote tallies). Zero system-design content; nothing belongs in the blueprint. Doc 02 (fabrication history, 6/6 new-feature Plans fabricated) is the highest-value post-mortem artifact in the set. Docs 03/04 carry "Status: CURRENT" stamps asserting Tenet compliance levels later contradicted by TENETS_FULFILLMENT.md — must not surface without that caveat.
- **Technical_Specifications**：Three strata: (1) docs 01-07 = pre-implementation fiction, all carry the 2026-06-11 quarantine banner — verified present; (2) docs 08-19 = engineering docs mostly anchored to real shipped code (secure-store, hmac-cleanup, structured-log, schema-drift-policy, gear-arbiter, sigma-regime all confirmed live in source) but with stale version stamps; (3) Plan49-60 = governance-wrapped binding specs whose verdict tracks the v0.58 amputation: library removed/never-wired → quarantine (banner already present), feature shipped and live → fix (strip vote-tally provenance + verify vs SDK), pure ratification/sunset records → archive.

## 後續動作建議（依序）

1. **第一波蒸餾**＝上方 ★ 檔（11 檔）：頂層時間膠囊脊柱（README＋帳本＋信＋retrospective＋GETTING_STARTED）＋六顆架構寶石。這一波可直接組成對外 blueprint repo 的初版。
2. **fix 層分流**：95 檔逐一過「驗證更新 or 漂移標記」——工作量大，建議僅對 blueprint 引用到的 fix 檔優先處理，其餘標記後緩處理。
3. **quarantine 層覆核**：29 檔確認 banner 在位（Tech Specs 01-07 已掛；本輪掃描新點名者待補掛）。
4. **archive 層不動**：133 檔原地保存，是 RETROSPECTIVE 的證據層。

---
*本清單由 distillation-sweep workflow（16 agents）於 2026-06-12 產出；分類為 agent 判斷，發表前以 SDK／測試為最終權威複核。*
