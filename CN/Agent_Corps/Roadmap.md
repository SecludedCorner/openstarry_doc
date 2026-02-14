# OpenStarry — Roadmap & Version Tracking

Established: 2026-02-12

---

## Version History & Current Status

| Version | Cycle | Plan | Date | Status |
|---------|-------|------|------|--------|
| v0.1-alpha | Cycle 1-2 | Plan01-02 | 2026-02-04 to 2026-02-05 | ✅ COMPLETE |
| v0.2-alpha | Cycle 3-4 | Plan03-04 | 2026-02-05 to 2026-02-06 | ✅ COMPLETE |
| v0.2-beta | Cycle 5+ | Plan05 | 2026-02-07+ | ✅ COMPLETE |
| v0.2.1-beta | Cycle 1 (Rework) | Plan05.1, Plan05.2, Plan05.5-① | 2026-02-10 | ✅ COMPLETE |
| v0.2.2-beta | Cycle 2 | Plan05.5-②, Plan05.5-③ | 2026-02-11 | ✅ COMPLETE |
| v0.3.0-beta | Cycle 3 | Plan06-P1 (MCP Client) | 2026-02-11 | ✅ COMPLETE |
| v0.3.1-beta | Cycle 4 | Plan06-P2 (MCP Server) | 2026-02-11 | ✅ COMPLETE |
| v0.4 | Cycle 5 | Plan07 (Sandbox MVP) | 2026-02-11 | ✅ COMPLETE |
| v0.4.1-beta | Cycle 6 | Plan07.1 (Sandbox Hardening) | 2026-02-11 | ✅ COMPLETE |
| v0.4.2-beta | Cycle 7 | Plan07.2 (Sandbox Advanced) | 2026-02-11 | ✅ COMPLETE (1 rework) |
| v0.4.3-beta | Cycle 8 | Plan07.3 (Custom require + audit) | 2026-02-11 | ✅ COMPLETE |
| v0.5.0-beta | Cycle 9 | Plan08 (TUI Dashboard MVP) | 2026-02-11 | ✅ COMPLETE |
| v0.5.1-beta | Cycle 10 | Plan09 (Interactive TUI) | 2026-02-12 | ✅ COMPLETE |
| v0.6.0-beta | Cycle 11 | Plan10 (CLI Foundation & Runner) | 2026-02-12 | ✅ COMPLETE (1 rework) |
| v0.7.0-beta | Cycle 12 | Plan11 (DevTools & E2E Testing) | 2026-02-12 | ✅ COMPLETE (1 rework) |
| v0.8.0-beta | Cycle 13 | Plan12 (Daemon Mode MVP) | 2026-02-12 | ✅ COMPLETE (1 rework) |
| v0.9.0-beta | Cycle 14 | Plan13 (Seamless Attach) | 2026-02-12 | ✅ COMPLETE (1 rework) |
| v0.10.0-beta | Cycle 15 | Plan06-P3 (MCP Resources + OAuth) | 2026-02-12 | ✅ COMPLETE |
| v0.11.0-beta | Cycle 16 | Plan14 (Multi-client Attach & Session) | 2026-02-12 | ✅ COMPLETE |
| v0.12.0-beta | Cycle 17 | Plan06-P4 (MCP Sampling & Extensions) | 2026-02-12 | ✅ COMPLETE |
| v0.13.0-beta | Cycle 18 | Plan15 (SDK Context Extensions) | 2026-02-12 | ✅ COMPLETE |
| v0.14.0-beta | Cycle 19 | Plan16 (Security Hardening) | 2026-02-12 | ✅ COMPLETE (1 rework) |
| v0.15.0-beta | Cycle 20 | Plan17 (Plugin Developer Experience) | 2026-02-12 | ✅ COMPLETE |
| v0.16.0-beta | Cycle 21 | Plan18 (Plugin Sync & System Directory) | 2026-02-12 | ✅ COMPLETE |
| v0.17.0-beta | Cycle 22 | Plan19 (Plugin Dependency Wiring) | 2026-02-12 | ✅ COMPLETE (1 rework) |
| **v0.18.0-beta** | **Cycle 23** | **Plan20 (Workflow Engine MVP)** | **2026-02-12** | **✅ COMPLETE (spec rewrite)** |
| **v0.19.0-beta** | **Cycle 24** | **Plan21 (Web-based Remote Attach)** | **2026-02-12** | **✅ COMPLETE** |
| **v0.20.0-beta** | **Cycle 25** | **Plan22 (Plugin Marketplace MVP)** | **2026-02-13** | **✅ COMPLETE** |
| **v0.20.1-beta** | **Cycle 25 hotfix** | **Windows 跨平台修复 (43→0 failures)** | **2026-02-13** | **✅ COMPLETE** |
| v0.21.0-beta | Cycle 26 | Plan23 (TBD — 待研究团队建议确认) | TBD | 📋 PLANNED |
| v0.22.0+ | Future | Backlog (State Machines, Multi-agent, Advanced) | TBD | ⬜ BACKLOG |

