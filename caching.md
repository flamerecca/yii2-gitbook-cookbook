# 快取（Caching） {#caching}

Yii 支援不同的快取機制。 In this recipe we'll describe how to solve typical caching tasks.

## 使用APC套件實做 PHP 變數快取 {#php-variables-caching-using-apc-extension}

Configure cache component in Yii application config file:

```php
'cache' => [
  'class' => 'yii\caching\ApcCache',
],
```

之後，我們就可以使用快取了，方法如下：

```php
$key = 'cacheKey'
$data = Yii::$app->cache->get($key);

if ($data === false) {
    // $data不在快取裡面，重新計算資料

    // 儲存$data，這樣下次就可以從快取裡面取得了
    $cache->set($key, $data);
}
```

Error`Call to undefined function yii\caching\apc_fetch()`means that you have problems with APC extension. Refer[PHP APC manual](http://php.net/manual/en/book.apc.php) for the details.

If cache doesn't work \(`$data is always false`\) check`apc.shm_size`property in`php.ini`. Probably your data size is more than allowed by this parameter.

## 用於靜態資源的 HTTP 快取 {#http-caching-for-assets-and-other-static-resources}

If expiration not specified for cacheable resources （`.js`、`.css`……等等）a speed of page loading process may be very slow. Such tool as`PageSpeed Insights for Chrome`determines`expiration not specified`problem as **crucial **for yii web page performance. It advices you to`Leverage browser caching`. You can do it by adding only one row to your application asset manager component:

```php
return [
    // ...
    'components' => [
        'assetManager' => [
            'appendTimestamp' => true,
        ],
    ],
];
```



