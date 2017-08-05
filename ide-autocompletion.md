# 自定義元件的 IDE 自動產生（IDE autocompletion for custom components） {#ide-autocompletion-for-custom-components}

因為其提供的各種好處，在現在的程式開發內，使用 IDE 已經是相當基本的事情。IDE 可以幫忙檢查打字錯誤、偵測程式問題、建議程式碼可以改進的地方，以及程式碼自動完成。

對 Yii 2.0 來說，用 IDE 來開發已經相當合適，不過一個例外是針對自己定義的應用程式元件，像是`Yii::$app->mycomponent->something`。IDE 比較難以知道這些自己定義元件的存在，並自動完成程式碼。

## 使用自定義 Yii 類別 {#using-custom-yii-class}

給予 IDE 這些類別存在提示的最好方法，是建立一個實際上沒有運作的`Yii`檔。這個檔案命名為 `Yii.php`，內容如下：

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

上面針對`BaseApplication`、`WebApplication`、`ConsoleApplication`裡面，透過`@property`來標記所使用元件的PHPDoc 註解，會被 IDE 用來作為自動生成程式碼的參考。

> **備註：**為了避免這份檔案與實際的 Yii.php 檔衝突，讓 PHPStorm  提示「Multiple Implementations」警告 ， 我們可以將 `vendor/yiisoft/yii2/Yii.php`檔，也就是實際運作的檔案，標註為「純文字檔案」或者不讓 IDE 參考。這樣也能讓自動產生的速度加快。

好了，現在`Yii::$app->user`不再是預設的user，而是註解內的`\app\components\User`元件。 其他`@property`裡面宣告的元件也一樣。

## 自動產生 Custom Yii 類別 {#custom-yii-class-autogeneration}

我們可以使用 Yii 應用程式的config，來自動產生我們定義的`Yii`類別。這邊請參考 [bazilio91/yii2-stubs-generator](https://github.com/bazilio91/yii2-stubs-generator) 套件。

### 自定義使用者元件 {#customizing-user-component}

要自動產生使用者元件，也就是`Yii::$app->user->identity`，`app\components\User`類別應該看起來如下：

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



