# 用 cronjobs 實做背景任務（Implementing backgroud tasks with cronjobs） {#implementing-backgroud-tasks-cronjobs}

要定期運行背景程式，至少有以下兩種方式：

* 運行本地端程式指令（running console application command）
* 瀏覽器模擬（browser emulation）

兩種方式都需要一個處理排程的程式，其不同的部份在於透過什麼方式來達成我們想做到的事情。

在 Linux 和 MacOS系統，一般使用 [cronjobs](https://en.wikipedia.org/wiki/Cron) 處理排程，Windows 系統的使用者可以參閱 [SchTasks](http://technet.microsoft.com/en-us/library/cc725744.aspx)。

## 運行本地端程式指令 {#running-console-application-command}

一般來說，比較建議用運行本地端程式的方式，來處理背景程式的需求。

首先，實做 [Yii 的命令列指令](http://www.yiiframework.com/doc-2.0/guide-tutorial-console.html)。然後在 crontab 檔案裡面加入：

```
42 0 * * * php /path/to/yii.php hello/index
```

crontab 裡面的`/path/to/yii.php`必須是到`yii.php`程式的完整路徑，不管是 basic 還是 advanced 版本的 Yii 裡面，`yii.php`都是在根目錄。然後接著程式的命令參數，像是`hello/index`。

`42 0 * * *`是 crontab 用來指定什麼時間運行特定指令的設置。對這部份不熟悉的人可以參考 cron 說明文件，或者使用 [crontab.guru](http://crontab.guru/) 來確認格式。

### 瀏覽器模擬 {#browser-emulation}

如果你沒有運行本機端程式的權限，只能透過外部來進行操作的話，瀏覽器模擬是個很便利的方案。

透過模擬使用者的行為，我們也可以處理一些定期排程的需求。不同的地方在於我們不將資料放在console裡面的控制器，而是放在網頁上，比方說`http://example.com/cron/`，然後透過以下指令取得網頁上的資料：

```
GET http://example.com/cron/
wget -O - http://example.com/cron/
lynx --dump http://example.com/cron/ >/dev/null
```

上面的 [lynx](https://zh.wikipedia.org/wiki/Lynx) 是一個純文字的瀏覽器。

注意，因為這種作法基本上允許任何人透過網頁與該控制器接觸，所以安全性的部份要更加小心。一個提高安全性的方式，是透過網址的 GET 參數，傳一個特殊數值作為密碼，並在網頁裡面檢查該密碼。

