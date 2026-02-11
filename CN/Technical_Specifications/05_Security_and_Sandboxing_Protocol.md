# 05. 安全与沙盒协议技术规范 (Security & Sandboxing Protocol)

本文件定义了 OpenStarry 的多层防御体系。作为一个拥有执行能力的 Agent 系统，安全性是不可妥协的基石。

## 1. 文件系统沙盒 (FileSystem Sandboxing)

所有涉及 IO 的操作（读、写、列表）必须受到 `PathGuard` 的严格监控。

### 1.1 监狱模式 (Chroot-like Jail)
Agent 预设只能访问其工作目录 (`workspace_root`)。任何试图访问父目录的操作都会被拦截。

*   **允许:** `./data/file.txt`, `src/index.ts`
*   **禁止:** `../secret.key`, `/etc/passwd`, `C:\Windows\System32`

### 1.2 路径规范化
在检查权限前，所有路径必须经过解析与规范化，以防止通过符号链接 (Symlink) 或相对路径欺骗 (`/app/workspace/../../etc/passwd`) 绕过检查。

---

## 2. 命令执行白名单 (Command Execution Whitelist)

对于 `shell:exec` 这类高风险工具，必须实施白名单策略。

### 安全策略配置 (`agent.json`)

```json
"security": {
  "allow_shell": true,
  "blocked_commands": ["rm -rf", "mkfs", ":(){:|:&};:", "format"],
  "allowed_commands": ["ls", "cat", "grep", "npm test", "git status"],
  "confirm_dangerous_actions": true
}
```

当 `confirm_dangerous_actions` 为 `true` 时，任何不在 `allowed_commands` 中的指令，核心将暂停执行并发出 `EVENT:REQUEST_CONFIRMATION`，等待人类批准后才放行。

---

## 3. 资源配额与熔断 (Resource Quotas & Circuit Breakers)

为了防止 DoS 攻击或无限循环，核心强制执行以下限额：

| 资源维度 | 限制阈值 (预设) | 触发后果 |
| :--- | :--- | :--- |
| **Max Loop Ticks** | 50 次/任务 | 强制暂停任务 (Safety Halt) |
| **Token Budget** | 100k / 会话 | 拒绝 LLM 请求 |
| **Tool Timeout** | 30 秒 | 强制终止工具进程 (SIGKILL) |
| **Output Size** | 1 MB | 截断输出 |

---

## 4. 插件权限模型 (Plugin Permission Model)

插件在加载时必须声明其所需权限 (Manifest Declaration)。

```json
// plugin.json
{
  "permissions": [
    "fs:read",
    "network:http",
    "core:state:write"
  ]
}
```

核心在初始化插件时，会对比 `agent.json` 中的授权列表。若插件请求了未授权的敏感权限，加载将被拒绝。
