# 26. 插件服务与生命周期管理 (Plugin Service & Lifecycle Management)

本文档定义了插件之间的**服务共享机制**与**动态生命周期管理**，为复杂的多插件协作场景提供架构基础。

---

## 1. 问题背景

### 1.1 现有机制的局限

当前架构中，五蕴插件通过 Registry 间接协作：

```
Plugin A (Tool) ──注册──► ToolRegistry ◄──查询── Core
Plugin B (Guide) ──注册──► GuideRegistry ◄──查询── Core
```

这对于标准五蕴插件已足够，但无法处理以下场景：

| 场景 | 问题 |
|------|------|
| **服务共享** | Docker Plugin 提供 API 给 WEB UI 使用 |
| **动态管理** | WEB UI 在运行时启用/禁用其他插件 |
| **依赖协调** | Plugin A 需要 Plugin B 先完成初始化 |

### 1.2 目标场景

```
Agent (VM 环境)
│
├─ 复合 UI 插件
│   ├─ CLI UI ──────── 终端界面
│   └─ WEB UI ──────── 浏览器管理界面
│       ├─ 插件管理面板 ── 启用/禁用插件
│       ├─ 服务状态监控 ── 查询 Docker/Nginx 状态
│       └─ 可视化配置 ──── 编辑插件设定
│
├─ Docker Plugin ────── 容器管理服务
├─ Nginx Plugin ─────── 反向代理服务
└─ [项目插件...] ────── 动态发现与加载
```

---

## 2. 服务注册与发现机制

### 2.1 扩展 IPluginContext

```typescript
export interface IPluginContext {
  // 现有字段
  bus: EventBus;
  workingDirectory: string;
  agentId: string;
  config: Record<string, unknown>;
  pushInput: (event: InputEvent) => void;

  // === 新增：服务机制 ===

  /**
   * 注册服务供其他插件使用
   * @param id 服务唯一标识符
   * @param service 服务实例
   */
  registerService<T>(id: string, service: T): void;

  /**
   * 取得已注册的服务
   * @param id 服务标识符
   * @returns 服务实例，若不存在则返回 undefined
   */
  getService<T>(id: string): T | undefined;

  /**
   * 插件管理器（仅限特权插件）
   * 若插件未被授予管理权限，此字段为 undefined
   */
  pluginManager?: IPluginManager;
}
```

### 2.2 服务提供者示例

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
      provides: ["docker-service"],  // 声明提供的服务
    },

    async factory(ctx: IPluginContext): Promise<PluginHooks> {
      const dockerService: DockerService = {
        async listContainers() { /* ... */ },
        async startContainer(id) { /* ... */ },
        async stopContainer(id) { /* ... */ },
      };

      // 注册服务
      ctx.registerService("docker", dockerService);

      return {
        tools: [createDockerTool(dockerService)],
      };
    },
  };
}
```

### 2.3 服务消费者示例

```typescript
// web-ui-plugin/src/index.ts