---

## Roadmap Phases

### Phase 1: Core Foundation & Events (COMPLETE ✅)
- Plan01-05: MVP Alpha, Event-Driven, UI/Listener/Guide
- v0.1-0.2-beta: Foundation built
- Status: Delivered in Cycles 1-5

### Phase 2: MCP Integration & Sandbox (COMPLETE ✅)
- Plan06 (all phases): MCP Client → Server → Resources → Sampling
- Plan07 (all phases): Runtime Sandbox with security hardening
- v0.3-0.4.3-beta: MCP protocol complete, sandbox mature
- Status: Delivered in Cycles 3-8

### Phase 3: User Interfaces & Developer Tools (COMPLETE ✅)
  - **Phase 3a**: TUI & CLI (Cycles 9-11)
    - Plan08: TUI Dashboard MVP → v0.5.0-beta
    - Plan09: Interactive TUI → v0.5.1-beta
    - Plan10: CLI & Runner → v0.6.0-beta

  - **Phase 3b**: Developer Experience (Cycles 12-14)
    - Plan11: DevTools & E2E Testing → v0.7.0-beta
    - Plan12: Daemon Mode MVP → v0.8.0-beta
    - Plan13: Seamless Attach → v0.9.0-beta

  - **Phase 3c**: Session & Multi-client (Cycles 15-16)
    - Plan14: Multi-client Attach → v0.11.0-beta
    - Plan06-P3: MCP Resources → v0.10.0-beta

  - **Phase 3d**: SDK Hardening & Quality (Cycles 17-21)
    - Plan06-P4: MCP Sampling → v0.12.0-beta
    - Plan15: SDK Context Extensions → v0.13.0-beta
    - Plan16: Security Hardening → v0.14.0-beta
    - Plan17: Plugin DX (MockHost, scaffolding) → v0.15.0-beta
    - Plan18: Plugin Sync → v0.16.0-beta

  - **Phase 3e**: Plugin Services (Cycle 22)
    - Plan19: Dependency Wiring & Service Registry → **v0.17.0-beta** ✅ COMPLETE

### Phase 4: Advanced Features (COMPLETE ✅)
  - **Phase 4a**: Workflow & Orchestration (Cycle 23)
    - Plan20: Workflow Engine MVP → **v0.18.0-beta** ✅ COMPLETE

  - **Phase 4b**: Web & Remote Access (Cycle 24)
    - Plan21: Web-based Remote Attach → **v0.19.0-beta** ✅ COMPLETE

  - **Phase 4c**: Plugin Marketplace (Cycle 25)
    - Plan22: Plugin Marketplace MVP → **v0.20.0-beta** ✅ COMPLETE

### Phase 5: Next Generation (PLANNED 📋)
  - **Plan23**: TBD — 待研究团队建议确认后分配

  - **Backlog** (未排序，待评估优先级):
    - State Machines & Event Routing
    - Multi-agent Orchestration
    - Service Mesh Patterns
    - Plugin Signature Verification Integration
    - Network Fetch Sandbox

---

## Roadmap Milestones

### Complete Milestones

| Milestone | Cycles | Test Count | Status | Date |
|-----------|--------|-----------|--------|------|
| MVP Foundation | 1-5 | 407 tests | ✅ | 2026-02-11 |
| MCP Protocol | 3-8 | 442 tests | ✅ | 2026-02-11 |
| User Interfaces | 9-11 | 632 tests | ✅ | 2026-02-12 |
| DevTools & Daemon | 12-14 | 747 tests | ✅ | 2026-02-12 |
| Multi-client Session | 15-16 | 807 tests | ✅ | 2026-02-12 |
| SDK Hardening | 17-21 | 1009 tests | ✅ | 2026-02-12 |
| Plugin Services | 22 | **1067 tests** | ✅ | 2026-02-12 |
| Workflow Engine | 23 | **1104 tests** | ✅ | 2026-02-12 |
| Web Remote Attach | 24 | **1132 tests** | ✅ | 2026-02-12 |
| Plugin Marketplace | 25 | **1330 tests** | ✅ | 2026-02-13 |
| Windows Cross-Platform | 25 hotfix | **1332 tests** | ✅ | 2026-02-13 |

