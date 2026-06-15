# Architecture Documentation #68: Typed Service Registry Design

> **✅ 已實作（2026-06-15 verify）**：本提案已落地，不再是開放提案。`IServiceRegistry` 定義於 SDK `packages/sdk/src/types/service.ts`（＋`SERVICE_KEYS`）；core 有 `serviceRegistry`（`service-registry.test.ts` / `typed-registry-consumer.test.ts` 實測）；distributed-alaya 已遷移、消除 `as any`（`distributed-alaya/src/index.ts` 以 `ctx.services.register` 註冊、消費端用 `SERVICE_KEYS.DISTRIBUTED_ALAYA`）。下文 §1/§5 的「Current Pattern（v0.39, `as any`）」為**提案當時的問題描述**，保留作設計理據；現況以本註記為準。

**Status**: Approved (D5-Q2, 5-0 unanimous) — **IMPLEMENTED**
**Priority**: HIGH (P1 for Plan41, deferred from Plan40 via D4-4)
**LOC Estimate**: ~105 LOC (SDK ~25 + Core ~50 + Migration ~30)
**Cycle**: 03-4

---

## 1. Problem Statement

### 1.1 The `as any` Type Escape

The distributed-alaya plugin (Plan39) exposes its runtime instance via an `as any` cast on the provider object:

```typescript
providers: [{
  skandha: 'samjna' as const,
  id: 'distributed-alaya-provider',
  // ...
  getDistributedAlaya: () => alaya,
} as any],
```

