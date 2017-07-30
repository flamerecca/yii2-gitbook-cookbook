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

上面的程式碼，我們使用了`Response::EVENT_AFTER_PREPARE`，讓函式可以hich is triggered right before sending content to a browser. In the callback`$event->sender`is our response object which keeps data to be sent in`content`參數。所以我們在這裡找尋並替換標籤。

