# 加入搜尋引擎最佳化標籤（Adding SEO tags） {#adding-seo-tags}

自然搜尋（organic search）是流量的重要來源。要最佳化這一部份，我們需要作到很多事情。

其中一個該做的事情，是在每個網頁上面加上不同的 meta tag。這會提供搜尋引擎一些網頁的資訊，可能會讓網站的自然搜尋表現更好。

以下簡述如何在網站裡面，加上搜尋相關的metadata。

{% youtube src="https://www.youtube.com/watch?v=9bZkp7q19f0" %}{% endyoutube %}

## Title {#title}

設置title非常簡單，在控制器（controller）裡面是：

```php
\Yii::$app->view->title = 'title set inside controller';
```

在視圖（view）裡面則是：

```php
$this->title = 'Title from view';
```

> 備註：在layout裡面設置`$this->title`，會蓋過所有的頁面，所以不要這麼做。

有預設 title 通常是個好主意，所以可以在layout裡面這樣寫：

```php
$this->title = $this->title ? $this->title : '預設標題';
```

## Description 和 Keywords {#description-and-keywords}

Yii 內`keywords`和`description` 沒有對應的參數。因為這兩個在網頁上都是meta標籤，所以我們可以使用`registerMetaTag()`函式。

在控制器（controller）裡面是：

```php
\Yii::$app->view->registerMetaTag([
    'name' => 'description',
    'content' => 'Description set inside controller',
]);
\Yii::$app->view->registerMetaTag([
    'name' => 'keywords',
    'content' => 'Keywords set inside controller',
]);
```

在視圖（view）裡面是：

```php
$this->registerMetaTag([
    'name' => 'description',
    'content' => 'Description set inside view',
]);
$this->registerMetaTag([
    'name' => 'keywords',
    'content' => 'Keywords set inside view',
]);
```

所有登記的 meta tag 會在`$this->head()`呼叫時產生。

這邊需要注意的是，如果我們對同一個標籤登記不只一次，那麼每一次登記的內容都會出現在網頁中。舉例來說，如果我們layout裡面登記了一個description，view裡面又登記了一個 description，那網頁會出現兩個 description。這樣通常對 SEO 不好。要避免這個狀況，我們可以在`registerMetaTag()`的第二個參數，加上對應的key：

```php
$this->registerMetaTag([
    'name' => 'description',
    'content' => '網頁描述一',
], 'description');

$this->registerMetaTag([
    'name' => 'description',
    'content' => '網頁描述二',
], 'description');
```

這樣的話，前面的 description 會被蓋過，網頁的description 會是「網頁描述二」。

## 其他資料 {#see-also}

* [中文維基-有機搜尋](https://zh.wikipedia.org/wiki/有機搜尋)



