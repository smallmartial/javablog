###自定义指令
```js
//全局注册
        Vue.directive('focus',{
        })
//局部注册
        var app = new Vue({
            el:"#pp"
            directives:{
              forcus:{
              
              }
            }
        }) 
```
写法与组建写法类似，方法名有component改变为directive。
自定义指令是有几个钩子函数组成的，每个都是可选项。
- bind
- inserted
- update
- componentUpdated
- unbind
每个钩子函数都有几个参数可用。
 - el 指令绑定的元素，可以直接用来操作DOM
- binding 一个对象，包含以下属性。
  - name
  - value
  - oldValue
  - expression
  - arg
  - modifiers
- vnode Vue编译生成虚拟节点
- oldVnode 上一个虚拟节点仅在update 和componentUpdated钩子中可用。

##Render函数
>Vue 推荐在绝大多数情况下使用 template 来创建你的 HTML。然而在一些场景中，你真的需要 JavaScript 的完全编程的能力，这时你可以用 render 函数，它比 template 更接近编译器。
###createElement用法
```js
// @returns {VNode}
createElement(
  // {String | Object | Function}
  // 一个 HTML 标签字符串，组件选项对象，或者
  // 解析上述任何一种的一个 async 异步函数。必需参数。
  'div',

  // {Object}
  // 一个包含模板相关属性的数据对象
  // 你可以在 template 中使用这些特性。可选参数。
  {

  },

  // {String | Array}
  // 子虚拟节点 (VNodes)，由 `createElement()` 构建而成，
  // 也可以使用字符串来生成“文本虚拟节点”。可选参数。
  [
    '先写一些文字',
    createElement('h1', '一则头条'),
    createElement(MyComponent, {
      props: {
        someProp: 'foobar'
      }
    })
  ]
)
```
第一个参数必选，可以是一个htm标签,也可以是一个组件或者函数，第二个是可选参数对象数据，在template中使用。第三个是子节点，也是可选参数。
###对于第二个数据对象，具有选项如下：
```js
{
  // 和`v-bind:class`一样的 API
  // 接收一个字符串、对象或字符串和对象组成的数组
  'class': {
    foo: true,
    bar: false
  },
  // 和`v-bind:style`一样的 API
  // 接收一个字符串、对象或对象组成的数组
  style: {
    color: 'red',
    fontSize: '14px'
  },
  // 普通的 HTML 特性
  attrs: {
    id: 'foo'
  },
  // 组件 props
  props: {
    myProp: 'bar'
  },
  // DOM 属性
  domProps: {
    innerHTML: 'baz'
  },
  // 事件监听器基于 `on`
  // 所以不再支持如 `v-on:keyup.enter` 修饰器
  // 需要手动匹配 keyCode。
  on: {
    click: this.clickHandler
  },
  // 仅用于组件，用于监听原生事件，而不是组件内部使用
  // `vm.$emit` 触发的事件。
  nativeOn: {
    click: this.nativeClickHandler
  },
  // 自定义指令。注意，你无法对 `binding` 中的 `oldValue`
  // 赋值，因为 Vue 已经自动为你进行了同步。
  directives: [
    {
      name: 'my-custom-directive',
      value: '2',
      expression: '1 + 1',
      arg: 'foo',
      modifiers: {
        bar: true
      }
    }
  ],
  // 作用域插槽格式
  // { name: props => VNode | Array<VNode> }
  scopedSlots: {
    default: props => createElement('span', props.text)
  },
  // 如果组件是其他组件的子组件，需为插槽指定名称
  slot: 'name-of-slot',
  // 其他特殊顶层属性
  key: 'myKey',
  ref: 'myRef',
  // 如果你在渲染函数中向多个元素都应用了相同的 ref 名，
  // 那么 `$refs.myRef` 会变成一个数组。
  refInFor: true
}
```
###template与Render对比：
####template用法：
```html
<script src="https://cdn.jsdelivr.net/npm/vue@2.6.10/dist/vue.js"></script>
<body>
    <div id='app'>
        <ele></ele> 
    </div>
</body>
<script>
    Vue.component('ele',{
        template: ` <div id="element" :class="{show:show}" @click="handleClick">文本内容</div>`,
        data: function(){
            return{
                show:true
            }
        },
        methods: {
            handleClick:function(){
                console.log('clicked!');
            }
        },
    });
    var app = new Vue({
        el:'#app'
    })
</script>
```
####Render用法：
```html
<body>
    <div id='app'>
        <ele></ele> 
    </div>
</body>
<script>
    Vue.component('ele',{
        render:function(createElement){
            return createElement(
                'div',{
                    class:{'show':this.show},
                    attrs:{id:'element'},
                    on:{click:this.handleClick}
                },
                '文本内容'
            )
        },
        data: function(){
            return{
                show:true
            }
        },
        methods: {
            handleClick:function(){
                console.log('clicked!');
            }
        },
    });
    var app = new Vue({
        el:'#app'
    })
</SCript>
```
###运行结果
![运行结果](https://upload-images.jianshu.io/upload_images/7149586-6f73e2c6e5bab72c.PNG?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
>针对此例，template的写法比Render写法简洁，所以要在合适的场景使用Render函数，否则只会增加负担。
