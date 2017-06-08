# 加入搜尋引擎最佳化標籤（Adding SEO tags） {#adding-seo-tags}

Organic search is an excellent traffic source. In order to get it you have to make a lot of small steps to improve your project.

One of such steps is to provide different meta tags for different pages. It will improve your site organic search appearance and may result in better ranking.

Let's review how to add SEO-related metadata to your pages.

## Title {#title}

設置title非常簡單，在控制器（controller）裡面是：

```php
\Yii::$app->view->title = 'title set inside controller';
```

在視圖（view）裡面則是：

```php
$this->title = 'Title from view';
```

> 備註：Setting`$this->title`in layout will override value which is set for concrete view so don't do it.

有預設 title 通常是個好主意，所以可以在layout裡面這樣寫：

```php
$this->title = $this->title ? $this->title : '預設 title';
```

## Description 和 Keywords {#description-and-keywords}

Yii內`keywords`和`description` 沒有對應的參數。因為這兩個在網頁上都是meta標籤，所以我們應該使用`registerMetaTag()`函式。

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

這邊需要注意的是，如果我們對同一個標籤登記不只一次，那麼每一次登記的內容都會出現在網頁中。舉例來說，如果我們layout裡面登記了一個description，view裡面又登記了一個description，那網頁會出現兩個description。這樣通常對 SEO 不好。要避免這個狀況，我們可以在`registerMetaTag()`的第二個參數，加上對應的key：

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

這樣的話，前面的 description 會被蓋過，網頁的description 會顯示為「網頁描述二」。

