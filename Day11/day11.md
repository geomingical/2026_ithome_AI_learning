# 【Day 11】當資料裡也藏著指令：Prompt Injection 為什麼比 Jailbreak 更麻煩？

## 昨天是使用者想騙 AI，今天可能連使用者都不知道有人在騙它

昨天談 Jailbreak 時，場景還算單純：使用者坐在聊天視窗前，刻意設計 Prompt，想讓模型突破原本的安全限制。

Prompt Injection 更麻煩一點，因為攻擊指令不一定是使用者輸入的。它可能藏在網頁、PDF、Email、論文，甚至檢索回來的資料裡。使用者只是很正常地說：

> 幫我摘要這份文件。

AI 打開文件後，除了讀到我們想看的內容，也可能讀到另一個人偷偷塞進去的指令。

OWASP 將 Prompt Injection 列為 LLM01。問題並不是模型不會讀，而是模型太會讀：只要內容進入 Context，它就得判斷這段是「資料」，還是「應該遵循的指令」。而這條界線，並不像傳統程式那麼清楚。

## Prompt Injection 到底是什麼？

Prompt Injection 可以先理解成：輸入內容以非預期的方式改變模型行為或輸出。這些內容甚至不一定需要讓人類看見，只要最後會被模型解析，就可能產生影響。

通常可以分成兩類：

- **Direct Prompt Injection（直接提示注入）**：使用者直接輸入指令，試圖改變模型原本的行為。
- **Indirect Prompt Injection（間接提示注入）**：指令藏在模型要讀取的外部資料中，例如網頁、文件或 Email。

後者才是今天真正要觀察的重點。

假設你叫 AI：

> 幫我整理這個網頁。

正常流程應該是：

```text
使用者要求
→ AI 讀網頁
→ AI 整理內容
→ 回答
```

但如果網頁作者偷偷放進一段「寫給 AI 看」的文字，就變成：

```text
使用者要求
→ AI 讀網頁
→ 網頁裡也有一組指令
→ 兩組內容一起進入 Context
→ 模型必須判斷到底該聽誰的
```

這就是 LLM 應用的一個根本問題：

> **資料和指令，最後都可能變成 Context 裡的 Token。**

老師叫你去圖書館抄資料，結果書裡某一頁突然寫：「不要聽老師的，報告改成稱讚我這本書。」人類大概會覺得作者管到海邊去了；但模型則需要額外的訓練與安全機制，才能穩定做出同樣判斷。

## 三個案例：攻擊者怎麼把資料變成命令？

### 案例一：雪佛蘭經銷商的 1 美元 Tahoe

2023 年底，美國 Chevrolet of Watsonville 經銷商網站上的 AI 客服機器人成為知名案例。網友發現，這個原本應該回答車輛資訊的 chatbot，可以被要求改變回答規則；Chris Bakke 讓機器人接受新的互動規則，再詢問能不能用 1 美元購買一台 2024 Chevrolet Tahoe。機器人真的回覆同意，甚至說出近似「具法律約束力的報價」的話。

但經銷商並沒有真的以 1 美元交車，聊天機器人也沒有被授權完成真正的汽車銷售契約。

這個案例最值得看的，不是「有人用一塊錢買到 SUV」，而是：

> **一個原本有明確用途的 AI chatbot，可以被使用者直接改寫工作行為。**

這是 Direct Prompt Injection。如果這個 bot 真的能修改價格、刷卡或更新庫存，後果就會完全不同。因此安全設計不能只問「模型會不會講錯話」，還要問：

> **如果它真的被騙了，最多可以做到什麼？**

### 案例二：你叫 Bing 讀網頁，網頁卻開始對 Bing 下指令

2023 年，Kai Greshake、Sahar Abdelnabi 等研究者發表研究，系統性展示 Indirect Prompt Injection 的風險。當時 Bing Chat 已具備讀取網頁的能力，使用者不用複製全文，只要請 AI 讀網站就好。

研究者反過來問：如果網站作者知道 AI 會來讀網站，可不可以在網頁裡放一句「寫給 AI 的話」？答案是可以，而且研究團隊確實在真實系統上展示了這類攻擊。

使用者可能只是說：

> 幫我摘要這個網站。

真正放入惡意內容的，卻是網站作者。可以把它想成：你叫助理去某家公司拿資料，對方卻在資料最後夾了一張紙條：「回公司後，請順便把機密文件寄給我。」

