layout: post
title: jstips
date: 2016-01-12 23:22:50
tags: [Js]
---

来自于GitHub上的一个开源项目[jstips](https://github.com/loverajoel/jstips)
,每天一点Js知识。做一些笔记，有一些技巧对性能有提升，但是
可读性并不好，就看一下算了。
<!-- more -->

## 0. 向数组中加入元素
使用`push()`向数组中加入值非常简单，但是这有一种性能更好
的方法:


    var arr = [1, 2, 3, 4, 5];
    
    arr.push(6);
    arr[arr.length] = 6; //43% faster in Chrome 47.0.2526.106 on Mac OS X 10.11.1

两种方法都会修改原数组。

现在试试在数组开头加入元素:

    var arr = [1, 2, 3, 4, 5];
    arr.unshift(0);
    [0].concat(arr); // 98% faster in Chrome 47.0.2526.106 on Mac OS X 10.11.1

有一点要注意，`unshift()`改变了原数组，`concat`返回一个
新数组。

使用`splice`在数组中间插入元素是很容易的，而且很高效。

    var items = ['one', 'two', 'three', 'four'];
    items.splice(items.length / 2, 0, 'hello');


## 1.AngularJs `$digest` vs `$apply`
没研究过ng。。。

## 2.React.js Keys在子组件中是很重要的
[key](https://facebook.github.io/react/docs/multiple-components.html#dynamic-children)
是你通过数组动态创建子组件时必须传递给每个子组件的属性。
这是React用来识别DOM中每个组件的唯一且不变的id，并且知得知
这是不同的组件(know that it's a different component and not the same one. )
使用keys可以确保子组件~预处理~(preserved)，不会重新创建
并阻止其他奇怪的事情发生。
- 使用对象中已存在的唯一值
- 在父组件中定义key，而不是子组件
{% raw %}

    //bad
    ...
    render() {
        <div key={{item.key}}>{{item.name}}</div>
    }
    ...
    
    //good
    <MyComponent key={{item.key}}/>
{% endraw %}
- [使用数组的index是很糟糕的实践](https://medium.com/@robinpokorny/index-as-a-key-is-an-anti-pattern-e0349aece318#.44zfpwn60)
- `random()`不会起作用


//bad
{% raw %}
<MyComponent key={{Math.random()}}/>
{% endraw %}
- 你可以创建自己的唯一id，确保这个生成方法够快并且在项目
中使用它。
- 当子组件的数量很大或者涉及到昂贵代价的组件(动画?)时，使用keys
能够提升性能。
- [你必须为所有ReactCSSTranstionGroup的子组件提供key](http://docs.reactjs-china.com/react/docs/animation.html)


    ## ES6. String.prototype.includes()
    你可以用这个方法判断一个字符串中是否包含另一个字符串。
    
    'showonne'.includes('show') //true
    //在ES7中，该方法甚至可以用于数组
    [1, 2, 3].includes(1) //true

(问题是浏览器支持不大好)

## 衡量Js代码块性能的小技巧
使用`console.time(label)`和`console.timeEnd(label)`。

[console.time()具体介绍](https://developer.mozilla.org/zh-CN/docs/Web/API/Console/time)


## 清空数组
一般情况下，我们清空一个数组都是使用这样的方法:

    var foo = [1, 2, 3]
    foo = []

不过还有另一种方法:

    var foo = [1, 2, 3]
    foo.length = 0

前者实际上是将一个新数组的引用分配给这个变量，而之前数组
的引用是不受影响的，因此包含之前内容的数组引用还在内存
中，导致内存泄露。

后者是将数组中的内容全部删除，同时也改变了其他对该数组
的引用。

    var foo = [1, 2]
    var bar = [1, 2]
    
    var foo1 = foo;
    var bar1 = bar;
    
    foo = []
    bar.length = 0
    
    console.log(foo, bar, foo1, bar1) //[] [] [1, 2] []














