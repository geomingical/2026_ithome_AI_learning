# 【Day 18】API Key：從使用 AI 產品，到自己串接 AI 能力

## 昨天把模型搬回家，今天換成把模型接進自己的程式

昨天談 Ollama 時，我們看到：除了透過 ChatGPT、Claude 或 Gemini 這些產品使用雲端模型，也可以把模型放在自己的電腦上，讓其他程式使用。

但如果今天不是呼叫自己的電腦，而是想使用 OpenAI、Google、OpenRouter 或 NVIDIA 提供的雲端模型，就會遇到一個幾乎每個 AI 開發教學都會出現的東西：

> **API Key**

第一次看到 API Key，感覺很容易是：

> 又一串不知道在幹嘛的英文亂碼。

但它其實沒有那麼神祕。

API 全名是 Application Programming Interface，是讓程式按照約定互相溝通的介面，**不是 AI 專有名詞**。天氣、地圖和付款服務也有 API。本文後面若只寫「API」，主要是指 AI 服務供應商的 API。

API Key 就像櫃台發給你的**門卡兼識別證**。程式送出請求時帶著它，服務端才知道：

> 你是誰？
> 你有沒有權限使用這個服務？
> 這次使用量應該算到誰身上？
> 你是不是已經撞到使用限制？

所以當我們從「打開 ChatGPT 網頁」走到「自己寫一個程式呼叫模型」時，API Key 就成了很重要的一步。

這也是 AI 從「產品」變成「零件」的開始。

## 用 OpenAI 當例子：ChatGPT、GPT 模型與 OpenAI API

這三個是 OpenAI 的具體名稱，不是所有 AI 公司的統一叫法。其他公司也有自己的產品、模型、API 與工具，名稱和能力不一定相同。

```text
以 OpenAI 為例

OpenAI
├── ChatGPT：已經組裝完成的 AI 產品
│   ├── GPT 模型
│   └── 聊天介面、檔案、搜尋、記憶、語音等功能
│
└── OpenAI API：讓程式取得 AI 能力的平台
    ├── 文字與推理模型
    ├── 圖像模型
    ├── 語音、即時互動與轉錄模型
    ├── Embedding 模型
    └── Moderation 模型
```

**ChatGPT 是產品**。OpenAI 已經幫使用者組合好模型、介面與各種功能，打開就可以使用。

**GPT 模型是其中的核心零件**。它負責理解輸入與產生內容，但只有模型，還不等於完整的 ChatGPT。

