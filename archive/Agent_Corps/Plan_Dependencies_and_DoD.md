# OpenStarry — Plan Dependencies & Definition of Done

Established: 2026-02-09

---

## 1. Plan 相依關係圖

```
Plan01 (MVP Alpha Foundation)           ✅ v0.1-alpha  2026-02-04
  ↓
Plan02 (Event-Driven & Safety)          ✅ v0.1.1      2026-02-05
  ↓
Plan03 (Quality & Skill Parser)         ✅ v0.2-alpha  2026-02-05
  ↓
Plan04 (IUI Interface & Guide Plugin)   ✅ v0.2-alpha  2026-02-06
  ↓
Plan05 (Multi-Channel UI & Listener)    ✅ v0.2-beta   2026-02-07
  ↓
  ├── Plan05.1 (Session Isolation)      ✅ v0.2.1-beta  2026-02-10 (Cycle 1)
  ├── Plan05.2 (HTTP SSE)              ✅ v0.2.1-beta  2026-02-10 (Cycle 1)
  ├── Plan05.5-① (Health Check)        ✅ v0.2.1-beta  2026-02-10 (Cycle 1)
  ├── Plan05.5-② (Metrics/Logging)     ✅ v0.2.2-beta  2026-02-11 (Cycle 2)
  ├── Plan05.5-③ (Error Handling)      ✅ v0.2.2-beta  2026-02-11 (Cycle 2)
  ├── Plan05.3 (DevTools)              ✅ → Plan11 (Cycle 12) 完成
  ├── Plan05.4 (E2E Testing)           ✅ → Plan11 (Cycle 12) 完成
  │     ↓
  │   Plan06 Phase 1 (MCP Client)       ✅ v0.3.0-beta  2026-02-11 (Cycle 3)
  │   Plan06 Phase 2 (MCP Server)       ✅ v0.3.1-beta  2026-02-11 (Cycle 4)
  │   Plan06 Phase 3 (Resources+Auth)   ✅ v0.10.0-beta 2026-02-12 (Cycle 15)
  │   Plan06 Phase 4 (Sampling+Ext)     ✅ v0.12.0-beta 2026-02-12 (Cycle 17)
  │     ↓
  │   Plan07 (Runtime Sandbox MVP)      ✅ v0.4        2026-02-11 (Cycle 5)
  │   Plan07.1 (Hardening)              ✅ v0.4.1-beta  2026-02-11 (Cycle 6)
  │   Plan07.2 (Advanced Hardening)     ✅ v0.4.2-beta  2026-02-11 (Cycle 7)
  │     ↓
  │   Plan07.3 (Custom require + audit) ✅ v0.4.3-beta  2026-02-11 (Cycle 8)
  │     ↓
  │   Plan08 (TUI Dashboard MVP)        ✅ v0.5.0-beta  2026-02-11 (Cycle 9)
  │   Plan09 (Interactive TUI)          ✅ v0.5.1-beta  2026-02-12 (Cycle 10)
  │   Plan10 (CLI Foundation & Runner)  ✅ v0.6.0-beta  2026-02-12 (Cycle 11)
  │     ↓
  │   Plan05.3 (DevTools) + 05.4 (E2E) ✅ → Plan11 (Cycle 12) 完成
  │   Plan11 (DevTools & E2E Testing)   ✅ v0.7.0-beta  2026-02-12 (Cycle 12)
  │     ↓
  │   Plan12 (Daemon Mode MVP)          ✅ v0.8.0-beta  2026-02-12 (Cycle 13)
  │     ↓
  │   Plan13 (Seamless Attach)          ✅ v0.9.0-beta  2026-02-12 (Cycle 14)
  │     ↓
  │   Plan14 (Multi-client Attach & Session Management) ✅ v0.11.0-beta 2026-02-12 (Cycle 16)
  │     ↓
  │   Plan15 (SDK Context Extensions & Provider Integration) ✅ v0.13.0-beta 2026-02-12 (Cycle 18)
  │     ↓
  │   Plan16 (Security Hardening & Quality Polish) ✅ v0.14.0-beta 2026-02-12 (Cycle 19)
  │     ↓
  │   Plan17 (Plugin Developer Experience) ✅ v0.15.0-beta 2026-02-12 (Cycle 20)
  │     ↓
  │   Plan18 (Plugin Sync & System Plugin Directory) ✅ v0.16.0-beta 2026-02-12 (Cycle 21)
  │     ↓
  │   Plan19 (Plugin Dependency Wiring & Cross-Plugin Services) ✅ v0.17.0-beta 2026-02-12 (Cycle 22)
  │     ↓
  │   Plan20 (Workflow Engine MVP)            ✅ v0.18.0-beta  2026-02-12 (Cycle 23)
  │     ↓
  │   Plan21 (Web-based Remote Attach)       ✅ v0.19.0-beta  2026-02-12 (Cycle 24)
  │     ↓
  │   Plan22 (Plugin Marketplace MVP)            ✅ v0.20.0-beta  2026-02-13 (Cycle 25)
  │     ↓
  │   Plan23 (Provider Expansion)                ✅ v0.21.0-beta  2026-02-14 (Cycles 27-30)
  │     ↓
  │   Cycles 31-33 (Security + IInferenceProvider + Banner) ✅ v0.22.1-beta  2026-02-16
  │     ↓
  │   Plan24 (Security & Quick Wins Sprint + C5/C6) ✅ v0.23.0-beta  2026-02-26 (Cycle 02-3)
  │     ↓
  │   Plan25 (Event Types & Core Infra + 五蘊 OOP Root Class) ✅ v0.24.0-beta  2026-03-01 (Cycle 02-3)
  │     ↓                    ↘
  │   Plan26 (Vedana 三受     Plan27a (IGearArbiter Types)     ✅ v0.25/0.26 (Cycle 02-3/02-4)
  │    + 我執機制)            Plan27b (Gear Routing Wiring)
  │     ↓                    ↙
  │   Plan28 (IVolition v1 + Safety Hardening)     ✅ v0.28.0-alpha  2026-03-05 (Cycle 02-4)
  │     ↓
  │   Plan32 (Tenet #7 絕對純淨 + Tenet #9 部分修復)  ✅ v0.32.0-alpha  2026-03-11 (Cycle 20260311_cycle1)
  │     ↓
  │   Plan29 (Advanced Security)                   📋 待研究團隊指示
  │     ↓
  │   Plan-AC (Agent 協調層設計文件)                📋 doc only ← 可與 Plan29 並行
  │     ↓
  │   Backlog: Multi-agent Orchestration, Service Mesh Patterns
  │
  └── (Extended Mode: 新 plugin、Web UI、安全稽核)
```

### 關鍵路徑（Critical Path）

```
Plan01-23 + Cycles 31-33 (全部完成 ✅)
  → Plan24 → Plan25 → Plan26/Plan27 → Plan28 → Plan29 / Plan-AC
```

Plan01-28、Cycles 31-33、和 Plan32 全部完成。關鍵路徑為 Plan24 → Plan25 → Plan26/27 (並行) → Plan28 → Plan32 → Plan29。Plan-AC 可與 Plan29 並行。Plan28 原定為 "Doc Alignment + DC-20~22" (doc only)，研究團隊在 cycle02-4 重新定義為 "IVolition v1 + Safety Hardening" (code plan)。Plan32 完成後，所有 Tenet #7「微內核與絕對純淨」的實現全部就位。

---

## 2. Phase 6 計畫依賴 (Plan37-39)

```
Plan37 (Multi-Agent Foundation)
  ├─ ProcessTree + IDaemonControlPlane + ICompositeAgent
  └─ ICommChannel + ITypedListener + Graceful Shutdown
       ↓ (完成)
Plan38 (Multi-Agent Communication Infrastructure)
  ├─ openstarry-channel Hub
  ├─ comm-proxy plugin (ICommChannel 裝飾器)
  ├─ Permission Lattice (路徑/token/信心)
  ├─ Dual Rate Limiter
  └─ IDistributedAlaya (基礎介面)
       ↓ (完成)
Plan39 (AC-7 + Audit Path + Registry Bridge)      ✅ v0.39.0-alpha  2026-04-04
  ├─ W0: Interface Amendments (SyncResult→ExchangeResult, SeedPatch)
  ├─ W1: AC-7 Full Runtime (BijaStore, SeedSignatureService, distributed consensus)
  ├─ W2: Comm Proxy Template Method (split bulkhead)
  ├─ W3: Registry Bridge & IPC (Daemon-authoritative, 4 events, AT-7)
  ├─ W4: Stability Fixes (withChannelGuard, per-cycle clamp reset, B-Modified delta)
  └─ W5: W2-R2 Calibration (CV-1~CV-7 verification)
       ↓ (完成)
Plan40 (Stabilization + Calibration + Security Hardening)  ✅ v0.40.0-alpha  2026-04-06
  ├─ W0: BABBAGE Revisit (mechanism vs policy classification audit)
  ├─ W1: CV-5 Hysteresis Tuning (confidence saturation bounds)
  ├─ W2: Skipped Wave
  ├─ W3: Registry ACL Refresh (token path constraints)
  └─ W4: Security Advisory Fixes (SEC-001 phantom cast mitigation)
       ↓ (完成)
Plan41 (Typed Registry + CV-5 Gear-Arbiter-Dynamic + Late-Joiner Snapshot)  ✅ v0.41.0-alpha  2026-04-07
  ├─ W0: Cleanup (deprecated SyncResult removed)
  ├─ W1: ServiceKey<T> Typed Registry (generic type safety)
  ├─ W2: gear-arbiter-dynamic plugin (CV-5 confidence routing)
  ├─ W3: Fabrication Automation (artifact generation + hash verification)
  └─ W4: Late-Joiner Snapshot (IAlayaSnapshot interface + snapshot writer)
       ↓ (完成 — 2026-04-07)
Plan42 (CV-5 Fix + Stabilization)  ✅ v0.42.0-alpha  2026-04-09
  ├─ W0: Test team code integration + DEV-1b fabrication reporter wiring (8 acceptance tests)
  ├─ W1: Vitest 4.x fix + DEV-1b callback integration (15 integration tests)
  ├─ W2: CV-5 observe mode fix + shadow counting per-gear queues (12 edge case tests)
  └─ W3: Key rotation documentation + external consumer API (2 integration tests)
       ↓ (完成 — 2026-04-09)
Plan43 (COND Backfill + Spec Ratification)  ✅ v0.43.0-alpha  2026-04-10
  ├─ W-1: Fabrication recovery GATE (6 historical plan verification)
  ├─ W0: COND-2 StateTracker spec amendment (Tier 1 ratification)
  ├─ W1-1: COND-3 DynamicArbiterOptions constructor refactor (8 tests)
  ├─ W1-2: COND-4 coldStartGear config injection (6 tests)
  └─ W1-3: COND-1 ILogger dependency injection into CalibrationBridge (9 tests)
       ↓ (完成 — 2026-04-10) / Deferred to Plan44: Phase 3 Shadow + M4a + Perturbation
```

