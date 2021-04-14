---
title: Gulp压缩和PWA
date: 2020-09-08 23:53:30
tags: 
  - 优化
categories: 
  - Hexo
description: Gulp压缩和PWA
comments: true
top_img: https://cdn.jsdelivr.net/gh/liuyeweiQX/picBed/img/post/img10.jpg
cover: https://cdn.jsdelivr.net/gh/liuyeweiQX/picBed/img/post/img10.jpg
---
### Gulp压缩
Gulp是一款自动化构建的工具
#### 安装Gulp
npm install --global gulp-cli
安装过程中若报权限错误，则执行sudo npm install --global gulp-cli命令
#### 插件安装
- HTML压缩插件：
npm install gulp-htmlclean --save-dev
npm install --save gulp-htmlmin
- CSS压缩插件：
npm install gulp-clean-css --save-dev
- JS压缩插件：（二选一）
terser 是 ES6+ 的 JavaScript 解析器
gulp-babel 是一个 JavaScript 转换编译器，可以把 ES6 转换成 ES5
  - npm install --save-dev gulp-uglify  ;  npm install terser
  - npm install --save-dev gulp-uglify  ;  npm install --save-dev gulp-babel @babel/core @babel/preset-env
- 图片压缩插件：
npm install --save-dev gulp-imagemin

#### 创建gulpfile.js文件
在 Hexo 的根目录，创建 gulpfile.js 文件
```
const gulp = require("gulp");
const workbox = require("workbox-build");
const cleanCSS = require('gulp-clean-css');
const htmlmin = require('gulp-htmlmin');
const htmlclean = require('gulp-htmlclean');
const imagemin = require('gulp-imagemin');
// tester (如果使用tester,把下面4行前面的//去掉)
// const uglifyjs = require('terser');
// const composer = require('gulp-uglify/composer');
// const pump = require('pump');
// const minify = composer(uglifyjs, console);

// babel (如果不是使用bebel,把下面兩行註釋掉)
const uglify = require('gulp-uglify')
const babel = require('gulp-babel')

// minify js - babel（ 如果不是使用bebel,把下面註釋掉）
gulp.task('compress', () =>
    gulp.src(['./public/**/*.js', '!./public/**/*.min.js'])
        .pipe(babel({
            presets: ['@babel/preset-env']
        }))
        .pipe(uglify().on('error', function (e) {
            console.log(e)
        }))
        .pipe(gulp.dest('./public'))
);

// minify js - tester (如果使用tester,把下面前面的//去掉)
// gulp.task('compress', function (cb) {
//   const options = {};
//   pump([
//     gulp.src(['./public/**/*.js', '!./public/**/*.min.js']),
//     minify(options),
//     gulp.dest('./public')
//   ],
//   cb
//   );
// });

// css
gulp.task('minify-css', () => {
    return gulp.src('./public/**/*.css')
        .pipe(cleanCSS({
            compatibility: 'ie11'
        }))
        .pipe(gulp.dest('./public'));
});

// 壓縮 public 目錄內 html
gulp.task('minify-html', () => {
    return gulp.src('./public/**/*.html')
        .pipe(htmlclean())
        .pipe(htmlmin({
            removeComments: true, // 清除 HTML 註釋
            collapseWhitespace: true, // 壓縮 HTML
            collapseBooleanAttributes: true, // 省略布爾屬性的值 <input checked="true"/> ==> <input />
            removeEmptyAttributes: true, // 刪除所有空格作屬性值 <input id="" /> ==> <input />
            removeScriptTypeAttributes: true, // 刪除 <script> 的 type="text/javascript"
            removeStyleLinkTypeAttributes: true, // 刪除 <style> 和 <link> 的 type="text/css"
            minifyJS: true, // 壓縮頁面 JS
            minifyCSS: true, // 壓縮頁面 CSS
            minifyURLs: true
        }))
        .pipe(gulp.dest('./public'))
});

// 压缩 public/uploads 目录内图片
gulp.task('minify-images', async () => {
    gulp.src('./public/img/**/*.*')
        .pipe(imagemin({
            optimizationLevel: 5, //类型：Number  预设：3  取值範围：0-7（优化等级）
            progressive: true, //类型：Boolean 预设：false 无失真压缩jpg图片
            interlaced: false, //类型：Boolean 预设：false 隔行扫描gif进行渲染
            multipass: false, //类型：Boolean 预设：false 多次优化svg直到完全优化
        }))
        .pipe(gulp.dest('./public/img'));
});

gulp.task('generate-service-worker', () => {
    return workbox.injectManifest({
        swSrc: './sw-template.js',
        swDest: './public/sw.js',
        globDirectory: './public',
        globPatterns: [
            "**/*.{html,css,js,json,woff2}"
        ],
        modifyURLPrefix: {
            "": "./"
        }
    });
});

// 執行 gulp 命令時執行的任務
gulp.task("default", gulp.series("generate-service-worker", gulp.parallel(
    'compress','minify-html', 'minify-css', 'minify-images'
)));
```
注意： 如果有使用到 Butterfly 主题提供的 mermaid 标签，那需要把 52 行.pipe(htmlclean()) 注释掉，同时，把 55 行的 collapseWhitespace: true, 改为 false

