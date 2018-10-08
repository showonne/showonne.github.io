layout: post
title: 使用Vue写一个datepicker
date: 2016-08-10 22:29:07
tags: ['Vue', '组件化']
---

前言
---
写插件是很有意思，也很锻炼人，因为这个过程中能发现许多的细节问题。在前端发展的过程中，jQuery无疑是一个重要的里程碑，围绕着这个优秀项目也出现了很多优秀的插件可以直接使用，大大节省了开发者们的时间。jQuery最重要的作用是跨浏览器，而现在浏览器市场虽不完美，但已远没有从前那么惨，数据驱动视图的思想倍受欢迎，大家开始使用前端框架取代jQuery，我个人比较喜欢Vue.js，所以想试着用Vue.js写一个组件出来。

为了发布到npm上，所以给项目地址改名字了，但是内部代码没有改，使用方法比之前方便。
Demo演示: [Here](http://www.showonne.com/vue-date/dist/browser/)
GitHub地址: [Here](https://github.com/showonne/vue-date)
<del><b>希望大家能给个star</b></del>
<!-- more -->

功能&期望
---
这个datepicker目前仅实现了一些常用的功能：
- 选择时间(<del>这话说得有点多余<del>)
- 最大/最小时间限制
- 中/英文切换(其实也就星期和月份需要切换)
- 可以以`.vue`形式使用，也可在浏览器环境中直接使用
- 没了。。。

目录结构
---
万事的第一步依然是创建项目，只是单一组件，结构并不复杂，Datepicker.vue是最重要的组件文件，dist是webpack的输出文件夹，index.js是webpack打包的入口文件，最后是webpack的配置文件，用来对我们的库文件进行打包用的。因此项目结构就是这样：

	.
    ├── LICENSE
    ├── README.md
    ├── index.js
    ├── package.json
    ├── src
    │   └── Datepicker.vue
    └── webpack.config.js

从Datepicker.vue入手
---
以`.vue`的方式写Vue组件是一种特殊写法，每个Vue文件包括`template`, `script`, `style`三部分，`template`最好不要成为片段实例,所以最外层先套一层`div`，当做整个组件的根元素。一个datepicker一般由两部分组成，一个用来显示日期的input框，一个用来选择日期的panel，因为我发现input在移动端会自动唤起键盘，所以没有使用input，直接用了div模拟,通过点击事件决定panel的显隐。`value`是最终的结果，需要和父组件通信，所以将value写成了prop，在父组件中使用`value.sync="xxx"`，datepicker的value就和父组件的`xxx`双向绑定了。

	<template>
		<div class="date-picker">
			<div class="input" v-text="value" @click="panelState = !panelState">
		</div>
		<div class="date-panel" v-show="panelState">
		</div>
	</template>
	
	<scrip>
		export default {
			data () {
				return {
					panelState: false //初始值，默认panel关闭
				}
			},
			props: {
				value: String
			}
		}
	</script>

渲染日期列表
---
一个月最少是28天，如果把周日排在开头，那么最少(1号恰好是周日)需要4行，但是每个月天数30，31居多，而且1号又不一定是周日，我索性干脆按最多的情况设计了，共6行，当月日期没填满的地方用上个月或下个月的日期补齐，这样就方便计算了，而且切换月份时候panel高度不会变化。日期列表的数组需要动态计算，Vue提供了[computed](http://vuejs.org.cn/api/#computed)这个属性，所以直接将日期列表`dateList`写成计算属性。我的方法是将日期列表固定为长度为42的数组，然后将本月，上个月，下个月的日期依次填充。

	computed: {
		dateList () {
			//获取当月的天数
			let currentMonthLength = new Date(this.tmpMonth, this.tmpMonth + 1, 0).getDate()
			//先将当月的日期塞入dateList
			let dateList = Array.from({length: currentMonthLength}, (val, index) => {
				return {
					currentMonth: true,
					value: index + 1
				}
			})
			//获取当月1号的星期是为了确定在1号前需要插多少天
			let startDay = new Date(this.year, this.tmpMonth, 1).getDay()
			//确认上个月一共多少天
      	  let previousMongthLength = new Date(this.year, this.tmpMonth, 0).getDate()
		}
		//在1号前插入上个月日期
		for(let i = 0, len = startDay; i < len; i++){
			dateList = [{previousMonth: true, value: previousMongthLength - i}].concat(dateList)
		}
		//补全剩余位置
		for(let i = 0, item = 1; i < 42; i++, item++){
			dateList[dateList.length] = {nextMonth: true, value: i}
		}
		return dateList
	}

这里用`Array.from`来初始化了一个数组，传入一个Array Like，转化成数组，在拼接字符串时候采用了`arr[arr.length]`和`[{}].concat(arr)`这种方式，因为在JsTips上学到这样做性能更好，文章的最后会贴出相关链接。
这样，日期列表就构建好了，在template中使用`v-for`循环渲染出来

	<ul class="date-list">
        <li v-for="item in dateList"
            v-text="item.value" 
            :class="{preMonth: item.previousMonth, nextMonth: item.nextMonth,
                selected: date === item.value && month === tmpMonth && item.currentMonth, invalid: validateDate(item)}"
            @click="selectDate(item)">
        </li>
    </ul>
                
样式上就可以自己发挥了，怎么喜欢怎么写。需要注意的是循环日期可能会出现上个月或这个月的日期，我通过`previuosMonth`,`currentMonth`和`nextMonth`分别做了标记，对其他功能提供判断条件。
年份和月份的列表都是差不多的道理，年份列表的初始值我直接写在了`data`里，以当前年份为第一个，为了和月份保持一致，每次显示12个，都通过`v-for`渲染。
	
	data () {
		return {
			yearList: Array.from({length: 12}, (value, index) => new Date().getFullYear() + index)
		}
	}

选择日期功能
---
选择顺序是：年 -> 月 -> 日，所以我们可以通过一个状态变量来控制panel中显示的内容，绑定适合的函数切换显示状态。

	<div>
		<div class="type-year" v-show="panelType === 'year'">
			<ul class="year-list">
	          <li v-for="item in yearList"
	              v-text="item"
	              :class="{selected: item === tmpYear, invalid: validateYear(item)}" 
	              @click="selectYear(item)"
	          >
	          </li>
	      </ul>
		</div>
		<div class="type-month" v-show="panelType === 'month'">
			<ul class="month-list">
	          <li v-for="item in monthList"
	              v-text="item | month language"
	              :class="{selected: $index === tmpMonth && year === tmpYear, invalid: validateMonth($index)}" 
	              @click="selectMonth($index)"
	          >
	          </li>
	      </ul>
		</div>
		<div class="type-date" v-show="panelType === 'date'">
			<ul class="date-list">
	          <li v-for="item in dateList"
	              v-text="item.value" 
	              track-by="$index" 
	              :class="{preMonth: item.previousMonth, nextMonth: item.nextMonth,
	                  selected: date === item.value && month === tmpMonth && item.currentMonth, invalid: validateDate(item)}"
	              @click="selectDate(item)">
	          </li>
	      </ul>
		</div>
	</div>

选择日期的方法就不细说了，在`selectYear`,`selectMonth`中对年份，月份变量赋值，再分别将`panelType`推向下一步就实现了日期选择功能。
不过在未选择完日期之前，你可能不希望当前年月的真实值发生变化，所以在这些方法中可先将选择的值赋给一个临时变量，等到`seletDate`的时候再一次性全部赋值。
    
    selectMonth (month) {
        if(this.validateMonth(month)){
            return
        }else{
            //临时变量
            this.tmpMonth = month
            //切换panel状态
            this.panelType = 'date'
        }
    },
    selectDate (date) {
        //validate logic above...
        //一次性全部赋值
        this.year = tmpYear
        this.month = tmpMonth
        this.date = date.value
        this.value = `${this.tmpYear}-${('0' + (this.month + 1)).slice(-2)}-${('0' + this.date).slice(-2)}`
        //选择完日期后，panel自动隐藏
        this.panelState = false
    }

最大/小时间限制
---
最大/小值是需要从父组件传递下来的，因此应该使用`props`，另外，这个值可以是字符串，也应该可以是变量(比如同时存在两个datepicker，第二个的日期不能比第一个大这种逻辑)，所以应该使用[Dynamically bind](http://vuejs.org.cn/api/#v-bind)的方式传值。

	<datepicker :value.sync="start"></datepicker>
    <!-- 现在min的值会随着start的变化而变化 -->
	<datepicker :value.sync="end" :min="start" ></datepicker>

增加了限制条件，对于不合法的日期，其按钮应该变为置灰状态，我用了比较时间戳的方式来判断日期是否合法，因为就算当前panel中的日期是跨年或是跨月的，通过日期构造函数创建时都会帮你转换成对应的合法值，省去很多判断的麻烦：
	
	new Date(2015, 0, 0).getTime() === new Date(2014, 11, 31).getTime() //true
	new Date(2015, 12, 0).getTime() === new Date(2016, 0, 0).getTime() //true
	
因此验证日期是否合法的函数是这样的:
	
	validateDate (date) {
      let mon = this.tmpMonth
      if(date.previousMonth){
          mon -= 1
      }else if(date.nextMonth){
          mon += 1
      }
      if(new Date(this.tmpYear, mon, date.value).getTime() >= new Date(this.minYear, this.minMonth - 1, this.minDate).getTime()
          && new Date(this.tmpYear, mon, date.value).getTime() <= new Date(this.maxYear, this.maxMonth - 1, this.maxDate).getTime()){
          return false
      }
      return true
  }

动态计算位置
---
当页面右侧有足够的空间显示时，datepicker的panel会定位为相对于父元素`left: 0`的位置，如果没有足够的空间，则应该置于`right: 0`的位置，这一点可以通过Vue提供的动态样式和样式对象来实现(动态class和动态style其实只是动态props的特例)，而计算位置的时刻，我放在了组件声明周期的`ready`周期中，因为这时组件已经插入到DOM树中，可以获取style进行动态计算：

	ready () {
		if(this.$el.parentNode.offsetWidth + this.$el.parentNode.offsetLeft - this.$el.offsetLeft <= 300){
            this.coordinates = {right: '0', top: `${window.getComputedStyle(this.$el.children[0]).offsetHeight + 4}px`}
        }else{
            this.coordinates = {left: '0', top: `${window.getComputedStyle(this.$el.children[0]).offsetHeight + 4}px`}
        }
	}
	<!-- template中对应的动态style -->
	<div :style="coordinates"></div>

为了panel的显隐可以平滑过渡，可以使用`transition`做过渡动画，这里我简单地通过一个0.2秒的透明度过渡让显隐更平滑。

	<div :style="this.coordinates" v-show="panelState" transition="toggle"></div>
	
	//less syntax
	.toggle{
        &-transition{
            transition: all ease .2s;
        }
        &-enter, &-leave{
            opacity: 0;
        }
    }


中英文切换
---
这里其实也很简单，这种多语言切换实质就是一个key根据不同的type而输出不同的value，所以使用filter可以很容易的实现它！比如渲染星期的列表:

	<ul class="weeks">
	     <li v-for="item in weekList" v-text="item | week language"></li>
	 </ul>
	 
	filters : {
		week (item, lang){
			switch (lang) {
	          case 'en':
	              return {0: 'Su', 1: 'Mo', 2: 'Tu', 3: 'We', 4: 'Th', 5: 'Fr', 6: 'Sa'}[item]
	          case 'ch':
	              return {0: '日', 1: '一', 2: '二', 3: '三', 4: '四', 5: '五', 6: '六'}[item]
	          default:
	              return item
	      }
		}
	}

多种使用方式
---
对于一个Vue组件，如果是使用`webpack + vue-loader`的`.vue`单文件写法，我希望这样使用：

	//App.vue
	<script>
		import datepicker from 'path/to/datepicker.vue'
		export default {
			components: { datepicker}
		}
	</script>
	
如果是直接在浏览器中使用，那么我希望`datepicker`这个组件是暴露在全局下的，可以这么使用：
	
	//index.html
	<html>
		<script src="path/to/vue.js"></script>
		<script src="path/to/datepicker.js"></script>
		<body>
			<div id="app"></div>
			<script>
				new Vue({
					el: '#app',
					components: { datepicker }
				})
			</script>
		</body>
	</html>

这里我选择了webpack作为打包工具，使用webpack的`output.library`和`output.linraryTarget`这两个属性就可以把你的bundle文件作为库文件打包。`library`定义了库的名字，`libraryTarget`定义了你想要打包的格式，具体可以看文档。我希望自己的库可以通过`datepicker`加载到，并且打包成`umd`格式，因此我的`webpack.config.js`是这样的:

	module.exports = {
        entry: './src/Datepicker.vue',
        output: {
            path: path.resolve(__dirname, './'),
            library: 'datepicker',
            filename: 'index.js',
            libraryTarget: 'umd'
        },
        module: {
            loaders: [
                {test: /\.vue$/, loaders: ['vue']},
                {test: /\.js$/, exclude: /node_modules/, loaders: ['babel']}
            ]
        }
    }

打包完成的模块就是一个`umd`格式的模块啦，可以在浏览器中直接使用，也可以配合require.js等模块加载器使用！

适配 Vue 2.x
---
Vue 2.0已经发布有段时间了，现在把之前的组件适配到Vue 2.0。迁移过程还是很顺利的，核心API改动不大，可以借助[vue-migration-helper](https://github.com/vuejs/vue-migration-helper)来找出废弃的API再逐步修改。这里只列举一些我需要修改的API。

## filter

2.0中的filter只能在mustache绑定中使用，如果想在指令式绑定中绑定过滤后的值，可以选择计算属性。我在月份和星期的显示中使用到了过滤器来过滤语言类型，但我之前是在指令式绑定中使用的filter，所以需要如下修改，：

    //修改前
    <div class="month-box" @click="chType('month')" v-text="tmpMonth + 1 | month language"></div>
    //修改后，filter传参的方式也变了，变成了函数调用的风格
    <div class="month-box" @click="chType('month')">{% raw %}{{tmpMonth + 1 | month(language)}}{% endraw %}</div>

## 移除`$index`和`$key`

这两个属性不会在`v-for`中被自动创建了，如需使用，要在`v-for`中自行声明：

    <li v-for="item in monthList" @click="selectMonth($index)"></li>
    //
    <li v-for="(item, index) in monthList" @click="selectMonth(index)"></li>

## `ready `生命周期移除

`ready`从生命周期钩子中移除了，迁移方法很简单，使用`mounted`和`this.$nextTick`来替换。

## `prop.sync`弃用

`prop`的`sync`弃用了，迁移方案是使用自定义事件，而且Datepicker这种input类型组件，可以使用[表单输入组件的自定义事件](http://vuejs.org/guide/components.html#Form-Input-Components-using-Custom-Events)作为替换方案。自定义组件也可以使用`v-model`指令了，但是必须满足两个条件：

1. 接收一个`value`的`prop`
2. 值发生变化时，触发一个`input`事件，传入新值。

所以Datepicker的使用方式也不是`<datepicker value.sync="now"></datepicker>`了，而是`<datepicker v-model="now"></datepicker>`。组件自身向父级传值的方式也不一样了：

    //1.x版本，设置了value的值会同步到父级
    this.value = `${this.tmpYear}-${('0' + (this.month + 1)).slice(-2)}-${('0' + this.date).slice(-2)}`
    
    //2.x版本，需要自己触发input事件，将新值作为参数传递回去
    let value = `${this.tmpYear}-${('0' + (this.month + 1)).slice(-2)}-${('0' + this.date).slice(-2)}`
    this.$emit('input', value)

总结
---
以上就是我在写这个datepicker时的大致思路，本身也是很简单的事情，没有处处展开来说，写在这里作为自己的一个总结，如果有刚开始使用Vue的同学也希望这篇文章可以在思路上帮助到你们:P，对于各位老鸟如果有什么指点的地方我也很感谢:D，那么差不多就这样，后面贴一些相关推荐阅读。



#### 推荐阅读
[高效地向数组中插值](http://www.jstips.co/zh_cn/insert-item-inside-an-array/)
[Vue.js-片段实例](http://vuejs.org.cn/guide/components.html#片断实例)
[Vue.js-动态绑定](http://vuejs.org.cn/api/#v-bind)
[Js日期对象基础](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Date)
[Webpack: export bundle as library](http://webpack.github.io/docs/configuration.html#output-library)
[UMD(universial Module Defination)](https://github.com/umdjs/umd)
