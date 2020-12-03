layout: post
title: 回头看看自己star过的项目
date: 2016-01-11 18:18:04
tags: [工具]
---

回头浏览一下当时star过的开源项目，看看这段时间都学到了
什么，用到了什么。
<!-- more -->

#### [cnpm](https://github.com/cnpm/cnpmjs.org)
淘宝npm镜像

#### [markdown-js](https://github.com/evilstreak/markdown-js)
markdown的一个javascript解析器，当时应该是为了让自己的应
用支持md格式找的相关库，不过当时好像没用明白，最后选择了
`marked`。

#### [node-browserify](https://github.com/substack/node-browserify)
Browserify让你可以在浏览器端使用`require('moudles')`打包
你的所有依赖。
你可以在自己的`main.js`中使用Node.js的`require()`语法，比
如:
```
var sub = require('sub');
```
然后通过Browserify打包你的文件，他会帮你处理这些依赖。
```
browserify main.js -o bundle.js
```
然后在页面中使用`bundle.js`就可以了~
最开始看到这个东西的时候以为就是可以在浏览器使用`require()`
后来在对Js的学习过程中才发现其实这货对Js模块化有很大的意
义。

#### [amdjs-api](https://github.com/amdjs/amdjs-api)
呃，Js模块化发展过程产物，AMD规范，在学习Js模块化历史的
过程中看到的。

#### [amazeui](https://github.com/amazeui/amazeui)
中国首个移动优先的模块化HTML5前端框架，当时amaze做活动
star送马克杯(-,-就star了，其实用法和bootstrap这种差不多
都是UI框架，现在amazeui有webapp和react版本的组件框架，
使用起来会方便很多。

#### [React](https://github.com/facebook/react)
目前最火的前端框架，组件化的思想和使用虚拟DOM的性能让很
多开发者为之欢呼。组件化在前端发展过程中已经得到大家认
可，可复用性强，另外大家发现操作DOM对性能影响很大，因此
虚拟DOM也成了一个优势(心疼jQuery，从从前的必备库变成大
家拿来黑的点也真是够了。。。)。

#### [React-Native](https://github.com/facebook/react-native)
使用React开发手机原生应用，现在ios和android版本都已经发
布了，RN的宗旨就是做到"learn once,write anywhere",尝试着
跑了一下官方demo，感觉好神奇。。。

#### [react-mixin](https://github.com/brigand/react-mixin)
react-mixin,并没有写过 - -@。

#### [Coding](https://github.com/Coding/Coding.github.io)
东半球最受欢迎的代码托管平台，这个hexo博客就是在WebIDE中
托管的，哈哈，再也不用担心blog文件夹丢失了~

#### [hjs-webpack](https://github.com/HenrikJoreteg/hjs-webpack)
可以帮你快速配置一个使用es6，react，webpack的开发环境，
不过这种东西还是自己配置更舒服些吧，haha~

#### [stf](https://github.com/openstf/stf)
通过浏览器控制和管理安卓设备，拿来debug应该会很方便，没
用过。。。

#### [iScroll](https://github.com/cubiq/iscroll)
smooth scrolling for the web，浏览器滑动解决方案。

#### [noder-react-native](https://github.com/soliury/noder-react-native)
用react-native写的cnodejs的web app。

#### [lodash](https://github.com/lodash/lodash)
Js工具库，提供了一些方便，高效的数组or对象处理方法。

#### [Socket.IO](https://github.com/socketio/socket.io)
为Node.js服务应用提供的实时应用框架，API简单，功能强大，
浏览器兼容好。

#### [N1](https://github.com/nylas/N1)
使用Electron, React, and Flux构建的开源邮件客户端。

#### [marked](https://github.com/chjj/marked)
markdown解析or生成库，方便好用。

#### [node](https://github.com/nodejs/node)
前端发展上的里程碑。

#### [Vue](https://github.com/vuejs/vue)
简单且强大的前端MVVM框架，API简单易用，以数据驱动视图，
让Web应用性能更好。

#### [js-yaml](https://github.com/nodeca/js-yaml)
用来解析yaml的js库。

#### [CSS-Secrets](https://github.com/cssmagic/CSS-Secrets)
由CSS魔法翻译的CSS书，解读了一些神奇的CSS效果，让你重
拾CSS的乐趣。

#### [fetch](https://github.com/github/fetch)
fetch API,以promise风格来写网络请求，将来应该可以取代
xhr吧。。。fetch请求默认**不带cookie**！

#### [js-module-7day](https://github.com/Huxpro/js-module-7day)
JavaScript模块化七日谈，Hux黄玄写的一个关于Js模块化的
Slide，讲得很好，是一个浪漫极客，不要太羡慕。

#### [chalk](https://github.com/chalk/chalk)
可以让你的命令行字符串变得colorful。

#### [gulp-nodemon](https://github.com/JacksonGariety/gulp-nodemon)
用来启动node服务的gulp插件。

#### [browser-sync](https://github.com/BrowserSync/browser-sync)
浏览器同步调试工具，再也不用手动刷新啦。

。。。写不动了，以后有时间再补吧，回头看了看自己star过
的项目，有的还能想起来当时为什么会用到这写东西，也有的
根本就没用过，不过都是学习的过程。

平时通过微博，论坛等途径了解到的东西还是不少的，以后的
主要任务就是逐渐消化了，并且要多写！


