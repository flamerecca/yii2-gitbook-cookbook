# 在第三方程式中使用Yii（Using Yii in third party apps） {#using-yii-in-third-party-apps}

之前的遺留程式碼（legacy code）。這問題我們都會遇到， 雖然很痛苦，但是還是得處理。

如果能從之前留下來的網頁程式，逐漸的轉換到Yii，會不會很酷呢？當然。

這邊，我們教你如何一邊用之前的PHP程式，一邊使用Yii 的功能

## 如何達成 {#how-to-do-it}

首先，我們需要安裝Yii。因為原本的網頁程式就有自己的routing，所以我們不需要應用程式的樣板。

我們用composer進行安裝，從`composer.json`開始。可以使用原本的檔案或者直接建立：

```js
{
    "name": "mysoft/mylegacyapp",
    "description": "Legacy app",
    "keywords": ["yii2", "legacy"],
    "homepage": "http://www.example.com/",
    "type": "project",
    "license": "Copyrighted",
    "minimum-stability": "dev",
    "require": {
        "php": ">=5.4.0",
        "yiisoft/yii2": "*"
    },
    "config": {
        "process-timeout": 1800
    },
    "extra": {
        "asset-installer-paths": {
            "npm-asset-library": "vendor/npm",
            "bower-asset-library": "vendor/bower"
        }
    }
}
```

執行`composer install`，然後Yii 就會安裝在`vendor`資料夾內

> 備註：Yii 程式所在的資料夾，不應該可以從網頁存取。可以放在webroot之外，或者設定存取權限拒絕該資料夾的存取

再來，我們建立一個初始化Yii 所用的程式碼。這邊我們命名為`yii_init.php`：

```php
<?php
// 以下這段在正式網頁應該，應該設置為false
defined('YII_DEBUG') or define('YII_DEBUG', true);

require(__DIR__ . '/vendor/autoload.php');
require(__DIR__ . '/vendor/yiisoft/yii2/Yii.php');

$config = require(__DIR__ . '/config/yii.php');

new yii\web\Application($config);
```

然後，建立 config檔`config/yii.php`:

```php
<?php

return [
    'id' => 'myapp',
    'basePath' => dirname(__DIR__),
    'bootstrap' => ['log'],
    'components' => [
        'request' => [
            // !!! insert a secret key in the following (if it is empty) - this is required by cookie validation
            'cookieValidationKey' => '',
        ],
        'cache' => [
            'class' => 'yii\caching\FileCache',
        ],
        'user' => [
            'identityClass' => 'app\models\User',
            'enableAutoLogin' => true,
        ],
        'log' => [
            'traceLevel' => YII_DEBUG ? 3 : 0,
            'targets' => [
                [
                    'class' => 'yii\log\FileTarget',
                    'levels' => ['error', 'warning'],
                ],
            ],
        ],
    ],
    'params' => [],
];
```

就這樣，現在我們可以使用原來的程式，並一併使用Yii的功能：

```php
// 遺留程式碼

$id = (int)$_GET['id'];


// 新程式碼
require 'path/to/yii_init.php';

$post = \app\models\Post::find()->where['id' => $id];

echo Html::encode($post->title);
```

## 運作原理 {#how-it-works}

在`yii_init.php`裡面， 我們包含了Yii框架的類別，以及composer的autoloading。一個重點是，因為要跳過Yii自己的routing跟controller，這邊不像傳統的Yii程式這樣呼叫`->run()`。

