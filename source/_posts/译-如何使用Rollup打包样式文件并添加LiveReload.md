layout: post
title: '[译]如何使用Rollup打包样式文件并添加LiveReload'
date: 2016-10-31 23:12:41
tags: ['Rollup', 'Js']
---


通过这个教程学习如何使用JavaScript打包工具Rollup配合PostCSS来取代Grunt或Gulp处理样式文件。

上一篇文章中，我们完成了[使用Rollup打包前端JavaScript入门](https://code.lengstorf.com/learn-rollup-js/)。

这篇文章包含`Part II`和`Part III`。

[Part II](https://code.lengstorf.com/learn-rollup-css/#stylesheets)会继续在上次的项目中进行，为Rollup添加处理样式的能力，使用PostCSS进行一些转换，让我们能使用更简单的变量写法和嵌套规则等语法糖。

然后完成[Part III](https://code.lengstorf.com/learn-rollup-css/#livereload)，圆满结束。第三部分将为项目添加文件监听和LiveReload，这样每当文件变化时就不用再手动地打包bundle文件了。

<!-- more -->

## 准备工作

我们会在上周的项目基础上继续进行，因此如果你还没有看上一节，推荐你[先看一下](https://code.lengstorf.com/learn-rollup-js/)。

>NOTE: 如果你没有项目的副本，你可以通过这条命令克隆在Part I结束这个状态下的项目：`git clone -b part-2-starter --single-branch https://github.com/jlengstorf/learn-rollup.git`

## Part II：如何在下一代应用中使用Rollup.js： PostCSS

你可以轻松地处理CSS并注入到文档的`head`中，这取决于你的项目如何配置，也是Rollup另一个优点。

另外，所有的构建过程都在一个地方，降低了开发流程的复杂度 - 这对我们很有帮助，尤其是在团队协作时。

不过糟糕的是，我们使得样式依赖JavaScript，并且在无样式HTML在样式注入前会产生一个短暂的闪烁。这对有些项目来说是无法接受的，并且应该通过像使用PostCSS提取等方式解决。

既然这篇文章是关于Rollup的，那么：来吧。让我们使用Rollup！

#### STEP 0: 在`main.js`中加载样式。

如果你之前从来没用过构建工具，可能感觉这样有点怪，但请跟着我继续。为了在文档中使用样式，我们不会像平常那样使用`<link>`标签，取而代之，我们将在`main.min.js`中使用`import`语句。

现在在`src/scripts/main.js`开头加载样式：

    + // Import styles (automatically injected into <head>).
    + import '../styles/main.css';

      // Import a couple modules for testing.
      import { sayHelloTo } from './modules/mod1';
      import addArray from './modules/mod2';

      // Import a logger for easier debugging.
      import debug from 'debug';
      const log = debug('app:log');

      // The logger should only be disabled if we’re not in production.
      if (ENV !== 'production') {

        // Enable the logger.
        debug.enable('*');
        log('Logging is enabled!');
      } else {
        debug.disable();
      }

      // Run some functions from our imported modules.
      const result1 = sayHelloTo('Jason');
      const result2 = addArray([1, 2, 3, 4]);

      // Print the results on the page.
      const printTarget = document.getElementsByClassName('debug__output')[0];

      printTarget.innerText = `sayHelloTo('Jason') => ${result1}\n\n`;
      printTarget.innerText += `addArray([1, 2, 3, 4]) => ${result2}`;

#### STEP 1: 安装PostCSS Rollup插件。

首先需要Rollup PostCSS插件，使用如下命令安装：

>npm install --save-dev rollup-plugin-postcss

#### STEP 2: 更新`rollup.config.js`.

然后，添加插件到`rollup.config.js`:

      // Rollup plugins
      import babel from 'rollup-plugin-babel';
      import eslint from 'rollup-plugin-eslint';
      import resolve from 'rollup-plugin-node-resolve';
      import commonjs from 'rollup-plugin-commonjs';
      import replace from 'rollup-plugin-replace';
      import uglify from 'rollup-plugin-uglify';
    + import postcss from 'rollup-plugin-postcss';

      export default {
        entry: 'src/scripts/main.js',
        dest: 'build/js/main.min.js',
        format: 'iife',
        sourceMap: 'inline',
        plugins: [
    +     postcss({
    +       extensions: [ '.css' ],
    +     }),
          resolve({
            jsnext: true,
            main: true,
            browser: true,
          }),
          commonjs(),
          eslint({
            exclude: [
              'src/styles/**',
            ]
          }),
          babel({
            exclude: 'node_modules/**',
          }),
          replace({
            ENV: JSON.stringify(process.env.NODE_ENV || 'development'),
          }),
          (process.env.NODE_ENV === 'production' && uglify()),
        ],
      };

#### 看一下生成的bundle。

现在我们已经能够处理样式了，可以看一下新生成的bundle，看看它是如何工作的。

运行`./node_modules/.bin/rollup -c`，然后看一下生成的`build/js/main.min.js`，在文件开头几行，可以看到一个名叫`__$styleInject()`的新函数：

    function __$styleInject(css) {
      css = css || '';
      var head = document.head || document.getElementsByTagName('head')[0];
      var style = document.createElement('style');
      style.type = 'text/css';
      if (style.styleSheet){
        style.styleSheet.cssText = css;
      } else {
        style.appendChild(document.createTextNode(css));
      }
      head.appendChild(style);
    }
    __$styleInject("/* Styles omitted for brevity... */");

简单地说，这个函数创建了一个`<style>`元素并设置样式，然后添加到文档的`<head>`标签中。

就在这个函数声明的下方，可以看到通过传入样式调用该函数，通过PostCSS输出。很酷，不是吗？

但现在这些样式并没有真正地被处理；PostCSS只是直接地传输了我们的样式。让我们添加一些需要的PostCSS插件，使得样式能在目标浏览器上工作。

#### STEP 3: 安装必要的PostCSS插件。

我爱PostCSS。我已经放弃LESS阵营了，当所有人都抛弃LESS时，我发现自己或多或少被影响加入了Sass阵营，后来PostCSS出现了我就非常开心地去学。

我喜欢它是因为它提供了部分在LESS和Sass中我喜欢的功能 - 嵌套，简单的变量 - 而且没有完全放弃LESS和Sass中我认为诱人也危险的功能，比如逻辑运算。

我喜欢它的插件模式，而不是一个叫做“PostCSS”的语言。我们可以只选择真正需要的特性 - 更重要的是，我们可以移除不想要的特性。

因此在我们的项目里，只会使用到四个插件 - 两个是语法糖，一个用来在兼容旧浏览器的新CSS特性，一个用来压缩，减少生成的样式文件大小。

- `postcss-simple-vars` — 可以使用Sass风格的变量(e.g. `$myColor: #fff;`，`color: $myColor;`)而不是冗长的CSS语法(e.g. `:root {--myColor: #fff}`，`color: var(--myColor)`)。这样更简洁；我更喜欢较短的语法。

- `postcss-nested` — 允许使用嵌套规则。实际上我不用它写嵌套规则；我用它简化[BEM友好的选择器](http://getbem.com/naming/)的写法并且划分我的区块，元素和修饰到单个CSS块。

- `postcss-cssnext` — 这个插件集使得大多数现代CSS语法([通过最新的CSS标准](https://www.w3.org/Style/CSS/current-work))可用，编译后甚至可以在不支持新特性的旧浏览器中工作。

- `cssnano` — 压缩，减小输出CSS文件大小。相当于JavaScript中对应的UglifyJS。

使用这个命令安装插件：

>npm install --save-dev postcss-simple-vars postcss-nested postcss-cssnext cssnano

#### STEP 4: 更新`rollup.config.js`。

接下来，在`rollup.config.js`引入PostCSS插件，在配置对象的`plugins`属性上添加一个`postcss`。

      // Rollup plugins
      import babel from 'rollup-plugin-babel';
      import eslint from 'rollup-plugin-eslint';
      import resolve from 'rollup-plugin-node-resolve';
      import commonjs from 'rollup-plugin-commonjs';
      import replace from 'rollup-plugin-replace';
      import uglify from 'rollup-plugin-uglify';
      import postcss from 'rollup-plugin-postcss';

    + // PostCSS plugins
    + import simplevars from 'postcss-simple-vars';
    + import nested from 'postcss-nested';
    + import cssnext from 'postcss-cssnext';
    + import cssnano from 'cssnano';

      export default {
        entry: 'src/scripts/main.js',
        dest: 'build/js/main.min.js',
        format: 'iife',
        sourceMap: 'inline',
        plugins: [
          postcss({
    +       plugins: [
    +         simplevars(),
    +         nested(),
    +         cssnext({ warnForDuplicates: false, }),
    +         cssnano(),
    +       ],
            extensions: [ '.css' ],
          }),
          resolve({
            jsnext: true,
            main: true,
            browser: true,
          }),
          commonjs(),
          eslint({
            exclude: [
              'src/styles/**',
            ]
          }),
          babel({
            exclude: 'node_modules/**',
          }),
          replace({
            ENV: JSON.stringify(process.env.NODE_ENV || 'development'),
          }),
          (process.env.NODE_ENV === 'production' && uglify()),
        ],
      };

>NOTE: 在`cssnext()`中配置了`{ warnForDuplicates: false }`是因为它和`cssnano()`都使用了[Autoprefixer](https://github.com/postcss/autoprefixer)，会导致一个警告。不用计较配置, 我们只需要知道它被执行了两次(在这个例子中没什么坏处)并且取消了警告。

#### 检查`<head>`中的输出内容。

插件安装完后，重新构建bundle文件。(`./node_modules/.bin/rollup -c`)，然后在浏览器中打开`build/index.html`。可以看到页面是有样式的，如果我们审查元素可以发现样式被注入到页面头部，压缩简化，添加所有浏览器前缀和其它我们预期的优点：

>样式被PostCSS处理并通过Rpllup注入

太棒了！我们现在拥有十分可靠的构建流程：JavaScript会被打包，无用的代码会被移除，输出文件是经过压缩精简的，样式时通过PostCSS处理后注入到文档头部的。

然而，最后仍然存在一个痛点，每当我们做了一些修改后都不得不手动地重新构建。因此在下个部分，我们让Rollup监听文件的变化，每当有文件改变时就让浏览器自动重新载入文件。

## Part III: 如何在下一代应用中使用Rollup.js：实时加载

现在，我们的项目已经可以打包JavaScript和样式文件了，但仍然是一个手动的过程。而且由于过程的每个步骤都是手动的，相比自动化流程失败风险更高 - 因为每次修改文件后都执行`./node_modules/.bin/rollup -c`太痛苦了 - 我们希望自动重新构建bundle。

>NOTE: 如果你没有项目的副本，你可以通过这条命令克隆在Part II结束这个状态下的项目：: `git clone -b part-3-starter --single-branch https://github.com/jlengstorf/learn-rollup.git`

#### STEP 0: 为Rollup添加监听插件。

基于Node.js的监听器是很常见的开发工具。如果你之前使用过webpack，Grunt，Gulp或者其他构建工具会很熟悉。

监听器是在一个项目中运行的进程，当你修改了它监听的文件夹内的任意文件时就会触发一个动作。对于构建工具而言，通常这个动作是触发重新构建。

在我们的项目中，我们需要监听`src`目录下的任何文件，并且探测到文件变化后希望Rollup重新打包。

为了达到目的，我们使用[`rollup-watch`](https://github.com/rollup/rollup-watch)插件，它和前面的其它Rollup插件有些不同 - but more on that in a bit。让我们先安装插件：

    npm install --save-dev rollup-watch

#### STEP 1: 传入`--watch`标识运行Rollup。

`rollup-watch`与其他插件的不同就是使用这个插件不需要对`rollup.config.js`做任何修改。

取而代之的是，在终端命令中添加一个标识：

    # Run Rollup with the watch plugin enabled
    ./node_modules/.bin/rollup -c --watch

运行完后，可以发现控制台的输出和之前有点不同：

    $ ./node_modules/.bin/rollup -c --watch
    checking rollup-watch version...
    bundling...
    bundled in 949ms. Watching for changes...

这个进程依然保持活动状态，正在监听变化。

如果我们在`src/main.js`做点小变化 - 比如加一条注释 - 在我们保存修改的那一刻新的bundle文件就生成了。

>监听程序执行时，变化会触发重新构建。LINTER会立刻捕获错误。很优雅，不是吗？

这为我们在开发过程中节省了大量时间，不过还可以更进一步。现在我们仍然需要手动刷新浏览器来获取更新后的bundle - 添加一个工具，在bundle更新后自动刷新浏览器。

>TIP: 在终端窗口输入`control + C`结束监听进程。

#### STEP 2: 安装Liveload自动刷新浏览器。

[LiveReload](https://www.npmjs.com/package/livereload)是加速开发的常用工具。它是一个跑在后台的进程，每当有它监听的文件变化时，他就会通知浏览器刷新。

先下载插件：

    npm install --save-dev livereload

#### STEP 3: 注入livereload脚本。

In `src/main.js`, add the following:

在LiveReload工作前，需要向页面中注入一段脚本用于和LiveReload的服务器建立连接。

不过只有开发环境下有这个需求，利用环境变量的能力判断只有在非生产环境下才注入脚本。

在`src/main.js`中添加下面代码：

      // Import styles (automatically injected into <head>).
      import '../styles/main.css';
      
      // Import a couple modules for testing.
      import { sayHelloTo } from './modules/mod1';
      import addArray from './modules/mod2';
      
      // Import a logger for easier debugging.
      import debug from 'debug';
      const log = debug('app:log');
      
      // The logger should only be disabled if we’re not in production.
      if (ENV !== 'production') {
      
        // Enable the logger.
        debug.enable('*');
        log('Logging is enabled!');
      
    +   // Enable LiveReload
    +   document.write(
    +     '<script src="http://' + (location.host || 'localhost').split(':')[0] +
    +     ':35729/livereload.js?snipver=1"></' + 'script>'
    +   );
      } else {
        debug.disable();
      }
      
      // Run some functions from our imported modules.
      const result1 = sayHelloTo('Jason');
      const result2 = addArray([1, 2, 3, 4]);
      
      // Print the results on the page.
      const printTarget = document.getElementsByClassName('debug__output')[0];
      
      printTarget.innerText = `sayHelloTo('Jason') => ${result1}\n\n`;
      printTarget.innerText += `addArray([1, 2, 3, 4]) => ${result2}`;

做一些修改然后保存，现在我们试试看。

>NOTE: Livereload是如何工作的并不重要，简单的解释就是命令行进程监听文件变化，然后通过websockets向客户端脚本发送消息触发重加载。

#### STEP 4: 运行Livereload。

LiveReload安装好并且脚本注入到文档中后，我们可以运行它去监听`build`目录：

    ./node_modules/.bin/livereload 'build/'

>NOTE: 监听`build/`是因为我们只需要在新的bundle产生时才重新构建。

输出结果像下面这样：

    $ ./node_modules/.bin/livereload 'build/'
    Starting LiveReload v0.5.0 for /Users/jlengstorf/dev/code.lengstorf.com/projects/learn-rollup/build on port 35729.

如果我们用浏览器打开`build/index.html` - 务必在开启LiveReload后再刷新页面，确保socket连接处于激活状态 - 可以发现`build/index.html`的变化会让浏览器自动重新加载：

>文件变化触发浏览器重新加载。

非常棒，但仍然不完美：现在我们只能运行Rollup的监听函数或者LiveReload，除非我们打开多个终端会话。这可不理想。接下来我们会选择一个解决办法。

STEP 5: 使用`package.json`脚本简化启动过程。

教程至今，我们都不得不输入`rollup`脚本的全路径，我猜你已经感觉到这很蠢了。

因此我们需要一个能同时运行监听任务和`Livereload`的工具，先将这两个`rollup`命令和LiveReload命令当做脚本写在`package.json`中。

打开`package.json` - 它位于项目根目录。在这个文件里能看到如下内容：

    {
      "name": "learn-rollup",
      "version": "0.0.0",
      "description": "This is an example project to accompany a tutorial on using Rollup.",
      "main": "build/js/main.min.js",
      "scripts": {
        "test": "echo \"Error: no test specified\" && exit 1"
      },
      "repository": {
        "type": "git",
        "url": "git+ssh://git@github.com/jlengstorf/learn-rollup.git"
      },
      "author": "Jason Lengstorf <jason@lengstorf.com> (@jlengstorf)",
      "license": "ISC",
      "bugs": {
        "url": "https://github.com/jlengstorf/learn-rollup/issues"
      },
      "homepage": "https://github.com/jlengstorf/learn-rollup#readme",
      "devDependencies": {
        "babel-preset-es2015-rollup": "^1.2.0",
        "cssnano": "^3.7.4",
        "livereload": "^0.5.0",
        "npm-run-all": "^3.0.0",
        "postcss-cssnext": "^2.7.0",
        "postcss-nested": "^1.0.0",
        "postcss-simple-vars": "^3.0.0",
        "rollup": "^0.34.9",
        "rollup-plugin-babel": "^2.6.1",
        "rollup-plugin-commonjs": "^3.3.1",
        "rollup-plugin-eslint": "^2.0.2",
        "rollup-plugin-node-resolve": "^2.0.0",
        "rollup-plugin-postcss": "^0.1.1",
        "rollup-plugin-replace": "^1.1.1",
        "rollup-plugin-uglify": "^1.0.1",
        "rollup-watch": "^2.5.0"
      },
      "dependencies": {
        "debug": "^2.2.0"
      }
    }

看到`scripts`属性了吗？我们将为它增加两个新属性：

- 一个运行Rollup打包的脚本(原先手动执行的`./node_modules/.bin/rollup -c --watch`)

- 一个启动LiveReload的脚本(原先手动执行的`./node_modules/.bin/livereload 'build/'`)


在`package.json`加上下面的内容：

      {
    "name": "learn-rollup",
    "version": "0.0.0",
    "description": "This is an example project to accompany a tutorial on using Rollup.",
    "main": "build/js/main.min.js",
    "scripts": {
    +     "dev": "rollup -c --watch",
    +     "reload": "livereload 'build/'",
          "test": "echo \"Error: no test specified\" && exit 1"
        },
        "repository": {
          "type": "git",
          "url": "git+ssh://git@github.com/jlengstorf/learn-rollup.git"
        },
        "author": "Jason Lengstorf <jason@lengstorf.com> (@jlengstorf)",
        "license": "ISC",
        "bugs": {
          "url": "https://github.com/jlengstorf/learn-rollup/issues"
        },
        "homepage": "https://github.com/jlengstorf/learn-rollup#readme",
        "devDependencies": {
          "babel-preset-es2015-rollup": "^1.2.0",
          "cssnano": "^3.7.4",
          "livereload": "^0.5.0",
          "npm-run-all": "^3.0.0",
          "postcss-cssnext": "^2.7.0",
          "postcss-nested": "^1.0.0",
          "postcss-simple-vars": "^3.0.0",
          "rollup": "^0.34.9",
          "rollup-plugin-babel": "^2.6.1",
          "rollup-plugin-commonjs": "^3.3.1",
          "rollup-plugin-eslint": "^2.0.2",
          "rollup-plugin-node-resolve": "^2.0.0",
          "rollup-plugin-postcss": "^0.1.1",
          "rollup-plugin-replace": "^1.1.1",
          "rollup-plugin-uglify": "^1.0.1",
          "rollup-watch": "^2.5.0"
        },
        "dependencies": {
          "debug": "^2.2.0"
        }
      }

这些脚本让我们可以使用简称来执行选择的脚本。

使用`npm run dev`运行Rollup。

使用`npm run reload`执行LiveReload。

接下来要做的就是让他们一起运行。

#### STEP 6: 安装同时运行watcher和Livereload的工具。

为了能同时执行Rollup和LiveReload，我们要使用一个叫做[`npm-run-all`](https://www.npmjs.com/package/npm-run-all)的工具。

这是一个强大的工具，我们先不讨论他的全部功能。我们现在只使用它并行执行脚本的能力 - 意味着同一个终端会话可以同时执行两个任务。

先安装`npm-run-all`:

    npm install --save-dev npm-run-all

然后我们要在`package.json`中再加入一条调用`npm-run-all`的脚本。在`scripts`代码块内，添加如下内容(简单起见我省略了文件的大部分内容)：

        "scripts": {
          "dev": "rollup -c --watch",
          "reload": "livereload 'build/' -d",
    +     "watch": "npm-run-all --parallel reload dev",
          "test": "echo \"Error: no test specified\" && exit 1"
        }

保存修改，切换到终端。准备好最后一步！

#### STEP 7: 执行最后的`watch`脚本。

就像之前做的一样。

在终端中运行下面的命令：

    npm run watch

然后刷新浏览器, 改变一下JS或CSS, 浏览器会加载更新后的bundle并自动刷新 - 太奇妙了！

>Liveload + 自动构建如同魔法一般。

现在我们是Rollup专家了。我们的打包代码变得更小更快，开发流程也更轻松快速。

## Further Reading

## 拓展阅读

[PostCSS](http://postcss.org/)

[Some discussion about using JS to insert styles, and when/whether it’s appropriate](http://stackoverflow.com/questions/26882177/react-js-inline-style-best-practices)

这篇文章的代码放在GitHub上。你可以[fork 这个仓库](https://github.com/jlengstorf/learn-rollup)进行修改或测试，[开issue](https://github.com/jlengstorf/learn-rollup/issues)或者报告bug，或者[新建pull request](https://github.com/jlengstorf/learn-rollup/compare)进行建议或者修改。

[原文链接](https://code.lengstorf.com/learn-rollup-css/)