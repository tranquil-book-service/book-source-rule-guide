# 書源規則文件


# 書源資訊

供書源列表顯示用


## 規格

| 欄位名稱 | 資料型別 | 是否必填 | 說明 | 備註 |
|---|---|---|---|---|
| type | Int | 是 | 書源類型 | 固定帶入**99** |
| name | String | 是 | 書源名稱(次要) |  |
| alias | String | 否 | 書源別名(主要) | 為空值時，系統預設使用name |
| index | Int | 是 | 書源索引 | 建議使用 **1000** 以上的數字，避免與既有書源衝突 |
| version | Int | 是 | 版本 |  |
| enabled | Bool | 是 | 是否啟用 | 固定 true |
| imageURL | String | 否 | 書源封面連結 |  |
| sortIndex | Int | 是 | 排序用索引 | 不可獨立設定，需與 index 使用相同值 |
| allowSearch | Bool | 是 | 是否允許搜尋 |  |
| allowDownload | Bool | 是 | 是否允許下載 | 預設 true；章節內容被拆成分頁，且從目錄 URL 無法得知內容頁數時設 false |
| backgroundColor | [Double] | 否 | 背景顏色(RGBA) | RGBA 陣列；RGB 範圍 0 ~ 255，A 範圍 0 ~ 1 |


## 範例

```json
{
  "type": 99,
  "name": "書源1",
  "alias": "源1",
  "index": 1000,
  "version": 1,
  "enabled": true,
  "imageURL": "https://random.imagecdn.app/600/900",
  "sortIndex": 1000,
  "allowSearch": true,
  "allowDownload": true,
  "backgroundColor": [
    68,
    76,
    88,
    1
  ]
}
```


# 請求規則


## 規格

| 欄位名稱 | 資料型別 | 是否必填 | 說明 | 備註 |
|---|---|---|---|---|
| chapter | Object | 否 | 小說章節目錄請求配置 |  |
| content | Object | 是 | 小說章節正文請求配置 |  |
| search | Object | 是 | 搜尋請求配置 |  |
| category | Object | 是 | 標籤列表請求配置 |  |
| detail | Object | 是 | 小說基本資訊請求配置 | 若 inPage 為 true 時，表示與章節目錄在同個頁面，則無需額外配置chapter |
| home | Object | 是 | 首頁請求配置 | 若 inPage 為 true 時，表示與標籤在同個頁面，則無需額外配置category<br>(註：此處的 category 指的是 home 底下的物件，並非外層的category) |

註：以上每個物件都有下列兩個物件


### Config

| 欄位名稱 | 資料型別 | 是否必填 | 說明 | 備註 |
|---|---|---|---|---|
| charEncodingRawValue | Int | 否 | Payload 編碼格式，預設使用 UTF-8 | 0: GBK <br>1: BIG5, <br>2: UTF-8 |
| requestMethodRawValue | Int | 否 | 請求方式，預設使用 URLSession | 0: URLSession<br>1: WebView |
| websiteDataStoreRawValue | Int | 否 | WebView 資料儲存方式，預設 nonPersistent（cookie 不跨實例保存）| `requestMethodRawValue: 1` 時生效<br>0: nonPersistent（預設，每個 WebView 實例獨立，cookies/storage 不持久化）<br>1: persistent（持久化，cookies 跨實例共用）<br><br>需要 cookie 持久化（例如挑戰驗證通過後重用 session cookie）時帶 1 |
| bodySelector | String | 否 | 內容選擇器 |  |
| mainFrameOnly | Bool | 否 | true: 只注入主頁面<br>false: 主頁與 iframe 都注入 |  |
| injectionTime | Int | 否 | 0: 頁面開始前就注入<br>1: DOM 結構建完時注入 |  |

```json
{
  "config": {
    "charEncodingRawValue": 0,
    "requestMethodRawValue": 0
  }
}
```


### Setting

| 欄位名稱 | 資料型別 | 是否必填 | 說明 | 備註 |
|---|---|---|---|---|
| apiQueryValueFromPath | [String: String] | 否 | key為查詢參數鍵<br>value為替換值，將 apiPath 中的特定值替換成空字串，只留下所需資訊，並將此數值放入查詢參數的值裡<br> | 此參數通常用於標籤列表頁<br><br>`apiPath = "/novel/class/romance"<br>`<br>`apiQueryValueFromPath = [<br>   "type": "/novel/class/"<br>]`<br><br>`queryParams["type"] = "romance"`<br><br>將 apiPath 中的 /novel/class/ 替換成空字串，留下 “romance”並放入查詢參數裡 |
| apiQueryTypeRawValue | Int | 否 | 查詢類型 | 只有在 httpMethod == “GET” 時才生效<br>0: 無<br>1: 參數<br>2: 路徑 |
| apiQueryLangRawValue | Int | 否 | 查詢參數語系 | 0: 繁體中文<br>1: 簡體中文 |
| shouldIgnorePageIndex | Bool | 否 | apiPath 是否需要忽略分頁索引 | 預設 false，此參數通常用於標籤列表頁 |
| needsCustomUserAgent | Bool | 否 | 是否需要客製化User Agent | 預設 false |
| apiQueryParams | [String: String] | 否 | 其餘查詢參數 |  |
| ignorePageIndexByKeywords | [String] | 否 | apiPath 根據關鍵字判斷是否忽略分頁索引 | 此參數通常用於標籤列表頁 |
| apiQueryPageIndexKey | String | 否 | 分頁索引查詢鍵 | 此鍵的值為系統所帶入的分頁索引，通常用於標籤列表頁 |
| apiPath | String | 是 | 請求路徑 以 / 開頭 | /book |
| httpMethod | String | 是 | HTTP Method | 只支援”GET”與”POST” |
| apiPathReplacements | [String: String] | 否 | 替換 apiPath 字串 |  |
| finalURLReplacements | [String: String] | 否 | 替換組合後的完整連結 | 在 apiHost + apiPath 等所有路徑組合完成後，對最終 URL 做字串替換<br>key: 替換目標<br>value: 需被替換的文字<br><br>`"finalURLReplacements": {`<br>`  "/index.html": "/"`<br>`}` |
| needsURLEncoding | Bool | 否 | 請求是否需要進行URL編碼 | 預設 false<br>一般GET請求會自動編碼，無須將此參數設為ture，否則會雙重編碼 |
| timeout | Double | 否 | 請求超時秒數 | 預設45秒超時 |
| apiHost | String | 是 | 請求 host 不含 / 結尾 | http://127.0.0.1 |
| apiPageIndexPrefix | String | 否 | apiPath 索引前綴 | http://127.0.0.1/test/book/1.html<br>通常用於標籤列表頁 |
| extraApiPathSuffix | String | 否 | 額外後綴路徑，附加於動態關鍵字／分頁索引之後 | http://127.0.0.1/search/{keyword}.html<br>用於 apiQueryTypeRawValue=2 將關鍵字寫入路徑後，仍需在最尾再加一段固定後綴的場景 |
| apiPathPrefix | String | 否 | apiPath 前綴 | http://127.0.0.1/prefix/book |
| apiQueryKey | String | 否 | 查詢參數主要鍵 | 此參數通常用於搜尋，值為使用者自行輸入的字串，系統預設將此字串轉換成hex字串<br><br>queryParams[”apiQueryKey”] = hexString |
| httpHeaders | [String: String] | 否 | HTTP Header | value帶入“request_url”系統則會帶入此次請求的完整連結 |
| apiPathSuffix | String | 否 | apiPath 後綴 | http://127.0.0.1/book/suffix |
| delaySeconds | Double | 否 | 延遲請求秒數 | 預設不延遲 |
| randomDelaySeconds | [Double] | 否 | 隨機延遲秒數區間，固定 2 個元素 `[最小值, 最大值]` | 當 `delaySeconds == 0 && delaySeconds == nil && randomDelaySeconds.count == 2` 時，系統會以 `randomDelaySeconds` 生成的隨機值賦予 `delaySeconds` |
| jsScripts | [JSScript] | 否 | 注入 JavaScript 並將執行結果作為查詢參數 | 流程：注入 source 方法 → 呼叫 invoke 方法 → 結果以 key 為鍵存入查詢參數字典<br><br>適用於需要動態計算查詢參數的場景（如加密簽名、混淆值等）<br><br>**JSScript 結構：**<br>`key`: 查詢參數鍵名（例如 `"a"`）<br>`source`: 要注入的 JavaScript 原始碼（函式定義）<br>`invoke`: 要呼叫的 JavaScript 方法名稱（含參數，例如 `"myFunc(keywords)"`）<br><br>系統會自動將**搜尋關鍵字**帶入 invoke 中名為 `keywords` 的變數 |

