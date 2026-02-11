# 14. 状态管理与持久化机制 (State Management & Persistence)

本文档定义了 OpenStarry 系统如何实现「崩溃恢复 (Crash Recovery)」与「状态一致性」。

设计遵循 **「核心定义状态，插件负责存储」** 的原则，确保 Agent Core 依然保持无状态 (Stateless) 与轻量化，而将复杂的 IO 操作委托给专门的插件。

## 1. 核心概念：快照 (The Snapshot)

为了保持核心简洁，Agent Core 在运行时维护一个纯 JSON 的状态对象。我们称之为 **`AgentSnapshot`**。

### 1.1 `AgentSnapshot` 数据结构

这是 Core 与 Storage Plugin 之间的唯一契约。Core 只负责生成这个对象，不关心它如何被存储。

```json
{
  "agent_id": "worker_007",
  "tick_index": 42,            // 逻辑时钟，每执行一次循环 +1
  "timestamp": 1706582400000,  // 快照生成时间
  "status": "WAITING_FOR_EVENT", // 当前状态机状态
  "memory": {
    "short_term_history": [    // 当前 Context Window 内的对话
      { "role": "user", "content": "..." },
      { "role": "assistant", "content": "..." }
    ],
    "working_variables": {     // 代理人当前持有的变量
      "current_file_path": "/tmp/report.txt",
      "retry_count": 0
    }
  },
  "event_queue_backup": [      // 尚未处理的事件 (用于恢复现场)
    { "source": "mcp", "type": "task", "payload": "..." }
  ]
}
```

---

## 2. 架构设计：Storage Plugin

为了符合「一切皆插件」的原则，持久化层被抽象为一种新的插件类型。

### 2.1 新增插件类型：`storage`

*   **职责：** 实现 `AgentSnapshot` 的 CRUD 操作。
*   **接口定义 (Interface):**
    所有 `storage` 插件必须实现以下方法：
    *   `save(snapshot: AgentSnapshot): Promise<void>`
    *   `load(agent_id: string): Promise<AgentSnapshot | null>`
    *   `delete(agent_id: string): Promise<void>`

### 2.2 实现示例

*   **`FileStoragePlugin` (开发环境):** 将快照保存为 `.json` 文件 (如 `./data/snapshots/worker_007.json`)。
*   **`RedisStoragePlugin` (生产环境):** 将快照序列化后存入 Redis Key，支持极高的读写速度。
*   **`SQLiteStoragePlugin` (边缘计算):** 存入本地 SQLite 数据库文件，提供 ACID 事务保障。

---

## 3. 工作流程 (The Workflow)

### 3.1 启动与恢复 (Hydration)

当 `Orchestrator Daemon` 启动一个 Agent Core 进程时：

1.  **Core 启动:** 实例化，加载 `PluginLoader`。
2.  **加载存储插件:** `PluginLoader` 根据配置加载指定的 `storage` 插件 (例如 `RedisStorage`)。
3.  **请求状态:** Core 调用 `await storage.load(my_agent_id)`。
4.  **状态注入 (Hydration):**
    *   **若有存档:** Core 将 `AgentSnapshot` 中的数据填充回自己的内存变量（恢复 `tick_index`, `history`, `event_queue`）。**代理人「复活」了，它记得崩溃前的一切。**
    *   **若无存档:** Core 初始化为一个全新的空白状态。

### 3.2 运行时持久化 (Runtime Persistence)

为了确保「严格的」数据安全性，采用 **"Save-After-Write" (处理后即存档)** 策略。

在 **`01_Execution_Loop`** 的每个循环末尾：

1.  **事件处理完成:** LLM 思考完毕，工具执行完毕。
2.  **状态更新:** Core 更新内存中的 `history` 和 `variables`。
3.  **触发存档:** Core 调用 `await storage.save(current_snapshot)`。
4.  **响应外部:** **只有在 `save` 成功返回后**，Core 才会将响应发送给 UI 或执行下一个 Action。
    *   *这确保了如果系统在响应前崩溃，重启后不会丢失刚才的执行结果，也不会重复执行已保存但未响应的步骤（需配合幂等性设计）。*

---

## 4. 区分「状态」与「记忆」

在架构中必须严格区分这两个概念，以免混淆：

| 特性 | **状态 (State / Persistence)** | **记忆 (Memory / RAG)** |
| :--- | :--- | :--- |
| **目的** | **生存** (Crash Recovery) | **知识** (Knowledge Retrieval) |
| **存储内容** | 完整的上下文堆栈 (Stack), 变量 | 处理过的文档, 经验总结 |
| **存储位置** | `storage` 插件 (Redis/SQL) | `RAG Engine` (Vector DB) |
| **生命周期** | 随任务结束可删除 (Ephemeral) | 永久保存 (Long-term) |
| **负责人** | **Agent Core 直接管理** | **Supporting Engine 外部服务** |

---

## 5. 总结

通过引入 `storage` 插件接口：

1.  **Core 保持了纯净:** 它不需要依赖任何特定的数据库驱动，只需要调用 `save()` / `load()`。
2.  **极致的灵活性:** 用户可以通过更换插件，将持久化层从本地文件无缝切换到云端数据库，而无需修改核心代码。
3.  **高可靠性:** 配合 Daemon 的重启机制，系统实现了「永不失忆」的健壮性。
