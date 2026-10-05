---
title: "Context Engineering for AI Agents: Lessons from Building Manus"
author: Yichao 'Peak' Ji（Manus）
published: 2025-07-18
url: https://manus.im/blog/Context-Engineering-for-AI-Agents-Lessons-from-Building-Manus
fetched: 2026-10-05
verified: 已讀原文
origin: 從 LLM-KnowHow 搬入（2026-04 撰寫，2026-10-05 對照原文重寫）
note: 舊檔檔名日期 2025-07-01、作者「Manus team」不精確；原文標示 2025/7/18，作者 Yichao 'Peak' Ji。
---

# Context Engineering for AI Agents: Lessons from Building Manus

## 一句話
Manus 選擇不訓練自己的模型、改在 context 上下功夫，整理出六條讓長迴圈 agent 又穩又便宜的 context 設計原則。

## 重點
- 為何選 context engineering——改進週期從數週變數小時：「This allows us to ship improvements in hours instead of weeks」；過程靠反覆試錯：「we've rebuilt our agent framework four times」
- KV-cache 命中率是首要指標：「the KV-cache hit rate is the single most important metric for a production-stage AI agent」。原因是輸入遠多於輸出：「the average input-to-output token ratio is around 100:1」；價差：「cached input tokens cost 0.30 USD/MTok, while uncached ones cost 3 USD/MTok—a 10x difference」
- 提高命中率的做法：prompt 開頭保持不變（別放精確到秒的時間戳）、只往後加、序列化要固定：「Make your context append-only. Avoid modifying previous actions or observations. Ensure your serialization is deterministic.」
- 工具用遮罩、不在中途增刪：「unless absolutely necessary, avoid dynamically adding or removing tools mid-iteration」；工具名用一致前綴（`browser_`、`shell_`），方便一次限定一整組
- 把檔案系統當 context，壓縮要能還原：「the content of a web page can be dropped from the context as long as the URL is preserved, and a document's contents can be omitted if its path remains available in the sandbox」
- 用 todo.md 複誦目標、拉回注意力：「A typical task in Manus requires around 50 tool calls on average.」「By constantly rewriting the todo list, Manus is reciting its objectives into the end of the context.」
- 錯誤要留在 context：「Erasing failure removes evidence. And without evidence, the model can't adapt.」作者認為 error recovery 被 benchmark 低估：「it's still underrepresented in most academic work and public benchmarks」
- 別讓 context 太一致而被自己的範例帶著走（例：連續審 20 份履歷會陷入固定節奏）：「The more uniform your context, the more brittle your agent becomes.」解法是加入少量變化：「different serialization templates, alternate phrasing, minor noise in order or formatting」
- 作者自稱這些是經驗、不是定律：「None of what we've shared here is universal truth—but these are the patterns that worked for us.」

## 對 skill 設計的意義
- 原文主張：在 context 末端反覆改寫計畫，可把目標拉回模型注意力。推論：長流程 skill 讓 agent 維護一份進度檔、每完成一步就更新，比只在開頭給一次計畫可靠。
- 原文主張：失敗的動作與錯誤訊息要留著。推論：測試失敗或 review 退回時，把原始錯誤內容交給下一輪，不要只寫「請修正」。
- 原文主張：壓縮時留下路徑或 URL 就能還原。推論：skill 各階段交接時傳檔案路徑，而不是把整份內容貼進對話。
