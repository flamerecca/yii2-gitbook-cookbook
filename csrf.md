# 跨網站請求偽造（CSRF） {#csrf}

跨網站請求偽造 \(Cross-site request Forgery, CSRF\) 是一種典型的網站應用弱點。此種攻擊是基於以下的假設: 使用者已在某些合法網站通過驗證\(A網站\)，之後他又造訪了攻擊者的網站\(B網站\)。B網站利用JavaScript、表單、&lt;img src=" 標籤或者其他任何方式對合法的A網站發出請求，攻擊者可以利用這種方法更改受害者的密碼，或者從受害者的銀行戶頭把轉錢出來\(當然，這裡是假設銀行網站不安全\)。

要避免這種攻擊，[其中一種方法](https://www.owasp.org/index.php/Cross-Site_Request_Forgery_%28CSRF%29_Prevention_Cheat_Sheet)就是使用一個特殊的token來驗證請求的來源。Yii預設對於form請求就是這麼做的。你可以在你的view裡面加入一個簡單的form然後送出試試看:

```HTML
<form method="POST">
<input type="submit" value="ok!">
</form>
```

你會得到如下的錯誤:

```
Bad Request (
#400
)
Unable to verify your data submission.
```

這是因為Yii對於每一個請求都會產生一個特殊的獨一無二的token，你必須將此token伴隨著其他資料一起送出。每當你送出請求Yii就會把產生出來的和接收到的token做比較。如果兩者相符Yii就會繼續處理請求。如果不相符或者送出的資料裡面沒有CSRF token，錯誤就會跑出來了。

這個由我們自己產生出來的token是攻擊者網站\(B網站\)不知道的，所以他不可能偽造你的請求。

有一個地方要提醒，是關於GET這種請求method。以上述為例，我們改成以下的form然後送出:

```HTML
<form method="GET">
<input type="submit" value="ok!">
</form>
```

沒有任何錯誤訊息。這是因為CSRF保護只對不安全的請求method有用，例如PUT、POST 或 DELETE。所以若想要讓你的網站應用安全，對於會改變應用狀態的請求，你不應該使用GET。

## 關閉CSRF保護 {#disabling-csrf-protection}

有時候你可能需要停用CSRF驗證。若要關閉CSRF驗證，只要將Controller的 $enableCsrfValidation 特性設為 false 就可以了:

```php
class MyController extends Controller
{   
public $enableCsrfValidation = false;
```

如果是要對某些action開啟，用以下代碼: :

```php
class MyController extends Controller
{   
    public function beforeAction ($action)
    {       
        if (in_array($action->id, ['incoming'])) {
            $this->enableCsrfValidation = false;
        }
    return parent::beforeAction($action);
}
```

詳細資訊可參考[處理第三方來的POST請求](https://yii2-cookbook.readthedocs.io/incoming-post/)

## 添加CSRF保護 {#adding-csrf-protection}

CSRF保護預設是開啟的，所以你只需要讓token伴隨你的請求一起送出就可以的，通常是透過隱藏欄位:

```HTML
<form method="POST">
    <input id="form-token" type="hidden" name="<?=Yii::$app->request->csrfParam?>" 
           value="<?=Yii::$app->request->csrfToken?>"/>
    <input type="submit" value="ok!">
</form>
```

如果有用到 ActiveForm，Yii會自動加入token，你不需要自己加這個隱藏欄位。

## 另外閱讀 {#see-also}

* [OWASP article about CSRF](https://www.owasp.org/index.php/Cross-Site_Request_Forgery_%28CSRF%29_Prevention_Cheat_Sheet)
* [Handling incoming third party POST requests](https://yii2-cookbook.readthedocs.io/incoming-post/)
  .



