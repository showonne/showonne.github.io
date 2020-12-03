---
title: 兼容iPhoneX刘海屏
date: 2019-09-03 23:41:32
tags:
---

最近在做一个WebApp，需要兼容 iPhoneX 的刘海屏幕，iPhoneX 与其他屏幕最主要的区别就是顶部的刘海和底部的小黑条。如果网页没有针对他们做过任何兼容，页面的部分内容就会被这两部分遮挡。因此官方也针对这种情况推出了一些新的设计规范和对应的解决方案。

<!-- more -->

## 安全区域
安全区域就是视窗里不会受到圆角，顶部状态栏和底部操作栏影响的区域，对 iPhoneX 的适配，就可以转化为将可视内容都放在安全区域来做，对于会被影响到的地方，采用 `padding` 填充。

<div style="display: flex;">
<img height="300" src="https://developer.apple.com/design/human-interface-guidelines/ios/images/LayoutGuides_Portrait.svg">
<img height="300" src="https://developer.apple.com/design/human-interface-guidelines/ios/images/LayoutGuides_Landscape.svg">
</div>

## 1. 使用整个屏幕
第一个特性叫`viewport-fit`，是对 `viewport` 拓展的元标签，用来控制插入行为。这一特性在 **iOS 11** 中可用。默认值是 `auto`，可以设置成 `cover` 让页面布局充满屏幕：

```html
<meta name='viewport' content='initial-scale=1, viewport-fit=cover'>
```

## 2. 考虑安全区域
先让页面布局充满屏幕，再通过 `padding` 将内容控制在安全区域内就可以完美兼容了。为此，WebKit 在 **iOS 11** 引入了一个新的CSS函数 `env()` 和一组预定义的环境变量 `safe-area-inset-left`, `safe-area-inset-right`, `safe-area-inset-top` 和 `safe-area-inset-bottom`，它们组合在一起就能展示出安全区域的范围了：

<img height="300" src="https://webkit.org/wp-content/uploads/safe-areas-1.png">

> 在 iOS 11 中，`env()` 函数叫做 `constant()`，从 `Safari Technology Preview 41` 和 `iOS 11.2 beta` 开始，`constant()` 就被移除了，可以利用 CSS 的 fallback 机制兼容不同版本。

`env()` 的用法和 `var()` 一样，比如用在 `padding` 中：

```css
.body {
    padding-left: env(safe-area-inset-left);
    padding-right: env(safe-area-inset-right);
}
```

## 注意事项
1. 只有设置了 `viewport-fit=cover`，`env()` 才会起作用。

## 最后
其实客户端启动 WebView 时，也可以直接使用安全区域布局，这样就不需要页面来兼容。