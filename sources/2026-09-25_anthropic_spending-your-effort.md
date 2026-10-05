---
title: "Using Claude Code: Spending your effort"
author: Thariq Shihipar（Anthropic）
published: 2026-09-25
url: https://claude.dev/blog/spending-your-effort/
fetched: 2026-10-05
verified: 已讀原文
origin: 新增
note: 原文圖表為互動圖，只讀到圖說與替代文字；Terminal-Bench 數字依原文註腳為內部測試（每題 5 次），與公開排行榜不同
---

# Using Claude Code: Spending your effort

## 一句話
effort 主要是在調 Claude 花多少力氣做驗證、測邊界情況、自己下判斷；邊界情況多的任務值得開高，規格寫得越細，不同 effort 的結果越接近。

## 重點
- effort 是什麼：「effort gives the model an approximation of how much compute you want it to spend on the task.」；「higher effort will involve Claude taking more independent action for judgement and verification.」
- 作者的總結：「effort was a great way of modulating how much verification and edgecase testing Claude did and how much of its own judgement it used.」
- 規格模糊時 effort 差很大，規格詳細時差很小：模糊需求（健身 app）耗時「low(1.5 min)medium(4 min)high(11 min)max(67 min)」；給了訪談產出的詳細 spec 後「low(16 min)medium(22 min)high(33 min)max(79 min)」，且「given this spec, the models behaved much more similarly.」
- 代價是替你做更多假設：「Low effort allows Claude to respond quickly with a starting point, higher effort levels will get more work done but Claude will also make more assumptions on my behalf.」
- 作者的開發迴圈：「Give Claude a spec and ask it to interview me about any details I’m missing」→「Implement it on low effort」→「Review to make sure it got the gist of it correct, iterate on low effort as needed」→「Verify and test on high effort」。
- 高 effort 修的是漏掉的邊界情況，修不了方向錯：「increasing effort tends to reduce failures due to missing edgecases (purple blocks), but does not fix when the model has the wrong approach (blue blocks).」例：「Fable 5.1 went from 1/5 at low to 5/5 at xhigh.」；整體「Fable 5.1 at low: 140 passed…of 370 attempts. Fable 5.1 at max: 214 passed」。
- 各領域受益不同（Fable 5.1，低 effort → 高 effort 通過率）：「Security 64% → 87%, Hardware 34% → 75%, ML 54% → 73%, Science 41% → 61%, Software 43% → 56%, Media 18% → 30%, Operations 12% → 22%.」
- 低 effort 失敗的樣子是跳過驗證：「At low (about a minute per attempt), Claude would edit the code before building it or running the reproducer, and did not check that its new test would have caught the original bug.」；有人在迴圈時情況不同：「If a user were in the loop, Claude may have asked the user about the way to set up the problem, but without a user in the loop, high effort does better.」

## 對 skill 設計的意義
- 原文主張：effort 分級建議為 Low（在迴圈中快速來回）、Medium（一般功能實作）、High（驗證重要或邊界情況多，如「fixing a bug in a brownfield codebase」）、Max（「operate fully autonomously」）。
- 推論：作者的『spec＋訪談 → 低 effort 實作 → 人審 → 高 effort 驗證』與舊 spec-workflow 的『規格消化 → 實作 → 人工手測』骨架相近；新 skill 可在各階段指定 effort，而不是靠更多文件規則去壓模型的行為。
- 推論：本文高 effort 成功的關鍵是模型自己寫對照測試（brute-force solver、reference、fuzzer）；舊設計『拔掉 mock 單元測試改靜態對照表』在新 skill 中需重新評估，避免擋掉模型自我驗證。
