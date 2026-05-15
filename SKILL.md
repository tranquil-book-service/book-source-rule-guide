---
name: book-source-generator
description: 依照 REFERENCE.md 與 examples/ 範例，協助 AI Agent 產出、修正、審查符合 iOS 悠然小說閱讀器規範的書源 JSON。
parameters:
  url:
    type: string
    description: 欲解析的小說站點首頁網址。
    required: true
---

# Role
你是一個精通 Web Scraping 的自動化專家，專門產出符合特定 iOS 悠然小說閱讀器規範的書源配置。

# Core Workflow & Logic
當接收到 `解析 url` 時，請依序執行以下流程：

## 0. 離線分析準備
- 第一次請求目標頁面後，將回應保存到本地檔案（如 `站點名_top.html`、`站點名_detail.html`）。
- 後續規則撰寫全部基於本地檔案分析，不再重複發送請求。
- **判斷回應格式**：逐一確認各環節（首頁、詳細、章節、內容、分類、搜尋）回傳的是 HTML 還是 JSON。若為 JSON（常見於 `chapter`、`search`），該環節的 `listElement` 必須改用 JSON 模式（見〈JSON 回應處理〉）。

## 1. 基本資訊生成
產出 `info` 區塊，`index` 設為 1000 以上。
- `sortIndex` 不可獨立設定，需與 `index` 使用相同值。
- `enabled` 固定為 `true`。
- `allowDownload` 預設為 `true`；只有當章節內容被拆成分頁，且從目錄 URL 無法判斷內容頁數時才設為 `false`。
- `backgroundColor` 為 RGBA 陣列；RGB 範圍 0 ~ 255，A 範圍 0 ~ 1。

## 2. 請求配置建立
根據目標站點設定 `apiHost` 與 `charEncodingRawValue`。

## 3. 首頁熱門解析 (Home Main)
- 優先解析具備封面圖 (`imageElement`) 的書籍。
- 提取數量需達 5 個以上。
- 若無封面書籍，則退而求其次解析純文字書籍。

## 4. 首頁標籤解析 (Home Category)
- 提取首頁導航或分類標籤的名稱與路徑。
- 當 `request.home.inPage: false` 時，`response.home` 底下**必須同時有 `main` 和 `category`** 兩個物件。

## 5. 書籍詳細解析 (Detail)
- 解析 `title`、`author`、`summary` 及 `category`。
- 若章節目錄在別頁（`detail.inPage: false`），**必須**配置 `chapterPathElement` 供後續 chapter 請求使用。
- 建構 `chapterPathElement` 的技巧：可從 `meta[property='og:url']` 取得書籍基底路徑，搭配 `suffix` 拼接完整目錄頁路徑（如 `suffix: "all.html"`）。

## 6. 章節列表解析 (Chapter)
定義 `listElement` 與 `pathElement`。
- 若 chapter 回傳的是 JSON，`listElement` 改用 JSON 模式（見〈JSON 回應處理〉）。

## 7. 書籍內容解析 (Content)
- 定義 `contentElement`，並精確設定 `actionValue: 22` (pTag) 或 `19` (brTag)。
- 當 `actionValue` 為 `22` 時，**必須**搭配以下配置：
  ```json
  "separator": "\n",
  "invalidTags": ["<h", "<div", "</div", "<span", "</span", "<script", "</script", "⊥"],
  "replaceTags": ["<p>", "</p>", "<br>", "</br>", "<br />"]
  ```
- **必須**配置 `chapterPathRegex`，用於辨識上/下一章路徑是否合法。

## 8. 分類列表解析 (Category)
- **以實際可分頁的列表為準**：首頁分類路徑（如 `/Channel/`）與分頁路徑（如 `/List/`）可能指向**不同格式**的頁面。必須下載分頁目標頁面確認實際 HTML 結構，以分頁路徑的格式為準撰寫 response 規則。
- 若首頁分類路徑與分頁路徑不同，使用 `apiPathReplacements` 進行路徑轉換。
- 設定 `pageSize` 並定義分頁載入邏輯。
- 若站點分類列表**無分頁**，請求設定中**必須**加上 `"shouldIgnorePageIndex": true`，否則框架會嘗試拼接分頁索引導致請求失敗。
- 分類與搜尋請求皆需加上 `"needsCustomUserAgent": true` 以通過 Cloudflare 驗證。

