# 透過表單上傳檔案（Forms uploading files） {#uploading-files}

上傳檔案的方式在[教學裡面已經解釋了](http://www.yiiframework.com/doc-2.0/guide-input-file-upload.html)，不過but it won't hurt to elaborate it a bit more because often, when Active Record model is reused as a form it causes confusion when a field storing file path is reused for file upload.

## 目標 {#objective}

We'll have a posts manager with a form. 在表單裡面，我們可以上傳一張圖片，附上標題與文字。Image is not mandatory. Existing image path should not be set to null when saving without image uploaded.

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

Next, let's generate`Post`model with Gii and a CRUD in`PostController`.

現在，準備開始撰寫留言的程式了。

## 留言模型修正 {#post-model-adjustments}

Post model's`image`stores a path to the image uploaded it should not be confused with the actual file uploaded so we'll need a separate field for that purpose. Since the file isn't saved to database we don't need to store it. Let's just add a public field called`upload`:

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

In the above we removed everything concerning`image`because it's not user input and added file validation for`upload`.

## 表單 {#a-form}

A form in the`views/post/_form.php`needs two things. First, we should remove a field for`image`. Second, we should add a file upload field for`upload`:

```
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

The handling, for simplicity sake, is done in`PostController`. Two actions:`actionCreate`and`actionUpdate`. Both are repetitive so the first step is to extract handling into separate method:

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

        if ($model->validate()) {
            if ($model->upload) {
                $filePath = 'uploads/' . $model->upload->baseName . '.' . $model->upload->extension;
                if ($model->upload->saveAs($filePath)) {
                    $model->image = $filePath;
                }
            }

            if ($model->save(false)) {
                return $this->redirect(['view', 'id' => $model->id]);
            }
        }
    }
}
```

In the code above right after filling model from POST we're also filling its`upload`field with an instance of the file uploaded. The important thing is that it should be done before validation. After validating a form, if there's file uploaded we're saving it and writing path into model's`image`field. Last, regular`save()`is called with a`false`argument which means "don't validate". It's done this way because we've just validated above and sure it's OK.

That's it. Objective achieved.

## A note on forms and Active Record {#a-note-on-forms-and-active-record}

For the sake of simplicity and laziness, Active Record is often reused for forms directly. There are scenarious making it doable and usually it doesn't cause any problems. However, there are situations when what's in the form actually differs from what is saved into database. In this case it is preferrable to create a separate form model which is not Active Record. Data saving should be done in this model instead of controller directly.