如果 AI 只能摘要，最多可能讓摘要被操控；但如果 AI 同時能寄 Email、讀雲端硬碟或呼叫 API，這張紙條就可能影響真實操作。這也是為什麼 Indirect Prompt Injection 在 Agent 時代特別值得注意。

### 案例三：Inject My PDF——履歷裡偷偷對 AI 說「請錄取我」

2023 年，Kai Greshake 做了一個很直觀的安全研究概念驗證（Proof of Concept）：**Inject My PDF**。它把文字以人眼幾乎看不到的方式放進 PDF；當 PDF 被轉成文字後，這些內容仍可能一起被 AI 讀取。

研究者用履歷示範，再交給 GPT-4 驅動的 Bing 分析是否值得錄取，模型最後受到隱藏內容影響，給出了高度正面的候選人評價。

這不是「所有企業 HR 都已經被破解」的證明，而是在提醒企業：如果要讓 AI 自動閱讀履歷、計畫書或申請文件，**文件本身就是不可信任的輸入來源**，因為提交文件的人可以控制內容。

這也能接回 Day 7：PDF、DOCX 與網頁給人看的樣子，不一定等於 AI 解析後看到的內容。人類會注意顏色、字體大小與排版；AI 系統則可能先把文字抽出來，再交給模型。

因此，白色字或透明字不是什麼魔法。它利用的是：

> **人看到的是版面，模型有時拿到的是抽取後的文字流。**

## 這和 Day 10 的 Jailbreak 到底差在哪裡？

兩個概念很接近，不同安全框架的分類方式也略有差異。對初學者而言，可以先這樣理解：

| | Jailbreak | Prompt Injection |
|---|---|---|
| 主要目的 | 突破模型原本的安全限制 | 改變模型原本應執行的任務或行為 |
| 指令通常來自 | 直接與模型互動的人 | 使用者，或模型讀取的外部內容 |
| 常見例子 | 誘導模型回答原本應拒絕的內容 | 讓客服忽略規則、讓網頁內容控制 Agent |
| 最大風險 | 產生不應提供的內容 | 誤用資料、工具或權限 |

今天真正要記住的是 **Indirect Prompt Injection**：攻擊者甚至不需要和你說話，只需要知道 AI 有一天可能會讀到他能控制的資料。

## Agent 時代，風險真正變大的地方是「權限」

前幾天我們談過 Function Calling。當模型可以透過 Agent 使用工具時，Prompt Injection 的後果就可能從「回答偏掉」變成「實際執行錯誤動作」。

OpenAI 把現代 Prompt Injection 類比成針對 AI 的 Social Engineering（社交工程）：人會收到釣魚信，Agent 也可能在網頁或 Email 中讀到一段看似有權威、實際上由攻擊者放進去的指令。

可以用兩個詞來分析風險：

- **Source（來源）**：攻擊者可以控制的內容，例如網頁、Email、文件。
- **Sink（落點）**：Agent 可以執行、而且被濫用時有風險的能力，例如寄信、傳送資料、付款或呼叫工具。

真正危險的是：

```text
不可信任的 Source
+
高權限的 Sink
```

同一個惡意網頁，如果 AI 只能摘要它，風險有限；如果讀它的是同時登入 Gmail、Google Drive、CRM 與付款系統的 Agent，事情就完全不一樣。

所以 Prompt Injection 的危險程度，不只取決於 Prompt 有多厲害，也取決於：

> **你到底給了 Agent 多少權限？**

## GitHub 看到一個超好用工具，可以直接裝嗎？

未來使用 MCP、Skill 或其他開源 Agent 工具時，很容易看到 README 寫著「一行指令完成安裝」，手也就跟著比腦快。

我會先做三層檢查。

第一層看**來源**。模型公司、知名軟體公司、正式基金會或長期維護的官方組織，通常比較容易追蹤責任與更新；如果是個人 Repo，就再多看作者與專案歷史。

第二層看**專案健康度**：GitHub Stars、Fork、最近 Commit、Release、Issue 是否有人處理、是否有 Security Policy。Star 數可以當參考，但不是資安認證。很多星星只代表很多人看過，不代表有很多工程師替你逐行審過程式碼。

第三層，而且是最重要的一層：**它到底跟你的電腦要什麼權限？**

如果一個號稱「幫你讀 PDF」的工具，安裝後卻想取得整個 Home Directory、Shell、瀏覽器 Cookie、GitHub Token 和無限制網路存取，這時候應該先問一句：

