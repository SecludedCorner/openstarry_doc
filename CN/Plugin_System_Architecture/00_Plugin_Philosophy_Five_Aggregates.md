# 00. 插件哲学：五蕴聚合体 (Plugin as Five Aggregates)

> **[重要说明]** 本文档描述的是 OpenStarry 插件系统背后的**架构理论映射**。

## 核心哲学

**一个插件不是某种类型，它是一组能力的聚合。**

就像生命体由五蕴聚合而成，一个功能完整的插件往往同时需要多种成分。OpenStarry 的插件系统允许单一插件包提供任意组合的五蕴成分。

---

## 五蕴成分定义 (The Aggregates)

在 `PluginHooks` 中，我们使用语义化字段来声明这些成分，它们在架构上严格对应五蕴：

```typescript
// PluginHooks 定义 (来自 @openstarry/sdk)
interface PluginHooks {
  ui?: IUI[];          // 色蕴 (Rupa) - 界面与显相，接收事件并呈现输出
  listeners?: IListener[]; // 受蕴 (Vedana) - 感官监听，接收外部输入
  providers?: IProvider[]; // 想蕴 (Samjna) - 认知大脑，LLM 适配器
  tools?: ITool[];     // 行蕴 (Samskara) - 执行工具，文件/API/代码
  guides?: IGuide[];   // 识蕴 (Vijnana) - 灵魂指引与技能，system prompt
  commands?: SlashCommand[];
  dispose?: () => Promise<void> | void;
}
```

## 常见聚合模式

### 1. 纯行蕴插件 (Pure Executor)
*   **成分：** 只有 `tools`。
*   **例子：** `fs-utils`。

### 2. 完整感官插件 (Full Sensory Plugin)
*   **成分：** `listeners` + `tools`。
*   **例子：** `discord-connector` (既能收信息，也能发信息)。

### 3. 灵魂注入插件 (Soul Injector)
*   **成分：** `guides` (可能附带特定的 `tools`)。
*   **例子：** `expert-coder-skill`。
    *   **识 (Guide):** 注入工程师人设。
    *   **行 (Tool):** 附带特定的 `linter` 工具。

---

## 架构优势

1.  **高内聚 (High Cohesion):** 相关的能力放在同一个包里。
2.  **原子化加载 (Atomic Loading):** 用户安装一个 npm 包，就获得了完整的领域能力。
3.  **哲学一致性:** 这与 Core 的五蕴架构完美呼应。Core 是五蕴的容器，而 Plugin 是五蕴的供给者。
