# OpenStarry Agent Corps — Lessons Learned

每輪迭代 Phase 4 結束後，由 doc-keeper 追加本輪回顧。

---

## 20260210_cycle1 + 20260211_cycle2

（Cycle 1/2 在 Agent Corps 建立初期完成，回顧併入 Cycle 3。）

---

## 20260211_cycle3: MCP Client Plugin (v0.3.0-beta)

### What Went Well

1. **Researcher pre-research was invaluable** — MCP pre-research report (including openoctopus reference analysis and Five Aggregates mapping) enabled rapid and confident Phase 0 scope decisions without lengthy debates.
2. **Custom JSON-RPC approach was correct** — ~400 lines of custom transport code avoided Zod version conflict with @modelcontextprotocol/sdk dependency. Clean, debuggable, and no external SDK lockstep.
3. **Parallel Phase 2a/2b execution** — SDK extensions and plugin implementation could proceed in parallel after interface freeze. Phase 2a (SDK) was small and completed fast, unblocking Phase 2b quickly.
4. **QA + Architect parallel in Phase 3** — Both verification streams completed concurrently, no serial bottleneck.
5. **Test coverage exceeded target** — 35 new tests vs 30-40 target; comprehensive coverage across schema, client, bridges, and factory.

### What Could Be Improved

1. **TypeScript strict mode + external type bridging** — Three distinct TS errors during build (double-cast for config, JSON Schema to Zod type mismatch, union type breaking recursion). Lesson: When bridging loosely-typed external protocols (JSON-RPC, JSON Schema) into strict TS, plan for an explicit boundary layer from the start (internal types + public adapter function).
2. **Test file organization** — All 35 tests in single `index.test.ts` file. Architect flagged this as minor issue. For next plugin, split test files per module from the start (matches Architecture Spec file inventory).
3. **Missing vitest.config.ts** — Forgot to create it despite being in Architecture Spec file inventory. Add to dev-plugin checklist: always create vitest.config.ts for new packages.
4. **Subagent write permissions** — Subagents hit permission blocks on share/ and agent_dev/ directories. Workaround: coordinator wrote files directly. Need to verify agent permission configs before cycle start.

### Patterns to Reuse

1. **JSON Schema → Zod converter pattern**: Split into internal recursive function (`convert(schema: InternalType)`) + public adapter (`publicFunc(schema: Record<string, unknown>)`). Reusable for any JSON Schema bridging.
2. **MockTransport pattern**: The `MockMcpTransport` class with method→response Map is a clean, reusable test pattern for protocol testing.
3. **Plugin factory resilience**: Continue-on-failure loop for multi-server initialization (`for (const server of config.servers) { try { ... } catch { logger.error(...) } }`) — good pattern for all multi-resource plugins.
4. **Namespaced tool IDs**: `serverName/toolName` pattern prevents cross-server collisions. Apply same pattern if multiple plugin sources provide tools.

### Action Items for Future Cycles

- [ ] Add vitest.config.ts to dev-plugin new package checklist
- [ ] Split test files per module for new plugins (not monolithic index.test.ts)
- [ ] Verify subagent write permissions before cycle start
- [ ] Consider protocol version validation in MCP client (optional enhancement from Architect review)

---

## 20260211_cycle4: MCP Server Plugin (v0.3.1-beta)

### What Went Well

1. **Architecture Spec was comprehensive** — The architect-produced spec covered all frozen interfaces, design decisions, risk assessment, and file inventory. Implementation was nearly mechanical.
2. **Split test files per module** — Applied Cycle 3 lesson: 7 separate test files (tool-adapter, prompt-adapter, handler, stdio, http, plugin) instead of monolithic. Achieved 52 tests with clear coverage boundaries.
3. **vitest.config.ts created from start** — Applied Cycle 3 action item. Package skeleton included vitest.config.ts in initial creation.
4. **zodToJsonSchema reuse** — Reusing `@openstarry/shared`'s `zodToJsonSchema` for the reverse bridge (ITool → MCP tool definition) worked perfectly. No new converter needed.
5. **Parallel Phase 3** — QA and Architect code review ran concurrently. Both completed within minutes.
6. **Test count exceeded target** — 52 tests vs spec target of 30-40. Comprehensive coverage across all modules.

### What Could Be Improved

1. **Subagent write permissions STILL blocked** — Both dev-plugin agent (Phase 2b) and QA agent were blocked from writing to share/ and openstarry_plugin/ directories. Coordinator had to implement the entire plugin and write all reports manually. This is the 3rd consecutive cycle with this issue.
2. **HTTP transport test port conflicts** — Initial HTTP tests used a single port for all tests, causing "socket hang up" errors when tests ran concurrently. Fixed by using unique ports per test via `nextPort()` counter.
3. **GET request test failure** — Initial GET test used `sendRequest` helper which sends a body, causing socket issues. Fixed by creating a separate `sendGetRequest` helper using `http.get`.
4. **Missing zod devDependency** — Initial package.json didn't include `zod` as devDependency (only used in tests). Tests importing `z` from `zod` failed with "Cannot find package 'zod'". Quick fix.

### Patterns to Reuse

1. **Unique port per HTTP test**: `let portCounter = 39800; function nextPort() { return portCounter++; }` — prevents port conflicts in parallel test execution.
2. **filterExposed generic**: `function filterExposed<T extends { id: string }>(items: T[], filter: string[] | "*" | undefined): T[]` — reusable whitelist filter for any ID-based filtering.
3. **Handler factory pattern**: `createMcpJsonRpcHandler(config, ctx)` returns a pure async function `(req) => Promise<response>`. Clean separation of handler logic from transport.
4. **Lazy proxy accessors on IPluginContext**: Optional `tools?` and `guides?` fields that delegate to registries without exposing mutation. Reusable pattern for any future cross-plugin registry access needs.

### Action Items for Future Cycles

- [ ] **CRITICAL**: Fix subagent write permissions before Cycle 5 (3 cycles of workarounds)
- [ ] Extract `@openstarry-plugin/mcp-common` shared types package (Architect Note 1)
- [ ] Add README.md to mcp-server package with usage/load order guidance (Architect Note 2)
- [x] Split test files per module (DONE — applied from Cycle 3)
- [x] Create vitest.config.ts for new packages (DONE — applied from Cycle 3)

---

## 20260211_cycle5: Plan07 Sandbox MVP (v0.4)

(Abbreviated — 3-day rapid cycle with focused scope on sandbox basics. Key lessons integrated into Cycle 6-7 improvements.)

### Highlights

1. **vm.createContext + worker_threads sandwich** — Effective two-layer isolation (JS context isolation + OS process boundary)
2. **Signature verification integration worked cleanly** — SHA-512 hash verification minimal and backward-compatible
3. **Sandbox escape testing revealed attack surfaces early** — Caught 3 potential escape vectors during Phase 3 code review

### Items Deferred to 07.1+

- CPU watchdog (too complex for MVP, added in 07.1)
- Worker pool pre-spawning (optimization, not critical for MVP)
- Asymmetric key signing (nice-to-have, added in 07.2)

---

## 20260211_cycle6: Plan07.1 Sandbox Hardening (v0.4.1-beta)

### What Went Well

1. **CPU watchdog via deadline timer** — Simple, non-invasive implementation. `setInterval` heartbeat check + AbortController + timeout rejection. No native C++ modules needed.
2. **Bidirectional EventBus via onAny() subscription** — Elegant pattern: worker subscribes to EventBus messages, handler broadcasts events. Enables worker→main notification without special RPC encoding.
3. **Worker restart with exponential backoff** — Graceful failure recovery. `1s → 2s → 4s → ...` prevents thundering herd on cascading failures.
4. **Async proxy for parentPort message handler** — Solves the worker context async/await problem. Wrapper function with proper Promise error handling.
5. **Parallel QA + Architect verification** — Phase 3 completed in parallel; no serial bottleneck. Both agents tested independently and concurrently reported PASS.

### What Could Be Improved

1. **Tests written late** — Implementation completed, then tests added after. Reverse chronology. Lesson: Write test spec in Phase 1, then implement+test in Phase 2.
2. **Message type schema scattered** — EventBus message types defined in messages.ts but documentation in Architecture Spec. Consider centralizing all message type docs in a dedicated messages.md reference.
3. **Worker restart manual testing required** — Tests cover happy path (ON_CRASH works), but manual testing with actual worker crashes needed for confidence. Recommendation: Create a "chaos" test suite that intentionally crashes workers.

### Patterns to Reuse

1. **Deadline timer pattern**: `const deadline = Date.now() + timeoutMs; const remaining = deadline - Date.now(); if (remaining <= 0) reject(new Error('timeout'))`. Reusable for any async operation timeout.
2. **Exponential backoff**: `const delay = Math.min(baseDelay * Math.pow(2, retries), maxDelay)`. Standard pattern, good for worker restarts and connection retries.
3. **EventBus.onAny() for debugging**: The onAny() subscription pattern is useful not just for EventBus forwarding, but also for central logging and metrics collection.

### Action Items for Future Cycles

- [ ] Establish test-first development process (write tests in Phase 1, not Phase 2)
- [ ] Create chaos test suite for worker restart scenarios
- [ ] Document all message types in dedicated reference (messages.md)

---

## 20260211_cycle7: Plan07.2 Advanced Hardening (v0.4.2-beta)

### What Went Well

1. **Static import analysis via @babel/parser** — AST-based approach is clean and maintainable. @babel/parser alone is sufficient (no need for @babel/traverse). Produces accurate import/require detection.
2. **Ed25519 PKI signing now built-in** — Node.js crypto.sign(null, ...) works perfectly for Ed25519. Backward-compatible format detection (128-char hex = legacy, object = PKI) is elegant and non-breaking.
3. **Plugin-signer CLI tool** — Three commands (keygen, sign, verify) cover full lifecycle. JSON output format is clean and parseable. Tool is self-contained, not library-dependent.
4. **Rework cycle efficiency** — 3 FAIL items all fixed in single 4-hour rework cycle. Clear root cause analysis + focused fixes + parallel re-verification. No ping-pong.
5. **Graceful package-name handling** — Plugins with npm package names can't be file-verified. Solution: warn in logs, continue execution. No blocking. Good UX.

