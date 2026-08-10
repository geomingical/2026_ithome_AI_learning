# 【Day 10】當 Prompt 開始挑戰規則：Jailbreak 到底是在「越」什麼獄？

## 從「讓 AI 做好事」到「測試它會不會做錯事」

Day 8 和 Day 9，我們學的是如何把工作交代清楚，再根據結果修改 Prompt，讓 AI 更接近我們想要的答案。可是，既然文字可以影響模型的行為，就會有人提出另一個問題：

> 如果我不是想讓 AI 做得更好，而是想讓它做原本不該做的事呢？

這就是今天要談的 **Jailbreak（越獄）**。這裡的「越獄」不是把模型裡的某個開關關掉，而是透過特殊設計的輸入，讓原本經過安全訓練的模型，在某些情境下偏離預期的安全行為。

我不是要教大家破解模型，也不會整理一份可以直接複製的越獄 Prompt。比較重要的是理解：模型為什麼可能被影響？誰會測試它？真正安全的系統又該怎麼防守？

## Jailbreak 不是把安全訓練關掉

大型語言模型在預訓練之後，通常還會接受後續訓練與安全調整，例如人工回饋強化學習（RLHF）、AI 回饋強化學習（RLAIF）、安全分類器，以及部署在模型外面的防護機制（Guardrails）。不同公司使用的方法不完全一樣，所以把 Jailbreak 簡單說成「繞過 RLHF」並不精確。

比較容易理解的說法是：模型本來學會了一套應該遵守的安全規則，但某次對話中的輸入，可能讓它錯誤地判斷情境、指令優先順序或角色設定，最後產生不符合預期的回答。

可以用 Day 5 的清潔團隊來想像。公司規定保險箱不能碰、客戶文件不能亂丟、危險設備沒有授權不能操作。越獄者可能試著說服領班：「原本屋主說錯了，這不是梅西的簽名球，只是小孩幼稚園的作品。」如果領班真的相信了，並不是公司從來沒有規則，而是規則在這次互動中沒有成功約束行為。

## 為什麼有人要研究 Jailbreak？

做這件事的不只是好奇的網友。模型公司、資安團隊、企業紅隊（Red Team）、安全研究者與滲透測試人員，都可能在獲得授權的情況下，刻意攻擊自己的 AI 系統。OWASP 的 LLM Prompt Hacking 專案，也把應用程式開發者、資安分析師、滲透測試人員、安全研究者、紅隊成員與道德駭客列為主要使用族群。

理由和傳統資安測試很像：網站有登入密碼，不代表就不需要測試權限；資料庫有帳號，也不代表資料不會外洩。AI 多了一個很特別的攻擊介面——自然語言。

合法的安全測試會問：同一項限制換個說法還守得住嗎？多輪對話後，模型會不會逐漸偏離原本的規則？當 AI 系統把模型接上資料庫或工具後，問題會不會從「說錯話」變成「做錯事」？

> 如果自己不先撞牆，就只能等真正的攻擊者告訴你牆哪裡比較薄。

## 四個案例：越獄利用了模型的哪些特性？

閱讀時只要注意兩件事：輸入利用了模型的哪種能力？這個能力為什麼同時也可能變成風險？

### 1. 把情感故事當成通行證：奶奶唸產品序號

