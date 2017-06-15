# Using slugs {#using-slugs}

即使使用了 pretty URLs套件，下面這種網址

```
http://example.com/post/42
```

看起來依舊不是很友善。

使用 Yii，不需要很多功夫就可以把網址改成像是：

```
http://example.com/post/hello-world
```

## 準備 {#preparations}

設置好資料庫，並建立以下表格：

```
post
====
id
title
content
```

使用 Gii 生成 `Post`模型，與對應的 CRUD 操作。

## 作法 {#how-to-do-it}

在`post`表格裡面加入`slug`欄位。 然後在模型裡面加入 sluggable behavior：

```php
<?php
use yii\behaviors\SluggableBehavior;

// ...

class Post extends ActiveRecord
{
    // ...

    public function behaviors()
    {
        return [
            [
                'class' => SluggableBehavior::className(),
                'attribute' => 'title',
            ],
        ];
    }

    // ...
}
```

現在，每當我們建立文章時，資料庫的`slug`欄位也會自動更新。

修改控制器（controller）的部份，加入以下函式：

```php
protected function findModelBySlug($slug)
{
    if (($model = Post::findOne(['slug' => $slug])) !== null) {
        return $model;
    } else {
        throw new NotFoundHttpException();
    }
}
```

改變`actionview()`函式：

```php
public function actionView($slug)
{
    return $this->render('view', [
        'model' => $this->findModelBySlug($slug),
    ]);
}
```

如果要產生連接文章的網址，需要傳遞對應的 slug ：

```php
echo Url::to(['post/view', 'slug' => $post->slug]);
```

## 標題改變處理 {#handling-title-changes}

有很多方式可以處理標題改變的狀況。其中一種方式是在網址裡面加入文章 ID，並利用 ID 來找文章內容

```
http://example.com/post/42/hello-world
```



