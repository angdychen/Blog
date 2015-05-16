之前有分享过一篇《[如何在移动设备上调试网页](http://www.codingserf.com/index.php/2014/05/debug-on-devices/)》，感谢 xyyjk 留言推荐[BrowserSync](http://www.browsersync.io/)这个工具，自己也花了点时间研究了一下，还是很好上手的。比起之前介绍的方法更加方便（之前的方法已经过时）。BrowserSync可以实时同步更新CSS、JS文件，此外最关键的是全平台支持，即你可以在手机QQ浏览器、微信浏览器里面调试。今天就介绍一下这个工具的用法（以下针对Mac OSX）。
## 一、安装
其实官网已经有很好的 [Get Started](http://www.browsersync.io/#install)来介绍如何安装。我现在是结合[Gulp](http://gulpjs.com/)来用的，在Gulp下更方便一点。官方也有BrowserSync for Gulp的插件，并有明确文档说明[如何与Gulp整合](http://www.browsersync.io/docs/gulp/)。
#### 1、安装开发依赖
在终端里输入：
```bash
$ npm install browser-sync gulp --save-dev
```
如果安装报错请加sudo：
```bash
$ sudo npm install browser-sync gulp --save-dev
```
#### 2、编辑gulpfile.js
如果项目下没有gulpfile.js文件请新建。先来看一下项目的目录结构，bin文件夹是最终发布的目录，我们把它当作BrowserSync代理服务器启动的根目录：
![目录结构](http://www.codingserf.com/wp-content/uploads/2015/03/QQ20150326-1@2x.png "目录结构")
gulpfile.js文件代码：
```
var gulp = require('gulp');
var browserSync = require('browser-sync');
// Static server
gulp.task('browser-sync', function() {
    browserSync({
        server: {
            //指定服务器启动根目录
            baseDir: "./bin"
        }
    });
    //监听任何文件变化，实时刷新页面
    gulp.watch("./bin/**/*.*").on('change', browserSync.reload);
});
```
## 二、启动
打开终端输入：
```
$ gulp browser-sync
```
此时浏览器会自动打开http://localhost:3000的页面，我们会发现这个页面正是指向./bin目录下的index.html，这与我们在gulpfile.js中的设置相符。我们在index.html中有一个蓝色边框的box，并输出浏览器的UA：
![./bin/index.html](http://www.codingserf.com/wp-content/uploads/2015/03/QQ20150326-4@2x.png)
BrowserSync启动后终端界面会有两个端口提示，分别是:3000的项目页面，和:3001的BrowserSync的UI界面，并且每个端口都有供本地（localhost）和外部（局域网IP）访问的URL：
![browsersync 终端](http://www.codingserf.com/wp-content/uploads/2015/03/QQ20150326-3@2x.png)
因为我们把BrowserSync的reload加到了gulp的watch里，所以只要./bin下面的文件发生变化所有访问local URL或External URL的客户端（client）都会自动刷新。
接下来我们把手机和电脑要连接在同一个无线局域网里，我们可以直接通过微信扫码访问External的URL，如本例中的http://192.168.0.141:3000。此时手机显示如下：
![weixin browser](http://www.codingserf.com/wp-content/uploads/2015/03/Screenshot_2015-03-26-12-12-58_WeChat.png)
## 三、调试
我们在PC的浏览器中打开http://localhost:3001，这个端口为3001的URL是BrowserSync的操作界面：
![BrowserSync UI](http://www.codingserf.com/wp-content/uploads/2015/03/QQ20150326-6@2x.png)
界面操作简洁易懂。我们着重关注的是Remote Debug这一项，默认它的所有选项都是关闭的。我们开启`Remote Debugger (weinre)`这一项，然后点击出现的红色字：Access remote debugger (opens in a new tab)，就会打开weinre的控制台界面：
![weinre for BrowserSync UI](http://www.codingserf.com/wp-content/uploads/2015/03/QQ20150326-5@2x.png)
这个界面上会列出当前连接在BrowserSync上的客户端，我们可以选择一个目标（target）来调试，因为我们的微信浏览器访问的是外部链接，即http://192.168.0.141:3000，所以我们点击图中*Targets*下的第一个link。点击后link会变为绿色，表示当前已经可以对这个客户端进行调试了。此时我们点到顶端的*Elements*选项卡上，就会看到微信浏览器里网页的HTML结构。鼠标移动到.box的div上的时候，微信浏览器里的相应的元素就会高亮起来：
![微信浏览器高亮](http://www.codingserf.com/wp-content/uploads/2015/03/QQ20150326-9@2x.png)
此时能做的事情就可想而知了。
需要注意的是，要先打开自己的项目页面，再打开Remote Debugger，这样才能列出当前已经连接的客户端。否则的话，即使项目页面和BrowserSync是连接状态，Remote Debugger也会出现捕捉不到的情况。因此，可能需要关闭再重新开启Remote Debugger进行调试。









