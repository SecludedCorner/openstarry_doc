# 00. 插件哲學：五蘊聚合體 (Plugin as Five Aggregates)

> **[重要說明]** 本文件描述的是 OpenStarry 插件系統背後的**架構理論映射**。

## 核心哲學

**一個插件不是某種類型，它是一組能力的聚合。**

就像生命體由五蘊聚合而成，一個功能完整的插件往往同時需要多種成分。OpenStarry 的插件系統允許單一插件包提供任意組合的五蘊成分。

---

## 五蘊成分定義 (The Aggregates)

在 `PluginHooks` 中，我們使用語義化字段來聲明這些成分，它們在架構上嚴格對應五蘊：

```typescript
// PluginHooks 定義 (來自 @openstarry/sdk)
interface PluginHooks {
  ui?: IUI[];          // 色蘊 (Rupa) - 介面與顯相，接收事件並呈現輸出
  listeners?: IListener[]; // 受蘊 (Vedana) - 感官監聽，接收外部輸入
  providers?: IProvider[]; // 想蘊 (Samjna) - 認知大腦，LLM 適配器
  tools?: ITool[];     // 行蘊 (Samskara) - 執行工具，檔案/API/程式碼
  guides?: IGuide[];   // 識蘊 (Vijnana) - 靈魂指引與技能，system prompt
  commands?: SlashCommand[];
  dispose?: () => Promise<void> | void;
}
```

## 常見聚合模式

### 1. 純行蘊插件 (Pure Executor)
*   **成分：** 只有 `tools`。
*   **例子：** `fs-utils`。

### 2. 完整感官插件 (Full Sensory Plugin)
*   **成分：** `listeners` + `tools`。
*   **例子：** `discord-connector` (既能收訊息，也能發訊息)。

### 3. 靈魂注入插件 (Soul Injector)
*   **成分：** `guides` (可能附帶特定的 `tools`)。
*   **例子：** `expert-coder-skill`。
    *   **識 (Guide):** 注入工程師人設。
    *   **行 (Tool):** 附帶特定的 `linter` 工具。

---

## 架構優勢

1.  **高內聚 (High Cohesion):** 相關的能力放在同一個包裡。
2.  **原子化加載 (Atomic Loading):** 用戶安裝一個 npm 包，就獲得了完整的領域能力。
3.  **哲學一致性:** 這與 Core 的五蘊架構完美呼應。Core 是五蘊的容器，而 Plugin 是五蘊的供給者。