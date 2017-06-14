# 偏好網址（Canonical URLs） {#canonical-urls}

Because of many reasons the same or nearly the same page content often is accessible via multiple URLs. There are valid cases for it such as viewing an article within a category and not so valid ones. For end user it doesn't really matter much but still it could be a problem because of search engines because either you might get wrong URLs preferred or, in the worst case, you might get penalized.

One way to solve it is to mark one of URLs as a primary or, as it called, canonical, one you may use`<link rel="canonical"`tag in the page head.

> 備註：這邊假設已經[啟用pretty URLs](/enable-pretty-urls.md)

現在，假設我們有兩個網址：

* `http://example.com/item1`
* `http://example.com/item2`

這兩個網址的內容幾乎是一樣的。

我們的目標是把第一個網址設置成偏好網址。另一個網址還是能讓使用者連結。The process of adding SEO meta-tags is descibed in "[adding SEO tags](https://yii2-cookbook.readthedocs.io/adding-seo-tags/)" recipe. Adding`<link rel="canonical"`is very similar. In order to do it from controller action you may use the following code:

```php
\Yii::$app->view->registerLinkTag(['rel' => 'canonical', 'href' => Url::to(['item1'], true)]);
```

In order to achieve the same from inside the view do it as follows:

```php
$this->registerLinkTag(['rel' => 'canonical', 'href' => Url::to(['item1'], true)]);
```

> Note: It is necessary to use absolute paths instead of relative ones.

As an alternative to`Url::to()`you can use`Url::canonical()`such as

```php
$this->registerLinkTag(['rel' => 'canonical', 'href' => Url::canonical()]);
```

The line above could be added to layout.`Url::canonical()`generates the tag based on current controller route and action parameters \(the ones present in the method signature\).

## 其他資料 {#see-also}

* [Google 說明 使用標準網址](https://support.google.com/webmasters/answer/139066?hl=zh-Hant)
  .



