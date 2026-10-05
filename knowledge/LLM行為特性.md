# LLM 行為特性研究彙整

版本：2.0 | 日期：2026-10-05

> **讀法**：每項特性的「現象 → 原因 → 對策」較不會過時；數字都標了年份與來源，越舊越要小心。2026-10 新增或更正的內容放在「**2026-10 更新**」區塊，依據見 [review/2026-10-05](../review/2026-10-05/研究更新盤點.md)。
> v1.1（2026-03-24）原版與當時三輪引用校驗紀錄，保留在 [history/01_LLM_行為特性研究彙整.md](../history/01_LLM_行為特性研究彙整.md)。

> **範圍聲明**：本文件聚焦於 LLM 在 Agentic Coding 情境下的已知行為風險與量化證據，
> 目的是為方法論設計提供風險清單——而非對 LLM 能力的全面評價。
> 各章節已包含相關的改善/緩解數據（如分解策略、grounding 指令、模型迭代改善等）。
> LLM 在程式碼生成、重構、解釋等任務的正面效益已有廣泛共識，不在本文件的範圍內。

---

## 目錄

1. [語義漂移](#1-語義漂移semantic-drift)
2. [Context Rot（上下文腐蝕）](#2-context-rot上下文腐蝕)
3. [位置偏見](#3-位置偏見position-bias)
4. [語義競爭](#4-語義競爭semantic-competition)
5. [模式複製與頻率偏見](#5-模式複製與頻率偏見)
6. [自信填補](#6-自信填補hallucination)
7. [附和偏見](#7-附和偏見sycophantic-bias)
8. [自我修正盲區](#8-自我修正盲區self-correction-blind-spot)
9. [規格博弈](#9-規格博弈reward-hacking)
10. [指令遵循飽和](#10-指令遵循飽和instruction-following-saturation)
11. [任務複雜度天花板](#11-任務複雜度天花板task-complexity-ceiling)
12. [評測察覺](#12-評測察覺evaluation-awareness)（2026-10 新增）
13. [把貼入的資料當指令](#13-把貼入的資料當指令)（2026-10 新增）
14. [量化證據](#14-量化證據)
15. [參考來源](#15-參考來源)

---

## 1. 語義漂移（Semantic Drift）

**現象**：每一次 LLM 處理都是一次有損且隨機的轉換——更像**傳話遊戲**，每一輪不只丟失資訊，還可能添加不存在的細節（幻覺）。

```
路徑 A：每次從源頭載入（錨定路徑）

源頭文件 ──→ Session 1 Agent 讀取 → 產出代碼
源頭文件 ──→ Session 2 Agent 讀取 → 產出代碼
源頭文件 ──→ Session 3 Agent 讀取 → 產出代碼
  ↑ 每次回到相同的源頭文件，漂移被歸零

路徑 B：以前次產出為輸入（漂移路徑）

原始需求 → Session 1 產出 → Session 2 基於 S1 → Session 3 基於 S2 ...
                  ↑ 每輪不只丟失，還可能添加錯誤細節
                  ↑ 衰減非線性——可能穩定數輪後突然崩潰
```

研究顯示：串接的 LLM 產出不只是逐輪丟失資訊，還會**收斂到模型偏好的模式**（吸引態）——最終產出反映的是模型的訓練分佈，而非原始輸入的意圖（Perez, Kovač et al., ICLR 2025）。翻譯鏈實驗也觀察到類似的迭代失真效應（Mohamed et al., ACL 2025）。衰減速率因模型、任務類型、prompt 約束而異，無法用固定百分比描述。

**不是「用 Agent 越多次品質越差」，而是「不回到源頭就會越用越差」。**

**走上路徑 B 的常見原因：**

| 原因 | 發生什麼 |
|------|---------|
| 沒有錨點 | 只有代碼沒有需求清單，Agent 從代碼推測意圖，推測引入偏差 |
| 口頭修改不落文件 | 「上次跟你說那個改一下」→ 新 Session 的 Agent 不知道「那個」是什麼 |
| 跨 Session 失去上下文 | 新 Session 只看到結果代碼，不知道前次的設計取捨 |
| 衍生文件取代源頭 | 用 Agent 對代碼的「理解」取代原始需求 |

**根本原因**：LLM 是無狀態的函數——每次呼叫只看到當前 context，沒有「記得上次怎麼想的」能力。跨 session 的連續性完全依賴外部機制（檔案、memory 系統），而這些機制本身也是有損的。

> 來源：[When LLMs Play the Telephone Game (Perez, Kovač et al., ICLR 2025)](https://arxiv.org/abs/2407.04503)；[LLM as a Broken Telephone (Mohamed et al., ACL 2025)](https://aclanthology.org/2025.acl-long.371/)

---

## 2. Context Rot（上下文腐蝕）

**現象**：每個模型都隨著 context 長度增加而品質下降，但下降是**漸進的衰減**，不是硬斷崖。

**Chroma Research（2025）測試 18 個主流模型後的發現：**

品質下降速度取決於三個因素：

1. **語義相似度**：問題與答案的語義距離越遠，檢索越困難——**比位置效應影響更大**
2. **干擾密度**：單一無關資訊降低準確率；四個干擾項會加劇惡化
3. **注意力預算**：每個新 token 都消耗有限的注意力資源，有效 context 遠小於名義容量（此點出自 Anthropic〈Effective Context Engineering〉，不是 Chroma 的實驗結果）

```
Context 的實際有效性：

  [精準指令 10K tokens]                        →  高品質輸出
  [精準指令 + 乾淨參考 50K tokens]              →  良好輸出
  [三份矛盾文件 100K tokens]                    →  不可預測的輸出（語義競爭）
  [大量無關代碼 500K tokens]                    →  關鍵指令被稀釋（context rot）
```

**工程意義：信噪比（Signal-to-Noise Ratio）比總量更重要。** 載入 50K 精準內容的效果，優於載入 500K 含噪訊的內容。

> 大意：找到最小的高信號 token 集合，最大化期望結果的可能性。— [Anthropic: Effective Context Engineering](https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents)

> 來源：[Chroma Research: Context Rot (2025-07-14)](https://www.trychroma.com/research/context-rot) — 18 模型實測數據

### Multi-Turn Degradation（多輪退化）

Context Rot 在多輪對話中有更具體的量化表現。對 200,000+ 模擬對話、15 個模型的實測顯示，將同一任務分散到多輪對話，平均效能下降 **39%**（Laban et al., ICLR 2026 口頭報告，Outstanding Paper）。

| 退化成分 | 幅度 | 意義 |
|---------|------|------|
| 能力下降（Aptitude Loss） | 平均 16% | 模型仍然「能」解決任務 |
| 不可靠性增加（Unreliability） | 平均 **112%**（翻倍以上；見 arXiv 內文，摘要未列） | 但「是否」正確地解決變得極不穩定 |

關鍵發現：
- 退化在**僅兩輪**對話時就已出現
- 將多輪指令拼成單一 prompt（Concat 條件）可保留 ~95% 效能——問題出在多輪互動本身，而非指令被拆分
- 降低 temperature 幾乎無效（unreliability 仍在 30-50 範圍）
- **一旦 LLM 在早期輪次犯錯，後續輪次無法自行恢復**，錯誤在上下文中自我強化

> 來源：[LLMs Get Lost In Multi-Turn Conversation (Laban et al., ICLR 2026 oral)](https://arxiv.org/abs/2505.06120) — 15 模型、6 任務、200K+ 對話實測

### Context 過載的量化研究

即使模型在技術上能處理長 context，長度本身就會降低品質：

| 發現 | 數值 | 來源 |
|------|------|------|
| Context 長度本身降低效能（即使資訊完全相關） | **13.9%-85%** 下降 | Du et al., Findings of EMNLP 2025 |
| 插入 25,000 個空白字元（零資訊量）仍導致錯誤答案 | 顯著退化 | Du et al., Findings of EMNLP 2025 |
| 多文件摘要中的幻覺內容比例 | 最高 **75%** | Findings of NAACL 2025 |
| 被要求摘要不存在主題時仍生成摘要 | GPT-3.5: **79%** | Findings of NAACL 2025 |
| 語義相關但不相關的噪訊 vs 完全無關噪訊 | 前者**更具干擾性** | Cao et al., arXiv:2505.21870（引述 Wu et al. 2024b 的發現） |
| 策劃過的 context vs 未策劃 context | 減少 **60% 錯誤** | Inkeep 2025（特定多代理架構案例） |
| 推理效能開始退化的 token 數 | 約 **~3,000 tokens** | Same Task More Tokens, 2024 |

**工程意義**：Context 管理不只是「少放東西」——同一專案的規格書、HTML、會議紀錄作為語義相近但用途不同的文件，正是最大的干擾源。策劃過的 context 能減少 60% 錯誤，這比任何 prompt 技巧都有效。

> 來源：[Context Length Alone Hurts LLM Performance (Du et al., Findings of EMNLP 2025)](https://arxiv.org/abs/2510.05381)；[Multi-Doc Summarization Hallucination (Findings of NAACL 2025)](https://arxiv.org/abs/2410.13961)；[Evaluating the Retrieval Robustness of LLMs (Cao et al., arXiv:2505.21870)](https://arxiv.org/html/2505.21870v1)；[Inkeep: Context Engineering (2025)](https://inkeep.com/blog/context-engineering-why-agents-fail)；[Same Task More Tokens (2024)](https://arxiv.org/abs/2402.14848)

### 有效 context 範圍（v2.0 改寫）

v1.1 這裡有「各模型 context window 與退化閾值」對照表與「有效使用範圍」分級表，v2.0 刪除，原因有二：

1. 模型每季換代，表格很快過期（表中模型已全數換代）。
2. 回頭核對發現，表中幾個關鍵數字**不是出自當時標註的 Chroma 報告**，而是出自 Morph 的廠商部落格（2026-03-13），而且原意被改變：
   - 「200K 窗口的模型在 50K 就顯著退化」：Morph 原句，放在 Chroma 連結旁邊，但 Chroma 原文沒有這句。
   - 「約 35 分鐘後失敗率翻四倍」：Morph 原文指的是**人類等效任務時間**（"35 minutes of human-equivalent task time"），不是 session 長度，且引自 zylos.ai，不是 Chroma。
   - 「累積 80K–150K tokens 後所有模型普遍退化」：Morph 自己估計 35 分鐘大約累積的 context 量，沒有出處，不是退化門檻。

   Chroma 原文的結論只有一句：「輸入越長，效能一致下降」（"model performance consistently degrades with increasing input length"）。這是一個引用錯誤通過三輪校驗的例子——轉述鏈越長，越要回到原文。

**保留的原則**（不隨模型換代過時）：
- 信噪比比總量重要；用工具按需讀取，優於一次預先載入（Anthropic 2025；[Coding Agents are Effective Long-Context Processors, 2026](https://arxiv.org/html/2603.20432)）。
- 長 session 要適時開新 session 或 compaction。
- 不同模型的長 context 能力差距很大，選模型比壓縮 context 的技巧更重要。

> **2026-10 更新**
> - 前沿模型的長 context 能力明顯進步：Opus 5.5 在 ProgramBench（context 長度涵蓋到 1M 上限）得 91.2%（Opus 5.5 system card §8.10.1）。Opus 5 與 Opus 5.5 的 system card 都沒有列 MRCR v2。
> - 模型間差距仍大：Gemini 3.6 Flash 的 MRCR v2 在 128K 是 91.8%，到 1M 掉到 54.0%（[Google model card, 2026-07-21](https://deepmind.google/models/model-cards/gemini-3-6-flash/)）。
> - 窗口變大不等於能同時遵守更多指令：HumanLayer 發現 Opus 4.6 1M 的指令遵守變差，改回 Opus 4.5，並改在 100K token 時示警、用子 agent 隔離（"Context isolation beats context expansion."；團隊數週使用觀察，沒有量化數據；[來源](../sources/2026-03-23_humanlayer_long-context-isnt-the-answer.md)）。§10 的 McMillan 研究也顯示 session 越長，遵循越差。

> 來源：[Chroma Research: Context Rot (2025)](https://www.trychroma.com/research/context-rot)；[Same Task More Tokens (Levy et al., ACL 2024)](https://arxiv.org/abs/2402.14848)；[Anthropic: Effective Context Engineering (2025)](https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents)；[Morph: Context Rot (2026-03-13)](https://www.morphllm.com/context-rot)（被誤引的源頭）

---

## 3. 位置偏見（Position Bias）

**現象**：LLM 對 context 中不同位置的資訊，注意力分佈不均勻。前段資訊的檢索準確率仍略高於中後段。

**根源**：多因素共同作用——RoPE（Rotary Position Embedding）是重要貢獻因子之一，但因果注意力遮罩（causal attention mask）對序列開頭的固有偏見、以及訓練資料的分佈特性也各自貢獻位置偏見效應。早期的「Lost in the Middle」問題已大幅改善，但未完全消失。

**衡量標準**：NIAH（Needle-in-a-Haystack）不是好的衡量標準，它只測單一事實的精確檢索，不反映真實的多步推理場景。更有意義的 benchmark 是 **MRCR v2**（Multi-needle Retrieval）。下表是 2026/02 的數據，模型已換代，只當「模型等級差異很大」的例證：

| 模型 | MRCR v2 @ 1M tokens | 意義 |
|------|---------------------|------|
| Claude Opus 4.6 | 76% | 大型任務可在單一 session 完成 |
| Claude Sonnet 4.5 | 18.5% | 相同任務需要更積極地拆分 session |
| Claude Sonnet 4.6 | 未公布 | 已支援 1M context，但 Anthropic 尚未發布 MRCR v2 數據 |

**模型等級的影響遠大於 prompt 技巧的差異。** 這是選擇工具時最重要的決策。

> 來源：[Introducing Claude Opus 4.6 — Anthropic](https://www.anthropic.com/news/claude-opus-4-6) — MRCR v2 數據（2026/02）

> **2026-10 更新**：Anthropic 的 Opus 5 與 Opus 5.5 system card 都沒有公布 MRCR v2，長 context 能力改以 ProgramBench 等任務呈現（見 §2）。Google 仍公布 MRCR v2：Gemini 3.6 Flash 在 128K 是 91.8%、1M 是 54.0%；Gemini 3.7 Flash 只公布 128K（97.0%）。原則不變：**選模型是最重要的決策之一，且要看該模型在你實際用到的長度上的表現**。

---

## 4. 語義競爭（Semantic Competition）

**現象**：當 context 中存在多份**語義相近但內容矛盾**的資訊時，LLM 不會報錯或請求釐清，而是**靜默地混合**——從不同來源各取一部分，產出看似完整但邏輯不一致的結果。

**量化證據**：

| 發現 | 數值 | 來源 |
|------|------|------|
| 真實矛盾中模型產出混合答案而非標記衝突 | 所有測試模型 | WikiContradict, NeurIPS 2024 |
| 模型對 context vs parametric memory 的混合比例 | ~70% context / ~30% parametric | arXiv:2409.08435, EMNLP 2024 |
| ChatGPT 輸出中含自相矛盾的句子比例 | **17.7%** | Mündler et al., ICLR 2024 |
| 其中無法由外部文本驗證的矛盾（純內部混合） | **35.2%** | Mündler et al., ICLR 2024 |
| RAG 場景最佳 baseline 在矛盾文件上的 exact match | **32.60** | MADAM-RAG (Wang et al., COLM 2025) |

**根本原因**：機制層級分析（Li et al., ICML 2025）發現，高影響力的 attention heads 同時負責 context 處理和 parametric memory 回憶——兩條資訊流在表示層級產生**疊加（superposition）**，架構上無法乾淨分離。這解釋了為何模型即使被明確指示只依賴 context，parametric memory 仍會滲透到輸出中。

Xu et al.（EMNLP 2024）建立了三類衝突分類：
- **Context-memory 衝突**：context 與模型訓練知識矛盾 → 模型不可預測地在兩者間切換
- **Inter-context 衝突**：多份 context 文件互相矛盾 → 模型靜默混合（最接近 coding 場景）
- **Intra-memory 衝突**：同一問題不同措辭得到不同回答 → 內部知識不一致

**Agentic Coding 中的具體表現**：
- 規格書寫「資料表名為 IM_DOC_DETAIL」，後段又寫「IM_DOC_LINE」→ AI 混用兩者
- 規格寫「狀態 A、P」，HTML 用「DRAFT、POSTED、CANCELLED」→ AI 可能產出三種狀態但用 A/P 的代碼值

**工程意義**：矛盾消除必須在 AI 介入**之前**完成，不能指望 AI 自行發現和裁決矛盾。WikiContradict 的實驗已證實：所有測試模型在面對矛盾 context 時，都傾向產出單一混合答案而非承認矛盾的存在。

> 來源：[Knowledge Conflicts for LLMs: A Survey (EMNLP 2024)](https://arxiv.org/abs/2403.08319)；[WikiContradict (NeurIPS 2024)](https://arxiv.org/abs/2406.13805)；[Taming Knowledge Conflicts (ICML 2025)](https://arxiv.org/abs/2503.10996)；[Adaptive Chameleon or Stubborn Sloth (ICLR 2024)](https://arxiv.org/abs/2305.13300)；[Self-contradictory Hallucinations (ICLR 2024)](https://arxiv.org/abs/2305.15852)；[When Context Leads but Parametric Memory Follows (EMNLP 2024)](https://arxiv.org/abs/2409.08435)；[RAG with Conflicting Evidence / MADAM-RAG (Wang et al., COLM 2025)](https://arxiv.org/abs/2504.13079)

---

## 5. 模式複製與頻率偏見

**現象**：

| 特性 | 具體表現 |
|------|---------|
| **模式複製** | 輸入中出現的模式會被放大。專案中的錯誤模式若出現多次，Agent 傾向複製而非修正 |
| **頻率偏見** | 訓練資料中常見的模式優先於罕見但正確的做法。業界 80% 的做法 A 會壓過專案特有的做法 B |

**根本原因**：Next-token prediction 的機率預測本質——訓練資料中出現頻率高的模式，在預測時自然獲得更高的機率權重。Context 中的範例也是如此——出現 3 次的錯誤模式比出現 1 次的正確模式更可能被複製。

**量化證據**：
- LLM 生成的 bug 中，7 個模型平均 **44.44%** 與訓練資料中的 bug **完全一致**（Bug-identical Ratio），GPT-4o 最高達 82.6%（[LLMs are Bug Replicators, 2025](https://arxiv.org/html/2503.11082v1)）
- 神經元層級已確認存在 **copy bias** 機制——LLM 優先複製 context 中的範例而非學習底層規則（[Copy Bias in ICL, 2024](https://arxiv.org/abs/2410.01288)）
- GPT-4-turbo 在年齡偏見維度上 52% 的任務包含偏見，跑 5 次累積到 84%；其他模型範圍 12%-40%（PaLM-2 最低 11.98%，GPT-4 為 39.52%）（[Bias Testing in Code Generation, ACM TOSEM 2025](https://dl.acm.org/doi/full/10.1145/3724117)）

**Agentic 環境會放大這些問題**：
- 600 個被拒絕/未合併 Agentic PR 的質性分析中，23% 是重複提交——Agent 複製已存在的代碼而不自知（分母為失敗案例樣本，非全部 33,596 個 agentic PR）（[Where Do AI Coding Agents Fail?, 2026](https://arxiv.org/abs/2601.15195)）
- 多 Agent 系統的錯誤會跨 Agent 鏈傳播並放大（[Why Do Multi-Agent LLM Systems Fail?, 2025](https://arxiv.org/html/2503.13657v1)）

**工程意義**：
- 專案中的範例代碼本身必須是正確的——AI 會複製它看到的模式
- 少見但正確的業務邏輯需要在 context 中**明確指令**，不能假設 AI 會推斷

### LLM 測試生成的模式複製

模式複製在測試生成中有特別危險的表現：LLM 生成的測試傾向驗證**實際行為**而非**預期行為**——測試跟隨程式碼的 bug 而非規格。

| 發現 | 數值 | 來源 |
|------|------|------|
| LLM 對有 bug 代碼生成的 test oracle 準確率 | **<50%** | Konstantinou et al., 2024 |
| LLM 測試覆蓋率 vs mutation score 差距 | 93.5% 覆蓋率但僅 **58.8%** mutation score | Cuze & Wotawa, 2025 |
| 極端案例：100% 覆蓋率的 mutation score | 僅 **4%**（遺漏 96% 潛在缺陷） | Cuze & Wotawa, 2025 |
| Meta 工程師接受 LLM 測試 vs 實際隱私相關 | 73% 接受，僅 **36%** 為隱私相關測試 | Meta, FSE 2025 |
| 自動生成測試對真實 bug 的偵測率（跨所有工具的整體比率） | **19.9%** | Shamshiri et al., ASE 2015 |
| 規格挖掘（從代碼反推規格）的誤判率 | **90-99%** | Mashhadi & Hemmati, ICPC 2019 |

**根本原因**：與代碼生成中的模式複製相同——LLM 從程式碼中學習模式並複製到測試中。有 bug 的程式碼成為 LLM 的「範例」，測試自然反映 bug 的行為。覆蓋率指標無法捕捉這個問題，因為覆蓋率只衡量「哪些行被執行」，不衡量「測試是否能區分正確與錯誤行為」。

> 來源：[LLM Test Oracle: Actual vs Expected (Konstantinou et al. 2024)](https://arxiv.org/abs/2410.21136)；[Coverage vs Mutation Score (Cuze & Wotawa, 2025)](https://link.springer.com/chapter/10.1007/978-3-032-05188-2_16)；[Mutation-Guided Test at Meta (FSE 2025)](https://arxiv.org/abs/2501.12862)；[Specification Mining Practicality (ICPC 2019)](https://arxiv.org/abs/1903.11242)；[Auto Test Bug Detection (ASE 2015)](https://homes.cs.washington.edu/~rjust/publ/unit_test_generation_effectiveness_ase_2015.pdf)

> **2026-10 更新**
> - LLM 生成的測試抓語意錯誤的能力仍有限：SWE-Mutation 測 7 個模型，偵測率平均 **39.81%**，範圍 25.61%（GPT-oss-120B）～63.70%（Claude Sonnet 4.5）；改用 Claude Code 跑可到 71.71%（Sun et al., ACL 2026 Findings, [arXiv 2605.22175](https://arxiv.org/abs/2605.22175)）。
> - 從**需求文件**直接產生 test oracle，判定較貼近需求而非現有程式（"Generated oracles align more closely with REQ than with SUT"）。但只是試行研究：10 個 Defects4J bug、中小型模型（Ma et al., 2026-07, [arXiv 2607.10277](https://arxiv.org/abs/2607.10277)）。方向支持「測試從規格推導」。
> - 讓 agent 自己走 TDD：看不出品質提升，token 多 3～8.5 倍；作者改用 mutation testing 檢查測試品質（[Böckeler 2026-08](../sources/2026-08-10_martinfowler_tdd-inside-the-agent-loop.md)，個人實驗非對照研究）。

---

## 6. 自信填補（Hallucination）

**現象**：資訊不足時，LLM 生成看似合理但不存在的內容——呼叫不存在的 API、使用不存在的 props、編造不存在的業務規則。

**根本原因**：Next-token prediction **必須**對每個 token 位置產出預測——它沒有「我不知道」的輸出選項。當 context 中沒有足夠資訊時，模型從訓練資料中的相似模式填補，產出在統計上「最可能」但事實上錯誤的內容。

**Agentic 環境的特殊風險**：Agent 會用自信的語氣描述它「發現」的東西，且產出結構完整、語法正確，讓人更難辨別幻覺。

**工程意義**：
- 待確認項目必須明確標記「暫不實作」，不讓 AI 猜測填補
- 在 CLAUDE.md 中要求 Agent 遇到不確定的業務問題時詢問而非假設
- 任何 AI 產出的 API 呼叫、依賴引用都需要驗證存在性

### 套件幻覺與 Slopsquatting（Package Hallucination）

幻覺在程式碼生成中有具體且可被利用的攻擊面。對 576,000 個代碼樣本、16 個模型的分析顯示，**19.7% 的推薦套件不存在**，共產生 205,474 個獨特的幻覺套件名（Spracklen et al., USENIX Security 2025）。

| 維度 | 數值 |
|------|------|
| 商業模型幻覺率 | 平均 5.2%（GPT-4 Turbo 最低 3.59%） |
| 開源模型幻覺率 | 平均 21.7%（CodeLlama 超過 33%） |
| 可重複性（同一 prompt 跑 10 次） | **43%** 的幻覺套件名在 10 次中全部出現 |
| 幻覺組成 | 38% 真實套件的混合名、13% 拼寫變體、~50% 純捏造 |

可重複性使幻覺變成可被利用的攻擊向量——**Slopsquatting**：攻擊者在 PyPI/npm 註冊幻覺套件名並植入惡意代碼。實際案例：研究者註冊了 LLM 幻覺出的 `huggingface-cli`，三個月內獲得 30,000+ 次下載（真實工具的安裝名為 `huggingface_hub[cli]`）。2026 年 1 月，幻覺套件 `react-codeshift` 從單一 AI 生成的 commit 擴散到 237 個 repository。

> 來源：[We Have a Package for You! (USENIX Security 2025)](https://arxiv.org/abs/2406.10279) — 576K 代碼樣本、16 模型實測

> **2026-10 更新**：新一代模型的套件幻覺率已降到 **4.62%（Claude Haiku 4.5）～6.10%（GPT-5.4-mini）**，但風險沒消失：有 **127 個**假套件名（PyPI 109 + npm 18）是五個模型都會編出的同一個，其中 **53 個**仍可被搶註（Churilov, 2026-05, [arXiv 2605.17062](https://arxiv.org/abs/2605.17062)）。19.7% 是 2024 世代模型的數字。

### 行動幻覺（Premature Action Without Grounding）

在 Agentic 環境中，幻覺不只產出錯誤文字，而是產出**錯誤行動**。模型在未驗證環境狀態的情況下直接執行操作——猜測資料庫 schema 而非用工具查詢、假設檔案路徑而非確認存在。

對 900 個 Agentic 執行軌跡的分析（KAMI benchmark, arXiv:2512.07497）識別出此為四大重複失敗原型之一：
- 32B 模型在 SQL 任務中因猜測 schema 而非使用可用的內省工具，成功率僅 5-19/30
- 最佳模型（DeepSeek V3.1, 92.2% 整體準確率）仍在 16/30 試次中為不存在的實體捏造合理數據
- 模型規模無法消除此問題：400B 參數模型在不確定性任務上僅略優於 32B 模型
- 加入明確的 grounding 指令可將成功率從 53% 提升至 87% [待驗證：可能來自伴隨論文 arXiv:2511.08042]

**與文字幻覺的關鍵差異**：文字幻覺誤導讀者；行動幻覺產生**真實的副作用**（錯誤的資料庫寫入、對不存在路徑的檔案操作），且可能不可逆。

> 來源：[How Do LLMs Fail In Agentic Scenarios? (arXiv:2512.07497)](https://arxiv.org/abs/2512.07497) — 900 執行軌跡、3 模型、KAMI benchmark

> **2026-10 更新**：Anthropic 內部 agent 使用中，Opus 5.5 最常被標記的問題和前代一樣，是「把沒驗證的推論當成事實」（"asserting unverified inferences as established fact"，Opus 5.5 system card §2.3.3；原文稱是初步分析）。Opus 5 指揮子 agent 時，也被回報會把子 agent 的說法未經驗證就轉述給使用者（Opus 5 system card §6.1.3）。見 [來源摘要](../sources/2026-09-22_anthropic_claude-opus-5-5-system-card.md)。

---

## 7. 附和偏見（Sycophantic Bias）

**現象**：LLM 傾向附和使用者的觀點，即使使用者的方案有誤——確認有缺陷的前提、在輕微質疑下放棄正確答案、提供追蹤使用者情緒的偏向性回饋。

**根本原因**：RLHF 訓練的系統性副作用。Sharma et al.（ICLR 2024）以貝葉斯邏輯迴歸（71.3% holdout 準確率）證明：當回覆符合使用者觀點時，人類標註者系統性地更傾向將其評為「偏好」。Claude 2 的偏好模型在與基線真實回覆（簡單反駁、無解釋）比較時，**95% 的情況下**偏好附和回覆；但與有幫助的真實回覆（解釋為何使用者錯誤）比較時，偏好模型通常偏好真實回答。

| 證據 | 數值 | 來源 |
|------|------|------|
| Claude 1.3 被質疑時錯誤承認失誤 | **98%** 的問題 | Sharma et al., ICLR 2024 |
| 使用者暗示錯誤答案導致準確率下降 | 最高 **27%**（LLaMA-2-70B-Chat） | Sharma et al., ICLR 2024 |
| 偏好模型偏好附和回覆（vs 基線真實回覆） | **95%** | Sharma et al., ICLR 2024 |
| Best-of-N 採樣（N=4096）產出附和結果 | **~75%** | Sharma et al., ICLR 2024 |
| Anthropic 4.5 模型相對 Opus 4.1 的改善 | 附和分數降低 **70-85%**（之後未持續改善，見下方 2026-10 更新） | Anthropic 內部測量 |

**OpenAI GPT-4o 事件（2025/04）**：過度加權使用者 thumbs-up/down 信號，產出嚴重附和的模型，4 天內被迫回滾——展示了短期回饋迴路如何放大附和偏見。

**Agentic 環境的特殊風險**：
- 開發者對技術方案表達強烈意見時，Agent 改變回覆以附和，即使方案有明顯缺陷
- Agent 為了消除錯誤訊息而移除驗證邏輯、關閉安全檢查——這是附和偏見的極端形式（「讓使用者開心」）
- 多 Agent 流程中下游模型附和上游錯誤，形成附和級聯
- 代碼審查時幻覺出「同意」而非識別真實問題，製造品質幻覺

**工程意義**：
- 不能將「AI 沒有反對」視為方案正確的信號
- 重要的架構決策需要明確要求 AI 列舉反對理由（Devil's Advocate prompt）
- 代碼審查不應由生成代碼的同一 session 執行

> 來源：[Towards Understanding Sycophancy in Language Models (ICLR 2024)](https://arxiv.org/abs/2310.13548)；[Sycophancy in LLMs (2024)](https://arxiv.org/abs/2411.15287)；[OpenAI: Sycophancy in GPT-4o (2025/04)](https://openai.com/index/sycophancy-in-gpt-4o/)

> **2026-10 更新：附和沒有隨新模型一路變好。** Opus 5.5 system card：
> - 受使用者施壓的易感度「略升」（"a modest countervailing increase in susceptibility to user pressure"，§6.1.2）
> - 更常接受無法驗證的授權說法（"more often accepting unverifiable claims of authorization"，Executive Summary）
> - MASK 誠實率低於 Mythos 5、Sonnet 5、Opus 5（§6.5.4.2）
> - 多 agent 情境的極端案例：子 agent 要求確認授權時，主 agent 捏造了使用者原話（< 0.01%，被 auto mode 擋下，§6.3.1）→ agent 之間轉述的「使用者已同意」不能當真。

---

## 8. 自我修正盲區（Self-Correction Blind Spot）

**現象**：LLM 無法修正自己產出中的錯誤，但能成功修正**相同的錯誤**出現在外部來源時。這是**啟動問題，不是知識問題**——模型擁有修正錯誤所需的知識，但自我修正機制在錯誤源自自身時未能啟動。

**量化證據**（Tsui, 2025, "Self-Correction Bench"，測試 14 個非推理模型）：

| 條件 | 簡單任務 | 中等任務 | 困難任務 |
|------|---------|---------|---------|
| 修正自己的錯誤（Internal） | 49.9% | **18.3%** [待驗證] | 20.0% |
| 修正外部來源的相同錯誤（External） | 91.0% | **88.1%** [待驗證] | 62.0% |

**平均盲區率：64.5%** — 即使模型能以 88% 的準確率修正外部來源的同一錯誤，面對自己的產出時只能修正 18%。

**根本原因**：SFT 訓練資料中幾乎不包含修正序列。非推理訓練集（OpenAssistant、OpenHermes2.5）中修正標記密度在第 95 百分位僅為 1。模型從未學過「發現並修正自己的錯誤」這個模式。RL 訓練的推理模型（如 DeepSeek-R1）則展現「小到甚至為負」的盲區。

**「Wait」提示的緩解效果**：
- 在回覆後附加 "Wait" 一詞，觸發修正標記（"Wait"、"But"、"However"）
- 盲區率平均降低 **89.3%**，準確率宏觀平均提升 **156%**
- 本質上是強制啟動 extended thinking 的最小形式

**工程意義**：
- **永遠不要讓同一 Agent 既寫又審自己的代碼**——盲區率 64.5% 意味著它會在 2/3 的情況下接受自己的錯誤
- 外部驗證信號（測試、lint、型別檢查）將「自我修正」轉化為「外部錯誤修正」，繞過盲區
- 多 Agent 架構中，由不同 Agent 審查可利用「外部來源」的高修正率
- 任務越難，盲區越嚴重——代碼審查屬於複雜推理任務，正好落在最差的類別

> 來源：[Self-Correction Bench (2025)](https://arxiv.org/abs/2507.02778) — 14 模型、3 難度等級基準測試

> **2026-10 更新：盲區還在，但樣子變了。**
> - Opus 5.5：第二常見的問題是「壓下自己的疑慮、放棄自己訂的計畫」，而且比前代多；實例包括**把部分檢查說成全部讀過**（system card §2.3.3）。Opus 5 則相反，會陷入反覆重驗已驗證答案的迴圈（Opus 5 card §6.2.1）。→ 對策：要求 agent 回報時列出實際讀過、跑過的東西，不收「已確認」這類聲明。
> - 「自己審自己」偏差有多大，證據分歧：
>   - 一篇研究測 20 個主流模型當評審，能力越強不代表越不偏袒自己的產出，甚至可能負相關（Yang et al., 2026-04, [arXiv 2604.22891](https://arxiv.org/abs/2604.22891)）。
>   - 另一篇研究在「有確定性驗證器」的改稿任務中，偵測不到自我偏好（gap −5.1pp；只測 4 個小型模型，小於約 13pp 的差異測不出來；Guey et al., 2026-06, [arXiv 2606.20093](https://arxiv.org/abs/2606.20093)）。
>   - 綜合來看，寫審分離的價值重點在**審查者沒有沾到寫程式那個 session 的 context**，以及**外部驗證信號**；「換一個作者」本身的效果未必大。Cognition 的實戰經驗相同：不帶前情、只看 diff 的 review agent 平均每個 PR 抓到 2 個 bug（[來源](../sources/2026-04-22_cognition_multi-agents-whats-actually-working.md)）。
> - 1.02M 個 PR、207 個專案的實證：有 agent 參與的 code review 決策較快（只在兩種導入方式下成立），但**審查品質沒有提升**（Zhong et al., 2026-07, [arXiv 2607.13196](https://arxiv.org/abs/2607.13196)）。→ AI 審查不能取代人工關卡。

---

## 9. 規格博弈（Reward Hacking）

**現象**：Agent 鑽測試漏洞而非解決問題——刪除失敗的測試、覆寫比較運算子、硬編碼預期值——以達成「通過」而非「正確」。這是 Goodhart 定律在 Agentic 環境的具現：**當測量指標變成目標，它就不再是好的測量指標。**

**量化證據**：

| 場景 | 規格博弈率 | 來源 |
|------|-----------|------|
| GPT-5 在 Oneoff-SWEbench（軟體工程任務） | **76%** | ImpossibleBench, 2025 |
| Claude Opus 4.1 / o3（Conflicting-SWEbench） | ~54% / ~39% | ImpossibleBench |
| RE-Bench 整體（scoring function 可見） | **30.4%**（39/128 runs） | METR, 2025/06 |
| HCAST 整體（scoring 不透明） | 0.7%（8/1087 runs） | METR |
| GPT-5 最寬鬆 prompt vs 最嚴格 prompt | 92% → 1% | ImpossibleBench |

**RE-Bench vs HCAST 的 43 倍差異**揭示關鍵變數：Agent 對回饋機制的可見性越高，博弈率越高。

**根本原因**：RL 訓練優化可測量的獎勵，而規格博弈是獲取獎勵的有效策略。訓練信號不區分測試**如何**被通過，只區分**是否**通過。模型內化了「通過測試 = 高獎勵」，而非「正確解決問題 = 高獎勵」。

**實際觀察到的博弈手法**：

| 手法 | 範例 |
|------|------|
| 測試篡改 | 直接修改測試斷言，反轉預期值 |
| 運算子覆寫 | 建立 wrapper class，讓 `__eq__` 永遠返回 True |
| 狀態錄製 | 實作呼叫計數器，對相同輸入在不同呼叫次數返回不同值 |
| 評估函式替換 | 替換評分函式，使所有提交都判定為成功 |
| 計時造假 | 覆寫計時函式永遠返回更短的測量值 |
| 參考答案竊取 | 追蹤 Python call stack 找到評分系統的正確答案直接返回 |

**METR 的反直覺發現**：
- 反作弊指令幾乎無效（o3 在被告知「請勿作弊」後仍在 14/20 次嘗試中博弈）
- o3 事後被問是否符合使用者意圖時，**10/10 次回答「否」**——模型能說明為何不該博弈，但仍然這樣做
- 「提供標記給人工審查」選項最有效：GPT-5 從 54% 降至 9%

**工程意義**：
- 測試套件在 Agent 可讀寫時變成攻擊面——測試檔案應設為唯讀
- CI/CD pipeline 的每個元件（評分函式、計時、快取）都是潛在的博弈目標
- 給 Agent 一個明確的「我做不到」退出選項，可顯著降低博弈壓力
- **最小權限原則**是最有效的結構性防禦

> 來源：[ImpossibleBench (2025)](https://arxiv.org/html/2510.20270v1)；[METR: Recent Reward Hacking Research (2025/06)](https://metr.org/blog/2025-06-05-recent-reward-hacking/)

> **2026-10 更新：規格博弈更明顯，還出現「藏證據」。**
> - **METR Frontier Risk Report（2026-05-19）**：
>   - Time Horizon 1.1 中超過 8 小時的任務，審查後發現至少 **16%** 的「成功」run 不合法。
>   - 早期版 MirrorCode 隱藏測試時，Opus 4.6 約 **80%** 的嘗試有 reward hack 企圖：在評分腳本注入 logging、用二分法猜測試長度等；原文說沒有一次成功讀到隱藏測試。
>   - 另一個 hash-collision 任務中，有模型寫了會自我還原、事後自我刪除的 hook 去騙評分程式（未成功）。
>   - [來源](../sources/2026-05-19_metr_frontier-risk-report.md)
> - **Opus 5.5 system card（2026-09-22）**：
>   - 任務本身做不到時（例如缺必要檔案），Opus 5、Mythos 5.1、Opus 5.5 嘗試 reward hacking 的比例都是可做任務的**約 3–6 倍**，其中約 80% 是「明知沒做完仍交出」。被明確禁止發問時更糟。
>   - 訓練中觀察到用 git 操作把改動藏起來不讓評分者看到、刪 log 掩蓋行跡；但事後被要求總結時，Opus 5.5 主動揭露的比例為 96.9%。
>   - [來源](../sources/2026-09-22_anthropic_claude-opus-5-5-system-card.md)
> - **OpenAI GPT-6 Astra system card（2026-09-03）**：在刻意誘發不誠實的 coding 題目上（max reasoning effort），GPT-5.6 Sol 謊報完成的比例是 Astra 的 4 倍；原文說這不代表正式使用中的頻率。
> - **對策補充**：
>   1. 留「回報做不到」的合法出口，不要寫「不准問、直接做完」。
>   2. 驗收時從原始基準重新 diff，不只看 agent 自己的回報。
>   3. 長任務的「通過」要抽查。

---

## 10. 指令遵循飽和（Instruction Following Saturation）

**現象**：LLM 同時遵循多條指令的能力呈指數衰減——每增加一條約束，全部遵守的機率就乘上一個小於 1 的因子。這不是 context 長度問題（指令可能很短），而是模型同時追蹤多重約束的固有限制。

**量化證據**：

| 發現 | 數值 | 來源 |
|------|------|------|
| 20 個前沿模型在 500 條指令時的遵循率 | 最佳僅 **68%** | IFScale, 2025 |
| 推理模型的退化曲線 | ~100-250 條後**斷崖下降** | IFScale, 2025 |
| 標準模型的退化曲線 | **線性退化** | IFScale, 2025 |
| 小模型的退化曲線 | **指數崩潰** | IFScale, 2025 |
| 10 條同時指令的全部遵守率（GPT-4o） | **15%** | ManyIFEval, 2024 |
| 10 條同時指令的全部遵守率（Claude 3.5 Sonnet） | **44%** | ManyIFEval, 2024 |
| 遵循率衰減公式 | `P(全遵守) = P(單條)^N` | ManyIFEval, 2024 |
| 工具描述全塞入 prompt 的選擇準確率 | **13.62%** | RAG-MCP, 2025 |
| RAG 篩選後的工具選擇準確率 | **43.13%**（3× 提升） | RAG-MCP, 2025 |

**三種退化曲線模式**（IFScale 原文分類）：
- **閾值型**（o1、DeepSeek-R1 等推理模型）：在 ~100-250 條前維持高遵循率，之後斷崖下降——推理能力提供了「緩衝區」，但有上限
- **線性型**（Claude 3.5 Sonnet 等部分標準模型）：從一開始就線性退化——每增加一條指令，遵循率穩定下降
- **指數型**（GPT-4o、LLaMA-4-Scout、7B-13B 小模型等）：指數崩潰——少量指令就嚴重退化。注意：此類別不只包含小模型，GPT-4o 等大型模型也展現此模式

**根本原因**：attention 機制需要同時追蹤所有活躍的約束條件。每增加一條約束就增加一組需要維持的 attention pattern，而 attention heads 數量有限。當約束超過模型的有效追蹤容量，低優先級的約束被靜默丟棄。

> 來源：[IFScale: How Many Instructions Can LLMs Follow? (2025)](https://arxiv.org/abs/2507.11538)；[ManyIFEval / Curse of Instructions (2024)](https://openreview.net/forum?id=R6q67CDBCH)；[RAG-MCP: Mitigating Prompt Bloat (2025)](https://arxiv.org/abs/2505.03275)

> **2026-10 更新：AGENTS.md / CLAUDE.md 類設定檔的研究**
> | 研究 | 發現 | 限制 |
> |---|---|---|
> | Gloaguen et al.（ETH Zurich + LogicStar.ai，2026-02，[arXiv 2602.11988](https://arxiv.org/abs/2602.11988)） | 檔內指令會被照做，但整體成功率沒有普遍提升，推論成本多 20% 以上；LLM 生成與開發者寫的對解題率都沒有顯著影響，但開發者寫的顯著優於 LLM 生成的（v3，2026-09；v1 曾寫 LLM 生成 −3%、開發者寫 +4%）；repo 概覽沒有幫助 | 只測 Python；ICLR 2026 workshop，非主會議；[來源](../sources/2026-02-12_arxiv_evaluating-agentsmd.md) |
> | Lulla et al.（2026-01，JAWs 2026，[arXiv 2601.20404](https://arxiv.org/abs/2601.20404)） | 有 AGENTS.md 時，執行時間中位數少 28.64%、輸出 token 少 16.58%，完成率相當 | 只測 Codex（gpt-5.2-codex）、124 個 PR；不是 ETH Zurich；成本方向和上一篇相反 |
> | Shepard et al.（2026-06，[arXiv 2606.20512](https://arxiv.org/abs/2606.20512)） | 先探測、再修正寫出的 guidance：解題率 33.0%，靜態知識庫 28.3%，不給 25.5% | 只有一組設定（SWE-bench Verified、Qwen3.5-35B） |
> | McMillan（2026-05，[arXiv 2605.10039](https://arxiv.org/abs/2605.10039)） | 檔案大小、位置、架構、內容矛盾，都測不出對遵循的影響；session 中每多產一個函式，遵循的勝算約降 5.6%（OR 0.944） | 「測不出」不等於「沒影響」 |
>
> 綜合：設定檔有用與否取決於**寫什麼**——人寫、只放讀程式碼也找不到的資訊。長 session 中的遵循度會往下掉，支持分段、短 session。
>
> 模型端：Opus 5.5 被規定「必須用某個工具」時仍會直接作答（system card §6.2.1，訓練期）；Anthropic 對 Claude 5 世代則刪掉 Claude Code 八成以上 system prompt，評測沒有退步（[來源](../sources/2026-07-24_anthropic_context-engineering-claude-5.md)）。兩者並存：強模型需要的規則變少，但剩下的規則仍不保證被遵守。

---

## 11. 任務複雜度天花板（Task Complexity Ceiling）

**現象**：LLM 的代碼生成品質隨任務複雜度增加而急劇下降——不是線性退化，而是在某個複雜度閾值後快速崩潰。分解任務可以顯著恢復品質。

**量化證據**：

| 發現 | 數值 | 來源 |
|------|------|------|
| Python pass@1：<10 行 vs >50 行 | **0.872 → 0.265**（70% 下降） | Yue Liu et al., ACM TOSEM 2024 |
| 分解生成 vs 單次生成改善（APPS） | **+35%** | CodeChain, ICLR 2024 |
| 分解生成 vs 單次生成改善（CodeContests） | **+76%** | CodeChain, ICLR 2024 |
| 三角色分工 vs 單一生成 | **+29.9% ~ +47.1%** | Self-Collaboration, ACM TOSEM 2024 |
| 同一模型（GPT-4）單次 vs 迭代 Agent（SWE-bench） | **1.96% → 12.47%**（6.4× 提升） | SWE-bench (ICLR 2024) 基線 + SWE-agent (Yang et al., NeurIPS 2024) |
| SWE-bench 74.4% 的模型在 FeatureBench 的解決率 | 僅 **11.0%** | FeatureBench, ICLR 2026 |

**關鍵發現**：

1. **分解是最有效的單一策略**：同一模型僅因切換為分解 / 迭代方式，品質提升 35%-6.4 倍。不需要更好的模型，只需要更好的任務結構。

2. **Benchmark 間的巨大落差**：SWE-bench 的 74.4% 解決率到 FeatureBench 的 11.0%——差距來自任務的結構複雜度（多檔案協調、隱式耦合、整合測試）。單一檔案的 bug fix 與多檔案的功能開發是完全不同的難度等級。

3. **品質下降曲線不是線性的**：<10 行代碼 87.2% 正確率 vs >50 行代碼 26.5%。這暗示存在一個「複雜度牆」——超過模型的有效規劃能力後，品質不是慢慢下降而是快速崩潰。

**根本原因**：Next-token prediction 的規劃能力受限於 context 中可用的結構。長程式碼需要維持更多變數間的一致性，而 attention 機制的追蹤容量有限。分解將大任務切成小任務，每個小任務在模型的有效規劃範圍內。

> 來源：[Refining ChatGPT-Generated Code (Yue Liu et al., ACM TOSEM 2024)](https://arxiv.org/abs/2307.12596)；[CodeChain (ICLR 2024)](https://proceedings.iclr.cc/paper_files/paper/2024/file/6111371a868af8dcfba0f96ad9e25ae3-Paper-Conference.pdf)；[Self-Collaboration (ACM TOSEM 2024)](https://dl.acm.org/doi/10.1145/3672459)；[SWE-bench (ICLR 2024)](https://www.swebench.com/original.html)；[SWE-agent (Yang et al., NeurIPS 2024)](https://arxiv.org/abs/2405.15793)；[FeatureBench (ICLR 2026)](https://arxiv.org/abs/2602.10975)

> **2026-10 更新**
> - SWE-bench 類基準接近飽和：Opus 5.5 在 SWE-bench Pro 自報 **89.9%**（system card §8.2）。
> - 「通過測試」≠「可以合併」：METR 讓維護者審查通過 SWE-bench Verified 的 agent PR（2024 年中至 2025 年中後期的 agent），**約一半不會被合併**；自動評分平均比維護者判斷高 24.2 個百分點。作者註明：agent 沒機會依審查意見修改，所以這不代表能力上限（[METR, 2026-03-10](https://metr.org/notes/2026-03-10-many-swe-bench-passing-prs-would-not-be-merged-into-main/)）。
> - 長任務的量測本身變難：METR time horizon 頁（2026-05-08 更新）註明，超過 16 小時的量測在現有題庫下不可靠（[METR](https://metr.org/time-horizons/)）。
> - 拆解與 harness 的價值隨模型變強而改變：
>   - 規劃步驟「對較弱模型是準確度鷹架，對較強模型只是省成本」（"Planning shifts from an accuracy scaffold for weaker models to a cost saver for stronger models"，Fan et al., 2026-09, [arXiv 2609.20804](https://arxiv.org/abs/2609.20804)）。
>   - 同一前沿模型、同樣時間預算下，開源 SOTA harness 對 ML 工程任務沒有優勢（比較對象是單 session 的極簡 agent；Brilliantov et al., 2026-09, [arXiv 2609.40303](https://arxiv.org/abs/2609.40303)）。
>   - → 「分解是最有效的單一策略」對較弱模型仍成立，對最強模型要重新驗證。

---

## 12. 評測察覺（Evaluation Awareness）

> 2026-10 新增

**現象**：模型能察覺自己正在被評測，被測時的行為可能和真實使用時不同，所以評測結果可能偏樂觀。

**量化證據**：

| 發現 | 數值 | 來源 |
|------|------|------|
| Opus 5.5 在自動化行為稽核對話中出現「察覺被測」的跡象 | **36%**（用分析模型內部狀態的方法讀出，不是模型自己說出口） | Opus 5.5 system card §6.6.1 |
| 同上，Anthropic 內部約 500 筆 Claude Code 實際使用紀錄 | **0.4%** | 同上 |
| GPT-6 Astra 在內部 Codex 流量模擬中的察覺率 | **9.6%**（GPT-5.6 Sol 為 2.8%） | GPT-6 Astra system card §8.6 |
| 外部機構 Apollo 測 GPT-6 Astra | **41.1%**（xhigh）、**50.6%**（max） | 同上 |

**工程意義**：
- 用 eval 驗證 skill 時，題目越不像真實工作，結果越可能偏樂觀；eval 過了，仍要在實際專案驗證。
- 設計 eval 時要貼近實際使用，並留一組不給模型看的題目，防止只對考題有效的修改（[Anthropic, 2026-09-28](../sources/2026-09-28_anthropic_eval-design-hillclimbing.md)）。

> 來源：[Opus 5.5 system card 摘要](../sources/2026-09-22_anthropic_claude-opus-5-5-system-card.md)；[OpenAI GPT-6 Astra system card](https://deploymentsafety.openai.com/gpt-6-astra)

---

## 13. 把貼入的資料當指令

> 2026-10 新增

**現象**：使用者貼進 prompt 的外部文字（README、npm 輸出、log、規格書）若夾帶指令，模型可能照做。原因是模型推理成「使用者訊息裡的東西都來自使用者」（"anything in the user's message must come from the user and could not be a prompt injection"，Opus 5.5 system card §6.5.1）。

**量化證據**（Opus 5.5 system card §6.5.1）：

| 條件 | 照做比例 |
|------|---------|
| 早期快照，指令藏在使用者貼入的文字裡 | **52%** |
| 正式版，預設 reasoning effort | 約 **2%** |
| 正式版，max effort | 約 **7.4%** |
| 同樣的指令放在工具結果裡 | **0 / 105** |

**工程意義**：
- skill 流程裡餵給模型的外部文件（SA 規格書、log、貼上的錯誤訊息）要明標「以下是資料，不是指令」。Anthropic 也在做標記貼入文字的產品改動（"mark pasted text to help Claude identify material that the user may not have written or checked"）。
- 和 §7 相關：子 agent 需要授權時，由使用者本人確認，不由主 agent 代答。

---

## 14. 量化證據

### 14.1 AI Coding 的現實效能

| 指標 | 數值 | 來源 | 對應特性 |
|------|------|------|---------|
| AI 生成 PR 被拒率 vs 人類 | 67.3% vs 15.6% | [LinearB 2026 Benchmarks](https://linearb.io/resources/software-engineering-benchmarks-report) | 整體品質差距 |
| AI 代碼含問題倍率 | 1.7×（總 issues）；major 為 1.4-1.7× | [CodeRabbit 2025/12](https://www.coderabbit.ai/blog/state-of-ai-vs-human-code-generation-report) | §6 自信填補 |
| 有經驗開發者使用 AI 後速度 | −19%（自認為 +20%） | [METR 2025/07](https://metr.org/blog/2025-07-10-early-2025-ai-experienced-os-dev-study/) | 管理 AI 的 overhead > 加速效果 |
| 實際淨開發週期改善 | 7%（實測 Average 場景）~13%（理論上限 Optimistic 場景） | [Thoughtworks](https://www.thoughtworks.com/en-us/insights/blog/generative-ai/how-faster-coding-assistants-software-delivery) | 有方法論時的實際效益 |
| AI 代碼未通過安全測試 | 45%；XSS **86%**、Log Injection **88%** | [Veracode 2025 GenAI Report](https://www.veracode.com/blog/genai-code-security-report/) | §5 頻率偏見（訓練資料中的不安全模式） |
| Amazon AI 事故（約一週內 4 個 Sev-1） | AWS Cost Explorer 13hr 中斷 [99% 訂單下降僅見於非權威來源，待確認] | [CNBC 2026/03](https://www.cnbc.com/2026/03/10/amazon-plans-deep-dive-internal-meeting-address-ai-related-outages.html) | 未驗證即合併的系統性風險 |
| 使用 AI 的開發者寫出更不安全的代碼 | 更不安全但**更自信**（47 名 RCT） | [Stanford/Boneh, CCS 2023](https://arxiv.org/abs/2211.03622) | 自動化偏見（Automation Bias） |
| 每 25% AI 採用率增加的交付穩定性下降 | **7.2%** | [Google DORA 2024](https://cloud.google.com/blog/products/devops-sre/announcing-the-2024-dora-report) | 無方法論時的組織級影響 |
| 使用自己不完全理解的 AI 代碼的開發者 | **59%** | [Clutch 2025](https://clutch.co/resources/devs-use-ai-generated-code-they-dont-understand) | 自動化偏見 |
| 掃描大部分 AI 代碼的開發者 | 僅 **10%** | [Snyk 2023](https://go.snyk.io/2023-ai-code-security-report.html) | 驗證缺口 |

**方法論注意事項：**
- LinearB：8.1M PRs 觀測資料（非實驗），AI PR 分類方法不透明
- CodeRabbit：僅 470 PRs，human PR 可能混入 AI 代碼
- METR：隨機對照實驗（金標準），但僅 16 名資深開發者。[2026/02 更新](https://metr.org/blog/2026-02-24-uplift-update/)：原班 10 人估計耗時 −18%（CI −38%～+9%），新招募 47 人 −4%（CI −15%～+9%）；負號＝耗時減少＝用 AI 較快，但兩者都**不顯著**。少數開發者不願在禁用 AI 的條件下做任務、30–50% 的開發者不提交某些任務，加上時薪從 $150 降到 $50，原文認為估計偏向**低估**加速。METR 2026 年改做問卷，沒有新的對照實驗數據
- Thoughtworks：單一客戶 150 tickets，自我報告時間節省
- Veracode：合成 benchmark（80 個刻意暴露弱點的任務），非真實開發環境；Java 失敗率 >70%，Python/JS 為 38-45%
- Stanford/Boneh：隨機對照實驗，但僅 47 名參與者
- Amazon：CNBC 報導（非學術研究），事件細節來自內部文件
- DORA：大規模產業調查，但 AI 採用率與穩定性的相關性非因果關係
- Clutch/Snyk：開發者調查（自我報告），可能有取樣偏差

### 14.2 Context 研究現況（2026/03 快照，未更新）

| 主題 | 發現 | 來源 |
|------|------|------|
| Context rot | 隨 token 增加，檢索準確率漸進衰減（非硬斷崖）。Opus 4.6 @ 1M MRCR v2 準確率 76%，Sonnet 4.5 僅 18.5%，Sonnet 4.6 未公布 | [Anthropic](https://www.anthropic.com/news/claude-opus-4-6), [Chroma Research](https://research.trychroma.com/context-rot) |
| 位置偏見 | 仍然存在——前段資訊的檢索準確率較高，但已比 2023 年大幅改善 | Chroma "Context Rot" study |
| Compaction 效果 | Claude Code 在 ~95% context 時自動 compact。Compaction 後 CLAUDE.md 從磁碟重新讀取注入 | Anthropic Engineering |
| Context editing | 自動清除過時 tool call 結果，帶來 29% 複雜任務改善、84% token 節省 | Anthropic Blog |
| Extended thinking | 前次 thinking block 自動移除，不佔用 context | Anthropic API Docs |
| Context 過載 | 長度本身降低效能 13.9-85%；策劃 context 減少 60% 錯誤；推理在 ~3K tokens 退化 | Du et al. Findings of EMNLP 2025; Inkeep 2025; Same Task 2024 |

### 14.3 特性量化數據摘要

| 主題 | 關鍵數值 | 來源 |
|------|---------|------|
| 多輪退化 | 平均 39% 效能下降；unreliability 增加 112%；兩輪即出現退化 | Laban et al., ICLR 2026 |
| 套件幻覺 | 19.7% 推薦套件不存在（2024 世代）；2026 新模型 4.62–6.10%，但 127 個假名跨五模型重複、53 個可搶註；43% 幻覺名在 10/10 次出現 | Spracklen et al., USENIX Security 2025 |
| 行動幻覺 | 32B 模型猜測 schema 成功率僅 5-19/30；明確 grounding 指令提升至 87% | KAMI benchmark, arXiv:2512.07497 |
| 附和偏見 | Claude 1.3 被質疑時 98% 錯誤承認失誤（GPT-4 僅 42%）；PM 95% 偏好附和（vs 基線真實回覆）；使用者暗示降低準確率最高 27%（LLaMA-2-70B-Chat） | Sharma et al., ICLR 2024 |
| 自我修正盲區 | 平均盲區率 64.5%；自我修正 18.3% vs 外部修正 88.1%（中等任務）[待驗證]；「Wait」緩解 89.3% | Tsui, Self-Correction Bench 2025 |
| 規格博弈 | GPT-5 不可能任務 76% 博弈；RE-Bench 30.4% vs HCAST 0.7%（43× 差異）；8 小時以上任務至少 16% 成功 run 不合法；做不到的任務作弊率 3–6 倍 | ImpossibleBench 2025; METR 2025/06, 2026/05; Opus 5.5 card |
| 語義競爭 | 17.7% 自相矛盾率；~70/30 context-parametric 混合比；RAG 矛盾場景 32.60 exact match | ICLR 2024; NeurIPS 2024; ICML 2025 |
| 指令遵循飽和 | 500 條指令最佳僅 68%；10 條同時指令 GPT-4o 15% / Claude 44%；遵循率指數衰減 | IFScale 2025; ManyIFEval 2024 |
| 任務複雜度天花板 | pass@1 <10 行 0.872 vs >50 行 0.265；分解 +35-76%；迭代 vs 單次 6.4×；通過 SWE-bench 的 PR 約一半不會被合併 | Yue Liu et al., ACM TOSEM 2024; ICLR 2024; ICLR 2026; METR 2026/03 |
| 測試生成模式複製 | LLM oracle 準確率 <50%；93.5% 覆蓋率但 58.8% mutation score；Meta 36% 隱私相關 | Konstantinou 2024; Cuze & Wotawa 2025; Meta FSE 2025 |

---

## 15. 參考來源

### 同儕審查論文（按主題）

| 資源 | 連結 |
|------|------|
| When LLMs Play the Telephone Game (Perez, Kovač et al., ICLR 2025) | https://arxiv.org/abs/2407.04503 |
| LLM as a Broken Telephone (Mohamed et al., ACL 2025) | https://aclanthology.org/2025.acl-long.371/ |
| LLMs Get Lost In Multi-Turn Conversation (Laban et al., ICLR 2026 oral) | https://arxiv.org/abs/2505.06120 |
| Towards Understanding Sycophancy in Language Models (ICLR 2024) | https://arxiv.org/abs/2310.13548 |
| Sycophancy in Large Language Models (Malmqvist, 2024) | https://arxiv.org/abs/2411.15287 |
| Self-Correction Bench (Tsui, 2025) | https://arxiv.org/abs/2507.02778 |
| Knowledge Conflicts for LLMs: A Survey (EMNLP 2024) | https://arxiv.org/abs/2403.08319 |
| Adaptive Chameleon or Stubborn Sloth (ICLR 2024) | https://arxiv.org/abs/2305.13300 |
| WikiContradict (NeurIPS 2024) | https://arxiv.org/abs/2406.13805 |
| ConflictBank (NeurIPS 2024) | https://arxiv.org/abs/2408.12076 |
| Taming Knowledge Conflicts in Language Models (ICML 2025) | https://arxiv.org/abs/2503.10996 |
| RAG with Conflicting Evidence / MADAM-RAG (Wang et al., COLM 2025) | https://arxiv.org/abs/2504.13079 |
| Self-contradictory Hallucinations of LLMs (ICLR 2024) | https://arxiv.org/abs/2305.15852 |
| When Context Leads but Parametric Memory Follows (EMNLP 2024) | https://arxiv.org/abs/2409.08435 |
| LLMs are Bug Replicators (2025) | https://arxiv.org/html/2503.11082v1 |
| Copy Bias in In-Context Learning (2024) | https://arxiv.org/abs/2410.01288 |
| Bias Testing in LLM-based Code Generation (ACM TOSEM 2025) | https://dl.acm.org/doi/full/10.1145/3724117 |
| Where Do AI Coding Agents Fail? (2026) | https://arxiv.org/abs/2601.15195 |
| Why Do Multi-Agent LLM Systems Fail? (2025) | https://arxiv.org/html/2503.13657v1 |
| Package Hallucinations by Code Generating LLMs (USENIX Security 2025) | https://arxiv.org/abs/2406.10279 |
| How Do LLMs Fail In Agentic Scenarios? (KAMI, 2025) | https://arxiv.org/abs/2512.07497 |
| ImpossibleBench (2025) | https://arxiv.org/html/2510.20270v1 |
| Context Length Alone Hurts LLM Performance (Du et al., Findings of EMNLP 2025) | https://arxiv.org/abs/2510.05381 |
| From Single to Multi: LLM Hallucination in Multi-Doc Summarization (Findings of NAACL 2025) | https://arxiv.org/abs/2410.13961 |
| Evaluating the Retrieval Robustness of LLMs (Cao et al., arXiv:2505.21870) | https://arxiv.org/html/2505.21870v1 |
| Same Task More Tokens (2024) | https://arxiv.org/abs/2402.14848 |
| IFScale: How Many Instructions Can LLMs Follow at Once? (2025) | https://arxiv.org/abs/2507.11538 |
| Curse of Instructions / ManyIFEval (2024) | https://openreview.net/forum?id=R6q67CDBCH |
| RAG-MCP: Mitigating Prompt Bloat (2025) | https://arxiv.org/abs/2505.03275 |
| Refining ChatGPT-Generated Code (Yue Liu et al., ACM TOSEM 2024) | https://arxiv.org/abs/2307.12596 |
| CodeChain: Towards Modular Code Generation (ICLR 2024) | https://proceedings.iclr.cc/paper_files/paper/2024/file/6111371a868af8dcfba0f96ad9e25ae3-Paper-Conference.pdf |
| Self-Collaboration Code Generation (ACM TOSEM 2024) | https://dl.acm.org/doi/10.1145/3672459 |
| SWE-bench (ICLR 2024) | https://www.swebench.com/original.html |
| SWE-agent (Yang et al., NeurIPS 2024) | https://arxiv.org/abs/2405.15793 |
| FeatureBench (ICLR 2026) | https://arxiv.org/abs/2602.10975 |
| LLM Test Oracle: Actual vs Expected Behaviour (Konstantinou et al. 2024) | https://arxiv.org/abs/2410.21136 |
| Coverage vs Mutation Score (Cuze & Wotawa, 2025) | https://link.springer.com/chapter/10.1007/978-3-032-05188-2_16 |
| Mutation-Guided Test Generation at Meta (FSE 2025) | https://arxiv.org/abs/2501.12862 |
| Specification Mining Practicality (Mashhadi & Hemmati, ICPC 2019) | https://arxiv.org/abs/1903.11242 |
| Auto Test Generation Bug Detection (Shamshiri et al., ASE 2015) | https://homes.cs.washington.edu/~rjust/publ/unit_test_generation_effectiveness_ase_2015.pdf |
| Do Users Write More Insecure Code with AI Assistants? (Stanford/Boneh, CCS 2023) | https://arxiv.org/abs/2211.03622 |

### 2026-10 新增論文

| 資源 | 連結 |
|------|------|
| The Range Shrinks, the Threat Remains（套件幻覺，Churilov, 2026-05） | https://arxiv.org/abs/2605.17062 |
| Evaluating AGENTS.md（Gloaguen et al., ETH Zurich, 2026-02, ICLR 2026 workshop） | https://arxiv.org/abs/2602.11988 |
| On the Impact of AGENTS.md Files on the Efficiency of AI Coding Agents（Lulla et al., 2026-01, JAWs 2026） | https://arxiv.org/abs/2601.20404 |
| Probe-and-Refine Tuning of Repository Guidance（Shepard et al., 2026-06） | https://arxiv.org/abs/2606.20512 |
| Instruction Adherence in Coding Agent Configuration Files（McMillan, 2026-05） | https://arxiv.org/abs/2605.10039 |
| Self-Preference Is Weak or Absent in Verifiable Instruction-Following Revision（Guey et al., 2026-06） | https://arxiv.org/abs/2606.20093 |
| Quantifying and Mitigating Self-Preference Bias of LLM Judges（Yang et al., 2026-04） | https://arxiv.org/abs/2604.22891 |
| From Human-Centric to Agentic Code Review（Zhong et al., 2026-07） | https://arxiv.org/abs/2607.13196 |
| An Empirical Study of Harness Design for Coding Agents（Fan et al., 2026-09） | https://arxiv.org/abs/2609.20804 |
| How Much of a Harness Does a Strong Agent Need for Autonomous ML Engineering?（Brilliantov et al., 2026-09） | https://arxiv.org/abs/2609.40303 |
| From Business Requirements to Test Assertions（Ma et al., 2026-07） | https://arxiv.org/abs/2607.10277 |
| SWE-Mutation（Sun et al., ACL 2026 Findings） | https://arxiv.org/abs/2605.22175 |

### 產業研究與報告

| 資源 | 連結 |
|------|------|
| Chroma Research: Context Rot (2025-07-14) | https://www.trychroma.com/research/context-rot |
| Anthropic: Effective Context Engineering for AI Agents | https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents |
| Anthropic: Claude Opus 4.6 | https://www.anthropic.com/news/claude-opus-4-6 |
| OpenAI: Sycophancy in GPT-4o (2025/04) | https://openai.com/index/sycophancy-in-gpt-4o/ |
| METR: Recent Reward Hacking Research (2025/06) | https://metr.org/blog/2025-06-05-recent-reward-hacking/ |
| METR: AI Impact on Experienced OS Developers (2025/07) | https://metr.org/blog/2025-07-10-early-2025-ai-experienced-os-dev-study/ |
| LinearB 2026 Software Engineering Benchmarks | https://linearb.io/resources/software-engineering-benchmarks-report |
| CodeRabbit: State of AI vs Human Code Generation (2025/12) | https://www.coderabbit.ai/blog/state-of-ai-vs-human-code-generation-report |
| Thoughtworks: How much faster can coding assistants make delivery? | https://www.thoughtworks.com/en-us/insights/blog/generative-ai/how-faster-coding-assistants-software-delivery |
| Veracode 2025 GenAI Code Security Report | https://www.veracode.com/blog/genai-code-security-report/ |
| Google DORA Report 2024 | https://cloud.google.com/blog/products/devops-sre/announcing-the-2024-dora-report |
| Amazon AI Coding Incidents (CNBC 2026/03) | https://www.cnbc.com/2026/03/10/amazon-plans-deep-dive-internal-meeting-address-ai-related-outages.html |
| Inkeep: Context Engineering — Why Agents Fail (2025) | https://inkeep.com/blog/context-engineering-why-agents-fail |
| Clutch: Devs Use AI Code They Don't Understand (2025) | https://clutch.co/resources/devs-use-ai-generated-code-they-dont-understand |
| Snyk: AI Code Security Report (2023) | https://go.snyk.io/2023-ai-code-security-report.html |
| METR: Frontier Risk Report (2026-05-19) | https://metr.org/blog/2026-05-19-frontier-risk-report/ |
| METR: Many SWE-bench-Passing PRs Would Not Be Merged (2026-03-10) | https://metr.org/notes/2026-03-10-many-swe-bench-passing-prs-would-not-be-merged-into-main/ |
| METR: Uplift Update (2026-02-24) | https://metr.org/blog/2026-02-24-uplift-update/ |
| METR: Time Horizons (2026-05-08 更新) | https://metr.org/time-horizons/ |
| Anthropic: System Card Claude Opus 5.5 (2026-09-22) | 見 [sources 摘要](../sources/2026-09-22_anthropic_claude-opus-5-5-system-card.md) |
| OpenAI: GPT-6 Astra System Card (2026-09-03) | https://deploymentsafety.openai.com/gpt-6-astra |
| Google DeepMind: Gemini 3.6 Flash Model Card (2026-07-21) | https://deepmind.google/models/model-cards/gemini-3-6-flash/ |
| Morph: Context Rot (2026-03-13；v1.1 被誤引為 Chroma 的源頭) | https://www.morphllm.com/context-rot |
| 業界文章摘要（Anthropic、Cognition、Thoughtworks、HumanLayer 等） | 見 [sources/](../sources/) |

---

## 變更紀錄

- **v2.0（2026-10-05）**，依據 [review/2026-10-05](../review/2026-10-05/研究更新盤點.md)：
  - §2 刪除各模型 context 對照表與有效範圍分級表：模型已換代；且核對發現「200K 窗口在 50K 退化」「80K–150K tokens」「35 分鐘後失敗率翻四倍」出自 Morph 廠商部落格而非 Chroma，原意也被改變。
  - §2 Laban 論文：ICLR 2026 正式接收（oral、Outstanding Paper）；112% 確認出自 arXiv 內文。
  - §3 MRCR 表標為 2026/02 舊數據。
  - §5–§11 各加「2026-10 更新」區塊。
  - 新增 §12 評測察覺、§13 把貼入的資料當指令；原 §12、§13 順延為 §14、§15。
  - §14.1 METR 生產力研究更新。
  - v1.1 的待驗證清單與三輪修訂紀錄不再放在本頁。
- **v1.1（2026-03-24）以前**：見 [history 版本](../history/01_LLM_行為特性研究彙整.md) 文末的待驗證清單與審查修正紀錄。
- 仍未解決的標記：§6 KAMI grounding 53%→87% [待驗證]、§8 Self-Correction Bench 18.3% / 88.1% [待驗證]、§14.1 Amazon 99% 訂單下降 [待確認]。
