# Selecting application language {#selecting-application-language}

當我們開發國際化網站時，總是需要支援多種語言。Yii 有內建的訊息翻譯方式，但是沒有提供語言選項的方式，因為該需求的實做會隨著網站不同而有相當的改變。

這邊會解釋語言選擇的幾種狀況，並提供想法和範例程式碼。這樣，我們可以選擇對應的狀況並實做於自己的專案中。

## 如何設置網站語言 {#how-to-set-application-language}

在 Yii 設置應用語言相當簡單。我們可以使用如下的程式碼：

```php
Yii::$app->language = 'ru_RU';
```

或者透過 config，像是`config/main.php`：

```php
return [
    // ...
    'language' => 'ru_RU',
];
```

注意，要讓所有輸出資源都翻譯到，每個使用者提出的要求都必須這樣設置。設置的位置可以在客製的`UrlManager`、客製的`UrlRule`、控制器和模型的 `beforeAction()`，或者應用的 bootstrap 階段。

## 自動偵測使用語言 {#detecting-language-automatically}

如果處理得好，自動偵測語言可以讓你的網站在國際市場更有競爭力。下面的程式碼能讓我們透過使用者瀏覽器的資訊提供建議語言 ，以及如何提供我們建立網站支援的語言：

```php
$supportedLanguages = ['en', 'ru'];
$languages = Yii::$app->request->getPreferredLanguage($supportedLanguages);
```

這邊的語言會在控制器 action 之前選擇好，實做一個語言選擇元件會是個好選擇：

```php
namespace app\components;
use yii\base\BootstrapInterface;
class LanguageSelector implements BootstrapInterface
{
    public $supportedLanguages = [];

    public function bootstrap($app)
    {
        $preferredLanguage = $app->request->getPreferredLanguage($this->supportedLanguages);
        $app->language = $preferredLanguage;
    }
}
```

要使用這個元件，我們還需要設置 config 檔：

```php
return [
    'bootstrap' => [
        [
            'class' => 'app\components\LanguageSelector',
            'supportedLanguages' => ['en_US', 'ru_RU'],
        ],
    ],
    // ...
];
```

上面的程式碼，也可以改成實做在自製的`UrlManager`、自製的`UrlRule`，以及控制器或模組的`beforeAction()`內。

## 手動選擇支援的語言 {#support-selecting-language-manually}

雖然自動偵測語言聽起來是個好主意，不過實際上通常有一些問題。偵測可能會有錯誤，並讓使用者導向某種他無法看懂的語言，或者，將使用者導向某種他能看懂，但是不是首選的語言。比方說，在旅遊資訊的網站上面，將地名等資料翻譯成英文，反而會讓人產生困惑。這些問題可以透過在網頁上提供語言選擇來解決。該功能要能記住使用者選了什麼語言，並9在之後繼續使用.

解決這個問題時，要處理三個部份：

1. 語言選擇
2. 儲存該選擇
3. 重複使用該選擇

首先我們來看語言選擇的套件。 基本上，這是一個簡單的選擇元件 ，內容是「語言代碼 =&gt; 語言名稱」的陣列。

```php
<?= Html::beginForm() ?>
<?= Html::dropDownList('language', Yii::$app->language, ['en-US' => '英文', 'zh-CN' => '中文']) ?>
<?= Html::submitButton('Change') ?>
<?= Html::endForm() ?>
```

表單處理應該放在控制器裡面。一個很適合的地方是`SiteController::actionLanguage`：

```php
$language = Yii::$app->request->post('language');
Yii::$app->language = $language;

$languageCookie = new Cookie([
    'name' => 'language',
    'value' => $language,
    'expire' => time() + 60 * 60 * 24 * 30, // 30 days
]);
Yii::$app->response->cookies->add($languageCookie);
```

上面我們使用 cookie 儲存語言。但是也可以是其他的方式，像是資料庫：

```php
$user = Yii::$app->user;
$user->language = $language;
$user->save();
```

然後我們稍微改進`LanguageSelector`：

```php
namespace app\components;
use yii\base\BootstrapInterface;

class LanguageSelector implements BootstrapInterface
{
    public $supportedLanguages = [];

    public function bootstrap($app)
    {
        $preferredLanguage = isset($app->request->cookies['language']) ? (string)$app->request->cookies['language'] : null;
        // or in case of database:
        // $preferredLanguage = $app->user->language;

        if (empty($preferredLanguage)) {
            $preferredLanguage = $app->request->getPreferredLanguage($this->supportedLanguages);
        }

        $app->language = $preferredLanguage;
    }
}
```

## 使用網址或子網域設置語言 {#language-in-url-subdomain}

目前為止，我們提到了偵測語言，手動選擇語言以及儲存的方法。 對內部網路的應用，或者搜尋引擎的排序不重要的時候來說， 這樣已經相當足夠了。不過有些時候，比方需要讓搜尋引擎好處理的時候，我們需要公開我們目前使用的語言。

最好的方法是將使用的語言包含在網址裡面，像是`http://example.com/ru/about`。  
或者使用子網域，像是 `http://ru.example.com/about`。

最直接的實做方式，是對每個使用的網址，建立網址處理器規則。我們在這些規則裡面，定義語言出現的位置，像是：

```php
'<language>/<page>' => 'site/page',
```

這樣做的缺點是重複性。我們需要重新定義所有的網址，並且每次建立網址時，都要在參數內加上現在使用的語言，像是：

```php
<?= Html::a('DE', ['post/view', 'language' => 'de']); ?>
```

由於 Yii 的機制，we have an ability to replace default URL class with our own right from config file：

```php
return [
    'components' => [
        'urlManager' => [
           'ruleConfig' => [
                'class' => 'app\components\LanguageUrlRule'
            ],
        ],
    ],
];
```

下面是一個能處理語言的網址規則類別看起來的樣子：

```php
class LanguageUrlRule extends UrlRule
{
    public function init()
    {
        if ($this->pattern !== null) {
            $this->pattern = '<language>/' . $this->pattern;
            // 子網域應該是：
            // $this->pattern =  'http://<language>.example.com/' . $this->pattern,
        }
        $this->defaults['language'] = Yii::$app->language;
        parent::init();
    }
}
```

### 套件 {#ready-to-use-extension}

[yii2-localeurls 套件](https://github.com/codemix/yii2-localeurls)實做了一個網址內處理語言上面一個相當可靠，並且彈性很高的方式。

