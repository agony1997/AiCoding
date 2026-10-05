---
title: "Background Coding Agents: Predictable Results Through Strong Feedback Loops (Honk, Part 3)"
author: Max Charas, Marc Bruggmann（Spotify Engineering）
published: 2025-12-09
url: https://engineering.atspotify.com/2025/12/feedback-loops-background-coding-agents-part-3
fetched: 2026-10-05
verified: 已讀原文
origin: 從 LLM-KnowHow 搬入（2026-04 撰寫，2026-10-05 對照原文重寫）
note: 系列文第 3 篇（原文另連到 part 1、2、4），內部代號 Honk。舊檔檔名日期 2025-12-01 有誤，原文標示 Dec 9, 2025。
---

# Background Coding Agents: Predictable Results Through Strong Feedback Loops (Honk, Part 3)

## 一句話
Spotify 讓無人監看的 coding agent 可靠的做法是：agent 只負責改碼，外面包一層它看不到內部細節的驗證器（build、test）和一個 LLM 評審，沒通過就不開 PR。

## 重點
- 三種失敗中最嚴重的是「過了 CI 但功能錯」：「The background agent produces a PR that passes CI but is functionally incorrect. This is the most serious error, as it erodes trust in our automation.」
- 失敗原因之一是 agent 自作主張：「the agent “gets creative” and decides to change things outside the scope of the prompt」
- 驗證器對 agent 是黑箱：「the agent doesn’t know what the verification does and how, it just knows that it can (and in certain cases must) call it to verify its changes」；依 codebase 內容自動啟用：「a Maven verifier activates if it finds a pom.xml file in the root of the codebase」
- 驗證器只回最相關的錯誤，省 context：「many of our verifiers use regular expressions to extract only the most relevant error messages on failure and return a very short success message otherwise」
- 開 PR 前強制全跑：「In the case of Claude Code, we do this with the stop hook. If one of the verifiers fails, the PR isn’t opened」
- LLM 評審比對 diff 與原始 prompt，抓「做太多」：「some agents were a bit too “ambitious”, trying to solve problems that weren’t strictly in their prompt, like refactoring code or disabling flaky tests」
- 評審數據：「out of thousands of agent sessions, the judge vetoes about a quarter of them. When that happens, the agent is able to course correct half the time.」最常見原因：「the agent going outside the instructions outlined in the prompt」。評審本身還沒做 eval：「We have yet to invest in evals for our judge.」
- agent 權限刻意很窄：「It can see the relevant codebase, use tools to edit files, and execute verifiers as tools.」推 code、在 Slack 跟人互動、寫 prompt 都由外部系統負責：「we believe that the reduced flexibility of the agent makes it more predictable」
- 沒有回饋迴圈的對照：「Without these feedback loops, the agents often produce code that simply doesn't work.」

## 對 skill 設計的意義
- 原文主張：驗證放在 agent 外面、用 hook 強制執行，agent 跳不過。推論：過去 skill 的「測試檔唯讀」「獨立 review」與此同方向；新 skill 可用 Claude Code 的 stop hook 在收尾前強制跑 build/test，而不是寫在指示裡請 agent 自己記得跑。
- 原文主張：評審最常抓到的是「超出 prompt 範圍」。推論：review 階段可把「diff 是否只改了規格要求的地方」列成獨立檢查項。
- 原文主張：降低 agent 彈性換取可預測性。推論：subagent 只給完成任務必需的工具，其他動作（commit、對外溝通）留在主流程或人手上。
