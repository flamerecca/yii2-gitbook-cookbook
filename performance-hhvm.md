# 在 HHVM 上面運行Yii 2.0（Running Yii 2.0 on HHVM） {#running-yii-20-on-hhvm}

HHVM 簡單說，是Facebook 所作的另一種 PHP 引擎替代品，其運行效能比起目前的 PHP 5.6 有顯著提高（比起 PHP 5.5 和 PHP 5.4 提高更多）。通常對一般的 PHP 程式，藉由引進HHVM，我們可以獲得約 10–40% 的效能提昇。如果是行程密集（processing-intensive）的程式，HHVM 比起常用的Zend PHP，可能會有數倍效率的提昇。

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

你可以在一個伺服器上同時運行 HHVM 和 php-fpm。因為這兩個都是透過 fastcgi 來運作，所以互相轉換很容易。我們甚至可以同時運作這兩者。

如果要這樣做，我們必須將一般的 PHP 與 HHVM，運行在不同的 port 上面。

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

如上，除了 port 不同以外，其他設置是幾乎一樣的。

## 測試 {#test-it-first}

HHVM 或多或少有[與其他框架一起測試運作](http://hhvm.com/frameworks/)，Yii 也包括在內，除了一些小問題之外應該都可以運行。

不過，HHVM 與 PHP相比[還是有許多不便](https://github.com/facebook/hhvm/labels/php5 incompatibility)，所以要實際上線之前，一定要重新測試過整個網頁。

## 錯誤回報 {#error-reporting}

HHVM 的錯誤回報與 PHP 的不同，根據預設，如果 HHVM 程式出現錯誤，你只會看到空白頁面，而不是錯誤訊息。

在 HHVM 的 config 裡面，加入以下條件，來修正這個問題：

```
hhvm.debug.server_error_message = true
```



