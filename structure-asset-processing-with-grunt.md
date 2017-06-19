# 使用 Grunt 進行資源管理（Asset processing with Grunt） {#asset-processing-with-grunt}

Yii 2.0 已經[有相當不錯的資源管理系統](http://www.yiiframework.com/doc-2.0/guide-structure-assets.html)。他可以處理發布，映射（mapping），格式轉換，組合和壓縮。 還算不錯，但是如果我們正在和前端團隊合作，而我們對於資源管理的需求可能超過 Yii 可以處理的情況。

這時候，將資源管理交給[ Grunt](http://gruntjs.com/) 是個好方法。因為 Grunt 有許多成熟的套件，可以滿足我們在客戶端開發上幾乎所有想到的需求。

## 準備 {#get-ready}

我們從Yii基本應用開始，安裝方法在[官方教學裡面](http://www.yiiframework.com/doc-2.0/guide-start-installation.html)。

如果你還沒[安裝 Node.js](http://nodejs.org/)，必須先安裝。安裝好Node.js過後， 安裝TypeScript、Grunt 以及 相關套件如下：

```
npm install -g grunt-cli

npm install grunt --save-dev
npm install grunt-contrib-copy --save-dev
npm install grunt-contrib-less --save-dev
npm install grunt-contrib-uglify --save-dev
npm install grunt-contrib-watch --save-dev
npm install grunt-concat-sourcemap --save-dev
npm install typescript --save-dev
npm install grunt-typescript --save-dev
```

## 作法 {#how-to-do-it}

首先，關閉 Yii 內建的資源管理，我們修改`config/web.php`如下：

```php
$params = require(__DIR__ . '/params.php');

$config = [
    // ...
    'components' => [
        // ...
        'assetManager' => [
            'bundles' => false,
        ],
    ],
];

// ...

return $config;
```

再來修改`views/layouts/main.php`。

在`<?= Html::csrfMetaTags() ?>`後面加上：

```php
<?= Html::cssFile(YII_DEBUG ? '@web/css/all.css' : '@web/css/all.min.css?v=' . filemtime(Yii::getAlias('@webroot/css/all.min.css'))) ?>
```

這會在 debug 模式中加入`http://example.com/css/all.css`，並在正式模式中加入含修改時間（避免 cache）的`http://example.com/css/all.min.css`。這個檔案會被 Grunt 發布出去。

在`<?php $this->endBody() ?>`前面，加入：

```php
<?= Html::jsFile(YII_DEBUG ? '@web/js/lib.js' : '@web/js/lib.min.js?v=' . filemtime(Yii::getAlias('@webroot/js/lib.min.js'))) ?>
<?= Html::jsFile(YII_DEBUG ? '@web/js/all.js' : '@web/js/all.min.js?v=' . filemtime(Yii::getAlias('@webroot/js/all.min.js'))) ?>
```

與前面設定CSS的部份相同，這會加入被Grunt 發布的JS檔案連接。

現在在根目錄建立`Gruntfile.js`。該檔案設定 grunt 會怎麼處理你的資源：

```js
module.exports = function (grunt) {
    grunt.initConfig({
        less: {
            dev: {
                options: {
                    compress: false,
                    sourceMap: true,
                    outputSourceFiles: true
                },
                files: {
                    "web/css/all.css": "assets/less/all.less"
                }
            },
            prod: {
                options: {
                    compress: true
                },
                files: {
                    "web/css/all.min.css": "assets/less/all.less"
                }
            }
        },
        typescript: {
            base: {
                src: ['assets/ts/*.ts'],
                dest: 'web/js/all.js',
                options: {
                    module: 'amd',
                    sourceMap: true,
                    target: 'es5'
                }
            }
        },
        concat_sourcemap: {
            options: {
                sourcesContent: true
            },
            all: {
                files: {
                    'web/js/all.js': grunt.file.readJSON('assets/js/all.json')
                }
            }
        },
        copy: {
            main: {
                files: [
                    {expand: true, flatten: true, src: ['vendor/bower/bootstrap/fonts/*'], dest: 'web/fonts/', filter: 'isFile'}
                ]
            }
        },
        uglify: {
            options: {
                mangle: false
            },
            lib: {
                files: {
                    'web/js/lib.min.js': 'web/js/lib.js'
                }
            },
            all: {
                files: {
                    'web/js/all.min.js': 'web/js/all.js'
                }
            }
        },
        watch: {
            typescript: {
                files: ['assets/ts/*.ts'],
                tasks: ['typescript', 'uglify:all'],
                options: {
                    livereload: true
                }
            },
            js: {
                files: ['assets/js/**/*.js', 'assets/js/all.json'],
                tasks: ['concat_sourcemap', 'uglify:lib'],
                options: {
                    livereload: true
                }
            },
            less: {
                files: ['assets/less/**/*.less'],
                tasks: ['less'],
                options: {
                    livereload: true
                }
            },
            fonts: {
                files: [
                    'vendor/bower/bootstrap/fonts/*'
                ],
                tasks: ['copy'],
                options: {
                    livereload: true
                }
            }
        }
    });

    // Plugin loading
    grunt.loadNpmTasks('grunt-typescript');
    grunt.loadNpmTasks('grunt-concat-sourcemap');
    grunt.loadNpmTasks('grunt-contrib-watch');
    grunt.loadNpmTasks('grunt-contrib-less');
    grunt.loadNpmTasks('grunt-contrib-uglify');
    grunt.loadNpmTasks('grunt-contrib-copy');

    // Task definition
    grunt.registerTask('build', ['less', 'typescript', 'copy', 'concat_sourcemap', 'uglify']);
    grunt.registerTask('default', ['watch']);
};
```

現在 Grunt 會根據`assets/js`、`assets/less`和`assets/ts`來管理用戶端原始檔。

建立`assets/js/all.json`：

```js
[
    "vendor/bower/jquery/dist/jquery.js",
    "vendor/bower/bootstrap/dist/js/bootstrap.js",
    "vendor/yiisoft/yii2/assets/yii.js",
    "vendor/yiisoft/yii2/assets/yii.validation.js",
    "vendor/yiisoft/yii2/assets/yii.activeForm.js"
]
```

`all.json`列出要處理進`lib.js`的JavaScript 檔案。上面的程式碼做的事情與一般的 Yii 資源管理相同：加入 jQuery、bootstrap、和 Yii 本身的 JavaScript。

現在建立`assets/less/all.less`：

```js
@import "../../vendor/bower/bootstrap/less/bootstrap.less";
@import "site.less";
```

以及建立`assets/less/site.less`。 `site.less`的內容應該從`web/css/site.css`複製過來。

## 使用方法 {#how-to-use-it}

* 運行`grunt build`來處理資源。
* 開發期間，我們可以運行 `grunt`，程式會監控檔案的改變，並在需要的時候重建檔案。
* 增加 JavaScript 檔案時，放在`assets/js`，並列入`assets/js/all.json`。
* 增加 [LESS 檔案](http://lesscss.org/)時，放在`assets/less`，並列入`assets/less/all.less`。
* 增加 [TypeScript 檔案](http://www.typescriptlang.org/)時，放在`assets/ts`。
  .



