---
title: "TDAD: Test-Driven Agentic Development - Reducing Code Regressions in AI Coding Agents via Graph-Based Impact Analysis"
author: Pepe Alonso, Sergio Yovine（Universidad ORT Uruguay）, Victor A. Braberman（DC, UBA, Argentina）
published: 2026-03-18
url: https://arxiv.org/abs/2603.17973
fetched: 2026-10-05
verified: 已讀原文
origin: 從 LLM-KnowHow 搬入（2026-04 撰寫，2026-10-05 對照原文重寫）
note: 日期取 arXiv v1 提交日（18 Mar 2026），v2 為 19 Mar 2026；舊檔名 2026-03-01 是猜的。全文讀 HTML 版 https://arxiv.org/html/2603.17973 （v2）。實驗只用兩個本機小模型、只有 Python，作者自承前沿模型未必有相同現象。
---

# TDAD: Test-Driven Agentic Development

## 一句話
與其教 agent「怎麼做 TDD」，不如直接告訴它「改這裡會影響哪些測試」；給情境資訊比給流程指令更能減少 regression。

## 重點
- 核心主張：「agents do not need to be told how to do TDD; they need to be told which tests to check.」
- 做法：先建「程式碼—測試」相依圖，輸出成靜態文字檔：「The agent receives this file plus a 20-line skill definition; no MCP server, API calls, or graph database is required at runtime.」執行時只需「grep and pytest」。
- Phase 1（Qwen3-Coder 30B，100 題）：「TDAD reduced test-level regression rate from 6.08% to 1.82% (562 → 155 P2P failures).」嚴重案例（全部 P2P 測試失敗）從 vanilla 3 件、TDD-only 5 件降到 1 件。代價：「Resolution decreased modestly (−2 pp)」，因為 agent 看到風險時更常放棄出 patch。
- TDD 提示悖論：「adding TDD procedural instructions (write tests first, then implement) without telling the agent which specific tests to check actually increased regressions to 9.94%—worse than vanilla.」原因一：「The TDD prompt consumed context tokens with procedural instructions, pushing out repository context」；原因二：「TDD-prompted agents attempted more ambitious fixes, touching more files.」
- Phase 2（Qwen3.5-35B-A3B + OpenCode，25 題）：「TDAD improved resolution from 24% to 32% and generation from 40% to 68%」，兩組 regression 都是 0%。
- 自動改良迴圈（15 輪、每輪用 10 題評估）：最有效的單一改動是把 SKILL.md「from 107 lines of detailed 9-phase TDD instructions to 20 lines of concise guidance: fix, grep, verify. This alone quadrupled resolution (12% → 50%)」。被退回的改動包括「make SKILL.md more prescriptive」。迴圈本身也有防作弊：「the evaluation script is checksummed (SHA-256) and set read-only」。
- 結論：「Tool designers for AI agents should prioritize information density over procedural completeness」。限制：「Frontier models may not exhibit the same TDD-prompting paradox.」

## 對 skill 設計的意義
- 原文主張：對小模型而言，冗長流程指令會擠掉有用的 context，甚至讓結果更差；107 行縮成 20 行，解題率變四倍。推論：使用者舊 skill「把判斷寫成文件給弱模型照做」若文件很長，可能正踩到這個問題；新 skill 給弱模型的部分應優先提供「要看哪些檔、哪些測試」這類事實，而非步驟說明。
- 原文主張：資訊（哪些測試有風險）勝過流程（先寫測試再實作）。推論：與其規定「測試檔唯讀」這類流程規則，也可在 skill 裡產生「本次改動影響的測試清單」交給 agent。
- 原文限制：只測過本機小模型與 Python 專案。推論：這些結論套到強模型時要打折，最好在自己的專案上實測。
