title: 使用CSS修改滚动条效果(webkit)
date: 2015-04-14 21:23:25
tags: Css
---

前几天找了找hexo的主题，无意间发现了这款"next"主题，瞬间吸引了我，只有黑白两种颜色的搭配简洁大方，如果非要挑些不和谐的地方就只有浏览器的滚动条了，不过也没什么问题，用简单的css代码就可以让其有所改善。<!-- more -->

实际应用时常用到的属性一般为三个:scrollbar,scrollbar-thumb(拖动条),scrollbar-track(轨道)。

以webkit内核为例:

	//设置滚动条的宽高，width控制纵向滚动条宽度，height同理.
	::webkit-scrollbar {
		width: 17px; 
		height: 17px;
	}
	//设置拖动条的样式
	::-webkit-scrollbar-thumb{
		background-color: rgb(34,34,34);
		//设置滚动条圆角
		border-radius: 4px; 
	}
	//设置滚动条轨道
	::-webkit-scrollbar-track{
		background-color: rgb(255,255,255);
		border-radius: 4px;
	}

这样就可以为滚动条添加上简单的样式，当然，还可以使用伪类添加一些简单的交互效果:

	//设置滚动条被点下时的样式
	::-webkit-scrollbar-thumb:active{
		background-color: #ccc;
	}

简单几步，就可以让滚动条更适应主题了！