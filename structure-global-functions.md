# 使用全域函式（Using global functions） {#using-global-functions}

Although it looks like a weird idea at the first glance, using just functions in PHP is actually nice. Code looks much shorter and, with a good naming choices, much simpler.

## 作法 {#how-to-do-it}

首先，建立儲存函式的檔案。假設是 create a file that will contain functions. Let it be`functions.php`right in the root of the application. In order to be used it should in`require`d. The best place to do it is`index.php`:

```php
// ...
require(__DIR__ . '/../vendor/autoload.php');
require(__DIR__ . '/../vendor/yiisoft/yii2/Yii.php');

$config = require(__DIR__ . '/../config/web.php');

$app = new yii\web\Application($config);
require(__DIR__ . '/../functions.php');
$app->run();
```

Note that we're requiring it after including config and creating application instance. That allows you to use config and application which is needed for many functions.

Also you can do it in`composer.json`:

```php
"autoload": {
    "files": [
        "functions.php"
    ]
},
```

Note that after add this section you need run`composer update`.

## 函式的點子 {#function-ideas}

以下是一些函式的點子，如果有需要可以自己增加：

```php
use yii\helpers\Url;
use yii\helpers\Html;
use yii\helpers\HtmlPurifier;
use yii\helpers\ArrayHelper;

function url($url = '', $scheme = false)
{
    return Url::to($url, $scheme);
}

function h($text)
{
    return Html::encode($text);
}

function ph($text)
{
    return HtmlPurifier::process($text);
}

function t($message, $params = [], $category = 'app', $language = null)
{
    return Yii::t($category, $message, $params, $language);
}

function param($name, $default = null)
{
    return ArrayHelper::getValue(Yii::$app->params, $name, $default);
}
```



