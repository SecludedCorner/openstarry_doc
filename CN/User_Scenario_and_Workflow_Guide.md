# OpenStarry 用户场景与操作工作流指南 (User Scenario and Workflow Guide)

本文件将带您深入体验 OpenStarry 的核心操作场景，从系统启动、设计 Agent 到实际对话互动，以及如何将 Agent 纳入系统的自动化管理流程。

---

## 1. 系统入口：启动与监控 (`openstarry`)

当您在终端输入 `openstarry` 时，您正在启动这个多智能体操作系统的**「总控制台 (Runtime Dashboard)」**。

### 操作场景
```bash
$ openstarry
```

### 系统行为
1.  **自动连接/启动守护进程 (Daemon):** 系统会检查后台是否已有 `Orchestrator Daemon` 在运行。若无，则自动启动它。
2.  **进入上帝视角 (Dashboard):** 显示一个实时更新的 TUI (Text User Interface) 仪表板。

### 视觉体验 (Dashboard)
```text
  ___                   _____ _
 / _ \ _ __   ___ _ __ /  ___| |_ __ _ _ __ _ __ _   _
| | | | '_ \ / _ \ '_ \\ `--.| __/ _` | '__| '__| | | |
| |_| | |_) |  __/ | | |`--. \ || (_| | |  | |  | |_| |
 \___/| .__/ \___|_| |_/\____/\__\__,_|_|  |_|   \__, |
      | |                                         __/' |
      |_|                                        |___/

[SYSTEM DASHBOARD] - Connected to Daemon (PID: 14023)
---------------------------------------------------------------------
● System Status:  HEALTHY   |   ● Uptime: 2d 4h 12m
● CPU Usage:      12%       |   ● Memory: 402MB / 8GB
---------------------------------------------------------------------

[RUNNING AGENTS]
ID       NAME            TYPE          STATUS    THOUGHTS/SEC   LAST ACTIVITY
sys-01   Master-Mind     Orchestrator  RUNNING   0.5            Just now
dev-01   Code-Helper     Worker        IDLE      0.0            5m ago
web-01   Search-Bot      Worker        BUSY      1.2            Processing...

[CONTROLS]
(q)uit view  (r)estart daemon  (k)ill agent  (d)esign mode
(a)ttach to agent (select via arrows)
```

---

## 2. 设计模式：创建与编辑 (`openstarry design`)

当您需要创建新生命或修改现有 Agent 时，请使用设计模式。该指令具备**上下文感知**能力。

### 场景 A：全局设计模式 (Global Design Mode)
*   **触发条件：** 在非项目目录下执行 `openstarry design`。
*   **功能：** 这是「造物主视角」。
    *   **Create New Agent Project:** 从标准模板 (Coding, Writing, Analysis) 孵化一个全新的 Agent 项目。
    *   **Manage Templates:** 下载或管理您的 Agent 基因库。
    *   **System Configuration:** 设置全局 API Key 或调整 Daemon 资源限制。

### 场景 B：项目设计模式 (Project Design Mode)
*   **触发条件：** 在 Agent 项目目录下执行 `openstarry design`。
*   **功能：** 这是「手术台视角」。系统会引导您定义 Agent 的 **五蕴 (Five Aggregates)**：
    1.  **识 (Guide):** 注入灵魂。修改 `system_prompt`，设置**人设**、**核心目标**与**记忆策略**。
    2.  **色 (UI/Body):** 塑造肉体。设置代理的**名称**、**外观**或**交互界面 (UI)**。
    3.  **受 (Listeners):** 配置感官。设置接收消息的通道 (如 `stdio`, `webhook`)。
    4.  **行 (Tools):** 装配肢体。安装工具插件 (如 `fs`, `gemini-search`)。
    5.  **想 (Providers):** 选择大脑。连接特定的 LLM 模型 (如 Gemini 1.5 Pro)。

---

## 3. 对话互动：与 Agent 连线 (`openstarry attach`)

要与 Agent 开始对话，我们使用 `attach` 指令。这是互动的**单一入口**。

### 场景 A：在 Dashboard 中连线
1.  在 `openstarry` 仪表板中，使用方向键选中目标 Agent (例如 `sys-01`)。
2.  按下键盘上的 **`a`** 键。
3.  画面将瞬间切换到该 Agent 的对话窗口。
4.  按下 `Ctrl+D` (Detach) 可返回仪表板，Agent 继续在后台运行。

### 场景 B：在项目目录下连线 (最常用)
在您的开发目录下，直接输入：
```bash
$ openstarry attach
```
*   **若 Agent 未运行：** 系统会自动启动它，并直接进入对话界面。
*   **若 Agent 已在后台：** 系统会直接连线进去。

### 场景 C：连线到任意后台 Agent
```bash
$ openstarry attach web-01
```

### 互动技巧：Slash Commands
在对话中，您可以使用 `/` 来直接调用工具，而不必请求 LLM。
*   `/login` -> 调用 `google-login` 工具 (若有注册别名)。
*   `/tool google-login` -> 直接执行工具。
*   `/clear` -> 清除当前 Context 记忆。

---

## 4. 部署与自动启动流程 (Deployment Workflow)

如何让您的 Agent 在系统启动时自动上线？

### 步骤 1：同步标准插件 (Sync)
确保您的系统拥有最新的标准能力库。
```bash
# 假设您已将 openstarry_plugin 仓库 clone 到本地
$ openstarry plugin sync ./openstarry_plugin
```

### 步骤 2：注册 (Register)
首先，告诉守护进程 (Daemon) 您的 Agent 在哪里。在项目目录下执行：
```bash
$ openstarry register
```
这会将项目路径写入 `~/.openstarry/registry.json`，使其成为「托管 Agent」。

### 步骤 3：加入自动启动列表 (Autostart)
目前有两种方式：

**方式 A (CLI):**
在 Dashboard 的「系统配置」菜单中，或使用未来规划的指令：
```bash
$ openstarry system config --autostart add <project-id>
```

**方式 B (手动配置):**
编辑 `~/.openstarry/daemon.json`：
```json
{
  "autostart": [
    "master-agent",
    "my-coding-bot",
    "daily-news-reporter"
  ]
}
```

### 步骤 4：验证 (Verification)
重启 Daemon (`openstarry daemon restart`) 或重新输入 `openstarry` 进入仪表板。您应该会看到您的 Agent 已经在 **[RUNNING AGENTS]** 列表中自动上线了。

---

## 5. 高级操作：工作流与多智能体协作 (Advanced Orchestration)

OpenStarry 的强大之处在于能执行「一连串」的 Agent。

### 步骤 1：定义工作流 (Define Workflow)
在项目或系统目录下创建 `workflow.yaml`：
```yaml
name: Daily Research
steps:
  - id: research
    agent: research-agent
    instruction: "搜集今日 AI 新闻"
  - id: summary
    agent: writer-agent
    input_from: research
    instruction: "总结为 3 点并翻译成中文"
```

### 步骤 2：执行工作流 (Execute)
在 CLI 中触发：
```bash
$ openstarry run ./workflow.yaml
```

### 步骤 3：观察协作 (Observe)
回到 Dashboard (`openstarry`)，您会看到：
1.  **动态孵化：** `research-agent` 突然出现在列表并显示 `BUSY`。
2.  **任务交接：** 几秒后，`research-agent` 消失（任务完成被销毁），`writer-agent` 出现并开始写作。
3.  **结果交付：** 最终结果会输出到您的终端或指定的文件中。

---

## 6. 故障排除与日志解读 (Troubleshooting & Logs)

当 Agent 行为不如预期时，如何诊断？

### 查看实时思考 (Live Thoughts)
使用 `attach` 进入 Agent 后，您会看到它的「内心独白」：
```text
[THOUGHT] User asked for file search.
[PLAN] I need to use 'fs' tool to list directory.
[CALL] fs.list_dir(path=".")
[RESULT] Success. Found 3 files.
[RESPONSE] I found 3 files...
```
如果卡住了，通常是卡在 `[CALL]` 失败或 `[THOUGHT]` 循环。

### 查看系统日志 (System Logs)
若 Agent 根本启动不了，请查阅 Daemon 日志：
```bash
$ openstarry system logs
```
常见错误：
*   **Template Not Found:** 注册表路径错误，请重新 `register`。
*   **Permission Denied:** 插件权限不足，请检查 `manifest.json` 或使用 `openstarry plugin grant`。