### Phase 6 合規進度

| 宣言 | v0.37.0-alpha | v0.38.0-alpha | v0.39.0-alpha | 目標 |
|-----|--------|--------|--------|------|
| Tenet #6 | CONDITIONAL | CONDITIONAL | **COMPLIANT** | ✅ |
| Tenet #8 | COMPLIANT | COMPLIANT (conditional) | **COMPLIANT** | ✅ |
| Tenet #10 | ~~NON-COMPLIANT~~ COMPLIANT | COMPLIANT | **COMPLIANT** | ✅ |
| **合規率** | 80% (8/10) | 80% (8/10 + B-modified pending) | **100% (10/10)** | ✅ |

### 可並行的工作

| 可並行 | 說明 |
|--------|------|
| Plan26 + Plan27 | Vedana Architecture 與 Lifecycle Hardening 可並行開發 |
| Plan28 (doc only) | 文件修正可與任何程式碼 Plan 並行 |
| Plan-AC + Plan29 | Agent 協調層設計文件可與 Advanced Security 並行 |

---

## 2. Definition of Done（完成定義）

### 2.1 通用 DoD（所有 Plan 適用）

每個 Plan 必須滿足以下**全部條件**才能標記 ✅：

| # | 條件 | 驗證方式 |
|---|------|---------|
| 1 | `pnpm build` 通過（monorepo + plugins） | qa 在 agent_test 驗證 |
| 2 | `pnpm test` 全部通過，且測試數 ≥ 前一輪基線 | qa 報告中的 regression check |
| 3 | `pnpm test:purity` 通過 | qa 報告中的 purity check |
| 4 | Architecture_Spec 中所有項目已實作 | architect Code Review PASS |
| 5 | 五蘊合規（新組件正確歸類） | architect Code Review 確認 |
| 6 | pushInput 模式合規 | architect Code Review 確認 |
| 7 | 無已知安全漏洞 | architect Code Review 確認 |
| 8 | dev log 已寫入 | Coordinator 確認檔案存在 |
| 9 | QA Report 和 Code Review 均為 PASS | Phase 4 判定 |
| 10 | doc-keeper 已更新 Plan 打 ✅ + Iteration_Log | Phase 4 收斂 |
| 11 | Snapshot 已建立 | `scripts/snapshot.sh` 完成 |
| 12 | Lessons Learned 已記錄 | Phase 4 回顧 |

### 2.2 各 Plan 專屬驗收條件

#### Plan05.1: Session Isolation ✅ (Cycle 1, 2026-02-10)
- [x] WebSocket 連線支援 session token / auth (session handshake on connect)
- [x] 多個 client 連線互相隔離（session 資料不交叉）
- [x] 單一 agent instance 可服務多個 session (SessionManager + default session)
- [x] Session 生命週期管理（建立、維持、銷毀）
- [x] 新增測試覆蓋 session isolation 場景 (17 tests)

#### Plan05.2: HTTP SSE ✅ (Cycle 1, 2026-02-10)
- [x] HTTP Server-Sent Events transport plugin 可用 (GET /api/events)
- [x] SSE 連線可接收即時串流回應
- [x] 與現有 HTTP webhook listener 共存不衝突
- [x] 新增測試覆蓋 SSE 場景 (11 tests)

#### Plan05.5-①: Health Check ✅ (Cycle 1, 2026-02-10)
- [x] WebSocket protocol ping/pong + stale connection cleanup
- [x] HTTP SSE heartbeat
- [x] 新增測試覆蓋 (6 tests)

#### Plan05.5-②: Metrics/Logging ✅ (Cycle 2, 2026-02-11)
- [x] MetricsCollector (increment/gauge/getSnapshot/reset)
- [x] Logger.time() (performance.now())
- [x] METRICS_SNAPSHOT event + /metrics command
- [x] Transport 插件結構化日誌遷移
- [x] 新增測試覆蓋 (19 tests)

#### Plan05.5-③: Error Handling ✅ (Cycle 2, 2026-02-11)
- [x] ErrorCode const (12 codes)
- [x] TransportError / SessionError / ConfigError
- [x] ES2022 Error cause chain
- [x] 新增測試覆蓋 (16 tests)

#### Plan06: MCP Protocol (Model Context Protocol) — Phased

**Cycle 3 (Plan06 Phase 1: MCP Client)** ✅ (Cycle 3, 2026-02-11):
- [x] `@openstarry-plugin/mcp-client` plugin 可用
- [x] MCP client mode：連接外部 MCP server，匯入工具 (Tool bridge: MCP tools → ITool)
- [x] MCP Prompt bridge：匯入 MCP prompts 為 SlashCommand
- [x] stdio transport：spawn child process, JSON-RPC over stdin/stdout
- [x] Streamable HTTP transport：POST JSON-RPC + optional SSE
- [x] Config-driven server list (agent.json plugins[].config.servers)
- [x] Slash commands: /mcp-status, /mcp-tools, /mcp-prompts
- [x] 新增測試覆蓋 MCP client 場景 (35 tests)

**Cycle 4 (Plan06 Phase 2: MCP Server)** ✅ (Cycle 4, 2026-02-11):
- [x] `@openstarry-plugin/mcp-server` plugin
- [x] MCP server mode：暴露 ITool/IGuide 為 MCP tools/prompts
- [x] Server transports: stdio + HTTP (http.createServer)
- [x] JSON-RPC 2.0 handler (initialize, tools/list, tools/call, prompts/list, prompts/get)
- [x] Config-driven: server name, version, exposed tools/guides whitelist
- [x] SDK: IPluginContext.tools + IPluginContext.guides (optional, non-breaking)
- [x] SDK: MCP_CLIENT_CONNECTED + MCP_CLIENT_DISCONNECTED events
- [x] 新增測試覆蓋 MCP server 場景 (52 tests)

**Cycle 15 (Plan06 Phase 3: Resources + Auth)** ✅ (Cycle 15, 2026-02-12):
- [x] MCP Resources bridge (resource access protocol, capability negotiation)
- [x] OAuth 2.1 Authorization (client credentials flow, token management)
- [x] Resource metadata handling (resource.list, resource.read)
- [x] Authorization context integration (scope validation, token refresh)
- [x] 新增測試覆蓋 MCP resources + auth 場景 (29 tests)

**Cycle 17 (Plan06 Phase 4: Sampling & Advanced Protocol Extensions)** ✅ (Cycle 17, 2026-02-12):
- [x] MCP Sampling protocol (server requesting LLM completions from client, bidirectional AI-to-AI)
- [x] Sampling handler with depth guard (max 5 levels), rate limiting (10/min)
- [x] MCP Logging protocol (structured logging from server to client)
- [x] Logging handler with 8→4 level mapping, rate limiting (100/sec)
- [x] MCP Roots protocol (file system root declarations client → server)
- [x] Roots handler exposing workingDirectory, change notifications
- [x] Bidirectional transport extensions (client + server stdio transports)
- [x] 8 new SDK event types (MCP_SAMPLING_request, MCP_SAMPLING_callback, MCP_SERVER_LOG, MCP_LOG_LEVEL_CHANGED, MCP_ROOTS_list, MCP_ROOTS_changed)
- [x] 5 new MCP error codes (McpErrorCode -32001 to -32005)
- [x] /mcp-loglevel slash command
- [x] 新增測試覆蓋 sampling/logging/roots/bidirectional scenarios (87 tests)
- **Target Version**: v0.12.0-beta ✅ (Cycle 17 Phase 4 PASS, 2026-02-12)
- **Pre-condition**: Plan06-P3 ✅ (Cycle 15, v0.10.0-beta)
- **Scope**: MCP Sampling & Advanced Protocol Extensions (4 core features)
- **Test Growth**: 807 → 894 (+87 new, +10.8%)
- **Packages**: 18 → 18 (0 new, existing mcp-client/mcp-server extended)
- **Rework Cycles**: 0 (first-pass PASS)
- **Non-blocking Advisories**: 4 deferred (sampling provider integration, roots simplification, HTTP bidirectional transport, log injection sanitization)
- **Status**: ✅ COMPLETE (Phase 4 Convergence PASS, 2026-02-12)

#### Plan15: SDK Context Extensions & Provider Integration ✅ (Cycle 18, 2026-02-12)
- [x] IPluginContext.providers accessor (readonly IProviderRegistry)
- [x] SessionConfig interface with optional allowedPaths field
- [x] ProviderRegistry implementation and core wiring
- [x] SamplingHandler real provider invocation (replaces stub)
- [x] RootsHandler session config allowedPaths integration
- [x] Sandbox RPC for provider discovery (list/get methods)
- [x] tsconfig build fix for test file exclusion
- [x] Non-breaking SDK changes (all backward-compatible)
- [x] 915 total tests (894 baseline + 21 new)
- [x] 18 packages, 77 test files
- [x] Zero regressions, zero rework cycles
- [x] Snapshot created: `share/openstarry_code_iteration/20260212_cycle18/`
- **Target Version**: v0.13.0-beta ✅
- **Status**: ✅ COMPLETE (Phase 4 Convergence PASS, 2026-02-12)

