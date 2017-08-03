# IDE autocompletion for custom components {#ide-autocompletion-for-custom-components}

因為其提供的各種好處，在現在的程式開發內，使用 IDE 已經是相當基本的事情。IDE 可以幫忙檢查打字錯誤、偵測程式問題、建議程式碼可以改進的地方，以及程式碼自動完成。

對 Yii 2.0 來說，用 IDE 來開發已經相當合適，除了針對  not in case of custom application components，像是`Yii::$app->mycomponent->something`。

## 使用自定義 Yii 類別 {#using-custom-yii-class}

The best way to give IDE some hints，是建立一個實際上沒有運作的`Yii`檔。This file could be named`Yii.php`and the content could be the following:

```php
<?php
/**
 * Yii bootstrap file.
 * Used for enhanced IDE code autocompletion.
 */
class Yii extends \yii\BaseYii
{
    /**
     * @var BaseApplication|WebApplication|ConsoleApplication the application instance
     */
    public static $app;
}

/**
 * Class BaseApplication
 * Used for properties that are identical for both WebApplication and ConsoleApplication
 *
 * @property \app\components\RbacManager $authManager The auth manager for this application. Null is returned if auth manager is not configured. This property is read-only. Extended component.
 * @property \app\components\Mailer $mailer The mailer component. This property is read-only. Extended component.
 */
abstract class BaseApplication extends yii\base\Application
{
}

/**
 * Class WebApplication
 * Include only Web application related components here
 *
 * @property \app\components\User $user The user component. This property is read-only. Extended component.
 * @property \app\components\MyResponse $response The response component. This property is read-only. Extended component.
 * @property \app\components\ErrorHandler $errorHandler The error handler application component. This property is read-only. Extended component.
 */
class WebApplication extends yii\web\Application
{
}

/**
 * Class ConsoleApplication
 * Include only Console application related components here
 *
 * @property \app\components\ConsoleUser $user The user component. This property is read-only. Extended component.
 */
class ConsoleApplication extends yii\console\Application
{
}
```

In the above PHPDoc of `BaseApplication`、`WebApplication`、`ConsoleApplication`will be used by IDE to autocomplete your custom components described via`@property`.

> **備註：** To avoid "Multiple Implementations" PHPStorm warning and make autocomplete faster ， exclude or "Mark as Plain Text"`vendor/yiisoft/yii2/Yii.php`file.

好了，現在`Yii::$app->user`不再是預設的user，而是註解內的`\app\components\User`元件。 The same applies for all other`@property`-declared components.

## 自動產生 Custom Yii 類別 {#custom-yii-class-autogeneration}

You can generate custom`Yii`class automatically, using the components definitions from the application config. Check out [bazilio91/yii2-stubs-generator](https://github.com/bazilio91/yii2-stubs-generator) 套件。

### Customizing User component {#customizing-user-component}

In order to get autocompletion for User's Identity i.e.`Yii::$app->user->identity`,`app\components\User`class should look like the following:

```php
<?php

namespace app\components;

use Yii;

/**
 * @inheritdoc
 *
 * @property \app\models\User|\yii\web\IdentityInterface|null $identity The identity object associated with the currently logged-in user. null is returned if the user is not logged in (not authenticated).
 */
class User extends \yii\web\User
{
}
```

最後， Yii config 可能會看起來像是這樣：

```php
return [
    ...
    'components' => [
        /**
         * User
         */
        'user' => [
            'class' => 'app\components\User',
            'identityClass' => 'app\models\User',
        ],
        /**
         * Custom Component
         */
        'response' => [
            'class' => 'app\components\MyResponse',
        ],
    ],
];
```



