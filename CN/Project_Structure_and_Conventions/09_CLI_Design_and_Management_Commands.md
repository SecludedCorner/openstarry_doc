# 09. CLI 设计与管理指令 (CLI Design & Management Commands)

OpenStarry CLI (`openstarry`) 是用户与 Agent 系统互动的统一入口。它具备上下文感知能力，能根据执行位置自动切换「项目模式」与「系统模式」。

## 1. 核心设计原则

### 上下文感知 (Context Awareness)
CLI 的「设计模式」(`openstarry design`) 具备强大的上下文感知能力，其行为取决于**您在哪里执行它**。
*   **全局设计模式 (Global Design Mode):** 在非项目目录下执行时，提供系统级管理功能（如创建新项目、管理全局模板）。
*   **项目设计模式 (Project Design Mode):** 在包含 `agent.json` 的目录下执行时，自动进入该 Agent 的编辑与配置界面。

### 运行时与设计时分离 (Runtime vs Design)
*   **`openstarry` (Runtime):** 专注于**执行与监控**。无论在哪里，它都连接到守护进程，提供一个实时的系统仪表板 (Dashboard)。
*   **`openstarry design` (Design):** 专注于**定义与配置**。它是您构建五蕴、组装 Agent 的工作台。

### 前台与后台 (Foreground vs Background)
*   **项目模式 (Foreground):** Agent 作为当前 Shell 的子进程运行。关闭终端 = Agent 停止。
*   **守护模式 (Background):** 通过 Daemon 启动，即便关闭终端，Agent 仍在后台运行。

---

## 2. 指令集详解

### A. 核心入口 (Core Entry Points)

```bash
# 启动 OpenStarry 运行时环境 (The Runtime Environment)
$ openstarry
# -> 检查 Orchestrator Daemon 是否运行。
# -> 若未运行，则自动启动 Daemon 与 Master Agent (System Mode)。
# -> 若已运行，则进入互动式控制台 (Console)，显示当前运行的 Agent 状态与系统指标。
# -> [Console Feature] 可在控制台中选择特定 Agent 并按下 'a' 键直接连线 (Attach) 进行对话。

# -> [Context Aware] 根据当前目录自动切换：
#    - 全局模式：显示「创建新项目」、「下载模板」、「管理全局设定」菜单。
#    - 项目模式：显示当前 Agent 的结构，引导用户定义 **五蕴** (色受想行识)：
#      * 设定人设与记忆 (识)
#      * 配置界面与名称 (色)
#      * 添加工具插件 (行)
#      * 选择感官监听器 (受)
#      * 指定 AI 模型 (想)
# -> 启动互动式 TUI (Text User Interface)。
# -> 自动生成或更新 agent.json 与目录结构。
```

### B. 项目生命周期 (Project Lifecycle)

这些指令用于管理当前目录下的「项目级 Agent」。

```bash
# 初始化一个新 Agent 项目
$ openstarry init [template-name]
# -> 创建 agent.json, package.json, .openstarry/ 结构

# [互动模式] 启动并进入对话
$ openstarry attach
# -> 在项目目录下执行时：
#    - 若 Agent 尚未运行：则自动启动 Agent 并立即进入互动界面。
#    - 若 Agent 已在后台运行：则直接连线 (Attach) 到该 Agent Morse。
# -> 这是与 Agent 开启对话的最常用指令。

# [启动模式] 仅启动进程到后台
$ openstarry start
# -> 读取 ./agent.json，将 Agent 启动为背景守护进程。
# -> 写入 ./openstarry/agent.pid，日志记录到文件。
# -> 不会占用当前终端。

# 直接执行工具 (Manual Tool Invocation)
$ openstarry run-tool <tool-name> [args...]
# -> 不经过 LLM，直接调用指定的 ITool。
# -> 常用语：登录 (google-login)、初始化设定、或是测试工具功能。
# -> 范例：openstarry run-tool google-login

# 清理项目运行数据
$ openstarry clean
# -> 删除 ./.openstarry/ 中的临时文件 (logs, state cache)
```

### B. 系统守护管理 (Daemon Management)

这些指令用于与 `~/.openstarry` 中的守护进程交互。

```bash
# 启动系统守护进程 (如果尚未运行)
$ openstarry daemon start

# 查看系统状态 (列出所有托管的 Agents)
$ openstarry ps
# OUTPUT:
# ID      NAME        STATUS    PID    MODE      LOCATION
# sys-01  Scheduler   RUNNING   1024   System    ~/.openstarry/agents/sys-01
# prj-01  MyBot       RUNNING   4521   Project   D:/Code/MyBot (Registered)

# 停止守护进程 (及所有子 Agent)
$ openstarry daemon stop
```

### C. 注册与托管 (Registration & Hosting)

将一个「项目级 Agent」升级为「系统级服务」。

```bash
# 将当前目录的 Agent 注册到 Daemon
$ openstarry register
# -> Daemon 会记录路径 "D:/Code/MyBot" 到 ~/.openstarry/registry.json
# -> Daemon 现在负责该 Agent 的重启与监控

# 连线至全局或特定 Agent (Attach)
$ openstarry attach [agent-id]
# -> 若带有 agent-id：连线至指定的后台 Agent。
# -> 若在项目目录下且无 agent-id：连线至当前项目 Agent。
# -> 按 Ctrl+C 断开连接（Detach），但 Agent 继续在后台运行。

# 刷新插件注册表 (Manual Refresh)
$ openstarry plugin refresh
# -> 强制 Daemon 重新扫描系统插件目录。
# -> 用于侦测手动复制进去的新插件。
```

## 3. 数据流向总结

| 模式 | 指令范例 | 配置读取位置 | 运行状态 (PID/State) | 日志位置 | 生命周期 |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **项目 (前台)** | `openstarry start` | `./agent.json` | `./.openstarry/` | `./.openstarry/logs/` | 随 Shell 结束 |
| **项目 (托管)** | `openstarry start -d` | `./agent.json` | `~/.openstarry/state/` | `~/.openstarry/logs/` | 由 Daemon 管理 |
| **系统 (内置)** | `openstarry system start` | `~/.openstarry/agents/` | `~/.openstarry/state/` | `~/.openstarry/logs/` | 常驻服务 |

此设计确保了开发时的灵活性（项目模式）与部署时的稳定性（托管模式）完美共存。
