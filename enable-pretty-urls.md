# 啟用 pretty URLs（Enable pretty URLs） {#enable-pretty-urls}

Sometimes users want to share your site URLs via social networks. For example, by default your\_about page\_URL looks like`http://webproject.ru/index.php?r=site%2Fabout`. Let's imagine this link on Facebook page. Do you want to click on it? Most of users have no idea what is`index.php`and what is`%2`. They trust such link less, so will click less on it. Thus web site owner would lose traffic.

URLs such as the following is better:`http://webproject.ru/about`. Every user can understand that it is a clear way to get to_about page_.

Let's enable pretty URLs for our Yii project.

## Apache 伺服器設置 {#apache-web-server-configuration}

If you're using Apache you need an extra step. Inside your`.htaccess`file in your webroot directory or inside location section of your main Apache config add the following lines:

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

