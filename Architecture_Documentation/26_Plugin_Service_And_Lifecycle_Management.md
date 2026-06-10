# 26. 插件服務與生命週期管理 (Plugin Service & Lifecycle Management)

本文件定義了插件之間的**服務共享機制**與**動態生命週期管理**，為複雜的多插件協作場景提供架構基礎。

---

## 1. 問題背景

### 1.1 現有機制的局限

當前架構中，五蘊插件透過 Registry 間接協作：

```
Plugin A (Tool) ──註冊──► ToolRegistry ◄──查詢── Core
Plugin B (Guide) ──註冊──► GuideRegistry ◄──查詢── Core
```

這對於標準五蘊插件已足夠，但無法處理以下場景：

| 場景 | 問題 |
|------|------|
| **服務共享** | Docker Plugin 提供 API 給 WEB UI 使用 |
| **動態管理** | WEB UI 在運行時啟用/禁用其他插件 |
| **依賴協調** | Plugin A 需要 Plugin B 先完成初始化 |

### 1.2 目標場景

```
Agent (VM 環境)
│
├─ 複合 UI 插件
│   ├─ CLI UI ──────── 終端界面
│   └─ WEB UI ──────── 瀏覽器管理界面
│       ├─ 插件管理面板 ── 啟用/禁用插件
│       ├─ 服務狀態監控 ── 查詢 Docker/Nginx 狀態
│       └─ 可視化配置 ──── 編輯插件設定
│
├─ Docker Plugin ────── 容器管理服務
├─ Nginx Plugin ─────── 反向代理服務
└─ [專案插件...] ────── 動態發現與加載
```

---

## 2. 服務註冊與發現機制

### 2.1 擴展 IPluginContext

```typescript
export interface IPluginContext {
  // 現有欄位
  bus: EventBus;
  workingDirectory: string;
  agentId: string;
  config: Record<string, unknown>;
  pushInput: (event: InputEvent) => void;

  // === 新增：服務機制 ===

  /**
   * 註冊服務供其他插件使用
   * @param id 服務唯一識別符
   * @param service 服務實例
   */
  registerService<T>(id: string, service: T): void;

  /**
   * 取得已註冊的服務
   * @param id 服務識別符
   * @returns 服務實例，若不存在則返回 undefined
   */
  getService<T>(id: string): T | undefined;

  /**
   * 插件管理器（僅限特權插件）
   * 若插件未被授予管理權限，此欄位為 undefined
   */
  pluginManager?: IPluginManager;
}
```

### 2.2 服務提供者範例

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
      provides: ["docker-service"],  // 聲明提供的服務
    },

    async factory(ctx: IPluginContext): Promise<PluginHooks> {
      const dockerService: DockerService = {
        async listContainers() { /* ... */ },
        async startContainer(id) { /* ... */ },
        async stopContainer(id) { /* ... */ },
      };

      // 註冊服務
      ctx.registerService("docker", dockerService);

      return {
        tools: [createDockerTool(dockerService)],
      };
    },
  };
}
```

### 2.3 服務消費者範例

```typescript
// web-ui-plugin/src/index.ts

