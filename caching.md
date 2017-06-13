# 快取（Caching） {#caching}

Yii 支援不同的快取機制。下面我們說明如何處理常見的快取問題。

## 使用APC套件實做 PHP 變數快取 {#php-variables-caching-using-apc-extension}

在 Yii config裡面設置快取套件，這邊我們使用 ApcCache：

```php
'cache' => [
  'class' => 'yii\caching\ApcCache',
],
```

之後，我們就可以使用快取了，方法如下：

```php
$key = 'cacheKey'
//取得快取資料
$data = Yii::$app->cache->get($key);

if ($data === false) {
    // $data不在快取裡面，重新計算資料

    // 儲存$data，這樣下次就可以從快取裡面取得了
    $cache->set($key, $data);
}
```

如果收到`Call to undefined function yii\caching\apc_fetch()`錯誤，代表 APC 套件安裝有問題。請參考 [PHP APC manual](http://php.net/manual/en/book.apc.php) 找出問題點。

如果快取沒有生效（`$data 總是 false` ），可以檢查`php.ini` 的 `apc.shm_size`參數。有可能是資料的大小超過該參數允許的大小。

## 用於靜態資源的 HTTP 快取 {#http-caching-for-assets-and-other-static-resources}

如果沒有指定靜態資源的過期時間 （像是`.js`、`.css`……等等），網頁存取的速度可能會變得非常慢。`Chrome的 PageSpeed Insights`指定`過期時間未指定`（expiration not specified）是Yii網頁效率的**重要問題。**PageSpeed Insights 會建議我們`使用瀏覽器快取功能`（Leverage browser caching）。我們可以在 asset manager 加上這段程式碼以改善這個問題：

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



