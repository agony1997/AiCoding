---
title: "Prompt Injection Attacks on Agentic Coding Assistants: A Systematic Analysis of Vulnerabilities in Skills, Tools, and Protocol Ecosystems"
author: Narek Maloyan, Dmitry Namiot
published: 2026-01-24
url: https://arxiv.org/html/2601.17548v1
fetched: 2026-10-05
verified: 已讀原文
origin: 從 LLM-KnowHow 搬入（2026-04 撰寫，2026-10-05 對照原文重寫）
note: arXiv v1 上傳於 2026-01-24（舊檔寫 2026-01-01）。這是文獻綜整（SoK）論文，多數數字引自其他研究。原文前後數字不一致——摘要寫 42 種攻擊技術、18 種防禦、文獻年份 2021–2026；貢獻段寫 31 種、12 種；方法段寫搜尋範圍 2024-01～2025-12。
---

# Prompt Injection Attacks on Agentic Coding Assistants: A Systematic Analysis of Vulnerabilities in Skills, Tools, and Protocol Ecosystems

## 一句話
綜整 78 篇研究後，作者認為 coding agent 的 prompt injection 根源在於 LLM 分不清「指令」和「資料」，過濾式防禦在會調整策略的攻擊下幾乎都被繞過，需要從架構層防護。

## 重點
- 規模與總結論：「Our meta-analysis synthesizes findings from 78 recent studies (2021–2026), consolidating evidence that attack success rates against state-of-the-art defenses exceed 85% when adaptive attack strategies are employed.」
- 根本原因：「LLMs cannot reliably distinguish between instructions and data」，且作者認為無法修補：「The attack surface is inherent to the architecture, not an implementation flaw that can be patched.」
- 三維分類：投遞方式（D1 直接／D2 間接，如 repo 檔案／D3 協定層，如 MCP）、手法（M1 文字／M2 語意／M3 多模態）、擴散（P1 單次／P2 持久／P3 病毒式）
- 防禦被繞過（Table III，引自 Nasr et al.）：例如 Protect AI 宣稱失敗率 <5%，adaptive 攻擊下 93%；「All evaluated defenses could be bypassed with attack success rates exceeding 78% using adaptive optimization」
- 信任邊界（使用者↔agent、agent↔工具、工具↔工具、session 之間）：「73% of tested platforms fail to adequately enforce at least one boundary」
- 規則檔攻擊案例（AIShellJack，經 `.cursorrules` 等檔案植入）：「314 unique payloads covering 70 MITRE ATT&CK techniques」「41%–84% success rate across platforms」，最高是 data exfiltration（84%）、最低是 persistence（41%）
- 平台評級（Table IV）：Claude Code Low、Gemini CLI Medium、Copilot High、Codex CLI High、Cursor Critical。Claude Code 的理由：「Mandatory tool confirmation, no auto-approve flag, sandboxed MCP servers by default, explicit permission prompts for sensitive operations」；Cursor：「Auto-approve available, MCP servers unsandboxed, .cursorrules processed without validation, no egress controls」
- skill 的弱點：「The vulnerability stems from skills defining tool types but not tool targets. A skill with Read access can read any file, not just project files.」
- 建議的分級人工關卡：(a) Silent：專案內唯讀；(b) Logged：寫專案檔；(c) Confirmed：「shell execution, network requests, cross-project access」；(d) Blocked：「credential access, system modification」。同時提醒「too many prompts cause approval fatigue」

## 對 skill 設計的意義
- 原文主張：skill 的 `allowed-tools` 只限制工具種類，不限制能碰哪些檔案。推論：設計 skill 時不要把 `allowed-tools` 當成安全邊界，要限制範圍得靠 hook 或權限設定。
- 原文主張：人工確認依風險分級，太多確認會讓人麻木。推論：過去 skill 的「每階段人工確認」可改成分級——唯讀步驟不問、寫檔記錄、執行 shell 或對外連線才停下來問。
- 原文主張：repo 內的規則檔是間接注入的管道。推論：skill 讀進來的外部內容（issue、規格、第三方文件）應當作資料處理，不當作要執行的指令。