```json
{
  "setting": {
    "apiHost": 0,
    "apiPath": 0,
    "apiPathReplacements": {
        "1.html": "" // 將路徑中的1.html替換成空字串
    },
    "apiPathPrefix": "/test",
    "apiPathSuffix": ".html",
    "apiPageIndexPrefix": "/",
    "extraApiPathSuffix": ".html",
    "timeout": 30,
    "httpMethod": "GET",
    "httpHeaders": {
      "Origin": "http://127.0.0.1",
      "Referer": "request_url"
    },
    "delaySeconds": 0,
    "randomDelaySeconds": [1, 3],
    "apiQueryKey": "searchkey",
    "apiQueryPageIndexKey": "",
    "apiQueryValueFromPath": "",
    "apiQueryLangRawValue": 0,
    "apiQueryTypeRawValue": 1,
    "apiQueryParams": {
      "searchtype": "all"
    },
    "needsURLEncoding": false,
    "needsCustomUserAgent": true,
    "shouldIgnorePageIndex": false,
    "ignorePageIndexByKeywords": ["author"],
    "jsScripts": [
      {
        "key": "a",
        "source": "function encode(s){ /* js source */ return result; }",
        "invoke": "encode"
      }
    ]
  }
}
```


## 範例


### Home

共有以下兩種格式

      1. 首頁資訊與標籤資訊都在同個頁面
```json
{
  "home": {
    "main": {
      "settings": {
        "apiHost": "https://127.0.1.com",
        "apiPath": "/novel/book_1.html",
        "httpMethod": "GET"
      },
      "config": {
        "charEncodingRawValue": 2
      }
    },
    "inPage": true
  }
}
```

      1. 首頁資訊與標籤資訊分別在不同頁面
```json
{
  "home": {
    "main": {
      "settings": {
        "apiHost": "https://127.0.1.com",
        "apiPath": "/novel/book_1.html",
        "httpMethod": "GET"
      },
      "config": {
        "charEncodingRawValue": 2
      }
    },
    "category": {
      "settings": {
        "apiHost": "https://127.0.1.com",
        "apiPath": "/class_1_1.html",
        "httpMethod": "GET"
      },
      "config": {
        "charEncodingRawValue": 2
      }
    },
    "inPage": false
  }
}
```


### Detail

```json
{
  "detail": {
    "main": {
      "settings": {
        "apiHost": "http://127.0.0.1",
        "httpMethod": "GET"
      },
      "config": {
        "charEncodingRawValue": 2
      }
    },
    "inPage": false, // 章節目錄是否在同個頁面
  }
}
```


### Chapter

共有以下兩種格式

      1. 無分頁
```json
{
  "chapter": {
    "main": {
      "settings": {
        "apiHost": "http://127.0.0.1",
        "httpMethod": "GET"
      },
      "config": {
        "charEncodingRawValue": 0,
        "requestMethodRawValue": 1
      }
    },
    "isPartial": false
  }
}
```

      1. 分頁模式
        1-1. main為主要請求
        1-2. pageCount為分頁數量請求，用於獲取分頁總數量
```json
{
  "chapter": {
    "main": {
      "settings": {
        "apiHost": "http://127.0.0.1",
        "httpMethod": "GET"
      },
      "config": {
        "charEncodingRawValue": 0,
        "requestMethodRawValue": 1
      }
    },
    "pageCount": {
      "settings": {
        "apiHost": "http://127.0.0.1",
        "httpMethod": "GET"
      },
      "config": {
        "charEncodingRawValue": 0,
        "requestMethodRawValue": 1
      }
    },
    "isPartial": true
  }
}
```


### Content

```json
{
  "content": {
    "settings": {
      "apiHost": "http://127.0.0.1",
      "httpMethod": "GET"
    },
    "config": {
      "charEncodingRawValue": 0,
      "requestMethodRawValue": 1
    }
  }
}
```


### Category

