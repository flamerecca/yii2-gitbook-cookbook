# 處理文字（Processing text） {#processing-text}

When implementing post or article publishing, it's very important to choose right tool for the job. Common approach is to use WYSIWYG \(what you see is what you get\) editor that produces HTML but that has significant cons. The most prominent con is that it's easy to break website design and to produce excessive and ugly HTML. The pro is that it's quite natural for people worked with MS Word or alike text processors.

不過現在，我們已經一些標記語言，像是 markdown。 While being very simple, it has everything to do basic text formatting: emphasis, hyperlinks, headers, tables, code blocks etc. For tricky cases it still accepts HTML.

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

### Where to do it {#where-to-do-it}

* The library used to convert markdown to HTML is fast so processing right in the view could be OK.
* Result could be saved into separate field in database or cached for extra performance. Both could be done in
  `afterSave`
  method of the model. Note that in case of database field we can't save processed HTML to the same field because of the need to edit original.

## 其他選擇 {#alternatives}

Markdown 並不是唯一的標記語言，對其他標記語言有興趣的話可以[參閱維基百科](https://zh.wikipedia.org/wiki/%E8%BD%BB%E9%87%8F%E7%BA%A7%E6%A0%87%E8%AE%B0%E8%AF%AD%E8%A8%80)。