#### Plan16: Security Hardening & Quality Polish ✅ (Cycle 19, 2026-02-12)
- [x] Provider Access Control Whitelist
  - [x] PluginManifest.allowedProviders field (optional, non-breaking)
  - [x] Runtime validation in PluginLoader
  - [x] Deny-by-default security model (empty array = no access)
  - [x] Wildcard `*` support for legitimate provider discovery
  - [x] 8 provider access control tests (4 core + 4 sandbox RPC)
- [x] Session Config Runtime Validation
  - [x] SessionManager.validateAllowedPaths method
  - [x] Path normalization and symlink resolution
  - [x] Subset validation (session paths ⊆ agent allowedPaths)
  - [x] Path traversal prevention (`../` patterns, absolute escapes)
  - [x] Audit logging for validation failures
  - [x] 6 session path validation tests
- [x] Log Injection Sanitization
  - [x] sanitizeLogMessage utility function
  - [x] Newline and control character removal
  - [x] JSON-safe escaping for structured logs
  - [x] Applied to MCP LoggingHandler
  - [x] 6 log injection tests
- [x] Security Test Coverage
  - [x] 20 new security tests (8 provider + 6 session + 6 log)
  - [x] 935 total tests (915 baseline + 20 new, +2.2%)
  - [x] 78 test files (77 baseline + 1 new)
  - [x] Zero regressions
- [x] Technical Debt Resolution
  - [x] Cycle 15: Session config validation → Resolved with validateAllowedPaths
  - [x] Cycle 17: Log injection from MCP servers → Resolved with sanitizeLogMessage
  - [x] Cycle 18: Provider access control advisories → Resolved with allowedProviders
- [x] Microkernel purity maintained (security additions, no core contamination)
- [x] No breaking changes to SDK
- [x] Snapshot created: `share/openstarry_code_iteration/20260212_cycle19/`
- **Target Version**: v0.14.0-beta ✅
- **Rework Cycles**: 1 (missing 4 sandbox RPC tests, resolved)
- **Status**: ✅ COMPLETE (Phase 4 Convergence PASS, 2026-02-12)

#### Plan17: Plugin Developer Experience (DX) ✅ (Cycle 20, 2026-02-12)
- [x] MockHost test utility in `@openstarry/sdk/testing`
  - [x] Full IPluginContext mock (EventBus, sessions, tools, guides, providers, pushInput)
  - [x] Event capture utilities (getEmittedEvents, getInputEvents, clearEvents)
  - [x] In-memory session management (create, get, list, destroy, getDefaultSession)
  - [x] Handler debugging (getHandlerCounts)
  - [x] 22 unit tests
- [x] CreatePluginCommand CLI scaffolding
  - [x] Interactive prompts (name, description, type, author)
  - [x] 6 plugin types mapping to Five Aggregates
  - [x] Template-based generation (package.json, tsconfig, vitest, index.ts, test, README)
  - [x] Plugin name validation (kebab-case only)
  - [x] Overwrite protection (--force flag)
  - [x] 13 unit tests
- [x] SDK subpath export: `@openstarry/sdk/testing`
- [x] Non-breaking changes only (no existing interface modifications)
- [x] Microkernel purity: PASS (no core changes)
- [x] 970 total tests (935 baseline + 35 new, +3.7%)
- [x] 80 test files (78 baseline + 2 new)
- [x] Snapshot created: `share/openstarry_code_iteration/20260212_cycle20/`
- **Target Version**: v0.15.0-beta ✅
- **Rework Cycles**: 0 (first-pass PASS)
- **Status**: ✅ COMPLETE (Phase 4 Convergence PASS, 2026-02-12)

#### Plan07: Runtime Sandbox ✅ (Cycle 5, 2026-02-11)
- [x] Plugin 執行環境隔離（Node.js vm.createContext + worker_threads）
- [x] 資源限制（CPU watchdog deadline timer、記憶體沙箱隔離、檔案存取）
- [x] Plugin 簽章驗證機制（SHA-512 hash + Ed25519 PKI support）
- [x] 新增測試覆蓋 sandbox escape 場景 (68 tests)
- Sandbox 版本：v0.4 (Cycle 5)

#### Plan07.1: Sandbox Hardening ✅ (Cycle 6, 2026-02-11)
- [x] CPU watchdog (deadline timer per RPC call)
- [x] Bidirectional EventBus (worker subscriptions + forwarding)
- [x] Worker restart policies (ON_CRASH, ON_TIMEOUT with exponential backoff)
- [x] Async proxy for worker context
- [x] 新增測試覆蓋 (31 tests)
- Sandbox 版本：v0.4.1-beta (Cycle 6)

#### Plan07.2: Sandbox Advanced Hardening ✅ (Cycle 7, 2026-02-11)
- [x] Static import restrictions (@babel/parser AST analysis, blocklist enforcement)
- [x] Worker pool pre-spawning (piscina connection pool, lazy initialization)
- [x] PKI asymmetric signing (Ed25519 + RSA fallback, backward-compatible format detection)
- [x] Plugin-signer CLI tool (keygen, sign, verify commands)
- [x] 新增測試覆蓋 (55 tests)
- Sandbox 版本：v0.4.2-beta (Cycle 7)
- **Status**: PASS (1 rework cycle: fixed import integration, plugin-signer package, package-name graceful handling)

#### Plan08: TUI Dashboard MVP ✅ (Cycle 9, 2026-02-11)
- [x] TUI Dashboard plugin (Ink v5 + React 18)
- [x] Event-to-state mapping (14 event types → TUI actions)
- [x] Pure state management with React useReducer
- [x] Streaming delta accumulation (APPEND_STREAM/FINALIZE_STREAM)
- [x] Terminal components: Header, ChatArea, EventLog, Footer
- [x] Formatting utilities: truncate, timestamp, status symbols
- [x] 新增測試覆蓋 (53 tests)
- **Version**: v0.5.0-beta
- **Stats**: 524 tests (82 new), 16 packages, 0 regressions

#### Plan09: Interactive TUI (Chat Input + Command Execution) ✅ (Cycle 10, 2026-02-12)
- [x] Text input component (custom useInput, no external deps) in TUI Dashboard
- [x] IListener (色蘊·輸入) implementation for keyboard input
- [x] Push text input to ctx.pushInput() for agent processing
- [x] Command history navigation (arrow up/down, max 50, dedup consecutive)
- [x] Slash command parsing and routing (/quit, /help, /mcp-status, etc.)
- [x] Visual feedback: typing indicator (isPending state), command execution status
- [x] Input mode management (chat mode vs command mode)
- [x] No SDK changes required (IListener + IUI already defined) — VERIFIED
- [x] Single plugin enhancement to @openstarry-plugin/tui-dashboard
- [x] 新增測試覆蓋 interactive input scenarios (35 tests)
- [x] Session management (create/destroy/lifecycle)
- [x] Local echo (optimistic UI updates)
- **Target Version**: v0.5.1-beta ✅
- **Pre-condition**: Plan08 ✅ (Cycle 9)
- **Functional Requirements Met**: 11/11 (100%)
  - [x] Custom text input handling
  - [x] IListener keyboard event integration
  - [x] ctx.pushInput() delegation
  - [x] Command history with dedup
  - [x] Arrow-key navigation
  - [x] Slash command prefix detection
  - [x] Command routing via ctx.pushInput()
  - [x] isPending state tracking
  - [x] Session lifecycle management
  - [x] Input mode toggle (chat/browse)
  - [x] Local echo optimization
- **Technical Requirements Met**: 6/6 (100%)
  - [x] pnpm build PASS (16/16 packages)
  - [x] pnpm test PASS (559/559 tests, 35 new)
  - [x] pnpm test:purity PASS
  - [x] Architecture Spec PASS (no SDK changes)
  - [x] Microkernel purity PASS (zero core contamination)
  - [x] Five Aggregates compliance PASS (IListener + IUI only)
- **Test Count**: 559 total (35 new)
- **Status**: PASS (no rework cycles)

#### Plan10: CLI Foundation & Runner Hardening ✅ (Cycle 11, 2026-02-12)
- [x] CLI subcommand router (start, init, version, help commands)
- [x] `openstarry start` command with --config, --verbose flags
- [x] `openstarry init` interactive config generator (guided setup)
- [x] Enhanced bootstrap with first-run UX (helpful messages, validation)
- [x] Config validation with descriptive error messages
- [x] Runner test suite (bootstrap, config loading, plugin resolution)
- [x] CLI routing tests (command dispatch, subcommand argument parsing)
- [x] No breaking changes to existing functionality
- [x] Backward compatibility: existing `node apps/runner/dist/bin.js` still works
- [x] Hand-rolled argument parser (no external CLI library)
- [x] Interactive init command with Node.js readline
- [x] Config validation with Zod + semantic checks + warnings
- [x] Plugin resolver with error accumulation pattern
- [x] 3-tier config path precedence (CLI flags > env > file > defaults)
- **Target Version**: v0.6.0-beta ✅
- **Pre-condition**: Plan09 ✅ (Cycle 10)
- **Scope**: CLI + runner in core monorepo; dev-core implementation
- **Test Count**: 632 tests (73 new)
- **Rework Cycles**: 1 (3 advisory fixes: --version flag, error messages, bash completion deferred)
- **Status**: ✅ COMPLETE (Phase 4 Convergence PASS)

#### Plan11: DevTools Plugin & E2E Testing Framework ✅ (Cycle 12, 2026-02-12)
- [x] DevTools Plugin (`@openstarry-plugin/devtools`):
  - [x] State inspector component (agent state, metrics, event timeline)
  - [x] Debug console with structured logging
  - [x] Slash commands: /devtools, /metrics, /debug-on, /debug-off
  - [x] Metrics exporter (JSON format)
  - [x] 38 tests delivered (exceeded 35 target)
- [x] E2E Testing Framework (`apps/runner/__tests__/e2e/`):
  - [x] CLI integration tests (init, start, signal handling)
  - [x] Plugin lifecycle tests (multi-plugin, transport initialization)
  - [x] Multi-session behavior tests (session isolation, state isolation)
  - [x] Agent workflow tests (input→loop→tool→response)
  - [x] 40 tests delivered (met 40 target)
