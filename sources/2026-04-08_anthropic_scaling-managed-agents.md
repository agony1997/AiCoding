---
title: "Scaling Managed Agents: Decoupling the brain from the hands"
author: Lance Martin, Gabe Cemaj, Michael Cohen（Anthropic Engineering Blog）
published: 2026-04-08
url: https://www.anthropic.com/engineering/managed-agents
fetched: 2026-10-05
verified: 已讀原文
origin: 從 LLM-KnowHow 搬入（2026-04 撰寫，2026-10-05 對照原文重寫）
note: 舊檔名日期 2026-02-01 是猜的，原文標示「Published Apr 08, 2026」。Managed Agents 是 Claude Platform 上的託管服務，不是 Claude Code 的功能。
---

# Scaling Managed Agents: Decoupling the brain from the hands

## 一句話
harness 會把「模型做不到什麼」寫死，模型變強後這些假設就過期；所以 Managed Agents 只把介面定死（session、harness、sandbox 三件可各自替換），不綁定背後的實作。

## 重點
- 開宗明義的主張：「Harnesses encode assumptions that go stale as models improve.」以及「harnesses encode assumptions about what Claude can't do on its own. However, those assumptions need to be frequently questioned because they can go stale as models improve.」
- 過期假設的實例：Sonnet 4.5 會因「context anxiety」提早收尾，所以 harness 加了 context reset；但「when we used the same harness on Claude Opus 4.5, we found that the behavior was gone. The resets had become dead weight.」
- 拆成三個元件：session（「the append-only log of everything that happened」）、harness（「the loop that calls Claude and routes Claude's tool calls」）、sandbox（「an execution environment where Claude can run code and edit files」）。原文：「This allows the implementation of each to be swapped without disturbing the others. We're opinionated about the shape of these interfaces, not about what runs behind them.」
- 舊做法把全部放在同一個 container，等於養寵物：「if a container failed, the session was lost」。拆開後 container 只是一個工具「execute(name, input) → string」，壞了就重建；harness 也能用「wake(sessionId)」從 session log 接續。
- 寫死的假設也出現在安全設計：用權限縮小的 token 來防護，「this encodes an assumption about what Claude can't do with a limited token—and Claude is getting increasingly smart.」改成結構性解法：「make sure the tokens are never reachable from the sandbox where Claude's generated code runs.」
- session 不等於 context window：壓縮、裁切都是不可逆的取捨，「It is difficult to know which tokens the future turns will need.」所以 session 只保證可回查（「getEvents()」可切片、倒回、重讀），怎麼整理 context 留給 harness，「because we can't predict what specific context engineering will be required in future models.」
- 數字：container 改成需要時才開，「our p50 TTFT dropped roughly 60% and p95 dropped over 90%.」
- 結論：「Managed Agents is a meta-harness in the same spirit, unopinionated about the specific harness that Claude will need in the future.」

## 對 skill 設計的意義
- 原文主張：harness 的每個元件都是對模型能力的假設，會隨模型變強而過期（context reset 在 Opus 4.5 變成「dead weight」）。推論：使用者舊 skill 的前提「把強模型判斷固化成文件讓弱模型照做」本身就是一種寫死的假設；新 skill 應把每條規則對應的「模型做不到什麼」寫出來，換模型時逐條檢查是否還成立。
- 原文主張：只把介面定死、實作可換。推論：新 skill 可以固定「產出物格式與存放位置」（如規格檔、審查紀錄），把「用幾個 agent、怎麼分工」留成可替換的部分，不要寫進硬規則。
- 原文主張：與其靠限制模型能做的事來防護，不如讓危險資源在結構上碰不到。推論：舊 skill 的「測試檔唯讀」若改由權限或 hook 強制，會比寫在 prompt 裡要求遵守更可靠。
