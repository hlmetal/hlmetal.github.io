---
layout: post
title:  "Vue.js基本知识"
date:   2019-03-18 11:25:35 +0200
categories: vue js
---

使用VUE和ELEMENT-UI开发内部数据查询系统时记录的有关Vue的部分知识。

## Vue.js简介
**Vue是一套构建用户界面的JS框架，只关注视图层，其核心概念是让用户不再操作DOM元素，可以更多关注业务逻辑。**

## Vue特性
#### 数据驱动
Vue使用了**MVVM**模型，实现数据与视图的双向绑定，通过视图中元素绑定的事件来修改数据，数据的变动来驱动视图的更新，无需关心具体如何操作DOM。
**MVVM模型**
* Model层： 数据源
* View层： 用户界面，即HTML页面
* ViewModel(VM)层： vue实例，是MVVM的核心，在Model层和View层之间的调度者

#### 组件化
组件化是Vue中的重要思想，它提供了一种抽象，也即将完整页面拆分成一个个组件，每个组件实现页面的一个功能，组件可以复用，降低了系统耦合性，同时方便组织管理代码，可维护性提高。组件的使用参考[Vue官方文档](https://cn.vuejs.org/guide/essentials/component-basics.html#defining-a-component)
##### 组件定义
``` js
    //全局组件定义
    //创建
    var comp = Vue.extend({templete:""}) //通过templete设置展示页面的HTML结构
    //注册
    Vue.component("myComp", comp)
    //使用
    <my-comp></my-comp> //直接以HTML标签的形式引入, 组件名称使用驼峰命名法则，引用时要改为小写并添加 - 

    //私有组件定义
    components:{ myComp:{templete:"<h1></h1>"}} 
    // 组件切换v-if/v-else
    <component :is="myComp"></component>
    //组件切换动画
    <transition mode="out-in"><component></component></transition>
```
##### 父子组件传值
子组件默认无法访问到父组件中的data和methods,父组件可以将需要传给子组件的数据通过属性绑定的方式传递。
``` js
    <parent><child :parentmsg="msg"></child></parent>
    //组件内部定义属性, 子组件中data和props中数据，前者为私有，后者为传递过来值，data数据可读可写，props只可读
    props:['parentmsg'] 
```

#### 轻量级
#### 虚拟DOM
虚拟DOM是一个JS对象，用于描述真实DOM
* 虚拟DOM中存在diff算法，是cpu密集型运算，占用内存较少，可以提高运行效率，并压缩运行时体积.
* 虚拟DOM可以跟踪状态变化，并通过比较前后两次状态差异更新真实的DOM,通过间接操作DOM，使得开发过程更加关注在业务代码的实现，而不需要关注如何操作DOM，从而提高开发效率。

## Vue实例生命周期
![](https://v2.cn.vuejs.org/images/lifecycle.png)
* beforeCreate执行时，data，methods中都还没初始化
* created执行时，data和methods已经被初始化完成
* beforeMount(模板挂载到页面上之前)，此时模板已经编译完成，但尚未渲染到页面中
* mounted此时用户已经可以看到渲染好的页面。它是实例创建期间的最后一个生命周期，实例已创建完毕，此时若没有其他操作，则此实例就在内存中不变
* beforeUpdate执行时页面没变，但数据已被更新，执行完后将数据挂载到VM
* updated执行时页面已被更新，VM的数据已被渲染到页面(beforeUpdate/updated:根据页面数据变化来执行0次-∞次)
* beforeDestroy执行时，vue就从运行到销毁阶段，实例身上的data和methods以及过滤器等都处于可用状态，此时还没有执行销毁过程
* destroyed执行时，此时组件已完全被销毁

## Vue扩展
#### vue-cli(vue脚手架) 
vue-cli是一个基于Webpack的Vue工具链,可快速创建vue工程化项目。安装vue-cli后使用*vue init webpack <projectname>*命令初始化项目，它会自动生产项目代码目录结构等，提高开发效率。Webpack是前端的一个项目构建工具，基于Node.js开发。
#### axios(http请求处理插件)
axios是一个基于promise的网络请求库，作用于node.js和浏览器中，在服务端它使用原生node.js http模块, 在浏览器则使用XMLHttpRequest。aixos可用于创建get、post、put等请求，处理并发请求。可以添加拦截器以拦截请求并处理相应逻辑。axios API使用参考[AXIOS官方文档](https://axios-http.com/docs/api_intro)
``` js
this.$http.get(url,[options]).then(successCallback,errorCallback)
//手动发起的post请求默认没有表单格式，有些服务器不能处理
//通过post方法的第三个数据设置 emulateJSON: true 可以解决
this.$http.post(url,[body],[options]).then(successCallback,errorCallback); //body不可省略

```
#### vue-router(vue路由)
Vue的路由管理器，和Vue.js的核心深度集成，非常方便的用于SPA应用程序的开发。所谓路由即根据不同的用户事件，显示不同的页面内容，建立起url和页面之间的映射关系。vue的单页面应用是基于路由和组件的，**路由用于设定访问路径，并将路径和组件映射起来**。传统的页面应用，是用一些超链接来实现页面切换和跳转的，而在vue-router单页面应用中，则是路径之间的切换，也即组件的切换。**主要通过url中的hash(#号)来实现不同页面之间的切换**。vue-router使用参考[Vue-router官方文档](https://router.vuejs.org/zh/guide/)
``` js
    //路由定义方式
    routers: [
        {path:"/login",component:login},  // path是路由地址，component是组件模板对象
        {path:"/",redirect:"/login"} // 如果请求的是根路径则重定向为login组件
    ]
    //路由嵌套
    {path: "/account",component:account,children:[{path:"login",component:login},{path:'',}]}
```
#### vuex(vue状态管理)
Vue的状态管理模式，用于管理共享数据，也即把组件共享状态抽取出来以一个全局单例模式管理。把共享的数据函数放进vuex中，任何组件都可以进行使用。vuex使用参考[vuex官方文档](https://vuex.vuejs.org/zh/installation.html)
**核心概念**
* state: 提供唯一的公共数据源，所有共享的数据都放到store的state进行储存
* mutations: 修改store中公共数据的唯一方法，包含两个参数，第一个参数为state是必须的，表示当前的state，第二个参数可选。类似于methods, mapMutations作为辅助函数可以将vuex中定义的方法映射为当前组件的methods。
* getters：计算属性，类似于computed。当Store数据源发生变化时，Getter的返回值会自动更新。也可通过mapGetters辅助函数， 将vuex中getters映射为当前组件的计算属性。
* actions: 与mutations类似，它是用来进行异步操作的，在actions中发异步请求获取数据后，调用mutations来修改数据
* modules: 用于拆分复杂业务的共享数据。将不同业务不同场景的数据放在不同模块中，结构清晰，方便开发维护。加上模块后，访问数据等需要额外添加模块名。

#### element(基于vue的UI)
基于vue.js的一套桌面端UI框架，element-ui使用简单，界面清晰，非常适合用于CMS。element组件使用参考[element官方文档](https://element.eleme.io/#/zh-CN)

#### *TO BE CONTIUED* 

## Vue常用属性
1. el 指示vue编译器从什么地方开始解析vue的语法。表示一个vue对象需要挂载到哪一个html对象上面，值为那个html对象的id
    ``` html
    <div id="app">
    </div>
    <script>
        const app = new Vue({
            el: '#app',
            data: {

            }
        })
    ```
2. data 存储数据，从服务器请求或者固定
    ``` js
    data: {
        msg: "Hello World",
        firstName: J,
        lastName: M
    }
    ```
3. template 用于设置模板，替换页面元素
    ``` js
    <template>
        <div id="tab">
            <slot></slot>
        </div>
    </template>
    ```
4. methods 方法，即用于书写业务逻辑
    ``` js
    methods: {
        getFullName: function() {
            console.log('getFullName');
            return this.firstName + ' ' + this.lastName
        }
    }
    ```
5. computed 计算属性，根据所依赖的数据动态显示新的计算结果，有缓存。
    ``` js
    computed: {
        fullName: function() {
            console.log('getFullName');
            return this.firstName + ' ' + this.lastName
        }
    }
    ```
6. watch 监听属性，监听data数据变化后触发对应函数,用于数据变化时执行异步或开销较大的操作
    ``` js
    watch: {
        watchValue: function(newValue, oldValue) {
            // logic
        }
    }
    ```

## Vue常用指令
1. v-text 更新元素的文本内容 
    ``` html
    <span v-text="msg"></span>
    ```
2. v-html 输出为HTML格式
    ``` html
    <div v-html="html"></div>
    ```
3. v-cloak 解决插值表达式闪烁问题，与css搭配使用
    ``` html
    //css
    [v-cloak] {
        display: none;
    }
    //html
    <div v-cloak>
        {{msg}}
    </div>
    ```
4. v-bind 绑定属性指令
    ``` js
     <div v-bind:title='mytitle'></div>
    ```
5. v-on 绑定事件指令, 可添加相应修饰符以达到不同效果
    ``` js
    <div v-on:click="post"></div>
    //...
    methods: { 
        post:function(){
            //logic
        }
    }
    // 修饰符
    //once 表示只触发一次
    <div v-on:click.once="post"></div>
    ```
6. v-model 双向数据绑定，只用于表单元素
    ``` js
    <input v-model="name" placeholder="input name">
    <p>name is: {{ name }}</p>
    ```
7. v-for 遍历数组，数字，对象等
    ``` html
    <div v-for="(item, index) in items"></div>
    <div v-for="item in items" :key="item.id">
        {{ item.text }}
    </div>
    ```
8. v-if/v-show 有条件地渲染元素，前者每次都会重新创建或删除元素，后者只会切换元素的display样式
    ``` js
    //v-if 条件为真时才渲染元素
    <h1 v-if="ok">Vue is ok!</h1>
    //v-show总是会渲染元素
     <h1 v-if="ok">Vue is ok!</h1>
    ```

## Vue常用API
1. filter 可被用作一些常见的文本格式化,只用于mustache插值表达式和v-bind表达式中，由管道符指示。
    ``` js
    //全局过滤器
    Vue.filter(filtername, function(data,parten) {
        //logic
    })
    //私有过滤器
    filters: {
        filtername, function(data,parten) {
            //logic
    }}
    ```
2. directive 自定义指令
    ``` js
    //全局自定义指令
    Vue.directive("focus", {
        //bind 是指只执行一次
        bind:function(el) { //el是绑定了指令的元素

        },
        //被绑定元素插入父节点时调用
        inserted:function(el) { //el是绑定了指令的元素

        },
        //当VNode更新时，触发，可能触发多次
        update:function(el) { //el是绑定了指令的元素

        }
        //...
    })
    ```
3. *TO BE CONTIUED*