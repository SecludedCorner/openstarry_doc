# 13. 理論視角：作為控制系統的 Agent Core (Agent Core as a Control System)

本文件提供了一個獨特的理論視角，運用 **控制理論 (Control Theory)** 的框架來建模與分析 OpenStarry 的 Agent Core 架構。這不僅有助於理解系統的動態特性，更為系統的穩定性設計提供了數學依據。

## 系統模型映射

我們可以將一個正在執行任務的 Agent 視為一個經典的 **閉環反饋控制系統 (Closed-loop Feedback Control System)**。

### 1. 參考輸入 (Reference Input, $r$)
*   **定義：** 用戶的意圖或任務目標。
*   **例子：** "幫我寫一份關於 AI 的報告。"
*   **在架構中：** 這是初始的 System Prompt 和 User Message。它是系統希望達到的理想狀態。

### 2. 控制器 (Controller, $C$)
*   **定義：** 根據誤差信號計算控制輸出的組件。
*   **在架構中：** **LLM (大語言模型)**。
*   **職責：** 它觀察當前的 Context（狀態），與目標進行比對，然後生成下一步的 Action（工具調用）。

### 3. 控制量 (Control Input, $u$)
*   **定義：** 施加於系統的操縱變量。
*   **在架構中：** **Tool Calls (工具調用)**。
*   **例子：** `google_search('AI trends')`, `write_file('report.md')`。

### 4. 被控對象/過程 (Plant/Process, $P$)
*   **定義：** 受到控制量影響並產生輸出的環境。
*   **在架構中：** **外部環境** (網際網路、文件系統、第三方 API)。
*   **動態性：** 環境是不確定的、充滿噪聲的（例如網頁內容變動、API 延遲）。

### 5. 傳感器 (Sensor, $H$)
*   **定義：** 測量被控對象狀態並反饋給控制器的組件。
*   **在架構中：** **Tool Outputs (工具執行結果)** 與 **Observer Plugins**。
*   **職責：** 將環境的變化（如文件被寫入、搜索結果返回）轉化為文本信息，注入 Context。

### 6. 誤差信號 (Error Signal, $e$)
*   **定義：** 當前狀態與目標狀態的差距 ($e = r - y$)。
*   **在架構中：** 這是隱含在 Context 中的。LLM 的核心能力就是**最小化誤差**——它不斷生成 Action，直到它認為 "Task Complete"（誤差趨近於零）。

---

## 系統穩定性分析

從控制理論角度，我們可以分析 Agent 常見的失效模式：

### 1. 震盪 (Oscillation)
*   **現象：** Agent 在兩個狀態之間反覆橫跳（例如：寫文件 -> 刪文件 -> 寫文件）。
*   **原因：** 控制器 (LLM) 過度反應，或者傳感器 (Tool Output) 提供了誤導性信息，導致系統沒有收斂。
*   **解決方案：** 引入 **「積分項 (Integral Term)」** 的概念——即 Context History。通過保留歷史記憶，Agent 可以「意識到」自己正在重複，從而抑制震盪。

### 2. 發散 (Divergence)
*   **現象：** Agent 的行為越來越離譜，偏離原始目標。
*   **原因：** 誤差積累，或者正反饋迴路。通常是因為 Context Window 被無關信息填滿，導致原始目標 ($r$) 被遺忘。
*   **解決方案：** **Context 錨定 (Context Anchoring)**。始終將原始任務目標 ($r$) 放在 Context 的高權重區域（System Prompt），防止目標漂移。

### 3. 穩態誤差 (Steady-State Error)
*   **現象：** Agent 認為自己完成了任務，但用戶覺得沒完成。
*   **原因：** 傳感器精度不足。Agent 無法準確感知其 Action 的真實效果。
*   **解決方案：** 引入 **「驗證步驟 (Verification Step)」**。強制 Agent 在宣佈完成前，調用一個 `verify_result()` 工具（類似 PID 控制中的微分項預測），檢查是否真正達標。

---

## 結論

將 Agent Core 視為控制系統，讓我們明白：**智能不僅僅在於 LLM 的 IQ，更在於整個反饋迴路的設計質量。** 高質量的傳感器（工具反饋）、穩定的控制器（Prompt Engineering）和清晰的目標輸入，缺一不可。
