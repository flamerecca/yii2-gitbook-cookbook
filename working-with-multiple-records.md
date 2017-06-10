# 多筆紀錄的處理（Working With Multiple Records） {#working-with-multiple-records}

有時候，使用者一個動作會同時建立或者影響不同表格的紀錄。這一類操作最好要遵守 [ACID](https://en.wikipedia.org/wiki/ACID) （原子性、一致性性、隔離性、持久性）的特性。Yii2 supports transactions, isolation levels, and allow you to validate data independently.

比方說，要開始一場交易，你可能同時需要儲存交易參考資料與檔案For example when creating a credit, you also need to store the credit references and files associated to the credit.

```php
public function actionCreate()
{
    $model = new Credit();
    $modelReferences = [new CreditRefence()];
    $modelFiles = [new CreditFile()];

    if (Yii::$app->request->isPost) {
        $model->load(Yii::$app->request->post());

        foreach (Yii::$app->requestPost($modelReferences[0]->formName())
            as $i => $data
        ) {
            $newModel = new CreditReference();
            $newModel->load($data, '');
            $modelReferences[$i] = $newModel;
        }

        foreach (Yii::$app->requestPost($modelFiles[0]->formName())
            as $i => $data
        ) {
            $newModel = new CreditFile();
            $newModel->load($data, '');
            $newModel->file = UploadedFile::getInstance($newModel, "[$i]file");
            $modelFiles[$i] = $newModel;
        }

        $transaction = Yii::$app->db->beginTransaction(
            Transaction::SERIALIZABLE
        );

        try {
            $valid = $model->validate();
            $valid = Model::validateMultiple($modelReferences, [
                    'name',
                    'last_name',
                    // other fields, make sure to NOT include credit_id since
                    // the record has not been created yet.
                ]) && $valid;
            $valid =  Model::validateMultiple($modelFiles, [
                    'file',
                    // other fields, make sure to NOT include credit_id since
                    // the record has not been created yet.
                ]) && $valid;

            if ($valid) {
                // 模型已經驗證過，所以這邊不再驗證
                $model->save(false);

                foreach ($modelReferences as $newModel) {
                    $newModel->credit_id = $model->id;
                    $newModel->save(false);
                }

                foreach ($modelFiles as $newModel) {
                    $newModel->credit_id = $model->id;
                    $newModel->file->saveAs(Yii::getAlias(
                        '@webroot/uploads/' . $model->file->name
                    ));
                    $newModel->save(false);
                }
                
                $transaction->commit();
                return $this->redirect(['credit/view', ['id' => $model->id]]);
            } else {
                //驗證失敗，交易取消
                $transaction->rollBack();
            }
        } catch (Exception $e) {
            //出現例外，交易取消
            $transaction->rollBack();
            throw new BadRequestHttpException($e->getMessage(), 0, $e);
        }
    }

    return $this->render('create', [
        'model' => $model,
        'modelReferences' => $modelReferences,
        'modelFiles' => $modelFiles,

    ]);
}
```

以下說明各個步驟：

1. 檢查輸入是否合法  
   In this case we are assuming the controller already checked the user credentials using filter and at the action its enough to check if the petition is using the`post`method.

2. 建立模型、輸入使用者資料  
   雖然只有一筆交易，但是可能影響多個檔案與相關資料

3. 開始交易  
   Its important to start the transaction at this point since some validations like`unique`and`exist`might be necessary so we start the transaction here to avoid \[Reading Phenomena\] \([https://en.wikipedia.org/wiki/Isolation\_\(database\_systems\)\#Read\_phenomena](https://en.wikipedia.org/wiki/Isolation_%28database_systems%29#Read_phenomena%29%29.  
   You should also notice that we created the transaction using`yii\db\Transaction::SERIALIZABLE`which is the highest [isolation level] %28[https://en.wikipedia.org/wiki/Isolation_%28database_systems%29#Isolation_levels]%28[https://en.wikipedia.org/wiki/Isolation_%28database_systems%29#Isolation_levels%29]%28[https://en.wikipedia.org/wiki/Isolation_%28database_systems%29#Isolation_levels%29%29]%28[https://en.wikipedia.org/wiki/Isolation_%28database_systems%29#Isolation_levels%29%29%29]%28[https://en.wikipedia.org/wiki/Isolation_%28database_systems%29#Isolation_levels%29%29%29%29%29%28[https://en.wikipedia.org/wiki/Isolation_%28database_systems%29#Isolation_levels%29%29%29%29%29]%28[https://en.wikipedia.org/wiki/Isolation_%28database_systems%29#Isolation_levels%29%29%29%29%29\)\]\([https://en.wikipedia.org/wiki/Isolation\_\(database\_systems\)\#Isolation\_levels\)\)\)\)\)\)\)\](https://en.wikipedia.org/wiki/Isolation_%28database_systems%29#Isolation_levels%29%29%29%29%29%29%29\)\).

4. 驗證模型  
   Its important to validate them all to show the user all the validation errors if necessary. Using an`if`statement like this  
   `if ($model->validate() && Model::validateMultiple(...))`  
   Is simpler to understand but if the first validation fails the second one won't be executed.  
   Also notice that we are not going to validate`credit_id`on the files and references since the credit has not been created yet.

   1. 如果驗證失敗， end the transaction with a`rollBack()`just in case any validation had updated anything  
      The action will then render the view and if you are using something like`ActiveForm`the user will see all the validation errors.

5. 如果交易驗證成功，我們就儲存所有相關的模型（model）  
   We will save them without validation since they were already validated and assign the`credit_id`to the files and references after the credit has been saved.

   1. Catch any exception from the validation or saving and execute `rollBack()`  
      For debugging purposes we throw a new exception with the previous one so it can get caught by the Yii2 exception manager.

6. 如果沒有任何例外，就`commit()`改變的部份  
   之後就執行交易成功後的行為，比方說導向其他頁面，或者更新頁面資訊。

## Operations Triggered by Events {#operations-triggered-by-events}

假設你有兩個模型：`Credit` 模型，包含`id`和`amount`兩個參數；還有一個`CreditPayment`模型，包含 `credit_id`和`amount`參數。 當你建立`CreditPayment`時，希望能同時更新`Credit`的amount：

```php
class CreditPayment extends ActiveRecord
{
    public function afterSave($insert, $changedAttributes)
    {
        parent::afterSave($insert, $changedAttributes);
        if ($insert === false) {
            return; // only work with newly created payments
        }

        $this->credit->amount = $this->credit->amount - $this->ammount;
        if ($this->credit->amount <= 0) {
            $this->credit->status = Credit::STATUS_PAYMENT_FINISHED;
        }

        if ($this->credit->save(false) === false) {
            throw new Exception("credit couldn't be update");
        }
    }

    public function getCredit()
    {
        return $this->hasOne(Credit::className(), ['id' => 'credit_id']);
    }
}
```

如果出現任何例外（exception），payment 本身會被儲存，但是不影響credit的amount。

We can encapsulate all the operation on a transaction very simply using the method`yii\db\ActiveRecord::transactions()`

```php
public function transactions()
{
    return [
        self::SCENARIO_DEFAULT => self::OP_INSERT,
    ];
}
```



