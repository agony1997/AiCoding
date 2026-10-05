---
title: "Multi-Agents: What's Actually Working"
author: Walden Yan（Cognition）
published: 2026-04-22
url: https://cognition.com/blog/multi-agents-working
fetched: 2026-10-05
verified: 已讀原文
origin: 新增
note: 頁面日期顯示為「04.22.26」。本文是作者前作〈Don't Build Multi-Agents〉的後續。
---

# Multi-Agents: What's Actually Working

## 一句話
多 agent 目前真正行得通的，是「只有一個 agent 動手寫、其他 agent 只出主意」的形態，例如不帶前情的 review agent；多個 agent 同時寫碼、或 agent 之間自由協商的 swarm，仍然不可行。

## 重點
- 可行範圍很窄，條件是寫入只走一條線：「setups where multiple agents contribute intelligence to a task while writes stay single-threaded」
- 多個 agent 同時寫碼的老問題沒變：「Actions carry implicit decisions.」、「decision-making can get quite fragmented in a multi-agent world where multiple agents are taking write actions.」
- Review loop 的數字：「Devin Review catches an average of 2 bugs per PR, of which roughly 58% are severe (logic errors, missing edge cases, security vulnerabilities).」；代價是可能多輪重跑：「Often the system will loop through multiple code-review cycles, finding new bugs each time (which isn't always great since it can take a while).」
- 反直覺發現：coder 與 reviewer 不共享 context 效果最好：「we found this technique to work best when the coding and review agents do not share any context beforehand.」理由一是不看 spec、從實作反推：「it is forced to reason backward from the implementation without the spec」；理由二是 context 越長判斷越差（Context Rot），reviewer 只看 diff：「The dedicated review agent gets to skip this extraneous context, only look at the diff, and re-discover any context it needs as it reads the code from scratch.」
- reviewer 回報的 bug 要由握有全局資訊的 coder 過濾，否則會出事：「does Devin properly use its broader context of user instructions, decisions, etc. to filter the bugs that come back from Devin Review? This is key to preventing looping, disobeying the user, doing work that is out of scope, and so on.」
- Smart Friend（小模型遇難題時呼叫大模型）只在兩邊都強時有效：「smart-friend works today when both models are strong. Getting it to work with an asymmetrically weaker primary, which is the version that leads to the biggest unlocks, is still an open problem」；交接資訊的折衷做法：「a reasonable 80/20 solution is to just share a fork of the full context of the primary model with the smart model.」
- Manager 拆工給子 agent 已上線，但踩到的坑：「Managers trained on small-scoped delegation default to being overly prescriptive, which backfires when the manager lacks deep codebase context. Agents assume they share state with their children when they don't.」
- 否定無結構 swarm：「the unstructured-swarm approach, arbitrary networks of agents negotiating with each other, is mostly a distraction. The practical shape is map-reduce-and-manage」

## 對 skill 設計的意義
- 原文主張：寫入只走一條線、其他 agent 只出主意。使用者過去「寫程式只在主 session、subagent 不寫碼」的做法與此一致，可保留。
- 原文主張：reviewer 不帶 coder 的前情、只看 diff，抓得比較多，這支持「獨立 review session」。推論：原文認為 reviewer「不看 spec、從實作反推」是優點，這和「拿 spec 逐條對照」的 review 是不同視角；新 skill 可考慮兩種 review 分開做。
- 原文主張：review 結果要由主 agent 依使用者指示與既有決策過濾，否則會無限重跑、越權。推論：skill 應明定 review finding 由主 session 決定採不採用，並設定 review 輪數上限。
