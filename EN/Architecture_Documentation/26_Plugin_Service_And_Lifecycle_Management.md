# 26. Plugin Service & Lifecycle Management

This document defines the **service sharing mechanism** and **dynamic lifecycle management** between plugins, providing the architectural foundation for complex multi-plugin collaboration scenarios.

---

## 1. Problem Background

### 1.1 Limitations of Existing Mechanisms

In the current architecture, Five Aggregates plugins collaborate indirectly via Registries:

```
Plugin A (Tool) ──Register──► ToolRegistry ◄──Query── Core
Plugin B (Guide) ──Register──► GuideRegistry ◄──Query── Core
```

While sufficient for standard aggregates, this does not handle the following scenarios:

| Scenario | Problem |
|------|------|
| **Service Sharing** | Docker Plugin provides an API for use by the WEB UI. |
| **Dynamic Management** | WEB UI enables/disables other plugins at runtime. |
| **Dependency Coordination** | Plugin A requires Plugin B to complete initialization first. |

### 1.2 Target Scenario

```
Agent (VM Environment)
│
├─ Composite UI Plugin
│   ├─ CLI UI ──────── Terminal Interface
│   └─ WEB UI ──────── Browser Management Interface
│       ├─ Plugin Management Panel ── Enable/Disable Plugins
│       ├─ Service Status Monitoring ── Query Docker/Nginx Status
│       └─ Visual Configuration ──── Edit Plugin Settings
│
├─ Docker Plugin ────── Container Management Service
├─ Nginx Plugin ─────── Reverse Proxy Service
└─ [Project Plugins...] ────── Dynamic Discovery and Loading
```

---

## 2. Service Registration & Discovery Mechanism

### 2.1 Extending IPluginContext

```typescript
export interface IPluginContext {
  // Existing fields
  bus: EventBus;
  workingDirectory: string;
  agentId: string;
  config: Record<string, unknown>;
  pushInput: (event: InputEvent) => void;

  // === New: Service Mechanism ===

  /**
   * Registers a service for use by other plugins.
   * @param id Unique service identifier.
   * @param service Service instance.
   */
  registerService<T>(id: string, service: T): void;

  /**
   * Retrieves a registered service.
   * @param id Service identifier.
   * @returns Service instance, or undefined if it does not exist.
   */
  getService<T>(id: string): T | undefined;

  /**
   * Plugin Manager (privileged plugins only).
   * This field is undefined if the plugin is not granted management permissions.
   */
  pluginManager?: IPluginManager;
}
```

### 2.2 Service Provider Example

```typescript
// docker-plugin/src/index.ts

interface DockerService {
  listContainers(): Promise<Container[]>;
  startContainer(id: string): Promise<void>;
  stopContainer(id: string): Promise<void>;
}

export function createDockerPlugin(): IPlugin {
  return {
    manifest: {
      name: "docker-plugin",
      version: "0.1.0",
      provides: ["docker-service"],  // Declare provided services
    },

    async factory(ctx: IPluginContext): Promise<PluginHooks> {
      const dockerService: DockerService = {
        async listContainers() { /* ... */ },
        async startContainer(id) { /* ... */ },
        async stopContainer(id) { /* ... */ },
      };

      // Register the service
      ctx.registerService("docker", dockerService);

      return {
        tools: [createDockerTool(dockerService)],
      };
    },
  };
}
```

### 2.3 Service Consumer Example

```typescript
// web-ui-plugin/src/index.ts

export function createWebUIPlugin(): IPlugin {
  return {
    manifest: {
      name: "web-ui-plugin",
      version: "0.1.0",
      dependencies: ["docker-plugin"],  // Declare dependencies
    },

    async factory(ctx: IPluginContext): Promise<PluginHooks> {
      // Retrieve the Docker service
      const docker = ctx.getService<DockerService>("docker");

      if (!docker) {
        throw new Error("Required service 'docker' not found");
      }

      return {
        ui: [createWebUI({
          docker,
          pluginManager: ctx.pluginManager
        })],
      };
    },
  };
}
```

---

## 3. Plugin Lifecycle Management

### 3.1 IPluginManager Interface

```typescript
export type PluginStatus =
  | "discovered"   // Discovered but not loaded
  | "loading"      // Currently loading
  | "running"      // Running
  | "stopped"      // Stopped
  | "error"        // Error state
  | "quarantined"; // Quarantined (loading failed)

export interface PluginDescriptor {
  name: string;
  version: string;
  path: string;
  status: PluginStatus;
  provides?: string[];      // Provided services
  dependencies?: string[];  // Dependent plugins
  error?: string;           // Error message (if any)
}

export interface IPluginManager {
  /**
   * Discovers available plugins.
   * @param searchPaths List of search paths.
   */
  discover(searchPaths: string[]): Promise<PluginDescriptor[]>;

  /**
   * Dynamically loads a plugin.
   * @param name Plugin name.
   * @throws If dependencies are not met or loading fails.
   */
  load(name: string): Promise<void>;

  /**
   * Unloads a plugin.
   * @param name Plugin name.
   * @throws If other plugins depend on this plugin.
   */
  unload(name: string): Promise<void>;

  /**
   * Reloads a plugin (unload followed by load).
   */
  reload(name: string): Promise<void>;

  /**
   * Gets the status of a plugin.
   */
  getStatus(name: string): PluginStatus;

  /**
   * Lists all plugins.
   */
  list(): PluginDescriptor[];

  /**
   * Listens for plugin status changes.
   */
  onStatusChange(
    callback: (name: string, status: PluginStatus) => void
  ): () => void;
}
```

