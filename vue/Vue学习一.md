---
title: Vue学习一
date: 2019-04-09 21:44:42
tags:
---
##Vue缩写语法：
v-bind缩写
```html
<!-- 完整语法 -->
<a v-bind:href="url">...</a>

<!-- 缩写 -->
<a :href="url">...</a>
```
v-on缩写
```html
<!-- 完整语法 -->
<a v-on:click="doSomething">...</a>

<!-- 缩写 -->
<a @click="doSomething">...</a>
```
## Vue组件的使用
组件是vue.js中最核心的功能，是整个框架设计最精彩的地方，也是最理解的。
###组件的用法
组件与创建vue类似，需要组测之后才能使用。
####全局注册
```js
    Vue.component('my-component',{
        template:"<h1>这是全局组件</h1>",
    })
```
渲染后结果
```
<div id="app">
    <h1>这是全局组件</h1>
</div>
```
####父子之间通信
在组件中使用props声明需要从父级接受的数据，props的值可以是两种：字符串或者对象。
```html
    <div id="app">
        <introduce :titles="msg"></introduce>
        <hr>
        <for-component :items="lessons"></for-component>
        <my-component></my-component>
    </div>
```
```js
<script>
    //父向子通信
    const introduce = {
        template:"<h1>{{titles}}</h1>",
        props:['titles']
    };
    const forComponent ={
            template:"<ul><li v-for='item in items'>{{item}}</li></ul>" ,
       // props:['items']
        props: {items:{
                type:Array,
                default:['java']
            }}
    };
    const app = new Vue({
        el:"#app",
        data:{
            msg:"大家好",
            lessons:["java",'php','python']
        },
        components:{
            introduce,forComponent
        }
    })
</script>
```
渲染后的结果
![父子通信结果](http://upload-images.jianshu.io/upload_images/7149586-5c8ac674dbc9f897.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
####子向父之间通信
子组件用$emit()来触发事件，示例代码如下：
```html
    <div id="app">
        <counter :num="count" @incr="add" @decr="reduce"></counter>
    </div>
```
```js
<script>
    const counter = {
        template:`
        <div>
         <button @click="handleAdd">+</button>
        <button @click="handleReduce">-</button>
        <h1>Num:{{num}}</h1>
        </div>`,
        props:['num'],
        methods: {
            handleAdd(){
                this.$emit('incr');
            },
            handleReduce(){
                this.$emit('decr');
            }
        }
    };
    const app = new Vue({
        el:"#app",
        data:{
            count:0,

        },
        components:{
            counter
        },
        methods:{
            add(){
                this.count++;
            },
            reduce(){
                this.count--;
            }
        }
    })
</script>    
```
const 是ec6中的语法，子向父通信较为麻烦。
渲染后的结果：
![父子通信结果](http://upload-images.jianshu.io/upload_images/7149586-a15535331b4dbab1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