#### 运行
在hexo g之后运行gulp

### PWA
渐进式网络应用程式（英语：Progressive Web Apps，简称：PWA）是一种普通网页或网站架构起来的网络应用程式，但它可以以传统应用程式或原生移动应用程式形式展示给用户。这种应用程式形态视图将目前最为现代化的浏览器提供的功能与行动装置的体验优势相结合。

当你的网站实现了 PWA，那就代表了:
- 用户可以添加你的博客到电脑╱手机的桌面，以原生应用般的方式浏览你的博客
- 用户可以更快速地浏览你的博客
- 用户可以离线浏览你的博客

使用 Service Worker。我们使用 Workbox 这个工具生成 sw.js 以快速实现 Service Worker ，并实现页面的预缓存和页面更新后的提醒功能。

#### 开启设置和配置manifest.json
主题配置文件中开启pwa选项
```
pwa:
  enable: true
  manifest: /img/pwa/manifest.json
  apple_touch_icon: /img/pwa/apple-touch-icon.png
  favicon_32_32: /img/pwa/32.png
  favicon_16_16: /img/pwa/16.png
  mask_icon: /img/pwa/safari-pinned-tab.svg
```
#### 在主题的source 目录中创建 manifest.json 文件。
```
{
    "name": "string", //应用全称
    "short_name": "Junzhou", //应用简称
    "theme_color": "#49b1f5", //匹配浏览器的地址栏颜色
    "background_color": "#49b1f5",//加载应用时的背景色
    "display": "standalone",//首选显示模式 其他选项有：fullscreen,minimal-ui,browser
    "scope": "/",
    "start_url": "/",
    "icons": [ //该数组指定icons图标参数，用来时适配不同设备（需为png，至少包含一个192px*192px的图标）
        {
          "src": "images/pwaicons/36.png", //图标文件的目录，需在source/目录下自行创建。
          "sizes": "36x36",
          "type": "image/png"
        },
        {
            "src": "images/pwaicons/48.png",
          "sizes": "48x48",
          "type": "image/png"
        },
        {
          "src": "images/pwaicons/72.png",
          "sizes": "72x72",
          "type": "image/png"
        },
        {
          "src": "images/pwaicons/96.png",
          "sizes": "96x96",
          "type": "image/png"
        },
        {
          "src": "images/pwaicons/144.png",
          "sizes": "144x144",
          "type": "image/png"
        },
        {
          "src": "images/pwaicons/192.png",
          "sizes": "192x192",
          "type": "image/png"
        },
        {
            "src": "images/pwaicons/512.png",
            "sizes": "512x512",
            "type": "image/png"
          }
      ],
      "splash_pages": null //配置自定义启动动画。
  }
```

#### 安装插件：
npm install workbox-build gulp --save-dev

#### 创建gulpfile.js文件（见上文）

#### 在 Hexo 的根目录，创建一个 sw-template.js 文件
```
const workboxVersion = '5.1.3';


importScripts(`https://storage.googleapis.com/workbox-cdn/releases/${workboxVersion}/workbox-sw.js`);


workbox.core.setCacheNameDetails({
    prefix: "your name"
});


workbox.core.skipWaiting();


workbox.core.clientsClaim();


workbox.precaching.precacheAndRoute(self.__WB_MANIFEST,{
    directoryIndex: null
});


