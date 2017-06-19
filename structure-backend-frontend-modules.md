===========================================

[https://github.com/yiisoft/yii2/issues/3647](https://github.com/yiisoft/yii2/issues/3647)

By default Yii 預設已有進階模組，可以建立後台與前台系統。一般來說這是後台系統需求的好解法。除非你需要重新分配你的部份檔案配置。在這種狀況下，使用模組會是一個很好的解決辦法。

## 資料夾結構 {#directory-structure}

```
common
  components
  models
backend
  controllers
  views
  Module
frontend
  controllers
  views
  Module
```

## 命名空間 {#namespaces}

命名空間的位置與其他擴充相同。

以`samdark\blog`為例（需要在`composer.json`裡面有PSR-4 紀錄）共用的檔案位於`samdark\blog\common`。後台模組位於`samdark\blog\backend\Module`，前台模組則位於`samdark\blog\frontend\Module`。

## 使用 {#using-it}

* 透過 composer 安裝
* 在 config 裡面使用以下模組：

```php
'modules' => [
  'blogFrontend' => [
    'class' => 'samdark\blog\frontend\Module',
    'anonymousComments' => false,
  ],
  'blogBackend' => [
    'class' => 'samdark\blog\backend\Module',
  ],
]
```

* 透過瀏覽器使用：

```
http://example.com/blog-frontend/post/view?id=10
http://example.com/blog-backend/user/index
```



