# Working with different response types {#working-with-different-response-types}

現在的網頁與手機應用已經不只是產生 HTML 格式了。Modern architecture moves the UI to the client, where all user interactions are handled by the client-side, utilizing server APIs to drive the frontend. 

JSON 與 XML 格式 are often used for serializing and transmitting structured data over a network, so the ability to create such responses is a must for any modern server framework.

## 回應格式 {#response-formats}

As you probably know, in Yii2 you need to`return`the result from your action, instead of echoing it directly:

```php
// returning HTML result
return $this->render('index', [
    'items' => $items,
]);
```

Good thing about it is now you can return different types of data from your action, namely:

* an array
* an object implementing
  `Arrayable`
  interface
* a string
* an object implementing
  `__toString()`
  method.

Just don't forget to tell Yii what format do you want as result, by setting`\Yii::$app->response->format`before`return`. For example:

```php
\Yii::$app->response->format = \yii\web\Response::FORMAT_JSON;
```

Valid formats are:

* FORMAT\_RAW
* FORMAT\_HTML
* FORMAT\_JSON
* FORMAT\_JSONP
* FORMAT\_XML

Default is`FORMAT_HTML`.

## JSON 格式 {#json-response}

### 回傳陣列

Let's return an array:

```php
public function actionIndex()
{
    \Yii::$app->response->format = \yii\web\Response::FORMAT_JSON;
    $items = ['some', 'array', 'of', 'data' => ['associative', 'array']];
    return $items;
}
```

And - voila! - we have JSON response right out of the box:

#### 結果

```js
{
    "0": "some",
    "1": "array",
    "2": "of",
    "data": ["associative", "array"]
}
```

**Note**: you'll get an exception if response format is not set.

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

Now $user is an instance of`ActiveRecord`class that implements`Arrayable`interface, so it can be easily converted to JSON:

#### **結果**

```js
{
    "id": 1,
    "name": "John Doe",
    "email": "john@example.com"
}
```

### 回傳物件陣列

We can even return an array of objects:

```
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

Just change response format to`FORMAT_XML`and that't it. Now you have XML:

```
public function actionIndex()
{
    \Yii::$app->response->format = \yii\web\Response::FORMAT_XML;
    $items = ['some', 'array', 'of', 'data' => ['associative', 'array']];
    return $items;
}
```

#### 結果

```
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

And yes, we can convert objects and array of objects the same way as we did before.

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

## Custom response format {#custom-response-format}

Let's create a custom response format. To make example a bit fun and crazy we'll respond with PHP arrays.

First of all, we need formatter itself. Create`components/PhpArrayFormatter.php`:

```
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

Now we need to registed it in application config \(usually it's`config/web.php`\):

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

Now it's ready to be used. In`controllers/SiteController`create a new method`actionTest`:

```
public function actionTest()
{
    Yii::$app->response->format = 'php';
    return [
        'hello' => 'world!',
    ];
}
```

That's it. After executing it, Yii will respond with the following:

```
<?php
return [
    'hello' => 'world!',
];
```

## 讓使用者選擇回傳格式 {#choosing-format-based-on-content-type-requested}

You can use the`ContentNegotiator`controller filter in order to choose format based on what is requested. In order to do so you need to implement`behaviors`method in controller:

```
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

That's it. Now you can test it via the following URLs:

* `/index.php?r=user/index&_format=xml`
* `/index.php?r=user/index&_format=json`



