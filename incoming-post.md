# 處理第三方來的POST請求（Handling incoming third party POST requests） {#handling-incoming-third-party-post-requests}

Yii 預設會啟用 [CSRF](/csrf.md) 保護，以驗證 POST請求是來自己方的程式。這是提昇安全性非常重要的環節。

不過，有時候我們也可能會暫時關閉這個保護，比方說需要允許其他程式透過 POST請求的時候。

另外，如果第三方透過 XMLHttpRequest （瀏覽器的 AJAX）存取資料，我們可能需要傳送符合[跨來源資源共享](https://zh.wikipedia.org/wiki/跨來源資源共享)（Cross-origin resource sharing，CORS）的標頭。

## 怎麼作到 {#how-to-do-it}

首先，永遠不要把 CSRF 保護整個取消。

如果CSRF會影響POST傳輸，僅僅針對單一控制器，甚至控制器的單一 action 取消就好。

### 為特定控制器取消 CSRF {#disabling-csrf-for-a-specific-controller}

為特定控制器取消 CSRF 保護很簡單：

```php
class MyController extends Controller
{
    public $enableCsrfValidation = false;
```

就這樣，我們加上一個公開參數 `enableCsrfValidation`，並將其設為`false`即可。

### 為特定控制器 action 取消 CSRF {#disabling-csrf-for-specific-controller-action}

如果我們只想要對特定 action 取消，則稍微麻煩一點：

```php
class MyController extends Controller
{
    public function beforeAction($action)
    {
        if (in_array($action->id, ['incoming'])) {
            $this->enableCsrfValidation = false;
        }
        return parent::beforeAction($action);
    }
```

這邊我們實做`beforeAction()`函式。 該函式會在整個 action 執行之前被呼叫。根據我們補充的程式，如果該函式的名字在我們的陣列裡面，則將 CSRF 保護關掉。注意在`beforeAction()`的最後，一定要`return parent::beforeAction($action);`

### 送出 CORS 標頭 {#sending-cors-headers}

Yii 有一個自己的 [Cors filter](http://www.yiiframework.com/doc-2.0/yii-filters-cors.html)，幫助我們要允許 CORS 的時候，傳輸對應的標頭。

在整個控制器上面允許 AJAX requests，你需要：

```php
class MyController extends Controller
{
    public function behaviors()
    {
        return [
            'corsFilter' => [
                'class' => \yii\filters\Cors::className(),
            ],
        ];
    }
```

如果只允許某個 action 的 AJAX requests，你需要：

```php
class MyController extends Controller
{
    public function behaviors()
    {
        return [
            'corsFilter' => [
                'class' => \yii\filters\Cors::className(),
                'cors' => [],
                'actions' => [
                    'incoming' => [
                        'Origin' => ['*'],
                        'Access-Control-Request-Method' => ['GET', 'POST', 'PUT', 'PATCH', 'DELETE', 'HEAD', 'OPTIONS'],
                        'Access-Control-Request-Headers' => ['*'],
                        'Access-Control-Allow-Credentials' => null,
                        'Access-Control-Max-Age' => 86400,
                        'Access-Control-Expose-Headers' => [],
                    ],
                ],
            ],
        ];
    }
```



