# 透過表單上傳檔案（Forms uploading files） {#uploading-files}

上傳檔案的方式在[教學裡面已經解釋了](http://www.yiiframework.com/doc-2.0/guide-input-file-upload.html)，不過還是需要再詳細說明一下，因為當我們使用 Active Record 作為表單的物件時，如果有上傳檔案，時常會將上傳檔案與檔案路徑這兩者混淆在一起。

## 目標 {#objective}

我們希望有個表單可以處理貼文。在表單裡面，我們可以上傳圖片，標題與文字。這邊的圖片不是必須的。並且當圖片已經存在所以沒有上傳時，不應該將圖片路徑設成 null。

## 準備 {#preparations}

我們建立資料庫表格如下：

```SQL
CREATE TABLE post
(
    id INT(11) PRIMARY KEY NOT NULL AUTO_INCREMENT,
    title VARCHAR(255) NOT NULL,
    text TEXT NOT NULL,
    image VARCHAR(255)
);
```

接著使用 Gii 工具建立`Post`模型以及在`PostController`裡面建立對應的CRUD。

現在，準備開始撰寫留言的程式了。

## 留言模型修正 {#post-model-adjustments}

Post 模型的`image`儲存上傳圖片的路徑。注意這邊不應該將檔案路徑與檔案本身搞混。所以我們用另一個變數進行區分。因為檔案本身不需要儲存在資料庫內，我們直接加入一個變數`$upload`：

```php
class Post extends \yii\db\ActiveRecord
{
    public $upload;
```

然後，修改驗證規則：

```php
/**
 * @inheritdoc
 */
public function rules()
{
    return [
        [['title', 'text'], 'required'],
        [['text'], 'string'],
        [['title'], 'string', 'max' => 255],
        [['upload'], 'file', 'extensions' => 'png, jpg'],
    ];
}
```

驗證規則裡面，因為與使用者輸入無關，我們將`$image`相關的驗證移除，並加入針對`$upload`的驗證。

## 表單 {#a-form}

`views/post/_form.php`的表單需要改變兩個部份。首先，我們移除`image`欄位。第二，我們加入上傳檔案`upload`的欄位：

```php
<?php $form = ActiveForm::begin(['options' => ['enctype' => 'multipart/form-data']]); ?>

    <?= $form->field($model, 'title')->textInput(['maxlength' => true]) ?>

    <?= $form->field($model, 'text')->textarea(['rows' => 6]) ?>

    <?= $form->field($model, 'upload')->fileInput() ?>

    <div class="form-group">
        <?= Html::submitButton($model->isNewRecord ? 'Create' : 'Update', ['class' => $model->isNewRecord ? 'btn btn-success' : 'btn btn-primary']) ?>
    </div>

<?php ActiveForm::end(); ?>
```

## 處理上傳 {#processing-upload}

處理上傳的部份，我們簡單的放進`PostController`內。兩個 action：`actionCreate()`和`actionUpdate()`。兩邊都同樣需要處理驗證跟儲存，所以我們將這段分離成`handlePostSave()`函式：

```php
public function actionCreate()
{
    $model = new Post();
    $this->handlePostSave($model);

    return $this->render('create', [
        'model' => $model,
    ]);
}

public function actionUpdate($id)
{
    $model = $this->findModel($id);

    $this->handlePostSave($model);

    return $this->render('update', [
        'model' => $model,
    ]);
}
```

然後我們實做`handlePostSave()`：

```php
protected function handlePostSave(Post $model)
{
    if ($model->load(Yii::$app->request->post())) {
        $model->upload = UploadedFile::getInstance($model, 'upload');
        //驗證
        if ($model->validate()) {
            //更新檔案
            if ($model->upload) {
                $filePath = 'uploads/' . $model->upload->baseName . '.' . $model->upload->extension;
                //上傳檔案
                if ($model->upload->saveAs($filePath)) {
                    $model->image = $filePath;
                }
            }
            //更新失敗
            if ($model->save(false)) {
                return $this->redirect(['view', 'id' => $model->id]);
            }
        }
    }
}
```

上面的程式碼內，一拿到從POST過來的資料，我們就將檔案實體放進`$upload`裡面。這邊的重點是，這件事情必須在驗證之前做。

驗證過後，如果檔案已經成功上傳了，我們就儲存檔案，並將檔案路徑寫進`$image`。 最後，我們呼叫`save()`的時候，使用`false`參數，代表「不驗證」。因為我們在前面已經驗證過了，不需要重複。

好了，我們的目標達成了。

## 網頁表單與 Active Record 的一點心得 {#a-note-on-forms-and-active-record}

為求簡化，Active Record 常常直接用在表單裡面。在許多狀況下，這種作法沒有問題。不過，有時候網頁表單的資料，與存進資料庫資料的結構並不相同。這種狀況下，會建議不要直接使用 Active Record，而是建立另外的表單模型。

資料儲存應該在模型裡面處理，而不是在控制器裡面。

