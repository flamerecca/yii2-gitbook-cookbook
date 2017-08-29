# 使用 yandex 作為翻譯來源（using yandex as translation source） {#using-machine-translation-as-translation-source}

有時候我們需要將網頁轉換成我們不懂的語言。這時候機器翻譯，像是 Google 或者 Yandex，起碼用來測試時可能是個解決方法。雖然這邊不建議在正式環境下使用機器翻譯的網頁，但是機器翻譯可以幫助測試 UI，並作為一個過渡期，讓整個專案能未來使用更完整的翻譯。

## 環境建立 {#how-to-do-it}

這裡，我們實做使用 Yandex APIs 來作為機器翻譯的範例。 我們需要回應`MissingTranslationEvent`。 這邊使用 component 裡面的 event handler函式會更方便，所以會繼承`\yii\base\Component`。儲存翻譯的部份，因為我們有使用`DbMessageSource`，所以[需要提供適當的資料庫遷移](http://www.yiiframework.com/doc-2.0/yii-i18n-dbmessagesource.html)。

建立`YandexTranslation`元件如下：

```php
namespace common\components;

use yii\db\Query;
use yii\helpers\ArrayHelper;
use yii\i18n\MissingTranslationEvent;

class YandexTranslation extends \yii\base\Component
{
        /**
         * Provider API key. Get it for free at
         * @link https://tech.yandex.com/keys/get/?service=trnsl
         */
        public $apiKey;

        /**
         * @param MissingTranslationEvent $event
         */
        public function handleMissingTranslation(MissingTranslationEvent $event)
        {
                /* @var $messageSource \yii\i18n\DbMessageSource */
                $messageSource = $event->sender;
                $db = $messageSource->db;

                // Extract first part of "en-EN" form
                $sourceLang = explode('-', $messageSource->sourceLanguage)[0];
                $targetLang = explode('-', $event->language)[0];

                $content = file_get_contents('https://translate.yandex.net/api/v1.5/tr.json/translate?' . http_build_query([
                        'key' => $this->apiKey,
                        'text' => $event->message,
                        'format' => 'plain',
                        'lang' => "$sourceLang-$targetLang",
                ]));
                if ($result = json_decode($content, true)) {
                        $event->translatedMessage = ArrayHelper::getValue($result, 'text.0');
                        if ($event->translatedMessage) {
                            // try to find message source
                            $id = (new Query())->select(['id'])
                                ->from($messageSource->sourceMessageTable)
                                ->where(['category' => $event->category, 'message' => $event->message])
                                ->createCommand()
                                ->queryScalar();
                            // if not found, insert a new one. Note: it's better to use "upsert" command (depending of the used DB engine)
                            if (!$id) {
                                $db->createCommand()->insert($messageSource->sourceMessageTable, [
                                        'category' => $event->category, 
                                        'message' => $event->message            
                                ])->execute();
                                $id = $db->getLastInsertId()
                            }
                            // insert new translated message.
                            $db->createCommand()->insert($messageSource->messageTable, [
                                    'id' => $id,
                                    'language' => $event->language,
                                    'translation' => $event->translatedMessage
                            ])->execute();
                        }
                }
                if (!$event->translatedMessage) {
                        $event->translatedMessage = "@MISSING: {$event->category}.{$event->message} FOR LANGUAGE {$event->language} @";
                }
                return null;
        }
}
```

然後，在 config 裡面設置翻譯器：

```php
'components' => [
    // ...
     'i18n' => [
        'translations' => [
            'machine' => [
                'class' => 'yii\i18n\DbMessageSource',
                'on missingTranslation' => [
                    new common\components\YandexTranslation([
                        'apiKey' => 'some api key',
                    ]), 
                    'handleMissingTranslation'
                ],
            ],
        ],
    ],
],
```

## 使用方法 {#how-it-works}

要使用我們設置好的翻譯器，在 Yii 的`t`函式裡面，加入`machine`類別，像是：`Yii::t('machine', 'Text to translate')`。  
因為翻譯的速度並不快，記得要使用 cache，以免網頁速度受到嚴重影響。

