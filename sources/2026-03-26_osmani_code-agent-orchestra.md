---
title: The Code Agent Orchestra - what makes multi-agent coding work
author: Addy Osmani
published: 2026-03-26
url: https://addyosmani.com/blog/code-agent-orchestra/
fetched: 2026-10-05
verified: 已讀原文
origin: 從 LLM-KnowHow 搬入（2026-04 撰寫，2026-10-05 對照原文重寫）
note: 舊檔名日期 2026-04-01 是猜的，原文標示 March 26, 2026。本文是作者在 O'Reilly AI CodeCon 演講的文字版。舊檔寫作者「(Google)」；頁面目前的作者簡介寫他是 Anthropic 的 Member of Technical Staff、先前在 Google 超過 14 年，發文當時任職何處頁面未載明。文中引用的 ETH Zurich 研究數字是轉述，未查原研究。
---

# The Code Agent Orchestra - what makes multi-agent coding work

## 一句話
開發者正從「和一個 AI 結對」變成「管理一組 agent」；產出已不是瓶頸，驗證才是，所以品質關卡與精確規格是整個做法的關鍵。

## 重點
- 轉變：「You used to pair with one AI. Now you manage an agent team.」單一 agent 的三道牆：context overload、no specialization、no coordination；「Subagents solve the first two. Agent Teams solve all three.」
- 專注勝過通才：「Three focused agents consistently outperform one generalist agent working three times as long.」團隊規模：「3-5 teammates is the sweet spot. Token costs scale linearly with team size.」subagent 模式「cost-neutral at roughly 220k tokens total」，缺點是要手動管相依、沒有 agent 間訊息、檔案範圍沒切好會互相覆寫。
- 品質關卡三件：計畫核准（「It's far cheaper to fix a bad plan than to fix bad code.」）、hooks（TaskCompleted 時跑 lint 與測試，「If the hook fails, the agent keeps working until it passes」）、AGENTS.md 累積經驗。
- 防卡死：「Every teammate gets a hard MAX_ITERATIONS=8」，重試前強制反思；「If stuck 3+ iterations on the same error, kill and reassign to a fresh agent.」專職審查者設定為「Model: Claude Opus 4.6 (read-only)」「Ratio: 1 reviewer per 3-4 builders」。
- 瓶頸轉移：「The bottleneck is no longer generation. It's verification.」且「Agents can write tests that are technically valid but miss the cases that matter.」結論：「Until verification infrastructure catches up with generation capabilities, human review isn't optional overhead. It's the safety system.」
- 為何關卡必要：人慢慢寫會及早感到痛，agent 大軍則讓小錯快速累積，「Your tests are equally untrustworthy because agents wrote those too.」
- 分工原則「Delegate the tasks, not the judgment」：agent 擅長「anything with a tight evaluation function」；人保留架構、決定不做什麼、以完整系統視角審查（「agents only ever have a local view」）。
- 規格是槓桿：「When you orchestrate fifty agents in parallel, vague thinking doesn't just slow you down - it multiplies.」AGENTS.md 要人寫：作者轉述 ETH Zurich 研究，LLM 生成的 AGENTS.md「offer no benefit and can marginally reduce success rates (~3% on average)」，「Never let an agent write to AGENTS.md directly.」

## 對 skill 設計的意義
- 原文主張：產出不再是瓶頸，驗證才是；agent 寫的測試同樣不可全信。推論：使用者舊 skill 的「測試檔唯讀」「獨立 review」與這個判斷一致，新 skill 可保留，並加上 hook 讓檢查自動執行。
- 原文主張：計畫核准是最便宜的關卡；判斷（架構、不做什麼）留給人。推論：人工關卡集中在計畫與架構決策，比每階段都停下來確認更有效率。
- 原文主張：唯讀審查者、迭代上限、卡住 3 次換 agent。推論：這些是可直接寫進新 skill 的具體數字規則，但數字是作者經驗值，需在自己專案驗證。