```json
{
  "category": {
    "main": {
      "settings": {
        "apiHost": "http://127.0.0.1",
        "apiPathPrefix": "",
        "apiPathSuffix": ".html",
        "apiPageIndexPrefix": "_",
        "apiPathReplacements": {
          "_1.html": ""
        },
        "needsCustomUserAgent": true,
        "httpMethod": "GET"
      },
      "config": {
        "charEncodingRawValue": 2
      }
    }
  }
}
```


### Search

```json
{
  "search": {
    "settings": {
      "apiHost": "http://127.0.0.1",
      "apiPath": "/modules/article/search.php",
      "apiQueryKey": "searchkey",
      "apiQueryParams": {
        "Submit": ""
      },
      "apiQueryLangRawValue": 1,
      "apiQueryTypeRawValue": 0,
      "needsCustomUserAgent": true,
      "httpMethod": "POST",
      "httpHeaders": {
        "Content-Type": "application/x-www-form-urlencoded"
      }
    },
    "config": {
      "charEncodingRawValue": 2,
      "requestMethodRawValue": 1
    }
  }
}
```


# 解析規則


## 規格

| 欄位名稱 | 資料型別 | 是否必填 | 說明 | 備註 |
|---|---|---|---|---|
| config | Object | 是 | 包含文字編碼、Cloudflare Challenge 配置 | 若書源無需進行 Cloudflare 挑戰驗證，則配置文字編碼即可 |
| category | Object | 是 | 解析標籤列表規則 |  |
| search | Object | 是 | 解析搜尋規則 |  |
| hostsToReplace | [String] | 否 | 用於將原始路徑替換成空字串，只保留Path | 範例：<br><br>原始值 = "http://127.0.0.1/book/test.html"<br><br>hostsToReplace = ["http://127.0.0.1"]<br><br>最終拿到的路徑為: /book/test.html |
| content | Object | 是 | 解析小說章節正文規則 |  |
| imageHost | String | 否 | 封面 host 不含 / 結尾 | 用於呈現小說封面 |
| home | Object | 是 | 解析首頁規則 | 通常包含兩個部分<br>1. 首頁推薦內容<br>2. 分類標籤 |
| chapter | Object | 是 | 解析小說章節目錄規則 |  |
| detail | Object | 是 | 解析小說基本資訊規則 |  |


### Config

| 欄位名稱 | 資料型別 | 是否必填 | 說明 | 備註 |
|---|---|---|---|---|
| challengeCookieDomains | [String] | 否 | 通過 Cloudflare 挑戰後，Cookie 所屬的域名 |  |
| waitForCookieIntervalSeconds | Double | 否 | 等待取得 Cookie 的間隔秒數 | waitForCookie 為 true & waitingForChallengeCount 有值時生效 |
| challengeKeywords | [String] | 否 | Cloudflare 挑戰關鍵字 用於判斷是否需要進行挑戰 | 根據 url 進行判斷 |
| authStorageTypeRawValue | Int | 否 | 驗證類型 | 若需驗證，通常固定帶0<br>0: Cookie<br>1: Local Storage<br>2. Cookie & Local Storage |
| waitForCookie | Bool | 否 | 是否等待取得 Cookie |  |
| challengeVerifiedKeywords | [String] | 否 | Cloudflare 挑戰完成後所獲得的 Key，用於判斷是否通過挑戰驗證 |  |
| waitForCookieMaxAttempts | Int | 否 | 等待取得 Cookie 的最大嘗試次數 | waitForCookie 為 true 時生效 |
| charEncodingRawValue | Int | 否 | Response 編碼格式，預設使用 UTF-8 | 0: GBK <br>1: BIG5, <br>2: UTF-8 |
| shouldReload | Bool | 否 | Cloudflare 挑戰驗證成功後，是否需要重新載入 |  |


### ParseElements

| 欄位名稱 | 資料型別 | 是否必填 | 說明 | 備註 |
|---|---|---|---|---|
| selectCssSelector2 | String | 否 | 指定元素，透過此參數解析 | 與 selectCssSelector 作用相同，差別在於若同個頁面有兩種不同的結構時，可透過此參數指定元素進行解析 |
| cssSelector2 | String | 否 | 解析用 | 與 cssSelector 作用相同，差別在於若同個頁面有兩種不同的結構時，可透過此參數進行解析 |
| selectIndex2 | Int | 否 | 透過此參數取得指定元素 | 與 selectIndex 作用相同，差別在於若同個頁面有兩種不同的結構時，可透過此參數指定元素 |
| selectCssSelector | String | 否 | 指定元素，透過此參數解析 |  |
| cssSelector | String | 是 | 解析用 |  |
| selectIndex | Int | 否 | 透過此參數取得指定元素 | 透過 cssSelector 解析後，若有多個結果，則使用此參數指定元素 |
| shouldUnescapeUnicode | Bool | 否 | 將 Unicode 跳脫序列轉為可讀文字 | 將 `\uXXXX` 格式的 Unicode 跳脫轉換為人類可讀的文字（如 `\u706b\u5f71` → `火影`）<br><br>執行順序：`shouldUnescapeUnicode` → `rawContentReplacements` → `shouldNormalise` |
| shouldNormalise | Bool | 否 | 開始解析前是否先正規化，補齊標籤 | 用於 HTML 結構不完整（如只有關閉標籤）的情況，正規化後再套用 cssSelector |
| rawContentReplacements | [String: [String]] | 否 | 解析前對原始內容進行文字替換 | key: 替換後的文字<br>value: 需被替換的文字陣列<br><br>主要用於清理無用文字、將非標準 HTML 轉為合法 HTML，再進行後續解析<br><br>`"rawContentReplacements": {`<br>`  "</": ["<\\ /"]`<br>`}` |
| isJSONFormat | Bool | 否 | 原始內容是否為 JSON 格式 | 為 true 時，系統會將此內容包裝成 HTML 列表格式，供後續 cssSelector 解析用<br><br>轉換規則：<br>- 物件/陣列遞迴展開為 `<ul>`，每個值/元素為 `<li>`<br>- 外層包裹 `<div id="json-content">`<br><br>原始 JSON：<br>`{"data": [{"title": "第一章", "ordernum": "1"}]}`<br><br>轉換後 HTML：<br>`<div id="json-content"><ul><li><ul><li><ul><li>第一章</li><li>1</li></ul></li></ul></li></ul></div>`<br><br>接著即可用 `cssSelector: "#json-content > ul > li"` 取列表 |
| jsonKey | String | 否 | 指定 JSON 鍵，只取該鍵對應的內容進行包裝 | `isJSONFormat` 為 true 時生效，預設不指定（整份 JSON 都會被包裝）<br><br>若 API 回傳 `{"status": 0, "data": [...]}`，設定 `"jsonKey": "data"` 只會包裝 `data` 陣列內容 |
| orderedKeys | [String] | 否 | 指定輸出的欄位集合及其排列順序 | `isJSONFormat` 為 true 時生效<br><br>只保留列出的鍵並依此順序輸出為 `<li>`，忽略其他鍵；不指定時輸出所有鍵（順序不保證與原 JSON 一致）<br><br>例：API 回 `{"ctype":"0","ordernum":"1","title":"第1章"}`，設定 `"orderedKeys": ["ordernum", "title"]` 會產生 `<ul><li>1</li><li>第1章</li></ul>`，然後用 `nth-of-type(1)` 取 ordernum、`nth-of-type(2)` 取 title |