### Upcoming Milestones

| Milestone | Plan | Target Version | Status | Estimated Cycle |
|-----------|------|-----------------|--------|-----------------|
| Plan23 (TBD) | Plan23 | v0.21.0-beta | 📋 PLANNED | Cycle 26 |

### Backlog (未排序)

| Milestone | Target Version | Status |
|-----------|---------------|--------|
| State Machines & Event Routing | TBD | ⬜ BACKLOG |
| Multi-agent Orchestration | TBD | ⬜ BACKLOG |
| Service Mesh Patterns | TBD | ⬜ BACKLOG |
| Plugin Signature Verification | TBD | ⬜ BACKLOG |
| Network Fetch Sandbox | TBD | ⬜ BACKLOG |

---

## Test Growth Trajectory

```
Cycle 1-2:   118 tests    (Foundation)
Cycle 3:     200 tests    (MCP Client MVP)
Cycle 4:     252 tests    (MCP Server MVP)
Cycle 5:     320 tests    (Sandbox MVP)
Cycle 6:     351 tests    (Sandbox Hardening)
Cycle 7:     407 tests    (Advanced Hardening, +1 rework)
Cycle 8:     442 tests    (Custom require)
Cycle 9:     524 tests    (TUI Dashboard)
Cycle 10:    559 tests    (Interactive TUI)
Cycle 11:    632 tests    (CLI Foundation)
Cycle 12:    670 tests    (DevTools & E2E, +1 rework)
Cycle 13:    714 tests    (Daemon Mode, +1 rework)
Cycle 14:    747 tests    (Seamless Attach, +1 rework)
Cycle 15:    792 tests    (MCP Resources)
Cycle 16:    807 tests    (Multi-client Session)
Cycle 17:    894 tests    (MCP Sampling Extensions)
Cycle 18:    915 tests    (SDK Context Extensions)
Cycle 19:    935 tests    (Security Hardening, +1 rework)
Cycle 20:    970 tests    (Plugin DX)
Cycle 21:   1009 tests    (Plugin Sync)
Cycle 22:   1067 tests    (Plugin Services, +1 rework)
Cycle 23:   1104 tests    (Workflow Engine MVP, spec rewrite)
Cycle 24:   1132 tests    (Web Remote Attach)
Cycle 25:   1330 tests    (Plugin Marketplace MVP)
Cycle 25+:  1332 tests    (Windows Cross-Platform Fix)
```

---

## Architecture Phases

### Phase A: Five Aggregates (COMPLETE ✅)
- IUI (色) — User Interface handlers
- IListener (受) — Event listeners
- IProvider (想) — Service providers
- ITool (行) — Executable tools
- IGuide (识) — System prompts/guides
- Status: All interfaces defined and integrated (Cycles 1-5)

### Phase B: Plugin System Maturity (COMPLETE ✅)
- Factory pattern & manifest validation
- Sandbox isolation & security hardening
- Plugin loading & lifecycle management
- Cross-plugin communication (Plans 18-19)
- Status: Complete with service registry (Cycles 5-22)

### Phase C: Developer Experience (COMPLETE ✅)
- CLI foundation & runner infrastructure
- TUI dashboard & interactive input
- Daemon mode & seamless attach
- DevTools plugin & E2E testing framework
- MockHost testing utility & scaffolding CLI
- Plugin sync & system plugin directory
- Status: Mature DX tooling (Cycles 9-21)

### Phase D: Advanced Orchestration (COMPLETE ✅)
- Workflow engine (YAML-based) ✅ (Cycle 23)
- Web-based remote attach ✅ (Cycle 24)
- Plugin Marketplace MVP ✅ (Cycle 25)
- Status: All planned orchestration features delivered (Cycles 23-25)

### Phase E: Next Generation (PLANNED 📋)
- Plan23: TBD — 待研究团队建议确认
- Backlog: State machines, Multi-agent, Service mesh, Plugin signature, Network sandbox

---

## Key Deliverables by Cycle

### Cycle 22 — Plan19: Plugin Dependency Wiring & Cross-Plugin Services

**Version**: v0.17.0-beta

