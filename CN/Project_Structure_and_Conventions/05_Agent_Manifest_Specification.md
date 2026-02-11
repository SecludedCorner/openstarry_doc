# 04. 代理人清单规范 (Agent Manifest Specification)

本文档定义了 `agent.json` 的数据结构。这是每个 Agent 实例的核心配置文件，位于 `[Agent_Root]/configs/agent.json`。

## 核心哲学：身份与能力的静态定义

`agent.json` 在 Agent 启动时被 Core 读取。它是**不可变**的（运行时不可修改），确保了 Agent 行为的可预测性与可审计性。

---

## 结构定义

```json
{
  "$schema": "./schemas/agent-manifest.schema.json",
  "identity": {
    "id": "log-analyst-01",
    "name": "Log Analysis Bot",
    "role": "data_engineer",
    "description": "负责分析服务器日志并生成报告"
  },
  
  "cognition": {
    "system_prompt": "configs/prompts/system.md", // 指向外部文件或直接内联字符串
    "provider": {
      "id": "gemini-pro-adapter",
      "config": {
        "model": "gemini-1.5-pro",
        "temperature": 0.2
      }
    },
    "context_strategy": {
      "type": "summarizer", // 引用策略插件 ID
      "config": {
        "threshold": 20
      }
    }
  },

  "capabilities": {
    "plugins": [
      // 引用插件完整包名，Core 会按照继承规则（项目优先 -> 系统）去查找
      "@openstarry-plugin/standard-function-fs",
      "@openstarry-plugin/standard-function-stdio",
      "project-log-parser"
    ],
    "permissions": {
      "fs_allow_paths": ["./logs", "./reports"], // 沙盒路径限制
      "network_allow_hosts": ["github.com"]
    }
  },

  "policy": {
    "max_steps": 50, // 防止死循环
    "circuit_breaker": {
      "error_threshold": 5
    }
  }
}
```

---

## 字段详解

### 1. `identity`
*   **id:** 全局唯一的标识符，用于日志追踪和 Daemon 管理。
*   **role:** 用于多 Agent 协作时的寻址（例如：「找一个 `data_engineer` 来帮忙」）。

### 2. `cognition` (想蕴配置)
*   **system_prompt:** 定义 Agent 的核心人格与职责。支持引用外部 Markdown 文件，方便编写长文本。
*   **provider:** 指定使用哪个 LLM 作为大脑。

## 3. 插件配置详解 (Plugin Configuration)

Agent 加载的插件可能包含多种成分，我们可以在 `agent.json` 中对其进行精细化配置。

```json
"plugins": {
  "@openstarry-plugin/standard-function-fs": {
    "tools": { "read": true, "write": false } // 限制行蕴
  },
  "@openstarry-plugin/standard-function-skill": {
    "guides": { "autoload": true } // 配置识蕴
  }
}
```

### 组件类型 (Component Types)

*   **tools (行):** 具体执行的函数。
*   **listeners (受):** 事件监听器。
*   **providers (想):** AI 模型后端。
*   **ui (色):** 界面组件。
*   **guides (识)::** 技能、流程与协议逻辑。

### 4. `policy` (超我配置)
*   **max_steps:** 单次任务的最大执行步数，防止资源耗尽。
*   **circuit_breaker:** 定义何时触发「强制冷却」或「求助」。