- [x] No SDK changes (pure plugin + test additions) — VERIFIED
- [x] Total test count: 670 tests (632 baseline + 38 DevTools + 40 E2E = 710 effective)
- [x] Microkernel purity: PASS (DevTools as plugin, not core) — VERIFIED
- **Target Version**: v0.7.0-beta ✅ (Cycle 12 Phase 4 PASS, 2026-02-12)
- **Pre-condition**: Plan10 ✅ (Cycle 11, v0.6.0-beta)
- **Scope**: Plan05.3 (DevTools, previously deferred) + Plan05.4 (E2E Testing, new)
- **Test Growth**: 632 → 670 (+38 new, +5.7%)
- **Packages**: 16 → 17 (+1: devtools plugin)
- **Rework Cycles**: 1 (3 advisory fixes: README docs added)
- **Status**: ✅ COMPLETE (Phase 4 Convergence PASS, 2026-02-12)

#### Plan12: Daemon Mode MVP (Background Agent Management) ✅ (Cycle 13, 2026-02-12)
- [x] CLI daemon subcommands (daemon-start, daemon-stop, daemon-ps)
- [x] Process spawning with detached flag (child_process.spawn + unref)
- [x] PID file management (~/.openstarry/agents/{agent-id}.pid)
- [x] IPC layer (Unix domain socket + JSON-RPC 2.0)
- [x] Health check RPC (agent.health → {ok, uptime, version})
- [x] Daemon plugin (IPC server, health provider)
- [x] Graceful shutdown with timeout enforcement
- [x] Signal handling (SIGTERM/SIGINT cascade)
- [x] 新增測試覆蓋 (44 tests)
- **Target Version**: v0.8.0-beta ✅ (Cycle 13 Phase 4 PASS, 2026-02-12)
- **Pre-condition**: Plan11 ✅ (Cycle 12, v0.7.0-beta)
- **Scope**: Daemon Mode MVP (background process management, IPC, health checks)
- **Test Growth**: 670 → 714 (+44 new, +6.6%)
- **Packages**: 17 → 18 (+1: daemon plugin)
- **Rework Cycles**: 1 (2 fixes: shutdown timeout enforcement + dead code removal)
- **Status**: ✅ COMPLETE (Phase 4 Convergence PASS, 2026-02-12)

#### Plan13: Seamless Attach (Interactive Terminal Connection) ✅ (Cycle 14, 2026-02-12)
- [x] IPC protocol extensions (agent.attach, agent.input, agent.detach)
- [x] CLI command (openstarry attach [agent-id] with daemon listing and auto-start)
- [x] Terminal I/O proxy (stdin → agent.input RPC, daemon events → stdout/stderr)
- [x] Auto-start feature (spawn daemon on demand if agent.json exists)
- [x] Event forwarder (core.bus → IPC bridge with sessionId filtering)
- [x] Session lifecycle management (create on attach, close on detach, timeout)
- [x] Security & validation (sessionId format, inputType whitelist, agent health check)
- [x] Graceful Ctrl+C handling (SIGINT → detach, not kill daemon)
- [x] 新增測試覆蓋 (33 tests)
- **Target Version**: v0.9.0-beta ✅ (Cycle 14 Phase 4 PASS, 2026-02-12)
- **Pre-condition**: Plan12 ✅ (Cycle 13, v0.8.0-beta)
- **Scope**: Seamless Attach (interactive connection to running daemon agents)
- **Test Growth**: 714 → 747 (+33 new, +4.6%)
- **Packages**: 18 → 18 (0 new, existing CLI + daemon extended)
- **Rework Cycles**: 1 (3 fixes: agentId validation before attach, event timestamp, error codes documentation)
- **Status**: ✅ COMPLETE (Phase 4 Convergence PASS, 2026-02-12)

#### Plan14: Multi-client Attach & Session Management ✅ (Cycle 16, 2026-02-12)
- [x] Multi-client simultaneous attach (multiple terminals to same daemon)
- [x] Session registry (track all active sessions per daemon)
- [x] Event fan-out broadcasting (events to all attached clients)
- [x] Session-aware event routing (sessionId tagging)
- [x] Client connection tracking (heartbeat, disconnect handling)
- [x] Persistent session history (FileSessionPersistence with atomic writes)
- [x] Session state serialization (~/.openstarry/sessions/{agentId}/{sessionId}.json)
- [x] History replay on reattach (last N messages)
- [x] Session expiry (24h TTL, automatic cleanup)
- [x] Tab-completion for slash commands (readline integration)
- [x] Input history persistence (command history per session)
- [x] Backpressure handling (5s slow-client timeout, auto-disconnect)
- [x] IPC protocol extensions (agent.list-clients RPC method)
- [x] Debounced saves (1s interval, configurable)
- [x] 新增測試覆蓋 (15 tests: session persistence, multi-client, tab-completion, backpressure)
- **Target Version**: v0.11.0-beta ✅ (Cycle 16 Phase 4 PASS, 2026-02-12)
- **Pre-condition**: Plan13 ✅ (Cycle 14, v0.9.0-beta) + Plan06-P3 ✅ (Cycle 15, v0.10.0-beta)
- **Scope**: Multi-client Attach & Session Management (5 core features)
- **Test Growth**: 792 → 807 (+15 new, +1.9%)
- **Packages**: 18 → 18 (0 new, existing CLI + daemon extended)
- **Rework Cycles**: 0 (first-pass PASS)
- **Non-blocking Advisories**: 3 deferred (session command stubs, debounce docs, integration tests)
- **Status**: ✅ COMPLETE (Phase 4 Convergence PASS, 2026-02-12)

#### Plan18: Plugin Sync & System Plugin Directory ✅ (Cycle 21, 2026-02-12)
- [x] Plugin scanner utility (scanPluginDirectory, shouldSyncPlugin, syncPlugin, readPluginVersion)
  - [x] Directory traversal with plugin detection
  - [x] Version-aware comparison (package.json parsing)
  - [x] File copying with overwrite control
- [x] Plugin sync CLI command (openstarry plugin sync <source-path>)
  - [x] --verbose flag for progress output
  - [x] --force flag to overwrite existing versions
  - [x] --dry-run flag for preview without side effects
- [x] Three-tier plugin resolution (path → system dir → node_modules)
  - [x] Enhanced plugin-resolver.ts with system directory fallback
  - [x] SystemConfig type with systemPluginDir path
  - [x] Integration with runner bootstrap
- [x] System plugin directory initialization (~/.openstarry/plugins/)
  - [x] Automatic creation on first use
  - [x] Proper permissions and cleanup handling
- [x] Compound command routing (bin.ts: `openstarry plugin sync`)
  - [x] Plugin subcommand with verb routing
  - [x] Plugin sync handler with argument parsing
- [x] bootstrap.ts exports SystemConfig for CLI access
- [x] 新增測試覆蓋 (39 tests: 19 scanner + 10 sync + 5 E2E + 5 resolver)
- **Target Version**: v0.16.0-beta ✅ (Cycle 21 Phase 4 PASS, 2026-02-12)
- **Pre-condition**: Plan17 ✅ (Cycle 20, v0.15.0-beta, 970 tests)
- **Scope**: Plugin Sync MVP (scanner, CLI command, three-tier resolver)
- **Test Growth**: 970 → 1009 (+39 new, +4.0%)
- **Packages**: 18 → 18 (0 new, existing core extended)
- **Rework Cycles**: 0 (first-pass PASS)
- **Non-blocking Advisories**: 2 deferred
  - [ADV-1] Path traversal validation (recommend additional validation for untrusted sources)
  - [ADV-2] Factory extraction dedup (recommend extracting factory pattern to utility)
- **Test Files**: 82 → 83 (+1 new file: sync-e2e.test.ts)
- **Key Deliverables**:
  - `plugin-scanner.ts` (230 LOC) — scanPluginDirectory, shouldSyncPlugin, syncPlugin, readPluginVersion
  - `plugin-sync.ts` (133 LOC) — PluginSyncCommand CLI handler
  - Enhanced `plugin-resolver.ts` (+120 LOC) — Three-tier resolution with system directory fallback
  - Enhanced `bin.ts` (+15 LOC) — Compound `plugin sync` command routing
  - Enhanced `bootstrap.ts` (+1 LOC) — SystemConfig export
- **Architecture Spec**: `share/test/reports/arch_reviews/20260212_cycle21/Architecture_Spec_Cycle21.md`
- **QA Report**: `share/test/reports/qa_results/20260212_cycle21/QA_Plan18.md`
- **Code Review**: `share/test/reports/arch_reviews/20260212_cycle21/Code_Review_Cycle21.md`
- **Dev Log**: `share/test/reports/dev_logs/20260212_cycle21/DevLog_Plan18_Plugin_Sync.md`
- **Snapshot**: `share/openstarry_code_iteration/20260212_cycle21/`
- **Status**: ✅ COMPLETE (Phase 4 Convergence PASS, 2026-02-12)

#### Plan19: Plugin Dependency Wiring & Cross-Plugin Services ✅ (Cycle 22, 2026-02-12)
- [x] IPluginService interface (SDK) — Base interface for plugin services with name/version
- [x] IServiceRegistry interface (SDK) — Service registry with register/get/has/list methods
- [x] ServiceRegistry class (Core) — In-memory implementation with fail-fast collision detection
- [x] IPluginContext.services — Optional service registry accessor on plugin context
- [x] PluginManifest.services/serviceDependencies — Manifest-level dependency declarations
- [x] PluginLoader.loadAll() — Topological sort using Kahn's algorithm for dependency-ordered loading
- [x] ServiceRegistrationError/ServiceDependencyError — Error types for service operations
- [x] 65 new tests — Total tests now 1067 (88 test files)
  - [x] service-registry.test.ts — Coverage of register(), get(), has(), list() operations
  - [x] plugin-loader-services.test.ts — Topological sort, dependency ordering, circular detection
  - [x] service-injection.test.ts — End-to-end service discovery and cross-plugin invocation