export function createWebUIPlugin(): IPlugin {
  return {
    manifest: {
      name: "web-ui-plugin",
      version: "0.1.0",
      dependencies: ["docker-plugin"],  // 聲明依賴
    },

    async factory(ctx: IPluginContext): Promise<PluginHooks> {
      // 取得 Docker 服務
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

## 3. 插件生命週期管理

### 3.1 IPluginManager 介面

```typescript
export type PluginStatus =
  | "discovered"   // 已發現但未加載
  | "loading"      // 正在加載
  | "running"      // 運行中
  | "stopped"      // 已停止
  | "error"        // 錯誤狀態
  | "quarantined"; // 已隔離（加載失敗）

export interface PluginDescriptor {
  name: string;
  version: string;
  path: string;
  status: PluginStatus;
  provides?: string[];      // 提供的服務
  dependencies?: string[];  // 依賴的插件
  error?: string;           // 錯誤訊息（若有）
}

export interface IPluginManager {
  /**
   * 發現可用插件
   * @param searchPaths 搜索路徑列表
   */
  discover(searchPaths: string[]): Promise<PluginDescriptor[]>;

  /**
   * 動態加載插件
   * @param name 插件名稱
   * @throws 若依賴未滿足或加載失敗
   */
  load(name: string): Promise<void>;

  /**
   * 卸載插件
   * @param name 插件名稱
   * @throws 若有其他插件依賴此插件
   */
  unload(name: string): Promise<void>;

  /**
   * 重新加載插件（卸載後重新加載）
   */
  reload(name: string): Promise<void>;

  /**
   * 取得插件狀態
   */
  getStatus(name: string): PluginStatus;

  /**
   * 列出所有插件
   */
  list(): PluginDescriptor[];

  /**
   * 監聽插件狀態變化
   */
  onStatusChange(
    callback: (name: string, status: PluginStatus) => void
  ): () => void;
}
```

### 3.2 加載順序與拓撲排序

當插件聲明了依賴關係，PluginManager 需要進行拓撲排序：

```
依賴圖：
  web-ui-plugin
       │
       ├──► docker-plugin
       │
       └──► nginx-plugin

加載順序：
  1. docker-plugin
  2. nginx-plugin
  3. web-ui-plugin
```

**實作邏輯：**

```typescript
function topologicalSort(plugins: PluginDescriptor[]): string[] {
  const graph = new Map<string, string[]>();
  const inDegree = new Map<string, number>();

  // 建立依賴圖
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

  // 檢測循環依賴
  if (result.length !== plugins.length) {
    throw new Error("Circular dependency detected");
  }

  return result;
}
```

---

## 4. 多通道 UI 協作

### 4.1 場景說明

同一個 Agent 可能同時有多個 UI 通道：

```
User A ──► CLI UI ──┐
                    ├──► Agent Core ──► Tools
User B ──► WEB UI ──┘
```

### 4.2 事件廣播機制

所有 UI 都透過 `UIRegistry` 接收事件：

```typescript
// transport/bridge.ts
function broadcast(event: AgentEvent): void {
  for (const ui of uiRegistry.list()) {
    ui.onEvent(event);  // CLI 和 WEB 都會收到
  }
}
```

### 4.3 輸入來源識別

透過 `InputEvent.source` 區分輸入來源：

```typescript
interface InputEvent {
  source: string;      // "cli" | "web" | "api" | ...
  inputType: string;
  data: unknown;
  replyTo?: string;    // 用於定向回覆
}
```

### 4.4 定向回覆（未來擴展）

若需要將回覆只發送給特定 UI：

```typescript
interface AgentEvent {
  type: string;
  timestamp: number;
  payload?: unknown;
  targetUI?: string;   // 若指定，只有該 UI 會收到
}
```

---

## 5. 權限控制

### 5.1 特權插件

並非所有插件都應該能管理其他插件。只有**特權插件**才能存取 `pluginManager`：

```json
// agent.json
{
  "plugins": [
    {
      "name": "@openstarry-plugin/web-ui",
      "privileged": true  // 授予管理權限
    },
    { "name": "@openstarry-plugin/docker" }
  ]
}
```

### 5.2 Core 實作

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

## 6. 實作路線圖

| Phase | 內容 | 優先級 |
|-------|------|--------|
| **Phase 1** | ServiceRegistry（registerService / getService） | 高 |
| **Phase 2** | PluginManifest 依賴聲明 + 拓撲排序 | 高 |
| **Phase 3** | IPluginManager 基礎實作（load/unload） | 中 |
| **Phase 4** | 插件發現機制（掃描專案資料夾） | 中 |
| **Phase 5** | 權限控制與特權插件 | 中 |
| **Phase 6** | 多通道定向回覆 | 低 |

---

## 7. 與現有文檔的關係

| 文檔 | 補充說明 |
|------|----------|
| `19_Agent_Coordination_Layer.md` | 本文擴展其「插件總機」概念，新增服務註冊 |
| `13_Composite_Plugins_and_Dependencies.md` | 本文提供具體 API 實作 |
| `20_Dependency_Injection_and_Control_Loop.md` | 本文新增動態注入機制 |
| `21_Plugin_Interface_Deep_Dive.md` | 本文擴展 IPluginContext |

---

## 8. 設計原則

1. **Core 保持純淨** — 服務註冊機制在 Core 層，不依賴具體插件實作
2. **漸進式採用** — 現有插件無需修改，新機制為可選擴展
3. **顯式依賴** — 透過 manifest 聲明，避免隱式耦合
4. **安全第一** — 插件管理需要顯式授權
