# 紀錄的問題和解法（Logging problems and solutions）

===============================

在Yii裡面，紀錄是有很大彈性的。基本的紀錄很簡單，但是有時需要花很多時間設定，來得到想要的紀錄方式。  
以下是一些需求的解法，希望讀者能找到適合自己需要的。

## 404 錯誤只寫檔紀錄，其他錯誤寄 email 提醒 {#write-404-to-file-and-send-the-rest-via-email}

遇到錯誤我們有時會希望有 email 提醒，但是 404 錯誤太常發生，跟其他錯誤不同，不值得寄 email 提醒，只值得紀錄進log檔裡面。  
我們來實做看看這樣的需求：

```php
'components' => [
    'log' => [
        'targets' => [
            'file' => [
                'class' => 'yii\log\FileTarget',
                'categories' => ['yii\web\HttpException:404'],
                'levels' => ['error', 'warning'],
                'logFile' => '@runtime/logs/404.log',
            ],
            'email' => [
                'class' => 'yii\log\EmailTarget',
                'except' => ['yii\web\HttpException:404'],
                'levels' => ['error', 'warning'],
                'message' => ['from' => 'robot@example.com', 'to' => 'admin@example.com'],
            ],
        ],
    ],
],
```

當遇到沒有處理的例外（exception ）狀況時，Yii 會進行紀錄，並顯示錯誤訊息或者客製化錯誤頁面給使用者。 這邊實際寫進紀錄的是例外訊息，而例外類別名稱則是當我們在設定錯誤訊息目標時，可以用來作為篩選訊息的分類。

404 錯誤可能是被`yii\web\NotFoundHttpException`觸發或者自己產生。兩種狀況下，處理該例外的類別都相同，並且繼承於 `\yii\web\HttpException`類別。另外 Yii 在這類錯誤紀錄的分類裡，使用`:`加上 HTTP 狀態碼做結尾，這其實在錯誤紀錄上是比較特殊的 。所以上面我們可以用這種方式，搭配使用`categories`來包括，使用`except`來排除  404 錯誤資訊，而不會影響其他的錯誤訊息。

## 即時紀錄 {#immediate-logging}

Yii 預設會等到程式運行結束，或者等紀錄程式累積到足夠的紀錄數量──預設是一千筆──才會把紀錄寫進檔案內。不過我們有可能希望能夠即時紀錄。比方說，當我們正在做重要的測試，而且是一邊做一邊監控 log 檔。這種狀況之下，我們要透過修改 config 檔來改變設定：

```php
'components' => [
    'log' => [
        'flushInterval' => 1, // <-- 改這裡
        'targets' => [
            'file' => [
                'class' => 'yii\log\FileTarget',
                'levels' => ['error', 'warning'],
                'exportInterval' => 1, // <-- 以及這裡
            ],
        ],
    ]
]
```

## 在不同檔案裡面紀錄不同狀況 {#write-different-logs-to-different-files}

程式有許許多多不同的功能，我們通常會希望藉由紀錄來監控這些功能的運作。可是，如果所有的東西都紀錄在同一個檔案裡面，這個檔案會變得太大、太難以維護了。比較好的作法是將不同的功能，紀錄在個別的紀錄檔內。

舉例來說，假設我們的網站有兩個功能：商品目錄以及購物籃。我們希望將紀錄分別寫進 catalog.log 和 basket.log。這時我們需要為紀錄資訊建立種類。將 config 檔修改如下，以連接不同種類的紀錄資訊與紀錄檔：

```php
'components' => [
    'log' => [
        'targets' => [
                [
                    'class' => 'yii\log\FileTarget',
                    'categories' => ['catalog'],
                    'logFile' => '@app/runtime/logs/catalog.log',
                ],
                [
                    'class' => 'yii\log\FileTarget',
                    'categories' => ['basket'],
                    'logFile' => '@app/runtime/logs/basket.log',
                ],
        ],
    ]
]
```

之後，藉由在紀錄函式的第二個參數裡加進種類名稱，我們就可以將錯誤寫進不同的紀錄檔。  
範例：

```php
\Yii::info('catalog info', 'catalog');
\Yii::error('basket error', 'basket');
\Yii::beginProfile('add to basket', 'basket');
\Yii::endProfile('add to basket', 'basket');
```

