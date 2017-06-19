# 使用全域函式（Using global functions） {#using-global-functions}

雖然乍看之下不是個好主意，但是在 PHP 宣告一些全域函式其實是好事。程式碼會變短，而且如果函式命名的好，會更簡潔易懂。

## 作法 {#how-to-do-it}

首先，建立儲存函式的檔案。假設是在根資料夾的`functions.php`。我們必須要將這個檔案被`require`。 最適合的地方是在`index.php`：

```php
// ...
require(__DIR__ . '/../vendor/autoload.php');
require(__DIR__ . '/../vendor/yiisoft/yii2/Yii.php');

$config = require(__DIR__ . '/../config/web.php');

$app = new yii\web\Application($config);
require(__DIR__ . '/../functions.php');
$app->run();
```

注意我們這裡是在 config 檔以及應用建立之後才宣告`require` 這允許我們在函式裡面使用config以及Yii裡面的函式。

另外，你也可以加在`composer.json`：

```php
"autoload": {
    "files": [
        "functions.php"
    ]
},
```

增加這段之後，記得要執行`composer update`，程式才會生效。

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



