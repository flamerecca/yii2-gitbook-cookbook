# SQL注入攻擊（SQL injection） {#sql-injection}

SQL注入攻擊會影響整個資料庫的內容。所以一定要驗證從資料庫傳來的指令。 

下面的例子示範如何透過Yii的函式，建立安全的資料庫指令：

#### 範例 \#1: {#example-1}

```php
$user = Yii::$app->db->createCommand('SELECT * FROM user WHERE id = :id')
           ->bindValue(':id', 123, PDO::PARAM_INT)
           ->queryOne();
```

#### 範例 \#2: {#example-2}

```php
$params = [':id' => 123];

$user  = Yii::$app->db->createCommand('SELECT * FROM user WHERE id = :id')
           ->bindValues($params)
           ->queryOne();

$user  = Yii::$app->db->createCommand('SELECT * FROM user WHERE id = :id', $params)
           ->queryOne();
```

#### 範例 \#3: {#example-3}

```php
$command = Yii::$app->db->createCommand('SELECT * FROM user WHERE id = :id');

$user = $command->bindValue(':id', 123)->queryOne();
```

#### 範例 \#4: 錯誤示範！不要這樣做！ {#example-4-wrong-dont-do-this}

```php
// 錯誤示範！不要這樣做！
$user = Yii::$app->db->createCommand('SELECT * FROM user WHERE id = ' . $_GET['id'])->queryOne();
```



