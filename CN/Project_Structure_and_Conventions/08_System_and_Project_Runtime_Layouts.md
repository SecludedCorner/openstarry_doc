# 08. 系统级与项目级运行时布局 (System & Project Runtime Layouts)

本文档落实了 **"Directory as Permission"** 的核心哲学。OpenStarry 区分了「全局系统空间」与「局部项目空间」，以实现权限隔离与环境可携性。

---

## 1. 系统全局空间 (System-wide Space)

这是 Orchestrator Daemon 的主目录，通常位于用户的主文件夹下。

*   **路径 (Windows):** `%USERPROFILE%\.openstarry`
*   **路径 (Linux/macOS):** `~/.openstarry/`

### 目录结构

```text
~/.openstarry/
├── daemon.pid              # Daemon 进程锁定文件
├── config.json             # Daemon 全局配置文件 (API Port, 存储路径等)
├── agents/                 # [系统级 Agent] 受 Daemon 托管的所有实体
│   ├── agent-001/          # 符合 04 号文档定义的标准目录
│   └── agent-002/ 
│   
├── plugins/                # [系统级插件] 全局可用的工具与 Provider
│   
├── storage/                # 全局持久化数据
│   └── main.db             # SQLite/LevelDB (存储 Agent 状态索引)
│   
└── logs/                   # Daemon 系统日志
```

---

## 2. 项目局部空间 (Project-specific Space)

当开发者在特定项目目录下启动 OpenStarry 时，系统会自动识别项目空间。

*   **路径:** `[Your-Project-Path]/.openstarry/`

### 目录结构

```text
.openstarry/
├── agent.json              # 项目专属 Agent 配置
├── plugins/                # [项目插件] 仅对本项目可见的能力
│   └── custom-tools/       # 项目私有工具
│   
├── data/                   # 项目产出的本地数据
│   
└── logs/                   # 项目运行日志
```

---

## 3. 加载优先级与覆盖机制 (Overlay Mechanism)

OpenStarry 采用类似于 Linux UnionFS 的加载逻辑。当 Agent 寻找一个插件或配置时，搜寻顺序如下：

1.  **项目空间 (`./.openstarry/`)**: 最高优先级，用于定制化与调试。
2.  **系统空间 (`~/.openstarry/`)**: 标准能力提供者。
3.  **内置空间 (Built-in)**: 随二进制文件发布的原始默认配置。

### 权限安全原则

*   **隔离性**: 项目 Agent 默认无法访问系统空间的其他 Agent 数据，除非明确获得权限。
*   **只读性**: 系统级插件对项目 Agent 来说通常是只读的，防止项目级误操作破坏系统稳定。

---

## 4. 哲学映射

*   **系统空间 = 「大环境」**: 数字物种生存的共同生态系统。
*   **项目空间 = 「栖息地」**: 数字物种具体的执行领地。
*   **目录即权限**: 插件被放置在 `~/.openstarry` 还是 `./.openstarry`，直接决定了它的信任等级与可见范围。