- **Files Created**:
  - `packages/sdk/src/types/service.ts` — IPluginService, IServiceRegistry interfaces
  - `packages/core/src/infrastructure/service-registry.ts` — ServiceRegistry implementation
  - `packages/core/__tests__/infrastructure/service-registry.test.ts`
  - `packages/core/__tests__/infrastructure/plugin-loader-services.test.ts`
  - `packages/core/__tests__/e2e/service-injection.test.ts`
- **Files Modified**:
  - `packages/sdk/src/types/plugin.ts` — IPluginContext.services?, PluginManifest extensions
  - `packages/sdk/src/errors/base.ts` — New error types
  - `packages/sdk/src/index.ts` — New exports
  - `packages/core/src/agents/agent-core.ts` — ServiceRegistry instantiation + wiring
  - `packages/core/src/infrastructure/plugin-loader.ts` — loadAll() + topologicalSort()
  - `packages/core/src/sandbox/sandbox-manager.ts` — Optional services field
- **Target Version**: v0.17.0-beta ✅ (Cycle 22 Phase 4 PASS, 2026-02-12)
- **Pre-condition**: Plan18 ✅ (Cycle 21, v0.16.0-beta, 1009 tests)
- **Scope**: Complete remaining Phase 3.1 items: plugin service registry & dependency wiring
- **Test Growth**: 1009 → 1067 (+65 new, +5.8%)
- **Test Files**: 83 → 88 (+5 new test files)
- **Packages**: 18 → 18 (0 new, existing SDK + core extended)
- **Rework Cycles**: 1 (Code Fix: missing has() method + field name alignment)
- **Key Deliverables**:
  - `types/service.ts` — IPluginService, IServiceRegistry interface definitions
  - `service-registry.ts` — In-memory ServiceRegistry with collision detection
  - `plugin-loader.ts` (+85 LOC) — loadAll() method with Kahn's topological sort
  - Enhanced `types/plugin.ts` — IPluginContext.services, PluginManifest enhancements
  - `errors/base.ts` — ServiceRegistrationError, ServiceDependencyError
  - 65 comprehensive service and dependency tests
- **Architecture Spec**: `share/test/reports/arch_reviews/20260212_cycle22/Architecture_Spec_Cycle22.md`
- **QA Report**: `share/test/reports/qa_results/20260212_cycle22/QA_Plan19.md`
- **Code Review**: `share/test/reports/arch_reviews/20260212_cycle22/Code_Review_Cycle22.md`
- **Dev Log**: `share/test/reports/dev_logs/20260212_cycle22/DevLog_Plan19_Dependency_Wiring.md`
- **Snapshot**: `share/openstarry_code_iteration/20260212_cycle22/`
- **Status**: ✅ COMPLETE (Phase 4 Convergence PASS after 1 rework, 2026-02-12)
- **Design Pattern**: Interface-based service registry (approved via Spec Addendum, architecturally superior to string-ID pattern)
- **Functional Equivalence**: VERIFIED — Topological sort correctly orders plugins based on service dependencies, circular detection prevents invalid configurations
- **Implementation Quality**: EXCELLENT — Kahn's algorithm (O(V+E)), zero blocking issues post-rework, comprehensive test coverage (65 tests)

#### Plan20: Workflow Engine MVP ✅ (Cycle 23, 2026-02-12)
- [x] IWorkflowDefinition (YAML structure with name, version, inputs, outputs, steps)
- [x] IWorkflowStep discriminated union (tool, service, llm, command step types)
- [x] Mustache template interpolation (`{{ variable }}` syntax) with HTML escaping disabled
- [x] WorkflowEngine class (sequential execution, LRU cache 100 entries, event emission)
- [x] Tool step executor (invoke ITool from registry, argument interpolation)
- [x] Service step executor (invoke cross-plugin services via IServiceRegistry)
- [x] LLM step executor (direct IProvider streaming API with text_delta event collection)
- [x] Command step executor (placeholder, throws CommandStepNotSupportedError in MVP)
- [x] Workflow YAML loader (file-based load with validation)
- [x] Event system (WORKFLOW_STARTED, STEP_STARTED, STEP_COMPLETED, STEP_FAILED, WORKFLOW_COMPLETED events)
- [x] Error hierarchy (WorkflowLoadError, WorkflowExecutionError, VariableInterpolationError, CommandStepNotSupportedError)
- [x] Zod schema validation with discriminated union matching TypeScript types
- [x] 37 comprehensive tests (schema, interpolation, engine, 4 executors, e2e integration)
- **Files Created**:
  - `src/index.ts` — Plugin factory (createWorkflowEnginePlugin)
  - `src/errors.ts` — 4 error classes
  - `src/types/workflow.ts` — Workflow type definitions and event payloads
  - `src/types/mustache.d.ts` — Module declaration for mustache
  - `src/schema/workflow-schema.ts` — Zod schema with discriminated union
  - `src/engine/interpolate.ts` — Mustache interpolation with escaping disabled
  - `src/engine/workflow-engine.ts` — WorkflowEngine class
  - `src/engine/executors/*.ts` — 4 executor implementations
  - `src/service/workflow-service.ts` — IWorkflowService implementation
  - `src/tool/workflow-tool.ts` — workflow:execute ITool
  - `src/command/workflow-command.ts` — /workflow slash command
- **Target Version**: v0.18.0-beta ✅ (Cycle 23 Phase 4 PASS, 2026-02-12)
- **Pre-condition**: Plan19 ✅ (Cycle 22, v0.17.0-beta, 1067 tests)
- **Scope**: Workflow Engine MVP (declarative YAML workflows, tool/service/LLM orchestration)
- **Test Growth**: 1067 → 1104 (+37 new, +3.5%)
- **Test Files**: 88 → 96 (+8 new test files)
- **Packages**: 18 → 18 (0 new, pure plugin implementation)
- **Rework Cycles**: 0 (first-pass PASS, but post-submission user-driven spec rewrite)
- **Spec Divergence**: MAJOR — User rewrote implementation with different step types (service, llm instead of transform, condition, prompt), Mustache instead of `$variable`, LRU instead of file persistence. Architect approved PASS despite divergence due to superior pragmatic design and code quality.
- **Key Deliverables**:
  - `workflow-engine.ts` (250 LOC) — Sequential execution engine with LRU cache
  - `interpolate.ts` (50 LOC) — Mustache template interpolation
  - 4 executors (tool, service, llm, command) — ~60 LOC each, specialized step execution
  - `workflow-service.ts` (150 LOC) — YAML loading and validation
  - 8 test files (37 tests) — Comprehensive coverage of all step types and integration scenarios
- **Architecture Spec**: `share/test/reports/arch_reviews/20260212_cycle23/Architecture_Spec_Plan20_Workflow_Engine_MVP.md`
- **QA Report**: `share/test/reports/qa_results/20260212_cycle23/QA_Report_Cycle23_v2.md`
- **Code Review**: `share/test/reports/arch_reviews/20260212_cycle23/Architecture_Review_Cycle23_v2.md`
- **Dev Log**: `share/test/reports/dev_logs/20260212_cycle23/DevLog_Plan20_Workflow_Engine.md`
- **Snapshot**: `share/openstarry_code_iteration/20260212_cycle23/`
- **Status**: ✅ COMPLETE (Phase 4 Convergence PASS, 2026-02-12, spec rewrite approved)
- **Design Pattern**: Factory pattern (createWorkflowEnginePlugin), Five Aggregates (ITool primary), microkernel purity (zero SDK/Core changes)
- **Functional Equivalence**: VERIFIED — All 4 step types functional and tested, Mustache interpolation handles nested objects/arrays, sequential execution preserves step output chaining
- **Implementation Quality**: EXCELLENT — Clean architecture, strong type safety (discriminated unions), comprehensive error handling, zero core modifications
- **Future Enhancements** (Plan22+):
  - README documentation (step types, Mustache syntax, examples)
  - Configurable LRU cache size (manifest or context config)
  - Conditional logic (condition step type or service-based branching)
  - Persistent execution history (file-based or database)
  - Command step support (currently MVP-blocked with clear NotSupportedError)

#### Plan21: Web-based Remote Attach ✅ (Cycle 24, 2026-02-12)
- [x] **@openstarry-plugin/http-static** (3 files, 12 tests)
  - [x] Static file serving for SPA assets (HTML, CSS, JS)
  - [x] MIME type detection
  - [x] Path traversal prevention
  - [x] ETag/304 Not Modified caching support
- [x] **@openstarry-plugin/web-ui** (4 files, 18 tests)
  - [x] Browser-based agent control dashboard
  - [x] Session listing and attachment interface
  - [x] Workflow execution monitoring
  - [x] Real-time WebSocket event updates
- [x] **@openstarry-plugin/transport-websocket** (enhanced, 32 tests)
  - [x] Token-based authentication with validation
  - [x] CORS header support (Origin, Credentials validation)
  - [x] Proxy header forwarding (X-Forwarded-For, X-Forwarded-Proto, X-Real-IP)
- [x] 62 new tests (http-static: 12, web-ui: 18, transport-websocket: 32)
- **Files Created**:
  - `@openstarry-plugin/http-static/` — Static file server plugin
  - `@openstarry-plugin/web-ui/` — Browser dashboard UI plugin
- **Files Enhanced**:
  - `@openstarry-plugin/transport-websocket/` — Auth, CORS, proxy support
- **Target Version**: v0.19.0-beta ✅ (Cycle 24 Phase 4 PASS, 2026-02-12)
- **Pre-condition**: Plan20 ✅ (Cycle 23, v0.18.0-beta, 1104 tests)
- **Scope**: Web-based Remote Attach (HTTP static server, web UI dashboard, enhanced WebSocket transport)
- **Test Growth**: 1104 → 1132 (+28 net, +2.5%)
  - Baseline maintained at 1104 tests
  - New tests added: 62
  - Net adjustment: +28 (some baseline tests refactored)
