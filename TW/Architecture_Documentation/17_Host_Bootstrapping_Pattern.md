# 17. 宿主引導模式 (Host Bootstrapping Pattern)

本文件解答了一個經典的架構悖論：**「如果 Agent Core 是絕對純淨且無 I/O 能力的，它如何讀取硬碟上的配置並加載插件？」**

答案是：**它不讀取，也不加載。這些都是『宿主 (Host)』的工作。**

## 1. 問題：雞生蛋悖論 (The Bootstrap Paradox)

我們要求 `packages/core`：
1.  **無頭 (Headless):** 不知道運行環境。
2.  **純淨 (Pure):** 不依賴 Node.js `fs` 或 `module` 模組。
3.  **微內核 (Microkernel):** 所有能力（包括讀檔）都來自插件。

但如果它連讀檔都不行，它怎麼讀取 `agent.json` 來知道自己需要加載 `fs-plugin` 呢？

## 2. 解法：宿主層 (The Host Layer)

我們在架構中明確區分了兩個運行時角色：

### 角色 A: 宿主 (Host / Coordinator)
*   **實體:** `apps/runner`, `apps/daemon`, `apps/web-server`。
*   **權限:** 擁有完整的作業系統權限（讀寫檔案、網絡請求、進程管理）。
*   **職責:** **開拓者**。它負責準備環境、讀取配置、將插件從硬碟搬運到內存。

### 角色 B: 內核 (Core)
*   **實體:** `packages/core` 的實例。
*   **權限:** 零。
*   **職責:** **繼承者**。它被動接收 Host 準備好的資源，並開始邏輯運算。

## 3. 啟動流程詳解 (The Sequence)

### Step 1. 宿主啟動 (Host Awake)
當用戶執行 `node apps/runner/dist/bin.js` 時，啟動的是 **Host (Runner)**。
Host 此時使用原生的 `fs` 模組，去讀取 `./agent.json`。

> **Host 視角:** "我讀到了配置，這個 Agent 需要 `fs` 和 `http` 插件。"

### Step 2. 物理加載 (Physical Loading)
Host 根據配置，對每個插件嘗試兩種解析策略：(1) `ref.path` 指定的檔案路徑、(2) `import(ref.name)` 套件名動態載入。使用 ESM 的 `import()` 將其載入內存。

> **Host 視角:** "我已經把 `fs-plugin.js` 載入到 RAM 了，它現在是一個物件。"

### Step 3. 依賴注入 (Injection)
Host 實例化 Core，並將這些**已經載入好的物件**傳進去。

```typescript
// Host 的代碼 (偽代碼)
const loadedPlugins = [ fsPluginObject, httpPluginObject ];
const core = new AgentCore({ plugins: loadedPlugins });
```

### Step 4. 內核覺醒 (Core Awake)
Core 啟動，它不需要去硬碟找插件，因為插件已經在它手裡了。它只需要呼叫 `plugin.initialize()`。

> **Core 視角:** "我醒來了，我手裡有工具，我不知道這些工具哪來的，但我可以用。"

## 4. 總結

**協調層 (Host) 負責「生存」，核心 (Core) 負責「生活」。**

*   **載入什麼插件？** -> 需要看資料夾內容 -> 這是 **Host** 的工作。
*   **如何使用插件？** -> 這是 **Core** 的工作。

這個模式讓我們保持了 Core 的絕對純淨，同時解決了物理加載的問題。
