title: CSS实用技巧
date: 2015-05-18 15:52:02
tags: Css
---

一些前端开发过程中的使用技巧,在这里做一下笔记,在学习的过程中不断更新。

<!-- more -->

1.全屏背景图片固定效果

	{
		background: url("path/to/img") no-repeat center center fixed;
		background-size: cover;
	}

2.使用CSS绘制三角形

	//三角形的颜色为border-bottom的颜色
	{
		height: 0;
        width: 0;
        border-bottom: 50px solid #ccc;
        border-right:  40px solid transparent;
        border-left:  40px solid transparent;
	}

3.将元素定位到页面正中心

	{
		height: 100px;
        width: 200px;
        background-color: #ccc;
        position: absolute;
        left: 50%;
        top: 50%;
        margin-top: -50px;
        margin-left: -100px;
	}

4.表格列宽自适应

	td{
		white-space: nowrap;
	}

5.包裹长文本时自动换行

	{
		white-space: pre-line;
        word-wrap: break-word;
        border: 1px solid #ccc;
        width: 100px;
	}
	
6.使用box-shadow模拟outline圆角效果

    //利用box-shadow第四个参数放大投影范围的作用，实际上是将1px的border-radius放大
    target{
        border-radius: 1px;
        box-shadow:0 0 0 15px #ccc;
    }
    
7.让图片随着父容器的大小变化而变化

    //为图片添加max-width:100%,它们的尺寸就会跟随它们的父元素一样扩展和收缩，就能自适应各种平面尺寸/分辨率的移动设备了。
	img{
      max-width: 100%;
	}
	
8.文字溢出部分用...代替

    div{
        overflow: hidden;
        text-overflow: ellipsis;
        white-space: nowrap;
    }
	