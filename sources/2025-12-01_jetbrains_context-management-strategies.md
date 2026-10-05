---
title: "Cutting Through the Noise: Smarter Context Management for LLM-Powered Agents"
author: Katie Fraser, Tobias Lindenbauer（JetBrains Research）
published: 2025-12-01
url: https://blog.jetbrains.com/research/2025/12/efficient-context-management/
fetched: 2026-10-05
verified: 已讀原文
origin: 從 LLM-KnowHow 搬入（2026-04 撰寫，2026-10-05 對照原文重寫）
note: 依 JetBrains 部落格文撰寫，沒有讀背後的論文（Lindenbauer et al. 2025，屬 TUM 碩士論文，發表於 NeurIPS 2025 的 Deep Learning 4 Code workshop）。舊檔標題「Efficient Context Management for LLM-Powered Agents」與原文標題不符。
---

# Cutting Through the Noise: Smarter Context Management for LLM-Powered Agents

## 一句話
在 SWE-bench Verified 上，「把舊的工具輸出換成佔位字」（observation masking）和「另用一個 LLM 寫摘要」都能省一半以上成本，而簡單的遮蔽常常更便宜、表現也不差；兩者混用最省。

## 重點
- 兩種做法的差別：遮蔽只動工具輸出、保留推理與動作：「observation masking targets the environment observation only, while preserving the action and reasoning history in full」；摘要則壓縮整段歷史（observation、action、reasoning）
- 實驗設定：遮蔽用 SWE-agent 的實作、摘要用 OpenHands 的實作；「we let our agents run for up to 250 turns」「keeping a window of the latest 10 turns」「we summarized 21 turns at a time, always retaining the most recent 10 turns in full」「All experiments were run on SWE-bench Verified, with 500 instances each」
- 兩種管理都省一半以上：「Both approaches (2) and (3) consistently cut costs by over 50% compared to (1), which leaves the agent’s memory unmanaged.」
- 遮蔽常勝：「In four out of five test settings, agents using observation masking paid less per problem and often performed better.」例：Qwen3-Coder 480B「observation masking boosted solve rates by 2.6% compared to leaving the context unmanaged, while being 52% cheaper on average」
- 摘要讓 agent 跑更久：「using LLM summarization led to agents running for an average of 52 turns, a whopping 15% longer than with observation masking」。作者推測摘要蓋掉了「該停了」的訊號：「LLM-generated summaries may actually smooth over, or hide, signs indicating that the agent should already stop trying to solve the problem」
- 摘要呼叫本身不便宜：「sometimes making up more than 7% of the total cost per instance, especially for the largest models」
- 參數不能跨 agent 照搬：遮蔽要「tuning the masking “window” hyperparameter for each agent scaffold」才追平摘要；「When we reused settings that worked for one agent in another setup, we didn’t always get the best results.」
- 混合法（平常遮蔽、太長才摘要）：「the hybrid technique reduced costs by 7% compared to pure observation masking and by 11% compared to using only LLM summarization」
- 不需訓練模型，可直接套到現有模型：「we can retrofit any existing model, including GPT-5 and Claude, with this approach」

## 對 skill 設計的意義
- 原文主張：簡單遮蔽常比聰明摘要划算，而摘要可能藏掉「該停了」的訊號。推論：subagent 回報主流程時，保留決策與結論、丟掉冗長原始輸出（log、整檔內容）即可；不要靠 LLM 改寫的長摘要來判斷「是否完成」。
- 原文主張：同一組參數換了 agent 就要重調。推論：從別處抄來的 skill 設定（保留幾輪、重試幾次）不能直接沿用，要在自己的流程上試過。
