# 處理斜線結尾的網址（Handling trailing slash in URLs） {#handling-trailing-slash-in-urls}

Yii 預設網址沒有斜線，如果使用結尾有斜線的網址，會導致 404 找不到頁面。

如果選定結尾有或者沒有斜線，並且在另一種狀況時使用 301 重新導向，而不是找不到頁面，一般來說是比較好的設計。

舉例來說，

```
/hello/world - 200
/hello/world/ - 301 重導向至 /hello/world
```

## 使用 UrlNormalizer {#using-urlnormalizer}

從 Yii 2.0.10 開始，有了`UrlNormalizer`類別，可以簡單的處理斜線結尾或者無斜線結尾的網址。

請參考 Yii 官方教學 [URL normalization](http://www.yiiframework.com/doc-2.0/guide-runtime-routing.html#url-normalization) 章節。

## 透過伺服器 config 重新導向 {#redirecting-via-web-server-config}

除了使用 PHP 程式，我們也可以使用伺服器來重新導向。

## 透過 nginx {#redirecting-via-nginx}

導向至尾端沒有斜線的網址：

```
location / {
    rewrite ^(.*)/$ $1 permanent;
    try_files $uri $uri/ /index.php?$args;
}
```

導向至尾端有斜線的網址：

```
location / {
    rewrite ^(.*[^/])$ $1/ permanent;
    try_files $uri $uri/ /index.php?$args;
}
```

## 透過 Apache {#redirecting-via-apache}

導向至尾端沒有斜線的網址：

```
RewriteEngine On
RewriteCond %{REQUEST_FILENAME} !-d
RewriteRule ^(.*)/$ /$1 [L,R=301]
```

導向至尾端有斜線的網址：

```
RewriteEngine On
RewriteCond %{REQUEST_FILENAME} !-f
RewriteRule ^(.*[^/])$ /$1/ [L,R=301]
```