## 在紀錄中忽略自己的操作 {#disable-logging-of-youself-actions}

### 問題 {#problem}

當有使用者註冊時，我們希望可以收到 email。但是不要在自己測試註冊的時候，還寄信提醒。

### 解法 {#solution}

At首先，在`config.php`設定紀錄目標：

```php
'log' => [
    // ...
    'targets' => [
            // 「email」是我們紀錄目標對應的關鍵字
            'email' => [  
                'class' => 'yii\log\EmailTarget',
                'levels' => ['info'],
                'message' => ['from' => 'robot@example.com', 'to' => 'admin@example.com'],
            ],
    // ...
```

然後，像是在`ControllerbeforeAction`，讀者可以建立一個狀況：

```php
    public function beforeAction($action)
    {
        // '127.0.0.1' - 或換成讀者自己的 IP
        if (in_array(@$_SERVER['REMOTE_ADDR'], ['127.0.0.1'])) {
            Yii::$app->log->targets['email']->enabled = false; // 這裡取消我們的target
        }
        return parent::beforeAction($action);
    }
```

## 留下完整紀錄但是顯示不同錯誤訊息 {#log-everything-but-display-different-error-messages}

### 問題 {#problem_1}

我們想紀錄具體的錯誤，但是只對使用者顯示粗略的解釋。

### 解法 {#solution_1}

如果你將例外成功的 catch ，那麼 log target 就不會生效。假設我們原本設定的紀錄目標如下：

```php
'components' => [
    'log' => [
        'targets' => [
            'file' => [
                'class' => 'yii\log\FileTarget',
                'levels' => ['error', 'warning'],
                'logFile' => '@runtime/logs/error.log',
            ],
        ],
    ],
],
```

作為示範，我們修改`actionIndex`如下：

```php
    public function actionIndex()
    {
        throw new ServerErrorHttpException('程式有問題喔！');
        // ...
```

這個例外沒有被 catch，進入`index`頁面，你會看到錯誤信息，並且`error.log`檔也會有一樣的紀錄。

現在我們修改`actionIndex()`：

```php
    public function actionIndex()
    {
        try {
            throw new ServerErrorHttpException('程式有問題喔！'); // 這邊是原本的程式
        }
        catch(ServerErrorHttpException $ex) {
            Yii::error($ex->getMessage()); // 給我們自己看的具體訊息
            throw new ServerErrorHttpException('網頁問題，抱歉'); // 給使用者看的粗略訊息
        }
    // ..
```

這樣的話，我們在瀏覽器上會看到`網頁問題，抱歉`，但是在`error.log`裡面，我們會同時看到**兩個**錯誤訊息。這邊的第二個錯誤訊息只是給使用者看，照理是不需要被紀錄的。

我們加上`category`來排除不必要的紀錄。

對`config`：

```php
'file' => [
    'class' => 'yii\log\FileTarget',
    'levels' => ['error', 'warning'],
    'categories' => ['serverError'], // 加入種類
    'logFile' => '@runtime/logs/error.log',
],
```

對`actionIndex()`：

```php
catch(ServerErrorHttpException $ex) {
    Yii::error($ex->getMessage(), 'serverError'); // 加入種類
    throw new ServerErrorHttpException('網頁問題，抱歉');
}
```

這樣`error.log`裡面，我們只會看到`程式有問題喔！`的錯誤紀錄。

### 這樣做的好處 {#even-more}

如果今天是遇到 bad request 例外，我們可能希望能夠照 Yii 預設的，紀錄並顯示相同的錯誤訊息給使用者。  
用上面的方法很容易達成這個目的，因為我們在 catch 這一段，只有處理到`ServerErrorHttpException`這一類例外 。

所以，如果我們丟出的例外是：

```php
throw new BadRequestHttpException('提供的信箱不可用');
```

那麼使用者就能如我們希望的看到該訊息。

## 其他資料 {#see-also}

* [Yii2 教學 - 處理錯誤](http://www.yiiframework.com/doc-2.0/guide-runtime-handling-errors.html)
* [Yii2 教學 - logging](http://www.yiiframework.com/doc-2.0/guide-runtime-logging.html)



