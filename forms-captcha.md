# 使用與客製化 CAPTCHA（Using and customizing CAPTCHA）

[根據維基百科](https://zh.wikipedia.org/wiki/%E9%AA%8C%E8%AF%81%E7%A0%81)，CAPTCHA 代表「Completely Automated Public Turing test to tell Computers and Humans Apart」（全自動區分電腦和人類的公開圖靈測試）。簡單說，CAPTCHA 提供一個人可以簡單回答，但是機器不行的問題。這個東西的目的，是在於避免程式濫用某些功能，比方說自動留下包含惡意網址的留言，或者在網頁投票上面洗票……等等。

一個電腦還是頗為棘手的問題是圖形辨識（image recognition）。這就是為什麼常見的 CAPTCHA 總是顯示一張包含文字的圖片，然後請使用者輸入該圖片的文字。

## 怎麼在表單中加入 CAPTCHA {#how-add-captcha-to-a-form}

Yii provides a set of ready to use classes to add CAPTCHA to any form. Let's review how it could be done.

First of all, we need an action that will display an image containing text. Typical place for it is`SiteController`. Since there's ready to use action, it could be added via`actions()`method:

```
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

In the above we're reusing`yii\captcha\CaptchaAction`as`site/captcha`route.`fixedVerifyCode`is set for test environment in order for the test to know which answer is correct.

Now in the form model \(it could be either ActiveRecord or Model\) we need to add a property that will contain user input for verification code and validation rule for it:

```
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

Now we can actually display image and verification input box in a view containing a form:

```
<?php $form = ActiveForm::begin(['id' => 'contact-form']); ?>
    // ...
    <?= $form->field($model, 'verifyCode')->widget(Captcha::className()) ?>
    // ...
<?php ActiveForm::end(); ?>
```

That's it. Now robots won't pass. At least dumb ones.

If the image is not being displayed a good way to test if captcha requirements are installed is by accessing the captcha action directly. So for example if you are using a controller called site try typing in "[http://blah.com/index.php/site/captcha](http://blah.com/index.php/site/captcha)" which should display an image. If not then turn on tracing and check for errors.

## 簡單數學的 captcha {#simple-math-captcha}

Nowadays CAPTCHA robots are relatively good at parsing image so while by using typical CAPTCHA you're significanly lowering number of spammy actions, some robots will still be able to parse the image and enter the verification code correctly.

In order to prevent it we have to increase the challenge. We could add extra ripple and special effects to the letters on the image but while it could make it harder for computer, it certainly will make it significantly harder for humans which isn't really what we want.

A good solution for it is to mix a custom task into the challenge. Example of such task could be a simple math question such as "2 + 1 = ?". Of course, the more unique this question is, the more secure is the CAPTCHA.

Let's try implementing it. Yii CAPTCHA is really easy to extend. The component itself doesn't need to be touched since both code generation, code verification and image generation happens in`CaptchaAction`which is used in a controller. In basic project template it's used in`SiteController`.

So, first of all, create`components/MathCaptchaAction.php`:

```
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

```
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



