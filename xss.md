# 跨網站指令碼（XSS） {#xss}

跨網站指令碼（Cross-site scripting，XSS），是因為輸出時檢查不充分，導致網站的安全漏洞。這類漏洞允許攻擊者在網頁內注入JavaScript程式。舉例來說，如果你的網站允許使用者留評論，攻擊者可能會留下：

```js
<script>
alert('Hello from hacker ;)');
</script>
```

的評論。

如果網站沒有做好過濾，每個看到評論頁面的使用者，都會看到一個「Hello from hacker」的跳出視窗，表示這段JavaScript 程式已經被執行。而能執行Javascript程式碼，代表攻擊者幾乎可以做任何事情（比方說收集使用者的cookie）。

這邊我們模擬可能的攻擊。在一個controller裡面，我們取得資料並傳到視圖（view）`index.php`裡面

```php
public function actionIndex()
{
    $data = '<script>alert("攻擊範例")</script>';
        return $this->render('index', [
        'data' => $data,
    ]);
}
```

然後，在`index.php`裡面，我們不對`$data`做任何處理：

```php
echo $data;
```

這個網站寫法是有漏洞的，只要進到首頁，我們就可以看到「攻擊範例」的跳出視窗。

下面我們解釋如何避免這類攻擊

## 基本輸出處理（output escaping） {#basic-output-escaping}

如果你確定你收到的資料是純文字，你可以在輸出時使用`Html::encode()`作為過濾：

`php echo Html::encode($data);`

## 處理 HTML 輸出 {#dealing-with-html-output}

如果你收到的資料可能有HTML，那會比較麻煩。Yii 有內建的[HtmlPurifier helper](http://www.yiiframework.com/doc-2.0/yii-helpers-basehtmlpurifier.html)，可以清理危險的HTML內容。我們可以在視圖裡面使用以下程式碼做過濾：

```php
echo HtmlPurifier::process(
$data
);
```

> 備註: `HtmlPurifier`不快，所以不要太常呼叫。最好將其輸出做快取

## 其他資料 {#see-also}

* [OWASP article about XSS](https://www.owasp.org/index.php/Cross-site_Scripting_%28XSS%29)
  .
* [HtmlPurifier helper class](http://www.yiiframework.com/doc-2.0/yii-helpers-basehtmlpurifier.html)
  .
* [HtmlPurifier website](http://htmlpurifier.org/)
  .



