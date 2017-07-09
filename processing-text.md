# 處理文字（Processing text） {#processing-text}

在實做貼文介面時，使用適合的編輯工具是很重要的。一般的作法是使用所見即所得（what you see is what you get，WYSIWYG）編輯器，來產生 HTML 格式的輸出。這種作法的好處是，對常用 Word 或者類似的文字編輯器的人來說，介面會很自然。但是這種作法有一些缺點，最明顯的是容易產生多餘而且難看的HTML內容，破壞網頁原本的設計。

不過現在，我們已經有一些標記語言，像是 markdown。雖然語言結構很簡單，不過已經滿足基本文字格式的所有需求：粗體、超連接、標題、表格、程式碼區塊……等等。For tricky cases it still accepts HTML.

## 將 markdown 轉換成 HTML {#converting-markdown-to-html}

### 作法 {#how-to-do-it}

Markdown helper 的用法非常簡單：

```php
$myHtml = Markdown::process($myText); // 使用原始的 markdown 
$myHtml = Markdown::process($myText, 'gfm'); // 使用 github 版本的 markdown
$myHtml = Markdown::process($myText, 'extra'); // 使用 markdown extra
```

### 確認輸出安全性 {#how-to-secure-output}

因為 markdown 允許使用純 HTML，所以有一些危險性（比方說XSS攻擊等）。所以我們會建議使用 HTMLPurifier 再處理一次輸出：

```php
$safeHtml = HtmlPurifier::process($unsafeHtml);
```

### 處理的位置 {#where-to-do-it}

* The library used to convert markdown to HTML is fast so processing right in the view could be OK.
* Result could be saved into separate field in database or cached for extra performance. Both could be done in
  `afterSave`
  method of the model. Note that in case of database field we can't save processed HTML to the same field because of the need to edit original.

## 其他選擇 {#alternatives}

Markdown 並不是唯一的標記語言，對其他標記語言有興趣的話可以[參閱維基百科](https://zh.wikipedia.org/wiki/轻量级标记语言)。

