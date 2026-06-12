<!-- Status: CURRENT -->
<!-- Written: 2026-06-12 -->
<!-- 性質: 操作手冊。Zenodo 存繳需要帳號持有人親手操作；本文把無需授權的準備工作全部做完，把需要 Master 的步驟明確標出。 -->

# Zenodo DOI 存繳 Runbook（openstarry_doc）

目的：把 openstarry_doc（含十大宣言、兌現帳本 TENETS_FULFILLMENT.md、致未來的信 LETTER_TO_THE_FUTURE.md）存繳為**可引用的研究 artifact**，取得 DOI。Software Heritage 已完成入庫（2026-06-11，保存）；Zenodo 補上的是 DOI 可引用性與學術檢索。

## 決策狀態（2026-06-12 更新）

| # | 決策 | 狀態 |
|---|---|---|
| 1 | **作者署名** | ✅ 已定（2026-06-12）：Yulin Yang（楊佑麟），affiliation = SecludedCorner |
| 2 | **License** | ✅ 已定（2026-06-12）：文件庫 CC-BY-4.0；代碼兩 repo Apache-2.0。三個 LICENSE 檔皆已入庫 |
| 3 | **Zenodo 帳號操作** | 🔴 **唯一剩餘步驟**：登入、確認 metadata、按下 Publish——DOI 一經發出不可撤回。Master 親手執行下方步驟 |

> `.zenodo.json` 與 `CITATION.cff` 已按上述決策**實際寫入 repo root**（非草稿）——下方範本僅供歷史對照，操作時無需再貼。

## 已備好的 metadata（決策確定後直接貼用）

`.zenodo.json`（放 repo root；`<TODO>` 兩處待決策 1/2）：

```json
{
  "title": "OpenStarry: An Agent OS Reference Architecture — Documentation Corpus, Tenets Fulfillment Ledger, and Letter to the Future",
  "description": "Documentation corpus of OpenStarry, a headless plugin-microkernel AI agent framework built on the Five Aggregates ontology. Includes: the Ten Tenets (canonical), the Tenets Fulfillment Ledger (per-tenet honest evidence against a running implementation, v0.59.x-alpha: 8 fully proven, #6 bounded at N=2, #10 proven to depth=3), the Letter to the Future (charter + honest process data: 87.5% spec-vs-merged gap, 96% closure inflation measured by internal audit), architecture decision records with rejected alternatives, and the microkernel purification rationale. Companion code repositories: https://github.com/SecludedCorner/openstarry and https://github.com/SecludedCorner/openstarry_plugin",
  "upload_type": "publication",
  "publication_type": "report",
  "creators": [{ "name": "<TODO: 決策 1>" }],
  "license": "<TODO: 決策 2，例 cc-by-4.0>",
  "keywords": ["AI agents", "agent operating system", "microkernel", "plugin architecture", "reference architecture", "Five Aggregates", "software documentation", "honest process data"],
  "related_identifiers": [
    { "relation": "isSupplementTo", "identifier": "https://github.com/SecludedCorner/openstarry_doc" },
    { "relation": "references", "identifier": "https://github.com/SecludedCorner/openstarry" },
    { "relation": "references", "identifier": "https://github.com/SecludedCorner/openstarry_plugin" }
  ]
}
```

`CITATION.cff`（放 repo root；同樣兩處 TODO）：

```yaml
cff-version: 1.2.0
message: "If you use this documentation corpus, please cite it as below."
title: "OpenStarry: An Agent OS Reference Architecture — Documentation Corpus"
authors:
  - name: "<TODO: 決策 1>"
license: "<TODO: 決策 2>"
repository: "https://github.com/SecludedCorner/openstarry_doc"
# doi: 存繳完成後 Zenodo 會發 DOI，回填此欄
```

## 操作步驟（Master 親手；二擇一）

**路徑 A — GitHub 整合（建議，之後每個 release 自動存繳新版）**
1. 用要掛名的帳號登入 https://zenodo.org（可用 GitHub OAuth）。
2. Zenodo → GitHub settings → 找到 `SecludedCorner/openstarry_doc` → 切換 ON。
3. 把 `.zenodo.json`（含已定的署名與 license）commit 進 repo root、push。
4. 在 openstarry_doc 發一個 GitHub Release（tag 例 `doc-v1.0.0`）→ Zenodo 自動建 deposit 並發 DOI。
5. 把 DOI 回填 CITATION.cff 與 README，再 push 一次。

**路徑 B — 手動上傳（一次性）**
1. 本機打包：`git -C openstarry_doc archive --format=zip -o openstarry_doc.zip publish`。
2. zenodo.org → New upload → 上傳 zip → 按上方 metadata 填表 → Publish。
3. DOI 回填同上。

## 完成判準

- [ ] DOI 已發出且 resolve 到 deposit 頁
- [ ] CITATION.cff 與 README 含 DOI badge/連結
- [ ] memory 與 CHANGELOG 記錄 DOI

---
*本 runbook 由 coordinator 於 2026-06-12 備妥；三個決策點之外的工作已全部完成。*