workbox.precaching.cleanupOutdatedCaches();


// Images
workbox.routing.registerRoute(
    /\.(?:png|jpg|jpeg|gif|bmp|webp|svg|ico)$/,
    new workbox.strategies.CacheFirst({
        cacheName: "images",
        plugins: [
            new workbox.expiration.ExpirationPlugin({
                maxEntries: 1000,
                maxAgeSeconds: 60 * 60 * 24 * 30
            }),
            new workbox.cacheableResponse.CacheableResponsePlugin({
                statuses: [0, 200]
            })
        ]
    })
);


// Fonts
workbox.routing.registerRoute(
    /\.(?:eot|ttf|woff|woff2)$/,
    new workbox.strategies.CacheFirst({
        cacheName: "fonts",
        plugins: [
            new workbox.expiration.ExpirationPlugin({
                maxEntries: 1000,
                maxAgeSeconds: 60 * 60 * 24 * 30
            }),
            new workbox.cacheableResponse.CacheableResponsePlugin({
                statuses: [0, 200]
            })
        ]
    })
);


// Google Fonts
workbox.routing.registerRoute(
    /^https:\/\/fonts\.googleapis\.com/,
    new workbox.strategies.StaleWhileRevalidate({
        cacheName: "google-fonts-stylesheets"
    })
);
workbox.routing.registerRoute(
    /^https:\/\/fonts\.gstatic\.com/,
    new workbox.strategies.CacheFirst({
        cacheName: 'google-fonts-webfonts',
        plugins: [
            new workbox.expiration.ExpirationPlugin({
                maxEntries: 1000,
                maxAgeSeconds: 60 * 60 * 24 * 30
            }),
            new workbox.cacheableResponse.CacheableResponsePlugin({
                statuses: [0, 200]
            })
        ]
    })
);


// Static Libraries
workbox.routing.registerRoute(
    /^https:\/\/cdn\.jsdelivr\.net/,
    new workbox.strategies.CacheFirst({
        cacheName: "static-libs",
        plugins: [
            new workbox.expiration.ExpirationPlugin({
                maxEntries: 1000,
                maxAgeSeconds: 60 * 60 * 24 * 30
            }),
            new workbox.cacheableResponse.CacheableResponsePlugin({
                statuses: [0, 200]
            })
        ]
    })
);
workbox.googleAnalytics.initialize();
```
#### 在主题配置文件中，添加需要的 css 和 js

```
inject:
  head:
    - '<style type="text/css">.app-refresh{position:fixed;top:-2.2rem;left:0;right:0;z-index:99999;padding:0 1rem;font-size:15px;height:2.2rem;transition:all .3s ease}.app-refresh-wrap{display:flex;color:#fff;height:100%;align-items:center;justify-content:center}.app-refresh-wrap a{color:#fff;text-decoration:underline;cursor:pointer}</style>'
  bottom:
    - '<div class="app-refresh" id="app-refresh"> <div class="app-refresh-wrap"> <label>✨ 网站已更新最新版本 👉</label> <a href="javascript:void(0)" onclick="location.reload()">点击刷新</a> </div></div><script>function showNotification(){if(GLOBAL_CONFIG.Snackbar){var t="light"===document.documentElement.getAttribute("data-theme")?GLOBAL_CONFIG.Snackbar.bgLight:GLOBAL_CONFIG.Snackbar.bgDark,e=GLOBAL_CONFIG.Snackbar.position;Snackbar.show({text:"已更新最新版本",backgroundColor:t,duration:5e5,pos:e,actionText:"点击刷新",actionTextColor:"#fff",onActionClick:function(t){location.reload()}})}else{var o=`top: 0; background: ${"light"===document.documentElement.getAttribute("data-theme")?"#49b1f5":"#1f1f1f"};`;document.getElementById("app-refresh").style.cssText=o}}"serviceWorker"in navigator&&(navigator.serviceWorker.controller&&navigator.serviceWorker.addEventListener("controllerchange",function(){showNotification()}),window.addEventListener("load",function(){navigator.serviceWorker.register("/sw.js")}));</script>'
```        
**重点：配置了这些，可能会出现很多问题，本人尝试后有取消了这些配置，谨慎对待，学习学习就好**                                                                                                                                               