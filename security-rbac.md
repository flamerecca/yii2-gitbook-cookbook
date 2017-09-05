# RBAC {#rbac}

以角色為基礎的存取控制（Role Based Access Control，RBAC）是 Yii 裡面內建的一個權限管理系統。 雖然[官方教學已經有說明](http://www.yiiframework.com/doc-2.0/guide-security-authorization.html#rbac) ，不過目前沒有提到如何使用的範例，這邊我們進行補充。

我們使用文章發布系統，像是 [YiiFeed](http://yiifeed.com/) 這樣的需求，作為此次的範例。

## 設置 RBAC 元件 {#configuring-rbac-component}

初始設置 of authentication manager component follows the same pattern 與[所有元件設置方式](http://www.yiiframework.com/doc-2.0/guide-structure-application-components.html)一樣：在程式 config 的`components`區塊，我們加上`authManager`部份，標記使用的類別以及元件的選項。驗證管理員有兩種後端可以選擇：PHP 檔案與資料庫。兩種方式使用的 API 相同，所以使用上差異不大，唯一的差別是 RBAC 資料儲存的方式。

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

PHP 檔案後端預設將 RBAC 資料存放於`@app/rbac`資料夾底下。所以`rbac`資料夾應該建立於應用資料夾裡面，並允許伺服器對該資料u夾有讀寫的權限。

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

> 備註：如果我們使用`yii2-basic-app`，there is a`config/console.php`configuration file where the`authManager`needs to be declared additionally to`config/web.php`. 如果使用的是`yii2-advanced-app`，he`authManager`should be declared only once in`common/config/main.php`.

Make sure you have database configured for both web and console applications then open console and run migration that would create all the tables necessary to store RBAC data:

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

> 備註：本圖使用[draw.io](https://www.draw.io/)製作

## 實做架構 {#filling-hierarchy}

如果你的專案有使用資料庫，and you're already familiar with [migrations](http://www.yiiframework.com/doc-2.0/guide-db-migrations.html)，使用 migration 來建立架構會比較好。

打開命令列，輸入

```
./yii migrate/create rbac_init
```

That would create new migration class with`up()`method in which we'd build the hierarchy and`down()`in which we'll destroy it.

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

上面的`createPermission()`和`createRole()`are creating new hierarchy objects but not yet saving them. In order to save them`add()`should be called.`addChild()`method is used to connect child object to their parents. When called this method saves connections immediately.

> 備註：It doesn't matter which backend you're using: PHP files or database. Authentication manager exposes exactly the same methods ，so hierarchy is built using exactly the same code.

如果我們的網頁沒有使用資料庫，或者因為某些原因不使用 migrations，you can do the same in a console command. For basic project template that would be`commands\RbacController.php`:

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

Since our default user doesn't have any role ，we don't need to worry about assigning it. User role management could be implemented either in admin panel or in console. Since our admins are cool guys, we'll create console contoller`commands\RbacController.php`:

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

In the code above we're finding a user by username specified. Then getting role object by its name and assigning role to a user by ID. Again, it doesn't matter if PHP backend or database backend is used. It would look exactly the same.

Also it would be exactly the same assignment in case of implementing admin UI or in case when you need role right away and assigning it right after user is successfully singed up.

Sign up three new users and assign two of them`admin`and`moderator`roles respectively:

```
./yii rbac/assign admin qiang
./yii rbac/assign moderator alex
```

## 檢查權限 {#checking-access}

Now we have RBAC in place and three users: regular user, moderator and admin. Let's start using what we've created.

### 權限過濾 {#access-filter}

The very basic access checks could be done via access control filter which is[covered well in the official guide](http://www.yiiframework.com/doc-2.0/guide-security-authorization.html#access-control-filter):

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

有些情況下，手動檢查是必要的。 在我們的例子裡面，像是檢查使用者是否允許編輯文章的需求，就需要手動檢查。這需求無法透過一般的權限檢查滿足，We can't do it via access control filter because we need to allow editing for regular users owning an article and moderators at the same time:

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
            throw new ForbiddenHttpException('You are not allowed to edit this article.');
        }
    }
}
```

In the code above we're checking if current user is either article owner or is allowed to manage articles. If either one is true, we're proceeding normally. Otherwise denying access.

