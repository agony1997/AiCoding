---
title: Harness design for long-running application development
author: Prithvi Rajasekaran（Anthropic Labs team，Anthropic Engineering Blog）
published: 2026-03-24
url: https://www.anthropic.com/engineering/harness-design-long-running-apps
fetched: 2026-10-05
verified: 已讀原文
origin: 從 LLM-KnowHow 搬入（2026-04 撰寫，2026-10-05 對照原文重寫）
note: 舊檔名日期 2026-03-01 是猜的，原文標示「Published Mar 24, 2026」。舊檔作者寫「Claude Code team」有誤，原文署名為 Labs team 的 Prithvi Rajasekaran。
---

# Harness design for long-running application development

## 一句話
把「做事的 agent」和「評分的 agent」分開，再加一個規劃 agent，能讓 Claude 長時間自主做出可用的全端 app；但每個 harness 元件都是對模型弱點的假設，換新模型就該拆掉不再需要的部分。

## 重點
- 三 agent 架構：「The final result was a three-agent architecture—planner, generator, and evaluator—that produced rich full-stack applications over multi-hour autonomous coding sessions.」規劃者只寫產品層級規格，不寫細節實作，因為「the errors in the spec would cascade into the downstream implementation.」
- 自評不可靠：「agents tend to respond by confidently praising the work—even when, to a human observer, the quality is obviously mediocre.」解法是分開評分者：「tuning a standalone evaluator to be skeptical turns out to be far more tractable than making a generator critical of its own work」。
- 評分者實際操作 app：「the evaluator used the Playwright MCP to click through the running application the way a user would, testing UI features, API endpoints, and database states.」每項標準都有門檻，「if any one fell below it, the sprint failed」。
- sprint contract：「Before each sprint, the generator and evaluator negotiated a sprint contract: agreeing on what "done" looked like for that chunk of work before any code was written.」用途是補上高層規格與可測實作之間的落差；兩方用檔案溝通。
- 評分者要調教：「Out of the box, Claude is a poor QA agent.」早期會「identify legitimate issues, then talk itself into deciding they weren't a big deal and approve the work anyway」，也「tended to test superficially」；做法是讀評分者 log、找出與人判斷不同處、改 prompt，重複數輪。
- 成本對比（遊戲製作器）：單 agent「20 min／$9」、完整 harness「6 hr／$200」，「The harness was over 20x more expensive」；單 agent 版「the actual game was broken」，完整版能玩但物理仍有瑕疵。
- 拆 harness：「every component in a harness encodes an assumption about what the model can't do on its own, and those assumptions are worth stress testing, both because they may be incorrect, and because they can quickly go stale as models improve.」Opus 4.6 時移除 sprint 結構，評分改為最後一次；評分者「is worth the cost when the task sits beyond what the current model does reliably solo.」第二版 DAW 實驗「3 hr 50 min」「$124.70」。
- 模型演進細節：Sonnet 4.5 有明顯 context anxiety，需要 context reset；「Opus 4.5 largely removed that behavior on its own, so I was able to drop context resets from this harness entirely.」結語：「the space of interesting harness combinations doesn't shrink as models improve. Instead, it moves」。

## 對 skill 設計的意義
- 原文主張：讓獨立評分者變嚴格，比讓產出者批判自己容易。推論：使用者舊 skill 的「獨立 review」方向有原文支持，但原文也指出評分者預設會放水、只做表面測試，必須讀它的 log 反覆調 prompt，不能寫完就信。
- 原文主張：sprint contract 在動工前由產出者與評分者協商「怎樣算完成」。推論：這可對應舊 skill 的人工確認關卡——關卡要確認的重點是「可驗證的完成標準」，不只是規格文字。
- 原文主張：評分者只在任務超出模型單獨可靠完成的範圍時才值得付出成本。推論：新 skill 的審查步驟可依任務難度開關，而不是每次固定執行。