#### JSON 回應處理

當某環節（常見於 `chapter`、`search`）回傳 JSON 而非 HTML 時，需在該環節的 `listElement` 同時設定 `isJSONFormat`、`jsonKey`、`orderedKeys` 協同運作；框架會先將 JSON 包裝成 HTML 列表（外層 `<div id="json-content">`），再照常套用 `cssSelector`。建議一併開啟 `shouldUnescapeUnicode`（還原 `\uXXXX`）與 `shouldNormalise`（補齊標籤）。

- `cssSelector` 一律從 `#json-content` 起算，列表通常為 `#json-content > ul > li`。
- 子元素（`pathElement`、`nameElement` 等）依 `orderedKeys` 的順序，用 `nth-of-type(n)` 取對應欄位，`actionValue` 用 `0` (text)。
- 若 JSON 僅提供 ID 而非完整路徑，用 `prefix` / `suffix`（見下方 ParseElement 表）拼回合法路徑。

完整範例（`chapter` 回傳 JSON，目錄項僅含 `ordernum`、`title`）：

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

> AI Agent 撰寫流程另見 `SKILL.md` 的〈JSON 回應處理〉小節。


### ParseElement

| 欄位名稱 | 資料型別 | 是否必填 | 說明 | 備註 |
|---|---|---|---|---|
| actionValue | Int | 是 | 解析類型 | 0: text<br>2: array<br>5: lastText<br>6: lastAttr<br>9: firstText<br>10: firstAttr<br>17: innerHtml<br>18: outerHtml<br>19: brTag<br>20: component<br>22: pTag |
| collectionSeparator | String | 否 | 分隔符 | actionValue為2時生效<br><br><br>`{<br>  "authorElement": {<br>    "actionValue": 2,<br>    "cssSelector": ".labelbox label",<br>    "collectionSeparator": ", ",<br>    "collectionActionValue": 0,<br>    "collectionValueIndexes": [<br>      0<br>    ]<br>  }<br>}`<br> |
| removeElements | [String] | 否 | 需被移除的元素 |  |
| separator | String | 否 | 分隔符 |  |
| replacements | [String: [String]] | 否 | 將解析完的原始文字替換成指定文字 | key: 替換目標<br>value: 需被替換的文字<br><br>`"replacements": {<br>    "歷史": ["曆史"]<br>}` |
| collectionValueIndexes | [Int] | 否 | 取得指定的索引值 | actionValue為2時生效 |
| invalidTags | [String] | 否 | 若原始HTML包含此參數的值，則會略過不處理 | actionValue為22時生效 |
| cssSelector | String | 是 | 解析用 |  |
| blankReplacements | [String: [String]] | 否 | 將解析完的原始文字替換成空白 | key: 模式，“0” = 預設, ”1” = 正則<br>value: 需被替換的文字或是正則<br><br>`"replaceTexts": {<br>    "0": ["http://127.0.0.1"]<br>}`<br><br>`"replaceTexts": {<br>    "1": ["(?m)^\\d+\\.\\s*"]<br>}` |
| prefix | String | 否 | 前綴 |  |
| suffix | String | 否 | 後綴 |  |
| parseableItems | [ParseableItem] | 否 | 解析非標準 HTML 字串 | actionValue為20時生效 |
| collectionActionValue | Int | 否 | 解析類型，同actionValue | actionValue為2時生效 |
| attributeKey | String | 否 | 要提取的屬性名稱 |  |


### ParseableItem

| 欄位名稱 | 資料型別 | 是否必填 | 說明 | 備註 |
|---|---|---|---|---|
| direction | Int | 是 | 裁切方向 | 1: 向前裁切<br>2. 向後裁切<br><br>若原始值為 ：<br>“<h2><a name="書源規則說明" class="md-header-anchor"></a></h2>”<br><br>配置如下：<br>`{<br>  "parseableItems": [<br>    {<br>      "value": "<a name=\"",<br>      "direction": 2<br>    },<br>    {<br>      "value": "\”",<br>      "direction": 1<br>    }<br>  ]<br>}`<br><br>結果為：書源規則說明 |
| value | String | 是 | 裁切目標 |  |


### Home

| 欄位名稱 | 資料型別 | 是否必填 | 說明 | 備註 |
|---|---|---|---|---|
| category | HomeObject | 否 | 解析標籤規則 |  |
| main | HomeObject | 是 | 解析首頁規則 |  |
| template | Int | 否 | 首頁佈局類型<br>0: grid<br>1: text with image<br>2: text with number | 預設使用grid |


### HomeObject

| 欄位名稱 | 資料型別 | 是否必填 | 說明 | 備註 |
|---|---|---|---|---|
| imageElement | ParseElement | 否 | 解析小說封面規則 |  |
| excludePaths | [String: String] | 否 | 需要被排除的路徑 | 若有不想呈現的標籤，可使用此參數進行排除 |
| authorElement | ParseElement | 否 | 解析小說作者規則 |  |
| imageRegex | String | 否 | 小說封面正則 |  |
| listElement | ParseElements | 否 | 解析小說元素集合 |  |
| chapterNameElement | ParseElement | 否 | 解析小說最新章節名稱規則 |  |
| removeElements | [String] | 否 | 需被移除的元素 |  |
| updatedTimeElement | ParseElement | 否 | 解析小說最後更新時間規則 |  |
| nameElement | ParseElement | 否 | 解析小說名稱規則 |  |
| pathRegex | String | 否 | 小說路徑正則 |  |
| categoryElement | ParseElement | 否 | 解析小說類別規則 |  |
| summaryElement | ParseElement | 否 | 解析小說簡介規則 |  |
| pathElement | ParseElement | 否 | 解析小說路經規則 |  |
| updatedTimeFormat | String | 否 | 小說最後更新時間格式 |  |
| nameRegex | String | 否 | 小說名稱正則 |  |


