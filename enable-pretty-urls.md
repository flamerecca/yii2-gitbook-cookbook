# 啟用 pretty URLs（Enable pretty URLs） {#enable-pretty-urls}

有時候我們會希望透過社群網站來分享網址。不過，舉例來說，網址可能會長得像`http://webproject.ru/index.php?r=site%2Fabout`。假設這個網址出現在臉書頁面，我們會想點擊嗎？多數使用者並不知道`index.php`和`%2F`是什麼，所以他們不大信任這類網址。這讓他們比較不願意點擊網址，網站就損失了流量。 

像是`http://webproject.ru/about`這樣的網址就比較好。每個使用者都可以理解，這網址會導向該網站的相關資料（_about page_）。

要達成這個目的，我們來啟用Yii專案的 pretty URLs 工具。

## Apache 伺服器設置 {#apache-web-server-configuration}

如果你正在使用 Apache 伺服器，你會需要多一個設定步驟。

在網頁根目錄或者設定檔資料夾的`.htaccess`檔案裡面，加入以下內容： 

```
RewriteEngine on
# If a directory or a file exists, use it directly
RewriteCond %{REQUEST_FILENAME} !-f
RewriteCond %{REQUEST_FILENAME} !-d
# Otherwise forward it to index.php
RewriteRule . index.php
```

## URL manager 設置 {#url-manager-configuration}

在 Yii config檔案裡面設置`urlManager`如下：

```php
'components' => [
    // ...
    'urlManager' => [
        'class' => 'yii\web\UrlManager',
        // Hide index.php
        'showScriptName' => false,
        // Use pretty URLs
        'enablePrettyUrl' => true,
        'rules' => [
        ],
    ],
    // ...
],
```

### 從網址中移除 site 參數 {#remove-site-parameter-from-url}

經過前面的步驟之後，你的網址會類似`http://webproject.ru/site/about`。這邊`site`參數對使用者沒有意義。所以我們可以透過`urlManager`規則把這個參數移除：

```php
    'rules' => [
        '<alias:\w+>' => 'site/<alias>',
    ],
```

之後，你的網址就會變成 `http://webproject.ru/about`。

