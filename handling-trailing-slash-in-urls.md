# Handling trailing slash in URLs {#handling-trailing-slash-in-urls}

By default Yii handles URLs without trailing slash and gives out 404 for URLs with it. It is a good idea to choose either using or not using slash but handling both and doing 301 redirect from one variant to another.

For example,

```
/hello/world - 200
/hello/world/ - 301 redirect to /hello/world
```

## Using UrlNormalizer {#using-urlnormalizer}

Since Yii 2.0.10 there's`UrlNormalizer`class you can use to deal with slash and no-slash URLs in a very convenient way. Check out the "[URL normalization](http://www.yiiframework.com/doc-2.0/guide-runtime-routing.html#url-normalization)" section in the official guide for details.

## Redirecting via web server config {#redirecting-via-web-server-config}

Besides PHP, there's a way to achieve redirection using web server.

## Redirecting via nginx {#redirecting-via-nginx}

Redirecting to no-slash URLs.

```
location / {
    rewrite ^(.*)/$ $1 permanent;
    try_files $uri $uri/ /index.php?$args;
}

```

Redirecting to slash URLs.

```
location / {
    rewrite ^(.*[^/])$ $1/ permanent;
    try_files $uri $uri/ /index.php?$args;
}
```

## Redirecting via Apache {#redirecting-via-apache}

Redirecting to no-slash URLs.

```
RewriteEngine On
RewriteCond %{REQUEST_FILENAME} !-d
RewriteRule ^(.*)/$ /$1 [L,R=301]
```

Redirecting to slash URLs.

```
RewriteEngine On
RewriteCond %{REQUEST_FILENAME} !-f
RewriteRule ^(.*[^/])$ /$1/ [L,R=301]
```