- **Test Files**: 96 → 97 (+1 new test file)
- **Packages**: 18 core (unchanged), 10 plugins (2 new: http-static, web-ui; 1 enhanced: transport-websocket)
- **Rework Cycles**: 0 (first-pass PASS)
- **Non-blocking Issues** (Deferred to Future Cycles):
  - [NBI-1] README files for http-static and web-ui (documentation phase)
  - [NBI-2] Advanced HTTP caching strategies (Plan22+)
  - [NBI-3] Web UI mobile responsiveness (Phase 2 enhancement)
- **Key Deliverables**:
  - `http-static-plugin/` (3 source + 2 test files) — Static file server
  - `web-ui-plugin/` (4 source + 3 test files) — Browser dashboard
  - `transport-websocket/` (enhanced, +32 tests) — Auth + CORS + proxy headers
- **Architecture Spec**: `share/test/reports/arch_reviews/20260212_cycle24/Architecture_Spec_Cycle24.md`
- **QA Report**: `share/test/reports/qa_results/20260212_cycle24/QA_Report_Cycle24.md`
- **Code Review**: `share/test/reports/arch_reviews/20260212_cycle24/Code_Review_Cycle24.md`
- **Dev Log**: `share/test/reports/dev_logs/20260212_cycle24/DevLog_Plan21.md`
- **Snapshot**: `share/openstarry_code_iteration/20260212_cycle24/`
- **Status**: ✅ COMPLETE (Phase 4 Convergence PASS, 2026-02-12)
- **Design Pattern**: Factory pattern (createHttpStaticPlugin, createWebUiPlugin), Five Aggregates (IUI for dashboard, IListener for HTTP, IProvider for session service)
- **Functional Equivalence**: VERIFIED — Static file serving supports MIME types and ETag, web UI communicates via WebSocket with auth, proxy headers properly forwarded
- **Implementation Quality**: EXCELLENT — Zero core modifications, comprehensive security gates (path traversal, token auth, CORS validation), test coverage exceeds targets
- **Future Enhancements** (Plan22+):
  - Mobile-responsive UI layout
  - Advanced caching headers (Cache-Control, ETag generation)
  - Plugin marketplace integration
  - Multi-agent orchestration dashboard

#### Plan28: IVolition v1 + Safety Hardening ✅ (Cycle 02-4, 2026-03-05)
- [x] **Wave 1 — SDK Type Foundations** (12 tests)
  - [x] DeliberationContext interface (routeResult + actionHistory)
  - [x] PlanDeliberationInput + ActionDeliberationInput extended with optional deliberationContext
  - [x] RouteResult.riskCategory (propagated from arbiter)
  - [x] VedanaEmergencyConfig + DEFAULT_VEDANA_EMERGENCY_CONFIG
  - [x] MohaConfig + DEFAULT_MOHA_CONFIG
  - [x] PluginHooks.volition (single slot, last plugin wins)
- [x] **Wave 2 — Core Infrastructure** (10 tests)
  - [x] ManoAggregator riskCategory propagation to RouteResult
  - [x] SafetyMonitor.postRouteCheck() — v1 passthrough
  - [x] SEC-027 fix: SHA-256 hash of workingDirectory for sessionId
- [x] **Wave 3 — VedanaEmergency + Moha** (15 tests)
  - [x] vedana-emergency.ts: pure stateful function (createVedanaEmergencyState + checkVedanaEmergency)
  - [x] Moha.updateFromAction() with MohaConfig (diminishing marginal returns formula)
  - [x] vijnana/index.ts barrel exports
- [x] **Wave 4 — ExecutionLoop Wiring** (12 tests)
  - [x] postRouteCheck wiring after Phase 2.5 route()
  - [x] DeliberationContext flow to deliberatePlan + deliberateAction
  - [x] Plugin-loader volition registration (getVolition() getter)
  - [x] Agent-core volition DI (neutral klesha/vedana wrapper) [Y3]
  - [x] VedanaEmergency → ManoAggregator wiring (vedanaFn + thresholdBoost) [R1]
- [x] **Wave 5 — volition-rule-engine plugin** (18 tests)
  - [x] Three-layer rule engine: hard (śīla) > soft (upāya) > heuristic (smṛti)
  - [x] Hard rule = MUST audit, 非 MUST veto [Y2]
  - [x] riskCategory undefined fallback via defaultRiskCategory [Y1]
- [x] 研究團隊 6 項修正建議 (R1/R2/Y1-Y4) 全部落實
- **Target Version**: v0.28.0-alpha ✅
- **Pre-condition**: Plan27b ✅ (v0.27.0-beta, 1654 tests)
- **Test Growth**: 1654 → 1722 (+68 new, +4.1%)
- **Packages**: 29 → 30 (+1: volition-rule-engine)
- **Rework Cycles**: 0 (first-pass PASS)
- **Status**: ✅ COMPLETE

#### Plan37: Phase 6 Multi-Agent Foundation ✅ (Cycle 03-1, 2026-03-24)

**Scope**: 4 Waves, 15 Components (C1-C15):
- W1: BUG-3 fix (worst-risk per-tool audit, symmetric clamp, must-invoke)
- W2: ICommChannel + CommChannelRegistry + PipelineChannel + IMcpTransport + Process Tree
- W3: AC-6 ITypedListener (front-five senses) + CommunicationConfig + MessageRouter
- W4: EventBridge + GlobalServiceRegistry + Graceful Shutdown

**Specification**: Doc 57_Multi_Agent_Communication_Interface_Spec.md

