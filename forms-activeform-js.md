# 透過 JavaScript 操作 ActiveForm（Working with ActiveForm via JavaScript） {#working-with-activeform-via-javascript}

一般來說，[在 Yii 2.0 官方教學說明很詳盡](http://www.yiiframework.com/doc-2.0/guide-input-forms.html)的，在PHP端處理ActiveForm的方式，已經能滿足絕大多數的網頁需求了 。

不過，如果要再更進一步，比方說動態的增減欄位，或者在某些狀況下，觸發特定欄位的驗證檢查。這種狀況就有點棘手了。

這邊我們介紹 ActiveForm 的 JavaScript API，以處理上述這類需求。

## 準備 {#preparations}

這裡我們使用基本 Yii 來示範，所以需要[先安裝 Yii](http://www.yiiframework.com/doc-2.0/guide-start-installation.html)。

## 對單一欄位觸發驗證 {#triggering-validation-for-individual-form-fields}

```js
$('#contact-form').yiiActiveForm('validateAttribute', 'contactform-name');
```

## 對整個表單觸發驗證 {#trigger-validation-for-the-whole-form}

```js
$('#contact-form').yiiActiveForm('validate', true);
```

第二個欄位設置為`true`，會使整張表單都需要驗證。

## 使用事件 {#using-events}

```js
$('#contact-form').on('beforeSubmit', function (e) {
    if (!confirm("Everything is correct. Submit?")) {
        return false;
    }
    return true;
});
```

可用的事件有：

* [`beforeValidate`](https://github.com/yiisoft/yii2/blob/master/framework/assets/yii.activeForm.js#L39)
* [`afterValidate`](https://github.com/yiisoft/yii2/blob/master/framework/assets/yii.activeForm.js#L50)
* [`beforeValidateAttribute`](https://github.com/yiisoft/yii2/blob/master/framework/assets/yii.activeForm.js#L64)
* [`afterValidateAttribute`](https://github.com/yiisoft/yii2/blob/master/framework/assets/yii.activeForm.js#L74)
* [`beforeSubmit`](https://github.com/yiisoft/yii2/blob/master/framework/assets/yii.activeForm.js#L83)
* [`ajaxBeforeSend`](https://github.com/yiisoft/yii2/blob/master/framework/assets/yii.activeForm.js#L93)
* [`ajaxComplete`](https://github.com/yiisoft/yii2/blob/master/framework/assets/yii.activeForm.js#L103)
  .

## 動態增減欄位 {#adding-and-removing-fields-dynamically}

在驗證清單裡面增加欄位：

```js
$('#contact-form').yiiActiveForm('add', {
    id: 'address',
    name: 'address',
    container: '.field-address',
    input: '#address',
    error: '.help-block',
    validate:  function (attribute, value, messages, deferred, $form) {
        yii.validation.required(value, messages, {message: "Validation Message Here"});
    }
});
```

在驗證清單裡面移除欄位：

```js
$('#contact-form').yiiActiveForm('remove', 'address');
```

## 更新單一參數的錯誤資訊 {#updating-error-of-a-single-attribute}

對單一參數新增錯誤資訊：

```js
$('#contact-form').yiiActiveForm('updateAttribute', 'contactform-subject', ["I have an error..."]);
```

移除該錯誤資訊：

```js
$('#contact-form').yiiActiveForm('updateAttribute', 'contactform-subject', '');
```

## 更新錯誤資訊或摘要 {#update-error-messages-and-optionally-summary}

```js
$('#contact-form').yiiActiveForm('updateMessages', {
    'contactform-subject': ['Really?'],
    'contactform-email': ['I don\'t like it!']
}, true);
```

最後一個參數指示出我們是否要更新摘要。

## 監控參數改變 {#listening-for-attribute-changes}

監控參數改變的事件，比方說選項按鈕被點選……等，可以使用以下程式碼：

```js
$("#attribute-id").on('change.yii',function(){
        //your code here
});
```

## 取得參數的值 {#getting-attribute-value}

要與第三方元件相容，取得參數的值最好方法是：

```js
$('#form_id').yiiActiveForm('find', '#attribute').value
```

## 自定義驗證 {#custom-validation}

根據狀況，如果我們需要透過 JavaScript 修改參數的驗證，可以使用規則的`whenClient`參數。

不過，假設我們需要純客戶端的驗證方式，可以用：

```js
$('#form_id').on('beforeValidate', function (e) {
            $('#form_id').yiiActiveForm('find', '#attribute').validate = function (attribute, value, messages, deferred, $form) {
                //自定義驗證程式碼
            }
        return true;
    });
```