### Detail

| 欄位名稱 | 資料型別 | 是否必填 | 說明 | 備註 |
|---|---|---|---|---|
| imageElement | ParseElement | 否 | 解析小說封面規則 |  |
| removeElements | [String] | 否 | 需被移除的元素 |  |
| chapterPathElement | ParseElement | 否 | 解析小說章節目錄規則 | 此參數只有當章節目錄在其他頁面時才須配置 |
| chapterPathRegex | String | 否 | 小說章節目錄正則 |  |
| updatedTimeFormat | String | 否 | 小說最後更新時間格式 |  |
| updatedTimeElement | ParseElement | 否 | 解析小說最後更新時間規則 |  |
| authorElement | ParseElement | 否 | 解析小說作者規則 |  |
| imageRegex | String | 否 | 小說封面正則 |  |
| categoryElement | ParseElement | 否 | 解析小說分類規則 |  |
| chapterNameElement | ParseElement | 否 | 解析小說最新章節名稱規則 |  |
| summaryElement | ParseElement | 否 | 解析小說簡介規則 |  |


### Chapter

| 欄位名稱 | 資料型別 | 是否必填 | 說明 | 備註 |
|---|---|---|---|---|
| subListElement | ParseElements | 否 | 解析次要章節目錄元素集合規則 |  |
| nameElement | ParseElement | 是 | 解析章節名稱規則 |  |
| pathRegex | String | 否 | 章節路徑正則 |  |
| nameRegex | String | 否 | 章節名稱正則 |  |
| bookPathReplacements | [String: [String]] | 否 | 替換書籍路徑<br><br>bookPath指的是紅色這段<br>”http:127.0.0.1`/book/40459.html`" | shouldReplaceBookPath 為 true 時生效<br><br>若原始值為： ”/book/40459.html"<br><br>配置如下：<br>`{<br>  "bookPathReplacements": {<br>    "/": [".html"]<br>  }<br>}`<br><br>結果為：/book/40459/ |
| reversed | Bool | 否 | 是否需要反轉資料 |  |
| sortTypeRawValue | Int | 否 | 排序類型，受到 reversed 參數影響<br>升冪 = reversed == true<br>降冪 = reversed == false<br><br>0: 預設排序<br>1: 按照章節路徑升降冪<br>2. 按照章節名稱升降冪 |  |
| shouldReplaceBookPath | Bool | 否 | 是否需要替換書籍路徑 |  |
| partialReversed | Bool | 否 | 是否需要反轉分頁資料 |  |
| pageCountElement | ParseElement | 否 | 解析分頁數量規則，只有在分頁目錄時才需用到 |  |
| pathElement | ParseElement | 是 | 解析章節路徑規則 |  |
| listElement | ParseElements | 是 | 解析章節目錄元素集合規則 |  |
| removeElements | [String] | 否 | 需被移除的元素 |  |


### Content

| 欄位名稱 | 資料型別 | 是否必填 | 說明 | 備註 |
|---|---|---|---|---|
| nameElement2 | ParseElement | 否 | 解析章節名稱規則 |  |
| chapterPathRegex | String | 是 | 章節路徑正則 |  |
| contentRegex | String | 否 | 正文正則 |  |
| isLastPathComponentReplaced | Bool | 否 | 是否替換路徑的最後一個元件 |  |
| bookPathReplacements | [String: [String]] | 否 | 替換書籍路徑<br><br>bookPath指的是紅色這段<br>”http:127.0.0.1`/book/40459.html`" | shouldReplaceBookPath 為 true 時生效<br><br>若原始值為： ”/book/40459.html"<br><br>配置如下：<br>`{<br>  "bookPathReplacements": {<br>    "/": [".html"]<br>  }<br>}`<br><br>結果為：/book/40459/ |
| nameElement | ParseElement | 是 | 解析章節名稱規則 |  |
| lastPathComponentPrefix | String | 否 | 路徑的最後一個元件 前綴 | 只有在 isLastPathComponentReplaced 為 true 時生效<br><br>從章節頁URL → 去掉章節檔名 → 取出小說ID → 用小說ID拼出新的頁面路徑<br>var chapterURL = “http:127.0.0.1/book/40459/473624283.html”<br>chapterURL = chapterURL.deletingLastPathComponent()<br>(chapterURL = “http:127.0.0.1/book/40459/”)<br>let lastPathComponent = chapterURL.lastPathComponent<br>(lastPathComponent = “40459”)<br>finalPath = `lastPathComponentPrefix` + lastPathComponent + lastPathComponentSuffix + chapterPath |
| contentElement2 | ParseElement | 否 | 解析正文規則 |  |
| lastPathComponentSuffix | String | 否 | 路徑的最後一個元件 後綴 | 只有在 isLastPathComponentReplaced 為 true 時生效<br><br>從章節頁URL → 去掉章節檔名 → 取出小說ID → 用小說ID拼出新的頁面路徑<br>`var chapterURL = “http:127.0.0.1/book/40459/473624283.html”<br>chapterURL = chapterURL.deletingLastPathComponent()`<br>(chapterURL = “http:127.0.0.1/book/40459/”)*<br>*`let lastPathComponent = chapterURL.lastPathComponent`<br>(lastPathComponent = “40459”)*<br>*finalPath = lastPathComponentPrefix + lastPathComponent + `lastPathComponentSuffix` + chapterPath |
| contentElement | ParseElement | 是 | 解析正文規則 |  |
| nameRegex | String | 否 | 章節名稱正則 |  |
| lastChapterPathElement2 | ParseElement | 否 | 解析上一章節路徑規則 |  |
| nextChapterPathElement2 | ParseElement | 否 | 解析下一章節路徑規則 |  |
| shouldReplaceBookPath | Bool | 否 | 是否需要替換書籍路徑 |  |
| lastChapterPathElement | ParseElement | 是 | 解析上一章節路徑規則 |  |
| nextChapterPathElement | ParseElement | 是 | 解析下一章節路徑規則 |  |