**Key Requirements**:
- [x] ICommChannel interface (bidirectional message passing) — SDK frozen
- [x] IDaemonControlPlane interface (daemon lifecycle) — SDK frozen
- [x] ICompositeAgent interface (multi-agent aggregation) — SDK frozen
- [x] ITypedListener interface (sensory type mapping) — SDK frozen
- [x] CommMessage schema with source/dest/payload/metadata
- [x] CommChannelRegistry (discovery and lifecycle management)
- [x] PipelineChannel plugin (composition and routing) — moved to openstarry_plugin/
- [x] ProcessTree (parent-child agent relationships)
- [x] MessageRouter (intelligent message routing)
- [x] EventBridge (event federation across agents)
- [x] GlobalServiceRegistry (shared service discovery)
- [x] GracefulShutdown (coordinated termination, max gracePeriodMs = 300000)
- [x] BUG-3 fix (per-tool audit mechanism + symmetric clamp enforcement)
- [x] 9 new baseline rules (#31-#39) documented

**Test Coverage**:
- [x] 1998 → 2097 tests (+99 new, +5.0%)
- [x] 196 test files, all green, 3 skipped
- [x] 0 new purity violations
- [x] 3 Code Fixes resolved (all in Phase 2)

**Security Audit** (2026-03-24):
- [x] SEC-001 HIGH fixed (daemon drain-evasion in `handleSpawnChild`)
- [x] SEC-002 through SEC-008 logged as advisory (Medium/Low/Info)
- [x] Secrets leak scan: PASS (no hardcoded credentials)
- [x] Personal info leak scan: PASS (no PII exposed)

**Simulation** (Release Verification):
- [x] Release build: v0.37.0-alpha compiled successfully
- [x] Release test suite: 2097 tests passed (196 files)
- [x] Release artifact: `release/cycle03-1_v0.37.0-alpha/` created
- [x] Snapshot: `share/openstarry_code_iteration/20260324_cycle03-1_openstarry/` persisted

**Packages** (38 total, +1 plugin):
- `@openstarry/sdk`: ICommChannel, IDaemonControlPlane, ICompositeAgent, ITypedListener
- `@openstarry/core`: CommChannelRegistry, ProcessTree, GracefulShutdown, EventBridge
- `@openstarry/shared`: MessageRouter utility
- `@openstarry-plugin/comm-pipeline`: PipelineChannel (NEW)
- `@openstarry-plugin/standard-listener-typed`: TypedListener (NEW)
- `@openstarry-plugin/mcp-transport`: IMcpTransport extension (updated)

**Target Version**: v0.37.0-alpha ✅
**Pre-condition**: Plan35 ✅ (v0.35.0-alpha, 1917 tests)
**Test Growth**: 1917 → 2094 (+177 net from all prior fixes/Plan36, +9.2% from Plan35)
**Packages**: 32 → 33 (+1 plugin)
**Rework Cycles**: 0 (first-pass PASS)
**Tenet Compliance**: 8 COMPLIANT + 2 Advancing (#3 AC-6, #10 Fractal)
**SOP Completion**:
- [x] Phase 0 — Planning: ✅ COMPLETE
- [x] Phase 1 — Design: ✅ COMPLETE (Doc 57 frozen)
- [x] Phase 2 — Implementation: ✅ COMPLETE (sec-001 fixed)
- [x] Phase 3 — Verification: ✅ COMPLETE (QA + Code Review PASS)
- [x] Phase 4 — Convergence: ✅ COMPLETE (Security audit PASS, Simulation PASS)
- [x] Release SOP: ✅ COMPLETE (v0.37.0-alpha released, 2026-03-24)
**Status**: ✅ COMPLETE (Full Release SOP PASS, 2026-03-24)

#### Plan38: Multi-Agent Communication Infrastructure ✅ (Cycle 03-2, 2026-03-28)

**Scope**: 5 Waves (W0-W4), +35th plugin:
- W0: openstarry-channel (standalone multi-agent comm hub, AgentRegistry, 6 L1 Tools, 7-step crash handling)
- W1: comm-proxy plugin (ICommChannel decorator, L2 Circuit Breaker, L3 Bulkhead, L5 Timeout Hierarchy)
- W2: Permission Lattice (F-5, 3 dimensions: path, token budget, confidence ceiling)
- W3: Dual Rate Limiter (per-agent + per-target token bucket)
- W4: IDistributedAlaya (AC-7, type-only, frozen)
- Security: SEC-002/003/005/007/008 resolution

**Specification**: Doc 57_Multi_Agent_Communication_Interface_Spec.md (extended)

**Key Requirements**:
- [x] openstarry-channel app with AgentRegistry + health state machine (HEALTHY→DEGRADED→UNREACHABLE→TERMINATED)
- [x] 6 L1 Tools (register_agent, deregister_agent, send_message, broadcast, list_agents, get_agent_status)
- [x] 7-step crash handling with timeout-triggered termination
- [x] comm-proxy plugin with L2 Circuit Breaker (per-target, configurable threshold)
- [x] L3 Bulkhead (per-target connection pool, no cross-contamination)
- [x] L5 Timeout Hierarchy (request < heartbeat < grace)
- [x] Permission Lattice (SpawnConstraints: path subset, token budget, confidence ceiling)
- [x] Cascading termination (child→parent chain)
- [x] Dual Rate Limiter (per-agent + per-target token buckets, mutex-protected refill)
- [x] IDistributedAlaya (type-only interface for distributed consciousness)
- [x] PID identity verification (fail-closed)
- [x] Path traversal prevention (pre-seeded)
- [x] traceDepth validation (pre-seeded)
- [x] PipelineChannel IPC wired to MessageRouter
- [x] CommMessage metadata limits (32 entries, 1024 bytes)

**Test Coverage**:
- [x] 2097 → 2255 tests (+158 new, +7.5%)
- [x] 202 test files, all green
- [x] 0 new purity violations (microkernel maintained)
- [x] 5 Phase 3 FAIL items (1 CRITICAL, 1 HIGH, 3 MEDIUM) — all fixed in Rework 1

**Security Audit** (2026-03-28):
- [x] SEC-002 (PID identity verification): Fixed
- [x] SEC-003 (Path traversal prevention): Fixed
- [x] SEC-005 (traceDepth validation): Fixed
- [x] SEC-007 (PipelineChannel IPC): Fixed
- [x] SEC-008 (CommMessage metadata limits): Fixed
- [x] 0 HIGH/CRITICAL advisories, 0 regressions

**Packages** (39 total, +1 app, +1 plugin):
- `apps/channel`: AgentRegistry, health state machine, 6 L1 Tools, crash handler
- `@openstarry-plugin/comm-proxy`: Circuit breaker, bulkhead, timeout decorator
- `@openstarry/sdk`: ICommProxy, SpawnConstraints, IPermissionLattice, IDualRateLimiter, IDistributedAlaya
- `@openstarry/core`: PermissionLattice, DualRateLimiter, AgentRegistryEntry (updated)
- `@openstarry/shared`: MessageRouter extensions (broadcast, routing tables)

**Target Version**: v0.37.0-alpha (unchanged — Standard SOP, not Release SOP) ✅
**Pre-condition**: Plan37 ✅ (v0.37.0-alpha, 2097 tests)
**Test Growth**: 2097 → 2255 (+158 net from Plan38, +7.5%)
**Packages**: 38 → 39 (+1 app, +1 plugin)
**Rework Cycles**: 1 (Phase 3 rework fixed 5 FAIL items)
**Tenet Compliance**: 8 COMPLIANT + 2 Advancing (#2 plugin, #7 purity maintained)
**SOP Completion**:
- [x] Phase 0 — Planning: ✅ COMPLETE
- [x] Phase 1 — Design: ✅ COMPLETE (Doc 57 extended)
- [x] Phase 2 — Implementation: ✅ COMPLETE
- [x] Phase 3 — Verification: ✅ COMPLETE (1 rework: 5 FAIL → PASS)
- [x] Phase 4 — Convergence: ✅ COMPLETE (Security audit PASS, no escalations)
**Status**: ✅ COMPLETE (Standard SOP PASS, 2026-03-28)

#### Plan39: AC-7 + Audit Path + Registry Bridge ✅ (Cycle 03-3, 2026-04-04)

**Scope**: 6 Waves (W0-W5), +56 new tests:
- W0: Interface Amendments (SyncResult→ExchangeResult, SeedPatch)
- W1: AC-7 Full Runtime (BijaStore, SeedSignatureService, distributed consensus)
- W2: Comm Proxy Template Method (split bulkhead isolation)
- W3: Registry Bridge & IPC (Daemon-authoritative model, 4 new events, AT-7)
- W4: Stability Fixes (withChannelGuard, per-cycle clamp reset, B-Modified delta)
- W5: W2-R2 Calibration (CV-1~CV-7 verification)

**Target Version**: v0.39.0-alpha ✅
**Pre-condition**: Plan38 ✅ (v0.37.0-alpha, 2255 tests)
**Test Growth**: 2255 → 2319 (+64 net, +2.8%)
**Packages**: 39 → 39 (0 new)
**Rework Cycles**: 0 (first-pass PASS)
**Tenet Compliance**: 10 COMPLIANT (all tenets now compliant)
**Status**: ✅ COMPLETE (Release SOP PASS, 2026-04-04)

#### Plan40: Stabilization + Calibration + Security Hardening ✅ (Cycle 03-4, 2026-04-06)

**Scope**: 5 Waves (W0-W4), BABBAGE revisit + CV-5 tuning:
- W0: BABBAGE Classification Audit (mechanism vs policy, SeedSignatureService classification)
- W1: CV-5 Hysteresis Tuning (WIENER constant calibration, confidence saturation bounds)
- W2: Skipped Wave (pre-allocated in spec, deferred)
- W3: Registry ACL Refresh (token path constraints validation)
- W4: Security Advisory Fixes (SEC-001 phantom cast mitigation, SEC-002 HMAC key in heap)

**Target Version**: v0.40.0-alpha ✅
**Pre-condition**: Plan39 ✅ (v0.39.0-alpha, 2319 tests)
**Test Growth**: 2319 → 2319 (0 new, +0%)
**Packages**: 39 → 39 (0 new)
**Rework Cycles**: 0 (first-pass PASS)
**Tenet Compliance**: 10/0/0 maintained
**Status**: ✅ COMPLETE (Standard SOP PASS, 2026-04-06)

#### Plan41: Typed Registry + CV-5 Gear-Arbiter-Dynamic + Late-Joiner Snapshot ✅ (Cycle 03-5, 2026-04-07)

**Scope**: 5 Waves (W0-W4), 31 acceptance criteria, 13 constraints:
- W0: Cleanup (deprecated SyncResult removed, 1 interface amendment)
- W1: ServiceKey<T> Typed Registry (generic type-safe wrapper, registry<T> updates, backward compatibility)
- W2: gear-arbiter-dynamic Plugin (CV-5 confidence routing, dynamic confidence calculation, 5 gear routes)
- W3: Fabrication Automation (artifact generation + hash verification, Rule #52 enforcement)
- W4: Late-Joiner Snapshot (IAlayaSnapshot interface, snapshot writer, scenario tests)

**Deferred to Plan42**: 
- IDistributedAlaya external consumer pattern
- IPC encryption (AT-4a)
- Key rotation design
- IRegistryEventBus freeze (conditional)
- Daemon-authoritative architecture
- Attestation tokens

**Deferred to Plan43**:
- HMAC key rotation implementation
- Plan C (tool-level routing)
- Dynamic alpha calculation

**Target Version**: v0.41.0-alpha ✅
**Pre-condition**: Plan40 ✅ (v0.40.0-alpha, 2319 tests)
**Test Growth**: 2319 → 2350 (+31 net, +1.3%)
**Test Files**: 220 → 221
**Build Projects**: 43
**Plugins**: 37 → 38 (+1 gear-arbiter-dynamic)
**Rework Cycles**: 0 (Phase 3 in-phase rework: 5 FAIL items fixed)
**Compliance**: 10/0/0 maintained
**Security Audit** (Phase 3.5): 0 Critical, 0 High, 2 Low advisory
- SEC-001 (phantom type cast): Design prevents runtime vulnerability
- SEC-002 (HMAC key in heap): Mitigated by deferral to Plan42 key rotation
**ENG-FAB Status**: 
- ✅ ENG-FAB-1a/1b/1c all PASS
- ✅ Rule #52 satisfied (221 test files in snapshot)
**DoD Checklist**:
- [x] All 31 acceptance criteria PASS
- [x] All 13 constraints PASS
- [x] W0-W4 completion verified
- [x] ServiceKey<T> type safety verified
- [x] CV-5 routing enforcement verified
- [x] Late-joiner snapshot scenario tests green
- [x] 2350 tests passing, 0 regressions
- [x] 0 purity violations
- [x] Phase 3 code review: 5 FAIL items fixed
- [x] Phase 3.5 security audit: PASS
- [x] Phase 4 convergence: PASS
**Status**: ✅ COMPLETE (Release SOP PASS, 2026-04-07)

#### Plan42: CV-5 Fix + Stabilization ✅ (Cycle 03-6, 2026-04-09)

**Scope**: 4 Waves (W0-W3), 35 acceptance criteria, 13 constraints:
- W0: Test team code integration + DEV-1b fabrication reporter wiring (8 acceptance tests)
- W1: Vitest 4.x fix + DEV-1b callback integration (15 integration tests)
- W2: CV-5 observe mode fix + shadow counting per-gear queues (12 edge case tests)
- W3: Key rotation documentation + external consumer API (2 integration tests)

**Deferred to Plan43**:
- Shadow decision computation
- Key rotation implementation
- IRegistryEventBus freeze (conditional)
- M4a baseline

**Spec Addenda** (Phase 3 resolved, no re-freeze required):
- COND-1: Method naming (onStateModify callback pattern)
- COND-2: Constructor form (fabrication reporter optional load)
- COND-3: Config exposure (CV5_THRESHOLDS parameterization)
- COND-4: Logger callback (audit event publishing)

**Target Version**: v0.42.0-alpha ✅
**Pre-condition**: Plan41 ✅ (v0.41.0-alpha, 2350 tests)
**Test Growth**: 2350 → 2375 (+25 net, +1.1%)
**Test Files**: 221 → 224 (+3 files)
**Build Projects**: 43 (44 + 1 optional plugin)
**Plugins**: 38 + 1 optional fabrication-reporter
**Rework Cycles**: 0 (Phase 3 in-phase resolution: 4 issues fixed without separate rework)
**Compliance**: 10/0/0 maintained
**Security Audit** (Phase 3.5): CONDITIONAL → PASS (1 MEDIUM SEC-001 fixed: gitignore assertion-coverage.json)
- A-9 integration verification PASS (assertion-coverage.json 63KB generated)
- Rule #52 PASS (224 test files in snapshot)
**ENG-FAB Status**:
- ✅ ENG-FAB-1a artifact created (Engineering_Delivery_Cycle_Plan42.md)
- ✅ All constraints verified PASS
- ✅ Fabrication lock confirmed (code changes before artifact approval)
**Root Cause Analysis**:
- RC-1 (W0): Payload extraction incomplete (missing seed hash in observer payload)
- RC-2 (W2): Priority deadlock via global queue → per-gear queues solution
- RC-3 (W2): Hardcoded gear=1 threshold → CV5_THRESHOLDS config table
- DEV-1b: Phantom integration pattern (code + tests exist but never invoked by host system)
**DoD Checklist**:
- [x] All 35 acceptance criteria PASS (9 CV-5 + 3 W1)
- [x] All 13 constraints PASS
- [x] W0-W3 completion verified
- [x] A-9 fabrication reporter wiring verified
- [x] CV-5 per-gear queues verified
- [x] Observe mode read-only guard verified
- [x] External consumer API implemented
- [x] 2375 tests passing, 0 regressions
- [x] 0 purity violations
- [x] Phase 3 code review: 4 findings fixed same phase
- [x] Phase 3.5 security audit: PASS
- [x] Phase 4 convergence: PASS
- [x] Rules #54, #55, #56 documented
**Status**: ✅ COMPLETE (Release SOP PASS, 2026-04-09)

---

## 3. 迭代排程建議

| 迭代 | 涵蓋 Plans | 目標版本 | 狀態 |
|------|-----------|---------|------|
| Cycle 1 | Plan05.1 + 05.2 + 05.5-① | v0.2.1-beta | ✅ 完成 (2026-02-10, 118 tests) |
| Cycle 2 | Plan05.5-② + 05.5-③ | v0.2.2-beta | ✅ 完成 (2026-02-11, 165 tests) |
| Cycle 3 | Plan06 Phase 1 (MCP Client) | v0.3.0-beta | ✅ 完成 (2026-02-11, 200 tests) |
| Cycle 4 | Plan06 Phase 2 (MCP Server) | v0.3.1-beta | ✅ 完成 (2026-02-11, 252 tests) |
| Cycle 5 | Plan07 (Sandbox MVP) | v0.4 | ✅ 完成 (2026-02-11, 320 tests) |
| Cycle 6 | Plan07.1 (Sandbox Hardening) | v0.4.1-beta | ✅ 完成 (2026-02-11, 351 tests) |
| Cycle 7 | Plan07.2 (Advanced Hardening) | v0.4.2-beta | ✅ 完成 (2026-02-11, 407 tests, 1 rework) |
| Cycle 8 | Plan07.3 (Custom require + audit) | v0.4.3-beta | ✅ 完成 (2026-02-11, 442 tests) |
| Cycle 9 | Plan08 (TUI Dashboard MVP) | v0.5.0-beta | ✅ 完成 (2026-02-11, 524 tests) |
| Cycle 10 | Plan09 (Interactive TUI) | v0.5.1-beta | ✅ 完成 (2026-02-12, 559 tests) |
| Cycle 11 | Plan10 (CLI Foundation & Runner) | v0.6.0-beta | ✅ 完成 (2026-02-12, 632 tests, 1 rework) |
| Cycle 12 | Plan11 (DevTools & E2E Testing) | v0.7.0-beta | ✅ 完成 (2026-02-12, 670 tests, 1 rework) |
| Cycle 13 | Plan12 (Daemon Mode MVP) | v0.8.0-beta | ✅ 完成 (2026-02-12, 714 tests, 1 rework) |
| Cycle 14 | Plan13 (Seamless Attach) | v0.9.0-beta | ✅ 完成 (2026-02-12, 747 tests, 1 rework) |
| Cycle 15 | Plan06-P3 (MCP Resources + OAuth) | v0.10.0-beta | ✅ 完成 (2026-02-12, 792 tests) |
| Cycle 16 | Plan14 (Multi-client Attach & Session Management) | v0.11.0-beta | ✅ 完成 (2026-02-12, 807 tests) |
| Cycle 17 | Plan06-P4 (MCP Sampling & Advanced Protocol Extensions) | v0.12.0-beta | ✅ 完成 (2026-02-12, 894 tests) |
| Cycle 18 | Plan15 (SDK Context Extensions & Provider Integration) | v0.13.0-beta | ✅ 完成 (2026-02-12, 915 tests) |
| Cycle 19 | Plan16 (Security Hardening & Quality Polish) | v0.14.0-beta | ✅ 完成 (2026-02-12, 935 tests, 1 rework) |
| Cycle 20 | Plan17 (Plugin Developer Experience) | v0.15.0-beta | ✅ 完成 (2026-02-12, 970 tests) |
| Cycle 21 | Plan18 (Plugin Sync & System Plugin Directory) | v0.16.0-beta | ✅ 完成 (2026-02-12, 1009 tests) |
| Cycle 22 | Plan19 (Plugin Dependency Wiring & Cross-Plugin Services) | v0.17.0-beta | ✅ 完成 (2026-02-12, 1067 tests, 1 rework) |
| Cycle 23 | Plan20 (Workflow Engine MVP) | v0.18.0-beta | ✅ 完成 (2026-02-12, 1104 tests, spec rewrite) |
| Cycle 24 | Plan21 (Web-based Remote Attach) | v0.19.0-beta | ✅ 完成 (2026-02-12, 1132 tests, +28 net) |
| Cycle 25 | Plan22 (Plugin Marketplace MVP) | v0.20.0-beta | ✅ 完成 (2026-02-13, 1330 tests) |
| Cycles 27-30 | Plan23 (Provider Expansion) | v0.21.0-beta | ✅ 完成 (2026-02-14, 1385 tests) |
| Cycle 31 | Security Advisory Fixes | v0.21.1-beta | ✅ 完成 (2026-02-15, 1404 tests) |
| Cycle 32 | IInferenceProvider + Per-session + 宣言合規 | v0.22.0-beta | ✅ 完成 (2026-02-15, 1451 tests) |
| Cycle 33 | Banner 動態化 + Provider Auto-Config | v0.22.1-beta | ✅ 完成 (2026-02-16, 1451 tests) |
| Plan24 | Plan24 (Security & Quick Wins Sprint + C5/C6) | v0.23.0-beta | ✅ 完成 (2026-02-18, 1475 tests) |
| Plan25 | Plan25 (Event Type System & Core Infra + 五蘊 OOP Root Class) | v0.24.0-beta | ✅ 完成 (2026-02-18, 1496 tests) |
| Plan25v2 | Plan25v2 (梵文重命名 + 多值 Skandha + 子介面 extends) | v0.25.0-beta | ✅ 完成 (2026-02-27, 1508 tests, +SEC hotfix) |
| Plan26 | Plan26 (Vedana Architecture — 三受系統與我執機制 + CoarisingBundle + IVolition Position B) | v0.26.0-beta | ✅ 完成 (2026-02-28, 1552 tests) |
| Plan27a | Plan27a (IGearArbiter SDK Types + ManoAggregator Core) | v0.27.0-alpha | ✅ 完成 (2026-03-04, 1611 tests) |
| Plan27b | Plan27b (Gear Routing Wiring + Event Emission + StaticRuleArbiter) | v0.27.0-beta | ✅ 完成 (2026-03-05, 1654 tests) |
| Plan28 | Plan28 (IVolition v1 + Safety Hardening) | v0.28.0-alpha | ✅ 完成 (2026-03-05, 1722 tests) |
| Plan32 | Plan32 (Tenet #7 絕對純淨 + Tenet #9 部分修復) | v0.32.0-alpha | ✅ 完成 (2026-03-11, 1803 tests) |
| Plan37 | Plan37 (Phase 6 多代理通訊基礎) | v0.37.0-alpha | ✅ 完成 (2026-03-24, 2097 tests) |
| Plan38 | Plan38 (Multi-Agent Communication Infrastructure) | v0.37.0-alpha (unchanged) | ✅ 完成 (2026-03-28, 2255 tests, 1 rework) |
| Plan39 | Plan39 (AC-7 + Audit Path + Registry Bridge) | v0.39.0-alpha | ✅ 完成 (2026-04-04, 2319 tests) |
| Plan40 | Plan40 (Stabilization + Calibration + Security Hardening) | v0.40.0-alpha | ✅ 完成 (2026-04-06, 2319 tests) |
| Plan41 | Plan41 (Typed Registry + CV-5 Gear-Arbiter-Dynamic + Late-Joiner Snapshot) | v0.41.0-alpha | ✅ 完成 (2026-04-07, 2350 tests) |
| Plan42 | Plan42 (CV-5 Fix + Stabilization) | v0.42.0-alpha | ✅ 完成 (2026-04-09, 2375 tests) |
| Plan43 | Plan43 (COND Backfill + Spec Ratification) | v0.43.0-alpha | ✅ 完成 (2026-04-10, 2380 tests) |
| TBD | Plan44 (Phase 3 Shadow + M4a + Perturbation Diagnostics) | TBD | 📋 PLANNED — deferred per Master ruling |
| TBD | Plan29 (Advanced Security) | TBD | 📋 PLANNED — 待研究團隊指示 |
| TBD | Plan-AC (Agent 協調層架構設計) | — (doc only) | 📋 PLANNED — 待 Phase 5 研究完成 |

---

## 4. 使用方式

- **Coordinator** 開始新迭代前，查閱本文件確認：
  1. 前置 Plan 是否已完成 ✅
  2. 該 Plan 的專屬驗收條件
  3. 是否有可並行的工作
- **architect** 設計 Spec 時，參考專屬驗收條件確保覆蓋
- **qa** 驗證時，逐項檢查通用 DoD + 專屬條件
- **doc-keeper** Plan 打 ✅ 時，確認 DoD 全部滿足
