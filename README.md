# LINE 電子名片

在 LINE 裡傳出去是**一張圖**，點下去開啟**自我介紹網站**，網站裡有 IG 圖片磚導到 Instagram。
純靜態，**不需要後端伺服器**。目前內容是假想範例，等真資料確定再替換。

## 運作方式

```
好友收到一張名片圖（Flex Message，滿版圖片）
        │ 點一下
        ▼
index.html  自我介紹網站
        ├─ 頭像 / 姓名 / 職稱 / 標語
        ├─ About 介紹文 + 技能標籤
        ├─ Instagram 三格圖片磚 ──► 你的 IG
        └─ Contact 連結清單（網站／作品集／Email／電話）
```

**為什麼不能直接傳圖片？** LINE 的純圖片訊息點不動，要「看起來是圖、但可以點」有兩條路。

## 兩條路線（圖片放的地方不一樣）

| | 路線 A：圖文訊息 | 路線 B：LIFF + Flex |
|---|---|---|
| 誰能發 | 只有官方帳號（歡迎訊息、關鍵字自動回應、群發） | 你本人在任何聊天室手動分享 |
| **圖片放哪** | **上傳到 LINE**（OA Manager 內） | 自己架的網站，用公開 https 網址 |
| 要寫程式 | 不用 | 要（本專案的 `index.html`） |
| 建議 | **先做這條**，一個下午能上線 | A 跑順之後再補 |

兩條都需要「自我介紹網站有公開網址」，這躲不掉。

### 路線 A 步驟（不用寫程式）

1. 用 `share-card.html` 匯出 `card.png`（已是 1040×1040）
2. OA Manager 左側 →「圖文訊息」→ 建立 → 版型選正方形一整格
3. 上傳 `card.png` ← **圖就是傳到這裡**
4. 動作選「連結」，網址填自我介紹網站
5. 填標題（只顯示在對話列表預覽）→ 儲存
6. OA Manager →「加入好友的歡迎訊息」→ 編輯 → 新增「圖文訊息」→ 選剛建的那則 → 儲存

結果：有人加好友 → 自動收到名片圖 → 點圖開網站 → 網站裡點 IG 磚進 Instagram。

> 版型選「上下兩塊」的話，可以上半連網站、下半直接連 IG，一張圖兩個連結。

**回應設定**：走路線 A 請讓「自動回應訊息」保持**開啟**，Webhook 維持關閉／留空。
（只有自架後端走 webhook 時才需要關掉自動回應。）

### 路線 B（本專案程式碼）

LINE 不提供使用者端的圖片空間，所以 Flex Message 的圖片只能用公開 https 網址，
`card.png` 必須跟網站一起上傳。

## 檔案

```
line_namecard/
├── index.html        ← 自我介紹網站（HTML+CSS+JS 全在裡面）＋ LIFF 分享按鈕
├── share-card.html   ← 名片圖產生器，匯出 1080×1080 的 card.png（工具，不用上傳）
├── avatar.jpg        ← 你的 AI 頭像（放進來就會自動顯示，缺檔會退回文字頭像）
├── ig1.jpg ig2.jpg ig3.jpg  ← IG 圖片磚用的三張圖（選填，缺檔顯示佔位圖示）
├── card.png          ← 由 share-card.html 產生，要上傳
└── README.md
```

## 要改的地方（都在 `index.html`）

| 位置 | 改什麼 |
|---|---|
| `<style>` 開頭 `:root{...}` | 配色，`--accent` 是主強調色 |
| `<header class="hero">` | 頭像、姓名、職稱、標語 |
| `<section class="bio">` | 介紹文與技能標籤 |
| Instagram 區塊 | 三個 `ig-tile` 的 `href` 與 `ig-cta` 的帳號 |
| Contact 區塊 | 連結清單 |
| `<script>` 的 `PROFILE` | vCard 與分享訊息用的資料，要跟畫面一致 |

`share-card.html` 的文字要同步改一次（它是獨立的一張圖）。

## 上線步驟（順序不能顛倒）

LIFF 應用建立時要填網址，所以得先有網頁。

**1. 產名片圖**
`avatar.jpg` 放好 → 瀏覽器開 `share-card.html` → 按「下載 card.png」→ 把 `card.png` 放回這個資料夾。

**2. 上線**
整個資料夾丟到 GitHub Pages 或 Netlify Drop，拿到 `https://...` 網址。
記下 `card.png` 的完整網址，例如 `https://你的帳號.github.io/namecard/card.png`。

**3. 建 LIFF**
LINE Developers → 你的 Channel → **LIFF** 頁籤 → Add
- Size：`Full`
- Endpoint URL：步驟 2 的網站網址
- Scopes：勾 `profile` 和 `chat_message.write`（**分享給好友必須勾後者**）
- 開啟 **Share target picker**

**4. 回填兩個值到 `index.html`，重新上傳**
```js
const LIFF_ID = "1661234567-AbCdEfGh";
const CARD_IMAGE_URL = "https://你的帳號.github.io/namecard/card.png";
```

**5. 掛到官方帳號**
OA Manager → 圖文選單 → 動作選「連結」→ 網址填 `https://liff.line.me/<LIFF_ID>`
（用這個網址才會在 LINE 內開啟，分享功能才有作用。）

## 注意事項

- `index.html` 是公開的前端。**Channel secret 和 access token 絕對不要寫進去**。
  LIFF ID 是公開識別碼，放前端沒問題。
- 「分享名片給好友」在 LINE 以外的瀏覽器會自動隱藏（`shareTargetPicker` 只在 LINE 內可用），
  所以用電腦開看不到按鈕，這是正常的。
- `CARD_IMAGE_URL` 留空時，分享出去會退回文字版卡片（仍可點，只是沒有圖）。
- Flex 圖片網址必須 https、JPEG 或 PNG。換圖後 LINE 會快取一段時間，
  建議換檔名（`card-v2.png`）而不是蓋掉同名檔。
- 「存聯絡人」在 LINE 內建瀏覽器可能被擋下載，用 Safari / Chrome 開沒問題。
