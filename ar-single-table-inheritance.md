# 單一表格繼承（Single table inheritance） {#single-table-inheritance}

大部分資料庫沒有處理繼承問題，所以網頁設計需要的時候，就必須要自己製作。其中一個方法是[single table inheritance](http://martinfowler.com/eaaCatalog/singleTableInheritance.html) 設計模式，關於這個設計模式 Martin Fowler 介紹得很完整。

根據這個模式，我們會在表格裡面加上`type`欄位，用來決定該資料屬於哪一個類別（class）的物件。

以下我們實做一個簡單的繼承類別結構：

```markdown
Car
|- SportCar
|- HeavyCar
```

## 準備 {#get-ready}

這邊我們使用基本的 Yii 設置。資料庫建立好了之後，執行以下SQL指令來建立表格以及放入資料：

```sql
CREATE TABLE `car` (
    `id` int NOT NULL AUTO_INCREMENT,
    `name` varchar(255) NOT NULL,
    `type` varchar(255) DEFAULT NULL,
    PRIMARY KEY (`id`)
);

INSERT INTO car (id, NAME, TYPE) VALUES (1, 'Kamaz', 'heavy'), (2, 'Ferrari', 'sport'), (3, 'BMW', 'city');
```

然後使用 Gii 工具，生成`Car`模型。

## 實做物件繼承 {#how-to-do-it}

要在每次詢問資料庫時都會檢查車子的type，我們要自己實做一個簡單的 query 類別。

建立`models/CarQuery.php`：

```php
namespace app\models;

use yii\db\ActiveQuery;

class CarQuery extends ActiveQuery
{
    public $type;
    public $tableName;

    public function prepare($builder)
    {
        if ($this->type !== null) {
            $this->andWhere(["$this->tableName.type" => $this->type]);
        }
        return parent::prepare($builder);
    }
}
```

接著，我們建立不同種類車子的物件類別。

首先是`models/SportCar.php`：

```php
namespace app\models;

class SportCar extends Car
{
    const TYPE = 'sport';

    public function init()
    {
        $this->type = self::TYPE;
        parent::init();
    }

    public static function find()
    {
        return new CarQuery(get_called_class(), ['type' => self::TYPE, 'tableName' => self::tableName()]);
    }

    public function beforeSave($insert)
    {
        $this->type = self::TYPE;
        return parent::beforeSave($insert);
    }
}
```

`models/HeavyCar.php`的部份：

```php
namespace app\models;

class HeavyCar extends Car
{
    const TYPE = 'heavy';

    public function init()
    {
        $this->type = self::TYPE;
        parent::init();
    }

    public static function find()
    {
        return new CarQuery(get_called_class(), ['type' => self::TYPE, 'tableName' => self::tableName()]);
    }

    public function beforeSave($insert)
    {
        $this->type = self::TYPE;
        return parent::beforeSave($insert);
    }
}
```

然後我們要覆蓋（override） `Car` 模型裡面的 `instantiate()`函式：

```php
public static function instantiate($row)
{
    switch ($row['type']) {
        case SportCar::TYPE:
            return new SportCar();
        case HeavyCar::TYPE:
            return new HeavyCar();
        default:
           return new self;
    }
}
```

我們也要覆蓋`Car` 模型裡面的`tableName()`函式，這樣所有模型才會都使用同一個表格：

```php
public static function tableName()
{
    return '{{%car%}}';
}
```

這樣就結束了。

我們來嘗試看看，首先我們在`SiteController`建立`actionTest()`，實際運作看看：

```php
// finding all cars we have
$cars = Car::find()->all();
foreach ($cars as $car) {
    echo "$car->id $car->name " . get_class($car) . "<br />";
}

// finding any sport car
$sportCar = SportCar::find()->limit(1)->one();
echo "$sportCar->id $sportCar->name " . get_class($sportCar) . "<br />";
```

輸出會是：

```
1 Kamaz app\models\HeavyCar
2 Ferrari app\models\SportCar
3 BMW app\models\Car
2 Ferrari app\models\SportCar
```

可以看到，物件成功的根據`type`欄位建立，搜尋功能也如同我們所希望的一樣正常運作。

## 運作原理 {#how-it-works}

`SportCar`和`HeavyCar`模型很相近。兩個均繼承`Car`模型， 並覆蓋（override）兩個函式。

在`find()`函式裡面，我們實例化（instantiating）我們自己的 query 類別，讓我們在建立 SQL query之前，也就是`prepare()`的時候，儲存並處理車子的type 。`SportCar` 只會找跑車，而`HeavyCar`只會找重型車。

在`beforeSave()`函式裡面，我們確認存檔時`type`欄位會寫入正確的資料。這邊`TYPE`常數的引用是為了方便。

`Car`模型物件基本上就是我們用 Gii 產生的物件，不過我們修改了`instantiate()`函式。該函式呼叫的時間點，介於從資料庫取出資料，以及初始化類別屬性這兩個行為之間。另外，該函數接收的參數是資料庫回傳的資料，回傳值則是尚未初始化的物件。

這些正好是我們需要的特性。

我們實做的部份，是一個簡單的 switch 語句，檢查`type`欄位是否符合我們支援的物件，如果有的話則生成相對應的物件。如果沒有符合的值，則回傳預設的`Car`模型物件，就像範例裡面遇到「city」時的狀況一樣。

## 處理唯一（unique）值 {#handling-unique-values}

如果有欄位被標記為唯一（unique），為了避免`UniqueValidator`出錯，需要標記 `targetClass`。

```php
public function rules()
{
    return [
        [['MyUniqueColumnName'], 'unique', 'targetClass' => '\app\models\Car'],
    ];
}
```



