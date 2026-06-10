# Plan57 — D-30-5 VasanaEngine 綁定規格

**狀態**: 綁定 (cycle 03-19 R3 D-§1 ratified 22/1 super-majority；pending Master Ratification Batch 16 #1)
**權威**: Master Ratification (Batch 16 dispatch 2026-05-01)
**循環**: 03-19 (Phase 6 第四棒；4/7 functional landing target)
**釋出**: v0.54.0-alpha 小版號升級
**計畫編號**: 57
**主題**: D-30-5 VasanaEngine — vasanā (習氣) 純存入觀察者 plugin

**繼承鏈**: Plan52 pushInput (cycle 03-14) → Plan54 AC-9 (cycle 03-17) → Plan56 D-30-4 多重意志 (cycle 03-18) → **Plan57 D-30-5 VasanaEngine (cycle 03-19)**

---

## §1 背景

D-30-5 VasanaEngine 實作 **vasanā** (Sanskrit: 習氣，"習氣痕跡") 作為 OpenStarry 五蘊架構中的**純存入觀察者存入日誌**。依唯識八識 mapping，vasanā 對應於阿賴耶識 (第八識) 中由過往業行 (意志、感知等) 留下的種子印象 (bīja)。

**與 D-30-4 多重意志 (Plan56) 的架構關係**:
- D-30-4 (Plan56) = 現行 — 多流並行意志佇列
- D-30-5 (Plan57) = 種子化痕跡 — 純被動觀察者紀錄印象

**MR-6 鐵律合規**: Plan57 僅在 plugin 層；ε-surface 與 Plan52 baseline 0-delta (0 欄位新增，0 常數新增)。

---

## §2 架構：Option C 雙軌純被動觀察者存入日誌

依 cycle 03-19 R3 D-§1 ratified 22/1 super-majority (DSS-CY19-§1-A LEIBNIZ Option C+ stub variant 少數保留):

### §2.1 Plan52/54/56 同構 10 維度逐字繼承

[Same 10-dimension table as EN; verbatim translation]

### §2.2 雙軌架構

**Track 1 — 純存入 (cycle 03-19 IN-SCOPE)**:
- 每 agent 實例的 append-only `vasana_log[]`
- 每筆: `{volition_id, deposit_time_utc, content_redacted, hmac_signature, nonce, prev_hash}`
- HMAC-chain 完整性
- 開機與運行時 append-only 重驗證 (D-§1-R2-E)
- 完整性違反時拒絕啟動

**Track 2 — 讀取 API (DEFERRED 至 Plan60 Blackboard-Alaya consumer)**:
- 本輪不實作
- 保留介面 stub (LEIBNIZ DSS-CY19-§1-A 少數偏好)
- Forward-only 承諾依 D-§1-clarify-A (MR-12)

**唯識戒律 5 種封閉模式 (5/5 保留)**:
1. 無中央登記 (vasanā 存入於每 agent 而非全域)
2. 無 vasanā-as-entity (恆條件性 bīja，無 svabhāva)
3. 無回溯修改 (append-only)
4. 無現行 emit 耦合 (純被動觀察者；存入 ≠ 主動影響)
5. 無 ε-surface 擴展 (僅 plugin 層)

---

## §3-§13 [其餘章節同 EN canonical Plan57_D30_5_VasanaEngine_Binding.md 對應；逐字翻譯]

### §3 ε-surface 0-delta 嚴格相等性 (7-sub-check 繼承)
[See EN §3]

### §4 三方 MR-6 簽核
[See EN §4]

### §5 Replay Cache 4 貢獻者結構化前綴表
[See EN §5]

### §6 存入內容遮蔽格式編碼
[See EN §6]

### §7 Windows fallback HMAC-chain 補償控制
[See EN §7]

### §8 Plan52 6 不變式邊界證明明文
[See EN §8]

### §9 開機與運行時 append-only 重驗證
[See EN §9]

### §10 Forward-only 承諾 + 反身性 MR-6 計數
[See EN §10]

### §11 LOC 軌跡 + 同 cycle 實作
[See EN §11]

### §12 異議保留依 MR-11
[See EN §12]

### §13 合規稽核
[See EN §13]

---

*Plan57 D-30-5 VasanaEngine 綁定規格 — cycle 03-19 R3 D-§1 ratified 22/1 — 2026-05-01*
*Master Ratification Batch 16 #1 dispatch ready*
*繼承: Plan52 → Plan54 → Plan56 → Plan57 (4/7 Phase 6 functional)*
*狀態: 綁定；待 Master Ratification*
*TW sibling per Rule #78 §78.5 BINDING-tier reflexive*