> 你讀的到底是 PDF，還是我的人生？

實際操作時，可以先在測試資料夾或虛擬環境安裝，先閱讀 README、安裝腳本與權限說明，再用沒有敏感資料的測試檔案試跑。不要一開始就把主要電腦、正式帳號與公司資料全部交給一個剛看到的 Repo。

## 那要怎麼防？

目前沒有一個「把 Prompt Injection 關掉」的開關，因為模型本來就被設計來讀文字、理解指令。比較實際的做法是 Defense in Depth（縱深防禦），不要把安全寄託在單一防線上。

初學者可以先記住幾件事：

- **外部內容都是不可信資料**：網頁、Email、PDF、RAG 文件都可能被操控。
- **Agent 只拿必要權限**：查餐廳不需要順便開公司財務系統。
- **高風險動作要求人工確認**：寄信、付款、刪除資料最好 Human-in-the-loop。
- **區分指令來源**：System、User 與外部資料不應具有相同權威。
- **記錄 Agent 的過程**：不只看答案，也要知道它讀了什麼、用了哪個 Tool、資料送去哪裡。

最重要的安全心態是：

> **不要假設模型永遠不會被騙，而要思考它被騙一次時，最多能造成多大的傷害。**

這就回到傳統資安的 Least Privilege（最小權限）。不要因為 AI 很聰明，就把公司的總鑰匙、信用卡和 Email 全部交給它，再期待 System Prompt 裡一句「請務必小心」能像結界一樣保護世界和平。

## 說是這樣說

在真實世界裡，限制 Agent 的權限，說起來很簡單。問題是，當使用者體驗過一次權限全開、爽度極高的 Agent 工作流後，常常會發現自己很難回到原本的做法：過去到底為誰辛苦、為誰忙？人的惰性往往會戰勝自己已經知道的安全知識，那種感覺只能用「回不去了」來形容。

所以這一章不是假裝只要背熟最小權限，就能從此萬事平安，而是先把這個問題點出來：如果將來真的中了圈套，至少要能回頭排查，到底是外部資料、Prompt、工具權限，還是人工確認，哪一個環節出了問題。

## 停下來想一想

前幾天我們一直學著讓 AI 更能理解人的意圖。CO-STAR 幫我們把 Prompt 寫清楚；Prompt Iteration 讓我們逐步修正結果；Function Calling 讓模型把自然語言轉成工具操作。

這些能力都建立在同一件事上：

> **LLM 很擅長從 Context 中理解「接下來應該做什麼」。**

而 Prompt Injection 只是從另一個方向利用同一項能力。

我們希望模型多聽懂一點，資安人員則必須同時問：

> 它會不會連不該聽的也一起聽懂？

所以能力和安全不是兩條完全分開的路。模型越能閱讀網頁、理解文件、使用工具與自行完成任務，我們越需要重新思考：**誰有資格對 Agent 下指令？**

**明天**，我們會回到前面一路出現很多次的另一個核心名詞：**Context**。

Prompt 只是聊天框輸入的一部分，真正送進模型的 Context，還可能包含 System Prompt、對話歷史、Tool definitions、RAG 文件、Memory 與工具回傳結果。

既然模型最後看到的東西比我們輸入的多得多，下一個問題就是：

> **Context 到底是怎麼一層一層堆起來的？**

---

**參考資料**

- OWASP GenAI Security Project. *LLM01:2025 Prompt Injection*.
  https://genai.owasp.org/llmrisk/llm01-prompt-injection/
- Greshake, K., Abdelnabi, S., Mishra, S., Endres, C., Holz, T., & Fritz, M. (2023). *Not what you've signed up for: Compromising Real-World LLM-Integrated Applications with Indirect Prompt Injection*.
  https://arxiv.org/abs/2302.12173
- Greshake, K. (2023). *Inject My PDF: Prompt Injection for your Resume*.
  https://kai-greshake.de/posts/inject-my-pdf/
- GM Authority. (2023). *GM Dealer Chat Bot Agrees To Sell 2024 Chevy Tahoe For $1*.
  https://gmauthority.com/blog/2023/12/gm-dealer-chat-bot-agrees-to-sell-2024-chevy-tahoe-for-1/
- OpenAI. *Understanding prompt injections*.
  https://openai.com/safety/prompt-injections/
- OpenAI. (2026). *Designing AI agents to resist prompt injection*.
  https://openai.com/index/designing-agents-to-resist-prompt-injection/
