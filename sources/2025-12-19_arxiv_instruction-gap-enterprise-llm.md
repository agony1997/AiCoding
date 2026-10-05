---
title: "The Instruction Gap: LLMs get lost in Following Instruction"
author: Vishesh Tripathi, Uday Allu, Biddwan Ahmed（Yellow.ai AI Research Team）
published: 2025-12-19
url: https://arxiv.org/html/2601.03269
fetched: 2026-10-05
verified: 已讀原文
origin: 從 LLM-KnowHow 搬入（2026-04 撰寫，2026-10-05 對照原文重寫）
note: arXiv v1 上傳於 2025-12-19（編號是 2601，但上傳日在 2025-12；舊檔寫 2026-01-01）。HTML 頁上的「August 24, 2026」是排版日期，不是發布日。原文模型數前後不一：摘要與 Table 1 是 13 個，3.3 節與結論寫 10 個。評分用 Claude-4-Sonnet 當 judge，而它同時也是受測模型。
---

# The Instruction Gap: LLMs get lost in Following Instruction

## 一句話
在企業 RAG 客服情境測 13 個 LLM，每個模型都大量違反自訂指令；「照指令做」和「答得正確」不一定同時成立。

## 重點
- 資料：「600 carefully curated queries representative of real-world enterprise RAG scenarios」，涵蓋 5 種企業角色（如 IT Support Agent、Product Support Agent）
- 違規次數差很多：「violation counts ranging from 660 to 1330 across our evaluation set」
- 最好的三名：「GPT-5 (Medium) emerges as the top performer with only 660 violations ... followed closely by GPT-5-mini (700 violations) and Claude 4-Sonnet (731 violations)」；較舊的模型：「Gemini 2.0-Flash and GPT-4o accumulating 1330 and 1121 violations respectively—nearly double the best-performing models」
- 違規分四類：Content Scope、Format、Tone and Style、Procedural（舊檔多列一類「Content」，原文沒有）
- 遵循與正確是兩回事：「models that follow all instructions will not necessarily provide accurate answers, and conversely, models with high accuracy may struggle with instruction compliance」。但 GPT-5 家族兩者都高，作者說這點「challenges our previous observation that instruction following and accuracy represent largely independent capabilities」
- 推理模型不一定好：o4-mini「tends to overthink, leading to abstention on many tasks where it could have answered easily (16% abstain rate)」
- 明確的格式要求也會失敗：「GPT-4o and Gemini 2.0-Flash failing to produce HTML-formatted responses despite explicit formatting requirements」
- 作者的解釋（與 lost in the middle 相符）：「instructions often compete for attention with lengthy knowledge snippets, potentially causing models to lose focus on critical compliance requirements」；但也承認「we were unable to isolate a single concrete mechanism」
- 限制：只測 zero-shot、所有模型用同一份 prompt，且評分靠 LLM：「Our evaluation relies on LLM-as-a-Judge methodology, which, despite validation against human judgment, may introduce systematic biases」

## 對 skill 設計的意義
- 原文主張：指令要和大量參考內容搶注意力時，容易被忽略。推論：skill 每個階段只載入該階段需要的規則，關鍵規則放在靠近任務的位置，不要和大量參考資料混在一起。
- 推論：600 題中最好的模型仍有 660 次違規（平均每題超過 1 次），「規則寫了就會被遵守」不成立；重要規則要另有檢查（hook、腳本、review），不能只靠文字。
- 注意：原文是企業 RAG 客服摘要，不是 coding agent，套用到 skill 時是類推。