**OpenAI API 是給程式使用的服務入口**。開發者可以選擇不同模型，接進自己的網站、App 或自動化流程。ChatGPT 的模型選單只是產品目前提供的選擇；OpenAI API 的[完整模型目錄](https://developers.openai.com/api/docs/models)還包含圖像、語音、即時互動、轉錄、Embedding 與 Moderation 等模型。

所以即使相同或相近的 GPT 模型可能同時出現在 ChatGPT 與 API，外圍的工具、記憶、操作方式與使用限制仍可能不同。

**API Key 不等於 API。**它是常見的身分驗證方式，但不是所有 API 都使用 API Key。本文提到的供應商用它辨識程式與計算用量；能用哪些模型，仍要看帳號權限與當下的模型目錄。

這也能說明另一個常見誤會：**聊天產品的訂閱，不等於 API 的使用額度。**有 ChatGPT Plus、Claude Pro 或其他聊天產品的訂閱，不代表會自動獲得同一家公司的 API 額度。產品和 API 各自計費，也有各自的限制。

在本文的 AI 情境裡，API 比較像：

> **我想讓自己的程式取得模型、圖像或語音等 AI 能力，再自己決定介面與工作流程。**

例如我可以寫一個 Python 程式：

```text
讀取資料夾裡的文章
↓
送到模型
↓
產生摘要
↓
存成 Markdown
```

這時候就不需要每次人工打開 ChatGPT、貼文字、複製答案。

模型變成程式裡的一個功能。

前面 Day 6 談 Function Calling 時，是「模型叫工具」。

今天則是反過來：

> **我們自己的程式在叫模型。**

## API Key 很像信用卡號：不要貼在聊天視窗裡裸奔

拿到 API Key 之後，第一個原則非常簡單：

> **千萬，千萬不要公開它。**

不要放進 GitHub。

不要寫進公開的前端 JavaScript。

也不要很自然地把整段 Code 貼給 AI，然後忘記裡面有：

```python
api_key="sk-xxxxxxxxxxxxxxxx"
```
如果 API Key 洩漏，別人就可能用你的身份呼叫 API。對付費服務而言，最直接的結果是：

> 他負責問問題，你負責收到帳單。

OpenAI 官方建議不要把 API Key 放進瀏覽器、手機 App 、程式碼庫，貼到聊天視窗或公開場所；懷疑 Key 已洩漏時，應立即撤銷並換新。Key 已經裸奔過一次，就不要再叫它回來上班。

因為：
> **知道這串字的人，可能就能以你的身份使用服務。**

## 最基本的做法：把 Key 放進 `.env`

對剛開始做個人專案的人，一個很常見的方式是使用 `.env`：把真正的 Key 放在獨立設定檔，而不是寫死在程式裡。

例如專案資料夾：

```text
my-ai-project/
├── app.py
├── .env
├── .env.example
└── .gitignore
```

真正的 `.env` 裡面放：

```env
OPENROUTER_API_KEY=你的真正金鑰
NVIDIA_API_KEY=你的真正金鑰
```

程式裡不要直接寫 Key，而是讀取環境變數：

```python
import os
from dotenv import load_dotenv

load_dotenv()

api_key = os.getenv("OPENROUTER_API_KEY")
```

接著最重要的一步，是在 `.gitignore` 裡加入：

```gitignore
.env
```

這樣 Git 就不會把真正的 `.env` 一起 Commit 上去。

如果專案要給別人用，可以留下 `.env.example`，內容只寫：

```env
OPENROUTER_API_KEY=your_api_key_here
NVIDIA_API_KEY=your_api_key_here
```

它只告訴別人程式需要哪些密鑰，不把自己的密鑰一起送出去。

第一次執行時，如果程式說「找不到 API Key」，先不要急著懷疑模型壞掉。通常只要依序檢查三件事：`.env` 是否真的在專案資料夾裡、變數名稱有沒有拼錯、程式是否有載入 `.env`。這些步驟很樸素，卻比重新申請三把 Key 更常解決問題。

也要記得，`.env` 是避免不小心上傳的基本做法，安全上不是讓 Key 可以放心傳給別人的免責護身符。

可以先這樣理解：

```text
.env
＝ 我真正的鑰匙

.env.example
＝ 告訴別人「這扇門需要一把鑰匙」

.gitignore
＝ 提醒 Git：「真正的鑰匙不要寄出去」
```

之後部署到雲端平台時，也要使用平台提供的環境變數或密鑰管理功能，而不是把 `.env` 上傳。Day 18 先記得：**密鑰不應該和程式碼綁死在一起。**

### Notebook 裡的 API Key 放哪裡？

`.env` 很適合一般專案；但如果是在 Google Colab 這類 Notebook 環境練習，應使用平台提供的 Secret 功能。程式只用名稱讀取 `OPENROUTER_API_KEY`，不把 Key 寫進儲存格、輸出或截圖。程式需要鑰匙，但不需要把鑰匙拿到投影幕前揮舞。

## 免費 API 可以從哪裡開始？

學生第一次接 API，不一定要先拿信用卡開始燒 Token。以下三種入口都適合學習與原型測試；它們送的不是同一份免費午餐，而是不同配菜。

### OpenRouter：一把 Key，可以試很多模型

OpenRouter 把許多供應商的模型整合到同一套 API 介面下。初學者可以先使用標示為免費的模型，或使用：

```text
openrouter/free
```

它會從目前可用的免費模型中，自動選擇符合需求的一個。

若想指定某個有免費版本的模型，會看到類似：

```text
model-name:free
```

免費模型的推論本身可為零成本，但模型清單與限制會變動，也不適合需要穩定性的正式服務。以 2026 年 8 月的規則，未購買 credits 的帳戶一天最多 50 次免費請求；累積購買至少 10 美元 credits 後，會提高到一天 1,000 次。免費比較像遊樂場發的一小袋代幣：足夠學會怎麼玩，不能當成無限供應。

### Groq：選定模型後，適合反覆練習

Groq 也有免費方案，而且限制是依模型分開計算。它適合已經選定某個支援模型、想反覆練習 API 或做快速展示的人。以文件目前列出的 Qwen3.6 27B 為例，免費方案有每分鐘 30 次、每天 1,000 次請求的限制；實際可用額度仍要以帳號頁面與當下模型規則為準。

可以這樣理解：OpenRouter 像自助餐，模型選擇多；Groq 比較像你常去的便當店，菜色較固定，但點熟悉的那一樣通常更快、更好預期。

### NVIDIA：另一個免費的開發入口

NVIDIA 的 `build.nvidia.com` 也提供免費的雲端 API，定位在開發與測試用途。選好模型後可以產生 API Key，並取得範例程式；不過同樣會有使用限制，不應當成正式服務的無限資源。

三者沒有「永遠最划算」的一家，因為模型、額度和速度都不同。剛開始可以這樣選：想練習切換模型，用 OpenRouter；想用固定模型反覆做展示，用 Groq；想試 NVIDIA 提供的模型，再選 NVIDIA。

它們都很適合拿來做：

```text
我的程式到底接不接得起來？
↓
這個提示能不能工作？
↓
JSON 能不能正常回傳？
↓
工具呼叫跑不跑得動？
```

也就是：

> **先把「會不會做」驗證出來，再決定要不要花錢放大。**

這和 Day 15 的 Google AI Studio 是同一個思路：先讓第一個請求活著回來，再考慮一百萬使用者的成本。

### 練習：孔明，你怎麼看？

現在可以真的做一次：在 [day18_practice.ipynb](day18_practice.ipynb) 裡，把 OpenRouter API Key 存進 Colab Secret，並用固定的免費模型 `google/gemma-4-26b-a4b-it:free` 建立一個「孔明，你怎麼看？」回覆器。

你只要改 system prompt，就會看到同一個模型從三國軍師穿越成現代的教練。它不是諸葛亮本人連上網路，也不是人生決策保證書；它是讓我們看見：**模型的人格與回答格式，會隨提示裡的規則改變。**

## API 為什麼一直在講 Token？

Day 2 已經談過 Token：模型不是直接按中文字數計費，而是先把文字切成它使用的基本單位。當時我們用 Token 理解「模型怎麼讀、怎麼一步一步生成」；到了 API，這些同一批 Token 多了一個很現實的身份：計費單位。API 帳單通常會分成兩部分：送進模型的輸入 Token，以及模型產生的輸出 Token。

```text
總成本
＝ 輸入 Token × 輸入單價
＋ 輸出 Token × 輸出單價
```

定價常以「每 100 萬 Token」表示。假設虛構模型的輸入為 US$0.20／1M Token、輸出為 US$1.00／1M Token；送進 1,000,000 Token、得到 100,000 Token，成本就是 US$0.2 + US$0.1 = **US$0.3**。真正價格會隨模型與版本改變，請看當下價格頁。

最容易被低估的是輸入 Token 不只等於「剛剛打的那一句」。模型若要理解「幫我再改短一點」，通常還要帶上前面對話、系統規則、檔案內容與工具說明。Day 12、Day 13 所說的Context，不只影響答案，也可能影響收到帳單時的心情；不必要的Context越多，長期成本越高。

輸出 Token 就是模型產生的內容。回答越長，通常越貴；而 Day 2 已經看過，模型是逐步產生下一個 Token，所以輸出常比輸入更貴。不過別為了省一點點費用，把該說清楚的話全部砍掉。

不同模型切 Token 的方式不同，部分模型還會記錄推理或快取 Token。因此不要背「一個中文字等於幾個 Token」；真正要算錢時，看供應商回傳的 `usage` 資訊最準。OpenRouter 會在回應中附上輸入、輸出、推理與快取等使用量。

## API Key 最後其實讓我們開始學「成本也是系統的一部分」

一開始接 API，很容易只關心：

> 回答對不對？

但真正把 API 放進工具之後，會慢慢多出幾個問題：

```text
一次請求多少錢？
一天會跑幾次？
如果失敗重試三次呢？
該用強模型，還是便宜模型？
能不能先用免費模型做分類？
複雜情況再交給最強的雲端模型？
```

這其實和 Day 17 的本機／雲端切換完全接在一起。

例如：

```text
YouTube 摘要
→ 本機模型

簡單文字分類
→ OpenRouter 免費模型

原型測試
→ Groq 或 NVIDIA 免費 API

高難度推理
→ 付費雲端模型
```

當然這只是例子，不是固定答案。

真正重要的是開始有這個觀念：

> **模型不是只能選一顆。**

程式可以根據工作難度分流。

便宜模型做大量例行工作。

困難問題才升級。

這時候 API 不再只是「怎麼把模型叫出來」。

而開始變成 AI 工程裡的一部分：

> **如何在品質、速度、成本與安全之間做選擇。**

## 停下來想一想

前面幾天我們從 OpenAI、Google、Claude 一路看到 Ollama。

最開始，我們使用 AI 的方式是：

```text
打開網站
↓
輸入問題
↓
得到答案
```

到了 API Key，工作方式開始改變：

```text
自己的程式
↓
準備脈絡
↓
帶上 API Key
↓
呼叫模型
↓
取得回應
↓
解析結果
↓
交給下一個工作步驟
```

這時候模型真的開始從「一個我去使用的產品」，變成：

> **一個我可以組進自己系統的能力。**

但取得能力的同時，也開始多出責任。

金鑰要保護。

權限要控制。

使用量要監控。

Token 要估算。

免費 API 要知道它的限制。

付費 API 要知道什麼時候值得用。

所以 API Key 看起來只是一串字，背後其實是一個很重要的轉折：

> **從使用別人設計好的 AI 工作流程，開始走向自己設計工作流程。**

而接下來，我們就要開始把前面散落的零件真正組起來。

> **當我們希望 AI 不只完成一次請求，而是依照規則、工具與工作環境持續完成任務時，Agent到底還需要哪些工作規則？**

---

## 參考資料

- OpenAI Developers. [Models](https://developers.openai.com/api/docs/models)
- OpenAI Help Center. [Best Practices for API Key Safety](https://help.openai.com/en/articles/5112595-best-practices-for-api-key-safety)
- OpenRouter. [Free Models Router](https://openrouter.ai/docs/guides/routing/routers/free-router)
- Groq. [Rate Limits](https://console.groq.com/docs/rate-limits)
- NVIDIA. [Try NVIDIA NIM APIs](https://build.nvidia.com/settings/api-keys)
