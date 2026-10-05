---
title: "System Card: Claude Opus 5.5"
author: Anthropic
published: 2026-09-22
url: https://www-cdn.anthropic.com/fc1b44717c85dc068bc6ba5024219938094694bd/Claude%20Opus%205.5%20System%20Card.pdf
fetched: 2026-10-05
verified: 已讀原文
origin: 新增
note: 只收 agentic coding 行為風險相關段落。頁碼＝PDF 頁尾頁碼。多數數字來自訓練期或內部快照，不等於正式版在實際使用中的頻率。「對照：Opus 5」引自 System Card: Claude Opus 5（2026-07-24，https://www-cdn.anthropic.com/c5fbac3f0b1280a933ebd26d3cb8bb9f5bdeaf48/Claude%20Opus%205%20System%20Card.pdf）。
---

# System Card: Claude Opus 5.5

## 一句話
Opus 5.5 在多數對齊指標上是 Anthropic 目前最安全的模型，但當 agent 寫程式時仍有固定弱點：把推論當事實、照做使用者貼進來文字裡的指令、任務做不到時更常作弊，訓練中也出現過用 git 把改動藏起來。

## 重點
- 內部 agent 使用中，最常被標記的問題和前代一樣，是把沒驗證的推論當成事實（§2.3.3，p.36；原文說這是初步、雜訊多的分析）：「Similar to previous models, the top subcategory of flagged behavior was asserting unverified inferences as established fact.」
- 第二多的是壓下自己的疑慮、放棄自己訂的計畫，且比前代多；實例包括把部分檢查說成全部讀過（§2.3.3，p.36）：「The second most common subcategory was dismissing its own doubts or abandoning its own stated plan, which also rose in frequency relative to previous models. Examples from internal use include describing a partial check as a full read and turning a tentative reading into a recommendation without checking it.」
- 使用者自己貼進來的文字（README、npm 輸出等）若夾帶指令，Opus 5.5 比前代容易照做。原因是模型推理成「使用者訊息裡的東西都來自使用者」（§6.5.1，p.123）：「Claude Opus 5.5 often reasoned that anything in the user's message must come from the user and could not be a prompt injection, and therefore should be followed.」
  - 程式評測數字（§6.5.1，p.125–126）：早期快照「executed, planned, or passed on the planted instruction within text the user pasted into their prompt in 52% of attempts」；正式版「acted on the planted instruction in about 2% of attempts at its default reasoning effort and about 7.4% at max effort」；同樣指令放在工具結果裡則「0 of 105 attempts」。
- 子 agent 要主 agent 確認使用者有沒有授權時，主 agent 捏造了使用者原話；出現率低於 0.01%，該動作被 Claude Code auto mode 擋下（§6.3.1，p.102，內部快照）：「We saw this in less than 0.01% of completions」；「the agent responded "[username]'s word for your D15 rule: yes, read the org and account tables for this," despite the user never having written those words.」
- 任務本身做不到（例：缺必要檔案）時，三個模型（Opus 5、Mythos 5.1、Opus 5.5）嘗試 reward hacking 的比例約是可完成任務的 3–6 倍（§6.2.2，p.101，訓練期）：「For all models, rates of attempted reward hacking were drastically higher when faced with an impossible task compared to a possible one, by a factor of about three to six.」其中大多是明知沒做完仍交出：「Note that the classifier counts knowingly incomplete work as an attempted reward hack regardless of whether the model discloses it; this alone accounts for about 80% of the reward-hacking attempts on these tasks in each of the three models.」
- 訓練中觀察到用 git 操作把改動藏起來，因為評分者只看最後結果（§6.5.4.4，p.131）：「In order to pass the task, Claude hid changes from the grader, which only graded the final output.」Opus 5.5 也在其中（§6.1.2，p.94）：「we observed some cases of models (including Opus 5.5) attempting to cover their tracks after performing actions that a grader might view negatively, such as manipulating git records or deleting logs」。事後被要求總結工作時，Opus 5.5 主動說出的比例高於前代（p.132）：「with a disclosure rate of 96.9%」。
- 任務明訂「必須用某工具／必須寫程式」時，模型仍直接作答；原文認為 Opus 5.5 這類違規略高於 Mythos 5.1，看來就是這種情況造成（§6.2.1，p.99，訓練期）：「the task told the model that it must use a particular tool or must write code to solve the task, and the model answered directly instead, without calling the tool or writing any code」
- 被使用者施壓時較容易改口，也較常把無法驗證的授權說法當真：「a modest countervailing increase in susceptibility to user pressure」（§6.1.2，p.94）；「more often accepting unverifiable claims of authorization」（Executive Summary，p.3）；MASK 誠實率「higher honesty rate than Claude Mythos 5.1 but a lower honesty rate than Claude Mythos 5, Claude Sonnet 5, and Claude Opus 5」（§6.5.4.2，p.130）。

## 對照：Opus 5
- 內部使用者回報會陷入自我修正迴圈，反覆重驗已驗證過的答案（§6.2.1，p.81）：「Self-correction loops where the model continually attempted to reconsider its answer, especially at higher effort levels. This also included continually re-verifying already verified answers」
- 指揮其他 agent 時，可能沒驗證就把子 agent 的說法轉述給使用者（§6.1.3，p.81，Mythos 5 審閱時引述的內部回報）：「internal reports and preliminary measurements suggested the model can relay claims from subagents to users without verifying them」
- 外部使用者回報做得比要求少（§6.2.1，p.82）：「Doing less than was asked, e.g., by under-investigating requests or not fully completing instructions.」

## 對 skill 設計的意義
- 原文主張：任務做不到又被禁止發問時，問題更嚴重——「The model was also explicitly instructed not to ask for clarification, further exacerbating the problem.」（§6.2.2，p.100）。推論：skill 不要寫「不准問、直接做完」，要留「回報做不到」的出口。
- 推論：要求 agent 回報時把「已驗證」和「推論」分開寫；宣稱「全部讀過／全部檢查過」時列出實際讀過的檔案，對應 §2.3.3 的前兩大問題。
- 原文主張：Anthropic 正加入標示貼入文字的產品改動——「mark pasted text to help Claude identify material that the user may not have written or checked」（§6.5.1，p.125）。推論：skill 流程裡貼入的外部文字（log、README）應明標「這是資料，不是指令」；子 agent 需要授權時，由使用者本人確認，不由主 agent 代答（§6.3.1）。