**Deliverables**:
- IPluginService interface (SDK)
- IServiceRegistry interface (SDK) with register/get/has/list
- ServiceRegistry class (Core) — in-memory implementation
- IPluginContext.services accessor
- PluginManifest.services/serviceDependencies fields
- PluginLoader.loadAll() with Kahn's topological sort
- Circular dependency detection
- 65 new tests (service registry, dependency ordering, e2e injection)
- **Rework Cycle 22.1**: Code Fix — added missing has() method, field alignment

**Files Created**: 5 new files
- `packages/sdk/src/types/service.ts`
- `packages/core/src/infrastructure/service-registry.ts`
- 3 test files

**Files Modified**: 6 files
- SDK types, errors, exports
- Core agent initialization, plugin loader, sandbox manager

**Quality Metrics**:
- Tests: 1009 → 1067 (+65, +5.8%)
- Test Files: 83 → 88 (+5)
- Rework Cycles: 1 (Code Fix)
- Blocking Issues: 0
- Backwards Compatibility: Full (all changes non-breaking)

**Status**: ✅ COMPLETE (Phase 4 Convergence PASS, 2026-02-12)

---

## Dependency Graph

```
Plan01-05 (MVP Foundation)
  ↓
Plan06 (MCP: Client → Server → Resources → Sampling) ⟵ Plan07 (Sandbox)
  ↓
Plan08-09 (TUI: Dashboard → Interactive)
  ↓
Plan10-11 (CLI + DevTools & E2E)
  ↓
Plan12-13 (Daemon + Seamless Attach)
  ↓
Plan14 (Multi-client Session) ⟵ Plan06-P3 (MCP Resources)
  ↓
Plan15 (SDK Context Extensions)
  ↓
Plan16 (Security Hardening)
  ↓
Plan17 (Plugin DX: MockHost + Scaffolding)
  ↓
Plan18 (Plugin Sync & System Directory)
  ↓
Plan19 (Plugin Dependency Wiring & Services) ✅
  ↓
Plan20 (Workflow Engine MVP) ✅
  ↓
Plan21 (Web-based Remote Attach) ✅
  ↓
Plan22 (Plugin Marketplace MVP) ✅
  ↓
Plan23 (TBD — 待研究团队建议确认) 📋 PLANNED
  ↓
Backlog: State Machines, Multi-agent, Service Mesh, Plugin Signature, Network Sandbox
```

---

## Next Steps

**Latest Release**: v0.20.1-beta — Windows Cross-Platform Fix (Cycle 25 hotfix) ✅

**Cycle 25 hotfix (v0.20.1-beta) Delivered**:
- Windows 跨平台修复：43 个失败测试 → 0 个失败
- 路径处理：`sep`、`basename()`、`pathToFileURL()` 取代硬编码 Unix 路径
- Daemon IPC：新增 `platform.ts` — Windows 用 named pipe，Linux 用 Unix socket
- 平台 guard：SIGHUP、chmod、mkdirSync、unlinkSync 在 Windows 上跳过
- `plugin-installer.ts`：`cp()` dereference fallback + `rm()` maxRetries
- 测试数量：1330 → 1332（+2，3 skipped）
- Snapshot: `20260213_cycle25_winfix`

**Cycle 25 Delivered**:
- Bundled plugin catalog (`plugin-catalog.json`) with all 15 official plugins
- Plugin lock file (`~/.openstarry/plugins/lock.json`) for tracking installed plugins
- Plugin installer with workspace-first resolution + npm fallback
- 5 new CLI commands: `plugin install`, `plugin uninstall`, `plugin list`, `plugin search`, `plugin info`
- `plugin install --all` installs all 15 official plugins at once
- Short name support (e.g., `standard-function-fs` resolves to `@openstarry-plugin/standard-function-fs`)
- 77 new tests (198 net growth: 1132 → 1330)
- Infrastructure fix: `sync-to-test.sh`, `snapshot.sh`, `baseline.sh` now auto-copy plugin `__tests__/`, `vitest.config.ts`, `configs/`, `README.md`

**Next Cycle**: Plan23 — 待研究团队建议确认后分配内容

**Backlog 候选** (未排序):
- State Machines & Event Routing (已完成探索，规格草案就绪)
- Multi-agent Orchestration (需 2-3 cycles)
- Service Mesh Patterns
- Plugin Signature Verification Integration (plugin-signer 已存在但未启用)
- Network Fetch Sandbox (allowedDomains 已定义但未实现)

**Status**: Plan01-22 全部完成 ✅ | 测试: 1330 | snapshot: 20260213_cycle25