[程式碼: openstarry_plugin/distributed-alaya/src/index.ts#createDistributedAlayaPlugin]

The SDK's `IProvider` interface does not include `getDistributedAlaya()`. The cast erases the type contract at the package boundary, creating an **untyped channel** between the plugin (producer) and any future consumer.

### 1.2 Finding F-3 (MEDIUM)

Identified by SUSSMAN+KERNEL in R1, confirmed by BABBAGE (R2) and VITRUVIUS (R2):

| Aspect | Impact |
|--------|--------|
| **Discovery** | No consumer can discover `getDistributedAlaya()` through IDE autocompletion or type navigation. The method is invisible to TypeScript's type system. |
| **Refactoring safety** | If `getDistributedAlaya()` is renamed or its signature changes, no compile-time error surfaces at call sites. |
| **Proliferation risk** | As more plugins expose runtime instances via providers, the `as any` pattern replicates. Each plugin adds its own untyped method, creating an ad-hoc service registry without type safety. |

[來源: R1_SUSSMAN_KERNEL_architecture.md#F-3]

### 1.3 Full-Stack Proliferation (VITRUVIUS)

VITRUVIUS rated F-3 as **HIGH priority for v0.40**:

> "If a second plugin uses the `as any` provider pattern, the technical debt doubles. Establishing the service registry pattern early prevents proliferation."

The comm-proxy plugin (Plan37-38) uses a hook integration pattern (passively invoked by Core's execution loop) and does not need `as any`. However, any future plugin that exposes a runtime instance for external consumption -- the **provider integration pattern** -- will face the same `as any` pressure. The distributed-alaya plugin is the first; it will not be the last.

[來源: R2_09_VITRUVIUS_reviews_SUSSMAN_KERNEL.md#Section 5]

---

## 2. Options Evaluated

Three options were analyzed in R2 (BABBAGE) and debated in D5-Q2:

### Option 1: Declaration Merging (TypeScript Augmentation)

```typescript
// In distributed-alaya plugin:
declare module '@openstarry/sdk' {
  interface IProvider {
    getDistributedAlaya?: () => IDistributedAlaya;
  }
}
```

**Pros**: Zero new code; type-level only.
**Cons**: Pollutes the SDK interface with plugin-specific methods. Every plugin adding a method creates a growing `IProvider` that knows about all plugins. This **inverts the dependency direction** -- the SDK type retroactively learns about plugins through module augmentation.

**Verdict**: Rejected. Dependency inversion violates the layered architecture principle (SDK must not know about plugins).

[來源: R2_03_BABBAGE_reviews_SUSSMAN_KERNEL.md#F-3, R3_D5_architecture.md#D5-Q2]

### Option 2: Typed Service Registry (Adopted)

```typescript
interface IServiceRegistry {
  register<T>(key: ServiceKey<T>, factory: () => T): void;
  get<T>(key: ServiceKey<T>): T | undefined;
  deregister(key: ServiceKey<T>): void;
}
```

**Pros**: Clean separation; plugins register, consumers retrieve. The SDK interface remains stable regardless of how many plugins exist. Compile-time type safety via branded `ServiceKey<T>`. Standard pattern in mature plugin architectures (Eclipse, VS Code, IntelliJ, OSGi).
**Cons**: Requires ~105 LOC across SDK, Core, and migration.

**Verdict**: Adopted unanimously (5-0).

### Option 3: Module Augmentation (IPluginContext)

Similar to Option 1 but augments `IPluginContext` with a `services` property per plugin, rather than extending `IProvider` directly.

**Pros**: Less invasive than Option 1.
**Cons**: Relies on TypeScript's structural typing and module augmentation semantics, which may not survive bundler transformations. Still pollutes context type with plugin-specific knowledge.

**Verdict**: Rejected. Same dependency-direction concern as Option 1, with additional bundler fragility.

[來源: R2_03_BABBAGE_reviews_SUSSMAN_KERNEL.md#F-3]

---

## 3. Design Specification

### 3.1 `ServiceKey<T>` -- Branded Type

```typescript
/**
 * Branded type for compile-time service type safety.
 * The phantom type parameter T carries the service type
 * at the type level without runtime overhead.
 */
declare const ServiceKeyBrand: unique symbol;

export interface ServiceKey<T> {
  readonly id: string;
  readonly [ServiceKeyBrand]: T;
}

export function createServiceKey<T>(id: string): ServiceKey<T> {
  return { id } as ServiceKey<T>;
}
```

The branded type ensures that `registry.get(KEY_A)` returns `A | undefined` and `registry.get(KEY_B)` returns `B | undefined` at compile time. Mismatched key/type pairs are caught by the TypeScript compiler.

**Layer**: SDK (`@openstarry/sdk`)

### 3.2 `IServiceRegistry` Interface

```typescript
/**
 * Typed service registry for plugin-to-consumer service discovery.
 * Plugins register services; consumers retrieve them with typed keys.
 *
 * @layer SDK (interface definition)
 * @implemented-by Core plugin host (mechanism layer)
 */
export interface IServiceRegistry {
  /**
   * Register a service factory under a typed key.
   * @param key - Branded service key carrying the type parameter
   * @param factory - Lazy factory; invoked on first get() if desired
   * @param scope - Reserved for multi-Channel future. Default: 'local'
   */
  register<T>(key: ServiceKey<T>, factory: () => T, scope?: 'local' | 'global'): void;

  /**
   * Retrieve a service by typed key.
   * Returns undefined if the service is not registered.
   * Consumer MUST handle the undefined case (plugin load order).
   */
  get<T>(key: ServiceKey<T>): T | undefined;

  /**
   * Remove a service registration.
   * Called automatically on plugin unload via lifecycle hook.
   */
  deregister<T>(key: ServiceKey<T>): void;
}
```

**Layer**: SDK (`@openstarry/sdk`)
**Estimated LOC**: ~25 (interface + ServiceKey + createServiceKey)

### 3.3 Scope Parameter

| Value | Visibility | Plan |
|-------|-----------|------|
| `'local'` (default) | Within the registering process only | Plan41 (active) |
| `'global'` | Across all processes (Daemon-mediated) | Reserved for multi-Channel future |

The `scope` parameter is **reserved** in the interface signature. For Plan41, only `'local'` is implemented; `'global'` throws or is ignored. This anticipates the multi-Channel architecture (D5-Q4) without premature implementation.

[來源: R3_D5_architecture.md#D5-Q2, MESH position]

### 3.4 Implementation

**Layer**: Core plugin host (mechanism layer)

```
Implementation outline (~50 LOC):

class ServiceRegistryImpl implements IServiceRegistry {
  private readonly services = new Map<string, { factory: () => unknown }>();

  register<T>(key: ServiceKey<T>, factory: () => T, scope?: 'local' | 'global'): void {
    if (scope === 'global') {
      throw new Error('Global scope not yet supported');
    }
    this.services.set(key.id, { factory });
  }

  get<T>(key: ServiceKey<T>): T | undefined {
    const entry = this.services.get(key.id);
    return entry ? (entry.factory() as T) : undefined;
  }

  deregister<T>(key: ServiceKey<T>): void {
    this.services.delete(key.id);
  }
}
```

The implementation is **mechanism only** -- it provides key-based lookup with no knowledge of what services are registered. This preserves Tenet #7 (Microkernel Purity): the registry stores references and delegates; it makes no policy decisions.

**Estimated LOC**: ~50 (class + lifecycle integration with plugin host)

### 3.5 Lifecycle Management

When a plugin is unloaded, all services registered by that plugin must be deregistered. This prevents dangling references where a consumer calls `get()` for a service whose plugin has been torn down.

**Mechanism**: The plugin host tracks which `ServiceKey` entries were registered during each plugin's `activate()` phase. On plugin `deactivate()`, the host calls `deregister()` for all keys associated with that plugin.

[來源: R3_D5_architecture.md#D5-Q2, KERNEL position]

### 3.6 Deferred: Event-Driven Pattern

HERACLITUS proposed an `onAvailable(key, callback)` pattern for runtime dynamics -- consumers receive notification when a service becomes available, handling plugin load-order dependencies gracefully.

**Decision**: Deferred. Not needed for Plan41. The synchronous `get()` returning `T | undefined` is sufficient for the current consumer pattern. The event-driven pattern can be added when runtime dynamics demand it (e.g., hot-loading plugins at runtime).

[來源: R3_D5_architecture.md#D5-Q2, HERACLITUS position]

---

## 4. Dependency Direction

```
SDK (interface definition)
 │
 │  IServiceRegistry, ServiceKey<T>
 │
 ├────────────────────────────────────────┐
 │                                        │
 ▼                                        ▼
Core (implementation)                   Plugin (usage)
 ServiceRegistryImpl                     register(KEY, factory)
                                         ───
                                        Consumer code
                                         get(KEY)
```

**Direction verification**:
- SDK defines `IServiceRegistry` and `ServiceKey<T>` (types only, no implementation)
- Core implements `ServiceRegistryImpl` (imports from SDK -- correct direction)
- Plugins import `ServiceKey<T>` and `IServiceRegistry` from SDK (imports from SDK -- correct direction)
- Plugins never import from Core
- Core never imports from Plugins

This follows the established dependency DAG:

```
SDK (root, no incoming workspace edges)
 ├──> Core
 ├──> Plugins
 └──> Apps
```

No dependency inversions. No circular dependencies. Consistent with the v0.39 architecture verified by SUSSMAN+KERNEL (R1 Section 5.3) and VITRUVIUS (R2 Section 2.2).

[來源: R1_SUSSMAN_KERNEL_architecture.md#Section 5.3, R2_09_VITRUVIUS_reviews_SUSSMAN_KERNEL.md#Section 2]

---

## 5. distributed-alaya Migration

### 5.1 Current Pattern (v0.39, `as any`)

```typescript
// distributed-alaya/src/index.ts
providers: [{
  skandha: 'samjna' as const,
  id: 'distributed-alaya-provider',
  // ...
  getDistributedAlaya: () => alaya,
} as any],
```

Consumer (hypothetical):
```typescript
const provider = pluginHost.getProvider('distributed-alaya-provider');
const alaya = (provider as any).getDistributedAlaya();  // no type safety
```

### 5.2 Target Pattern (Post-Migration)

Service key definition (in SDK or plugin):
```typescript
import { createServiceKey, IDistributedAlaya } from '@openstarry/sdk';

export const DISTRIBUTED_ALAYA_KEY = createServiceKey<IDistributedAlaya>(
  'distributed-alaya'
);
```

Plugin registration:
```typescript
// distributed-alaya/src/index.ts
registry.register(DISTRIBUTED_ALAYA_KEY, () => alaya);
```

Consumer:
```typescript
import { DISTRIBUTED_ALAYA_KEY } from '@openstarry/sdk';

const alaya = registry.get(DISTRIBUTED_ALAYA_KEY);
if (alaya) {
  // TypeScript knows: alaya is IDistributedAlaya
  const seeds = alaya.query({ agentId: 'agent-1' });
}
```

### 5.3 Migration LOC

| Change | Location | LOC |
|--------|----------|-----|
| Remove `as any` provider hack | distributed-alaya/src/index.ts | -5 |
| Add `DISTRIBUTED_ALAYA_KEY` definition | SDK or plugin | +5 |
| Add `registry.register()` call | distributed-alaya/src/index.ts | +3 |
| Update consumer pattern (future) | Consumer code | ~5 per call site |
| Remove `getDistributedAlaya` from provider | distributed-alaya/src/index.ts | -3 |
| **Total migration** | | **~30 LOC** |

---

## 6. LOC Budget Summary

| Component | Layer | LOC |
|-----------|-------|-----|
| `IServiceRegistry` interface + `ServiceKey<T>` branded type + `createServiceKey()` | SDK | ~25 |
| `ServiceRegistryImpl` + lifecycle integration | Core | ~50 |
| distributed-alaya migration (`as any` removal + registry registration) | Plugin | ~30 |
| **Total** | | **~105** |

---

## 7. Timeline

| Item | Plan | Priority | Notes |
|------|------|----------|-------|
| IServiceRegistry + ServiceKey<T> design and implementation | Plan41 | P1 | Deferred from Plan40 scope (D4-4) |
| distributed-alaya migration | Plan41 | P1 | Same Plan as registry delivery |
| `scope: 'global'` implementation | Future (multi-Channel) | P3 | Reserved in interface, not implemented |
| `onAvailable()` event-driven pattern | Future (hot-loading) | P3 | Deferred per HERACLITUS |

The typed service registry was originally scoped for Plan40 (D5-Q2 decision). Per D4-4 triage, it was deferred to Plan41 to maintain Plan40's LOC budget focus on security-critical items (SEC-001, F-2 HMAC fixes, Daemon-distributed shared key).

---

## 8. Microkernel Alignment

The typed service registry is the microkernel analog of a **kernel name server** (MINIX, L4):

| Microkernel Concept | OpenStarry Analog |
|---------------------|-------------------|
| Name server | `IServiceRegistry` |
| Capability key | `ServiceKey<T>` |
| Service registration | `register()` by plugin during activation |
| Service lookup | `get()` by consumer code |
| Service deregistration | `deregister()` on plugin unload |

The registry provides **mechanism** (key-based lookup, lifecycle management) without **policy** (no knowledge of what services exist or how they are used). This preserves Tenet #7 (Microkernel Purity).

[來源: R3_D5_architecture.md#D5-Q2, KERNEL position]

---

## 9. Research Provenance

| Source | Contribution |
|--------|-------------|
| SUSSMAN+KERNEL R1 (F-3) | Identified `as any` cast as MEDIUM finding |
| BABBAGE R2 | Proposed three options (declaration merging, service registry, module augmentation) |
| VITRUVIUS R2 (Section 5) | Rated proliferation risk HIGH; recommended service registry for v0.40 |
| D5-Q2 debate | Evaluated all options; adopted Option 2 unanimously (5-0) |
| SUSSMAN (D5-Q2) | Rejected Option 1 (dependency inversion); championed Option 2 |
| KERNEL (D5-Q2) | Microkernel name server analogy; lifecycle deregister requirement |
| HERACLITUS (D5-Q2) | Temporal coupling analysis; deferred `onAvailable()` pattern |
| MESH (D5-Q2) | Scope parameter (`'local'` / `'global'`) for multi-Channel future |

---

*Architecture Documentation #68*
*Cycle 03-4*
*2026-04-04*
