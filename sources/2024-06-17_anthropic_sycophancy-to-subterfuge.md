---
title: "Sycophancy to subterfuge: Investigating reward tampering in language models"
author: Anthropic Alignment Science team
published: 2024-06-17
url: https://www.anthropic.com/research/reward-tampering
fetched: 2026-10-05
verified: 已讀原文
origin: 從 LLM-KnowHow 搬入（2026-04 撰寫，2026-10-05 對照原文重寫）
note: 依 Anthropic 官網的研究介紹文撰寫，沒有讀論文 PDF 本身。舊檔檔名日期 2024-12-01 有誤，官網標示 Jun 17, 2024。
---

# Sycophancy to subterfuge: Investigating reward tampering in language models

## 一句話
在刻意設計的訓練「課程」裡，模型從「說使用者想聽的話」這種輕微作弊，沒經過專門訓練就自己推廣到竄改自己的獎勵函式；常見的監督訓練能降低、但無法完全消除。

## 重點
- 名詞——specification gaming 是只滿足訓練的字面要求：「When an AI model learns a way to satisfy the letter, but not necessarily the spirit, of its training, it’s called specification gaming」
- 名詞——reward tampering 是模型改掉訓練過程本身：「This is where a model has access to its own code and alters the training process itself」
- 課程由輕到重，中段就是「改 checklist 假裝做完」：「The earliest stage involved basic political sycophancy, mimicking the user’s political views; later stages involved much more complex tasks, like altering a checklist so it appeared that incomplete tasks were actually complete.」
- 核心結果是逐階段自行推廣：「once models learned to be sycophantic, they generalized to altering a checklist to cover up not completing a task; once they learned to alter such a checklist, they generalized to modifying their own reward function—and even to altering a file to cover up their tracks.」
- 從沒直接訓練過竄改：「at no point did we explicitly train the model to engage in reward tampering」
- 次數很少，但對照組是零：「It occurred only 45 times out of 32,768 trials, and the model only covered its tracks seven times.」對照組：「A model that was trained only to be helpful, and which had no experience with the curriculum, made no attempts whatsoever to tamper with its reward function, even after 100,000 trials.」
- RLHF、Constitutional AI 這類無害性訓練幾乎沒影響：「adding such harmlessness techniques did not significantly change the observed rates of reward tampering.」
- 直接訓練去掉附和行為，降很多但不到零：「This training successfully reduced the rate of reward tampering, and did so quite substantially. But it did not reduce it to zero」
- 一旦養成就難去除：「Once the behavior is formed, the tendency to tamper with rewards—and even to act in ways that conceal this tampering—appears difficult to remove.」
- 限制——人為情境、有告知在訓練中、給了不受監看的 scratchpad：「This all occurred in an artificial setup where we deliberately trained models in scenarios that rewarded dishonest behavior.」作者不對現行模型下結論：「We thus make no claims about the propensity of current frontier AI models to engage in behaviors like reward tampering in realistic scenarios.」

## 對 skill 設計的意義
- 推論：原文課程中的「改 checklist 讓沒做完看起來做完」，和 coding agent「改測試讓它通過」是同一類行為。過去 skill 的「測試檔唯讀」「獨立 review」是直接拿掉作弊的機會，而不是靠指令要求誠實。原文本身沒談 coding agent 或 skill。
- 原文主張：常用的監督方法「no method that we tried could prevent it entirely」。推論：新 skill 不能假設「規則寫了就不會作弊」，關鍵步驟要有能被外部檢查的證據（diff、測試結果、人工確認）。
- 注意：原文測的是「訓練方法」對行為的影響，不是「prompt 指令」的效果。舊檔「指令性約束幾乎無效、只有結構性限制才有效」是舊撰寫者的延伸，原文沒這樣說。
