# 偏好網址（Canonical URLs） {#canonical-urls}

因為各種原因，我們可能會讓多個不同網址，連接到幾乎相同或者完全相同的頁面，像是列舉某個種類的文章列表。這件事情對使用者來說幾乎沒有影響。但是對於搜尋引擎來說，這還是有些問題。因為可能會導致錯誤的網址出現在搜尋結果內，甚至最差的情況下，會被搜尋引擎認為是違規行為而被懲罰。 

其中一個解決方法，是標記其中一個網址為偏好網址（Canonical URLs），也就是主要的網址。One way to solve it is to mark one of URLs as a primary or, as it called, canonical, 我們可以在網頁標頭使用`<link rel="canonical"`標籤來達到這個效果。 

> 備註：這邊假設已經[啟用pretty URLs](/enable-pretty-urls.md)功能

現在，假設我們有兩個網址：

* `http://example.com/item1`
* `http://example.com/item2`

這兩個網址的內容幾乎是一樣的。

我們的目標是把第一個網址設置成偏好網址。另一個網址還是能讓使用者連結。這邊的程序跟「[加入搜尋引擎最佳化標籤](/adding-seo-tags.md) 」的方式類似。只是變成加入`<link rel="canonical"`。

要在控制器（controller）裡面加入該標籤，我們使用的程式碼如下：

```php
\Yii::$app->view->registerLinkTag(['rel' => 'canonical', 'href' => Url::to(['item1'], true)]);
```

要在視圖（view）裡面加入該標籤，我們使用的程式碼如下：

```php
$this->registerLinkTag(['rel' => 'canonical', 'href' => Url::to(['item1'], true)]);
```

> 備註：這邊必須使用絕對路徑，不可以使用相對路徑。

除了用`Url::to()`，我們也可以用`Url::canonical()`：

```php
$this->registerLinkTag(['rel' => 'canonical', 'href' => Url::canonical()]);
```

上面的程式碼也可以加在layout裡面。`Url::canonical()`generates the tag based on current controller route and action parameters \(the ones present in the method signature\).

## 其他資料 {#see-also}

* [Google 說明 使用標準網址](https://support.google.com/webmasters/answer/139066?hl=zh-Hant)
  .



