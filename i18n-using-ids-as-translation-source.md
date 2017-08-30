# Using IDs as translation source {#using-ids-as-translation-source}

Default way of translating content in Yii is to use English as a source language. It is quite convenient but there's another convenient way some developers prefer: using IDs such as`thank.you`.

## 環境建立 {#how-to-do-it}

Using keys is very easy. In order to do it, specify`sourceLanguage`as`key`in your message bundle config and then set`forceTranslation`to`true`:

```php
'components' => [
    // ...
    'i18n' => [
        'translations' => [
            'app*' => [
                'class' => 'yii\i18n\PhpMessageSource',
                //'basePath' => '@app/messages',
                'sourceLanguage' => 'key', // <--- 這裡
                'forceTranslation' => true, // <--- 以及這裡
                'fileMap' => [
                    'app' => 'app.php',
                    'app/error' => 'error.php',
                ],
            ],
        ],
    ],
],
```

Note that you have to provide translation for the application language \(which is`en_US`by default\) as well.

## 使用方法 {#how-it-works}

將`sourceLanguage`設置成`key`，會保證沒有語言符合，並強迫執行翻譯。

