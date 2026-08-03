# 【Day 02】AI 不是在讀文字：Transformer 如何一路猜出下一個 Token？

## 昨天談怎麼學，今天開始拆開 AI

如果你搜尋「AI 怎麼運作」，常見的答案大概有兩種：第一種是「文字接龍」，第二種是「預測下一個 Token」。

這兩句都沒有錯，但問題也正好藏在裡面：文字究竟怎麼「接」？下一個 Token 又是怎麼「預測」出來的？

先說明一下，AI 的範圍其實很廣。為了方便這個系列的討論，當文章提到 AI 時，主要是指以大型語言模型（Large Language Model, LLM）為核心的生成式 AI 工具。

理解文字如何進入模型、又如何被輸出，是使用生成式 AI 時相當底層的一塊。雖然不需要一路算矩陣算到懷疑人生，但掌握基本流程後，之後談 Prompt、Context，甚至模型為什麼會胡說八道，都會比較可以理解。

許多現代 LLM 的核心架構，都可以追溯到 Google 研究團隊在 2017 年發表的〈[Attention Is All You Need](https://arxiv.org/abs/1706.03762)〉。這篇論文提出 Transformer 架構，最初主要用於機器翻譯，後來逐漸成為大型語言模型的重要基礎。

第一次看到 Transformer 的經典架構圖時，我的感想很直接：這不是在解釋 AI，比較像是在測試一個人會不會立刻關掉分頁。

好消息是，對一般使用者而言，目前只要先抓住三件事：

**文字怎麼進去、中間大概發生什麼，以及文字怎麼出來。**

## 電腦其實看不懂文字

假設我們輸入：

> 我想開始學習人工智慧

人看到的是一句話，但模型真正能處理的是數字。因此，文字進入模型以前，必須先經過一位翻譯員：**Tokenizer**。

Tokenizer 會把文字切成較小的單位，稱為 **Token**。Token 不一定等於一個完整的英文單字，也不一定等於一個中文字。它可能是一個單字、單字的一部分、中文字或標點符號，也可能把前面的空白一起包含進去。

實際怎麼切，取決於模型使用的 Tokenizer 與詞彙表。

例如，`transformer` 在某些 Tokenizer 中，可能被拆成 `transform` 與 `er`。常見的文字片段可以保留成一塊，較少見的詞則拆成能夠重複利用的小零件。

為什麼不直接替每個完整單字編號？因為詞彙表很快就會膨脹，`run`、`running`、`runner` 可能被視為完全不同的項目。反過來，若全部拆成單一字元，句子又會變得太長，每個零件能攜帶的資訊也比較少。

所以 Tokenization 有點像樂高積木：不替每一棟房子製作專用模具，而是準備一套可以反覆組合的零件。

不同模型家族或不同版本，可能使用相同或不同的 Tokenizer。各位可以透過 OpenAI 提供的 [Tokenizer 工具](https://platform.openai.com/tokenizer)，觀察不同文字會如何被切分。

不過要注意，OpenAI Tokenizer 與稍後使用的 Transformer Explainer 不一定採用相同的編碼方式，因此同一句話的切法與 Token ID 可能不同。翻譯員換人，置物櫃的編號也會跟著換。

## Token ID 只是置物櫃號碼（輸入）

文字切成 Token 後，每個 Token 會對應到詞彙表中的一個編號，也就是 **Token ID**。

但這個數字本身沒有語意。編號 8000 並不比編號 2000 更聰明，它比較像置物櫃號碼，只是告訴模型要到哪一格取得對應資料。

Token ID 接著會對應到一組 **Embedding**。Embedding 可以先想成高維空間中的一組座標，是模型在訓練過程中學到的向量表示。

因此，輸入流程可以先簡化成：

**文字 → Token → Token ID → Embedding**

## 中間的計算

這部分我們快速帶過。因為再細講下去，可能文章還沒讀完，你已經先把網頁關掉了。對一般使用者而言，目前不理解公式，不太影響後續使用。

除了 Token 的 Embedding，模型還需要知道每個 Token 的順序。舉例來說，「狗咬人」與「人咬狗」使用的是相同的字，但新聞價值差很多。因此，模型還要加入位置資訊，分辨誰在前面、誰在後面。

進入 Transformer Block 後，會遇到 **Attention**。它可以先理解為：

**每個 Token 都會觀察前文，判斷哪些 Token 與目前位置比較相關。**

以 GPT 這類生成模型來說，模型會使用遮罩，避免目前的 Token 偷看到未來的答案。畢竟如果考試可以先看答案，那就不叫預測，只能叫開卷作弊。

Attention 的計算會使用 Query、Key 與 Value。可以用搜尋引擎來類比：

- **Query**：我現在想找什麼？
- **Key**：哪些資訊可能與我相關？
- **Value**：找到相關資訊後，真正要取回的內容是什麼？

模型會計算 Token 之間的關聯權重，再經過多層 Transformer Block，反覆整理上下文。Transformer Block 裡不只有 Attention，也包含 MLP 等運算；目前不用背公式，只要先知道：Attention 負責讓 Token 交換資訊，後續運算再繼續整理這些表示。

## 模型不是一次寫完整句子

進入輸出階段後，可以打開 [Transformer Explainer](https://poloclub.github.io/transformer-explainer/) 實際觀察。

這個網站使用的是 GPT-2 Small，具有約 1.24 億個參數。它不是目前最新的大型語言模型，但保留了文字生成 Transformer 的核心流程，很適合拿來拆解基本原理。

模型並不是先在腦中寫好一整段答案，再一次貼給你。它會根據目前已有的 Token，替詞彙表中的所有 Token 計算分數，也就是 **Logits**。接著再透過 Temperature、Top-k 等設定與 Softmax，轉換成下一個 Token 的機率分布(可以直接看下面的例子就好，這邊很容易有每個字都認識，合起來變成他哪位...)。

以網站中的例子來說，輸入：

> Data visualization empowers users to

經過 GPT-2 的 Tokenizer 後，會被拆成六個 Token。`empowers` 並不是一個完整 Token，而是被拆成 `em` 與 `powers`；另外有些 Token 會包含前導空白。

以下數值是我在撰文時，以 `Temperature = 1.0`、`Top-k = 5` 觀察到的結果：

- `visualize`（ID：38350；47.36%）
- `create`（ID：2251；21.92%）
- `see`（ID：766；14.16%）
- `make`（ID：787；8.36%）
- `easily`（ID：3838；8.20%）

這裡要特別釐清：**Token ID 不是模型剛剛算出來的答案。**ID 是 Token 在詞彙表裡早已存在的編號；模型真正計算的是每個 Token ID 對應的 Logit 與機率。

47.36% 也不代表輸入 100 次，就一定會剛好出現 47 次 `visualize`。比較精確的說法是：在前文與設定都相同、並反覆進行大量隨機取樣時，`visualize` 出現的比例理論上會逐漸接近 47.36%。只做 100 次，結果仍可能上下浮動，機率不會準時打卡。

假設這一輪選中了 `visualize`，下一輪在概念上的輸入就會變成：

> Data visualization empowers users to visualize

此時，下一個 Token 的候選也會跟著改變。例如撰文時網站顯示：

- `and`（33.09%）
- `the`（23.08%）
- `their`（18.22%）
- `data`（13.88%）
- `,`（11.73%）

系統會選出一個 Token，接回原本的內容，再重新預測下一個 Token：

**選出下一個 Token → 接回前文 → 再預測下一個 Token**

你看到的一整段回答，就是這個步驟快速重複許多次。AI 看起來像一口氣說完，骨子裡仍是一位速度非常快的文字接龍選手。

至於模型最多能輸出多少 Token，除了受到模型或服務設定的最大輸出量限制，也可能受到 Context Window 的總長度影響。也就是說，輸入內容占得越多，能留給輸出的空間可能越少；實際規則仍要看各模型與 API 的設定。

## Top-k 決定保留多少候選 Token

剛才我們將 Top-k 設定為 5，代表在取樣階段，只保留機率最高的 5 個候選 Token。

如果把第一個範例的 Top-k 改成 2，系統就只保留 `visualize` 與 `create`，再把兩者的機率重新正規化：

- `visualize`：68.36%
- `create`：31.64%

因此，Top-k 並不是叫模型「只認識兩個字」，也不是只計算兩個 Token。模型仍然先替整個詞彙表打分數，只是在最後取樣前，把候選名單縮短。

## Temperature 不是模型今天心情好不好

在 Transformer Explainer 的輸出區，也可以調整 **Temperature**，觀察下一個 Token 的機率如何改變。

- **Temperature 較低**：機率更集中在高分選項，輸出通常較穩定、可預測。
- **Temperature 較高**：機率分布較平坦，原本機率較低的 Token 也更有機會被選中，輸出會更有變化，也更容易出現意外。

但低溫不等於正確，高溫也不等於真正具有創造力。Temperature 改變的是取樣時的機率分布，不會替模型增加知識，也不是把幻覺關掉的冷氣旋鈕。

## 停下來想一想

既然語言模型的核心工作，是根據前文預測下一個 Token，那麼一段話讀起來非常流暢，是否就代表內容一定正確？

答案顯然不是。

模型很擅長產生「統計上接得下去」的內容，但**流暢、合理與真實，是三件不同的事**。因此，在醫療、法律、防災或其他專業領域中，模型輸出的內容仍需要回到可靠資料、來源與領域專家的檢核。

理解 Transformer，不只是為了多記一個名詞，而是為了知道自己正在把什麼工作交給 AI，以及哪些責任仍然不能外包。

如果想更進一步的可以看Github上的 day2.ipynb 裡面簡單展示了QKV機制，以及用Qwen2.5-0.5B-Instruct測試temperature以及topk-k。

**明天**，我們會繼續沿著這條線往前走：既然模型只是在預測下一個 Token，為什麼規模變大之後，它卻開始像是會回答問題、寫文章，甚至進行推理？

今天的內容稍微有點多，第一次玩Transformer Explainer也不見得可以很好的理解，如果有問題，各位可以把文章同步貼到自己喜歡的AI網頁，像：GPT, Claude, Gemini, Grok...等，讓AI來輔助自己學習。

---

**參考資料**

- Vaswani, A., et al. (2017). *Attention Is All You Need*. [https://arxiv.org/abs/1706.03762](https://arxiv.org/abs/1706.03762)
- Cho, A., et al. *Transformer Explainer: LLM Transformer Model Visually Explained*. [https://poloclub.github.io/transformer-explainer/](https://poloclub.github.io/transformer-explainer/)
- OpenAI. *Tokenizer*. [https://platform.openai.com/tokenizer](https://platform.openai.com/tokenizer)
- OpenAI Help Center. *What are tokens and how to count them?* [https://help.openai.com/en/articles/4936856](https://help.openai.com/en/articles/4936856)
