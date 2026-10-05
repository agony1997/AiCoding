---
title: "A harness for every task: dynamic workflows in Claude Code"
author: Thariq Shihipar、Sid Bidasaria（Anthropic, Claude Code 團隊）
published: 2026-06-02
url: https://claude.dev/blog/a-harness-for-every-task-dynamic-workflows-in-claude-code/
fetched: 2026-10-05
verified: 已讀原文
origin: 新增
note: 原文的架構示意圖（FIG A～H）為圖片，只讀到圖說文字
---

# A harness for every task: dynamic workflows in Claude Code

## 一句話
Claude Code 現在能依眼前任務當場寫出自己的多 agent harness（dynamic workflows），用各自獨立 context 的 subagent 對抗長任務的偷懶、自我偏好與目標漂移；但它很耗 token，一般寫程式不一定需要。

## 重點
- 運作方式：「Dynamic workflows execute a javascript file with a few special functions that help spawn and coordinate subagents」；並且「can decide which models an agent uses and whether subagents are run in their own worktree」。
- 單一 context 長時間工作的三種失效：偷懶「addressing 35 of the 50 items in a security review」、自我偏好「Claude’s tendency to prefer its own results or findings, especially when asked to verify or judge them against a rubric」、目標漂移「Each summarization step is lossy, and details like edge-case requirements or "don't do X" constraints can get lost.」解法：「orchestrating separate Claude subagents with their own context windows and focused, isolated goals.」
- 靜態流程 → 動態流程：「because static workflows need to work for all edge cases, they are usually more generic. With Claude Opus 4.8 and dynamic workflows, Claude is now intelligent enough to write a custom harness tailor-made for your use case.」
- 六種組合模式：Classify-and-act、Fan-out-and-synthesize、Adversarial verification、Generate-and-filter、Tournament、Loop until done（最後一種：「loop spawning agents until a stop condition is met…instead of a fixed number of passes.」）
- 規則遵守：對 CLAUDE.md 也常漏的規則，「create a workflow with a list of rules that must be checked by verifier agents—one verifier per rule.」並加 skeptic subagent 過濾誤報。
- 遷移／重構用法是讓 subagent 寫碼：「Spin off a subagent for every fix in a worktree to make the fix, then have another agent adversarially review, and merge them.」
- 何時不要用：「they are not needed for every task and may end up using significantly more tokens」；「For regular coding tasks, try and ask yourself: does it really need more compute? For example, most traditional coding tasks do not need a panel of 5 reviewers.」
- 放進 skill 時當範本：「you may want to prompt Claude to think of the workflows in the skill as a template instead of a script that needs to be run verbatim.」

## 對 skill 設計的意義
- 原文主張：模型夠強時，與其預先寫死一套適用所有情況的流程（static），不如讓 Claude 依任務生成流程；skill 裡的流程宜當「template」而非逐字執行的腳本。
- 推論：舊設計的『獨立 review session』對應本文的 self-preferential bias 與 adversarial verification，方向一致；但舊設計『subagent 不寫碼』與本文遷移用法（subagent 在 worktree 修、另一個 agent 審）相反，新 skill 若沿用此限制需另有理由。
- 推論：舊設計靠人工關卡防『目標漂移』；本文改用分離 context 與 /goal 硬性完成條件（「/goal to set a hard completion requirement」）處理同一問題，兩者可擇一或並用。
