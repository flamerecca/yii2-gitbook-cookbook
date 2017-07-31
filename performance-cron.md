# 用 cronjobs 實做背景任務（Implementing backgroud tasks with cronjobs） {#implementing-backgroud-tasks-cronjobs}

要定期運行背景程式，至少有以下兩種方式：

* 運行控制台應用命令
* 瀏覽器模擬

兩種方式都需要一個處理排程的程式，其不同的部份在於透過什麼方式來達成我們想做到的事情。

在 Linux 和 MacOS系統，一般使用 [cronjobs](https://en.wikipedia.org/wiki/Cron) 處理排程，Windows 系統的使用者可以參閱 [SchTasks](http://technet.microsoft.com/en-us/library/cc725744.aspx)。

## 運行控制台應用命令 {#running-console-application-command}

Running console application command is preferred way to run code. First, implement[Yii's console command](http://www.yiiframework.com/doc-2.0/guide-tutorial-console.html). 然後在 crontab 檔案裡面加入：

```
42 0 * * * php /path/to/yii.php hello/index
```

crontab 裡面的`/path/to/yii`必須是完整路徑，full absolute path to`yii`console entry script. For both basic and advanced project templates it's right in the project root. Then follows usual command syntax.

`42 0 * * *`是s crontab's configuration specifying when to run the command. Refer to cron docs and use [crontab.guru](http://crontab.guru/) service to verify syntax.

### 瀏覽器模擬 {#browser-emulation}

如果你沒有運行本機端程式的權限，只能透過外部來進行操作的話，瀏覽器模擬是個很便利的方案。透過模擬使用者的行為，我們也可以處理一些定期排程的行為。

The difference here is that instead of console controller you're putting your code into web controller and then fetching corresponding page via one of the following commands:

```
GET http://example.com/cron/
wget -O - http://example.com/cron/
lynx --dump http://example.com/cron/ >/dev/null
```

Note that you should take extra care about security since web controller is pretty much exposed. A good idea is to pass special key as GET parameter and check it in the code.

