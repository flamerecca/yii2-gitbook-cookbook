# 使用 ID 作為預設語言（Using IDs as translation source） {#using-ids-as-translation-source}

Yii 翻譯語言的預設值，是用英文作為原始語言，這設定方式在大多時候很方便。

但是另一個也很方便的方式，是將翻譯語言設置成某個開發者自定義的ID，比方說`thank.you`。

## 環境建立 {#how-to-do-it}

使用ID當作語言非常簡單。我們只需要將`sourceLanguage`設置成`key`，並且將`forceTranslation`設置成`true`：

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

注意，這樣設置之後，我們對應用程式本身語言，通常是`en_US`，也必須提供翻譯內容，這樣才能正常運作。

## 使用方法 {#how-it-works}

將`sourceLanguage`設置成`key`，會保證沒有語言符合，並強迫執行翻譯。