### What Could Be Improved

1. **Integration point was a comment initially** — Phase 2 implementation had `// TODO: call analyzeImports()` instead of actual code. Caught by QA in Phase 3. Lesson: No placeholder comments for integration points; write actual code.
2. **Plugin-signer package not created in Phase 2** — Oversight in scope tracking. Package was in Architecture Spec but not created. Lesson: Maintain checklist of all files/packages mentioned in spec, verify during Phase 2 completion.
3. **Signature verification threw for package-name plugins** — Code path assumed all plugins have file paths. Oversight: Code assumed invariant without checking. Lesson: Validate assumptions, add guards early.
4. **require() proxy secondary defense wasn't tested** — We have AST analysis + require() proxy, but only AST is tested. require() proxy is untested fallback. Future: Add dedicated test for require() interception.

### Root Cause of Phase 3 FAILs

1. **FAIL-1 (Integration)**: Missing integration code, not just documentation → Design/Implementation mismatch
2. **FAIL-2 (Scope)**: File mentioned in spec but not created → Process gap in Phase 2 checklist
3. **FAIL-3 (Assumption)**: Code assumed file path exists without guard → Missing input validation

**Meta-lesson**: All 3 FAILs were preventable via process discipline:
- FAIL-1 → Code review before commit (check for TODO comments)
- FAIL-2 → Phase 2 checklist review (match spec file inventory)
- FAIL-3 → Input validation review (check assumptions)

### Patterns to Reuse

1. **AST-based analysis pattern**:
   ```typescript
   const ast = parse(code);
   const imports = new Set<string>();
   traverse(ast, { enter(node) { ... } });
   return imports;
   ```
   Reusable for any code analysis: linting, security scanning, dependency extraction.

2. **Backward-compatible union type format detection**:
   ```typescript
   if (typeof integrity === 'string' && integrity.length === 128) {
     // Legacy SHA-512 hash
   } else if (typeof integrity === 'object') {
     // PKI format
   }
   ```
   Pattern: Detect format by type/length/shape, then dispatch to handler. Reusable for any schema versioning.

3. **Graceful degradation with warning**:
   ```typescript
   if (!filePath) {
     logger.warn('Package-name plugin, skipping file verification');
     return; // Continue, don't block
   }
   ```
   Pattern: Log discrepancy, continue operation. Good for optional features or constrained environments.

4. **CLI tool structure** (plugin-signer):
   ```
   src/
     ├── commands/
     │   ├── keygen.ts
     │   ├── sign.ts
     │   └── verify.ts
     ├── utils/
     │   └── crypto.ts
     └── index.ts (command router)
   ```
   Reusable structure for any multi-command CLI tool.

### Action Items for Future Cycles

- [ ] **PROCESS**: Add Phase 2 checklist item: "Verify all files/packages in Architecture Spec are created"
- [ ] **PROCESS**: Code review checklist: "No placeholder comments in integration points"
- [ ] **TESTING**: Add dedicated test for require() proxy interception (secondary defense)
- [ ] **TOOLING**: Consider adding spec-to-checklist generator (automated verification of spec file inventory)
- [ ] **DOC**: Create messages.md centralized reference for all message types (deferred from Cycle 6)

### Lessons for Rework Cycles

1. **Clear root cause analysis** — Each FAIL traced to specific root cause (missing integration, missing package, missing guard). Enabled focused fixes.
2. **Minimal rework scope** — Total rework was ~30 lines + 1 new package. No cascade of changes. Reduced risk of introducing new bugs.
3. **Parallel re-verification** — QA + Architect both verified rework in parallel, completed within minutes of fixes landing.
4. **Test expansion during rework** — Added specific test for import integration, package-name graceful handling. Prevents regression.

---

## 20260211_cycle8: Plan07.3 Sandbox Final Hardening (v0.4.3-beta)

### What Went Well

1. **Module._load patching was the right approach** — Replacing `Module._load` directly proved more reliable than globalThis.require Proxy. Node.js internal require routing goes through _load, capturing all paths (direct require, dynamic import→createRequire, etc.).
2. **Audit logging JSONL format** — Line-delimited JSON is perfect for structured logging + streaming. Each line independently parseable. Rotation and cleanup worked without issues.
3. **Buffered audit writes** — Batch writes (flushIntervalMs, maxBufferSize) reduced I/O overhead while maintaining responsiveness. Tests confirmed consistent sanitization and ordering.
4. **Lifecycle + RPC + tool logging coverage** — Three distinct event categories (lifecycle: init/stop, RPC: call/response/error, tool: invoke/success/failure) provide full audit trail.
5. **Sanitization patterns** — Regex-based and value-list patterns both worked. Tests verified secret redaction across nested objects (apiKey, password, secret, token, etc.).
6. **PASS with no rework needed** — Clean implementation. QA and Architect verified in parallel; no FAIL items found.

### What Could Be Improved

1. **Node.js WriteStream async timing** — `fs.createWriteStream()` opens files asynchronously. Initial test failures were due to writes happening before 'open' event. Solution: Always await 'open' event before first write or use buffering queue (which we did). Lesson: Understand Node.js stream lifecycle quirks.
2. **Secret pattern matching was too broad initially** — Regex `/key/i` matches "apiKey", "sessionKey", "toolKey", etc. more broadly than intended. Design decisions: Either use exact key name list (`['apiKey', 'password', 'secret', 'token']`) or very specific patterns (`/^(api)?[_-]?key$/i`). We chose list.
3. **Module._load typing** — TypeScript doesn't provide types for internal Module._load API. Workaround: `(Module as any)._load(...)`. Document this pattern for future module-level patching.
4. **Audit log location** — No standard consensus on where to store audit logs. We used `process.cwd()/agent-audit.log`. Future: Make configurable via SandboxAuditConfig.logPath.
5. **Test flakiness from file I/O timing** — Cleanup between tests required explicit file deletion + await on WriteStream 'close' event. Pattern: Always await stream.close() in test teardown.

### Patterns to Reuse

1. **Module._load patching pattern**:
   ```typescript
   const originalLoad = Module._load;
   Module._load = function(request: string, parent: any) {
     if (blockedModules.has(request)) {
       throw new Error(`Module '${request}' is blocked`);
     }
     return originalLoad.apply(this, arguments);
   };
   ```
   Reusable for any runtime module interception (security, feature flags, mock injection).

2. **JSONL audit logger with buffering**:
   ```typescript
   buffer.push(entry);
   if (buffer.length >= maxBufferSize || timeSinceLastFlush > flushIntervalMs) {
     flush(); // Write all entries at once
   }
   ```
   Reusable for any high-volume structured logging: compliance audits, transaction logs, error tracking.

3. **Value-list sanitization** (vs regex):
   ```typescript
   const sensitiveKeys = ['apiKey', 'password', 'secret', 'token', 'privateKey'];
   if (sensitiveKeys.includes(key)) {
     return '[REDACTED]';
   }
   ```
   More maintainable than regex patterns. Reusable for any secret redaction.

4. **WriteStream lifecycle handling**:
   ```typescript
   const stream = createWriteStream(path);
   await new Promise<void>((resolve) => {
     stream.once('open', () => resolve());
   });
   // Now safe to write
   ```
   Pattern: Always await 'open' for buffering guarantees. Applies to all file-backed streams.

### Action Items for Future Cycles

- [ ] Make audit log location configurable (SandboxAuditConfig.logPath or SandboxAuditConfig.logDir)
- [ ] Document Module._load typing workaround in Architecture reference
- [ ] Test write timing with real-world file I/O (add "file I/O stress" test to chaos suite from Cycle 6)
- [ ] Consider configurable sanitization key list (allow-list approach vs hard-coded list)

### Lessons for Runtime Module Interception

1. **Module._load is the single point** — All require() paths (sync, async, dynamic) funnel through _load. Patching here is 100% effective.
2. **Strict/warn/off modes** — Three enforcement levels (strict: throw, warn: log+continue, off: passthrough) provide operational flexibility.
3. **Module cache bypass** — require() checks require.cache first, so patching _load AFTER cache hits doesn't work. Patch early in sandbox initialization, before any requires.
4. **No Proxy needed** — globalThis.require Proxy approach from earlier was overengineered. Direct _load patching is simpler, more performant, and more reliable.

---

## 20260211_cycle9: TUI Dashboard MVP (v0.5.0-beta)

### What Went Well

1. **No SDK changes needed** — IUI.onEvent() was sufficient for the TUI. This simplified the cycle enormously (no interface freeze, no SDK addendum, direct to implementation).
2. **Ink v5 + React pattern** — Proven framework, React knowledge transfer, clean component model.
3. **Pure reducer pattern** — tuiReducer is fully testable without React or Ink dependencies. 21 reducer tests run in <15ms.
4. **Independent audit caught real bugs** — Post-implementation deep audit found 2 major issues (Ink unmount leak, missing dispose hook) and 4 medium issues that would have caused problems in production.
5. **Exceeded test target** — 58 tests vs 18-25 target (294% of minimum).

### What Could Be Improved

1. **QA agent fabricated test descriptions** (CRITICAL PROCESS ISSUE)
   - The QA report described actions and events that do not exist in the codebase: `UPDATE_MESSAGE`, `SET_SKILL_STATUS`, `CLEAR_MESSAGES`, `UPDATE_ACTIVE_SKILL`, `MESSAGE_AGENT`, `SKILL_STARTED`, etc.
   - Root cause: QA agent likely generated descriptions from the architecture spec or general knowledge rather than reading the actual test files.
   - Fix applied: Updated `qa.md` agent definition with explicit requirement to Read each test file before describing it. Added "CRITICAL: Test Description Accuracy" section.
   - Lesson: **Agent-generated reports must be verified against source code.** Numbers can be correct while descriptions are fabricated.

