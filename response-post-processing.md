# 回應後期處理（Post-processing response）

有時候，我們需要對傳到瀏覽器的資料，進行後期處理（post-process）。比方說 Wordpress 引擎使用的標籤：

```
This is [username]. We have [visitor_count] visitors on website.
```

然後 username 和 visitor\_count 會自動換成對應的內容。

## 作法

Yii 在這方面的彈性很高，所以很容易達成：

```php
Yii::$app->getResponse()->on(Response::EVENT_AFTER_PREPARE, function($event) {
    /** @var User $user */
    $user = Yii::$app->getUser()->getIdentity();
    $replacements = [
        '[username]' => $user->username,
        '[visitor_count]' => 42,
    ];

    $event->sender->content = str_replace(array_keys($replacements), array_values($replacements), $event->sender->content);
});
```

上面的程式碼，我們使用了`Response::EVENT_AFTER_PREPARE`，讓該函式可以在傳至瀏覽器之前呼叫。在回傳事件裡面的`$event->sender`是我們包裝回應的物件，回應的內容則裝在`content`參數內。所以我們在`$event->sender->content`找尋並替換標籤。

