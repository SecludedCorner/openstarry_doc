# 架構概覽

## 五蘊（Five Aggregates）

OpenStarry 的所有插件功能映射到五個核心介面：

| 蘊 | 介面 | 角色 |
|----|------|------|
| 色 | IUI | 使用者介面（輸出） |
| 受 | IListener | 事件監聽（輸入） |
| 想 | IProvider | LLM 服務提供者 |
| 行 | ITool | 可執行工具 |
| 識 | IGuide | 系統提示詞/引導 |

## 微核心設計

- **Core 不依賴任何插件**，所有功能透過插件擴展
- 插件透過 `ctx.pushInput()` 與核心通訊，不直接調用 API
- 插件使用工廠模式：`createXxxPlugin()` → `IPlugin { manifest, factory(ctx) }`

### 核心套件

| 套件 | 說明 |
|------|------|
| `packages/sdk` | 類型契約（介面、事件、錯誤） |
| `packages/core` | Agent 核心（EventBus、ExecutionLoop、安全層） |
| `packages/shared` | 共用工具（logger、UUID、驗證） |
| `packages/plugin-signer` | 插件簽名驗證 |
| `apps/runner` | CLI 啟動器（所有指令入口） |

## 事件驅動流程

```
User Input → IListener → pushInput() → EventBus → ExecutionLoop → IProvider (LLM)
                                                                      ↓
UI Output ← IUI ← AgentEvent ← EventBus ← Tool Results ← ITool ← LLM Response
```

1. **輸入**：IListener 接收使用者輸入，透過 `pushInput()` 注入事件匯流排
2. **處理**：ExecutionLoop 從 EventBus 取得事件，交給 IProvider（LLM）處理
3. **工具呼叫**：LLM 回應中的工具請求由 ITool 執行，結果回流至 EventBus
4. **輸出**：IUI 監聽 AgentEvent，將回應呈現給使用者

## 延伸閱讀

- `packages/sdk/README.md` — 所有介面定義
- `packages/core/README.md` — 核心架構細節
