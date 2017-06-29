驗證碼生成因為使用與客製化 CAPTCHA（Using and customizing CAPTCHA）

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

上面的程式碼裡，我們使用`yii\captcha\CaptchaAction`作為`site/captcha`的路徑，並設置`fixedVerifyCode`，作為測試環境時固定正確答案用。 現在在 form 模型裡面（也可以是其他的模型或者 ActiveRecord），我們加上一個參數`$verifyCode`來儲存使用者輸入，以加入驗證 CAPTCHA 正確的規則：

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

如果圖片沒有成功送出，一個測試的好方法，是直接存取圖片對應的 action 來偵測錯誤。 假設生成圖片的控制器是 site，可以測試「[http://我們的網址/index.php/site/captcha](http://blah.com/index.php/site/captcha)」來看看有沒有圖片。 如果沒有成功，可以開啟追蹤功能，來尋找錯誤的部份。

## 簡單數學的 captcha {#simple-math-captcha}

現在的 CAPTCHA 破解機器人很先進，對於圖形辨識的問題越來越拿手了。所以使用傳統的圖形驗證 CAPTCHA，雖然可以降低機器人的貼文，但是還是有部份的程式可以成功破解這些驗證。

要解決這個問題，必須提昇 CAPTCHA 的難度。我們可以透過增加圖片的雜訊來提昇辨識難度，這樣能成功的讓電腦辨識圖片的難度提高。但是這麼做的同時，也會讓使用者更難以辨識圖上的文字，而這不是很符合我們的期望。

另一個不錯的解決方法，是加入一些自定義的問題。比方說圖片顯示的不是單純的文字，而是簡單的數學問題，像是「2 + 1 = ?」。當然，問題越是特殊，CAPTCHA的安全度就越高。

我們來實做看看。Yii 的 CAPTCHA 非常容易擴充，CAPTCHA component 本身不需要更改，因為驗證碼生成、驗證碼確認以及圖片生成都放在`CaptchaAction`裡面。在基本套件的 Yii 內，會將`CaptchaAction`放在`SiteController`裡面。

首先，建立`components/MathCaptchaAction.php`：

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

上面的程式裡面，我們將`generateVerifyCode()`修改成產出一到一百的隨機亂數。在生成圖片的部份，使用`getText()`產生出一個簡單的數學問題，然後用`renderImage()`生成圖片。

然後，我們要把`controllers/SiteController.php`裡面原本預設的 captcha 行為，改成我們自己設計的行為。在`actions()`函式裡面：

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



