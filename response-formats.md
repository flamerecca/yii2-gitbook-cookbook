# 處理不同回應格式（Working with different response types） {#working-with-different-response-types}

現在的網頁與手機應用已經不只是產生 HTML 格式了。新的架構流行讓伺服器 API 傳輸資料給前端，由前端處理UI的部份。而不是由伺服器端的程式處理外觀的部份。

傳輸資料時，通常會選用 JSON 與 XML 格式，所以對任何現代的框架，透過這兩種格式資料是必須的功能。

## 回應格式 {#response-formats}

如我們所知道，Yii2 我們不是用 echo，而是用`return`來回傳我們 action 內處理好的結果：

```php
// 回傳 HTML 結果
return $this->render('index', [
    'items' => $items,
]);
```

這樣的好處是，你可以傳其他型態的資料，像是：

* 陣列
* 任何實做`Arrayable`介面的物件
* 字串
* 任何實做`__toString()`函式的物件

不過我們還需要告訴 Yii 我們要回傳的資料型態。所以要在`return`之前設定`\Yii::$app->response->format`，像是：

```php
\Yii::$app->response->format = \yii\web\Response::FORMAT_JSON;
```

可以用的型態有：

* FORMAT\_RAW
* FORMAT\_HTML
* FORMAT\_JSON
* FORMAT\_JSONP
* FORMAT\_XML

預設是`FORMAT_HTML`。

## JSON 格式 {#json-response}

### 回傳陣列

透過 JSON 格式回傳陣列：

```php
public function actionIndex()
{
    \Yii::$app->response->format = \yii\web\Response::FORMAT_JSON;
    $items = ['some', 'array', 'of', 'data' => ['associative', 'array']];
    return $items;
}
```

這樣就好了！我們成功的傳出了 JSON 格式的陣列

#### 結果

```js
{
    "0": "some",
    "1": "array",
    "2": "of",
    "data": ["associative", "array"]
}
```

> 備註：如果回應格式沒有設定，回傳陣列會產生例外（exception）。

### 回傳物件

如上面的作法，我們也可以回傳物件：

```php
public function actionView($id)
{
    \Yii::$app->response->format = \yii\web\Response::FORMAT_JSON;
    $user = \app\models\User::find($id);
    return $user;
}
```

因為 Yii 的`ActiveRecord`有實做`Arrayable`，所以我們可以將其轉換成JSON：

#### **結果**

```js
{
    "id": 1,
    "name": "John Doe",
    "email": "john@example.com"
}
```

### 回傳物件陣列

如果該類別有實做`Arrayable`，我們甚至可以直接回傳該物件的陣列：

```php
public function actionIndex()
{
    \Yii::$app->response->format = \yii\web\Response::FORMAT_JSON;
    $users = \app\models\User::find()->all();
    return $users;
}
```

Now`$users`is an array of ActiveRecord objects, but under the hood Yii uses`\yii\helpers\Json::encode()`that traverses and converts the passed data, taking care of types by itself:

#### 結果

```js
[
    {
        "id": 1,
        "name": "John Doe",
        "email": "john@example.com"
    },
    {
        "id": 2,
        "name": "Jane Foo",
        "email": "jane@example.com"
    },
    ...
]
```

## XML 回應 {#xml-response}

要將輸出的資料改成XML，只需要將`\Yii::$app->response->format`設置成`FORMAT_XML`，就可以了：

```php
public function actionIndex()
{
    \Yii::$app->response->format = \yii\web\Response::FORMAT_XML;
    $items = ['some', 'array', 'of', 'data' => ['associative', 'array']];
    return $items;
}
```

#### 結果

```php
<response>
    <item>some</item>
    <item>array</item>
    <item>of</item>
    <data>
        <item>associative</item>
        <item>array</item>
    </data>
</response>
```

跟JSON的時候一樣，我們也可以回傳物件或者物件的陣列：

```
public function actionIndex()
{
    \Yii::$app->response->format = \yii\web\Response::FORMAT_XML;
    $users = \app\models\User::find()->all();
    return $users;
}
```

#### 結果

```
<response>
    <User>
        <id>1</id>
        <name>John Doe</name>
        <email>john@example.com</email>
    </User>
    <User>
        <id>2</id>
        <name>Jane Foo</name>
        <email>jane@example.com</email>
    </User>
</response>
```

## 自定義回應格式 {#custom-response-format}

如果 JSON 與 XML 格式都不能滿足我們的需求，我們得自己定義回傳的格式。這邊的例子為了好玩一點，我們假設得回傳 PHP 程式碼作為回傳格式。

首先，我們要自製一個格式產生器（formatter），建立`components/PhpArrayFormatter.php`：

```php
<?php
namespace app\components;

use yii\helpers\VarDumper;
use yii\web\ResponseFormatterInterface;

class PhpArrayFormatter implements ResponseFormatterInterface
{
    public function format($response)
    {
        $response->getHeaders()->set('Content-Type', 'text/php; charset=UTF-8');
        if ($response->data !== null) {
            $response->content = "<?php\nreturn " . VarDumper::export($response->data) . ";\n";
        }
    }
}
```

現在，我們需要將這個程式紀錄進Yii 的 config 內（一般會是`config/web.php`）：

```
return [
    // ...
    'components' => [
        // ...
        'response' => [
            'formatters' => [
                'php' => 'app\components\PhpArrayFormatter',
            ],
        ],
    ],
];
```

現在這個格式產生器已經準備好了，如果我們在`controllers/SiteController`建立`actionTest()`：

```php
public function actionTest()
{
    Yii::$app->response->format = 'php';
    return [
        'hello' => 'world!',
    ];
}
```

好了，如果執行的話，Yii 的回應會變成：

```
<?php
return [
    'hello' => 'world!',
];
```

## 讓使用者選擇回傳格式 {#choosing-format-based-on-content-type-requested}

我們可以使用`ContentNegotiator`這個控制器過濾（controller filter）來選擇對應的格式。要這樣做，我們得先在控制器裡面實做`behaviors()`函式：

```php
public function behaviors()
{
    return [
        // ...
        'contentNegotiator' => [
            'class' => \yii\filters\ContentNegotiator::className(),
            'only' => ['index', 'view'],
            'formatParam' => '_format',
            'formats' => [
                'application/json' => \yii\web\Response::FORMAT_JSON,
                'application/xml' => \yii\web\Response::FORMAT_XML,
            ],
        ],
    ];
}

public function actionIndex()
{
    $users = \app\models\User::find()->all();
    return $users;
}

public function actionView($id)
{
    $user = \app\models\User::findOne($id);
    return $user;
}
```

好了，現在我們可以測試看看下面的網址：

* `/index.php?r=user/index&_format=xml`
* `/index.php?r=user/index&_format=json`



