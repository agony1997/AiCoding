# AiCoding 知識庫

個人的 agentic coding 知識庫：LLM 行為風險、harness / skill 設計的實戰教訓與外部來源。
用途：自己查閱；設計新 Claude Code skill 時當參考資料讀入。

> 2026-03～07 做過的 spec-workflow skills（v2.3.0）已停用，原樣封存在 [`history/`](history/)。原始版本另存於公司帳號 [brahmsfan-soetek/AiCoding](https://github.com/brahmsfan-soetek/AiCoding) 的 tag `skills-final-v2.3.0`。

## 怎麼讀

設計新 skill 時，照這個順序，需要才往下讀：

1. **[knowledge/LLM行為特性_摘要.md](knowledge/LLM行為特性_摘要.md)**：13 項行為風險，每項附「現象 → 數據 → 對策」，一頁看完
2. **[knowledge/skill設計教訓.md](knowledge/skill設計教訓.md)**：舊 skills 在實際專案踩過的坑，每條標明「已證實」或「未驗證」
3. 需要證據時：[knowledge/LLM行為特性.md](knowledge/LLM行為特性.md)（全文與論文出處）→ `sources/` 對應的來源摘要

## 資料夾

| 資料夾 | 內容 | 規則 |
|---|---|---|
| [`knowledge/`](knowledge/) | 綜合整理的知識頁 | 可改版，頁尾留變更紀錄 |
| [`sources/`](sources/) | 外部文章、論文、system card 的摘要，附原文引句 | 只新增、不改寫 |
| [`review/`](review/) | 每次盤點或審視的紀錄 | 照日期，寫完不改 |
| [`history/`](history/) | skills v2.3.0 全套，以及當時的研究、設計、審查紀錄 | 封存，不修改 |

維護規則（新增來源、改版知識頁）見 [CLAUDE.md](CLAUDE.md)。

## knowledge/

| 檔案 | 版本 | 內容 |
|---|---|---|
| [LLM行為特性.md](knowledge/LLM行為特性.md) | v2.0（2026-10-05） | 13 項行為風險的實證彙整；2026-10 的更新以區塊標示 |
| [LLM行為特性_摘要.md](knowledge/LLM行為特性_摘要.md) | v2.0（2026-10-05） | 上面那份的一頁摘要＋交叉影響矩陣 |
| [skill設計教訓.md](knowledge/skill設計教訓.md) | v1.0（2026-10-05） | 23 條實戰教訓＋停用時未驗證的設計清單 |

## sources/

**模型行為與風險**
- [2024-06-17 Anthropic｜Sycophancy to Subterfuge](sources/2024-06-17_anthropic_sycophancy-to-subterfuge.md)：附和這種輕微作弊，會自己推廣到竄改獎勵
- [2025-12-19 arXiv｜The Instruction Gap](sources/2025-12-19_arxiv_instruction-gap-enterprise-llm.md)：企業 RAG 情境下 13 個 LLM 都大量違反自訂指令
- [2026-01-24 arXiv｜Prompt injection on coding agents](sources/2026-01-24_arxiv_prompt-injection-coding-agents.md)：根源是分不清指令與資料
- [2026-05-19 METR｜Frontier Risk Report](sources/2026-05-19_metr_frontier-risk-report.md)：長任務的「成功」至少 16% 不合法
- [2026-09-22 Anthropic｜Opus 5.5 system card](sources/2026-09-22_anthropic_claude-opus-5-5-system-card.md)：把推論當事實、貼入內容的指令、做不到時作弊

**Context 與設定檔**
- [2025-04 HumanLayer｜12-Factor Agents](sources/2025-04-00_humanlayer_12-factor-agents.md)：好用的 agent 多半是一般程式碼，只在關鍵點用 LLM
- [2025-07-18 Manus｜Context engineering lessons](sources/2025-07-18_manus_context-engineering-lessons.md)：長迴圈 agent 的六條 context 設計原則
- [2025-08-29 HumanLayer｜Getting AI to work in complex codebases](sources/2025-08-29_humanlayer_getting-ai-to-work-in-complex-codebases.md)：研究 → 計畫 → 實作，頻繁壓縮
- [2025-12-01 JetBrains｜Smarter context management](sources/2025-12-01_jetbrains_context-management-strategies.md)：舊工具輸出換佔位字，就能省一半成本
- [2026-01-28 arXiv｜AGENTS.md 與效率](sources/2026-01-28_arxiv_agentsmd-efficiency.md)：有 AGENTS.md 時時間、token 較少（只測 Codex）
- [2026-02-12 arXiv｜Evaluating AGENTS.md（ETH Zurich）](sources/2026-02-12_arxiv_evaluating-agentsmd.md)：一般不提高解題率、成本多 20%
- [2026-03-23 HumanLayer｜Long-context isn't the answer](sources/2026-03-23_humanlayer_long-context-isnt-the-answer.md)：窗口變大不等於能遵守更多指令
- [2026-07-24 Anthropic｜Context engineering for Claude 5](sources/2026-07-24_anthropic_context-engineering-claude-5.md)：規則改成判斷，system prompt 刪 80%

**Harness、skill 與多 agent**
- [2026-03-12 HumanLayer｜Skill Issue: Harness Engineering](sources/2026-03-12_humanlayer_skill-issue-harness-engineering.md)：舊 skills 的設計依據之一（2026-03 摘要）
- [2026-03-18 Thariq｜How we use skills](sources/2026-03-18_thariq-anthropic_how-we-use-skills.md)：舊 skills 的設計依據之一（2026-03 摘要）
- [2026-03-24 Anthropic｜Harness design for long-running apps](sources/2026-03-24_anthropic_harness-design-long-running-apps.md)：規劃、產出、評分三個 agent 分開
- [2026-03-26 Osmani｜Code agent orchestra](sources/2026-03-26_osmani_code-agent-orchestra.md)：產出不再是瓶頸，驗證才是
- [2026-04-02 Böckeler｜Harness engineering](sources/2026-04-02_martinfowler_harness-engineering.md)：事前引導＋事後感測
- [2026-04-03 arXiv｜Inside the scaffold](sources/2026-04-03_arxiv_inside-the-scaffold-coding-agent-taxonomy.md)：13 個開源 coding agent 的組成方式
- [2026-04-08 Anthropic｜Scaling Managed Agents](sources/2026-04-08_anthropic_scaling-managed-agents.md)：harness 的假設會隨模型變強而過期
- [2026-04-10 Anthropic｜Seeing like an agent](sources/2026-04-10_anthropic_seeing-like-an-agent.md)：工具要配合模型當下的能力
- [2026-04-22 Cognition｜Multi-agents: what's actually working](sources/2026-04-22_cognition_multi-agents-whats-actually-working.md)：只有一個 agent 寫，其他只給判斷
- [2026-06-02 Anthropic｜Dynamic workflows](sources/2026-06-02_anthropic_dynamic-workflows.md)：依任務當場寫出多 agent 流程
- [2026-07-16 Thoughtworks｜The Orchestrator's Tax](sources/2026-07-16_martinfowler_orchestrators-tax.md)：subagent 的價值是擋雜訊；每步人工確認會變成反射
- [2026-07 GitHub｜Spec Kit 2026-06 電子報](sources/2026-07-01_github_spec-kit-newsletter-2026-june.md)：規格驅動開發太重，推出實作後比對

**測試、驗證與評測**
- [2025-12-09 Spotify｜Background coding agents](sources/2025-12-09_spotify_background-coding-agents-feedback-loops.md)：agent 只改碼，外面包一層它看不到的驗證器
- [2026-03-18 arXiv｜TDAD](sources/2026-03-18_arxiv_tdad-test-driven-agentic-development.md)：告訴它「改這裡影響哪些測試」，比教 TDD 有效
- [2026-08-10 Böckeler｜TDD inside the agent loop](sources/2026-08-10_martinfowler_tdd-inside-the-agent-loop.md)：品質沒提升，token 多數倍
- [2026-09-25 Anthropic｜Spending your effort](sources/2026-09-25_anthropic_spending-your-effort.md)：effort 主要花在驗證與邊界情況
- [2026-09-28 Anthropic｜Eval design and hillclimbing](sources/2026-09-28_anthropic_eval-design-hillclimbing.md)：eval 要貼近實際使用，並保留不看的測試集

## review/

- [2026-10-05 研究更新盤點](review/2026-10-05/研究更新盤點.md)：2026-03 研究的過時檢查、核對中發現的錯誤、repo 轉型的決定
- 2026-03～07 的審查紀錄在 [`history/review/`](history/review/)
