layout: post
title: 原来这就是UMD
date: 2016-06-09 15:55:02
tags: ['Js']
---

JavaScript在模块化的发展过程中出现和很多概念，比如[CommonJS](http://www.commonjs.org/)，[AMD (Asynchronous Module Definition)](https://github.com/amdjs/amdjs-api/wiki/AMD)，[CMD (Common Module Definition)](https://github.com/cmdjs/specification/blob/master/draft/module.md)等，后来CommonJs规范被Node.js采用，我们可以在Node.js中通过`var http = require('http')`这种写法来引入模块，通过`module.exports`来导出自定义的模块，AMD和CMD也是requirejs和seajs推广过程中的规范化产出，因此让一个模块可以同时支持这么多的规范，就需要在模块定义时进行判断处理。这个判断处理的代码在流行的库和框架中随处可见，不过今天才知道，其实这种模式叫做[UMD (Universal Module Definition)](https://github.com/umdjs/umd)。
<!-- more -->

这个repo规范了UMD API和的定义和实现，让这些模块可以在客户端(浏览器)，服务端或其他地方工作。README里面的一些变体(Variations)规定了各种常规模块的写法，比如`amdWeb.js`规定了可以运行在AMD规范下(即使用requirejs)和浏览器全局下的模块的写法，点进去会发现其实是一小段示例代码:

    (function (root, factory) {
      if (typeof define === 'function' && define.amd) {
        // AMD. Register as an anonymous module.
        define(['b'], factory);
        } else {
          //Browser globals
          root.amdWeb = factory(root.b);
        }
      }(this, function (b) {
        return {};
    }));

可以看出是一段自执行函数，接收`root`和`factory`两个参数，`root`为全局对象，`factory`则为工厂函数，用来返回一个模块的实例化对象。这段代码先通过`typeof define === 'function' && define.amd`来判断当前环境是否遵循AMD规范(因为`requirejs`和`seajs`都是使用`define`来定义模块，所以需要`define.amd`进一步区分两者)，如果是，通过`define(['b'], factory)`就可以对模块进行定义，如果不是，就执行`root.amdWeb = factory(root.b)`，此时`root === window`，这时window上就挂载了一个factory执行完返回的对象，这个模块就可以全局调用了。

其他的常规模块的定义也都是一个道理，比如`returnExports`定义了一个在nodejs,AMD,browser global下工作的模块，判断AMD环境仍然是通过`typeof define === 'function' && define.amd`，判断nodejs环境则通过`typeof module === 'object' && module.exports`，一般的模块都是判断了所有环境后，如果没有符合条件的就默认为浏览器环境，直接挂在在window上。

JavaScript的世界真是其乐无穷。。。



