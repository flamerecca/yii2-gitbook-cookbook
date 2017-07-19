# Reusing views via partials {#reusing-views-via-partials}

一個開發很重要的原則是「不要自我重複」（don't repeat yourself，DRY）。 重複的程式碼在開發期間到處都會出現，包括視圖的部份，這邊我們建立一些可重複使用的視圖，來解決這問題。

## 建立 partial view {#creating-partial-view}

下面是常見的`views/site/index.php`程式碼：

```php
<?php
/* @var $this yii\web\View */
$this->title = 'My Yii Application';
?>
<div class="site-index">
<div class="jumbotron">
    <h1>Congratulations!</h1>
    <p class="lead">You have successfully created your Yii-powered application.</p>
    <p><a class="btn btn-lg btn-success" href="http://www.yiiframework.com">Get started with Yii</a></p>
</div>
<div class="body-content">
//...
```

舉例來說，我們希望`<div class="jumbotron">`HTML 區塊的部份，可以同時出現在首頁與「關於我們」`views/site/about.php`裡面。並且這段區塊不要自我重複。

我們來建立獨立的視圖檔`views/site/_jumbotron.php`，並把下面的HTML放進來：

```php
<div class="jumbotron">
    <h1>Congratulations!</h1>
    <p class="lead">You have successfully created your Yii-powered application.</p>
    <p><a class="btn btn-lg btn-success" href="http://www.yiiframework.com">Get started with Yii</a></p>
</div>
```

## 使用 partial view {#using-partial-view}

將原本`views/site/index.php`內，`<div class="jumbotron">`HTML 區塊的東西，改成如下：

```php
<?php
/* @var $this yii\web\View */
$this->title = 'My Yii Application';
?>
<div class="site-index">
<?=$this->render('_jumbotron.php')?>; // 取代原本的 jumbotron 區塊
<div class="body-content">
//...
```

在「關於我們」`views/site/about.php`裡面，加入一樣的程式碼（也可以加在其他的視圖內）：

```php
<?php
use yii\helpers\Html;
/* @var $this yii\web\View */
$this->title = 'About';
$this->params['breadcrumbs'][] = $this->title;
?>
<div class="site-about">
    <?=$this->render('_jumbotron.php')?>; // 取代原本的 jumbotron 區塊
```

上面的程式碼內，我們靠[`View::render()`](http://www.yiiframework.com/doc-2.0/yii-base-view.html#render%28%29-detail)函式生成指定的 which renders a view specified and returns its output which we're echoing immediately.

## 加入變數 {#adding-variables}

Let's customize message displayed in jumbotron. By default it will be the same message but user should be able to pass custom message via`message`parameter.

首先， 我們自製的`views/site/_jumbotron.php`，改成：

```php
<?php
$message = isset($message) ? $message : 'You have successfully created your Yii-powered application.';
?>
<div class="jumbotron">
    <h1>Congratulations!</h1>
    <p class="lead"><?= $message ?></p>
    <p><a class="btn btn-lg btn-success" href="http://www.yiiframework.com">Get started with Yii</a></p>
</div>
```

然後在「關於我們」裡面，傳入`message`參數：

```php
<?php
use yii\helpers\Html;
/* @var $this yii\web\View */
$this->title = 'About';
$this->params['breadcrumbs'][] = $this->title;
?>
<div class="site-about">
    <?=$this->render('_jumbotron.php', [
        'message' => 'This is about page!',
    ])?>; // 我們的程式碼
```



