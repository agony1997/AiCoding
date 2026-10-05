---
title: "Inside the Scaffold: A Source-Code Taxonomy of Coding Agent Architectures"
author: Benjamin Rombaut
published: 2026-04-03
url: https://arxiv.org/abs/2604.03515
fetched: 2026-10-05
verified: 已讀原文
origin: 從 LLM-KnowHow 搬入（2026-04 撰寫，2026-10-05 對照原文重寫）
note: 日期取 arXiv v1 提交日（3 Apr 2026），v2 為 10 Apr 2026；舊檔名 2026-04-15 有誤。全文讀 HTML 版 https://arxiv.org/html/2604.03515 。分析的 13 個開源 agent 發布於 2023-06 至 2025-03；Claude Code 因非開源被排除在外。
---

# Inside the Scaffold: A Source-Code Taxonomy of Coding Agent Architectures

## 一句話
讀 13 個開源 coding agent 的原始碼後發現，它們不適合分成幾種「架構類型」，而是由幾種迴圈組件自由組合、在連續光譜上各占位置；外部限制強的地方設計趨同，沒有定論的地方各自發散。

## 重點
- 範圍與自評：分析「13 open-source coding agent scaffolds at pinned commit hashes」，12 個維度分三層（控制架構、工具與環境介面、資源管理）；作者稱這是「to the best of the current literature search, the first comparative architectural study of coding agents at the implementation level」。
- 迴圈是組件不是類型：五種迴圈組件（ReAct、generate-test-repair、plan-execute、multi-attempt retry、tree search），「11 of 13 agents compose multiple primitives rather than relying on a single control structure.」
- 誰驅動迴圈被視為最根本的差異：Aider 由使用者驅動（LLM 有 0 個可呼叫工具）、Agentless 與 AutoCodeRover 由 scaffold 排程、「9 of 13 agents give the LLM full autonomy over tool selection」。
- 工具數「range from 0 to 37」，但能力收斂到讀、搜尋、編輯、執行四類；編輯格式趨向字串替換：str_replace_editor 類工具「appears in 5 of 13 agents」，因為「exact string matching is more reliable than line-number-based or unified-diff-based editing」。
- 趨同與發散：工具能力類別、編輯格式、執行隔離（benchmark agent 趨向 Docker）趨同；context 壓縮（「seven distinct strategies」）、狀態管理、多模型路由仍發散，「it reflects genuine uncertainty about the best approach.」
- 多模型路由的主因：「Cost optimization is the primary driver.」唯一例外是 Codex CLI 用另一個模型評估工具呼叫的安全性。
- 壓縮不是選配：唯一沒有壓縮策略的 mini-swe-agent「crashes when the context window is exceeded」。
- 工具數量的取捨：「more specialized tools reduce the LLM's per-tool reasoning burden but increase the action space it must navigate」；有的 agent 依階段限縮可見工具。
- 子 agent：13 個中有 5 個支援明確的子 agent，機制各不相同，「the absence of a dominant pattern suggest that delegation is an active design frontier.」作者提醒這些是觀察到的模式，「rather than prescriptive recommendations」。

## 對 skill 設計的意義
- 原文主張：架構是迴圈組件的組合，不是單選題。推論：新 skill 不必在「階段式流程」與「自由 ReAct」之間二選一，可在固定階段內讓 agent 自由探索。
- 原文主張：子 agent 委派仍是發散中的設計題，沒有主流做法（舊摘要寫「正在成為標準」與原文不符）。推論：舊 skill「subagent 不寫碼」這類委派規則屬於個人取捨，沒有業界標準可依循，應以自己專案的實測結果決定。
- 原文主張：依階段限縮可見工具能降低選擇負擔。推論：新 skill 可在不同階段只開放必要工具（例如審查階段唯讀），兼顧防呆與降低模型負擔。
