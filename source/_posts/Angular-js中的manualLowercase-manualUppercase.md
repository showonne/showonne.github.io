layout: post
title: Angular.js中的manualLowercase/manualUppercase
date: 2016-11-13 22:25:33
tags: ['Angular.js']
---

今天在看Angular源码时看到了这么一段：

    // src/Angular.js
    var lowercase = function(string) {return isString(string) ? string.toLowerCase() : string;};
    var uppercase = function(string) {return isString(string) ? string.toUpperCase() : string;};

    var manualLowercase = function(s) {
      /* eslint-disable no-bitwise */
      return isString(s)
          ? s.replace(/[A-Z]/g, function(ch) {return String.fromCharCode(ch.charCodeAt(0) | 32);})
          : s;
      /* eslint-enable */
    };
    var manualUppercase = function(s) {
      /* eslint-disable no-bitwise */
      return isString(s)
          ? s.replace(/[a-z]/g, function(ch) {return String.fromCharCode(ch.charCodeAt(0) & ~32);})
          : s;
      /* eslint-enable */
    };

前两个函数很容易就看出来了，用来进行大小写转换，后两个函数明显也是进行大小写转换，只是通过位运算来进行。但是为什么要这么写呢？

<!-- more -->

`String.fromCharCode`是一个静态方法，根据指定的Unicode编码中的序号值返回一个字符串。

    String.fromCharCode(num1, ..., numN) 

    String.fromCharCode(65, 66, 67) // 'ABC'

`ch.charCodeAt()`返回0到65535之间的整数，表示索引处字符串的UTF-16编码单元。

    str.charCodeAt(index)

    'ABC'.charCodeAt(0) //65
    'ABC'.charCodeAt(1) //66
    'ABC'.charCodeAt(2) //67



Angular.js里把这个函数重写的是因为`String.prototype.toLocaleLowerCase()`表现不一致。

>toLocaleLowerCase()方法返回调用该方法的字符串被转换成小写之后的值，转换规则根据任何本地化特定的大小写映射。toLocaleLowerCase()并不会影响字符串自身的值。在大多数情况下，该方法产生的结果和调用toLowerCase()的结果相同，但是在某些本地环境中，比如土耳其语，它的大小写映射并不遵循在Unicode中的默认的大小写映射，因此会有一个不同的结果。
-摘自MDN的描述

差异就在土耳其和其它的本地化映射不一致。

所以这个函数是把字符串严格按照Unicode来进行映射了一遍，中间通过位运算处理了一下大小写转换(同一个字母大小写之间的ASCII差了32)。

那么问题来了，`String.prototype.toLocaleLowerCase()`有问题，为什么要重写`toLowerCase`?[看这个issue](https://github.com/angular/angular.js/issues/11387)里面的讨论仿佛在说这段代码更像是对老浏览器的hack方法，所以现在可有可无了。


[String.prototype.charCodeAt()](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/String/charCodeAt)

[String.fromCharCode()](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/String/fromCharCode)

[String.prototype.toLocaleLowerCase()](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/String/toLocaleLowerCase)

[按位操作符](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/Bitwise_Operators)