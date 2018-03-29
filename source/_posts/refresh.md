---
layout: '[layout]'
title: 实现exprss的全栈自动刷新
date: 2018-2-25 09:31:03
comments: true
toc: true
tags:
	- 前端
	- express
---
**前言**
项目的空档期，就在东一头西一头的折腾（生命在于折腾嘛）。想使用express跟mongodb做一个小小的管理系统，各种准备工作做好了，数据库也跑通了，但是每次修改代码的时候都要重启服务，简直不能忍受。
**1.尝试supervisor**
<!-- more -->
supervisor原本是用于服务器上`Node.js`应用崩溃的时候，自己重新启动。当然它也可以监控你的项目的js文件变化，进而重启来方便我们调试应用程序。
安装：`npm install supervisor -g`
启动：`supervisor -w routes views ./bin/www`,其中`-w routes views`表示要监听的目录
supervisor还支持多种参数，列举如下：
```js
//要监控的文件夹或js文件，默认为'.'
-w|--watch <watchItems>
//要忽略监控的文件夹或js文件
-i|--ignore <ignoreItems>
//监控文件变化的时间间隔（周期），默认为Node.js内置的时间
-p|--poll-interval <milliseconds>
//要监控的文件扩展名，默认为'node|js'
-e|--extensions <extensions>
//要执行的主应用程序，默认为'node'
-x|--exec <executable>
//开启debug模式（用--debug flag来启动node）
--debug
//安静模式，不显示DEBUG信息
-q|--quiet
```
虽然能做到监听文件修改，但是每次都要手动刷新浏览器才能看到效果。
**2.使用gulp+browsersync+nodemon**
安装：`npm install --save-dev gulp`
            `npm install --save-dev browser-sync`
                `npm install --save-dev gulp-nodemon`
添加gilpfile文件：
```js
var browserSync = require('browser-sync');
var reload = browserSync.reload;
var nodemon = require('gulp-nodemon');
var gulp = require('gulp');
var less = require('gulp-less')；

//express启动项
gulp.task("node", function() {
    nodemon({
        script: './bin/www',
        ext: 'js html',
        env: {
            'NODE_ENV': 'development'
        }
    })
});

// 编译less
// 在命令行输入 gulp less 启动此任务
gulp.task('less', function () {
    // 1. 找到 less 文件
    gulp.src('public/less/**/**.less')
    // 2. 编译为css
        .pipe(less())
        // 3. 另存文件
        .pipe(gulp.dest('public/stylesheets/css'))
});

gulp.task('auto', function () {
    // 监听文件修改，当文件被修改则执行 less任务
    gulp.watch('public/less/**.less', ['less'])
})；

gulp.task('start', ["node","less","auto"], function() {
    //所需要监听的文件
    var files = [
        "routes/**",
        'views/**/*.html',
        'views/**/*.ejs',
        'views/**/*.jade',
        'views/**/*.pug',
        'public/**/*.*'
    ];

    browserSync.init(files, {
        proxy: 'http://192.168.49.1:3000', //所要代理的地址，端口要与bin/www中的端口一致
        browser: 'chrome',
        notify: false,
        port: 3001    //代理地址的端口号
    });

    gulp.watch(files).on("change", reload);
});
```
运行：`gulp start`启动服务

![服务启动日志.jpg](http://upload-images.jianshu.io/upload_images/5725206-308b7c92abd58ac6.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
展示一张实际效果图

![](http://upload-images.jianshu.io/upload_images/5725206-ff5f0d1cd6ec692d.gif?imageMogr2/auto-orient/strip)
ok，可以愉快的开发了。不过服务端文件修改后要按两次`ctrl+s`才会生效。