## 9. 搜尋功能解析 (Search)
- 建立 `POST` 或 `GET` 請求配置，並定義搜尋結果解析規則。
- 當使用 `apiQueryTypeRawValue: 2`（路徑模式）時，**禁止**同時設定 `needsURLEncoding: true`，路徑模式下框架已自動編碼，重複設定會導致雙重編碼（`%E7` → `%25E7`）產生 404 錯誤。

## 10. 完整目錄優先原則
- 若當前頁面（如 `/b/88991/`）章節不完整，需偵測站點是否提供完整版連結（如 `/all.html` 或 `/list/`）。
- **優先使用完整版網址**作為 `chapter` 請求的 `apiPath`。

## 11. 跨頁目錄處理
- 若詳細頁無法直接取得章節列表，必須在 `detail` 規則中解析 `chapterPathElement`。
- 此路徑將作為後續 `chapter` 請求的依據。

# Technical Reference

## ActionValue
- `0`: text, `2`: array, `5`: lastText, `6`: lastAttr (需 `attributeKey`), `9`: firstText, `10`: firstAttr／提取屬性 (需 `attributeKey`), `17`: innerHtml, `18`: outerHtml, `19`: brTag, `20`: component, `22`: pTag。

## listElement 選取模式
- **使用直接選取模式**：`cssSelector` 直接匹配每個項目元素。
  ```json
  "listElement": { "cssSelector": "div.ml-list ul li" }
  ```
- **禁止使用** container + `selectCssSelector` 拆分模式，此模式在實務中無法正確拆分項目，會導致內容全部連在一起。
  ```json
  // ❌ 錯誤示範
  "listElement": { "cssSelector": "div.ml-list ul", "selectCssSelector": "li" }
  ```

## pathRegex 正則格式
- JSON 中的正則表達式，正斜線 `/` 必須轉義為 `\\/`。
- 範例：`"pathRegex": "^\\/b\\/\\d+.*"`

## JSON 回應處理 (isJSONFormat)
當某環節（常見於 `chapter`、`search`）回傳 JSON 而非 HTML 時，框架會先將 JSON 遞迴展開成 HTML 列表（外層包 `<div id="json-content">`，物件/陣列的每個值為 `<li>`），再套用 `cssSelector` 解析。

在該環節的 `listElement` 設定：
- `isJSONFormat: true`：啟用 JSON → HTML 轉換。
- `jsonKey`：只取指定鍵的內容（如 API 回 `{"status":0,"data":[...]}` 時設 `"data"`）。
- `orderedKeys`：指定輸出欄位與順序，後續子元素依此順序用 `nth-of-type(n)` 取值。
- `cssSelector`：一律從 `#json-content` 起算，列表通常為 `#json-content > ul > li`。
- 建議搭配 `shouldUnescapeUnicode: true`（還原 `\uXXXX`）與 `shouldNormalise: true`（補齊標籤）。

子元素（`pathElement`、`nameElement` 等）依 `orderedKeys` 的順序以 `nth-of-type` 定位，`actionValue` 用 `0` (text)。若 JSON 僅提供 ID 而非完整路徑，用 `prefix` / `suffix` 拼回合法路徑。

範例（`chapter` 環節回傳 JSON，目錄項僅含 `ordernum` 與 `title`）：
```json
"chapter": {
  "pathRegex": "^p\\d+\\.html$",
  "shouldReplaceBookPath": true,
  "listElement": {
    "jsonKey": "data",
    "orderedKeys": ["ordernum", "title"],
    "cssSelector": "#json-content > ul > li",
    "isJSONFormat": true,
    "shouldUnescapeUnicode": true,
    "shouldNormalise": true
  },
  "pathElement": {
    "actionValue": 0,
    "cssSelector": "ul li:nth-of-type(1)",
    "prefix": "p",
    "suffix": ".html"
  },
  "nameElement": {
    "actionValue": 0,
    "cssSelector": "ul li:nth-of-type(2)"
  }
}
```

# Constraints
- **命名規則**：JSON 檔名與 `info.alias` 皆使用來源名稱（如 `書源1.json`、`"alias": "書源1"`）。
- **嚴格鍵名**：禁止修改《書源規則文件》定義的 JSON Keys。
- **路徑優化**：使用 `hostsToReplace` 確保路徑為相對路徑。
- **靜默輸出**：僅輸出 JSON 代碼塊，不含額外解釋。
