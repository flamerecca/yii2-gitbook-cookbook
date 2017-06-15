# 使用重新導向（Using redirects） {#using-redirects}

## 301 {#301}

假設我們本來有個頁面`http://example.com/item2`，但是這個頁面已經永遠移動到`http://example.com/item1`了。因為很可能有不少使用者或搜尋引擎已經使用了`http://example.com/item2`，不管是書籤、文章、或者存進資料庫……等等，所以我們不能直接移除 `http://webproject.ru/item2`這個網址。

這時候，我們可以使用 301 重新導向：

```php
class MyController extends Controller
{
    public function beforeAction($action)
    {
        if (in_array($action->id, ['item2'])) {
            Yii::$app->response->redirect(Url::to(['item1']), 301);
            Yii::$app->end();
        }
        return parent::beforeAction($action);
    }
```

如果要更方便，可以增加一個陣列。這樣當你需要增加重導向網址的時候，就只需要多加一個鍵值對（key-value pair）：

```php
class MyController extends Controller
{
    public function beforeAction($action)
    {
        $toRedir = [
            'item2' => 'item1',
            'item3' => 'item1',
        ];

        if (isset($toRedir[$action->id])) {
            Yii::$app->response->redirect(Url::to([$toRedir[$action->id]]), 301);
            Yii::$app->end();
        }
        return parent::beforeAction($action);
    }

```

## 其他資料 {#see-also}

* [Handling trailing slash in URLs](https://yii2-cookbook.readthedocs.io/handling-trailing-slash-in-urls/)



