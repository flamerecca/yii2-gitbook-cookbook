# 處理 cookie {#managing-cookies}

用純 PHP 處理 HTTP cookie 並不困難。不過 Yii 框架讓這件事情更簡易。

下面我們說明如何處理一般的 cookie 行為。

## 設置 cookie {#setting-a-cookie}

要設置 cookie，或者說，要建立一個 cookie 檔案並傳給瀏覽器的話，在 Yii 我們可以建立一個`\yii\web\Cookie`物件，並加入回應cookies 集合：

```php
$cookie = new Cookie([
    'name' => 'cookie_monster',
    'value' => 'Me want cookie!',
    'expire' => time() + 86400 * 365,
]);
\Yii::$app->getResponse()->getCookies()->add($cookie);
```

In the above we're passing parameters to cookie class constructor. 這些參數基本上與 PHP 原本的[setcookie](http://php.net/manual/en/function.setcookie.php) 函式參數相同：

* `name`
  * cookie的名稱
* `value`
  * cookie的值。必須確定是一個字串。如果不是字串的話，可能會讓大多瀏覽器不太開心。
* `domain`
  * cookie 的網域
* `expire`
  * unix 時間戳記，標記 cookie 該被刪掉的時間
* `path`
  * cookie 的伺服器路徑。
* `secure`
  * 設置為`true`的話， cookie 只有在使用 HTTPS 的狀況下才會建立。
* `httpOnly`
  * 設置為`true`的話，cookie 無法透過 JavaScript存取。

## 取得 cookie {#reading-a-cookie}

取得cookie的程式碼如下：

```php
$value = \Yii::$app->getRequest()->getCookies()->getValue('my_cookie');
```

## 在哪邊設置與取得cookie？ {#where-to-get-and-set-cookies}

cookie 是 HTTP 請求的一部分，而網頁請求與回應都屬於控制器（controller）的責任，所以建議把cookie相關的程式碼放在控制器裡面。

## 子網域的 cookie {#cookies-for-subdomains}

因為安全性問題，Because of security reasons, by default cookies are accessible only on the same domain from which they were set. 舉例來說，如果你把cookie的網域是`example.com`，那麼`www.example.com`就不能取得這個cookie。So if you're planning to use subdomains \(i.e. admin.example.com, profile.example.com\), you need to set`domain`explicitly:

```php
$cookie = new Cookie([
    'name' => 'cookie_monster',
    'value' => 'Me want cookie everywhere!',
    'expire' => time() + 86400 * 365,
    'domain' => '.example.com' // <<<=== HERE
]);
\Yii::$app->getResponse()->getCookies()->add($cookie);
```

Now cookie can be read from all subdomains of`example.com`.

## 跨子網域身份驗證和身份cookie {#cross-subdomain-authentication-and-identity-cookies}

In case of autologin or "remember me" cookie, the same quirks as in case of subdomain cookies are applying. But this time you need to configure user component, setting`identityCookie`array to desired cookie config.

Open you application config file and add`identityCookie`parameters to user component configuration:

```php
$config = [
    // ...
    'components' => [
        // ...
        'user' => [
            'class' => 'yii\web\User',
            'identityClass' => 'app\models\User',
            'enableAutoLogin' => true,
            'loginUrl' => '/user/login',
            'identityCookie' => [ // <---- here!
                'name' => '_identity',
                'httpOnly' => true,
                'domain' => '.example.com',
            ],
        ],
        'request' => [
            'cookieValidationKey' => 'your_validation_key'
        ],
        'session' => [
            'cookieParams' => [
                'domain' => '.example.com',
                'httpOnly' => true,
            ],
        ],

    ],
];
```

Note that`cookieValidationKey`should be the same for all sub-domains.

Note that you have to configure the`session::cookieParams`property to have the samedomain as your`user::identityCookie`to ensure the`login`and`logout`work for all subdomains. This behavior is better explained on the next section.

## Session cookie 參數 {#session-cookie-parameters}

Session cookies parameters are important both if you have a need to maintain session while getting from one subdomain to another or when, in contrary, you host backend app under`/admin`URL and want handle session separately.

```php
$config = [
    // ...
    'components' => [
        // ...
        'session' => [
            'name' => 'admin_session',
            'cookieParams' => [
                'httpOnly' => true,
                'path' => '/admin',
            ],
        ],
    ],
];
```

## 其他資料 {#see-also}

* [API reference](http://stuff.cebe.cc/yii2docs/yii-web-cookie.html)
* [PHP documentation](http://php.net/manual/en/function.setcookie.php)
* [RFC 6265](http://www.faqs.org/rfcs/rfc6265.html)



