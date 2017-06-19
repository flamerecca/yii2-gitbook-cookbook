# 使用客製化的遷移模板（Using custom migration template） {#using-custom-migration-template}

很多時候，我們會需要客製化 `./yii migrate/create`時運作的程式碼。一個例子是遷移到 MySQL InnoDB，我們需要指定引擎的時候。

## 作法 {#how-to-do-it}

首先，複製標準模板`framework/views/migration.php`到你的程式裡面， 比方說移動到`views/migration.php`.

然後修改模板：

```php
<?php
/**
 * This view is used by console/controllers/MigrateController.php
 * The following variables are available in this view:
 */
/* @var $className string the new migration class name */

echo "<?php\n";
?>

use yii\db\Migration;

class <?= $className ?> extends Migration
{
    public function up()
    {
        $tableOptions = null;
        if ($this->db->driverName === 'mysql') {
            // http://stackoverflow.com/questions/766809/whats-the-difference-between-utf8-general-ci-and-utf8-unicode-ci
            $tableOptions = 'CHARACTER SET utf8 COLLATE utf8_unicode_ci ENGINE=InnoDB';
        }


    }

    public function down()
    {
        echo "<?= $className ?> cannot be reverted.\n";

        return false;
    }

    /*
    // Use safeUp/safeDown to run migration code within a transaction
    public function safeUp()
    {
    }

    public function safeDown()
    {
    }
    */
}
```

現在，在 config 檔裡面，像是 `config/console.php`標記該使用的新模板：

```php
return [
...

    'controllerMap' => [
        'migrate' => [
            'class' => 'yii\console\controllers\MigrateController',
            'templateFile' => '@app/views/migration.php',
        ],
    ],

...
];
```

好了，現在如果執行`./yii migrate/create`會使用剛剛定義的模板，而不是標準模板。

