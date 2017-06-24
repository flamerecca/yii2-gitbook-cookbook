# Handling incoming third party POST requests {#handling-incoming-third-party-post-requests}

Yii 預設啟用 CSRF protection that verifies that POST requests could be made only by the same application. It enhances overall security significantly but there are cases when CSRF should be disabled i.e. when you expect incoming POST requests from a third party service.

Additionally, if third party is posting via XMLHttpRequest \(browser AJAX\), we need to send additional headers to allow CORS \([cross-origin resource sharing](https://en.wikipedia.org/wiki/Cross-origin_resource_sharing)\).

## 怎麼作到 {#how-to-do-it}

首先，永遠不要把 CSRF 保護整個取消。如果CSRF會影響POST傳輸，僅僅針對單一控制器，甚至控制器的單一 action 取消就好。

### 為特定控制器取消 CSRF {#disabling-csrf-for-a-specific-controller}

為特定控制器取消 CSRF 保護很簡單：

```php
class MyController extends Controller
{
    public $enableCsrfValidation = false;
```

就這樣，我們加上一個公開參數 `enableCsrfValidation`，並將其設為 `false`即可。

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

這邊我們實做`beforeAction()`函式。 It is invoked right before an action is executed so we're checking if executed action id matches id of the action we want to disable CSRF protection for and, if it's true, disabling it. 注意在`beforeAction`的最後一定要`return parent::beforeAction($action);`

### 送出 CORS 標頭 {#sending-cors-headers}

Yii has a [special Cors filter](http://www.yiiframework.com/doc-2.0/yii-filters-cors.html) that allows you sending headers required to allow CORS.

To allow AJAX requests to the whole controller you can use it like that:

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

In order to do it for a specific action, use the following:

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