### 3.2 Loading Sequence & Topological Sort

When plugins declare dependencies, the PluginManager must perform a topological sort:

```
Dependency Graph:
  web-ui-plugin
       │
       ├──► docker-plugin
       │
       └──► nginx-plugin

Loading Order:
  1. docker-plugin
  2. nginx-plugin
  3. web-ui-plugin
```

**Implementation Logic:**

```typescript
function topologicalSort(plugins: PluginDescriptor[]): string[] {
  const graph = new Map<string, string[]>();
  const inDegree = new Map<string, number>();

  // Build dependency graph
  for (const p of plugins) {
    graph.set(p.name, p.dependencies ?? []);
    inDegree.set(p.name, (p.dependencies ?? []).length);
  }

  // Kahn's algorithm
  const queue = [...inDegree.entries()]
    .filter(([_, deg]) => deg === 0)
    .map(([name]) => name);

  const result: string[] = [];

  while (queue.length > 0) {
    const current = queue.shift()!;
    result.push(current);

    for (const [name, deps] of graph) {
      if (deps.includes(current)) {
        inDegree.set(name, inDegree.get(name)! - 1);
        if (inDegree.get(name) === 0) {
          queue.push(name);
        }
      }
    }
  }

  // Detect circular dependencies
  if (result.length !== plugins.length) {
    throw new Error("Circular dependency detected");
  }

  return result;
}
```

---

## 4. Multi-Channel UI Collaboration

### 4.1 Scenario Description

The same Agent may have multiple UI channels simultaneously:

```
User A ──► CLI UI ──┐
                    ├──► Agent Core ──► Tools
User B ──► WEB UI ──┘
```

### 4.2 Event Broadcasting Mechanism

All UIs receive events via the `UIRegistry`:

```typescript
// transport/bridge.ts
function broadcast(event: AgentEvent): void {
  for (const ui of uiRegistry.list()) {
    ui.onEvent(event);  // Both CLI and WEB receive it
  }
}
```

### 4.3 Input Source Identification

Distinguish input sources via `InputEvent.source`:

```typescript
interface InputEvent {
  source: string;      // "cli" | "web" | "api" | ...
  inputType: string;
  data: unknown;
  replyTo?: string;    // Used for targeted replies
}
```

### 4.4 Targeted Replies (Future Extension)

If a reply needs to be sent only to a specific UI:

```typescript
interface AgentEvent {
  type: string;
  timestamp: number;
  payload?: unknown;
  targetUI?: string;   // If specified, only this UI receives it
}
```

---

## 5. Permission Control

### 5.1 Privileged Plugins

Not all plugins should be able to manage other plugins. Only **privileged plugins** can access the `pluginManager`:

```json
// agent.json
{
  "plugins": [
    {
      "name": "@openstarry-plugin/web-ui",
      "privileged": true  // Grant management permissions
    },
    { "name": "@openstarry-plugin/docker" }
  ]
}
```

### 5.2 Core Implementation

```typescript
function getPluginContext(
  pluginRef: PluginRef,
  isPrivileged: boolean
): IPluginContext {
  return {
    bus,
    workingDirectory: process.cwd(),
    agentId: config.identity.id,
    config: pluginRef.config ?? {},
    pushInput: (event) => core.pushInput(event),
    registerService: (id, svc) => serviceRegistry.register(id, svc),
    getService: (id) => serviceRegistry.get(id),
    pluginManager: isPrivileged ? pluginManager : undefined,
  };
}
```

---

## 6. Implementation Roadmap

| Phase | Content | Priority |
|-------|------|--------|
| **Phase 1** | ServiceRegistry (registerService / getService) | High |
| **Phase 2** | PluginManifest Dependency Declaration + Topological Sort | High |
| **Phase 3** | IPluginManager Basic Implementation (load/unload) | Medium |
| **Phase 4** | Plugin Discovery Mechanism (Scanning project folders) | Medium |
| **Phase 5** | Permission Control & Privileged Plugins | Medium |
| **Phase 6** | Multi-Channel Targeted Replies | Low |

---

## 7. Relationship with Existing Documents

| Document | Supplementary Explanation |
|------|----------|
| `19_Agent_Coordination_Layer.md` | Extends the "Plugin Registry Authority" concept, adding service registration. |
| `13_Composite_Plugins_and_Dependencies.md` | Provides specific API implementations. |
| `20_Dependency_Injection_and_Control_Loop.md` | Adds dynamic injection mechanisms. |
| `21_Plugin_Interface_Deep_Dive.md` | Extends IPluginContext. |

---

## 8. Design Principles

1. **Keep Core Pure** — The service registration mechanism resides in the Core layer and does not depend on specific plugin implementations.
2. **Progressive Adoption** — Existing plugins require no modification; the new mechanism is an optional extension.
3. **Explicit Dependencies** — Declared via manifest to avoid implicit coupling.
4. **Security First** — Plugin management requires explicit authorization.