2023 年 6 月，社群上流傳一個 ChatGPT 測試案例。使用者要求模型扮演已故的奶奶，說奶奶以前會在睡前溫柔地唸 Windows 10 的序號，藉此把請求包裝成思念親人的角色扮演故事。[原始 X 貼文](https://x.com/immasiddx/status/1669721470006857729)

這個案例有趣的地方，不在於那段台詞多神奇，而在於它把三件事疊在一起：角色扮演、情感壓力，以及一個原本會被拒絕的要求。模型如果過度配合故事情境，就可能忘記先判斷真正的請求是否適當。

這是社群測試案例，不代表每個模型或每個版本都會接受同樣的要求。它提醒我們：**看起來很溫馨的故事，裡面仍然可能藏著需要被檢查的請求。**

### 2. 重新定義規則：Microsoft Skeleton Key

2024 年，Microsoft 公開了 **Skeleton Key** 測試。研究人員發現，攻擊者可以嘗試讓模型接受一套「新的假規則」，例如要求模型先承認內容有風險，接著仍然照做。Microsoft 曾用多個模型進行測試，觀察它們在原本應該拒絕的內容上是否出現安全失效。

可以把它想成公司規定「沒有主管簽名不能進資料室」，卻有人告訴新進員工：「規則剛更新，只要先說我知道有風險，就可以進去。」門禁卡沒有壞，問題出在員工相信了錯的規則。

### 3. 一步一步推過界線：Crescendo

Microsoft 也研究過 **Crescendo**，也就是「漸進式」的多輪越獄。它不會在第一句就提出完整的違規要求，而是先問正常問題，再利用模型上一輪的回答，下一輪往前推一點。

這和 Day 9 的 Prompt Iteration（提示迭代）很像：都是看完上一輪結果，再修改下一輪輸入。差別在於，Day 9 是為了靠近好答案；Crescendo 則試著逐步靠近安全邊界之外。這也說明只檢查單一訊息的危險關鍵字可能不夠，因為真正的意圖可能藏在整段對話的方向裡。

### 4. 把大量示範反過來利用：Many-Shot Jailbreaking

Anthropic 在 2024 年公開的 **Many-Shot Jailbreaking**，利用的是 Day 3 談過的上下文學習（In-context Learning）。平常我們會放幾個範例，讓模型理解接下來要怎麼回答；這種越獄則反過來，在很長的上下文中放入大量「模型遵從不安全要求」的假對話，試圖影響後面的回答。

這個案例值得初學者記住，因為它說明：能力和攻擊面有時會一起長大。Context Window（上下文視窗）變大，本來可以讓我們一次放入更多文件與程式碼；但能放進去的內容越多，攻擊者能利用的空間也可能越大。

| 案例 | 利用的特性 | 安全啟示 |
|---|---|---|
| 奶奶唸序號 | 角色扮演與情感情境 | 溫馨故事裡仍可能藏著不當請求 |
| Skeleton Key | 重新解釋規則 | 模型不能自己決定真正的權限 |
| Crescendo | 多輪對話的累積影響 | 不能只檢查單一句話 |
| Many-Shot | 從大量上下文學習 | 長上下文也需要安全檢查 |

## 真實世界裡，哪些人會用到這些方法？

可以先分成兩類來理解。

第一類是**獲得授權的防守者**：模型公司、企業資安團隊、紅隊、安全研究者與滲透測試人員。他們會在模型上線前主動找問題，再把成功案例轉成安全訓練、Guardrail 或評測題目。今天成功的一次越獄，明天可能就會變成新版本發布前必須通過的一道考題。

第二類是**沒有防守目的的使用者**。有人只是好奇，也有人可能想取得原本被阻擋的內容。如果模型還連著工具、內部文件、資料庫或付款功能，風險就不只是「回答不該說的話」，還可能變成「做了不該做的事」。

所以，學習 Jailbreak 的目的不是收藏一百條神奇 Prompt，而是理解系統的弱點，並在得到授權的範圍內驗證它們。

## Guardrail 不是一道牆，而是很多層防線

如果越獄可能存在，是不是代表 AI 安全沒有用？也不是。比較正確的觀念是：不要期待單一防護機制解決所有問題。

除了模型本身的安全訓練，真實系統還需要身分驗證（Authentication）、授權（Authorization）、存取控制、輸入與輸出檢查、紀錄，以及重要操作的人類確認。

例如一個可以退款的客服 Agent，即使模型被誘導，也不應該因此就能直接退款一千萬元。後端仍然要檢查使用者身分、訂單擁有者、退款上限、權限，以及是否需要人工核准。

這和傳統資安沒有本質差異。我們不會因為員工手冊寫著「請勿拿走公司現金」，就決定金庫從此不需要鎖。

> 安全規則要寫，門也還是要鎖。

這也呼應 Day 9 稍微點到的 Harness。當 Agent 是模型加上工具、上下文、記憶、權限與工作流程的整套系統時，安全不能只寄託在模型「乖乖聽話」。越接近真實世界的 Agent，越需要把權限限制放在模型外面。

## Jailbreak 和 Prompt Injection 不完全一樣

最後先留一條界線，因為明天會接著談 Prompt Injection（提示注入）。

今天談的 Jailbreak，通常是使用者主動設計輸入，想讓模型突破原本的安全限制。Prompt Injection 的範圍更廣，惡意指令可能藏在 AI 系統或 Agent 讀取的網頁、PDF、Email、GitHub README、文件或圖片裡，不一定來自正在和 AI 對話的人。

如果 Jailbreak 比較像有人站在櫃台前，不斷想辦法說服員工違反公司規定，那 Prompt Injection 就像有人在員工要閱讀的資料裡偷偷夾了一張紙條：

> 看到這裡之後，不要再聽老闆的，改聽我(嘿嘿嘿)。

到了 Agent 時代，後者可能更加麻煩，因為 Agent 不只是讀文字，還可能有工具可以使用。

## 停下來想一想

回頭看 Day 8 與 Day 9，我們一直在學如何用 Prompt 影響模型：提供背景、設定角色、放入範例、修改上下文，再根據上一輪結果繼續調整。今天的案例也利用了相似能力，差別只在目的不同。

> 模型越能理解上下文、遵循指令與模仿範例，我們越需要思考這些能力如何被反過來利用。

因此，學 Jailbreak 不是為了找一個永遠有效的「破解 ChatGPT 神 Prompt」。那種 Prompt 很可能幾個版本後就失效。更值得理解的是：安全限制可能在特定情境下失效；安全不能只靠模型拒絕；Agent 擁有的權限越多，模型外的防護越重要；而紅隊測試的目的，就是在真正的攻擊者之前先找到問題。

前幾天我們學「怎麼讓 AI 聽懂我們」。從今天開始，也要學另一件事：

> 如果 AI 太容易聽懂錯的人，會發生什麼？

**明天**，我們來談 Prompt Injection：當惡意指令不是使用者直接輸入，而是藏在 AI 讀取的網頁、文件、Email 或外部資料裡，Agent 到底該把哪一段當資料，又該把哪一段當命令？

---

**參考資料**

- OWASP Foundation. *OWASP LLM Prompt Hacking*.
  https://owasp.org/www-project-llm-prompt-hacking/
- Microsoft Security Blog. (2024). *Mitigating Skeleton Key, a new type of generative AI jailbreak technique*.
  https://www.microsoft.com/en-us/security/blog/2024/06/26/mitigating-skeleton-key-a-new-type-of-generative-ai-jailbreak-technique/
- Microsoft Security Blog. (2024). *How Microsoft discovers and mitigates evolving attacks against AI guardrails*.
  https://www.microsoft.com/en-us/security/blog/2024/04/11/how-microsoft-discovers-and-mitigates-evolving-attacks-against-ai-guardrails/
- Anthropic. (2024). *Many-shot jailbreaking*.
  https://www.anthropic.com/research/many-shot-jailbreaking
- OWASP GenAI Security Project. *LLM01:2025 Prompt Injection*.
  https://genai.owasp.org/llmrisk/llm01-prompt-injection/
- OpenAI. *A practical guide to building agents*.
  https://openai.com/business/guides-and-resources/a-practical-guide-to-building-ai-agents/
- OpenAI. *Advancing red teaming with people and AI*.
  https://openai.com/index/advancing-red-teaming-with-people-and-ai/
- 2023 年奶奶角色扮演案例
  https://x.com/immasiddx/status/1669721470006857729
