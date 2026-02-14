# OpenStarry — Plan Dependencies & Definition of Done

Established: 2026-02-09

---

## 1. Plan 相依关系图

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
  ├── Plan05.3 (DevTools)              ⬜ → 延后至 Plan06 之后
  ├── Plan05.4 (E2E Testing)           ⬜ → 待排程
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
  │   Plan05.3 (DevTools) + 05.4 (E2E) ⬜ → Plan11 (Cycle 12)
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
  │   Plan22+ (Advanced Features: Plugin Marketplace, Multi-agent Orchestration)
  │
  └── (Extended Mode: 新 plugin、Web UI、安全审计)
```

### 关键路径（Critical Path）

```
Plan06 → Plan07 → Plan08-09
```

Plan06 (MCP Client + Server) 已完成 ✅，**Plan07 (Runtime Sandbox) 已解除阻挡**，是关键路径上的下一个待实现项目。

### 可并行的工作

| 可并行 | 说明 |
|--------|------|
| Plan06 实现 + Plan05.3/05.4 | DevTools / E2E 测试可与 MCP 并行 |
| Plan06 的预研 + Cycle 2 收尾 | ✅ 已完成 — researcher 可预研 Plan06 |

---

## 2. Definition of Done（完成定义）

### 2.1 通用 DoD（所有 Plan 适用）

每个 Plan 必须满足以下**全部条件**才能标记 ✅：

| # | 条件 | 验证方式 |
|---|------|---------|
| 1 | `pnpm build` 通过（monorepo + plugins） | qa 在 agent_test 验证 |
| 2 | `pnpm test` 全部通过，且测试数 ≥ 前一轮基线 | qa 报告中的 regression check |
| 3 | `pnpm test:purity` 通过 | qa 报告中的 purity check |
| 4 | Architecture_Spec 中所有项目已实现 | architect Code Review PASS |
| 5 | 五蕴合规（新组件正确归类） | architect Code Review 确认 |
| 6 | pushInput 模式合规 | architect Code Review 确认 |
| 7 | 无已知安全漏洞 | architect Code Review 确认 |
| 8 | dev log 已写入 | Coordinator 确认文件存在 |
| 9 | QA Report 和 Code Review 均为 PASS | Phase 4 判定 |
| 10 | doc-keeper 已更新 Plan 打 ✅ + Iteration_Log | Phase 4 收敛 |
| 11 | Snapshot 已建立 | `scripts/snapshot.sh` 完成 |
| 12 | Lessons Learned 已记录 | Phase 4 回顾 |

### 2.2 各 Plan 专属验收条件

#### Plan05.1: Session Isolation ✅ (Cycle 1, 2026-02-10)
- [x] WebSocket 连接支持 session token / auth (session handshake on connect)
- [x] 多个 client 连接互相隔离（session 数据不交叉）
- [x] 单一 agent instance 可服务多个 session (SessionManager + default session)
- [x] Session 生命周期管理（建立、维持、销毁）
- [x] 新增测试覆盖 session isolation 场景 (17 tests)

#### Plan05.2: HTTP SSE ✅ (Cycle 1, 2026-02-10)
- [x] HTTP Server-Sent Events transport plugin 可用 (GET /api/events)
- [x] SSE 连接可接收即时流式响应
- [x] 与现有 HTTP webhook listener 共存不冲突
- [x] 新增测试覆盖 SSE 场景 (11 tests)

#### Plan05.5-①: Health Check ✅ (Cycle 1, 2026-02-10)
- [x] WebSocket protocol ping/pong + stale connection cleanup
- [x] HTTP SSE heartbeat
- [x] 新增测试覆盖 (6 tests)

#### Plan05.5-②: Metrics/Logging ✅ (Cycle 2, 2026-02-11)
- [x] MetricsCollector (increment/gauge/getSnapshot/reset)
- [x] Logger.time() (performance.now())
- [x] METRICS_SNAPSHOT event + /metrics command
- [x] Transport 插件结构化日志迁移
- [x] 新增测试覆盖 (19 tests)

#### Plan05.5-③: Error Handling ✅ (Cycle 2, 2026-02-11)
- [x] ErrorCode const (12 codes)
- [x] TransportError / SessionError / ConfigError
- [x] ES2022 Error cause chain
- [x] 新增测试覆盖 (16 tests)

#### Plan06: MCP Protocol (Model Context Protocol) — Phased

**Cycle 3 (Plan06 Phase 1: MCP Client)** ✅ (Cycle 3, 2026-02-11):
- [x] `@openstarry-plugin/mcp-client` plugin 可用
- [x] MCP client mode：连接外部 MCP server，导入工具 (Tool bridge: MCP tools → ITool)
- [x] MCP Prompt bridge：导入 MCP prompts 为 SlashCommand
- [x] stdio transport：spawn child process, JSON-RPC over stdin/stdout
- [x] Streamable HTTP transport：POST JSON-RPC + optional SSE
- [x] Config-driven server list (agent.json plugins[].config.servers)
- [x] Slash commands: /mcp-status, /mcp-tools, /mcp-prompts
- [x] 新增测试覆盖 MCP client 场景 (35 tests)

**Cycle 4 (Plan06 Phase 2: MCP Server)** ✅ (Cycle 4, 2026-02-11):
- [x] `@openstarry-plugin/mcp-server` plugin
- [x] MCP server mode：暴露 ITool/IGuide 为 MCP tools/prompts
- [x] Server transports: stdio + HTTP (http.createServer)
- [x] JSON-RPC 2.0 handler (initialize, tools/list, tools/call, prompts/list, prompts/get)
- [x] Config-driven: server name, version, exposed tools/guides whitelist
- [x] SDK: IPluginContext.tools + IPluginContext.guides (optional, non-breaking)
- [x] SDK: MCP_CLIENT_CONNECTED + MCP_CLIENT_DISCONNECTED events
- [x] 新增测试覆盖 MCP server 场景 (52 tests)

**Cycle 15 (Plan06 Phase 3: Resources + Auth)** ✅ (Cycle 15, 2026-02-12):
- [x] MCP Resources bridge (resource access protocol, capability negotiation)
- [x] OAuth 2.1 Authorization (client credentials flow, token management)
- [x] Resource metadata handling (resource.list, resource.read)
- [x] Authorization context integration (scope validation, token refresh)
- [x] 新增测试覆盖 MCP resources + auth 场景 (29 tests)

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
- [x] 新增测试覆盖 sampling/logging/roots/bidirectional scenarios (87 tests)
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
- [x] Plugin 执行环境隔离（Node.js vm.createContext + worker_threads）
- [x] 资源限制（CPU watchdog deadline timer、内存沙箱隔离、文件访问）
- [x] Plugin 签名验证机制（SHA-512 hash + Ed25519 PKI support）
- [x] 新增测试覆盖 sandbox escape 场景 (68 tests)
- Sandbox 版本：v0.4 (Cycle 5)

#### Plan07.1: Sandbox Hardening ✅ (Cycle 6, 2026-02-11)
- [x] CPU watchdog (deadline timer per RPC call)
- [x] Bidirectional EventBus (worker subscriptions + forwarding)
- [x] Worker restart policies (ON_CRASH, ON_TIMEOUT with exponential backoff)
- [x] Async proxy for worker context
- [x] 新增测试覆盖 (31 tests)
- Sandbox 版本：v0.4.1-beta (Cycle 6)

#### Plan07.2: Sandbox Advanced Hardening ✅ (Cycle 7, 2026-02-11)
- [x] Static import restrictions (@babel/parser AST analysis, blocklist enforcement)
- [x] Worker pool pre-spawning (piscina connection pool, lazy initialization)
- [x] PKI asymmetric signing (Ed25519 + RSA fallback, backward-compatible format detection)
- [x] Plugin-signer CLI tool (keygen, sign, verify commands)
- [x] 新增测试覆盖 (55 tests)
- Sandbox 版本：v0.4.2-beta (Cycle 7)
- **Status**: PASS (1 rework cycle: fixed import integration, plugin-signer package, package-name graceful handling)

#### Plan08: TUI Dashboard MVP ✅ (Cycle 9, 2026-02-11)
- [x] TUI Dashboard plugin (Ink v5 + React 18)
- [x] Event-to-state mapping (14 event types → TUI actions)
- [x] Pure state management with React useReducer
- [x] Streaming delta accumulation (APPEND_STREAM/FINALIZE_STREAM)
- [x] Terminal components: Header, ChatArea, EventLog, Footer
- [x] Formatting utilities: truncate, timestamp, status symbols
- [x] 新增测试覆盖 (53 tests)
- **Version**: v0.5.0-beta
- **Stats**: 524 tests (82 new), 16 packages, 0 regressions

#### Plan09: Interactive TUI (Chat Input + Command Execution) ✅ (Cycle 10, 2026-02-12)
- [x] Text input component (custom useInput, no external deps) in TUI Dashboard
- [x] IListener (受蕴) implementation for keyboard input
- [x] Push text input to ctx.pushInput() for agent processing
- [x] Command history navigation (arrow up/down, max 50, dedup consecutive)
- [x] Slash command parsing and routing (/quit, /help, /mcp-status, etc.)
- [x] Visual feedback: typing indicator (isPending state), command execution status
- [x] Input mode management (chat mode vs command mode)
- [x] No SDK changes required (IListener + IUI already defined) — VERIFIED
- [x] Single plugin enhancement to @openstarry-plugin/tui-dashboard
- [x] 新增测试覆盖 interactive input scenarios (35 tests)
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
- [x] 新增测试覆盖 (44 tests)
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
- [x] 新增测试覆盖 (33 tests)
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
- [x] 新增测试覆盖 (15 tests: session persistence, multi-client, tab-completion, backpressure)
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
- [x] 新增测试覆盖 (39 tests: 19 scanner + 10 sync + 5 E2E + 5 resolver)
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

---

## 3. 迭代排程建议

| 迭代 | 涵盖 Plans | 目标版本 | 状态 |
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
| Cycle 25+ | Plan22+ (Advanced Features: Plugin Marketplace, Multi-agent Orchestration) | v0.20.0+ | ⬜ 待排程 |

---

## 4. 使用方式

- **Coordinator** 开始新迭代前，查阅本文件确认：
  1. 前置 Plan 是否已完成 ✅
  2. 该 Plan 的专属验收条件
  3. 是否有可并行的工作
- **architect** 设计 Spec 时，参考专属验收条件确保覆盖
- **qa** 验证时，逐项检查通用 DoD + 专属条件
- **doc-keeper** Plan 打 ✅ 时，确认 DoD 全部满足
