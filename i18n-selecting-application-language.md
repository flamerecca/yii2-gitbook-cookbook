# Selecting application language {#selecting-application-language}

當我們開發國際化網站時，總是需要支援多種語言。Yii 有內建的has built in solution for handling message translations ，但是沒有提供語言選項的方式，因為該需求的實做會隨著網站不同而有相當的改變。

這邊， we’ll describe some typical cases of language selection and provide ideas and code snippets so you’ll be able to pick what’s required and implement it in your project.

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

Note that it should be done every request before any output in order for outputted content to be affected. Good places to consider are custom`UrlManager`, custom`UrlRule`, controller's or module's`beforeAction()`or application bootstrap.

## 自動偵測使用語言 {#detecting-language-automatically}

Detecting language automatically could help your application to conquer international markets if done properly. The following code shows selecting a language using information supplied by user’s browser and a list of languages your application supports:

```php
$supportedLanguages = ['en', 'ru'];
$languages = Yii::$app->request->getPreferredLanguage($supportedLanguages);
```

Note that language should be set prior to controller action so it’s a good idea to create language selection component:

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

In order to use the component you should specify it in the application config like the following:

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

As was mentioned above, it could be implemented in custom`UrlManager`, custom`UrlRule`or controller's / module's`beforeAction()`instead.

## 手動選擇支援的語言 {#support-selecting-language-manually}

While it sounds like a great idea to always detect language, it’s usually not enough. Detection could fail so user will get language he doesn’t know, user may know many languages but prefer, for example, English for information about travelling. These problems could be solved by providing visible enough language selector that somehow remembers what was selected and uses it for the application further.

解決問題時，要處理三個部份：

1. 語言選擇
2. 儲存該選擇
3. 重複使用該選擇

Let’s start with language selector widget. Overall it’s a simple select widget pre-filled with an array of language code =&gt; language name pairs.

```php
<?= Html::beginForm() ?>
<?= Html::dropDownList('language', Yii::$app->language, ['en-US' => 'English', 'zh-CN' => 'Chinese']) ?>
<?= Html::submitButton('Change') ?>
<?= Html::endForm() ?>
```

Form handling should be done in controller. A good place to do it is`SiteController::actionLanguage`:

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

Now we can improve`LanguageSelector`a bit:

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

目前為止， we’ve found a way to detect language, select it manually and store it. For intranet applications and applications for which search engine indexing isn’t important, it is already enough. For others you need to expose each application language to the world.

The best way to do it is to include language in the URL such as`http://example.com/ru/about`or subdomain such as`http://ru.example.com/about`.

The most straightforward implementation is about creating URL manager rules for each URL you have. In these rules you need to define language part i.e.:

```php
'<language>/<page>' => 'site/page',
```

The con of this approach is that it is repetitive. You have to define it for all URLs you have and you have to put current language to parameters list each time you’re creating an URL such as:

```php
<?= Html::a('DE', ['post/view', 'language' => 'de']); ?>
```

Thanks to Yii we have an ability to replace default URL class with our own right from config file:

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

Here’s what language aware URL rule class could look like:

```php
class LanguageUrlRule extends UrlRule
{
    public function init()
    {
        if ($this->pattern !== null) {
            $this->pattern = '<language>/' . $this->pattern;
            // for subdomain it should be:
            // $this->pattern =  'http://<language>.example.com/' . $this->pattern,
        }
        $this->defaults['language'] = Yii::$app->language;
        parent::init();
    }
}
```

### 套件 {#ready-to-use-extension}

[yii2-localeurls 套件](https://github.com/codemix/yii2-localeurls) 實做了一個網址內處理語言上面一個相當可靠，並且彈性很高的方式。