### Category

| 欄位名稱 | 資料型別 | 是否必填 | 說明 | 備註 |
|---|---|---|---|---|
| updatedTimeFormat | String | 否 | 小說最後更新時間格式 |  |
| chapterNameElement | ParseElement | 否 | 解析小說最新章節名稱規則 |  |
| nameRegex | String | 否 | 小說名稱正則 |  |
| pageCount | Int | 否 | 共有幾頁 | 若此參數與 pageCountElement 為空值時，系統則會自行判斷是否需要加載下一頁，當獲取的結果數量不等於 pageSize 時停止加載 |
| listElement | ParseElements | 是 | 解析小說元素集合 |  |
| categoryElement | ParseElement | 否 | 解析小說分類規則 |  |
| pageCountElement | ParseElement | 否 | 解析分頁數量規則 |  |
| summaryElement | ParseElement | 否 | 解析小說簡介規則 |  |
| pageSize | Int | 是 | 一頁幾筆 |  |
| pathRegex | String | 否 | 小說路徑正則 |  |
| imageRegex | String | 否 | 小說封面正則 |  |
| removeElements | [String] | 否 | 需被移除的元素 |  |
| nameElement | ParseElement | 是 | 解析小說名稱規則 |  |
| imageElement | ParseElement | 否 | 解析小說封面規則 |  |
| pathElement | ParseElement | 是 | 解析小說路徑規則 |  |
| updatedTimeElement | ParseElement | 否 | 解析小說最後更新時間規則 |  |
| authorElement | ParseElement | 否 | 解析小說作者規則 |  |
| subListElement | ParseElements | 否 | 解析小說次要元素集合 |  |


### Search

| 欄位名稱 | 資料型別 | 是否必填 | 說明 | 備註 |
|---|---|---|---|---|
| mNameElement2 | ParseElement | 否 | 解析小說名稱規則 (多筆) | 多筆資料時使用，此參數用於獲取備用小說名稱，與 mNameElement 作用相同 |
| imageRegex | String | 否 | 小說封面正則 |  |
| sImageElement | ParseElement | 否 | 解析小說封面規則 (單筆) | 單筆資料時使用 |
| mImageElement | ParseElement | 否 | 解析小說封面規則 (多筆) | 多筆資料時使用 |
| nameRegex | String | 否 | 小說名稱正則 |  |
| subListElement | ParseElements | 否 | 解析小說次要元素集合 |  |
| mPathElement2 | ParseElement | 否 | 解析小說路徑規則 (多筆) | 多筆資料時使用，此參數用於獲取備用小說路徑，與 mPathElement 作用相同 |
| mPathElement | ParseElement | 否 | 解析小說路徑規則 (多筆) | 多筆資料時使用 |
| sImageElement2 | ParseElement | 否 | 解析小說封面規則 (單筆) | 單筆資料時使用，此參數用於獲取備用封面，與 sImageElement 作用相同 |
| sPathElement | ParseElement | 否 | 解析小說路徑規則 (單筆) | 單筆資料時使用 |
| mImageElement2 | ParseElement | 否 | 解析小說封面規則 (多筆) | 多筆資料時使用，此參數用於獲取備用封面，與 mImageElement 作用相同 |
| pathRegex | String | 否 | 小說路徑正則 |  |
| cssSelector | String | 否 | 解析用 | 用於搜尋結果僅有一筆且自動跳轉至詳細頁時，<br>指定代表該筆結果的根節點，<br>使後續解析流程與多筆搜尋結果保持一致 |
| sRemoveElements | [String] | 否 | 需被移除的元素 (單筆) | 單筆資料時使用 |
| sNameElement | ParseElement | 否 | 解析小說名稱規則 (單筆) | 單筆資料時使用 |
| listElement | ParseElements | 是 | 解析小說元素集合 |  |
| removeElements | [String] | 否 | 需被移除的元素 |  |
| mNameElement | ParseElement | 否 | 解析小說名稱規則 (多筆) | 多筆資料時使用 |


## 範例


### Config

```json
{
  "config": {
    "shouldReload": true,
    "charEncodingRawValue": 0,
    "authStorageTypeRawValue": 0,
    "challengeKeywords": [
      "challenges.cloudflare"
    ],
    "challengeVerifiedKeywords": [
      "cf_clearance"
    ]
  }
}
```


### Home

```json
{
  "home": {
    "template": 2,
    "main": {
      "pathRegex": "/book.*\\.(htm|html)$",
      "listElement": {
        "selectIndex": 0,
        "cssSelector": "#article_list_content",
        "selectCssSelector": "li"
      },
      "pathElement": {
        "actionValue": 10,
        "cssSelector": "h3 > a",
        "attributeKey": "href"
      },
      "nameElement": {
        "actionValue": 0,
        "cssSelector": "h3 > a"
      },
      "authorElement": {
        "actionValue": 2,
        "cssSelector": ".labelbox label",
        "collectionSeparator": ", ",
        "collectionActionValue": 0,
        "collectionValueIndexes": [
          0
        ]
      },
      "summaryElement": {
        "actionValue": 0,
        "cssSelector": ".ellipsis_2"
      }
    },
    "category": {
      "pathRegex": "/novels/class/\\d+\\.(htm|html)$",
      "listElement": {
        "cssSelector": "div.weekl_yrank ul li"
      },
      "pathElement": {
        "actionValue": 10,
        "cssSelector": "a",
        "attributeKey": "href"
      },
      "nameElement": {
        "actionValue": 0,
        "cssSelector": "a"
      }
    }
  }
}
```


### Detail

