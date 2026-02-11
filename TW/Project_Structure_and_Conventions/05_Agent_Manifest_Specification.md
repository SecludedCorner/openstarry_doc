# 04. 代理人清單規範 (Agent Manifest Specification)

本文件定義了 `agent.json` 的數據結構。這是每個 Agent 實例的核心配置文件，位於 `[Agent_Root]/configs/agent.json`。

## 核心哲學：身份與能力的靜態定義

`agent.json` 在 Agent 啟動時被 Core 讀取。它是**不可變**的（Runtime 時不可修改），確保了 Agent 行為的可預測性與可審計性。

---

## 結構定義

```json
{
  "$schema": "./schemas/agent-manifest.schema.json",
  "identity": {
    "id": "log-analyst-01",
    "name": "Log Analysis Bot",
    "role": "data_engineer",
    "description": "負責分析服務器日誌並生成報告"
  },
  
  "cognition": {
    "system_prompt": "configs/prompts/system.md", // 指向外部文件或直接內聯字符串
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
      // 引用插件完整包名，Core 會按照繼承規則（專案優先 -> 系統）去查找
      "@openstarry-plugin/standard-function-fs",
      "@openstarry-plugin/standard-function-stdio",
      "project-log-parser"
    ],
    "permissions": {
      "fs_allow_paths": ["./logs", "./reports"], // 沙盒路徑限制
      "network_allow_hosts": ["github.com"]
    }
  },

  "policy": {
    "max_steps": 50, // 防止死循環
    "circuit_breaker": {
      "error_threshold": 5
    }
  }
}
```

---

## 欄位詳解

### 1. `identity`
*   **id:** 全局唯一的標識符，用於日誌追蹤和 Daemon 管理。
*   **role:** 用於多 Agent 協作時的尋址（例如：「找一個 `data_engineer` 來幫忙」）。

### 2. `cognition` (想蘊配置)
*   **system_prompt:** 定義 Agent 的核心人格與職責。支持引用外部 Markdown 文件，方便編寫長文本。
*   **provider:** 指定使用哪個 LLM 作為大腦。

## 3. 插件配置詳解 (Plugin Configuration)

Agent 加載的插件可能包含多種成分，我們可以在 `agent.json` 中對其進行精細化配置。

```json
"plugins": {
  "@openstarry-plugin/standard-function-fs": {
    "tools": { "read": true, "write": false } // 限制行蘊
  },
  "@openstarry-plugin/standard-function-skill": {
    "guides": { "autoload": true } // 配置識蘊
  }
}
```

### 組件類型 (Component Types)

*   **tools (行):** 具體執行的函數。
*   **listeners (受):** 事件監聽器。
*   **providers (想):** AI 模型後端。
*   **ui (色):** 介面組件。
*   **guides (識)::** 技能、流程與協議邏輯。

### 4. `policy` (超我配置)
*   **max_steps:** 單次任務的最大執行步數，防止資源耗盡。
*   **circuit_breaker:** 定義何時觸發「強制冷卻」或「求助」。
