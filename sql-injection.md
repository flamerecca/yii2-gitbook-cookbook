A SQL injection exploit can modify a database data. Please, always validate all input on the server. The following examples shows how to build parameterized queries:

#### Example \#1: {#example-1}

```php
$user = Yii::$app->db->createCommand('SELECT * FROM user WHERE id = :id')
           ->bindValue(':id', 123, PDO::PARAM_INT)
           ->queryOne();

```

#### Example \#2: {#example-2}

```php
$params = [':id' => 123];

$user  = Yii::$app->db->createCommand('SELECT * FROM user WHERE id = :id')
           ->bindValues($params)
           ->queryOne();

$user  = Yii::$app->db->createCommand('SELECT * FROM user WHERE id = :id', $params)
           ->queryOne();
```

#### Example \#3: {#example-3}

```php
$command = Yii::$app->db->createCommand('SELECT * FROM user WHERE id = :id');

$user = $command->bindValue(':id', 123)->queryOne();

```

#### Example \#4: Wrong: don't do this! {#example-4-wrong-dont-do-this}

```php
// Wrong: don't do this!
$user = Yii::$app->db->createCommand('SELECT * FROM user WHERE id = ' . $_GET['id'])->queryOne();
Next  Previous

```