```json
{
  "detail": {
    "chapterPathRegex": ".*\\/book\\/[0-9]+.*|\\/chapters.*\\.(htm|html)$",
    "removeElements": [
      "script",
      "style",
      "link",
      "header",
      "div.foot",
      "div.modbg",
      "div.modelbg",
      "div.leftmenu"
    ],
    "authorElement": {
      "actionValue": 10,
      "cssSelector": "meta[property=og:novel:author]",
      "attributeKey": "content"
    },
    "summaryElement": {
      "actionValue": 2,
      "cssSelector": "div.navtxt > p",
      "collectionSeparator": "\n"
    },
    "categoryElement": {
      "actionValue": 10,
      "cssSelector": "meta[property=og:novel:category]",
      "attributeKey": "content"
    },
    "chapterPathElement": {
      "actionValue": 10,
      "cssSelector": "a.btn.more-btn",
      "attributeKey": "href"
    },
    "updatedTimeElement": {
      "actionValue": 10,
      "cssSelector": "meta[property=og:novel:update_time]",
      "attributeKey": "content"
    }
  }
}
```


### Chapter

```json
{
  "chapter": {
    "reversed": true,
    "pathRegex": ".*/txt/[0-9]+.*",
    "removeElements": [
      "script",
      "style",
      "meta",
      "link",
      "header",
      "onclick",
      "div.foot",
      "div.modbg",
      "div.modelbg",
      "div.leftmenu",
      "div.catalog:not([id])"
    ],
    "listElement": {
      "cssSelector": "ul > li"
    },
    "pathElement": {
      "actionValue": 10,
      "cssSelector": "a",
      "attributeKey": "href"
    },
    "nameElement": {
      "actionValue": 0,
      "cssSelector": "a",
      "blankReplacements": {
        "1": [
          "(?m)^\\d+\\.\\s*"
        ]
      }
    }
  }
}
```


### Content

```json
{
  "content": {
    "chapterPathRegex": ".*/txt/[0-9]+.*",
    "nameElement": {
      "actionValue": 0,
      "cssSelector": "div.txtnav > h1",
      "blankReplacements": {
        "1": [
          "(?m)^\\d+.*章\\s",
          "(?m)^\\d+\\.\\s*"
        ]
      }
    },
    "contentElement": {
      "separator": "\n",
      "invalidTags": [
        "<h",
        "<div",
        "</div",
        "<span",
        "</span",
        "<script",
        "</script",
        "⊥"
      ],
      "replaceTags": [
        "<p>",
        "</p>",
        "<br>",
        "</br>",
        "<br />"
      ],
      "actionValue": 22,
      "cssSelector": "div.txtnav",
      "blankReplacements": {
        "1": [
          "\\n無一錯.*?！",
          "\\n无一错.*?！"
        ]
      }
    },
    "lastChapterPathElement": {
      "actionValue": 10,
      "cssSelector": ".page1 a",
      "attributeKey": "href"
    },
    "nextChapterPathElement": {
      "actionValue": 6,
      "cssSelector": ".page1 a",
      "attributeKey": "href"
    }
  }
}
```


### Category

```json
{
  "category": {
    "pageSize": 50,
    "pathRegex": "/book.*\\.(htm|html)$",
    "removeElements": [
      "script",
      "style",
      "meta",
      "link",
      "header",
      "onclick",
      "div.foot",
      "div.modbg",
      "div.modelbg",
      "div.leftmenu",
      "div.catalog:not([id])"
    ],
    "listElement": {
      "selectIndex": 0,
      "cssSelector": "#article_list_content",
      "selectCssSelector": "li"
    },
    "pathElement": {
      "actionValue": 10,
      "cssSelector": "h3 > a",
      "attributeKey": "href"
    },
    "nameElement": {
      "actionValue": 0,
      "cssSelector": "h3 > a"
    },
    "authorElement": {
      "actionValue": 2,
      "cssSelector": ".labelbox label",
      "collectionSeparator": ", ",
      "collectionActionValue": 0,
      "collectionValueIndexes": [
        0
      ]
    },
    "summaryElement": {
      "actionValue": 0,
      "cssSelector": ".ellipsis_2"
    },
    "categoryElement": {
      "actionValue": 2,
      "cssSelector": ".labelbox label",
      "collectionSeparator": ", ",
      "collectionActionValue": 0,
      "collectionValueIndexes": [
        1
      ]
    }
  }
}
```


### Search

```json
{
  "search": {
    "pathRegex": ".*\\/book\\/[0-9]+.*|\\/article.*\\.(htm|html)$",
    "listElement": {
      "selectIndex": 0,
      "cssSelector": ".newbox ul",
      "selectCssSelector": "li"
    },
    "mPathElement": {
      "actionValue": 10,
      "cssSelector": "h3 > a",
      "attributeKey": "href"
    },
    "mNameElement": {
      "actionValue": 0,
      "cssSelector": "h3 > a"
    },
    "mImageElement": {
      "actionValue": 10,
      "cssSelector": "img",
      "attributeKey": "data-src"
    },
    "mImageElement2": {
      "actionValue": 10,
      "cssSelector": "img",
      "attributeKey": "src"
    }
  }
}
```


# 完整範例


### 規格

| 欄位名稱 | 資料型別 | 是否必填 | 說明 | 備註 |
|---|---|---|---|---|
| info | Object | 是 | 書源基本資訊 |  |
| data | Object | 是 | 書源請求、解析規則 |  |


### 範例

