---
title: Long-Context Isn't the Answer
author: Kyle（HumanLayer）
published: 2026-03-23
url: https://www.humanlayer.dev/blog/long-context-isnt-the-answer
fetched: 2026-10-05
verified: 已讀原文
origin: 新增
note: 文中「instruction budget」研究圖為圖片，未讀到數據；退化的證據是團隊數週使用觀察，原文沒有量化數字
---

# Long-Context Isn't the Answer

## 一句話
Context window 變大（Opus 4.6 的 1M）不等於能力變強：模型能同時遵守的指令數量沒有跟著變大，HumanLayer 因指令遵守變差改回 Opus 4.5，並主張用 sub-agent 隔離 context，而不是把 context 撐大。

## 重點
- 實際決定：「Anthropic just switched the default model in Claude Code to Opus 4.6 with a 1M context window. We tried it when it launched. But now we're switching back to Opus 4.5」。
- 觀察到的問題（數週使用心得，非量化測量）：「we noticed over the course of a couple of weeks that instruction adherence was dramatically degraded, and not just at longer context lengths.」即使在 200k 模型的「smart zone」內也一樣：「It would ignore design documents and other inputs when writing a plan file. It would make trivial mistakes, or misunderstand simple instructions - or worse, directly disobey them.」
- 原因：instruction budget 是「a measurable property of LLMs which describes how many instructions they can follow reasonably well before instruction adherence drops off」，且「strongly correlated with the size of the model」。長 context 版本通常是「the same model with some clever math (e.g. YaRN) to extend the sequence length」，所以「while the context window size increases, the instruction budget remains the same.」
- 乾草堆比喻：CLAUDE.md 每一行、工具描述、工具結果、system prompt、使用者訊息都是乾草；「imagine we increase the size of the haystack by 500% - but the size of the needle remains the same. Unless our ability to find the needle also increases by 500%, we will have a dramatically harder time finding it.」
- 解法是更積極地管理 context，作者寫了 subagent-orchestrator skill，核心一句：「All non-trivial operations should be delegated to sub-agents.」並要求「use separate sub-agents for separate tasks, and you may launch them in parallel - but do not delegate multiple tasks that are likely to have significant overlap to separate sub-agents.」
- 為何有效：「sub-agents encapsulate context and ensure that only highly-relevant context (the prompt, and the focused sub-agent result) end up in the context window, avoiding context rot」。
- 產品調整：context 警告改用絕對門檻，「trigger at the 100k token mark instead of 40% of the usable context. For opus 1m this is only 10% of the context window.」
- TL;DR 三點：「Long-context models degrade at all context lengths, not just long ones.」「More context isn't more capability - the instruction budget doesn't scale with the context window.」「Context isolation beats context expansion.」

## 對 skill 設計的意義
- 原文主張：非小事的操作都交給 sub-agent，不同任務用不同 sub-agent、重疊的任務不要拆給多個 sub-agent；主 context 只留 prompt 與精簡結果。
- 原文主張：context 警告看絕對 token 數（約 100k），不看佔窗口百分比。推論：設計長流程 skill 時，該在約 100k token 前安排交接或拆段，不要因為窗口有 1M 就把整個流程塞進同一個 context。
- 推論：CLAUDE.md 與 skill 指令本身也算乾草；指令越多越吃 instruction budget，換到長 context 模型也不會變多，所以 skill 規則應精簡、按需載入。
