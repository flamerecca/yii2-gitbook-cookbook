# 加入 搜尋引擎最佳化標籤（Adding SEO tags） {#adding-seo-tags}

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

There are no dedicated view parameters for`keywords`or`description`. Since these are meta tags and you should set them by`registerMetaTag()`method.

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

All registered meta tags will be rendered inside layout in place of`$this->head()`call.

Note that when the same tag is registered twice it's rendered twice. For example, description meta tag that is registered both in layout and a view is rendered twice. Usually it's not good for SEO. In order to avoid it you can specify key as the second argument of`registerMetaTag`:

```php
$this->registerMetaTag([
    'name' => 'description',
    'content' => 'Description 1',
], 'description');

$this->registerMetaTag([
    'name' => 'description',
    'content' => 'Description 2',
], 'description');
```

In this case later second call will overwrite first call and description will be set to "Description 2".

