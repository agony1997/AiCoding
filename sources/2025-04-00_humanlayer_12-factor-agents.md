---
title: "12-Factor Agents - Principles for building reliable LLM applications"
author: Dex Horthy（HumanLayer）
published: 2025-04-00
url: https://github.com/humanlayer/12-factor-agents
fetched: 2026-10-05
verified: 已讀原文
origin: 從 LLM-KnowHow 搬入（2026-04 撰寫，2026-10-05 對照原文重寫）
note: 原文沒寫發布日。GitHub repo 建立於 2025-03-30（GitHub API created_at）；factor 3 原文說 2025-06 的推文出現在「About 2 months after 12-factor agents was published」，故推定 2025-04。repo 會持續更新，本檔依 2026-10-05 的 main 分支（最後 commit 2025-09-21），讀了 README 與 content/ 下各 factor 檔。舊檔檔名日期 2025-06-01 有誤。
---

# 12-Factor Agents - Principles for building reliable LLM applications

## 一句話
好用的 production agent 大多是一般程式碼、只在關鍵點用 LLM；作者整理出 12 條（加 1 條附錄）可單獨採用的設計原則。

## 重點
- 核心觀察：「A lot of them are mostly deterministic code, with LLM steps sprinkled in at just the right points to make the experience truly magical.」
- 套框架的常見結局：先到「Get to 70-80% quality bar」，接著「Realize that getting past 80% requires reverse-engineering the framework, prompts, flow, etc.」，最後「Start over from scratch」
- 建議做法是挑小概念放進既有產品：「take small, modular concepts from agent building, and incorporate them into their existing product」
- 12 條原則（名稱照原文）：

| # | 原文名稱 | 原文要點 |
|---|---|---|
| 1 | Natural Language to Tool Calls | 把自然語言轉成結構化物件，交給確定性程式處理 |
| 2 | Own your prompts | 「Don't outsource your prompt engineering to a framework.」 |
| 3 | Own your context window | 「Everything is context engineering.」 |
| 4 | Tools are just structured outputs | 「The LLM decides what to do, but your code controls how it's done.」 |
| 5 | Unify execution state and business state | 「If possible, SIMPLIFY - unify these as much as possible.」 |
| 6 | Launch/Pause/Resume with simple APIs | 長時間操作時能暫停，外部事件（如 webhook）能接續 |
| 7 | Contact humans with tool calls | 把「找人」也做成一種 tool call（如 `request_human_input`） |
| 8 | Own your control flow | 能在「選定工具」與「執行工具」之間中斷 |
| 9 | Compact Errors into Context Window | 錯誤放回 context 讓模型自己修，但要設上限 |
| 10 | Small, Focused Agents | 小而專注，步數少 |
| 11 | Trigger from anywhere, meet users where they are | 從 Slack、email、sms 等管道觸發與回應 |
| 12 | Make your agent a stateless reducer | 原文幾乎只有圖：「This one is mostly just for fun.」 |
| 13（附錄） | Pre-fetch all the context you might need | 確定會用到的資料先抓好 |

- Factor 8 的理由：若不能在選定與執行之間停下，「there's no way to review/approve the tool call before it runs」
- Factor 9 的上限：「limit to ~3 attempts of a single tool」；連續失敗到門檻「might be a great place to escalate to a human」
- Factor 10 的步數：「By keeping agents focused on specific domains with 3-10, maybe 20 steps max, we keep context windows manageable and LLM performance high.」
- Factor 13：「If you already know what tools you'll want the model to call, just call them DETERMINISTICALLY and let the model do the hard part of figuring out how to use their outputs」

## 對 skill 設計的意義
- 原文主張（factor 8）：高風險動作要能在執行前停下等人批准。推論：過去 skill 的「每階段人工確認關卡」就是放在這個點；新 skill 應把關卡放在「會改東西的動作之前」，而不是放在動作之後補看。
- 原文主張（factor 10）：agent 控制在 3～10 步、最多約 20 步。推論：拆 subagent 時用「步數與 context 大小」當切分標準。
- 原文主張（factor 13）：確定要用的資料由程式先抓。推論：skill 開場用腳本先收集 git diff、規格檔路徑等，不要讓模型自己去找。
