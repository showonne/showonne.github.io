title: "transform,trasition,animation"
date: 2015-05-20 21:45:18
tags: Css
---

**transform(转换)**: 用于对元素的2D或3D转换，常用的有转换(translate),缩放(scale),旋转(rotate)和倾斜(skew).


<!-- more -->
例如，我们将页面中的一个盒子沿X轴变换10像素，沿Y轴变换20像素，X,Y轴方向均缩放为二倍，旋转45度，沿X,Y轴各倾斜20度.

	{
		transform:
			translate(10px,20px)
			scale(2,2)
	        rotate(45deg)       
	        skew(20deg,20deg)
	}

**transition(过渡)**:用于设置元素的某个或全部样式是否需要添加过渡效果.其四个属性分别用于控制需要过渡的样式，完成过渡效果所需时间，过渡效果的速度曲线，过渡何时开始。

如果我们想实现一个鼠标放在div上让其变色的例子，使用hover伪类就可以简单完成，但是hover颜色的变化是在一瞬间完成的，如果我们想让它变化得平缓，就可以使用transition完成,需要注意的是:transition属性是加在要变换的元素上而不是其伪类上。

	div{
		transition: all 3s; //指定所有样式过渡时间为3s
		background-color: black;
	}

	div:hover{
		background-color: white;
	}


**animation(动画)**:可以用来实现自定义的动画效果，即元素样式的自定义变化。在使用animation属性之前，要使用@keyframes来创建动画，再绑定到需要使用动画的元素上。

使用@keyframes定义动画非常简单，只需要指定一个动画的名称用于绑定到元素上，然后用CSS将效果描述出来就可以了。

定义一个fir的动画:

	@keyframes fir{
		from{
			background-color: black;
		}
		to{
			background-color: white;
		}
	}

其中的`from`，`to`也可以使用百分比代替，使动画更灵活。

创建好动画规则，就可以为元素添加动画，使用animation属性为元素添加相应动画。使用animation时至少要指定动画名，完成动画所需时间两个参数.

	div{
		animation: fir 5s;
	}

由于都是CSS3的新特性，浏览器对这些属性的支持程度不同，所以使用时需要添加对应浏览器前缀.

	{
		transform:rotate(7deg);
		-ms-transform:rotate(7deg);
		-moz-transform:rotate(7deg); 
		-webkit-transform:rotate(7deg); 
		-o-transform:rotate(7deg);
	}

	{
		transition: all 3s;
		-moz-transition: all 3s;
		-webkit-transition: all 3s;
		-o-transition: all 3s;
	}
	//注意:不是"-浏览器前缀-@keyframes"
	{
		@keyframes fir{}
		@-moz-keyframes fir{}
		@-webkit-keyframes fir{}
		@-o-keyframes fir{}

		animation: fir 2s;
		-moz-animation: fir 2s;
		-webkit-animation: fir 2s;
		-o-animation: fir 2s;
	}