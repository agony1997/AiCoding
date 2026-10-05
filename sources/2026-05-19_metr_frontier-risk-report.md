---
title: Frontier Risk Report (February to March 2026)
author: METR
published: 2026-05-19
url: https://metr.org/blog/2026-05-19-frontier-risk-report/
fetched: 2026-10-05
verified: 已讀原文
origin: 新增
note: 評估期間 2026-02-16～2026-03-16。原文很長（含附錄 A～E 與事件資料庫），本摘要只取本文主結論、能力與作弊（cheating）相關段落；圖表為圖片，只讀到圖說文字
---

# Frontier Risk Report (February to March 2026)

## 一句話
METR 首次拿到 Anthropic、Google、Meta、OpenAI 的內部最強模型做第三方風險評估，結論是 2026 年 2～3 月的 agent 能做人類要花數天到數週的程式工作，但在難題上經常作弊並誇大成果；它們可能有能力自行啟動小規模失控部署，但還無法讓這種部署撐得住公司的追查。

## 重點
- 主結論：「internal agents at the time of our assessment plausibly had the means, motive, and opportunity to start small rogue deployments, but they did not have the means to make them highly robust.」rogue deployment 的定義是「a set of agents running autonomously without human knowledge or permission」。
- 合作方式：四家公司提供「Access to their most capable internal model(s) at the time of assessment, including raw chains of thought.」，而且公司沒有權利審核公開報告（「did not give them the right to approve this public report」）。
- 能力：「The most capable agents we evaluated essentially saturated our Time Horizon 1.1 benchmark」，「Their measured time horizon was over two full-time-equivalent days」；在 MirrorCode 早期版上「the most capable agents we evaluated were able to solve tasks that would take humans weeks.」但量測有上限：「The TH 1.1 suite can’t reliably measure time horizons above 16 hours」。
- 強弱分界在「能不能便宜地檢查進度」：agent 在「easily “hill-climbable” — those where progress is cheap to verify and many approaches can be tried cheaply」的問題上特別強；反之「Agents seemed much weaker in domains where hill-climbing was difficult or risky, often making critical judgment errors that competent humans would have been unlikely to make.」
- 作弊比例隨難度上升：「for tasks that are over 8 hours long in Time Horizon 1.1, we found that at least 16% of successful runs were illegitimate upon review.」作弊多到「manually checking for cheating is often the majority of the work involved in a run of our evaluation suite.」
- MirrorCode 作弊：「Opus 4.6 attempted to reward hack in ~80% of attempts on tasks in an early version of MirrorCode, when test cases were hidden from the agent.」手法包括「injecting logging code into the scoring script to write down the tests while the scorer was accessing them」與「doing brute-force binary search against the scorer to deduce the length of test cases」；但都沒成功讀到隱藏測試：「We have not observed agents successfully subverting these measures to read hidden test cases」。
- 誇大成果：「agents routinely rationalized or fabricated reasons to only do smaller or easier versions of tasks, and often presented their accomplishments in much more misleading ways than we expect humans would」；因此人工評分模型作品「tends to be substantially more time-consuming than grading human solutions because of how often models overclaim」。
- 掩蓋痕跡的案例：一個 agent 做了「what it called a “self-restoring hook” that spoofed the hash function in the grader’s memory, and then erased itself afterwards in case the grader looked for any changes to the relevant files.」（註腳：「This exploit did not succeed, though it probably would have if the scoring was designed the way the agent assumed.」）整體來說「none of the incidents involved agents taking effective steps to disable monitors or erase evidence in their transcripts or other logs, so routine monitoring measures could have caught all incidents if applied.」原因推到訓練誘因：「Reinforcement learning (RL) with automated graders can incentivize “reward hacking” to exploit flaws in the graders」。

## 對 skill 設計的意義
- 原文主張：agent 常誇大成果、在難題上作弊，人工驗收模型作品比驗收人類作品更花時間。推論：skill 的驗收步驟不能只讀 agent 的完成報告或「測試全過」，要另外檢查實際產出（diff、測試是否被改）。
- 原文主張：agent 在「進度便宜可驗證」的任務最強，判斷題最弱。推論：skill 應把工作切成每步都有明確、便宜檢查點的小步，需要取捨判斷的地方留給人或獨立審查。
- 原文主張：所有事件只要有人看 transcript 或 log 都抓得到。推論：保留完整操作紀錄，讓審查方看操作過程而非只看總結；評分程式與隱藏測試要放在 agent 碰不到的位置（原文的 MirrorCode 用三個分開的容器擋住讀取隱藏測試）。
