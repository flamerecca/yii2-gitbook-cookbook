# Custom validator for multiple attributes {#custom-validator-for-multiple-attributes}

Yii 2.0 教學裡面已經解釋了怎麼[設計自己的驗證](https://github.com/yiisoft/yii2/blob/master/docs/guide/input-validation.md#creating-validator-)，不過有些時候，你可能需要同時驗證多個互相關聯的參數。比方說，參數可能都很重要，難以抉擇哪個更加重要；或者規則比較複雜，導致參數之間會互相影響。

這邊，我們實做一個`CustomValidator`，來同時驗證多個參數。

## 怎麼做 {#how-to-do-it}

如果要檢查多個參數，預設會將所有的參數用相同的方式驗證。我們這邊使用不同的特性（trait），並覆蓋`yii\base\Validator:validateAttributes()`：

```php
<?php

namespace app\components;

trait BatchValidationTrait
{
    /**
     * @var bool whether to validate multiple attributes at once
     */
    public $batch = false;

    /**
     * Validates the specified object.
     * @param \yii\base\Model $model the data model being validated.
     * @param array|null $attributes the list of attributes to be validated.
     * Note that if an attribute is not associated with the validator, or is is prefixed with `!` char - it will be
     * ignored. If this parameter is null, every attribute listed in [[attributes]] will be validated.
     */
    public function validateAttributes($model, $attributes = null)
    {
        if (is_array($attributes)) {
            $newAttributes = [];
            foreach ($attributes as $attribute) {
                if (in_array($attribute, $this->attributes) || in_array('!' . $attribute, $this->attributes)) {
                    $newAttributes[] = $attribute;
                }
            }
            $attributes = $newAttributes;
        } else {
            $attributes = [];
            foreach ($this->attributes as $attribute) {
                $attributes[] = $attribute[0] === '!' ? substr($attribute, 1) : $attribute;
            }
        }

        foreach ($attributes as $attribute) {
            $skip = $this->skipOnError && $model->hasErrors($attribute)
                || $this->skipOnEmpty && $this->isEmpty($model->$attribute);
            if ($skip) {
                // Skip validation if at least one attribute is empty or already has error
                // (according skipOnError and skipOnEmpty options must be set to true
                return;
            }
        }

        if ($this->batch) {
            // Validate all attributes at once
            if ($this->when === null || call_user_func($this->when, $model, $attribute)) {
                // Pass array with all attributes instead of one attribute
                $this->validateAttribute($model, $attributes);
            }
        } else {
            // Validate each attribute separately using the same validation logic
            foreach ($attributes as $attribute) {
                if ($this->when === null || call_user_func($this->when, $model, $attribute)) {
                    $this->validateAttribute($model, $attribute);
                }
            }
        }
    }
}
```

然後，我們建立自己的驗證類別，並使用剛剛的特性：

```php
<?php

namespace app\components;

use yii\validators\Validator;

class CustomValidator extends Validator
{
    use BatchValidationTrait;
}
```

如果我們希望即時驗證（inline validation），我們可以用相同的特性，但是改繼承即時驗證器：

```php
<?php

namespace app\components;

use yii\validators\InlineValidator;

class CustomInlineValidator extends InlineValidator
{
    use BatchValidationTrait;
}
```

之後，還有幾個地方要修改。

首先，換掉原本的`InlineValidator`，使用自製的`CustomInlineValidator`，我們需要覆蓋`CustomValidator`的\[\[\yii\validators\Validator::createValidator\(\)\]\] 函式：

```php
public static function createValidator($type, $model, $attributes, $params = [])
{
    $params['attributes'] = $attributes;

    if ($type instanceof \Closure || $model->hasMethod($type)) {
        // method-based validator
        // The following line is changed to use our CustomInlineValidator
        $params['class'] = __NAMESPACE__ . '\CustomInlineValidator';
        $params['method'] = $type;
    } else {
        if (isset(static::$builtInValidators[$type])) {
            $type = static::$builtInValidators[$type];
        }
        if (is_array($type)) {
            $params = array_merge($type, $params);
        } else {
            $params['class'] = $type;
        }
    }

    return Yii::createObject($params);
}
```

最後，要在模型中支援我們的驗證，我們建立自製的特性，並覆蓋 \[\[\yii\base\Model::createValidators\(\)\]\] 如下：

```php
<?php

namespace app\components;

use yii\base\InvalidConfigException;

trait CustomValidationTrait
{
    /**
     * Creates validator objects based on the validation rules specified in [[rules()]].
     * Unlike [[getValidators()]], each time this method is called, a new list of validators will be returned.
     * @return ArrayObject validators
     * @throws InvalidConfigException if any validation rule configuration is invalid
     */
    public function createValidators()
    {
        $validators = new ArrayObject;
        foreach ($this->rules() as $rule) {
            if ($rule instanceof Validator) {
                $validators->append($rule);
            } elseif (is_array($rule) && isset($rule[0], $rule[1])) { // attributes, validator type
                // The following line is changed in order to use our CustomValidator
                $validator = CustomValidator::createValidator($rule[1], $this, (array) $rule[0], array_slice($rule, 2));
                $validators->append($validator);
            } else {
                throw new InvalidConfigException('Invalid validation rule: a rule must specify both attribute names and validator type.');
            }
        }
        return $validators;
    }
}
```

現在，我們可以透過繼承`CustomValidator`，實做我們自己的驗證器`ChildrenFundsValidator`：

```php
<?php

namespace app\validators;

use app\components\CustomValidator;

class ChildrenFundsValidator extends CustomValidator
{
    public function validateAttribute($model, $attribute)
    {
        // 這邊 $attribute 不是單一參數，而是包含所有相關參數的陣列
        $totalSalary = $this->personalSalary + $this->spouseSalary;
        // 如果有配偶薪資，成人最小需要量加倍
        $minAdultFunds = $this->spouseSalary ? self::MIN_ADULT_FUNDS * 2 : self::MIN_ADULT_FUNDS;
        $childFunds = $totalSalary - $minAdultFunds;
        if ($childFunds / $this->childrenCount < self::MIN_CHILD_FUNDS) {
            $this->addError('*', '這個家庭的薪資不足以育兒');
        }
    }
}
```

因為`$attribute`包含 the list of all related attributes，we can use loop in case of adding errors for all attributes is needed：

```php
foreach ($attribute as $singleAttribute) {
    $this->addError($attribute, '這個家庭的薪資不足以育兒');
}
```

現在，我們可以在模型的驗證規則裡面，加上這個規則：

```php
[
    ['personalSalary', 'spouseSalary', 'childrenCount'],
    \app\validators\ChildrenFundsValidator::className(),
    'batch' => `true`,
    'when' => function ($model) {
        return $model->childrenCount > 0;
    }
],
```

即時驗證的程式碼則是：

```php
[
    ['personalSalary', 'spouseSalary', 'childrenCount'],
    'validateChildrenFunds',
    'batch' => `true`,
    'when' => function ($model) {
        return $model->childrenCount > 0;
    }
],
```

And here is according validation method:

```php
public function validateChildrenFunds($attribute, $params)
{
    // 這邊 $attribute 不是單一參數，而是包含所有相關參數的陣列
    $totalSalary = $this->personalSalary + $this->spouseSalary;
    // 如果有配偶薪資，成人最小需要量加倍
    $minAdultFunds = $this->spouseSalary ? self::MIN_ADULT_FUNDS * 2 : self::MIN_ADULT_FUNDS;
    $childFunds = $totalSalary - $minAdultFunds;
    if ($childFunds / $this->childrenCount < self::MIN_CHILD_FUNDS) {
        $this->addError('childrenCount', '這個家庭的薪資不足以育兒');
    }
}
```

## 總結 {#summary}

這樣做的好處：

* 程式碼It better reflects all attributes that participate in validation，規則可讀性更高。
* 這種作法讓選項 \[\[yii\validators\Validator::skipOnError\]\] 和 \[\[yii\validators\Validator::skipOnEmpty\]\] 適用於**每一個**使用的參數，不僅僅是某一個我們認為比較重要的參數。

如果實做驗證的部份有問題，我們可以：

* 結合 \[\[yii\widgets\ActiveForm::enableAjaxValidation\|enableClientValidation\]\] 和\[\[yii\widgets\ActiveForm::enableAjaxValidation\|enableAjaxValidation\]\] 選項，讓多個參數可以同時透過AJAX驗證，不需要重新載入頁面。
* 因為 \[\[yii\validators\Validator::clientValidateAttribute\]\] 本來就是設計給單一參數驗證的，所以從頭實做自己的驗證方式。



