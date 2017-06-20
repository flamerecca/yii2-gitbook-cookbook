# pretty URLs 分頁（Pagination pretty URLs） {#pagination-pretty-urls}

For example we can render our site content by GridView. If there are a lot of content rows we use pagination. And it is necessary to provide GET request for every pagination page. Thus search crawlers can index our content. We also need our URLs to be pretty. Let's do it.

## 初始狀態 {#initial-state}

假設我們最開始的網址是`http://example.com/schools/schoolTitle`。`schoolTitle`是程式碼`title`參數的輸入資料。這個作法的細節可以參考[這邊](/urls-variable-number-of-parameters.md)。

在 Yii config 檔裡面：

```php
$config = [
    // ...
    'components' => [
        // ...
        'urlManager' => [
            'showScriptName' => false,
            'enablePrettyUrl' => true,
            'rules' => [
              'schools/<title:\w+>' => 'site/schools',
            ],
        ],
        // ...
    ],
    // ...
```

We decided to add a on the page`http://example.com/schools/schoolTitle`加上 GridView。

## pretty URLs 分頁 {#pagination-pretty-urls_1}

上面完成之後，當我們點擊下一頁的網址，會長得像是`http://example.com/schools/schoolTitle?page=2`。我們希望分頁網址長得像是`http://example.com/schools/schoolTitle/2`。

所以我們再加入一條 urlManager 規則，這規則要比原先的規則**權限更高**，如下：

```php
$config = [
        // ...
        'urlManager' => [
            // ...
            'rules' => [
              'schools/<title:\w+>/<page:\d+>' => 'site/schools', // 新規則
              'schools/<title:\w+>' => 'site/schools',
            ],
        ],
        // ...
```



