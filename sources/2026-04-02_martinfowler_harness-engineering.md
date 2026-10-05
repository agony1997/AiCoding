---
title: Harness engineering for coding agent users
author: Birgitta Böckeler（Thoughtworks Distinguished Engineer，刊於 martinfowler.com）
published: 2026-04-02
url: https://martinfowler.com/articles/harness-engineering.html
fetched: 2026-10-05
verified: 已讀原文
origin: 從 LLM-KnowHow 搬入（2026-04 撰寫，2026-10-05 對照原文重寫）
note: 原文「Significant Revisions」記載 2026-02-17 先發表初版 memo，2026-04-02 發表完整文章並取代 memo（同一網址）。本摘要依 2026-04-02 完整版。作者只有 Böckeler，Martin Fowler 是網站主人，非共同作者。
---

# Harness engineering for coding agent users

## 一句話
要讓 coding agent 少被盯著，就要在它外面搭一套「事前引導（guides）＋事後感測（sensors）」的控制系統；目前最弱的一環是驗證功能是否正確。

## 重點
- 兩種控制：guides 是「feedforward controls」，在 agent 動手前引導；sensors 是「feedback controls」，事後觀察讓它自我修正。只用一種都不行：「you get either an agent that keeps repeating the same mistakes (feedback-only) or an agent that encodes rules but never finds out whether they worked (feed-forward-only).」
- 兩種執行方式：computational（測試、linter、型別檢查）「Run in milliseconds to seconds; results are reliable.」；inferential（AI code review、LLM as judge）「Slower and more expensive; results are more non-deterministic.」
- LLM 審查的定位：可處理需要語意判斷的問題，「but expensively and probabilistically. Not on every commit.」而最高衝擊的問題兩者都抓不穩：「Misdiagnosis of issues, overengineering and unnecessary features, misunderstood instructions.」且「Correctness is outside any sensor's remit if the human didn't clearly specify what they wanted in the first place.」
- 功能行為是最大缺口（behaviour harness）：常見做法是「Feed-forward: A functional specification」加上「Feed-back: Check if the AI-generated test suite is green」再配人工測試；作者評價：「This approach puts a lot of faith into the AI-generated tests, that's not good enough yet.」
- 檢查要盡量往前放：「the earlier you find issues, the cheaper they are to fix」；快的（linter、快速測試、基本 review agent）在 commit 前跑，貴的（mutation testing、大範圍 review）放到整合後。
- 人的角色是調整 harness：「Whenever an issue happens multiple times, the feedforward and feedback controls should be improved」。harness 是把人的經驗外顯化，但有極限：「A good harness should not necessarily aim to fully eliminate human input, but to direct it to where our input is most important.」
- 開放問題（原文列為待解，不是結論）：guides 與 sensors 如何保持一致不互相矛盾；「If sensors never fire, is that a sign of high quality or inadequate detection mechanisms?」；如何衡量 harness 的涵蓋度。

## 對 skill 設計的意義
- 原文主張：spec 屬於事前引導，只有引導、沒有事後感測，agent 永遠不知道規則有沒有生效。推論：使用者舊 skill 偏重規格與確認關卡（事前），新 skill 應為每條重要規則配一個能自動執行的檢查（測試、lint、hook）。
- 原文主張：AI 寫的測試目前不夠可信；LLM 審查昂貴且結果機率性。推論：舊 skill「測試檔唯讀」「獨立 review」的方向合理，但 review 不宜每次都跑，應放在能抓語意問題的時間點，並優先用確定性的檢查。
- 原文主張：重複出現的問題才去改 harness。推論：新 skill 的規則可附「出處事件」，沒有實際事件支撐的規則先不加。