export function createWebUIPlugin(): IPlugin {
  return {
    manifest: {
      name: "web-ui-plugin",
      version: "0.1.0",
      dependencies: ["docker-plugin"],  // 声明依赖
    },

    async factory(ctx: IPluginContext): Promise<PluginHooks> {
      // 取得 Docker 服务
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

## 3. 插件生命周期管理

### 3.1 IPluginManager 接口

```typescript
export type PluginStatus =
  | "discovered"   // 已发现但未加载
  | "loading"      // 正在加载
  | "running"      // 运行中
  | "stopped"      // 已停止
  | "error"        // 错误状态
  | "quarantined"; // 已隔离（加载失败）

export interface PluginDescriptor {
  name: string;
  version: string;
  path: string;
  status: PluginStatus;
  provides?: string[];      // 提供的服务
  dependencies?: string[];  // 依赖的插件
  error?: string;           // 错误信息（若有）
}

export interface IPluginManager {
  /**
   * 发现可用插件
   * @param searchPaths 搜索路径列表
   */
  discover(searchPaths: string[]): Promise<PluginDescriptor[]>;

  /**
   * 动态加载插件
   * @param name 插件名称
   * @throws 若依赖未满足或加载失败
   */
  load(name: string): Promise<void>;

  /**
   * 卸载插件
   * @param name 插件名称
   * @throws 若有其他插件依赖此插件
   */
  unload(name: string): Promise<void>;

  /**
   * 重新加载插件（卸载后重新加载）
   */
  reload(name: string): Promise<void>;

  /**
   * 取得插件状态
   */
  getStatus(name: string): PluginStatus;

  /**
   * 列出所有插件
   */
  list(): PluginDescriptor[];

  /**
   * 监听插件状态变化
   */
  onStatusChange(
    callback: (name: string, status: PluginStatus) => void
  ): () => void;
}
```

### 3.2 加载顺序与拓扑排序

当插件声明了依赖关系，PluginManager 需要进行拓扑排序：

```
依赖图：
  web-ui-plugin
       │
       ├──► docker-plugin
       │
       └──► nginx-plugin

加载顺序：
  1. docker-plugin
  2. nginx-plugin
  3. web-ui-plugin
```

**实现逻辑：**

```typescript
function topologicalSort(plugins: PluginDescriptor[]): string[] {
  const graph = new Map<string, string[]>();
  const inDegree = new Map<string, number>();

  // 建立依赖图
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

  // 检测循环依赖
  if (result.length !== plugins.length) {
    throw new Error("Circular dependency detected");
  }

  return result;
}
```

---

## 4. 多通道 UI 协作

### 4.1 场景说明

同一个 Agent 可能同时有多个 UI 通道：

```
User A ──► CLI UI ──┐
                    ├──► Agent Core ──► Tools
User B ──► WEB UI ──┘
```

### 4.2 事件广播机制

所有 UI 都通过 `UIRegistry` 接收事件：

```typescript
// transport/bridge.ts
function broadcast(event: AgentEvent): void {
  for (const ui of uiRegistry.list()) {
    ui.onEvent(event);  // CLI 和 WEB 都会收到
  }
}
```

### 4.3 输入来源识别

通过 `InputEvent.source` 区分输入来源：

```typescript
interface InputEvent {
  source: string;      // "cli" | "web" | "api" | ...
  inputType: string;
  data: unknown;
  replyTo?: string;    // 用于定向回复
}
```

### 4.4 定向回复（未来扩展）

若需要将回复只发送给特定 UI：

```typescript
interface AgentEvent {
  type: string;
  timestamp: number;
  payload?: unknown;
  targetUI?: string;   // 若指定，只有该 UI 会收到
}
```

---

## 5. 权限控制

### 5.1 特权插件

并非所有插件都应该能管理其他插件。只有**特权插件**才能存取 `pluginManager`：

```json
// agent.json
{
  "plugins": [
    {
      "name": "@openstarry-plugin/web-ui",
      "privileged": true  // 授予管理权限
    },
    { "name": "@openstarry-plugin/docker" }
  ]
}
```

### 5.2 Core 实现

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

## 6. 实现路线图

| Phase | 内容 | 优先级 |
|-------|------|--------|
| **Phase 1** | ServiceRegistry（registerService / getService） | 高 |
| **Phase 2** | PluginManifest 依赖声明 + 拓扑排序 | 高 |
| **Phase 3** | IPluginManager 基础实现（load/unload） | 中 |
| **Phase 4** | 插件发现机制（扫描项目文件夹） | 中 |
| **Phase 5** | 权限控制与特权插件 | 中 |
| **Phase 6** | 多通道定向回复 | 低 |

---

## 7. 与现有文档的关系

| 文档 | 补充说明 |
|------|----------|
| `19_Agent_Coordination_Layer.md` | 本文扩展其「插件总机」概念，新增服务注册 |
| `13_Composite_Plugins_and_Dependencies.md` | 本文提供具体 API 实现 |
| `20_Dependency_Injection_and_Control_Loop.md` | 本文新增动态注入机制 |
| `21_Plugin_Interface_Deep_Dive.md` | 本文扩展 IPluginContext |

---

## 8. 设计原则

1. **Core 保持纯净** — 服务注册机制在 Core 层，不依赖具体插件实现
2. **渐进式采用** — 现有插件无需修改，新机制为可选扩展
3. **显式依赖** — 通过 manifest 声明，避免隐式耦合
4. **安全第一** — 插件管理需要显式授权
