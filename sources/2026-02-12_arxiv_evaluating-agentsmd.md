---
title: "Evaluating AGENTS.md: Are Repository-Level Context Files Helpful for Coding Agents?"
author: Thibaud Gloaguen, Niels Mündler-Sasahara, Mark Niklas Müller, Veselin Raychev, Martin Vechev（ETH Zurich、LogicStar.ai）
published: 2026-02-12
url: https://arxiv.org/html/2602.11988v3
fetched: 2026-10-05
verified: 已讀原文
origin: 從 LLM-KnowHow 搬入（2026-04 撰寫，2026-10-05 對照原文重寫）
note: 這才是 ETH Zurich 的 AGENTS.md 研究；舊檔把它和 arXiv 2601.20404（另一組作者）合寫在一份，現拆成兩檔。版本：v1 2026-02-12、v2 2026-06-23、v3 2026-09-29，本檔依 v3，也對照讀了 v1。v1 與 v3 的差異——benchmark 名稱 AGENTbench 改為 CTXbench；v1 結論是「LLM 生成的 context file 平均降 3%、開發者寫的平均升 4%」（舊檔用的就是這組數字），v3 改為兩者對解題率都沒有統計上顯著的影響，但開發者寫的顯著優於 LLM 生成的。
---

# Evaluating AGENTS.md: Are Repository-Level Context Files Helpful for Coding Agents?

## 一句話
AGENTS.md 這類 context file 一般不會提高 coding agent 的解題率，卻讓成本增加 20% 以上；指令有被遵守，但 repo 概覽沒有用。

## 重點
- 主結論：「providing context files does not generally improve task success rates, while increasing inference cost by over 20% on average」
- 實驗規模：自建 CTXbench「138 unique instances ... across 12 recent and niche repositories」，另加 SWE-bench；四組 agent／模型：「Claude Code [3] with Sonnet-4.5 [5], Codex [26] with GPT-5.2 and GPT-5.1 mini [32], and Qwen Code [28] with Qwen3-30b-coder [34]」
- LLM 生成的 context file：解題率「reduced by 0.5% and 2% on average on SWE-bench and CTXbench」，統計上不顯著；步數「on average by 2.45 and 3.92」，成本「cost increase of 20% and 23% on average」
- 開發者寫的 context file：「Developer-provided context files improve agent performance by 2.4% on average (p=21%), significantly outperforming LLM-generated ones (p=3.8%)」；但也增加成本「on average by 3.34 steps and at most 19%」；且「they improve performance for all agents but Claude Code」
- 指令有被遵守，所以問題不在「不聽話」：「uv is used 1.6 times per instance on average when mentioned in the context files, compared to fewer than 0.01 times when it is not mentioned」
- 有 context file 時 agent 跑更多測試、讀寫更多檔：「when context files are present, the coding agents run more tests」
- repo 概覽沒幫助：「We conclude that context files are not effective at providing a repository overview.」
- 作者假設「多出來的指令讓任務變難」：「We hypothesize that these additional instructions make the task harder.」佐證是推理 token 增加，例：LLM 生成的檔案讓 GPT-5.2 在 SWE-bench 上增加「22%」
- LLM 生成的檔案多半和既有文件重複：把 repo 文件全刪掉後，「LLM-generated context files not only consistently improve performance by 2.7% on average, but also outperform developer-written ones」
- 建議：「context files ... should only contain specific additional instructions beyond what is already available in the codebase」。限制：只測 Python。

## 對 skill 設計的意義
- 原文主張：context file 裡的指令會被認真執行，連帶增加測試、探索與推理成本。推論：CLAUDE.md 或 skill 每多寫一條「要做 X」都有實際成本，只留模型無法從 codebase 得知的資訊。
- 原文主張：LLM 自動生成的 context file 大多重複既有文件，且不比不寫好。推論：skill 的規則文件應由人根據實際遇到的問題來寫，不要用 `/init` 一類工具產生後就直接採用。
- 原文主張：repo 概覽沒有幫 agent 更快找到要改的檔案。推論：skill 不必塞目錄介紹，省下的篇幅留給特殊工具、慣例等非看 code 不能得知的內容。
