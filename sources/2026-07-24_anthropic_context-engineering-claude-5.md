---
title: The new rules of context engineering for Claude 5 generation models
author: Thariq Shihipar（Anthropic）
published: 2026-07-24
url: https://claude.dev/blog/the-new-rules-of-context-engineering-for-claude-5-generation-models/
fetched: 2026-10-05
verified: 已讀原文
origin: 新增
---

# The new rules of context engineering for Claude 5 generation models

## 一句話
Claude 5 世代模型判斷力夠好，過去為防最壞情況而加的規則、範例、重複指示反而綁手綁腳；Anthropic 刪掉 Claude Code 八成以上 system prompt 也沒退步，建議大家一起簡化。

## 重點
- 刪減幅度與結果：「We removed over 80% of Claude Code’s system prompt for models like Claude Opus 5 and Claude Fable 5 with no measurable loss on our coding evaluations.」
- 問題是過度約束、多層指示互相打架，而舊約束現在可以拿掉：「Overall, we found that we were overconstraining Claude Code, both through our system prompt and in our CLAUDE.md files and skills.」；「while these constraints were once needed to avoid worst case scenarios, we have since found we can delete many of them and let the model use surrounding context and judgement instead.」
- 規則 → 判斷：舊 prompt 寫「In code: default to writing no comments. Never write multi-paragraph docstrings or multi-line comment blocks — one short line max.」，新 prompt 改成「Write code that reads like the surrounding code: match its comment density, naming, and idiom.」理由：「newer models have better judgement and can handle these decisions well without explicit rules.」
- 範例 → 介面設計；重複叮嚀 → 只寫在工具說明：「With our newest models, we’ve found that giving examples actually constrains them to a certain exploration space.」；「We found we could delete these repeat examples and put instructions on how to use tools in the tool descriptions rather than the system prompt.」
- 全部前置 → 漸進揭露：「we moved verification and code review into their own skills that Claude Code could selectively call.」並點名迷思「A common myth is that you want to make these a central repository for every known practice that you might run into…Instead, consider having a tree of files that can be loaded at the right time.」
- 簡單 spec → 豐富參考物：「A spec may also be a detailed test suite, or a function in a different codebase that Claude might port.」；「a HTML mockup of a design will generally produce better results than a description of the design or a screenshot.」
- CLAUDE.md 寫法：「Keep your CLAUDE.md lightweight and briefly describe what your repo is for, but spend most of the tokens on gotchas inside of the codebase.」；「Avoid stating ‘the obvious’ things Claude should know by looking at your file system or your repo.」
- Skill 寫法：「Think of skills as lightweight guides to let Claude find information when needed. Avoid making them overconstrained, except in highly important areas.」；「It’s best when skills encode particular opinions, knowledge, or best practices that are particular to you, your team, or product.」（並提供「/doctor in Claude Code to rightsize your skills, and CLAUDE.md files」）

## 對 skill 設計的意義
- 原文主張：硬性約束只留在「highly important areas」，其餘交給模型判斷；長 skill 拆成多檔、按需載入。
- 推論：舊 spec-workflow『把強模型判斷固化成文件與關卡，讓較弱模型照做』，正是本文說的「once needed to avoid worst case scenarios」型做法。本文的刪減結論只針對 Opus 5 / Fable 5 這類模型；若新 skill 仍要給較弱模型跑，不能直接套用。若給 Claude 5 世代跑，應把多數規則改寫成 gotchas，只把真正高風險處（例如測試檔唯讀 hook）留成硬關卡。
- 推論：『規格消化』階段可改用測試套件、HTML mockup、既有程式碼當 spec，取代純 markdown 說明。
