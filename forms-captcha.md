# 使用與客製化 CAPTCHA（Using and customizing CAPTCHA）

[根據維基百科](https://zh.wikipedia.org/wiki/验证码)，CAPTCHA 代表「Completely Automated Public Turing test to tell Computers and Humans Apart」（全自動區分電腦和人類的公開圖靈測試）。簡單說，CAPTCHA 提供一個人可以簡單回答，但是機器不行的問題。這個東西的目的，是在於避免程式濫用某些功能，比方說自動留下包含惡意網址的留言，或者在網頁投票上面洗票……等等。

一個電腦還是頗為棘手的問題是圖形辨識（image recognition）。這就是為什麼常見的 CAPTCHA 總是顯示一張包含文字的圖片，然後請使用者輸入該圖片的文字。

## 怎麼在表單中加入 CAPTCHA {#how-add-captcha-to-a-form}

Yii 提供一些類別，用來實做於表單中加入CAPTCHA的功能。我們來複習一下怎麼作。

首先，我們要有一個 action 函式來顯示含有文字的圖片。一般這會放在`SiteController`裡面。 既然可以使用action，我們就放在`actions()`內：

```php
class SiteController extends Controller
{
    // ...
    public function actions()
    {
        return [
            // ...
            'captcha' => [
                'class' => 'yii\captcha\CaptchaAction',
                'fixedVerifyCode' => YII_ENV_TEST ? 'testme' : null,
            ],
        ];
    }

    // ...
}
```

上面的程式碼裡，我們使用`yii\captcha\CaptchaAction`作為`site/captcha`route，並設置`fixedVerifyCode`is set for test environment in order for the test to know which answer is correct.

現在在 form 模型裡面，我們加上一個\(it could be either ActiveRecord or Model\) we need to add a property that will contain user input for verification code and validation rule for it:

```php
class ContactForm extends Model
{
    // ...
    public $verifyCode;

    // ...
    public function rules()
    {
        return [
            // ...
            ['verifyCode', 'captcha'],
        ];
    }

    // ...
}
```

然後，我們在表單裡面，顯示圖片以及驗證輸入欄位：

```php
<?php $form = ActiveForm::begin(['id' => 'contact-form']); ?>
    // ...
    <?= $form->field($model, 'verifyCode')->widget(Captcha::className()) ?>
    // ...
<?php ActiveForm::end(); ?>
```

好了，現在機器人沒法自動貼文了，起碼笨的機器人不行。

If the image is not being displayed a good way to test if captcha requirements are installed is by accessing the captcha action directly. So for example if you are using a controller called site try typing in "[http://blah.com/index.php/site/captcha](http://blah.com/index.php/site/captcha)" which should display an image. If not then turn on tracing and check for errors.

## 簡單數學的 captcha {#simple-math-captcha}

現在的 CAPTCHA 破解機器人很先進，對於圖形辨識的問題越來越拿手了。所以使用傳統的圖形驗證 CAPTCHA 雖然可以降低機器人的貼文，但是還是有部份的程式可以成功破解這些驗證。

要解決這個問題，必須提昇 CAPTCHA 的難度。In order to prevent it we have to increase the challenge. We could add extra ripple and special effects to the letters on the image but while it could make it harder for computer, it certainly will make it significantly harder for humans which isn't really what we want.

A good solution for it is to mix a custom task into the challenge. Example of such task could be a simple math question such as "2 + 1 = ?". Of course, the more unique this question is, the more secure is the CAPTCHA.

Let's try implementing it. Yii CAPTCHA is really easy to extend. The component itself doesn't need to be touched since both code generation, code verification and image generation happens in`CaptchaAction`which is used in a controller. In basic project template it's used in`SiteController`.

那麼首先，建立`components/MathCaptchaAction.php`：

```php
<?php
namespace app\components;

use yii\captcha\CaptchaAction;

class MathCaptchaAction extends CaptchaAction
{
    public $minLength = 0;
    public $maxLength = 100;

    /**
     * @inheritdoc
     */
    protected function generateVerifyCode()
    {
        return mt_rand((int)$this->minLength, (int)$this->maxLength);
    }

    /**
     * @inheritdoc
     */
    protected function renderImage($code)
    {
        return parent::renderImage($this->getText($code));
    }

    protected function getText($code)
    {
        $code = (int)$code;
        $rand = mt_rand(min(1, $code - 1), max(1, $code - 1));
        $operation = mt_rand(0, 1);
        if ($operation === 1) {
            return $code - $rand . '+' . $rand;
        } else {
            return $code + $rand . '-' . $rand;
        }
    }
}
```

In the code above we've adjusted code generation to be random number from 0 to 100. During image rendering we're generating simple math expression based on the current code.

Now what's left is to change default captcha action class name to our class name in`controllers/SiteController.php`,`actions()`method:

```php
public function actions()
{
    return [
        // ...
        'captcha' => [
            'class' => 'app\components\MathCaptchaAction',
            'fixedVerifyCode' => YII_ENV_TEST ? '42' : null,
        ],
    ];
}
```



