---
title: "The Orchestrator's Tax"
author: Rahul Garg（Thoughtworks Principal Engineer，刊於 martinfowler.com）
published: 2026-07-16
url: https://martinfowler.com/articles/orchestrator-tax.html
fetched: 2026-10-05
verified: 已讀原文
origin: 新增
note: 作者自述是「exploratory work, built from one real incident」；文中成本排序是 orchestrator 自評，不是實測。規則門檻以 Claude Sonnet 5 校準。
---

# The Orchestrator's Tax

## 一句話
subagent 的主要價值不是平行加速，而是把雜訊擋在 orchestrator（主 session）的 context 之外；設計時該問的不是「開幾個 agent」，而是「什麼東西值得進到主 session 的 context」。

## 重點
- 核心主張：「the real value of a subagent is what it keeps out of that context, not how fast it runs.」
- 事件：Claude Code 在 .NET 專案上一次開 4 個 subagent，平行確實省時：「wall-clock time was around twelve minutes instead of something closer to twenty-five if the work had been serialized.」但意外的成本來自查進度：「check on the agents」這一步「pulled back the full raw transcript of a background agent: tens of thousands of tokens of JSONL, intermediate reasoning, and tool output, imported wholesale into the main thread.」作者保留：「treat that ranking as the orchestrator's account, not a measured fact.」
- token 與 context 是兩種成本：「Tokens are spent once. Context shapes every decision that follows.」；而且「A bigger context window doesn't fix that.」
- 依「需要的知識」分工，而不是依任務分工（作者稱 cognitive locality）：「Tasks that need the same mental model should usually stay together. Splitting them just forces multiple agents to rebuild the same understanding from scratch.」
- 從事件整理出四條規則，例如：「Prefer two to four agents in one wave. If the orchestrator wants five or more, it should first ask whether tasks sharing files or conventions ought to be merged.」、「Do not allow repository-wide git operations inside concurrent agent prompts.」（起因是某 agent 在其他 agent 寫檔時跑了 git stash／stash pop）、「Treat overlapping file ownership as a consolidation signal, not a cue to spawn more agents.」門檻不是通用常數：「They were also calibrated against Claude Sonnet 5」。
- subagent 不會繼承主 session 的 skill：「A subagent doesn't inherit skills active in the parent session unless the orchestrator passes them along explicitly.」修法是指出要讀哪個 skill 檔，「rather than pasting the whole skill inline」。
- 作者放棄「每次開 agent 前都要人確認」的關卡：「A universal confirmation gate would have added a round-trip to every similar session, and before long I'd almost certainly have started approving those prompts on autopilot.」、「I wasn't really improving governance. I was just adding another ritual.」
- 寫規則前的自問：「Before adding a line to a standing instruction file, ask whether a reasonably competent orchestrator would make the right decision once it knew the one missing fact.」若修法開始規定 approvals、checkpoints、mandatory steps，「that's usually a sign I'm encoding process where a small clarification would have done the job.」

## 對 skill 設計的意義
- 原文主張：到處加確認關卡，最後會變成反射性按同意。推論：使用者過去「每階段人工確認關卡」值得逐一檢查，只留真的擋得住錯的關卡，其餘改成「補一句缺的事實」。
- 原文主張：subagent 不繼承主 session 的 skill，要明確告訴它讀哪個檔。推論：新 skill 若用 subagent 做讀取或審查，prompt 應指向規則檔原文，不要另寫一份改述，避免兩份內容不一致。
- 原文主張：subagent 只把結論帶回主 session，不要把完整過程拉回來；共用同一套背景知識的工作不要拆開。推論：任務拆解階段可依「是否需要同一套背景知識」分組，而不只依功能切。
