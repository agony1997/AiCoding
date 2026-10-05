---
title: "TDD inside the agent loop - theater or actual value?"
author: Birgitta Böckeler（Thoughtworks，martinfowler.com「Exploring Gen AI」系列）
published: 2026-08-10
url: https://martinfowler.com/articles/exploring-gen-ai/tdd-in-the-agent-loop.html
fetched: 2026-10-05
verified: 已讀原文
origin: 新增
note: 文章頁面本身沒有顯示日期；2026-08-10 取自系列索引頁 martinfowler.com/articles/exploring-gen-ai.html 的「10 August 2026」與頁面 meta 的 og:article:modified_time。
---

# TDD inside the agent loop - theater or actual value?

## 一句話
小樣本實驗中，叫 agent 在自己的迴圈裡跑 TDD，品質沒有看得出的提升，token 卻多花數倍；作者已不再要求 agent 先寫測試，改用 mutation testing 監看回歸測試的品質。

## 重點
- 結論：「Based on Opus's judgment of the quality of the outcomes, there was no clearly discernable difference based on TDD workflow versus no TDD workflow. On the contrary, more than once Opus ranked the non-TDD workflow solutions slightly higher in design and test quality. There was also no meaningful difference in mutation scores across the solutions.」
- 實驗規模與限制：「I created 5 batches of solutions, with two non-TDD and two TDD solutions each.」；Sonnet 4.6 寫、Opus 4.8 評；作者自承「This is obviously a very small sample size, so take it with a grain of salt」，且任務「were all greenfield and relatively small, purely about business logic」。
- Token 倍數：標題寫「At least 3x the tokens」；附錄的 TDD／非 TDD 平均比為 Small「8.50x」、Medium「2.96x」、Large「4.89x」。作者提醒這含 cache 讀取，「it likely overstates TDD's true dollar cost」、「treat the multipliers as directional: TDD reliably cost several times more, how many times exactly is variable.」
- 非 TDD 為何較好（Opus 看 session 紀錄後的假說）：非 TDD 與 test-first 的 run「always created the full design (architecture, data types, edge cases, contracts) before writing any code or tests」；TDD 的設計則「tended to land on whatever shape the first test happened to lock in. Behaviour the agent didn't think to write a test for didn't get implemented at all.」
- 先寫測試擋不住「拿實作驗實作」的套套邏輯測試：「Writing the test first doesn't reliably prevent this」
- 沒有人看時，紅燈證明不了什麼：「When the agent both writes the test and confirms it failed, a red test tells you the agent ran it and saw failure, not that the failure was for the right reason.」實驗中 agent「still sometimes skipped or faked the red step」。
- TDD 指令難維護：「It's like an uphill battle against the training data」，且這類 prompt「even more volatile across models than simpler instructions are」。
- 方向：「being overly specific about how we want a model to do something is not a sustainable approach. Instead, we should find as many ways as we can to monitor the outcomes and give feedback.」作者「personally have stopped telling my coding agents to write tests first」，改用 mutation testing，並試用 Approved Scenarios（半人工測試，人確認後把預期結果「freeze」起來）。

## 對 skill 設計的意義
- 原文主張：TDD 的好處多半來自人在「寫測試」與「寫實作」之間停下來檢查：「Without a human checkpoint between the two, is there really any purpose left to writing the test first?」原文另列一種用法「Review checkpoint for the human」（人先看過測試再讓 AI 實作），不在本文否定範圍。推論：使用者過去「只對驗證邏輯走 TDD」若是 agent 自己跑紅綠，價值存疑；若改成人先審測試再實作，較站得住。
- 原文主張：先做整體設計再寫碼的 run 結果較好。推論：新 skill 的任務拆解階段應先產出整體設計（資料型別、邊界情況、介面約定），而不是一個測試一個測試往前推。
- 原文主張：用「監看結果」取代「規定過程」。推論：使用者的「測試檔唯讀 hook」屬於守住結果，方向一致；「靜態對照表」與 Approved Scenarios「人確認後凍結預期」的做法相近（原文未提及對照表）。
