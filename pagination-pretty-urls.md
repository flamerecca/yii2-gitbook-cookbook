# pretty URLs 分頁（Pagination pretty URLs） {#pagination-pretty-urls}

假設我們使用 GridView 來展示我們的網頁資料。如果有很多資料的話，我們會使用分頁。使用 GET 參數來提供分頁資訊是很重要的，因為這樣搜尋網站才能對網頁的資料建立索引。除此之外，我們也希望網址看起來漂亮。

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

我們希望可以在`http://example.com/schools/schoolTitle`加上 GridView。

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



