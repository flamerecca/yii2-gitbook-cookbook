===============================

在Yii裡面，紀錄是有很大彈性的。基本的紀錄很簡單，但是有時需要花很多時間設定，來得到想要的紀錄方式。

以下是一些需求的解法，希望讀者能找到適合自己需要的。

## 404 錯誤只寫檔紀錄，其他錯誤寄 email 提醒 {#write-404-to-file-and-send-the-rest-via-email}

404 錯誤太常發生，跟其他錯誤不同，不值得寄 email 提醒。不過還是值得紀錄進log檔裡面。

我們來實做看看：

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

When there's unhandled exception in the application Yii logs it additionally to displaying it to end user or showing customized error page. Exception message is what actually to be written and the fully qualified exception class name is the category we can use to filter messages when configuring targets. 404 can be triggered by throwing`yii\web\NotFoundHttpException`or automatically. In both cases exception class is the same and is inherited from`\yii\web\HttpException`which is a bit special in regards to logging. The speciality is the fact that HTTP status code prepended by`:`is appended to the end of the log message category. In the above we're using`categories`to include and`except`to exclude 404 log messages.

## 即時紀錄 {#immediate-logging}

By default Yii accumulates logs till the script is finished or till the number of logs accumulated is enough which is 1000 messages by default for both logger itself and log target. It could be that you want to log messages immediately. For example, when running an import job and checking logs to see the progress. In this case you need to change settings via application config file:

```php
'components' => [
    'log' => [
        'flushInterval' => 1, // <-- 這裡
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

Usually a program has a lot of functions. Sometimes it is necessary to control these functions by logging. If everything is logged in one file this file becomes too big and too difficult to maintain. Good solution is to write different functions logs to different files.

For example you have two functions: catalog and basket. Let's write logs to catalog.log and basket.log respectively. In this case you need to establish categories for your log messages. Make a connection between them and log targets by changing application config file:

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

After this you are able to write logs to separate files adding category name to log function as second parameter. Examples:

```php
\Yii::info('catalog info', 'catalog');
\Yii::error('basket error', 'basket');
\Yii::beginProfile('add to basket', 'basket');
\Yii::endProfile('add to basket', 'basket');
```

## 在紀錄中忽略自己的操作 {#disable-logging-of-youself-actions}

### 問題 {#problem}

當有使用者註冊時，希望可以收到 email。但是不要再自己測試註冊的時候，還寄信提醒。

### 解法 {#solution}

At first mark logging target inside`config.php`by key:

```php
'log' => [
    // ...
    'targets' => [
            // email is a key for our target
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
            Yii::$app->log->targets['email']->enabled = false; // 這裡取消我們的targets
        }
        return parent::beforeAction($action);
    }
```

## 留下完整紀錄但是顯示不同錯誤訊息 {#log-everything-but-display-different-error-messages}

### 問題 {#problem_1}

我們想紀錄具體的錯誤，但是只對使用者顯示粗略的解釋。

### 解法 {#solution_1}

If you catch an error appropriate log target doesn't work. Let's say you have such log target configuration:

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

As an example let's add such code line inside`actionIndex`：

```php
    public function actionIndex()
    {
        throw new ServerErrorHttpException('Hey! Coding problems!');
        // ...
```

Go to`index`page and you will see this message in the browser and in the`error.log`file.

Let's modify our`actionIndex`:

```php
    public function actionIndex()
    {
        try {
            throw new ServerErrorHttpException('Hey! Coding problems!'); // here is our code line now
        }
        catch(ServerErrorHttpException $ex) {
            Yii::error($ex->getMessage()); // concrete message for us
            throw new ServerErrorHttpException('Server problem, sorry.'); // broad message for the end user
        }
    // ..
```

As the result in the browser you will see`Server problem, sorry.`. But in the`error.log`you will see **both **error messages. In our case second message is not necessary to log.

Let's add`category`for our log target and for logging command.

For`config`:

```php
'file' => [
    'class' => 'yii\log\FileTarget',
    'levels' => ['error', 'warning'],
    'categories' => ['serverError'], // category is added
    'logFile' => '@runtime/logs/error.log',
],
```

For`actionIndex`:

```php
catch(ServerErrorHttpException $ex) {
    Yii::error($ex->getMessage(), 'serverError'); // category is added
    throw new ServerErrorHttpException('Server problem, sorry.');
}
```

As the result in the`error.log`you will see only the error related to`Hey! Coding problems!`.

### 更多 {#even-more}

If there is an bad request \(user side\) error you may want to display error message 'as is'. You can easily do it because our catch block works only for`ServerErrorHttpException`error types. So you are able to throw something like this:

```php
throw new BadRequestHttpException('Email address you provide is invalid');
```

As the result end user will see the message 'as is' in his browser.

## 其他資料 {#see-also}

* [Yii2 教學 - 處理錯誤](http://www.yiiframework.com/doc-2.0/guide-runtime-handling-errors.html)
* [Yii2 教學 - logging](http://www.yiiframework.com/doc-2.0/guide-runtime-logging.html)



