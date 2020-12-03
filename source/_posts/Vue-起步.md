layout: post
title: Vue 起步
date: 2016-07-03 21:58:52
tags: ['Js', 'Vue']
---

试着使用`Vue`写了一个cnode的webapp，使用了`Vue`，`vue-router`，`vue-resource`，`Mint-UI`，没有做发帖功能，也有一些地方参照了[Vue-cnodejs](https://github.com/shinygang/Vue-cnodejs)，因为最开始没有从整体上考虑，所以有些地方处理得不好，还有些bug，[repo在这](https://github.com/showonne/vue-cnode)。
<!-- more -->

#### 指令
vue也是通过指令来扩展html的，所以也有丰富的指令，而且都是编程中很常用的功能，大大提高了开发效率。在绑定变量时，使用{% raw %}`{{}}`{% endraw %}很常见的问题就是页面刚加载时{% raw %}`{{}}`{% endraw %}并没有被正确编译时会直接显示在页面上，会很难看，解决方式之一是使用`v-text`代替{% raw %}`{{}}`{% endraw %}，如果你真的很喜欢写{% raw %}`{{}}`{% endraw %}，那么可以在这个标签上加一个`v-cloak`属性，并且在你的css中加上一条:
    
    [v-cloak]{
      display: none;
    }

`v-cloak`这个属性在相关vue实例未完成编译前会一直保留，这样就可以解决这个问题了。（只是想到了提一下，其实我并没有这么写- -。）

####vue-router
使用`vue-router`做前端路由，引入`vue-router`后，实例化一个router实例，然后通过实例的`map`方法来管理路由关系，`vue-router`自身就可以做到组件按需加载，很方便。

    router.map({
      '/': {
        name: 'welcom',
        component: (resolve) => {
            require(['./components/Welcom.vue'], resolve)
        },
        auth: true
      }
    })

另外在路由跳转时进行权限验证也是很常用的功能，你可以在配置路由规则时通过`auth`属性来表明该路径是否需要验证，然后在`router.beforeEach`时候通过`transition.to.auth`判断。

    router.beforeEach((transition) => {
      if(transition.to.auth){
        //auth logic...
      }
      transition.next()
    })

另外在引入`vue-router`后，你可以在vue组件中通过`route`属性来使用vue-router。

    <script>
      export default {
        route: {
          data(transition){
            //...
          }
        }
      }
    </script>

#### vue-resource
`vue-resource`是用来创建ajax或者restful请求的插件，引入`vue-resource`后，组件就会有`$http`属性可以使用，`vue-resource`的请求返回结果是一个promise，而且在callback中的`this`还是绑定为vue实例的。

#### vue-cli
[vue-cli](https://github.com/vuejs/vue-cli)是一个用来快速构建vue应用的命令行工具，使用起来很方便。

#### SOS
大概写了差不多之后，我发现bundle达到了1.7m，`build`完的项目，其中一个vender也达到了900+kb，总共引入了`vue-resource`，`vue-router`,`Mint-UI`三个库，`Mint-UI`还是按需加载的，也没使用几个组件，不知道bundle为何会这么大，求帮助。

####总结
由于个人比较懒，同时也是一个拿来属性框架的练手项目，所以写了一阵子就不大想写了，没有写发帖功能，还有一些bug没有改掉，先这样，有时间的话可以再完善。
Vue的发展越来越好，不过相比React和Angular生态还是比较小，所以三方库会比较少，这是不利的地方，好处就是不会因可选择的太多而懵逼，好在Vue作者尤小右本人是非常高产的大神，把几乎必须的库都实现了，而且是官方出品，质量有保障。