2. **Ink lifecycle management was missed in initial implementation**
   - `render()` returns an Instance that must be stored and `.unmount()`ed on stop. This was not in the architecture spec's code examples.
   - Lesson: For any framework that takes over terminal/UI state, always plan for proper cleanup in the spec.

3. **Event race condition between Core and React**
   - Events arriving before React mounts are silently dropped. The DispatchBridge pattern needs a buffer.
   - Lesson: When bridging synchronous event emitters to async UI frameworks, always buffer early events.

4. **Reducer purity violation** — `Date.now()` inside reducer is a side effect. Caught by audit, not by tests.
   - Lesson: Reducers should never call Date.now(), Math.random(), or crypto.randomUUID(). Pass timestamps/IDs via action payloads.

### Patterns to Reuse

1. **DispatchBridge pattern** — Bridge React internal dispatch to external plugin code via a null-initialized reference + useEffect callback + pending event buffer.
2. **APPEND_STREAM / FINALIZE_STREAM** — Dedicated actions for streaming content accumulation. Cleaner than overloading ADD_MESSAGE with magic IDs.
3. **Dynamic import for heavy dependencies** — `await import("ink")` inside `start()` avoids loading React/Ink when the plugin is just registered but not active.
4. **Post-implementation audit** — Running an independent code audit after Phase 3 PASS caught issues that both QA and architect missed. Consider making this a standard reinforcement step.

### Action Items for Future Cycles

- [ ] Add event log scrolling tests (keyboard binding `[`/`]` now implemented but not tested)
- [ ] Consider adding `ink-testing-library` for component rendering tests in a future cycle
- [ ] Document stdio/TUI mutual exclusion in runner configuration guide
- [ ] Add upper bound clamping to scroll offsets (prevent arbitrarily large values)
- [ ] Review all past QA reports for similar accuracy issues

---

## 20260212_cycle12: Plan11 DevTools & E2E Testing Framework (v0.7.0-beta)

### What Went Well

1. **Dev-plugin subagent permission workaround successful** — Although dev-plugin agent was blocked from writing to agent_dev/openstarry_plugin/ directory, coordinator was able to implement the entire DevTools plugin directly. Demonstrates that process can continue despite permission issues with a flexible workflow.
2. **Headless UI design enabled excellent testability** — DevTools plugin uses React components without Ink dependency in tests. Pure reducer pattern (tuiReducer from prior cycle) proved reusable, making DevTools state management trivial to test (38 tests with high coverage).
3. **E2E fixture pattern proved robust** — MockProvider + AgentTestFixture pattern (introduced in Cycle 11 runner tests) scaled excellently for multi-session, multi-plugin, workflow testing. Fixtures are composable and reusable.
4. **README creation as rework item** — Cycle 11 experience showed that README requests become conditional architectural passes. Cycle 12 proactively included README items in initial design, leading to rework but also ensuring documentation quality from Phase 2.
5. **Test count efficiency** — 75 new tests (38 DevTools + 40 E2E) across new plugin + test framework achieved comprehensive coverage without bloat. Average 2-3 tests per major feature.

### What Could Be Improved

1. **README should be Phase 2 deliverable, not Phase 3 advisory** — Three consecutive cycles (Cycle 9-11-12) had README as conditional/advisory. Pattern: Architect should specify README in Phase 1 as part of Phase 2 deliverables, not as Phase 3 advisory. Process improvement for Phase 1 spec format.
2. **Subagent write permissions remain unresolved** — Fourth consecutive cycle where dev-plugin agent hit permission blocks. Coordinator has developed a workaround (implement directly), but this is not sustainable. Root cause: Subagent permission configs need audit and fix before Cycle 13.
3. **E2E fixture documentation oversized initially** — Initial E2E helpers (MockProvider, AgentTestFixture, SessionHelper) were 800+ lines each. Rework added 200+ lines of inline documentation. Lesson: Complex fixtures need inline documentation from first implementation, not retroactively.

### Patterns to Reuse

1. **Pure reducer for state management** — DevTools reuses tuiReducer pattern from Cycle 9. Zero React/framework dependencies in tests. This pattern should become standard for all UI state (TUI, DevTools, future Web UI).
2. **E2E fixture composition** — Stack pattern works well:
   ```typescript
   const session = new SessionHelper(agent);
   const mock = new MockProvider(session);
   const fixture = new AgentTestFixture(mock);
   ```
   Fixtures compose cleanly without tangling. Reusable for future frameworks (multi-agent testing, daemon testing).
3. **README integration test** — E2E framework includes a test that reads the README, parses examples, and verifies they run. This ensures documentation doesn't decay. Pattern applies to any complex framework.

### Lessons Learned

1. **Headless design is better than framework-dependent** — DevTools avoided Ink dependency in core logic, using React only for rendering. This made testing easier and implementation faster. Recommendation: Design UI components headless where possible (state + render separation).
2. **E2E tests catch real bugs** — The E2E framework caught 2 subtle bugs in CLI routing (argument parsing for `--config path/to/file`, not `--config=path/to/file`) and plugin resolution (circular dependency in plugin loading order). These bugs would not be caught by unit tests.
3. **Fixture library maturity accelerates testing** — By Cycle 12, the fixture library (MockProvider, AgentTestFixture, SessionHelper) was mature and well-documented. New E2E tests went 4x faster to write than Cycle 11 runner tests. Lesson: Invest in good test infrastructure early, reuse aggressively.
4. **Documentation as code works** — E2E README includes runnable examples. Test suite includes test that verifies examples work. Documentation never decays. Lesson: For complex APIs, consider making examples executable tests.

### Action Items for Future Cycles

- [ ] **PROCESS**: Change Phase 1 Spec format to include README as deliverable (not advisory), specify content outline
- [ ] **INFRASTRUCTURE**: Audit and fix subagent write permissions for dev-plugin (5 cycles of workarounds)
- [ ] **TESTING**: Make E2E fixture documentation inline from first implementation (not retroactive rework)
- [ ] **STANDARDS**: Recommend headless UI design pattern for all future UI components
- [ ] **TOOLING**: Consider automated README validation (examples are runnable tests)

---

## 20260212_cycle13: Plan12 Daemon Mode MVP (v0.8.0-beta)

### What Went Well

1. **Daemon process lifecycle handling was straightforward** — `child_process.spawn` with `detached: true` + `unref()` pattern is elegant and reliable. Process cleanly separates from parent.
2. **IPC over Unix domain sockets was lightweight** — Custom ~200-line JSON-RPC implementation avoided heavyweight dependencies. Socket-based communication is fast and doesn't require port conflicts.
3. **Health check provider pattern validated** — Using IProvider for agent health endpoint (uptime, version, status) is clean. Future seamless attach can build on this foundation.
4. **Signal cascade worked as designed** — SIGTERM → graceful shutdown → socket cleanup → PID file cleanup. Multi-stage shutdown is more resilient than single-signal approach.
5. **Graceful failure on shutdown timeout** — When agent doesn't shut down within 5s, daemon process force-kills and reports status. Good operational behavior.

### What Could Be Improved

1. **`createWriteStream` timing issue with file descriptors** — Initial implementation passed `WriteStream` to `spawn(stdio)`, causing async ENOENT errors when test directories were cleaned up. Solution: Use `openSync()` to get file descriptor immediately, pass FD number instead of stream object. Lesson: Never pass async objects to `spawn()` stdio config; use synchronous file operations.
2. **Dead code in initial implementation** — Architect review caught unused helper functions (`shutdownGracefully` defined but not called, replaced by inline `shutdownWithTimeout`). Lesson: Code review checklist item "no dead code" must be enforced before Phase 3.
3. **Mock daemon entry script integration test** — Testing daemon startup required a mock entry script that bypassed LLM provider initialization. Pattern: Create a minimal daemon-entry-mock.js for integration tests, verify with real entry point separately. This enabled fast testing of IPC protocol without full agent startup.
4. **PID file race conditions under load** — If multiple `daemon start` commands fired simultaneously, PID files could be written out-of-order. Solution: Atomic write (write temp file, rename). Lesson: Any persistent state (PID files, sockets) needs atomic operations.

### Patterns to Reuse

1. **Daemon entry script pattern**:
   ```typescript
   // daemon-entry.ts: Runs in background process
   import { AgentCore } from '@openstarry/core';
   import { createDaemonPlugin } from '@openstarry-plugin/daemon';

   const agent = new AgentCore();
   await agent.bootstrap({
     configPath: process.argv[2],
     transportMode: 'daemon',  // Signal to skip TUI, enable IPC
   });
   ```
   Reusable for any background service (workers, orchestrators, data processors).

2. **File descriptor instead of WriteStream**:
   ```typescript
   const logFd = fs.openSync(logPath, 'a');
   const proc = spawn(entry, args, {
     stdio: ['inherit', logFd, logFd],  // Use FD, not stream
   });
   ```
   Safer than passing streams to `spawn()`. Use for any long-lived child processes.

3. **Health check RPC for background services**:
   ```typescript
   // IPC method: agent.health() → {ok: bool, uptime: number, version: string}
   // Enables external monitoring, orchestration, seamless attach
   ```
   Reusable for any long-lived background process. Foundation for future graceful reload, canary deployments.

4. **SetDaemonEntryOverride testing hook**:
   ```typescript
   // In tests, override daemon entry point to skip provider init
   setDaemonEntryOverride(mockEntryPath);
   const proc = spawn(mockEntryPath, ...);
   ```
   Clean dependency injection for testing without changing frozen interfaces. Pattern applies to any override-able service locator needs.

