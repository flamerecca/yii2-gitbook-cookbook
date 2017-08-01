# 在 HHVM 上面運行Yii 2.0（Running Yii 2.0 on HHVM） {#running-yii-20-on-hhvm}

HHVM 簡單說，是Facebook 所作的另一種 PHP 引擎替代品，其運行效能比起目前的 PHP 5.6 有顯著提高（比起 PHP 5.5 和 PHP 5.4 提高更多）。通常對一般的 PHP 程式，藉由引進HHVM，我們可以獲得約 10–40% 的效能提昇。如果是 For processing-intensive ones it could be times faster than with usual Zend PHP.

## 只限 Linux {#linux-only}

HHVM 只限 Linux可以使用，沒有對應 Windows 的產品，而在 MacOS 環境下，只有有限的功能，沒有JIT編譯器（JIT compiler）。

## 安裝 HHVM {#installing-hhvm}

HHVM 的安裝很簡單，下面是在 Debian 的安裝方法：

```
sudo apt-key adv --recv-keys --keyserver hkp://keyserver.ubuntu.com:80 0x5a16e7281be7a449
echo deb http://dl.hhvm.com/debian wheezy main | sudo tee /etc/apt/sources.list.d/hhvm.list
sudo apt-get update
sudo apt-get install hhvm
```

其他的發布方式教學[在這](https://docs.hhvm.com/hhvm/getting-started/getting-started)。

## Nginx 設置 {#nginx-config}

You can have both HHVM and php-fpm on the same server. Switching between them using nginx is easy since both are working as fastcgi. You can even have these side by side. In order to do it you should run regular PHP on one port and HHVM on another.

```nginx
server {
    listen 80;
    root /path/to/your/www/root/goes/here;

    index index.php;
    server_name hhvm.test.local;

    location / {
        try_files $uri $uri/ /index.php?$args;
    }

    location ~ \.php$ {
        include fastcgi_params;
        fastcgi_param SCRIPT_FILENAME $document_root/$fastcgi_script_name;
        fastcgi_pass   127.0.0.1:9001;
        try_files $uri =404;
    }
}

server {
    listen 80;
    root /path/to/your/www/root/goes/here;

    index index.php;
    server_name php.test.local;

    location / {
        try_files $uri $uri/ /index.php?$args;
    }

    location ~ \.php$ {
        include fastcgi_params;
        fastcgi_param SCRIPT_FILENAME $document_root/$fastcgi_script_name;
        fastcgi_pass   127.0.0.1:9000;
        try_files $uri =404;
    }
}
```

As you can see, configurations are identical except port number.

## 測試 {#test-it-first}

HHVM 或多或少有[與其他框架一起測試運作](http://hhvm.com/frameworks/)，Yii 也包括在內，除了一些小問題之外應該都可以運行。

不過Still, there are [many incompatibilities compared to PHP](https://github.com/facebook/hhvm/labels/php5 incompatibility)so make sure to test application well before going live.

## 錯誤回報 {#error-reporting}

HHVM behavior about errors is different than PHP one so by default you're getting nothing but a blank white screen instead of an error. Add the following to HHVM config to fix it:

```
hhvm.debug.server_error_message = true
```



