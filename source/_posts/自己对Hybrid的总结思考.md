layout: post
title: 自己对Hybrid的总结思考
date: 2016-12-11 00:37:45
tags: ['Hybrid']
---

公司最近上线了一款Hybrid App，所谓Hybrid是指Web和Native的混合形态，很大的一个好处就是Web部分的内容可以随时发布更新，应用无需重新发布。我负责的那部分并没有和Native交互的地方，所以代码和写纯Web时没什么区别。结合整个项目的代码，和一篇来自有赞技术团队的[H5与Native交互之JSBridge技术](http://tech.youzan.com/jsbridge/)文章，总结一下我自己对Hybrid的认识。

<!-- more -->

## 基本概念
- JsBridge: 一种用来连接Web和Native的技术方案，让Native端(Swift, OC, Java)和Web端(JavaScript)可以互相调用。
- webview: Native组件，有类似浏览器的功能。

所以混合开发的本质就是将一些Native的UI替换成webview，webview的内容可以由前端开发，和开发Web App差不多，而和Web App相比，这些webview里的html5页面需要和Native通信。

## Native调用Js

#### ios
1. 通过`webview`的`stringByEvaluatingJavaScriptFromString`实现，本质是调用`window`下的方法。

#### android
1. 通过`webview`的`loadUrl`进行调用。

## Js调用Native

#### ios
1. 通过拦截`webview`里的请求来实现。通过`Web`和`Native`约定好的自定义协议(在ios里叫`scheme`)进行通信。

#### android
1. 和ios一样通过自定义协议进行通信。
2. 安卓可以直接向`webview`中注入全局对象，所以注入后，页面里的Js可以直接调用。
3. 安卓的`webview`层还可以重写`prompt`, `console.log`, `alert`这三个方法。

## 问题
对于自定义协议，Js过修改`location.href`去触发，也可以用添加`iframe`触发。`location.href`连续多次调用，Native只能接收到最后一个。所以使用iframe的方案更好一些，另外对于JsBridge管理的内容，最好封装在一个全局对象中，便于管理。

## 个人思考
混合开发接触的略少，之前提到Hybrid，我能说出的只是“一部分是Native，一部分是Web，Web的部分写好给Native套到webview里就行了。”，然后随之会想到一些框架，比如Ionic，当初还跑过一个Ionic的`tabs`的demo，打开的第一感觉就是: 真的只是套个Web页面啊，一唤起键盘就发现了移动端经典的定位bug。我们公司的原生部分是ios和android开发人员自己写的，和我之前随口说的Ionic的解决方案不大一样，至少前端有了Ionic，很多的Native组件也可以很方便地直接引入，所以可以让Native开发者的参与部分更少，不过如果要定制Native组件UI的话，前端还是不一定能搞定。
特地看了一下Ionic放出`v1.0.0-beta-1`已经是三年前(2013年)的事情了，而我第一次听说Ionic貌似是15年初，分享会没怎么认真听，当时介绍的同学跑了一个`tab`的demo，然后html5版本的就直接运行成功了，我当时还以为是快速构建Web应用的框架，囧。不过这才过了一年时间，混合开发好像就不大那么受追捧了，因为有一个很严重的问题是：在弱网和没网的条件下，webview显示的结果还是超长时间的白板，或者直接当前页面不可用，好像没有一个App应该有的样子。
到现在，React Native和Weex出现了，Js to Native这个方案变得火了起来，另外一方面，看了一些PWA的概念，虽然还是Web，但是已经可以更好地处理离线情况，至少有个App Shell在那里，体验更好了。不知道未来哪种方案会更好，感觉Js to Native好一些。






