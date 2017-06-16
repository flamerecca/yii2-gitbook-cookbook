# 可變參數的網址（URLs with variable number of parameters） {#urls-with-variable-number-of-parameters}

許多時候，我們需要從網址取得的不同數目的參數。舉例來說，從網址`http://example.com/products/cars/sport`裡面，我們可能希望會進入`ProductController::actionCategory`，並得到包含`cars`和`sport的陣列。`

## 準備 {#get-ready}

首先，我們要啟用 pretty URLs。首先我們在 config 檔裡面加入：

```php
$config = [
    // ...
    'components' => [
        // ...
        'urlManager' => [
            'showScriptName' => false,
            'enablePrettyUrl' => true,
            'rules' => require 'urls.php',
        ],
    ]
```

注意這邊，我們在`rule`使用分開的 PHP 檔案，而不是直接寫在config檔案裡面。這在程式成長的時候會很有幫助。

然後在`config/urls.php`裡面，加上以下規則：

```php
<?php
return [
    [
        'pattern' => 'products/<categories:.*>',
        'route' => 'product/category',
        'encodeParams' => false,
    ],
];
```

現在建立`ProductController`：

```php
namespace app\controllers;

use yii\web\Controller;

class ProductController extends Controller
{
    public function actionCategory($categories)
    {
        $params = explode('/', $categories);
        print_r($params);
    }
}
```

好了，如果現在你進入`http://example.com/products/cars/sport`，你會得到：

```
Array ( [0] => cars [1] => sport)
```



