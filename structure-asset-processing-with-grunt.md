# 使用 Grunt 進行資源管理（Asset processing with Grunt） {#asset-processing-with-grunt}

Yii 2.0 已經[有相當不錯的資源管理系統](http://www.yiiframework.com/doc-2.0/guide-structure-assets.html)。他可以處理發布，映射（mapping），格式轉換，組合和壓縮。 還算不錯，但是如果我們正在和前端團隊合作，而我們對於資源管理的需求可能超過 Yii 可以處理的情況。

這時候，將資源管理交給[ Grunt](http://gruntjs.com/) 是個好方法。因為 Grunt 有許多成熟的套件，可以滿足我們在客戶端開發上幾乎所有想到的需求。

## 準備 {#get-ready}

We'll start with basic application template. Its installation is [described in official guide](http://www.yiiframework.com/doc-2.0/guide-start-installation.html).

If you haven't [installed Node.js](http://nodejs.org/), do so. After it's done install TypeScript, Grunt and its required plugins by executing the following commands in project root directory:

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

It adds a link to`http://example.com/css/all.css`in debug mode and a link to`http://example.com/css/all.min.css`with modification time \(cache busting\) in production mode. The file itself will be published by Grunt.

Right before`<?php $this->endBody() ?>`add:

```php
<?= Html::jsFile(YII_DEBUG ? '@web/js/lib.js' : '@web/js/lib.min.js?v=' . filemtime(Yii::getAlias('@webroot/js/lib.min.js'))) ?>
<?= Html::jsFile(YII_DEBUG ? '@web/js/all.js' : '@web/js/all.min.js?v=' . filemtime(Yii::getAlias('@webroot/js/all.min.js'))) ?>
```

Same as with CSS, it adds a link for JS that is published via Grunt.

Now create`Gruntfile.js`in the root of the project. The file describes what grunt will do with your assets:

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

Now Grunt will look in`assets/js`,`assets/less`and`assets/ts`for clientside source files.

Create`assets/js/all.json`:

```js
[
    "vendor/bower/jquery/dist/jquery.js",
    "vendor/bower/bootstrap/dist/js/bootstrap.js",
    "vendor/yiisoft/yii2/assets/yii.js",
    "vendor/yiisoft/yii2/assets/yii.validation.js",
    "vendor/yiisoft/yii2/assets/yii.activeForm.js"
]
```

`all.json`lists JavaScript files to process into`lib.js`. In the above we're doing the same things standard Yii asset management does: adding jQuery, bootstrap and Yii's JavaScript.

Now create`assets/less/all.less`:

```js
@import "../../vendor/bower/bootstrap/less/bootstrap.less";
@import "site.less";
```

and`assets/less/site.less`. Its content should be copied from`web/css/site.css`.

## 使用方法 {#how-to-use-it}

* Run`grunt build`to process assets.
* During development you could run`grunt`and the process will watch for changes and rebuild files necessary.
* In order to add JavaScript files, put these into`assets/js`and list their names in`assets/js/all.json`.
* In order to add [LESS files](http://lesscss.org/), put these into`assets/less`and list their names in`assets/less/all.less`.
* In order to add [TypeScript files](http://www.typescriptlang.org/) just put these into`assets/ts`
  .