```json
{
  "info": {
    "type": 99,
    "name": "測試書源",
    "index": 1000,
    "version": 1,
    "enabled": true,
    "imageURL": "https://random.imagecdn.app/600/900",
    "sortIndex": 1000,
    "allowSearch": true,
    "allowDownload": true
  },
  "data": {
    "searchIndex": 1000, // 搜尋排序索引，數值越小越前面
    "request": {
      "home": {
        "main": {
          "settings": {
            "apiHost": "http://127.0.0.1",
            "httpMethod": "GET"
          },
          "config": {
            "charEncodingRawValue": 2
          }
        },
        "inPage": true
      },
      "detail": {
        "main": {
          "settings": {
            "apiHost": "http://127.0.0.1",
            "httpMethod": "GET"
          },
          "config": {
            "charEncodingRawValue": 2
          }
        },
        "inPage": true
      },
      "content": {
        "settings": {
          "apiHost": "http://127.0.0.1",
          "apiPathPrefix": "/",
          "httpMethod": "GET"
        },
        "config": {
          "charEncodingRawValue": 2
        }
      },
      "category": {
        "main": {
          "settings": {
            "apiHost": "http://127.0.0.1",
            "apiPageIndexPrefix": "/",
            "needsCustomUserAgent": true,
            "httpMethod": "GET"
          },
          "config": {
            "charEncodingRawValue": 2
          }
        }
      },
      "search": {
        "settings": {
          "apiHost": "http://127.0.0.1",
          "apiPath": "/s",
          "apiQueryLangRawValue": 0,
          "apiQueryTypeRawValue": 2,
          "needsCustomUserAgent": true,
          "httpMethod": "GET"
        },
        "config": {
          "charEncodingRawValue": 2
        }
      }
    },
    "response": {
      "config": {
        "shouldReload": true,
        "charEncodingRawValue": 0,
        "authStorageTypeRawValue": 0,
        "challengeKeywords": [
          "challenges.cloudflare"
        ],
        "challengeVerifiedKeywords": [
          "cf_clearance"
        ]
      },
      "hostsToReplace": [
        "http://127.0.0.1"
      ],
      "imageHost": "http://127.0.0.1",
      "home": {
        "template": 0,
        "main": {
          "pathRegex": "^/n/.*",
          "listElement": {
            "selectIndex": 0,
            "cssSelector": "div.main > div.container > ul:nth-of-type(1)",
            "selectCssSelector": "li"
          },
          "pathElement": {
            "actionValue": 10,
            "cssSelector": "div.novel-item-title > a",
            "attributeKey": "href"
          },
          "nameElement": {
            "actionValue": 0,
            "cssSelector": "div.novel-item-title > a"
          },
          "imageElement": {
            "actionValue": 10,
            "cssSelector": "div.novel-item-thumbnail > img",
            "attributeKey": "src"
          }
        },
        "category": {
          "pathRegex": "^/c/.*",
          "listElement": {
            "selectIndex": 0,
            "cssSelector": "div.header > div.container > ul.nav.menu",
            "selectCssSelector": "li"
          },
          "pathElement": {
            "actionValue": 10,
            "cssSelector": "a",
            "attributeKey": "href"
          },
          "nameElement": {
            "actionValue": 0,
            "cssSelector": "a"
          }
        }
      },
      "detail": {
        "imageElement": {
          "actionValue": 10,
          "cssSelector": "div.novel-detail > div.thumbnail > img",
          "attributeKey": "src"
        },
        "authorElement": {
          "actionValue": 0,
          "cssSelector": "div.novel-detail > div.info-wrap > div.info > span.author > a"
        },
        "summaryElement": {
          "separator": "\n",
          "actionValue": 19,
          "cssSelector": "div.novel-detail > div.info-wrap > div.description"
        },
        "categoryElement": {
          "actionValue": 0,
          "cssSelector": "div.novel-detail > div.state > table > tbody > tr:nth-of-type(1) > td:nth-of-type(2)"
        },
        "updatedTimeElement": {
          "actionValue": 0,
          "cssSelector": "div.novel-detail > div.state > table > tbody > tr:nth-of-type(4) > td:nth-of-type(2)"
        }
      },
      "chapter": {
        "reversed": true,
        "pathRegex": "^/n/.*/.*",
        "nameRegex": "^(?!.*錯誤章).*",
        "listElement": {
          "selectIndex": 0,
          "cssSelector": "ul.nav.chapter-list",
          "selectCssSelector": "li"
        },
        "pathElement": {
          "actionValue": 10,
          "cssSelector": "a",
          "attributeKey": "href"
        },
        "nameElement": {
          "actionValue": 0,
          "cssSelector": "a",
          "blankReplacements": {
            "1": [
              "(?m)^\\d+\\.\\s*"
            ]
          }
        }
      },
      "content": {
        "pathRegex": "^/n/.*/.*",
        "nameElement": {
          "actionValue": 0,
          "cssSelector": "div.chapter-detail.style-left > div.name",
          "blankReplacements": {
            "0": [
              "《",
              "》"
            ],
            "1": [
              "",
              "(?m)^\\d+\\.\\s*"
            ]
          }
        },
        "contentElement": {
          "separator": "\n",
          "actionValue": 19,
          "cssSelector": "div.chapter-detail.style-left > div.content"
        },
        "lastChapterPathElement": {
          "actionValue": 10,
          "cssSelector": "ul.nav.chapter-nav > li > a.prev-chapter",
          "attributeKey": "href",
          "blankReplacements": {
            "0": [
              "http://127.0.0.1"
            ]
          }
        },
        "nextChapterPathElement": {
          "actionValue": 6,
          "cssSelector": "ul.nav.chapter-nav > li > a.next-chapter",
          "attributeKey": "href",
          "blankReplacements": {
            "0": [
              "http://127.0.0.1"
            ]
          }
        }
      },
      "category": {
        "pageSize": 70,
        "pathRegex": "^/n/.*",
        "listElement": {
          "selectIndex": 0,
          "cssSelector": "ul.nav.novel-list.style-default",
          "selectCssSelector": "li"
        },
        "pathElement": {
          "actionValue": 10,
          "cssSelector": "div.novel-item-cover-wrapper > a",
          "attributeKey": "href"
        },
        "nameElement": {
          "actionValue": 0,
          "cssSelector": "div.novel-item-cover-wrapper > a > div.novel-item-title"
        },
        "imageElement": {
          "actionValue": 10,
          "cssSelector": "div.novel-item-cover-wrapper > a > div.novel-item-thumbnail > img",
          "attributeKey": "src"
        },
        "authorElement": {
          "actionValue": 0,
          "cssSelector": "div.novel-item-author > a"
        },
        "chapterNameElement": {
          "actionValue": 0,
          "cssSelector": "div.novel-item-newest-chapter > a"
        },
        "updatedTimeElement": {
          "actionValue": 0,
          "cssSelector": "div.novel-item-date"
        }
      },
      "search": {
        "pathRegex": "^/n/.*",
        "listElement": {
          "selectIndex": 0,
          "cssSelector": "ul.nav.novel-list.style-default",
          "selectCssSelector": "li"
        },
        "mPathElement": {
          "actionValue": 10,
          "cssSelector": "div.novel-item-cover-wrapper > a",
          "attributeKey": "href"
        },
        "mNameElement": {
          "actionValue": 0,
          "cssSelector": "div.novel-item-cover-wrapper > a > div.novel-item-title"
        },
        "mImageElement": {
          "actionValue": 10,
          "cssSelector": "div.novel-item-cover-wrapper > a > div.novel-item-thumbnail img",
          "attributeKey": "src"
        }
      }
    }
  }
}
```
