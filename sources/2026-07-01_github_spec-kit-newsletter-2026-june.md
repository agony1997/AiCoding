---
title: "Spec Kit - June 2026 Newsletter"
author: GitHub Spec Kit 專案（原文未署名；檔案由 Manfred Riem 提交）
published: 2026-07-01
url: https://github.com/github/spec-kit/blob/main/newsletters/2026-June.md
fetched: 2026-10-05
verified: 已讀原文
origin: 新增
note: 原文沒有署名與日期；2026-07-01 是該檔首次 commit（e8ade110）的日期。2026-08-04 曾修訂（刪掉「75 substantive articles」等新聞量數字），本摘要依修訂後版本。lean preset 與 TinySpec 在原文只被點名，沒有說明內容。
---

# Spec Kit - June 2026 Newsletter

## 一句話
Spec Kit 六月推出 /speckit.converge：實作後拿規格比對程式碼現況，只追加補漏任務、不改規格；同時坦承「review 負擔太重、小任務流程太繁瑣」仍是最常見的批評。

## 重點
- converge 的定位：「adds a ninth step that runs after /speckit.implement and answers the single most-cited concern in every review of the project: does the code actually match the spec?」
- 怎麼比：只以三份文件為準，看程式碼「現況」，不看 diff：「Converge reads spec.md, plan.md, and tasks.md as the sole source of intent」、「It is deliberately not a diff or git tool」。缺口分四類 missing／partial／contradicts／unrequested（unrequested 指「work the spec never called for」），並分嚴重度，「a constitution-MUST violation always the highest.」
- 只追加、不改寫：「Its defining design choice is that it is append-only and never rewrites.」它「never modifies the spec or plan, never renumbers existing tasks, and never touches application code」；沒有缺口時「leaves tasks.md byte-for-byte unchanged」。每個追加任務帶 source-ref（如 FR-003），形成「converge → implement → converge — that runs until no gaps remain.」
- Review overload 是公認代價：「Spec Kit is the heaviest and most flexible option (30+ agents, a full constitution/lifecycle model), which brings both the widest capability surface and the most review overhead.」；Particula Tech 的評語是「"prone to review overload" — match tool weight to task.」
- 官方的回應寫在 roadmap：「review overload, ceremony for small tasks, and verbose markdown output remain the most-cited concerns across June's balanced reviews」、「The lean preset, TinySpec, /speckit.converge, and role bundles provide answers; surfacing them to new users is the ongoing opportunity.」
- 外部實測的適用範圍：vc.ru 試了四個專案，「concluding roughly 30% of the author's work suits it — strong on greenfield, weak on research and existing code.」
- 規模數字：「Twenty-five releases shipped (v0.9.0 through v0.12.2)」；stars「106,951」→「~116,500」。SNCF Connect & Tech 自述「2–4× velocity gains」，同時「candidly flagging token-cost and governance concerns」。

## 對 skill 設計的意義
- 原文主張：converge 只讀規格、看程式碼現況、只追加任務，不碰規格與程式碼。推論：可放在「人工手測」之前，當作自動的「規格 vs 程式碼」對照；與使用者過去的「靜態對照表」目的相近，差別在由 agent 產出、每項可追溯到需求編號。
- 原文主張：review 負擔與小任務流程繁瑣是最常被抱怨的問題。推論：新 skill 應準備輕量路徑，讓小任務略過多數關卡；lean preset／TinySpec 的具體做法本文沒寫，參考前需另查原始文件。
- 原文主張：unrequested 類缺口專抓「規格沒要求卻做了」的部分。推論：這可對應 agent 寫超出範圍的傾向，適合列為 review 必查項。
