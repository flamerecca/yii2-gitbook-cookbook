# 動態改變佈景（Switching themes dynamically） {#switching-themes-dynamically}

View themes are useful for overriding extension views and making special view versions。[Yii 官方教學](http://www.yiiframework.com/doc-2.0/guide-output-theming.html) 已經指導我們怎麼在靜態網頁裡面設置佈景。這邊我們指導如何動態的改變佈景。

## 準備 {#preparations}

我們安裝  [基本 Yii 應用套件](http://www.yiiframework.com/doc-2.0/guide-start-installation.html)。確定安裝成功並能正常運作。

## 目標 {#the-goal}

這裡我們簡單的使用 GET 參數來改變佈景，像是 `themed=1`。

## 作法 {#how-to-do-it}

只要在 render 樣板之前，我們可以在任何時間點改變佈景。

為求清楚，我們這裡在`controllers/SiteController.php`的action裡面做修改：

```php
public function actionIndex()
{
    if (Yii::$app->request->get('themed')) {
        Yii::$app->getView()->theme = new Theme([
            'basePath' => '@app/themes/basic',
            'baseUrl' => '@web/themes/basic',
            'pathMap' => [
                '@app/views' => '@app/themes/basic',
            ],
        ]);
    }
    return $this->render('index');
}
```

如果`themed`GET 參數we're configuring current a theme which takes view templates from`themes/basic`directory。現在我們在`themes/basic/site/index.php`裡面，加入自己的佈景樣板：

```
Hello, I'm a custom theme!
```

好了，現在我們可以看看在有或沒有 `themed`GET 參數的狀況下，看首頁會有什麼樣的改變。

