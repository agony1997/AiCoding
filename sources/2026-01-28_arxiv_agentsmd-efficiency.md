---
title: "On the Impact of AGENTS.md Files on the Efficiency of AI Coding Agents"
author: Jai Lal Lulla, Seyedmoein Mohsenimofidi, Matthias Galster, Jie M. Zhang, Sebastian Baltes, Christoph Treude（Singapore Management University、Heidelberg University、University of Bamberg、King’s College London）
published: 2026-01-28
url: https://arxiv.org/html/2601.20404v1
fetched: 2026-10-05
verified: 已讀原文
origin: 從 LLM-KnowHow 搬入（2026-04 撰寫，2026-10-05 對照原文重寫）
note: 舊檔（2026-01-01_eth-zurich_agentsmd-impact-research.md）把本篇標為 ETH Zurich 的研究，錯誤——作者單位沒有 ETH Zurich。ETH Zurich 那篇是 arXiv 2602.11988，已拆成另一檔 2026-02-12_arxiv_evaluating-agentsmd.md。本篇 v1 上傳 2026-01-28，v2 於 2026-03-30 上傳，本檔依 v1。投稿 JAWs 2026 workshop。
---

# On the Impact of AGENTS.md Files on the Efficiency of AI Coding Agents

## 一句話
在 10 個 repo、124 個小型 PR 上讓 Codex 做「有／無 AGENTS.md」配對實驗，有 AGENTS.md 時執行時間較短、輸出 token 較少；本篇只量效率，不量改得對不對。

## 重點
- 實驗設計：「We analyze 10 repositories and 124 pull requests, executing agents under two conditions: with and without an AGENTS.md file.」
- 只用一個 agent：「the latest available Codex model was gpt-5.2-codex (4), which we use consistently across all experiments」
- 只挑小 PR：「total additions + deletions ≤ 100 LoC」，且「≤ 5 modified files」
- 只挑「根目錄只有一份 AGENTS.md」且內容含 coding conventions、architecture/project structure、project description 的 repo：「we focus on the simplest configuration: repositories that contain one AGENTS.md file only at the repository root」
- 時間（中位數）：「Median completion time shows a similar reduction, decreasing from 98.57s to 70.34s (28.23s, ≈28.64%)」；平均「decreases from 162.94s ... to 129.91s」（≈20.27%）。作者說不是少數極端值造成的：「the reduction is not driven solely by a small number of extreme runs」
- 輸出 token：「Median output tokens decrease more modestly, from 2,925.00 to 2,440.00 (485 tokens, ≈16.58%)」；平均從 5,744.81 降到 4,591.46（≈20.08%）。作者解讀：「AGENTS.md primarily reduces token usage in a small number of very high-cost runs」
- 輸入 token：平均「353,010.01 to 318,651.51; 9.73%」，但中位數「essentially unchanged or slightly higher」
- 沒測正確性：「these metrics do not capture whether agent-produced changes are correct, maintainable, or aligned with developer intent」
- 作者對原因的推測（未驗證）：「we speculate that some of the efficiency gains reported in this paper arise because AGENTS.md files describe repository structure and conventions upfront」

## 對 skill 設計的意義
- 原文主張：有 AGENTS.md 時 Codex 跑得較快、輸出較少，但只限小 PR、單一 agent，且沒驗證正確性。
- 推論：本篇與 ETH Zurich 那篇（arXiv 2602.11988）方向不同——本篇推測「事先描述 repo 結構」帶來效率；ETH 篇則發現 context file 裡的 repo 概覽沒幫助、反而增加步數與成本。兩篇的 agent、任務、量測指標都不同，不能直接合成一個結論。舊檔「兩篇結論互補非矛盾」「人工撰寫的 agentfile 提升效率（↓28%）」是舊撰寫者拼接兩篇的說法，原文都沒這樣說。
