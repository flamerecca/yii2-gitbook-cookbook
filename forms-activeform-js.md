# 透過 JavaScript 操作 ActiveForm {#working-with-activeform-via-javascript}

PHP side of ActiveForm, which is usually more than enough for majority of projects[ 在 Yii 2.0 官方教學說明很詳盡](http://www.yiiframework.com/doc-2.0/guide-input-forms.html)的PHP端如何處理ActiveForm，通常 。

It is getting a bit more tricky when it comes to advanced things such as adding or removing form fields dynamically or triggering individual field validation using unusual conditions.

這邊我們介紹 ActiveForm JavaScript API。

## 準備 {#preparations}

這裡我們使用基本Yii來示範，所以需要[先安裝Yii](http://www.yiiframework.com/doc-2.0/guide-start-installation.html)。

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

## Update error messages and, optionally, summary {#update-error-messages-and-optionally-summary}

```js
$('#contact-form').yiiActiveForm('updateMessages', {
    'contactform-subject': ['Really?'],
    'contactform-email': ['I don\'t like it!']
}, true);
```

The last argument in the above code indicates if we need to update summary.

## 監控參數改變 {#listening-for-attribute-changes}

監控參數改變的事件，比方說選項按鈕被點選……等，可以使用以下程式碼：

```js
$("#attribute-id").on('change.yii',function(){
        //your code here
});
```

## 取得參數的值 {#getting-attribute-value}

In order to be compatible with third party widgets like \(Kartik\), the best option to retrieve the actual value of an attribute is:

```js
$('#form_id').yiiActiveForm('find', '#attribute').value
```

## 自定義驗證 {#custom-validation}

In case you want to change the validation of an attribute in JS based on a new condition, you can do it with the rule property whenClient, but in the case you need a validation that doesn't depends on rules \(only client side\), you can try this:

```js
$('#form_id').on('beforeValidate', function (e) {
            $('#form_id').yiiActiveForm('find', '#attribute').validate = function (attribute, value, messages, deferred, $form) {
                //自定義驗證程式碼
            }
        return true;
    });
```



