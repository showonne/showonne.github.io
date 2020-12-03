layout: post
title: '[译] 如果你会了SASS，你就会了ES2015'
date: 2016-08-31 20:03:48
tags: [Sass]
---

>如果你会了Sass，你就会了ES2015!...虽然不是全部，但也有很多惊人的相似之处！当学习[最新的ECMAScript 规范](http://www.ecma-international.org/ecma-262/6.0/)，ES2015(以前被称为ES6)时，碰到了一些让我立刻想到Sass的特性。


<!-- more -->

## SASS语言和ES2015
[Sass语言](http://sass-lang.com/)非常有表现力，最初于2006年发布，使用Ruby编写，也模仿了Ruby的语法和一些语言场景。至于其他特性，分号和花括号在[3.0.0版本](http://sass-lang.com/documentation/file.SASS_CHANGELOG.html#scss_sassy_css)实现了，使Sass感觉更像(也使他能兼容)CSS。这些变化，再加上Sass核心的可扩展性和SassScript的实用性，意味着这是一个新的兼容CSS的强大编程语言。写样式变得更有力并且编程在设计师和视觉开发中更受欢迎。不幸的是，[Sass开发依然遭到没有成为"真正的开发"的炮轰，我希望这篇文章可以改变这些观念。](https://twitter.com/intent/tweet?via=Una&text=Sass%20devs%20still%20get%20flack%20for%20not%20being%20%22real%20devs.%22%20I%20hope%20this%20post%20changes%20those%20minds.&url=http%3A%2F%2Funa.im%2Fsass-es2015%2F&original_referer=http%3A%2F%2Funa.im%2Fsass-es2015%2F)


每个Sass语言场景的背后都有很多思考，其中很重要的一点就是准确性高于定制性(也就是[为什么](https://github.com/sass/sass/issues/525#issuecomment-8995033)Sass不允许像[PostCSS](http://postcss.org/)那样自定义属性)。新的ECMAScript 2015 标准(ES2015)在2015年六月正式发布，为JavaScript添加了大量新的，实用的特性，使其更清晰并且更易用。

然而大多数浏览器仍然不支持ES2015，你可以使用像[Babel](https://babeljs.io/)这样的转换器将代码转换成兼容性好的ES5代码。预编译步骤(当然，我知道他们不完全相同)只是这两种语言相似部分的开始。

>作为一个Sass开发者，你可能没注意到很多Sass的特性现在在ES2015中已经实现了，对于JavaScript开发者，反之亦然。

## 1.模板字符串插值
这个ES2015特性让我第一次意识到这两种规范多么的相似。两种字符串插值几乎完全相同(这让我很开心)。事实上，ES2015引入了一个**很赞的**基于反引号的字符串模板。

这意味着你不再需要为了普通字符串拼接和静态DOM操作(许多情况下)而引入其他的模板引擎了。这真是太棒了！还提高了代码间的一致性。

>Wow，[Dictionary.com](http://www.dictionary.com/browse/interpolation)对于插值的定义真的很烂。看下面的例子,本质上只是建立字符串变量到其真实值的引用的一种方式。

看上去就像这样 —— 不用这么写:

    "First Name: " + fName + "\nLast Name: " + lName

只需要这样：

    `First Name: ${fName}
     Last Name: ${lName}`

这**恰好**和Sass很像！好好想想，在Sass中，变量是这样的:`$varName`，因为`$`已经被占用了，所以使用`#`来表示插值。

    //.js
    `variable: ${varName}`

    //.scss
    "variable: #{$varName}"

字符串插值在写Sass的混合和向DOM插入内容(例如：使用CSS的content属性)时超级好用。我们会在后面的例子中发现它在Sass开发中多么重要和常用。

## 默认参数
默认参数在编写灵活的函数(在Sass中叫混合)时特别有用。我经常在Sass中使用它，为没定义任何参数的函数或混合提供预定义的默认值，当你在使用时就可以选择传入自己定义的值或者空着它并且使用默认值。现在，你在JavaScript中也可以使用这些特性！

声明函数时，你可以在参数列表中为变量在右侧声明一个默认参数(像Sass一样):

    //.js
 
    function sayHello(recipient = "beautiful"){
      return "Hello" + recipient;
    } 
    sayHello();
    sayHello("sunshine");

    /*console*/
    > Hello beautiful
    > Hello sunshine
 
 

    //.scss 
    @mixin showOutline($color: #f00) {
      outline: 1px solid $color;
    }
 
    .one {
      @include showOutline()
    }
 
    .two {
      @include showOutline(#ccc)
    }

    /*css ouput*/
 
    .one {
      outline: 1px solid #f00;
    }
 
    .two {
       outline: 1px solid #ccc;
    }

在Sass中同样需要注意任何带有默认值的变量一定要跟在没有默认值的变量的后面(也就是声明函数/混合时参数的顺序)。如果你在想为何要遵守这个规则，可以很容易发现如果一个含有默认值的参数后面跟着一个不含默认值的参数(必填参数)时，如果你不传入必填参数就会崩溃。

## 3.rest参数
关于函数/混合参数中另一个相似的地方，ES2015的`rest参数`和Sass中的`参数列表`参数类型也很像！定义函数时，rest参数(参数列表)也必须位于一串参数的最后一个。

函数定义中Rest参数通过[展开运算符](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Spread_operator)(`...`)创建，你可能发现了展开运算符也可以用于数组中的可迭代对象，但是和在函数**定义**中使用时表现有点[不同](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/rest_parameters)。

在传入参数的长度不确定时就可以使用参数列表或rest参数。举个例子，Sass中`box shadow`和`gradients`是很常用的。在Javascript中，rest参数通过在参数名前加`...`表示，Sass中则是在参数后面:

    //.js

    function PrintMe(firstArg, ...theRest) {
      console.log(firstArg, theRest);
    }
 
    PrintMe('Hello', 'You', 'Look', 'Nice');
 
    /*console*/

    > Hello ["You", "Look", "Nice"]
 
    //.scss
 
    @mixin funCircle($size, $gradient...) {
      width: $size;
      height: $size;
      border-radius: 50%;
      background: radial-gradient($gradient);
    }
 
    .entrancing {
      @include funCircle(50px, blue 10%, red 80% ,pink);
    } 

    /*css output*/

    .entrancing {
      width: 50px;
      height: 50px;
      border-radius: 50%;
      background: radial-gradient(blue 10%, red 80%, pink);
    }

上面两个例子中，参数列表/rest参数的值在没被任何处理的情况下会被拼在一起返回。

## 4."FOR-OF"循环
ES2015中数组[新增的循环结构](https://hacks.mozilla.org/2015/04/es6-in-depth-iterators-and-the-for-of-loop/)和Sass中的循环结构非常像。ES2015中引入了`for-of`循环。也许你很好奇`他和for-in有什么不同？`

`for-of`循环迭代器引用的是数组元素的`值`而不是`索引`。这样更灵活而且你可以直接遍历数据(数组中的值)而不是对象的属性(像`for-in`那样)。由于这个限制，`for-in`的问题之一就是你不能从一个`for-in`循环中跳出或者返回。

在Sass中，任何循环的工作方式都如同`for-of`一样而不是`for-in`，无论是map还是list，或者lists中的list。虽然语法不同(`each-in` vs `for-of`)循环机制是相同的：

    //.js

    let colorArray = ["red", "yellow", "green"];
 
    for (let color of colorArray) {
      console.log(color);
    }

    //.scss

    $colorList: "red", "yellow", "green";
 
    @each $color in $colorList {
      @debug $color;
    }
 

如果我使用`for (let color in colorArray)`(用`in`代替`of`)结果将变成`0, 1, 2`而不是`red, yellow, green`。这两种循环方法最大的不同就在这里。

>Sass中的`@debug`语句(如上所示)和`console.log`很像，不过，你也可以使用`@return`这个方法。而且，这些循环在生成多个类和样式时(后面的部分会有例子)很常用。

## 5.MAPS
ECMAScript2015中提供了一种新的数据结构：maps！Map非常适合用于[配置信息](http://hugogiraudel.com/2014/05/05/bringing-configuration-objects-to-sass/)并且能够通过`key: value`对形式获取它们的元素。`它和对象有什么不同？`好问题。对象中的`“key”`总是会被转换成字符串。而[map](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Map)中的key可以是任何类型，不会被转换成字符串。对于混合值类型和在运行时之前就知道所有值的情况下仍然应该使用对象，不过map在其他方面真的很有用。

Maps也是可迭代的！所以你可以像下面展示的那样(在`for-of`循环中)去获取值(Sass中也可行！)。这有一个简单的例子：

    //.js

    let colorMap = new Map();
 
    colorMap.set("primary", "red");
    colorMap.set("secondary", "yellow");
    colorMap.set("tertiary", "green");
 
    for (let [key, value] of colorMap) {
      console.log(`key: ${key}, value: ${value}`);
    }
 
    /*console*/

    > key: primary, value: red
    > key: secondary, value: yellow
    > key: tertiary, value: green
 

    //.scss
 
    $colorMap: (
      "primary": "red",
      "secondary": "yellow",
      "tertiary": "green"
    );
 
 
    @each $key, $value in $colorMap {
      .color--#{$key} {
        color: #{$value};
      }
    }
 
    /*css output*/

    .color--primary {
      color: red;
    }
 
    .color--secondary {
      color: yellow;
    }
 
    .color--tertiary {
      color: green;
    }

>你可能已经发现我在这个例子中使用了插值语法(比如`#{$value}`而不是简单的`$value`)。因为使用后者时，Sass将在结果中输出字符串：(比如：`color: "green"`而不是`color: green`),这在CSS中将无法生效。

Sass提供了几个像`map-get`函数这样的细节实现，通常用来获取map的值，`map-has-key`对调试非常有用：

    //.scss

    @if map-has-key($colorMap, $color) {
        $color: map-get($colorMap, $color);
    } @else {
      @if type-of($color) != color {
        @error "Invalid color name: `#{$color}`.";
      }
    }

JavaScript的map也有[get和set](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Map/get)方法！在Javascript中是简单的`get()`而不是`map-get()`：

    //.js

    var colorMap = new Map();
    colorMap.set("primary", "red");
 
    colorMap.get("primary");
    // red


你可以在Sass(和Js)中对map进行删除，合并，压缩(zip?)等操作。这真的是个组织代码的好方法，将对象信息转化成颜色指数。


## 6.类继承
最后但并非最不重要的，ES2015引入了类(当然，不是真正的类，而是基于原型的让我们假装写类的语法糖)。还有和类一起引入的继承！(听起来很熟悉？)

它们的使用方法和Sass的`@extend`指令非常相似，Javascript中的继承允许创建一个继承其父类方法和属性的新的类(然后，`super()`方法能调用新类父级的方法。例如`class Corgi extends Dog`)，`Corgi`能获取`Dog`的`bark()`方法通过`super.bark()`。

你也可以在构造函数中执行此方法来实例化所有父类的方法。这让`extends`的工作方式和`@extend`更像！

很困惑？看代码更容易些：

    //.js

    class Dog {
      constructor(name) {
        this.name = name;
      }
 
      bark(words) {
        console.log(`WOOF!!! ${words} WOOF!`);
      }
    }
 
    class Corgi extends Dog {
      constructor() {
        super();
      }
 
      waddle() {
      // corgi waddle
      }
    }
 
    //Steve is a cori
    let steve = new Corgi;
 
    //Steve can bark like a Dog
    steve.bark('I like bacon!')
 
    //AND Steve can waddle, too!
 
    /*console*/

    > WOOF!!! I like bacon! WOOF!
 

    //.scss
 
    .plain-link {
      font-weight: 600;
      text-decoration: none;
      border-bottom: 2px dotted currentColor;
    }
 
    .pink-link {
      @extend .plain-link;
      color: hotpink;
    }
 
    .blue-link {
      @extend .plain-link;
      color: blue;
    }
 
    /*css output*/

    .plain-link, .pink-link, .blue-link {
      font-weight: 600;
      text-decoration: none;
      border-bottom: 2px dotted currentColor;
    }
 
    .pink-link {
      color: hotpink;
    }
 
    .blue-link {
      color: blue
    }

对于Sass中的混合来说`@extend`和`@include`有一个很关键的不同。我们使用`@extend`获取了来自`.plain-link`的全部样式，而且这些样式可以在每个继承该类的元素间共享 —— 像JavaScript的例子中从父类获取的JS方法一样。然后我们可以使用额外的不同属性(Sass例子中的color，JavaScript例子中的waddle方法)，他们在父类中不可用，但是在继承他们的子类中可用。

>为了满足大家的好奇心，我在[这](http://una.im/classy-css/#💁)写过关于`@extend`的全部知识和我如何系统地在Sass中使用他们。

虽然这些都是不完整的比较，但[Sass和JavaScript之间真的存在很多相通的地方](https://twitter.com/intent/tweet?via=Una&text=there%20are%20very%20real%20connections%20between%20Sass%20and%20JavaScript&url=http%3A%2F%2Funa.im%2Fsass-es2015%2F&original_referer=http%3A%2F%2Funa.im%2Fsass-es2015%2F)，尤其是一些ES2015的新特性。

`希望你喜欢这篇文章并从中学到了新东西！`

可以在[Ruben](https://twitter.com/rubencodes)提出建议确保我的文章更好理解！

原文地址: http://una.im/sass-es2015/#💁z

