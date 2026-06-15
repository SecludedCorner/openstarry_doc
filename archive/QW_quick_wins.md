# Cycle 02-10 Quick Wins — Codex 審閱回應文件產出

**來源**：D4 辯論（D4-R1 雙軌策略, D4-R3 三層架構, D4-R4 Quick Wins 邊界）
**交付類型**：Track A — 研究團隊本 cycle 直接產出
**狀態**：供工程團隊整合至 openstarry_doc/ 及 README

---

## QW-1：Plugin 類別表（~30 行，目標 README Architecture 節）

<!-- Layer: 1-Engineering -->

> **用途**：降低概念入門成本（Codex 第二大發現：Onboarding Cost）。以純工程語言呈現 plugin 分類，Sanskrit code name 僅作為程式碼值出現，不提供佛學解釋。

### Plugin Categories

| Engineering Term | Code Value | Description |
|-----------------|------------|-------------|
| **Input/Sensor** | `'rupa'` | Receives external signals — CLI prompts, HTTP requests, WebSocket messages, file watchers |
| **Feedback/Sensing** | `'vedana'` | Evaluates interaction quality — tool outcome sensing, safety check results, confidence gap detection |
| **Model/Cognition** | `'samjna'` | Processes information — LLM backends, context management, cognitive processing strategies |
| **Action/Tool** | `'samskara'` | Executes concrete actions — file operations, shell commands, API calls, code generation |
| **Control/Governance** | `'vijnana'` | Routes decisions — confidence routing, gear arbitration, threshold auditing, agent persona |

### Plugin Criticality Levels (Plan33)

| Level | Behavior on Missing | Example |
|-------|-------------------|---------|
| `required` | Agent refuses to start | Context manager, model selector |
| `optional-degraded` | Agent starts with reduced capability + warning | Loop quality monitor, threshold auditor |
| `optional-no-effect` | Agent starts normally, feature simply absent | Custom sensors, telemetry exporters |

**約束**：本表為 Layer 1 Engineering 文件，不解釋 Sanskrit 術語的佛學含義（D4-R3）。

---

## QW-2：Audience Tags 範本（~70 行，目標前 20 最常引用文件）

### 標記格式（PROC-DOC-3 擴展版）

每份 openstarry_doc/ 文件頂部加入：

```markdown
<!-- Status: CURRENT | HISTORICAL | ROADMAP -->
<!-- Layer: 1-Engineering | 2-Philosophy | 3-Research -->
<!-- Applies to: v0.33.0-alpha -->
<!-- Last verified: 2026-03-13 -->
> **Audience**: [目標讀者]
> **Prerequisite**: [前提知識]
```

### 建議標記（前 20 文件）

| 文件 | Layer | Status | Audience |
|------|-------|--------|----------|
| README.md（openstarry_doc/） | 1-Engineering | CURRENT | All developers |
| 01_Project_Overview.md | 1-Engineering | CURRENT | New developers, evaluators |
| 02_Architecture_Overview.md | 2-Philosophy | CURRENT | Architects, senior engineers |
| 03_Core_Concepts.md | 2-Philosophy | CURRENT | Developers seeking design rationale |
| 08_System_Runtime_Layouts.md | 1-Engineering | CURRENT | DevOps, deployment engineers |
| 10_Plugin_Development_Guide.md | 1-Engineering | CURRENT | Plugin authors |
| 20_Five_Aggregates_Mapping.md | 2-Philosophy | CURRENT | Architects interested in design philosophy |
| 21_Two_Truths_Framework.md | 2-Philosophy | CURRENT | Research team, philosophy-curious developers |
| 30_Control_Loop_Model.md | 3-Research | CURRENT | Control theory researchers |
| 31_Lyapunov_Stability.md | 3-Research | CURRENT | Stability analysis researchers |
| 40_Security_Model.md | 1-Engineering | CURRENT | Security engineers, auditors |
| 41_Sandbox_Architecture.md | 1-Engineering | CURRENT | Security engineers |
| 50_Iteration_Log_Cycle02.md | 1-Engineering | CURRENT | All team members |
| 51_Compliance_Report.md | 2-Philosophy | CURRENT | Project leads, compliance reviewers |
| 52_Microkernel_Purification.md | 2-Philosophy | CURRENT | Architects, core developers |
| 53_Security_Analysis.md | 1-Engineering | CURRENT | Security engineers |
| Roadmap.md | 1-Engineering | CURRENT | All stakeholders |
| CLI_Reference.md | 1-Engineering | CURRENT | CLI users, plugin authors |
| Config_Reference.md | 1-Engineering | CURRENT | All developers |
| API_Reference.md | 1-Engineering | CURRENT | Plugin authors, integrators |

---

## QW-3：GETTING_STARTED.md（~150 行，獨立新檔 Layer 1）

**狀態**：✅ 完整版已交付 → [`GETTING_STARTED.md`](../GETTING_STARTED.md)

**內容**：五步驟教學（Create Config → Start Agent → Write Plugin → Register → Next Steps），包含完整 TypeScript 程式碼範例、agent config JSON 範例、plugin factory 模式、工具定義與 zod schema、plugin 解析順序說明、預建 plugin 列表。

**關鍵約束**：
- 不含 `.openstarry/` 細節（Plan34 依賴，Plan34 DOC SOP 補充）
- 10 分鐘上手體驗，~150 行
- Sanskrit code values（如 `skandha: 'samskara'`）出現在程式碼中，但不解釋佛學含義
- 直接回應 Codex DX Review：「one 'build your first plugin' path」

---

## 三層文件架構定義（D4-R3 正式採納）

| Layer | 定位 | Sanskrit 規範 | 二諦聲明 |
|-------|------|-------------|---------|
| **Layer 1 Engineering** | API / 工程入門 | Zero Sanskrit *explanations*（程式碼命名可出現，不提供佛學解釋） | 不需要 |
| **Layer 2 Philosophy** | 哲學映射說明 | 完整使用 | 入口文件完整聲明；其他一行引用 |
| **Layer 3 Research** | 控制理論 / 研究 | 假設讀者有背景 | 視情況引用 Layer 2 入口 |

---

*Cycle 02-10 Quick Wins 交付物*
*LINNAEUS (#13) 三層架構設計；SCRIBE (#02) 文件標記；DARWIN (#06) 優先排序*
*整合 D4 辯論 Track A 決議*
*2026-03-13*
