# 【Day 03】從 GPT-1 到 GPT-3：模型只是越做越大嗎？

## 昨天拆開 Transformer，今天來看 GPT 怎麼長大

昨天談到，語言模型的核心工作，是根據前面的 Token 預測下一個 Token。

但這裡很容易產生一個疑問：如果它的工作始終只是文字接龍，為什麼後來的模型卻開始能回答問題、翻譯文章，甚至只看幾個例子，就大概知道我們想叫它做什麼？

要回答這個問題，可以回頭看看 GPT-1、GPT-2 到 GPT-3 的發展。

這三代模型使用的核心架構沒有突然變成另一種生物，仍然是以 Transformer 為基礎，訓練目標也都離不開預測下一個 Token。真正逐步改變的，是模型規模、訓練資料，以及人類把任務交給模型的方式。

換句話說，它不是突然換了一顆腦，而是同一種腦開始讀更多資料、長得更大，也逐漸學會看懂人類放在上下文裡的暗示。

## GPT 到底是哪三個字？

GPT 是 **Generative Pre-trained Transformer** 的縮寫：

- **Generative**：可以生成內容
- **Pre-trained**：先經過大規模預訓練
- **Transformer**：採用 Transformer 架構

今天我們習慣把第一代稱為 GPT-1，不過它在 2018 年原始論文中的名稱其實是〈[Improving Language Understanding by Generative Pre-Training](https://cdn.openai.com/research-covers/language-unsupervised/language_understanding_paper.pdf)〉，當時還沒有在封面上很有自信地寫著「GPT-1」。畢竟發表第一代時，大概也沒有人知道後面會一路數到哪裡。

## GPT-1：先把語言學起來，再準備不同考試

在 GPT-1 以前，許多自然語言處理任務會各自訓練模型。

情緒分類有情緒分類的模型，問答有問答的模型，文字相似度又是另一個模型。這有點像每換一門考試，就重新培養一位考生，不只花時間，也需要大量人工標記的題目與答案。

GPT-1 提出了一個兩階段流程：

1. **預訓練（Pre-training）**：先用大量沒有人工標籤的文字，學習預測下一個 Token。
2. **微調（Fine-tuning）**：再使用規模較小、具有標籤的資料，適應分類、問答或語意判斷等特定任務。

舉例來說，**標籤資料（Labeled Data）** 就像字卡的正反面，正面是題目，翻到背面才印著標準答案：

- 輸入資料（Input）：一張貓咪的照片
- 標籤（Label）：「貓」

（因為資料標註有助於 AI 的準確度，這幾年也多了不少「資料標註員」的遠端工作，時薪從幾百到上千元台幣都有。）

GPT-1 使用包含超過 7,000 本未出版書籍的 BooksCorpus。預訓練階段就像先讓一個人廣泛閱讀，建立語感與基礎知識——它並不知道自己未來會參加哪一場考試，只是不斷閱讀文字，練習下一個 Token 應該接什麼。

等到要處理特定任務時，就像準備公務員考試、英文檢定或研究所考試一樣，臨時抱佛腳做短期衝刺：再替模型加上簡單的輸出層，並用有標籤的資料進行微調。模型的主要參數也會在微調過程中更新。

GPT-1 約有 1.17 億個參數。以今天的標準看起來不算大，但它的重要性不只在數字，而是證明了：

> **先從大量文字學到通用表示，再微調到不同任務，是一條可行的路。**

## GPT-2：先不要急著微調，看看它自己學到了什麼

2019 年的 GPT-2 沒有大幅改造 GPT-1 的核心架構，而是採取一個非常樸實、也非常需要經費的策略：

> **把模型放大，把資料增加，然後繼續訓練。**

GPT-2 最大版本有 15 億個參數，是 GPT-1 的十倍以上。訓練資料也從書籍擴大到網路世界，也就是所謂的 WebText：研究團隊蒐集 Reddit 上至少獲得 3 個 karma 的外部連結，經過清理與去除重複後，形成超過 800 萬份文件、約 40 GB 的文字資料。這些資料比書本資料多了一層「曾經有人覺得這個連結值得按讚」的篩選，可以想成把臉書上一大堆貼文和留言，抓下來不同的貼文和留言，獲得的讚數本來就不一樣多。

GPT-2 的核心觀察是：網路文字裡本來就存在許多任務的自然示範。

例如：

- 問題後面接著答案
- 英文句子後面接著翻譯
- 長篇文章後面接著摘要
- 一段文字後面接著評論或分類

模型雖然只是在學習預測下一個 Token，卻可能同時看過大量「輸入後面應該接什麼輸出」的形式。當我們提供相似的上下文時，它便有機會延續這個結構。這就是 GPT-2 所探索的 **Zero-shot**：沒有針對該任務另外微調，也不提供完整的訓練資料，只透過文字提示要求模型完成任務。

但 Zero-shot 並不是無中生有。

比較接近的理解是：

> **提示喚起了模型在訓練資料中曾經學過的條件結構。**

如果資料裡有很多問答形式，模型較容易接出像答案的內容；如果某種輸入—輸出關係很少出現，表現自然就會受到限制。GPT-2 在部分問答與閱讀理解任務上展現出能力，但翻譯、摘要等任務仍遠非當時最佳結果。這也提醒我們：模型能力不只由參數量決定，也受到訓練資料中有哪些語言、知識與任務結構影響。

模型吃什麼長大，多少會反映在它後來會做什麼。人類如此，AI 也沒有完全逃過飲食管理。所以這幾年偶爾會看到「模型公司大量買書、掃描、銷毀」之類的負面新聞，說穿了都是為了餵給模型更多預訓練資料。

## GPT-3：不改參數，只在上下文裡示範

2020 年的 GPT-3 把規模繼續往上推到 **1,750 億個參數**。GPT-3 論文真正想探討的，不只是「模型能不能再做大」，而是模型放大後，能不能只靠自然語言指令與少量範例，適應新的任務。

下面用一則模擬的川普發文內容來舉例：

- **預訓練資料：**決定模型是否認識川普、知不知道常見的財經用語。
- **Zero-shot**：只給任務指令，不給任何示範，直接要模型回答。
- **One-shot**：在提問中先提供「一組」輸入與輸出的示範。
- **Few-shot**：在提問中提供「少數幾組」示範，讓模型抓規律。

以下實際示範 Few-shot 的用法，我們可以在 Prompt 中放入：

```text
[1] 川普説即日起對進口鋼鐵全面加徵 25% 關稅！ → 大盤下跌
[2] 川普説科技大廠宣佈在美擴建新廠，釋出數千就業機會！ → 大盤上漲

[目標問題] 川普説今天與主要貿易夥伴順利達成歷史性降稅協議！ →
```

模型看懂前兩組推文與股市反應的對照關係後，就很有機會推測出最後一題應該接「大盤上漲」。這個過程稱為 **In-context Learning**。它和傳統微調最大的差別在於：模型參數沒有更新，改變的只是這一次輸入的 Context。

可以把兩者想成：

- **Fine-tuning**：真的去上課，模型內部參數被修改
- **In-context Learning**：考試前拿到幾題範例，暫時照著規則作答

前者比較像長期學習，後者比較像把便利貼貼在桌上。對話結束、Context 消失後，那張便利貼也就跟著消失了。

因此，說 GPT-3「在推論時學會了新任務」，這句話得先打個問號。它確實能根據上下文調整行為，但參數並沒有改變；它究竟是在學習全新的任務，還是辨認並模仿訓練中見過的模式，至今仍是值得研究的問題。

## 三代 GPT，真正改變的是任務放在哪裡

把 GPT-1 到 GPT-3 放在一起，可以看到一條很有意思的路線：

| 模型 | 最大規模 | 主要資料 | 任務適應方式 | 可以怎麼記 |
|---|---:|---|---|---|
| GPT-1 | 約 1.17 億參數 | BooksCorpus | 預訓練後進行任務微調 | 先讀書，再準備特定考試 |
| GPT-2 | 15 億參數 | WebText | 探索 Zero-shot 任務轉移 | 從大量文字中認出任務形式 |
| GPT-3 | 1,750 億參數 | 更大且更多元的文字語料 | Zero-shot、One-shot、Few-shot | 把指令與範例直接放進 Context |

GPT-1 的任務主要放在**微調資料**裡；到了 GPT-3，任務可以直接靠**指令與範例**放進 Context 裡。

這個轉變，也逐漸改變了人類與模型互動的方式。過去想讓模型處理一個新任務，可能需要整理資料、設計輸出層，再重新訓練；到了 GPT-3，人們開始發現，一段自然語言加上幾個例子，就可能暫時塑造模型的行為。

Prompt 因此不再只是「問一句話」，而開始像是一個用自然語言撰寫的任務介面。

這也是後來 Prompt Engineering 逐漸受到重視的原因之一。這裡也值得多想一層：如果給 AI 的指令裡充滿「白痴」、「笨蛋」之類的字眼，某種程度上也是在引導 AI 往那個方向前進。

## 所以，答案只是把模型做大嗎？

不完全是。

規模確實很重要。GPT-3 論文顯示，許多任務的 Zero-shot、One-shot 與 Few-shot 表現會隨模型規模提升，而大型模型通常更能利用 Context 中的示範。

但「越大越好」的 Scaling Law，仍然是一個過度簡化的答案。

模型最後能展現什麼能力，至少同時受到幾件事影響：
- 模型有多大
- 訓練資料有多少
- 資料包含哪些語言與任務結構
- Context 如何描述任務
- 評估方式是否真的測到我們在意的能力

更大的模型可以容納與組合更多模式，卻不保證每次回答都正確，也不代表它真正理解了任務。GPT-3 的論文同樣記錄了不少表現不穩定的任務，以及訓練資料重疊、偏誤與評估上的限制。所以從 GPT-1 到 GPT-3，最值得觀察的或許不只是參數從「億」一路膨脹到「千億」，而是任務逐漸從模型外部的訓練流程，移進了我們提供的上下文。

## 停下來想一想

如果 GPT-3 可以從 Prompt 裡的幾個例子，推測我們想要的輸入—輸出規則，那麼我們和 AI 溝通時，真正重要的只是把問題寫得很長嗎？

顯然不是。

比長度更重要的，是我們有沒有把任務、範例、限制與預期輸出表達清楚。模型不是因為看到一篇落落長的作文就突然感動，而是從 Context 中尋找可以延續的模式。

理解 GPT-1 到 GPT-3 的發展，不只是記住三個參數量，而是看見一個重要轉變：

> **學習不再只發生在模型訓練時，也開始透過 Context 暫時影響模型的行為。**

這條路之後會一路延伸到 Prompt Engineering、Context Engineering，甚至我們今天使用 Agent 的方式。

**延伸學習**：如果想更進一步，可以看 GitHub 上的 [day3_practice.ipynb](day3_practice.ipynb)，裡面簡單展示了一次文字資料清理流程。另外也推薦一本書《深度學習革命》，內容補足了更早期的 AI 演化史，有不少有趣的故事。

**明天**，我們會繼續觀察：既然模型只是根據上下文預測下一個 Token，把複雜問題拆成步驟，為什麼有時會讓它表現得更好？這就會談到常被稱為 Chain-of-thought 的方法。

---

**參考資料**

- Radford, A., Narasimhan, K., Salimans, T., & Sutskever, I. (2018). *Improving Language Understanding by Generative Pre-Training*. [https://cdn.openai.com/research-covers/language-unsupervised/language_understanding_paper.pdf](https://cdn.openai.com/research-covers/language-unsupervised/language_understanding_paper.pdf)
- OpenAI. (2018). *Improving language understanding with unsupervised learning*. [https://openai.com/index/language-unsupervised/](https://openai.com/index/language-unsupervised/)
- Radford, A., Wu, J., Child, R., Luan, D., Amodei, D., & Sutskever, I. (2019). *Language Models are Unsupervised Multitask Learners*. [https://cdn.openai.com/better-language-models/language_models_are_unsupervised_multitask_learners.pdf](https://cdn.openai.com/better-language-models/language_models_are_unsupervised_multitask_learners.pdf)
- OpenAI. (2019). *Better language models and their implications*. [https://openai.com/index/better-language-models/](https://openai.com/index/better-language-models/)
- Brown, T. B., et al. (2020). *Language Models are Few-Shot Learners*. [https://arxiv.org/abs/2005.14165](https://arxiv.org/abs/2005.14165)
