<!-- Status: CURRENT -->
<!-- Applies to: v0.59.2-alpha -->
<!-- Last verified: 2026-06-11 (every claim below checked against source + a passing test on this date; gap-fill pass v0.59.1 same day); on-prem provider hardening pass 2026-06-12 (v0.59.2) -->

# 十大宣言兌現帳本 (The Ten Tenets — Fulfillment Ledger)

> 這份文件是寫給未來的。
>
> 十大宣言（canonical：[README §十大核心宣言](./README.md#-十大核心宣言-the-ten-tenets)）是這個專案對「AI 落地那一天」的預先回答。本帳本逐條記錄每一條宣言在 v0.59.x-alpha（最終驗證 v0.59.2）的**誠實兌現狀態**：哪些已被可運行的 code＋通過的測試證明、哪些只證明到了明確標注的範圍、哪些仍是留給後人的未竟工程。
>
> 撰寫紀律（本專案用 96% 灌水的教訓換來的）：**每一個「已證明」都附證據指標；每一個邊界都明說；寬於實證的措辭一律不寫。** 一封給未來的信，可信度來自自我標注的未完成，而非宣稱的完美。

## 帳本

| # | 宣言 | 狀態 | 證據與誠實邊界 |
|---|---|---|---|
| 1 | 代理人即操作系統進程 | ✅ **已證明** | daemon 生命週期（PID 檔、named pipe/UDS IPC、attach/detach、session 落盤）全活：`apps/runner/src/daemon/` ＋ 18 個 daemon 測試檔實跑真進程。 |
| 2 | 一切皆插件 | ✅ **已證明** | 43 個可載入插件（另 1 個共享型別庫 mcp-common，無 manifest 不入 loader）；連 context manager 都強制走插件（core 缺之即 throw）。purity check（`scripts/check-purity.sh`）機器強制「core 不含插件 code」。 |
| 3 | 五蘊聚合架構 | ✅ **已證明** | 五種插件 hook 全部有活的實作與消費者：色（stdio/web UI＋listeners）、受（vedana-sensor-core→`createVedanaFn`）、想（8 個 provider）、行（tools）、識（guide-character-init）。Core 無自性＝缺五蘊插件時依三級關鍵性降級：optional-degraded（受蘊空 registry→中性值）已入測；required→throw（context manager 缺即 throw）為 production 行為（source 無條件存在，未獨立斷言）；optional-no-effect 無專屬行為測試。**邊界**：criticality 目前為宣告式 metadata，三級行為由各子系統各自硬編碼，非單一受 criticality 欄位驅動的 runtime 機制。 |
| 4 | 目錄結構即協議 | ✅ **已證明** | 插件解析三策略（path→package→系統目錄掃描＋monorepo sibling 內建路徑，v0.58 修復）；`~/.openstarry` 佈局＋`.openstarry/` 專案目錄自動偵測。 |
| 5 | 目錄結構即權限 | ✅ **已證明** | 專案層 restrict-only 權限模型（`permissions.json`，只縮不放）＋path jail（allowedPaths＋safeRealpath）＋manifest capabilities 的 runtime 工具過濾（Plan46，audit 事件實測）。 |
| 6 | 八識俱轉與功能性代理 | 🟡 **N=2 已證明，群體規模未證** | 末那對應層（klesha 濾波／volition 審議）已活（見 #8）；vitakka watchdog 為工程歸組——教義上尋（vitakka）唯與第六意識相應、不屬末那心所，此處僅指其代碼位於同一調度模組。**阿賴耶識自 v0.59 起跨進程為真**：種子（bija）以 HMAC 簽章經 daemon IPC 跨 OS 進程傳播，接收端以自己的 cluster-key 副本**獨立**驗章後入庫；錯 key 拒收的反證測試保證驗證非套套邏輯（`alaya-two-process.e2e.test.ts`）。**防重放（v0.59.3-alpha，Spec Addendum 2026-06-15）**：ISeed 凍結介面經正式 Addendum 加入選填 `nonce`（自動納入 HMAC 簽章），接收端 `acceptRemote` 接通既有 `verifyNonce` 單調計數器、拒收重放/亂序種子（`replay-nonce.test.ts` 6/6：重放拒收、竄改破簽章、跨 agent 獨立、向後相容）。**明確不宣稱**：單機限定（named pipe/UDS）、信任父進程的 key 分發、nonce 計數器未跨重啟持久化（per-run 記憶體態）、exchangeSeeds／late-joiner snapshot／subscribe 跨進程仍未實作、cross-host transport 未做。教義邊界：本映射為結構性借喻（無薰習 vāsanā／異熟 vipāka 機制，詳 doc 52 二諦聲明），不宣稱唯識學意義的阿賴耶識。 |
| 7 | 微內核與絕對純淨 | ✅ **已證明** | 編譯後 core 零插件 code（purity 機器檢查）；零 policy 常數（Plan32 遷移＋BABBAGE 8/53 分類，doc 50）；無頭設計實證＝同一 core 在 CLI REPL、daemon、MCP stdio 子進程、HTTP 端點四種色身運行（分形 e2e 以前景模式同時用上 MCP stdio 與 HTTP；daemon 色身由 daemon-process-tree.e2e 證明、CLI 由 cli e2e 證明）。 |
| 8 | 控制理論閉環模型 | ✅ **已證明（v0.59 閉環；v0.59.1 補全）** | 誤差最小化迴圈＋safety monitor 一直是活的；v0.59 補上最後一段反饋線：**vedana（感受）→ 四煩惱濾波器 → θ(t) 增益調度 → gear 仲裁**。N=2 實證：同一 arbiter 同一信心值，中性感受史被拒（gear 2）、持續樂受後被接受（gear 1）——**agent 的感受改變了它自己的調度決策**（`mano-aggregator-klesha.test.ts`）。v0.59.1 再補 **VedanaEmergency**：持續苦受觸發 thresholdBoost 封鎖快速路徑、冷卻後恢復（自 Plan28 R1 死線，`mano-aggregator-vedana-emergency.test.ts` 實證）。**明確不宣稱**：opt-in（`kleshaModulation` 缺席＝舊行為逐位元相同）；dispatcher 的 perceiveAll 半邊未接（信號走共享流）；權重與 L2/L3 值仍為 HYPOTHESIS（從未以真實數據調參）。 |
| 9 | 可插拔的記憶策略 | ✅ **已證明** | context-sliding-window／context-summary 可換裝；IContextManager 為強制插件類別。workflow 執行狀態自 v0.58 可落盤（`OPENSTARRY_WORKFLOW_STATE_DIR`）。**邊界**：CLI 模式對話歷史仍為記憶體態（落盤僅 daemon 模式）。 |
| 10 | 分形社會結構 | 🟢 **depth=3 已證明；機制同構 ⇒ 任意有限深度無已知障礙（受資源上界約束）** | v0.59 端到端實證 depth=2；**v0.59.1 升至 depth=3**：一個外部呼叫穿越三代進程（父→中→孫，各自完整認知迴圈），逐層蓋章返回 `PARENT-FINAL:MID-FINAL:CHILD-ANSWER:<孫代PID>`（`fractal-depth3.e2e.test.ts`，<2s）——**每一層用完全相同的機制（agent-ask＋mcp-server＋mcp-client），遞迴結構本身就是「由一而生萬物」**（語出道家「道生一⋯三生萬物」之轉用，非佛教語彙——中觀正面批判一因生萬法，此處僅取分形意象）。承重件＝新插件 `agent-ask`（把認知迴圈暴露為可委派工具；此前 MCP 組合的只是工具表）。v0.59.1 同時修真 daemon 層：root 自註冊（processTree 出廠即空殼→真樹）、spawnChild 樹邊＋SEC-003 拒絕實測、**父亡子收**（孤兒回收級聯首次存在，`daemon-process-tree.e2e.test.ts`）。**明確不宣稱**：路由是 MCP（MessageRouter／channel／comm-pipeline 為驗證層或未接線）；depth>3 未實測（但每層機制同構，歸納論證成立）；agent 於運行中自主決定生子的工具面尚不存在（現為組態時生子＋daemon RPC 生子）。 |

## 帳本之外：本輪挖出並修復的「交付即死」化石

完成宣言的過程本身就是考古。三條宣言的實證 e2e 在第一次運行時各自挖出一個**從交付那天起就不可能工作**的子系統——它們都有測試、都「過了」，但從未被端到端使用過：

1. **mcp-server stdio transport 是聾的**：readline 收行後重新以 `\n` split 一個永遠不含 `\n` 的 buffer——派發迴圈從未執行過一次。（分形 e2e 揪出）
2. **mcp-client 在 Windows 上 `shell:true`**：任何含空格的執行檔路徑（即預設的 `C:\Program Files\nodejs\node.exe`）必炸。（同上）
3. **DaemonKeyProvider 的「Daemon 分發 cluster key」名存實亡**：daemon 產 key、轉發給子 daemon，卻從未注入插件。（阿賴耶 e2e 揪出）

> 教訓重申（這封信最想留給未來的一句話）：**單元測試證明零件會轉，只有端到端的使用證明系統活著。「每個子系統要嘛是活的，要嘛被誠實標記」必須在交付時執行，而不是事後 reconcile。**

## 給接手者的未竟清單（依價值排序；v0.59.1 已清掉 T1b／T3b／depth=3）

1. 阿賴耶識：~~replay 防護（需 ISeed Spec Addendum 加 nonce）~~ ✅ v0.59.3-alpha 已做（Master 授權 Spec Addendum 加 nonce＋接通 verifyNonce，`replay-nonce.test.ts`）；**仍未竟**：跨主機 transport、nonce 計數器跨重啟持久化、exchangeSeeds／snapshot 跨進程化、N>2 gossip 拓撲。
2. 分形：agent 於運行中**自主決定**生子的工具面（LLM 可呼叫的 spawn 工具）；depth>3 鏈實測。
3. klesha：權重與 L2/L3 值的實證調參（Rule #72 N≥10 校準門從未達到，仍為 HYPOTHESIS）。
4. ~~地端關鍵路徑：provider-lmstudio（0 測試）與 provider-local-llama 打磨＋Ollama e2e。~~ ✅ 2026-06-12 已清：兩 provider 純函數抽取＋75 個單元測試（36＋39，wire 行為逐位元保留）；Ollama 真實 e2e 為 skipIf 門控——**本開發機無 Ollama 故從未實跑**，留給裝有 Ollama 的環境驗證。
5. CLI 模式對話歷史落盤（現僅 daemon 模式）。

---

*v0.59.0-alpha「Tenet Completion」＋ v0.59.1-alpha 缺口清理，2026-06-11；v0.59.2-alpha 地端 provider 硬化，2026-06-12。最新驗證快照：294 test files / 3155 passed / 0 failed / 4 skipped（第 4 個 skip＝Ollama e2e 於無 Ollama 的機器誠實跳過）；microkernel purity PASS。最終帳不變：8 條完全證明、#6 已證至 N=2（明確標界）、#10 已證至 depth=3（機制同構，任意有限深度無已知障礙）。*
