# RBAC {#rbac}

以角色為基礎的存取控制（Role Based Access Control，RBAC）是 Yii 裡面內建的一個權限管理系統。 雖然[官方教學已經有說明](http://www.yiiframework.com/doc-2.0/guide-security-authorization.html#rbac) ，不過目前沒有提到如何使用的範例，這邊我們進行補充。

我們使用文章發布系統，像是 [YiiFeed](http://yiifeed.com/) 這樣的需求，作為此次的範例。

## 設置 RBAC 元件 {#configuring-rbac-component}

認證管理員元件的初始設置與[所有元件設置方式](http://www.yiiframework.com/doc-2.0/guide-structure-application-components.html)一樣：在程式 config 的`components`區塊，我們加上`authManager`部份，標記使用的類別以及元件的選項。驗證管理員有兩種後端可以選擇：PHP 檔案與資料庫。兩種方式使用的 API 相同，所以使用上差異不大，唯一的差別是 RBAC 資料儲存的方式。

### PHP 後端 {#php-backend}

要設置 PHP 後端的部份，在 config 加上：

```php
return [
    // ...
    'components' => [
        // ...
        'authManager' => [
            'class' => 'yii\rbac\PhpManager',
        ],
    ],
    // ...
];
```

> 備註：如果我們使用`yii2-basic-app`，除了宣告在`config/console.php`以外，`authManager`必須要在`config/web.php`也宣告一次。如果是使用`yii2-advanced-app`的話，`authManager`只需要在`common/config/main.php`宣告一次。

PHP 檔案後端預設將 RBAC 資料存放於`@app/rbac`資料夾底下。所以`rbac`資料夾應該建立於應用資料夾裡面，並允許伺服器對該資料夾有讀寫的權限。

### 資料庫後端 {#database-backend}

設置資料庫後端比較複雜。首先，在 config 檔加入以下內容：

```php
return [
    // ...
    'components' => [
        'authManager' => [
            'class' => 'yii\rbac\DbManager',
        ],
        // ...
    ],
];
```

> 備註：如果我們使用`yii2-basic-app`，除了宣告在`config/console.php`以外，`authManager`必須要在`config/web.php`也宣告一次。如果是使用`yii2-advanced-app`的話，`authManager`只需要在`common/config/main.php`宣告一次。

再次確認網頁以及終端都已經設置好資料庫，然後打開終端，並運行以下指令，讓資料庫建立 RBAC 資料所需要的表單：

```
yii migrate --migrationPath=@yii/rbac/migrations
```

## 設計角色與權限架構 {#planning-roles-and-permissions-hierarchy}

這裡我們使用文章發布系統作為範例。對應這個範例，需要設定三種使用者：

* 一般使用者：可以閱讀、推薦、發布文章。也可以編輯或者刪除自己發布的文章。
* 板主：可以編輯、刪除、認可、拒絕所有的文章。板主也可以查看事件列表。
* 管理員：擁有板主的所有權限，加上可以看使用者名單，以及編輯使用者資料

這邊我們建議，可以拿出紙筆，或者使用 [yEd](https://www.yworks.com/products/yed) 之類的軟體來整理架構圖。

想成功使用 RBAC 的首要原則，是讓架構越簡單越好。我們的範例裡面，一般使用者的權限可視為預設值，並不需要特別設置角色。編輯、刪除、認可、拒絕文章可簡化為「管理文章」。看使用者名單、編輯使用者資料則可簡化為「管理使用者」這個權限。於是我們的架構可以設置如下：

![](/assets/Untitled Diagram.png)

> 備註：本圖使用 [draw.io](https://www.draw.io/) 製作

## 實做架構 {#filling-hierarchy}

如果你的專案有使用資料庫，並且你對 [migration](http://www.yiiframework.com/doc-2.0/guide-db-migrations.html) 很熟悉的話，使用 migration 來建立架構會比較好。

打開命令列，輸入

```
./yii migrate/create rbac_init
```

這會建立一個新的 migration 類別，使用`up()`函式來建立架構，使用`down()`函式來刪除架構。

```php
use yii\db\Migration;

class m141204_121823_rbac_init extends Migration
{
    public function up()
    {
        $auth = Yii::$app->authManager;

        $manageArticles = $auth->createPermission('manageArticles');
        $manageArticles->description = 'Manage articles';
        $auth->add($manageArticles);

        $manageUsers = $auth->createPermission('manageUsers');
        $manageUsers->description = 'Manage users';
        $auth->add($manageUsers);

        $moderator = $auth->createRole('moderator');
        $moderator->description = 'Moderator';
        $auth->add($moderator);
        $auth->addChild($moderator, $manageArticles);

        $admin = $auth->createRole('admin');
        $admin->description = 'Administrator';
        $auth->add($admin);
        $auth->addChild($admin, $moderator);
        $auth->addChild($admin, $manageUsers);
    }

    public function down()
    {
        Yii::$app->authManager->removeAll();
    }
}
```

上面的`createPermission()`和`createRole()`可以用來建立層級物件，但是沒有儲存。要儲存物件時，應該要呼叫`add()`。`addChild()`則是在連接子物件至父物件時呼叫。呼叫這些函式之後，應該要馬上儲存連接。

> 備註：不管你使用哪種後端：PHP檔案或者資料庫，驗證管理員使用的是相同的函式。所以，建立架構的函式是一模一樣的。

如果我們的網頁沒有使用資料庫，或者因為某些原因不使用 migration，我們也可以使用 console command。如果使用 basic template 的話，應該修改 `commands\RbacController.php`如下：

```php
<?php
namespace app\commands;

use yii\console\Controller;

class RbacController extends Controller
{
    public function actionInit()
    {
        if (!$this->confirm("Are you sure? It will re-create permissions tree.")) {
            return self::EXIT_CODE_NORMAL;
        }

        $auth = Yii::$app->authManager;
        $auth->removeAll();

        $manageArticles = $auth->createPermission('manageArticles');
        $manageArticles->description = 'Manage articles';
        $auth->add($manageArticles);

        $manageUsers = $auth->createPermission('manageUsers');
        $manageUsers->description = 'Manage users';
        $auth->add($manageUsers);

        $moderator = $auth->createRole('moderator');
        $moderator->description = 'Moderator';
        $auth->add($moderator);
        $auth->addChild($moderator, $manageArticles);

        $admin = $auth->createRole('admin');
        $admin->description = 'Administrator';
        $auth->add($admin);
        $auth->addChild($admin, $moderator);
        $auth->addChild($admin, $manageUsers);
    }
}
```

上面的指令可以使用`./yii rbac/init`進行呼叫。

## 分配使用者角色 {#assigning-role-to-user}

預設使用者沒有任何角色，所以不需要考慮怎麼分配。角色管理可以在 admin panel 或者 console 裡面實做。因為我們的管理員都很帥，我們建立 `commands\RbacController.php`如下：

```php
<?php
namespace app\commands;

use yii\console\Controller;

class RbacController extends Controller
{
    public function actionAssign($role, $username)
    {
        $user = User::find()->where(['username' => $username])->one();
        if (!$user) {
            throw new InvalidParamException("There is no user \"$username\".");
        }

        $auth = Yii::$app->authManager;
        $roleObject = $auth->getRole($role);
        if (!$roleObject) {
            throw new InvalidParamException("There is no role \"$role\".");
        }

        $auth->assign($roleObject, $user->id);
    }
}
```

上面的程式碼內，we're finding a user by username specified. Then getting role object by its name and assigning role to a user by ID. Again, it doesn't matter if PHP backend or database backend is used. It would look exactly the same.

Also it would be exactly the same assignment in case of implementing admin UI or in case when you need role right away and assigning it right after user is successfully singed up.

Sign up three new users and assign two of them`admin`and`moderator`roles respectively:

```
./yii rbac/assign admin qiang
./yii rbac/assign moderator alex
```

## 檢查權限 {#checking-access}

現在 RBAC 已經上線，而且我們有三種使用者：一般使用者，板主，管理員。現在我們要使用這些角色了。

### 權限過濾 {#access-filter}

基本的權限過濾可以透過 access control filter 實做，[官方教學有詳細說明](http://www.yiiframework.com/doc-2.0/guide-security-authorization.html#access-control-filter)：

```php
namespace app\controllers;

use yii\web\Controller;
use yii\filters\AccessControl;

class ArticleController extends Controller
{
    public function behaviors()
    {
        return [
            'access' => [
                'class' => AccessControl::className(),
                'only' => ['suggest', 'queue', 'delete', 'update'], //only be applied to
                'rules' => [
                    [
                        'allow' => true,
                        'actions' => ['suggest', 'update'],
                        'roles' => ['@'],
                    ],
                    [
                        'allow' => true,
                        'actions' => ['queue', 'delete'],
                        'roles' => ['manageArticles'],
                    ],
                ],
            ],
            'verbs' => [
                'class' => VerbFilter::className(),
                'actions' => [
                    'delete' => ['post'],
                ],
            ],
        ];
    }

    // ...
```

We're allowing any authenticated user to suggest articles. Same applied to editing articles \(it's explained in the next section\). Viewing moderation queue and deleting articles are available only to roles which have`manageArticles`permission. In our case it's both`admin`and`moderator`since`admin`inherits all`moderator`permissions.

Same simple checks via access control filter could be applied to`UserController`which handles admin actions regarding users.

### 手動檢查 {#doing-manual-checks}

有些情況下，手動檢查是必要的。 在我們的例子裡面，像是檢查使用者是否允許編輯文章的需求，就需要手動檢查。這需求無法透過一般的權限檢查滿足，因為除了板主可以編輯以外，還有文章發布者可以編輯該文章：

```php
namespace app\controllers;

use app\models\Article;
use yii\web\Controller;
use yii\filters\AccessControl;

class ArticleController extends Controller
{
    // ...
    public function actionUpdate($id)
    {
        $model = $this->findModel($id);
        if (Yii::$app->user->id == $model->user_id || \Yii::$app->user->can('manageArticles')) {
            // ...
        } else {
            throw new ForbiddenHttpException('您不被允許編輯此文章');
        }
    }
}
```

上面的程式碼裡面，我們分別檢查了使用者的權限是否可以編輯文章，以及使用者是否為文章作者。如果滿足其中一個條件，我們就允許編輯文章；反之則丟出`ForbiddenHttpException`。