5. **Atomic PID file writes**:
   ```typescript
   const tempPath = pidPath + '.tmp';
   fs.writeFileSync(tempPath, pid.toString());
   fs.renameSync(tempPath, pidPath);  // Atomic swap
   ```
   Standard pattern for persistent state. Prevents partial writes and race conditions.

### Lessons for Socket/IPC Programming

1. **Unix domain sockets have no port conflicts** — Unlike TCP, you can safely reuse socket paths across test instances without `EADDRINUSE` errors. Use `unlink()` in test cleanup.
2. **JSON-RPC 2.0 `id` field must match** — Request with id=1 must get response with id=1. Early implementation used `undefined` IDs; caught by tests.
3. **`connect()` may return immediately or call callback later** — Both behaviors are valid. Always handle both paths (add event listener, don't assume synchrony).
4. **Socket `destroy()` vs `end()`** — `end()` gracefully closes; `destroy()` force-closes. For cleanup, use `destroy()`. For graceful shutdown, use `end()`.

### Root Cause of Phase 3 Conditional Pass

1. **CONDITIONAL (2 fixes)**:
   - **Fix 1**: Shutdown timeout enforcement missing — `shutdownWithTimeout()` function defined but not called in daemon lifecycle. Architect flagged as dead code. Solution: Call in IPC server cleanup on SIGTERM.
   - **Fix 2**: Dead code removal — Unused helper functions removed. Lesson: Phase 2 cleanup checklist: "grep for unused functions before Phase 3".

**Rework was minimal**: 2 lines of code changes (one function call, one code removal). Indicates good initial design; issues were oversight in wiring, not architecture.

### Action Items for Future Cycles

- [ ] Add file descriptor handling (openSync + FD numbers) to test infrastructure docs
- [ ] Create test infrastructure for daemon-entry mocking (reusable for Plan13+)
- [ ] Document Unix domain socket best practices (socket path cleanup, atomic writes)
- [ ] Consider adding "no dead code" linter rule to Phase 2 checklist
- [x] Plan13: Build seamless attach on top of health check RPC (foundation is ready) — DONE (Cycle 14)

---

## 20260212_cycle14: Plan13 Seamless Attach (v0.9.0-beta)

### What Went Well

1. **Event forwarder pattern was elegant** — Session-filtered event delivery via sessionId in core.bus bridge enables clean separation of concerns. Forwarding logic is ~100 lines, minimal and testable.
2. **IPC protocol extension was natural** — Adding `agent.attach`, `agent.input`, `agent.detach` RPC methods felt organic to existing IPC layer. No SDK changes needed; reused existing daemon infrastructure.
3. **Terminal I/O proxy was straightforward** — stdin → IPC client → agent.input RPC → event streaming → stdout pattern works reliably. No special terminal handling needed (readline + native streams sufficient).
4. **Session lifecycle management simplified testing** — sessionId creation on attach, cleanup on detach, timeout handling all tested with simple session state checks. Pure async/await code, minimal state machinery.
5. **Auto-start feature enabled seamless UX** — `openstarry attach [agent-id]` automatically spawns daemon if not running, then connects. No manual `daemon start` step needed. Users get a transparent experience.

### What Could Be Improved

1. **Security validation should be earlier in design** — Architect flagged "validate agentId before attach" as conditional PASS. This was obvious in retrospect (check daemon is alive, agent responsive) but was deferred to Phase 3 rework. Lesson: Enumerate all security checks in Phase 1 Architecture Spec, not found during review.
2. **Event timestamp field was added in rework** — Initial event forwarding lacked timestamps, making traceability harder. Architect correctly flagged this. Lesson: Any event system must include timestamp from day one. Add to Five Aggregates event interface standard.
3. **Error codes documentation missing** — Phase 2 implementation had error throws but no centralized error code documentation. Architect flagged this in conditional PASS. Solution: Created ErrorCode const with documented error codes (AGENT_OFFLINE, SESSION_TIMEOUT, INVALID_INPUT_TYPE, MESSAGE_SIZE_LIMIT_EXCEEDED). Lesson: Error handling as part of API surface needs explicit documentation.
4. **Single-client constraint not enforced** — Architecture spec mentions "single client per daemon attach at a time (multi-client deferred)". Implementation doesn't enforce this; multiple clients can attach to same daemon. Risk: Not enforced by design, only by documentation. Lesson: Document enforcement mechanism in spec if it's not code-enforced.

### Patterns to Reuse

1. **Session-filtered event forwarding**:
   ```typescript
   // Core.bus emits all events
   core.bus.on(event => {
     // Filter by sessionId
     if (event.sessionId === attachedSessionId) {
       ipcClient.notifyEvent(event);  // Forward to attached client
     }
   });
   ```
   Reusable for multi-client scenarios where each client should receive different events. Pattern scales to multi-agent, multi-tenant, any scenario with event filtering.

2. **Terminal I/O proxy pattern**:
   ```typescript
   // stdin → IPC
   readline.on('line', (input) => {
     ipcClient.call('agent.input', { sessionId, message: input });
   });
   // IPC → stdout
   ipcClient.on('event', (event) => {
     process.stdout.write(formatEvent(event));
   });
   ```
   Simple, effective. Reusable for any CLI tool that needs to proxy to remote agent.

3. **Auto-start with cleanup pattern**:
   ```typescript
   let autoStartedDaemon = false;
   try {
     if (!isDaemonRunning()) {
       await startDaemon();
       autoStartedDaemon = true;
     }
     await attachToAgent();
   } finally {
     if (autoStartedDaemon) {
       await stopDaemon();  // Clean up only if we started it
     }
   }
   ```
   Enables seamless UX without permanent footprint. Applies to any on-demand background service.

4. **RPC health check before operation**:
   ```typescript
   // Validate agent is responsive before attach
   const health = await ipcClient.call('agent.health', { agentId });
   if (!health.ok) {
     throw new Error(`Agent offline: ${health.reason}`);
   }
   ```
   Simple validation that prevents cascading errors. Pattern applies to any distributed system where you need to verify peer health before complex operation.

### Lessons for Session-Based IPC

1. **sessionId should be UUID** — Makes session tracking reliable, globally unique without central registry. Validates well in schema checks. Recommendation: Always use UUIDs for distributed session IDs.
2. **Event timestamp is non-negotiable** — Any event forwarded to client needs `timestamp: ISO8601` field for debugging, replay, audit trails. Add to core Five Aggregates event interface spec going forward.
3. **Error codes should be centralized** — Don't throw arbitrary error messages. Define ErrorCode const with all possible error codes and include in API documentation. Enables client-side error handling.
4. **Input validation must include size limits** — Without 64KB message size limit + 1MB event limit, malicious or buggy clients could flood daemon with huge payloads. Schema validation + runtime checks both needed.

### Root Cause of Phase 3 Conditional Pass

1. **CONDITIONAL (3 fixes)**:
   - **Fix 1**: AgentId validation before session creation — Check daemon alive + agent responsive before creating attach session. Prevents invalid session state.
   - **Fix 2**: Event timestamp field — Add `timestamp: ISO8601` to forwarded events for traceability. Enables audit trail.
   - **Fix 3**: Error codes documentation — Document 4 error codes (AGENT_OFFLINE, SESSION_TIMEOUT, INVALID_INPUT_TYPE, MESSAGE_SIZE_LIMIT_EXCEEDED) with exact error messages.

**Rework was minimal**: 3 small enhancements, no architecture changes. Indicates solid Phase 1 design; fixes were refinements, not reengineering.

### Action Items for Future Cycles

- [x] Add security checks to Phase 1 Architecture Spec checklist (validate all inputs before state creation)
- [x] Add timestamp field to all event interfaces in Five Aggregates (EventBase.timestamp required)
- [x] Create ErrorCode documentation standard (centralize all error codes, document in API spec)
- [ ] Plan14: Enforce single-client-per-daemon constraint via daemon registry (counter tracked per agent)
- [ ] Plan14: Add session timeout enforcement (auto-detach after 5 minutes inactivity) — currently spec says it, not implemented
- [ ] Plan15: Web-based remote attach (HTTP/WebSocket, encrypted channels, authentication)
- [ ] Plan15: Multi-client attach (concurrent sessions, shared event stream with client-specific filtering)

---

## 20260212_cycle18: Plan15 SDK Context Extensions & Provider Integration (v0.13.0-beta)

### What Went Well

1. **Lazy accessor pattern maturity** — The IPluginContext.providers accessor followed the established tools/guides pattern from Cycles 3-4. No design innovation needed; reusing proven pattern (readonly getter, interface-only, non-breaking) made Phase 1 design quick and confident.
2. **Deferred technical debt consolidation** — Cycle 17 deferred two issues (sampling provider integration, roots allowedPaths context). Cycle 18 tackled both directly in a focused scope. Clear dependency chain and rework-free implementation.
3. **SessionConfig as typed metadata interface** — Introducing SessionConfig as an optional interface (getSessionConfig/setSessionConfig helpers) was cleaner than ad-hoc session context wiring. Enables plugins to read session-level config without core state mutation.
4. **First-pass PASS with zero rework** — 894 → 915 tests (+21 new tests), all code review items PASS, no conditional fixes. Indicates Architecture Spec completeness and Phase 2 alignment.
5. **Provider registry kept plugin-owned** — Key architectural decision: provider registry lives in mcp-common or provider plugin, SDK only exposes IProviderRegistry interface. Core remains minimal (no provider implementations). Microkernel purity maintained perfectly.

### What Could Be Improved

1. **Sync environment issues encountered** — Phase 2.5 sync to agent_test encountered stale node_modules (yaml + vitest module resolution). Workaround: performed verification in agent_dev. Root cause: previous dev/test cycles left inconsistent state. Lesson: Need Infrastructure-as-Code for node_modules cleanup (Plan16+ task).
2. **Test count target was conservative** — Spec estimated 26 new tests, delivered 21 actual. Reason: Some test coverage was already partially present in baseline. Lesson: When deferring features across cycles, verify test overlap in baseline to avoid double-counting in projections.
3. **No design alternatives explored** — Phase 1 spec went directly to final design (lazy accessor). Could have evaluated alternative: `ISessionContext` containing providers + allowedPaths (more explicit but more coupling). Lesson: For SDK extensions, document design alternatives in Architecture Spec, even if consensus is obvious.

### Patterns to Reuse

1. **Lazy accessor pattern for cross-plugin registries**:
   ```typescript
   interface IPluginContext {
     readonly providers?: IProviderRegistry;
     readonly tools?: IToolRegistry;
     readonly guides?: IGuideRegistry;
   }
   ```
   This pattern is now proven across 4 cycles. Future registry needs (prompts, resources, scripts, etc.) should follow: readonly getter, interface-only exposure, lazy instantiation in core, never expose mutations. Reusable for any plugin-owned resource registry.

2. **SessionConfig fallback chain pattern**:
   ```typescript
   const allowedPaths = ctx.session?.config?.allowedPaths
     || agentConfig.fileSystem.allowedPaths
     || [process.cwd()];
   ```
   Safe degradation from session → agent → defaults. Applies to any hierarchical configuration (session > user > global). Eliminates need for defensive nulls; clear priority chain.

3. **Provider fallback in handlers**:
   ```typescript
   const result = ctx.providers
     ? await ctx.providers.get('sampling')?.invoke(req)
     : null;
   return result || await fallbackLLMInvocation(req);
   ```
   Clean two-tier delegation: check capability-specific provider first, fall back to default behavior. Pattern scales to any pluggable fallback system.

4. **Sandbox RPC for plugin capability discovery**:
   ```typescript
   // IPC bridge: sandbox.providers.list() → returns all available providers
   // enables safe provider inspection without direct require('provider-package')
   ```
   Separates capability discovery (allowed in sandbox) from provider instantiation (requires IPluginContext). Keeps sandbox confined while enabling plugin introspection.

### Lessons for SDK Extension Design

1. **Optional fields are safer than union types** — SessionConfig with optional allowedPaths field is simpler and more forward-compatible than SessionConfig | undefined or trying to extend IPluginContext with conditional fields. Lesson: When adding to SDK, prefer optional properties over type unions.

2. **Readonly accessors prevent accidental mutations** — IPluginContext.providers is readonly to prevent plugins from calling `ctx.providers.unregister()` (which doesn't exist anyway). But readonly in TypeScript is a compile-time guarantee, not runtime. Lesson: Document that "readonly" means "don't mutate"; for runtime safety, return Object.freeze() wrapped registries in future if mutation surface is added.

3. **Interface-only exposure decouples implementations** — IProviderRegistry in SDK is an interface; actual ProviderRegistry is in core and plugin layer. Allows core to swap implementations or plugins to provide their own registries. Lesson: SDK should expose interfaces, not implementations. Applies to all future registry additions.

4. **Session context should be separate from plugin context** — This cycle accessed session config via ctx.session (assuming it exists). Future cycles may want to separate IPluginContext (plugin-specific) from ISessionContext (session-specific). Lesson: Document IPluginContext boundary; if session context grows, consider splitting interfaces in Plan16+.

### Root Cause Analysis: Zero Issues

No FAIL or CONDITIONAL items. Phase 3 verification was straightforward:
- QA: 915 tests PASS (no regressions, comprehensive coverage)
- Architect: 100% spec compliance, 3 non-blocking advisories deferred to Plan16+ (provider access control scoping, session config validation, advanced async patterns)

**Why clean PASS?**: Architecture Spec was comprehensive, frozen interfaces were fully implemented, no surprise design mismatches, implementation followed spec exactly, test coverage was adequate.

### Non-Blocking Issues Deferred to Plan16+

1. **Provider access control** — Could add scoping to prevent sampling handler from calling file system providers or vice versa. Deferred: complexity vs benefit not justified yet; validate in Plan16.
2. **Session config validation at startup** — SessionConfig.allowedPaths could be validated against actual file system permissions at agent startup. Deferred: potential race condition with changed perms; revisit in Plan16.
3. **Advanced async provider patterns** — MCP sampling with provider-based completions (e.g., Claude Claude-to-Claude) opens new UX possibilities. Deferred: design needed; not scope for Plan15.

### Action Items for Plan16+

- [ ] **Infrastructure fix**: Create node_modules cleanup script + CI hook (prevents stale dependencies in dev/test cycles)
- [ ] **Documentation**: Add "SDK Extension Design Guidelines" (lazy accessors, interface-only, optional fields, readonly)
- [ ] **Validation**: Plan16 session config hardening (validation at startup + dynamic permission checks)
- [ ] **Refactoring**: Consider separating ISessionContext from IPluginContext if session concerns grow
- [x] **Architectural**: Sampling + Roots handlers now complete with real provider delegation — DONE
- [x] **Provider registry maturity**: Established as stable, plugin-owned pattern — DONE

---

## 20260212_cycle22: Plan19 — Plugin Dependency Wiring & Cross-Plugin Services (v0.17.0-beta)

### What Went Well

1. **Spec Addendum process worked well** — Architect issued Spec Addendum recommending interface-based IServiceRegistry pattern instead of string-ID pattern from original Architecture Spec. Coordinator approved; implementation proceeded with superior design. Process enabled rapid design iteration without full rework.

2. **IServiceRegistry pattern was architecturally superior** — Full-featured interface (register, get, has, list) proved more flexible and type-safe than string-ID lookup initially specified. Enabled better IDE support and future extensibility with metadata.

3. **Topological sort + circular dependency detection** — Kahn's algorithm implementation (145 LOC) cleanly handles plugin load ordering with explicit error reporting on cycles. Proven pattern; future dependency graphs can reuse this.

4. **Service registry is now plugin-owned pattern** — ServiceRegistry lives in plugin system, exposed via SDK interface. Reduces core coupling and enables plugins to extend services independently. Pattern established: core exposes interface, plugins implement.

5. **Rework cycle was quick and productive** — 1 rework cycle (1 day) identified 2 items: missing has() method and field name alignment. Both straightforward fixes; no architectural rework needed. Verification was clean PASS.

6. **Test coverage comprehensive despite rework** — Final 1067 tests (65 new for Plan19, +58 from initial + 7 from rework) across service registry, topological sort, plugin loader, and e2e injection tests. No regressions detected.

### What Could Be Improved

1. **tsbuildinfo stale state issue in agent_test** — After sync from agent_dev to agent_test, build encountered stale tsbuildinfo files causing "incremental build mismatch" errors. Workaround: clean agent_test/openstarry node_modules before sync. Lesson: Sync script should validate or auto-clean tsbuildinfo artifacts.

2. **Spec Addendum communication flow** — Took initial review → coordinator approval → rework cycle before final design was clear. Lesson: When architect recommends design changes mid-cycle, escalate to coordinator immediately for explicit approval to avoid implementation false starts.

3. **Field name consistency** — PluginManifest initially used mixed naming (services vs provided, serviceDependencies vs consumed). Fixed in rework but added extra cycle. Lesson: Architect should flag naming conventions in Phase 1 (Design) to catch misalignments before implementation.

### Patterns to Reuse

1. **Spec Addendum for design pivot** — Coordinator + Architect can issue Spec Addendum when implementation reveals superior design without blocking development. Process: Architect issues recommendation → Coordinator approves in writing → dev implements approved design → no re-planning needed. Reusable for similar mid-cycle design refinements.

2. **Kahn's algorithm for plugin dependency ordering** — O(V+E) topological sort with cycle detection via in-degree tracking. Proven pattern for ordered plugin initialization. Reusable for any DAG-based dependency system (workflows, services, providers).

3. **Circular dependency detection with clear error reporting** — Error message: "Circular dependency detected: X → Y → Z → X". Provides actionable debugging info. Pattern reusable for any topological sort with human-visible feedback.

4. **Interface-based service registry** — Expose IServiceRegistry (interface) in SDK, implement in core/plugin layer. Enables swappable implementations and plugin-owned registries. Pattern scales to prompts, resources, guides registries.

### Lessons for Rework Cycles

1. **Missing method discovery in code review** — Architect code review caught missing has() method (part of IServiceRegistry interface contract). Lesson: Code review should enumerate interface methods vs implementation as explicit check.

2. **Field alignment across files** — Field naming mismatch (services vs provided) spanned PluginManifest → types → tests. One fix rippled across 3 files. Lesson: Use grep-based checklist during Phase 2 implementation (search all references to catch naming early).

### Root Cause of Phase 3 Rework

Architect found 7 items in initial review:
- 4 items: Missing has() method (architectural gap)
- 3 items: Field naming inconsistencies (implementation error)

Both resolved in 1 rework cycle via Code Fix (no design rework needed). QA re-verification confirmed all fixes; no new issues.

### Action Items for Plan20+

- [x] **Build Infrastructure**: Add tsbuildinfo cleanup to sync-to-test.sh (prevents agent_test build failures) — DONE (Cycle 23)
- [ ] **Coordinator checklist**: When Spec Addendum issued, update Architecture Spec explicitly in repo before implementation starts
- [ ] **Code review checklist**: Interface method enumeration (all methods implemented per interface contract)
- [ ] **Dev checklist**: Use grep to verify field naming consistency across files (manifest, types, tests, loaders)
- [ ] **Documentation**: Update Architecture Spec template to include "Fields & Naming Conventions" section
- [x] **Service registry**: Stable pattern established (interface-based, plugin-owned, core-exposed) — DONE
- [x] **Dependency ordering**: Kahn's algorithm proven for DAG-based plugin loading — DONE

---

## 20260212_cycle23: Plan20 — Workflow Engine MVP (v0.18.0-beta)

### What Went Well

1. **Pure plugin implementation maintained** — Zero SDK/Core modifications despite complex feature (workflow orchestration, streaming API). Microkernel purity preserved perfectly.
2. **Flexible spec allowed pragmatic rewrite** — When user identified architectural improvements (Mustache vs custom syntax, service/llm steps vs transform/condition, LRU vs file persistence), no rigid adherence to frozen spec allowed rapid architectural improvements with architect approval.
3. **Build error diagnosis was systematic** — 6 build issues from rewrite were methodically diagnosed and fixed:
   - Mustache HTML escaping disabling needed for data workflows (not HTML)
   - IProvider.chat() streaming API required AsyncIterable + text_delta event collection
   - Message type requires `id` field (randomUUID)
   - AgentEvent requires `timestamp` field
   - LRUCache undefined check on empty eviction
   - tsbuildinfo cleanup needed in sync pipeline
4. **Post-rewrite code quality superior** — Despite 50% reduction in test count (67→37), comprehensive coverage maintained with simpler, more maintainable codebase.
5. **Architect code review was constructive** — 10/10 checklist PASS despite spec divergence; architect justified PASS on pragmatic grounds (superior design, production quality, core goals achieved).
6. **Documentation captured real lessons** — Dev logs and architect review thoroughly documented the architectural rewrite, enabling future decisions on similar divergences.

### What Could Be Improved

1. **Spec adherence vs pragmatism tension** — Original frozen spec prescribed step types (transform, condition, prompt, return) that were entirely different from implementation (tool, service, llm, command). While the user's rewrite was architecturally superior, this could have been discussed earlier during Phase 1 design review.
   - **Lesson**: Architect should flag "frozen spec may not be optimal" concerns during Phase 1, allowing early Spec Addendum discussion before implementation.
2. **Mustache HTML escaping subtlety** — Default Mustache behavior broke non-HTML workflows (encoding `/` as `&#x2F;`). The fix (disabling escaping) was correct but not obvious from library documentation.
   - **Lesson**: When using template engines for non-HTML data, explicitly document escaping strategy in code comments (DONE in this cycle).
3. **mcp-common tsbuildinfo recurring issue** — This is the 3rd cycle with sync-to-test.sh hitting stale .tsbuildinfo. Cleanup step was added but should be proactive in future syncs.
   - **Lesson**: Sync script should unconditionally clean all .tsbuildinfo files before rebuild (implement in Phase 5 tooling).
4. **Command step placeholder confusion** — Command step is defined in schema but throws error. While intentional for MVP forward-compatibility, could confuse users.
   - **Lesson**: Add clear documentation (README) with "not yet supported" marker. Consider schema-level validation warning or explicit Future type marker.

### Patterns to Reuse

1. **Spec Rewrite Approval Process** — When frozen spec diverges significantly from implemented design:
   - Document divergence explicitly (architect review should list all divergences)
   - Justify divergence on quality/pragmatic grounds
   - Get explicit PASS despite divergence
   - Update iteration log to record the decision
   - Reusable for future cycles with similar situations (Plan21+ conditional logic, persistence, etc.)

2. **Mustache Interpolation Pattern** — For non-HTML template data:
   ```typescript
   // Disable Mustache's default HTML escaping — workflow data is not HTML.
   Mustache.escape = (value: string) => value;
   ```
   - Reusable pattern for any data-oriented templating in future plugins

3. **Streaming API Collection Pattern** — For streaming providers like LLM:
   ```typescript
   let responseText = "";
   for await (const event of provider.chat(request)) {
     if (event.type === "text_delta") {
       responseText += event.text;
     }
   }
   ```
   - Reusable pattern for any provider that returns AsyncIterable

4. **Discriminated Union Type Safety** — TypeScript discriminated unions enable exhaustiveness checking:
   ```typescript
   export type IWorkflowStep = IToolStep | IServiceStep | ILLMStep | ICommandStep;
   // Matches Zod schema with same discriminator
   const result = WorkflowSchema.safeParse(data); // Ensures runtime-TypeScript alignment
   ```
   - Apply this pattern to any plugin with multiple variants (service types, message types, etc.)

5. **Error Context Preservation** — Workflow errors capture step context:
   ```typescript
   error: { message: string; step?: string; cause?: unknown }
   ```
   - Reusable pattern for any engine with sequential processing (future workflow enhancements, batch processors, etc.)

### Action Items for Plan21+

- [x] **Build Infrastructure**: tsbuildinfo cleanup in sync-to-test.sh (DONE this cycle)
- [ ] **Spec Divergence Handling**: Document approval process in Agent SOP (recommend: Architect flags divergence early, Coordinator approves, recorded explicitly)
- [ ] **Workflow-engine README**: Document step types, Mustache syntax, examples, command step limitation (MINOR-1 from architect review)
- [ ] **LRU Cache Configurability**: Make cache size configurable via manifest or context (MINOR-4 from architect review)
- [ ] **Sync Script Hardening**: Unconditionally clean .tsbuildinfo files before rebuild (recurring issue #3)
- [ ] **Conditional Logic Design**: Plan21 should decide between condition step type, service-based branching, or JavaScript expressions (deferred from MVP)
- [ ] **Persistence Layer**: Plan21 should implement optional file-based or database execution history (deferred from MVP)

---

## 20260311_cycle1: Plan32 — Tenet #7 絕對純淨 + Tenet #9 部分修復 (v0.32.0-alpha)

### What Went Well

1. **Consolidated cycle02-8 research was comprehensive** — Research team's 8-cycle consolidation (D5 + 02-8_a D1~D3 + 02-8_b E1_v2 + 02-8_c Wave 6) provided complete specifications for all 6 waves. No ambiguity during implementation; clear BABBAGE classification and SUSSMAN three-layer architecture.

2. **BABBAGE continuity test proved sound** — Using BABBAGE's mathematical definition ("if removing constant breaks system math, it's mechanism; else policy") cleanly separated 8 mechanism values (preserved in Core) from 34 policy values (migrated to SDK). No edge cases or ambiguities during classification.

3. **SUSSMAN three-layer config pattern validated across all subsystems** — IAgentConfig → SDK DEFAULT_* → Core zero-default pattern was implemented consistently for all 6 waves:
   - Wave 1: Auditor auto-mount removal (Optional-degraded level)
   - Wave 2: 3 plugin packages (threshold, passthrough, monitor)
   - Wave 3: P0 20 values (calibration + safety-critical)
   - Wave 4: P1+P2 34 values (remaining policies)
   - Wave 5: Schema expansion (audit:completed payload)
   - Wave 6: Context manager extraction (Required level)

   Pattern proved to be the architectural standard for all future SDK extensions.

4. **Architecture review caught 2 critical issues early** — Architect identified:
   - Dead field in IAgentConfig.stalenessMs (not used in any Wave) → removed during Phase 2
   - Missing plugin-context-proxy.ts in Wave 6 delivery → added before verification

   Both caught in Code Review Phase (not implementation); minimal rework needed.

5. **Build & test coverage exceeded expectations** — 34 projects PASS, 1803 tests passed, purity check PASS. No regressions despite large scope (all 6 waves, 42 values processed, 8 new SDK constants).

6. **Microkernel purity maintained perfectly** — Despite Tenet #7 intensity (removal of all fallbacks, all constants, all auto-mounts), Core remained clean. All externalized to plugins and SDK defaults; no policy leakage into Core logic.

### What Could Be Improved

1. **Research team documentation sync workflow** — Doc-keeper discovered that `share/openstarry_doc/` was overwritten with cycle02-8_final deliverables at start of this cycle. While complete and high-quality, this fragmented the update process. Lesson: Coordinate explicitly with research team on when docs are pushed; doc-keeper should be notified in advance to prevent mid-cycle overwrites.

2. **Plan32 scope sprawl across 6 waves** — While each wave was well-defined, the cumulative scope (Tenet #7 + Tenet #9 partial) spanned very different concerns (auto-mounts, plugins, policies, schema, context manager). Future large-scope plans should consider splitting into multiple smaller plans to improve iterability.

3. **Wave 3 BABBAGE classification could have been published earlier** — The BABBAGE continuity test definition (lines 75-77) was perfect for standardization but only appeared in Phase 0 research. Lesson: When new classification methodology is proven, promote it to Architecture_Documentation for future reference immediately.

### Patterns to Reuse

1. **BABBAGE Continuity Test for Mechanism/Policy Classification** — Formal definition applicable to any value/constant in the system:
   - **Mechanism**: Removing it breaks mathematical definition (division by zero, NaN propagation, identity element removal)
   - **Policy**: Removing it changes behavior but system remains defined

   Reusable for: classifying future SDK constants, validating migrations, auditing Core constants

2. **SUSSMAN Three-Layer Configuration Architecture** — Proven pattern for all SDK extensions:
   ```
   User Config (IAgentConfig)  →  SDK Defaults (DEFAULT_*)  →  Core Zero-Default
   ```
   Reusable for: all future policy migrations, SDK constant management, runtime configuration design

3. **Three-Level Criticality Model for Missing Components** — Established clear behavior tiers:
   - **Required**: throw Error() at start() (context manager)
   - **Optional (degraded)**: neutral value like delta=0 (auditors, monitors)
   - **Optional (no-effect)**: feature unavailable (sensors)

   Reusable for: future plugin criticality design, component fallback strategies

4. **Architecture Review Checklist for Tenet Compliance** — Code review should enumerate:
   - [ ] Core contains zero policy constants (only mechanism values)
   - [ ] All policy values in SDK DEFAULT_* or user config
   - [ ] All auto-mounts removed
   - [ ] All fallback defaults eliminated (Required level errors instead)
   - [ ] Plugin extraction complete (no hardcoded implementations)

   Reusable checklist for Plan33+

### Root Cause Analysis: Zero Failed Items

No FAIL or rework cycles. Phase 4 convergence PASS:
- **QA**: 1803 tests PASS (no regressions)
- **Architect**: 100% spec compliance, 2 issues caught and fixed during code review (not in rework)
- **Build**: 34 projects PASS, purity PASS

**Why clean PASS?**: Research team provided comprehensive specifications for all 6 waves; BABBAGE classification was mathematically sound; SUSSMAN pattern was systematic; architect review caught edge cases early; implementation was straightforward.

### Action Items for Plan33+

- [x] **BABBAGE Classification Methodology**: Document formally in Architecture_Documentation as standard for mechanism/policy decisions (RECOMMENDED for Plan33+)
- [x] **SUSSMAN Three-Layer Pattern**: Established as architectural standard for SDK configuration (applies to all future SDK extensions)
- [x] **Three-Level Criticality Model**: Recorded for future plugin component design
- [x] **Plan32 Snapshot**: `share/openstarry_code_iteration/20260311_cycle1/` created
- [x] **DOC SOP Lesson**: Always sync research team docs first before doc-keeper updates; coordinate timing with research team
- [x] **Tenet #7 Audit**: Final confirmation that Core contains zero policy constants (AC-3 validation confirmed in Plan33)
- [x] **Tenet #9 Completion**: Partial Tenet #9 work in Plan32; full pluggability deferred to Plan38

---

## 20260324_cycle03-1: Plan37 — Phase 6 Multi-Agent Foundation (v0.37.0-alpha)

### What Went Well

1. **Frozen interface design was comprehensive and stable** — Doc 57 (Multi_Agent_Communication_Interface_Spec) provided complete definitions for all 5 core interfaces (ICommChannel, IDaemonControlPlane, ICompositeAgent, ITypedListener, CommMessage). Implementation was nearly mechanical; no surprises during Phase 2.

2. **Plugin relocation (PipelineChannel) reinforced Tenet #2/#7** — Moving PipelineChannel from core to openstarry_plugin/ reduced core complexity by ~200 LOC and eliminated potential circular dependencies. Clean boundary enforcement matched established architectural patterns from Plan32.

3. **BUG-3 fix was high-impact and well-justified** — The per-tool audit mechanism fix (symmetric clamp enforcement + must-invoke validation) prevented cascading failures in nested agent scenarios. Risk assessment in Phase 0 correctly identified this as worst-risk; resolution was surgical and non-invasive.

4. **ProcessTree + parent tracking was elegant** — Parent-child relationship management required only ~80 LOC. IDaemonControlPlane interface enabled process-like lifecycle management without adding complexity to Core.

5. **Graceful shutdown pattern with timeout validation** — max gracePeriodMs = 300000 (5 min) provided safety bounds without over-engineering. Simple validation in constructor prevented hung agents in production.

6. **First-pass PASS with zero rework cycles** — QA (2094 tests green) + Code Review (all interfaces verified) + Architect confirmed purity maintained. No FAILs, no design issues, no escalations needed.

7. **New baseline rules were well-documented** — 9 new rules (#31-#39) directly traced to implemented components (ProcessTree, CommMessage, EventBridge, etc.). Clear enforcement points in code made rules practical and verifiable.

### What Could Be Improved

1. **CommMessage.source naming collision** — Using "source" instead of "from" initially caused confusion with HTTP protocol semantics. EventBridge federation had to map `CommMessage.source` → HTTP `X-Source-Agent` header explicitly. Lesson: When bridging protocols, conduct a cross-layer naming audit early. Apply naming from outermost protocol inward (HTTP → MCP → CommMessage).

2. **EventBridge subscription cleanup was incomplete initially** — First Phase 2 implementation forgot to unsubscribe listeners on shutdown, causing memory leaks in test cleanup. Code review caught this; lesson: Always add explicit teardown tests for subscription patterns. Pattern: `test('unsubscribes all listeners on shutdown', () => { ... bridge.shutdown(); expect(listeners).toBe(empty); })`.

3. **ITypedListener sensory type coverage light** — Only 8 tests for 5 sensory types. Would have benefited from property-based or data-driven tests covering all permutations (5 types × 8 message categories = 40 scenarios, implemented 8). Lesson: For enum-based interfaces, use vitest parametrized tests (`test.each()`) to cover all combinations automatically.

4. **GlobalServiceRegistry interface was deferred** — Spec included interface definition but implementation was stubbed for Phase 38. This created a gap in Phase 37 deliverables. Lesson: If interface is frozen but implementation deferred, document the deferral explicitly in DoD and mark as "Not Implemented—Future" in code comments.

### Patterns to Reuse

1. **Frozen Interface + Two-Phase Implementation Pattern**:
   - **Phase 1**: Publish SDK interfaces in packages/sdk (immutable contract)
   - **Phase 2a**: Core implementation in packages/core (concrete classes)
   - **Phase 2b**: Plugin implementations in openstarry_plugin/ (extensions)
   - **Phase 3**: Unit tests + integration tests in separate suites

   Reusable for all future multi-layer interface designs.

2. **Process Tree Parent Tracking**:
   ```typescript
   class ProcessTree {
     private parentId: Map<string, string> = new Map();
     getParent(agentId: string) { return this.parentId.get(agentId); }
     setParent(childId: string, parentId: string) { this.parentId.set(childId, parentId); }
   }
   ```
   Reusable for hierarchical structures (Fractal Social Structure — Tenet #10).

3. **Graceful Shutdown with Timeout Bounds**:
   ```typescript
   validateGracePeriod(ms: number) {
     const MAX_GRACE = 300000;
     if (ms > MAX_GRACE) throw new Error('Grace period max 5 min');
     return ms;
   }
   ```
   Reusable for all timeout configurations (prevents resource exhaustion).

4. **Subscription Lifecycle Cleanup Pattern**:
   ```typescript
   class EventBridge {
     shutdown() {
       this.subscriptions.forEach((sub) => sub.unsubscribe());
       this.subscriptions.clear();
     }
   }
   ```
   Reusable for all pub/sub patterns (prevents memory leaks in test cleanup).

5. **Cross-Protocol Naming Convention** — Establish once, apply everywhere:
   - Use outermost protocol's conventions (HTTP first)
   - Map inbound: HTTP Header → MCP Field → CommMessage Property
   - Document mapping in Architecture Spec (e.g., Doc 57 Appendix)
   - Verify in integration tests

### Root Cause Analysis: Zero Failed Items

No FAIL or rework cycles. Phase 4 convergence PASS:
- **QA**: 2094 tests PASS (+96 new, all green)
- **Code Review**: All 5 frozen interfaces verified, compliance 8/1/1 maintained
- **Build**: 38 packages PASS, purity check PASS (0 new violations)
- **Architecture Spec**: Doc 57 covered all components; implementation matched 1:1

**Why clean PASS?** Frozen interfaces from Phase 1 design eliminated uncertainty. BUG-3 risk identified early. Plugin relocation decision was orthogonal to core logic. Graceful shutdown timeout was conservative. No cross-cutting concerns introduced into Core.

### Action Items for Cycle 04 (Plan38)

- [ ] **CommMessage Protocol Audit**: Verify "source" naming is consistent across all bridges (EventBridge federation, MCP transport, HTTP headers). Document in Architecture_Documentation.
- [ ] **Subscription Lifecycle Test Template**: Create reusable test pattern for pub/sub cleanup. Add to Agent_Core_Components_Deep_Dive/02_Communication_Interface.md
- [ ] **ITypedListener Coverage**: Add property-based tests using `test.each(sensorTypes)` in Cycle 04 (no blocking rework, just expansion)
- [ ] **GlobalServiceRegistry Implementation**: Move from deferred to Phase 38 execution (created interface in 37, implement concrete class in 38)
- [ ] **Process Tree Documentation**: Add to Implementation_Examples/ with real multi-agent scenarios (parent-child spawn/attach patterns)
- [ ] **Tenet #3 AC-6 Deep Dive**: ITypedListener is advancing Tenet #3 (Five Aggregates sensory mapping). Create Architecture_Documentation entry elaborating this connection.

---

## 20260407_cycle03-5: Plan41 — Typed Registry + CV-5 Gear-Arbiter-Dynamic + Late-Joiner Snapshot (v0.41.0-alpha)

### What Went Well

1. **ServiceKey<T> migration was surgical despite wide blast radius** — Changing IServiceRegistry interface signature required updating all consumers (monorepo + 3 plugins: comm-proxy, workflow-engine, distributed-alaya). Upfront planning identified 11 call sites (8 in core/sdk, 3 in plugins). Updated 8 test files to match new signature. Zero regression tests failed; type system caught all mismatches at compile time.

2. **WIENER constant formula lesson validated** — Discovered that formula-derived constants (1.5 * SIGMA = 0.046547) created rounding errors vs spec (0.047). Applied lesson: Always use spec's hardcoded constants directly, never derive from formulas. This prevented Phase 3 Constraint Verification failure (C41-4).

3. **Audit callback pattern strengthened testability** — Plugins now receive ctx.pushInput as callback at construction time, not full ctx reference. Enables easier unit testing (mock callback) vs integration testing (real ctx). Applied consistently to gear-arbiter-dynamic and audit-event-emitter.

4. **Phase 3 in-phase rework achieved clean convergence** — Found 5 FAIL items during code review (FAIL-1 through FAIL-5), fixed all in Phase 3 without interface changes. Re-verification: 2350 tests PASS. No separate rework cycle needed. Clean Phase 4 PASS.

5. **Rule #52 (Snapshot Completeness) first enforcement successful** — First Plan where test snapshot inclusion was BLOCKING (Plan40 used DEFER). Successfully included 221 test files in snapshot. Process validated; Rule #52 pattern works.

6. **ENG-FAB triple-gate enforcement prevented fabrication** — ENG-FAB-1a/1b/1c gates caught 5 FAIL items before Phase 4:
   - ENG-FAB-1a: Artifact manifest vs actual files
   - ENG-FAB-2: Code diff vs reported changes
   - ENG-FAB-3: Artifact hash verification

### What Could Be Improved

1. **ServiceKey<T> migration blast radius required early enumeration** — Changing interface signature cascaded across monorepo + plugins. Lesson: For any Interface change (even frozen interfaces with Spec Addendum), enumerate all call sites before implementation. Create migration checklist:
   ```
   [ ] packages/sdk/types/registry.ts (interface definition)
   [ ] packages/core/services/ServiceRegistry.ts (implementation)
   [ ] 3 plugin call sites (comm-proxy, workflow-engine, distributed-alaya)
   [ ] 8 test files
   [ ] Example code in docs
   ```
   Reusable for Plan42+ interface changes.

2. **WIENER constant rounding introduced late-phase constraint failure** — Using formula derivation instead of spec-hardcoded value almost failed C41-4 (confidence threshold verification). Lesson: Establish constant sourcing rule:
   - **Spec-derived**: Use hardcoded value from spec (e.g., 0.047)
   - **Formula-derived**: Only if spec explicitly says "calculated as X * Y"
   - **Measured empirically**: Document source and validation method

3. **Late-Joiner Snapshot scenarios needed more coverage initially** — W4 tests (5 scenarios) were minimal. Code review found IDistributedAlaya missing snapshot() and restore() methods. Lesson: For snapshot-like features, use matrix-based testing:
   - 3 state types (consistent, diverged, empty)
   - 2 recovery patterns (state-led, request-led)
   - = 6 scenarios minimum
   Test count: (6 scenarios × 4 assertions per scenario) = 24 tests recommended.

### Patterns to Reuse

1. **Audit Callback Pattern** — Plugins emit audit events via callback:
   ```typescript
   const plugin = createGearArbiterDynamicPlugin(ctx);
   // Inside factory:
   const auditCallback = ctx.pushInput;
   return {
     manifest,
     factory: (ctx) => new GearArbiter(auditCallback, ctx.config)
   };
   ```
   Reusable for: any plugin needing structured audit trails, distributed state machines, rule evaluation engines.

2. **Constant Sourcing Decision Tree**:
   ```
   Does spec define constant value explicitly?
     YES → Use hardcoded value (e.g., WIENER = 0.047)
     NO → Does spec define formula?
       YES → Verify formula delivers spec-compliant result (test)
       NO → Derive empirically, document source + validation
   ```
   Reusable for: all future constants in Plan42+

3. **Interface Change Migration Checklist**:
   ```markdown
   - [ ] Interface signature changed (packages/sdk)
   - [ ] Core implementation updated
   - [ ] All monorepo consumers updated (via grep)
   - [ ] All plugin call sites updated
   - [ ] All example code in docs updated
   - [ ] Backward compat test (if applicable)
   - [ ] 0 compilation errors across workspace
   ```
   Reusable for: Release SOP, major interface refactoring.

4. **Snapshot Feature Test Matrix**:
   ```
   State Types: [consistent, diverged, empty]
   Recovery Patterns: [state-led, request-led, timeout-triggered]
   Matrix: 3 × 3 = 9 scenario combinations
   Per scenario: [setup, execute, verify state, verify audit log]
   ```
   Reusable for: Plan42 IDistributedAlaya consumer patterns.

### Root Cause Analysis: Clean Phase 4 PASS (5 FAIL in Phase 3, 0 escalations)

Phase 3 code review identified 5 FAIL items:
1. **FAIL-1 (has/unregister non-generic)**: Interface overly specific; relaxed to generic T|undefined
2. **FAIL-2 (hysteresis gap)**: Constant rounding; corrected to spec hardcoded value
3. **FAIL-3 (console.log→pushInput)**: Audit callback pattern; refactored for testability
4. **FAIL-4 (skandha samjna→samskara)**: Manifest classification; corrected to action/routing category
5. **FAIL-5 (IDistributedAlaya missing snapshot methods)**: Interface incomplete; added snapshot()/restore()

All fixed in Phase 3 without escalation. **Why?** ENG-FAB-2 (code review) caught pattern violations early; type system verified fixes; re-verification passed immediately.

### Action Items for Plan42

- [x] **ServiceKey<T> Migration Checklist Template**: Document in Agent_Roles_and_SOP.md (Rule #54 candidate for Plan42+)
- [x] **Constant Sourcing Decision Tree**: Add to Architecture_Documentation as standard practice
- [x] **Snapshot Feature Test Matrix**: Add to Implementation_Examples/ for future snapshot-based features
- [x] **Plan41 Snapshot**: `share/openstarry_code_iteration/20260407_cycle03-5_snapshot/` created
- [x] **Rule #52 Validation**: Confirmed Rule #52 (Snapshot Completeness) enforcement works; ready for Plan42+
- [x] **ENG-FAB Pattern**: Triple-gate enforcement (ENG-FAB-1a/1b/1c) validated; extend to Plan42

---

## 20260409_cycle03-6: Plan42 — CV-5 Fix + Stabilization (v0.42.0-alpha)

### Key Findings

1. **Phantom Integration Pattern (Finding 2-3)**: Code + tests can exist but never be invoked by the host system. Unit tests passing doesn't prove integration.
   - Evidence: A-9 fabrication reporter plugin had 100% unit test coverage (assertion-coverage.json 63KB generated) but was never invoked by the host ExecutionLoop
   - Root Cause: DEV-1b callback pattern documentation incomplete; onStateModify hook missing from integration path
   - Resolution: A-9 integration verification enforced (Rule #54 added), actual invocation traced and confirmed
   - Lesson: Integration tests must verify host→plugin communication, not just plugin internals

2. **Shadow Counting Design (Finding W2)**: Decoupling observation from decision authority breaks the priority deadlock while maintaining safety.
   - Per-gear priority queues prevent global deadlock while static authority remains source of truth
   - CV5_THRESHOLDS config enables dynamic routing without compromising type safety
   - Lesson: When design conflicts arise, temporal separation (observation ≠ decision) can resolve them

3. **Even Plan Discipline (Finding Rule #47)**: Full traceability + full test evidence caught test count delta and forced reconciliation.
   - Cycle 03-5: 2350 tests (with expected baseline)
   - Cycle 03-6: 2375 tests expected (+25 per plan)
   - Actual: All 2375 passed, but 3 skipped; rule #47 forced explanation
   - Lesson: Snapshot rule #52 (file count + test file count) should also track test execution counts

### What Went Well

1. **Phase 3 In-Phase Resolution**: 4 code review findings (DEV-1b wiring, CV-5 routing, observe mode, external consumer API) resolved same phase without interface changes or separate rework cycle.
2. **Root Cause Analysis Complete**: RC-1, RC-2, RC-3 documented with design impact; DEV-1b phantom pattern identified for Rule #54 enforcement.
3. **Security Audit Pass**: SEC-001 (assertion-coverage.json inclusion) fixed in gitignore; A-9 integration verified via artifact generation.
4. **ENG-FAB Enforcement**: A-9 integration verification through artifact (63KB generated); Rule #52 snapshot completeness confirmed (224 test files).
5. **Backward Compatibility**: Fabrication reporter as optional plugin (1 new plugin, 38→39) ensures Plan41 plugins remain compatible.

### What Could Be Improved

1. **Test Execution Tracking**: Rule #47 should include test execution counts (passed/failed/skipped) in addition to file counts.
2. **Integration Verification Checklist**: A-9 integration should be part of Phase 1 design checklist (not discovered in Phase 3.5).
3. **Shadow Decision Authority Documentation**: CV5_THRESHOLDS parameterization benefits from explicit design doc (Part 1: observation, Part 2: authority transfer).

### Root Cause Analysis

**Phantom Integration Pattern**:
- Code review (Phase 3) found DEV-1b reporter missing from ExecutionLoop state modification path
- Tests all passed because reporter unit tests exercise the reporter in isolation
- But the host system never calls the reporter (phantom integration)
- Rule #54 now mandates: Components must be actually invoked by the host, not just unit-tested in isolation

**CV-5 Deadlock**:
- Global priority queue caused deadlock: all gears competing for single queue head
- Per-gear queues allow each gear's requests to be prioritized independently
- Static authority (original CV-5 calculation) remains source of truth; observation (per-gear counts) is advisory only

**Test Count Reconciliation**:
- Expected: 2350 + 25 = 2375 tests
- Actual: 2375 passed, 3 skipped
- Rule #47 forced explanation: skipped tests are vitest-generated test placeholders (not test defects)
- Action: Next cycle should document skipped test reasons explicitly

### Patterns to Reuse

1. **A-9 Integration Verification**: Artifact generation (assertion-coverage.json) + size check ≥ 50KB indicates fabrication reporter actually ran + generated meaningful output.
2. **Per-Gear Isolation Pattern**: When global resource causes deadlock, isolate per-consumer queue with unified decision authority upstream (applies to priority queues, rate limiters, etc.).
3. **Observation ≠ Authority**: Shadow counting, confidence routing, and advisory systems should never mutate authoritative state. Keep decision authority separate.

### Action Items for Plan43

- [ ] **A-9 Integration Verification**: Add Phase 1 design checklist item: "Components must be invoked by host, not just unit-tested" (Rule #54 enforcement)
- [ ] **Test Execution Tracking**: Extend Rule #47 snapshot checklist to include test execution counts (passed/failed/skipped), not just file counts
- [ ] **Shadow Decision Authority Docs**: Create Implementation_Examples/Shadow_Decision_Authority_Pattern.md (observation vs authority separation)
- [ ] **DEV-1b Callback Documentation**: Add Agent_Roles_and_SOP.md section: DEV-1b callback integration checklist for state modification plugins

---
