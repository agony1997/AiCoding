---
title: Automating eval design and hillclimbing with Claude
author: Lance Martin（Anthropic）
published: 2026-09-28
url: https://claude.dev/blog/automating-eval-design-and-hillclimbing/
fetched: 2026-10-05
verified: 已讀原文
origin: 新增
note: 原文圖表（FIG 1～9）為圖片，只讀到圖說文字
---

# Automating eval design and hillclimbing with Claude

## 一句話
好的 eval 要貼近實際使用、留有改進空間、結果穩定；在 eval 上逐步調 prompt／skill 時，必須用保留不看的測試集抓『只對考題有效』的修改，claude-api skill 的 build-eval 與 hillclimb 指令把這套做法自動化。

## 重點
- 好 eval 的四要素：貼近實際使用（「Eval tasks mirror production.」）、強模型與高 effort 應該分數更高（「If they don’t, ambiguous tasks or a miscalibrated grader often are hobbling performance.」）、留有空間（「The most capable model at the highest effort should be well below 100%」）、每次跑結果穩定（「Low run-to-run variance.」）。好題目的標準：「A good task is one where two domain experts would reach the same verdict and everything the grader checks is stated in the task.」
- 不要專挑現在模型會錯的題目：「If you pick cases because today's model fails them, you are sampling the valleys of one model's capability surface」；改成「a useful test is to be able to say why a task is hard before you include it.」
- 評分者要先驗證：LLM 評分用「a rubric written as checkable claims (not a 1-to-5 scale)」，且「You pick the judge model, and it should not be the model you are testing.」；「it is important to read a sample of scored transcripts before believing your evaluator」。若基準已「about 95% or higher」，skill 會提醒改優化成本或延遲。
- 適合反覆調的地方是文字：「Many internal efforts and customers have focused hillclimbing on text, such as prompts and skills. These are easy to change and revert.」成功例子之一是 skill 觸發率：「The evaluation metric (the trigger rate for the skill) is directly coupled to the skill description that is being modified.」
- 防止只對考題有效（overfitting）：「Split the cases.」「Never paste failures into the prompt.」「Keep the answers structurally out of the model's reach.」hillclimb 的判斷：「if the train set improves but the test set is flat, Claude suspects overfitting and reverts the patch.」；「If the gain is within noise, it says so and recommends against merging.」
- 降成本案例，第一步就是刪舊 prompt 的儀式：44 張客服單（30 張調、14 張保留），起點「Opus 4.8 at default (high) effort settings with 74.4% decision accuracy…and a token cost of 4.6 cents per ticket.」；「The hillclimb first audited the prompt, removing mandatory tool-call rituals, a scratchpad step, and contradictory rules.」之後 Opus 5.5 low「87.8% and cut cost to 1.9 cents per ticket」，Sonnet 5 low「88.9%, at about half the cost, 1 cent per ticket」。
- 降成本案例最終結果（保留集）：「the final configuration scored 90.5% against the original setup's 78.6%, at about one fifth of the cost.」
- 提升效能案例（claude-api skill）：「From 66.1% to 87.9% in three phases of work」。關鍵發現：「the skill content was present but Claude was simply writing older API shapes (e.g., from its trained priors)」，解法是在 skill 頂端加一張舊寫法 → 新寫法對照表；另外「Tasks that never improved in performance despite addressing obvious content gaps are tells that the example or grader is flawed.」

## 對 skill 設計的意義
- 原文主張：skill 本身就是適合用 eval 反覆調整的對象（文字、好改好退）；換新模型時，舊 prompt 的「mandatory tool-call rituals」「scratchpad step」「contradictory rules」是第一批被刪的東西，刪完更便宜的模型反而更準。
- 推論：新 skill 若要決定『哪條舊規則可以拿掉』，應先有一組小 eval 和保留測試集來證明沒退步，而不是憑感覺刪或憑感覺留。
- 推論：模型會照訓練時的舊印象寫（trained priors），即使 skill 裡有正確內容；對這類已知錯誤，放在 skill 前段的『舊 → 新』對照表比散落的說明有效。舊設計的『測試檔唯讀 hook』與本文「Keep the answers structurally out of the model's reach」同屬『用結構而非叮嚀防作弊』的思路。
