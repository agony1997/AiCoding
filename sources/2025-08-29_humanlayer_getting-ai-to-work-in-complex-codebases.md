---
title: Getting AI to Work in Complex Codebases
author: Dex Horthy（HumanLayer）
published: 2025-08-29
url: https://github.com/humanlayer/advanced-context-engineering-for-coding-agents/blob/main/ace-fca.md
fetched: 2026-10-05
verified: 已讀原文
origin: 從 LLM-KnowHow 搬入（2026-04 撰寫，2026-10-05 對照原文重寫）
note: 文章放在 GitHub repo，無標示發布日；日期取該檔案的建立 commit（2025-08-29），之後到 2025-12-03 仍有修改。舊檔名寫 2026-04-01，差了七個月。作者名取自 commit 紀錄（dexhorthy），內文未署名。內文說本文根據 2025-08-20 在 Y Combinator 的演講。
---

# Getting AI to Work in Complex Codebases

## 一句話
不必等更強的模型；把整個開發流程圍繞 context 管理來設計（研究→計畫→實作，頻繁壓縮），並把人力審查放在研究與計畫上，現有模型就能處理大型舊程式庫。

## 重點
- 主張：「you can get really far with today's models if you embrace core context engineering principles」。方法稱為「frequent intentional compaction」：「designing your ENTIRE WORKFLOW around context management, and keeping utilization in the 40%-60% range (depends on complexity of the problem )」。
- 流程分三步（作者說是「three (ish)」，有時跳過研究或做多輪研究）：Research 理解程式碼與資訊流；Plan 寫出精確步驟，「being super precise about the testing / verification steps in each phase」；Implement 逐階段執行，驗證後把狀態壓回計畫檔。
- subagent 的用途是控制 context，不是扮演角色：「Subagents are not about playing house and anthropomorphizing roles. Subagents are about context control.」
- context 最怕的依序是：「Incorrect Information」「Missing Information」「Too much Noise」。
- 人力槓桿：「A bad line of code is… a bad line of code. But a bad line of a plan could lead to hundreds of bad lines of code. And a bad line of research … could land you with thousands of bad lines of code.」所以「When you review the research and the plans, you get more leverage than you do when you review the code.」
- 實例：在 300k LOC 的 Rust 專案 BAML 修 bug，PR 隔天早上獲 maintainer 核准；兩人 7 小時（「3 hours on research/plans, 4 hours on implementation」）送出 35k LOC，對方估計每個 PR 資深工程師要「3-5 days」。三人團隊「averaging about $12k on opus per month」；實習生「shipped 2 PRs on his first day, and 10 on his 8th day」。
- 不是萬靈丹：移除 parquet-java 的 hadoop 相依 7 小時失敗，因為研究沒挖夠深；另有一個 race condition 讓團隊「spent 2 weeks spinning circles」。作者強調「You have to engage with your task when you're doing this or it WILL NOT WORK.」並認為「you probably need at least one person who is an expert in the codebase」。
- spec 是團隊對齊工具：「I can't read 2000 lines of golang daily. But I *can* read 200 lines of a well-written implementation plan.」

## 對 skill 設計的意義
- 原文主張：審查研究與計畫比審查程式碼槓桿更高。推論：使用者舊 skill 的「每階段人工確認關卡」中，研究與計畫關卡最值得保留，程式碼階段可改由自動檢查為主。
- 原文主張：subagent 是用來隔離 context，不是分角色。推論：舊 skill「subagent 不寫碼」可改用「subagent 負責搜尋與摘要、回傳精簡結果」來定義，而不是用職務名稱定義。
- 原文主張：這套流程需要人深度參與，且最好有熟悉該程式庫的人。推論：新 skill 不應宣稱能讓不熟專案的人或弱模型單獨完成困難任務。
