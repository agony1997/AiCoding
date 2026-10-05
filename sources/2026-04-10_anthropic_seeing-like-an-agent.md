---
title: "Seeing like an agent: how we design tools in Claude Code"
author: Thariq Shihipar（Anthropic，Claude Code 團隊）
published: 2026-04-10
url: https://claude.com/blog/seeing-like-an-agent
fetched: 2026-10-05
verified: 已讀原文
origin: 從 LLM-KnowHow 搬入（2026-04 撰寫，2026-10-05 對照原文重寫）
note: 來源是 LLM-KnowHow 的 Clippings 全文剪存（2026-04-14 剪存），已與線上原文比對內容一致。原網址現轉址到 https://claude.dev/blog/seeing-like-an-agent/ ，頁面標示「PUBLISHED Apr 10, 2026」。剪存檔 frontmatter 的 published: 2001-04-10 是錯的，author 欄空白。
---

# Seeing like an agent: how we design tools in Claude Code

## 一句話
設計 agent 的工具要配合模型當下的能力，靠讀它的輸出、反覆實驗來判斷；模型變強後，以前需要的工具可能反過來綁住它，所以要定期回頭檢查。

## 重點
- 基本原則：「You want to give it tools that are shaped to its own abilities. But how do you know what those abilities are? You pay attention, read its outputs, experiment. You learn to see like an agent.」
- AskUserQuestion 試了三次：在 ExitPlanTool 加問題參數會讓 Claude 混淆；改用特定 markdown 格式則「Claude could usually produce this format, but not reliably」；最後做成獨立工具、跳出視窗並暫停迴圈等使用者回答。關鍵：「even the best designed tool doesn't work if Claude doesn't understand how to call it.」
- 工具會過期：早期用 TodoWrite，還「inserted system reminders every 5 turns」提醒目標；模型變強後，「Being sent reminders of the todo list made Claude think that it had to stick to the list instead of modifying it when it realized it needed to change course.」於是改成可設相依、可在 subagent 間共享的 Task 工具。
- 由此的通則：「As model capabilities increase, the tools that your models once needed might now be constraining them. It's important to constantly revisit previous assumptions on what tools are needed.」也因此「it's useful to stick to a small set of models to support that have a fairly similar capabilities profile.」
- 讓模型自己找 context：早期用 RAG 預先餵程式碼片段，但「Claude was *given* this context instead of finding the context itself.」改給 Grep 工具；之後 Agent Skills 帶入 progressive disclosure，讓 agent「incrementally discover relevant context through exploration」。
- 加工具門檻很高：「Claude Code currently has ~20 tools」，「The bar to add a new tool is high, because this gives the model one more option to think about.」
- Claude Code Guide 的例子：把說明文件塞進 system prompt 會造成 context rot；只給文件連結又會把大段文件拉進 context；最後做成 subagent，「does the doc-searching in its own context … and hands back only the answer. The main agent's context stays clean.」作者也承認「this isn't a perfect solution」。

## 對 skill 設計的意義
- 原文主張：模型變強後，提醒與清單會讓模型以為必須照清單走、不敢改方向。推論：使用者舊 skill 為弱模型寫的固定步驟與反覆提醒，對強模型可能反成限制；新 skill 應定期檢查哪些規則已不需要。
- 原文主張：skill 的價值之一是 progressive disclosure——先給入口，需要時再讀下一層檔案。推論：新 skill 主檔保持精簡，細節規則放在被引用的子檔，讓模型需要時才讀。
- 原文主張：工具要配合「你用的模型」，最好只支援能力相近的少數模型。推論：舊 skill「強模型寫規則、弱模型照做」要同時服務能力差距大的模型，與這個建議有衝突；新 skill 應標明針對哪一級模型設計。
