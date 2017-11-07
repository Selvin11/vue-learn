# Vue-plan
以Vue为例，学习MVVM及其它框架模型


1. [Vue实例对象暴露的属性和方法](#1)
2. [模板语法](#2)
3. [计算属性（computed）和 Methods](#3)
4. [组件](#4)
5. [slot内容分发](#5)
6. [单文件组件](#6)
7. [Virtual DOM 虚拟DOM详解](#7)
8. [bus.js组件间的简单状态管理](#8)

<h2 id="1">1. Vue实例对象暴露的属性和方法</h2>

* data属性，每个 Vue 实例都会代理其 data 对象里所有的属性，只有这些被代理的属性是响应的

* Vue中的数据绑定及响应：HTML标签中的属性值，以及其`value`值都是根据Vue实例对象中的data对象进行响应的

    ```javascript
        var data = { a: 1 }
        var vm = new Vue({
          data: data
        })
        vm.a === data.a // -> true
        // 设置属性也会影响到原始数据
        vm.a = 2
        data.a // -> 2
        // ... 反之亦然
        data.a = 3
        vm.a // -> 3
    ```

* Vue 实例暴露了一些有用的实例属性与方法，这些属性与方法都有前缀 $，以便与代理的 data 属性区分
    ```javascript 
        var data = { a: 1 }
        var vm = new Vue({
          el: '#example',
          data: data
        })
        vm.$data === data // -> true
        vm.$el === document.getElementById('example') // -> true
        // $watch 是一个实例方法
        vm.$watch('a', function (newVal, oldVal) {
          // 这个回调将在 `vm.a`  改变后调用
        })
    ```

<h2 id="2">2. 模板语法</h2>

* HTML中的**属性**不能使用Mustache语法（双大括号{{}}），因此采用`v-*:[attr]="value"`

    ```javascript
    // 如需在html标签中的属性传入data数据，如href,style,src...都需要用v-bind:等方式    
    //下面两种是等价的
        <a v-bind:href="url"></a>
        <a href={{url}}></a>
       	<img v-bind:src="url" /> ==> <img src="url" />
        
    ```
* HTML中的特性不识别大小写，驼峰写法会被转为短横线拼接，因此组件中的`props`一般采用kebab-case (短横线隔开式) 命名

<h2 id="3">3. 计算属性（computed）和 Methods</h2>
* 不宜将过重的逻辑放置模板中，这时可以在实例中设置相应的计算属性，提供逻辑操作的方法；

* 虽然在`Methods`中定义相同的函数也可以达到效果，但最大的区别是**计算属性是基于它的依赖缓存，只要它相关的对象数据没有改变，则一直返回之前的计算结果，并不再次执行**


<h2 id="4">4. 组件</h2>

* 组件注册(**子组件只能在父组件的template中使用**)
    ```javascript

      <div id="example"> // 挂载点 my-counter的父组件
        <h1>父组件标题</h1> // 挂载点的内容
        <my-counter></my-counter> // 子组件及其它包含的模板
      </div>

      // 全局注册
      var data = { counter: 0 }

      // 父组件编译作用域（容器）<my-counter></my-counter> ，此容器的属性和方法均来自外层挂载元素'#example'
      
      // 子组件模板（template）中调用的属性和方法均来自一下容器内部

      Vue.component('my-component', {
        template: '<h2>子组件标题</h2>', 
        //此处的data为引用
        data: function () {
           return data
         }
      })

      // 局部注册 对象的属性值只能是数字，字符串，变量
      var Child = {
        template: '<div>A custom component!</div>'
      };

      new Vue({
        // ...
        components: {
          // <my-component> 将只在父模板可用
          'my-component': Child
        }
      })
    ```
* 组件中的`data`必须是函数，当前组件内调用的数据，均为当前组件的编译作用域提供的`data`对象

* 组件的模板是在其作用域内编译的，那么组件选项对象中的数据也应该是在组件模板中使用的。

* 父子组件间的通信

  组件中的数据传递（父子），父组件通过`props`传递数据给子组件，子组件通过`events`发送消息给父组件

  * `props`：当引用组件时，如`<hello :my-message="message"></hello>`，此时`hello`为组件模板的父级

      ```javascript
        // 在另一个组件中引用hello组件
        // 这里<hello>作为父级将prop为'my-message'的值传递给组件模板
        // vue在标签中模板引擎解析引号中的数值机制为：
            1. 如果双引号中仅有数值，无大括号，则在data中寻找其对应的值
            2. 如果双引号中有大括号，则将其视为一段js代码进行执行
            3. 标签中不存在双大括号引用，双大括号一般出现在标签中包含的纯内容，解析变量，等同于一般的模板引擎机制
        <div class="app">
          <hello :my-message="message"></hello>
        </div>
      ```


        export default {
                name: 'app',
                data () {
                    return {
                        message: "hello world"
                    }
                }
            }


        // hello组件模板

        <p>{{myMessage}}</p>

        <script>
          export default {
                  name: 'hello',
                  props:{
                      myMessage:{
                          type: String,
                          default: ''
                      }
                  },
                  data () {
                      return {
    
                      }
                  }
              }
        </script>
      ```

* 非父子组件间的通信

  ```javascript 
      var bus = new Vue()

      // 触发组件 A 中的事件，即在A组件方法中出发B组件中定义的事件，并传递数据
      bus.$emit('id-selected', 1)

      // 在组件 B 创建的钩子中监听事件 ，即在B组件中定义监听事件
      bus.$on('id-selected', function (id) {
        // ...
      })

  ```

<h2 id="5">5. slot</h2>

* 本质：组件（如<my-component></my-component>中添加其它HTML内容时，编译时直接被忽略，不会输出），slot的出现就是处理组件间包含的内容的显示，即内容插槽，这些内容在编译后会被插入slot中

* slot：为了让组件可以组合，我们需要一种方式来混合父组件的内容与子组件自己的模板(并进行HTML的转译)。这个过程被称为**内容分发**
* 最初在 <slot> 标签中的任何内容都被视为备用内容,备用内容在子组件的作用域内编译，并且只有在宿主元素(或父组件内容)为空，且没有要插入的内容时才显示备用内容。
* 单个slot：除非子组件模板包含至少一个 <slot> 插口，否则父组件的内容将会被丢弃。
* 命名后的slot：

    ```javascript
        // app为挂载元素
        <div id="app">
          <app-layout>
            // 编译时将此内容插入头部插槽
            <h1 slot="header">这里可能是一个页面标题</h1>

            <p>主要内容的一个段落。</p>
            <p>另一个主要段落。</p>
            
            // 编译时将此内容插入底部插槽
            <p slot="footer">这里有一些联系信息</p>
          </app-layout>
        </div>

        <template id="app-layout">
          <div class="container">
            <header>
              <!-- 头部内容插槽 -->
              <slot name="header"></slot>
            </header>
            <main>
              <!-- 中部无名内容插槽 -->
              <slot></slot>
            </main>
            <footer>
              <!-- 底部内容插槽 -->
              <slot name="footer"></slot>
            </footer>
          </div>
        </template>
        
        Vue.component('app-layout',{
          template:"#app-layout"
        });

        var app = new Vue({
          el: '#app'
        });
    ```

* 作用域插槽
    ```javascript
        <div id="app">
            <my-component>
                <child></child>
            </my-component>
        </div>
        <!-- 父组件模板 -->
        <template id="my-component">
            <div class="parent">
                <span>hello from parent</span>
                <child>
                    <!-- scope接受子组件传递过来的text数据值 -->
                    <template scope="props">
                        <span>{{ props.text }}</span>
                    </template>
                </child>
            </div>
        </template>
        <!-- 子组件模板 -->
        <template id="child">
            <div class="child">
                <!-- 作用域插槽 -->
                <slot text="hello from child"></slot>
            </div>
        </template>

        <script>
        Vue.component('my-component', {
            template: '#my-component'
        });
    ```


        Vue.component('child', {
            template: '#child'
        });
    
        var app = new Vue({
            el: '#app',
        });
        </script>
    ​```

<h2 id="6">6. 单文件组件</h2>

* 单文件组件与原生组件的区别

    ```javascript
      // 原始写法
      Vue.component('hello',{
        template:
                  `
                    <div class="hello">
                      <h1 @click="add()">{{ msg }}</h1>
                      <h2>{{count}}</h2>
                    </div>
                  `,
        data: function(){
            return {
              msg: 'Hello Word'
            }
        },
        computed:{

        },
        methods:{

        }
      })

      // 单文件写法
      <template>
        <div class="hello">
          <h1 @click="add()">{{ msg }}</h1>
          <h2>{{count}}</h2>
        </div>
      </template>

      <script>
      export default {
        name: 'hello',
        data () {
          return {
            msg: 'Hello Word'
          }
        },
        computed:{
          count(){
            return this.$store.state.count
          }
        },
        methods:{
          add(){
            this.$store.commit('increment')
          }
        }
      }
      </script>
    ```

<h2 id="7">7. Virtual DOM 虚拟DOM详解</h2>

* 从实际问题出发：在对表格数据排序时，我们会根据ID,Size,Score...等等属性进行升降排序

> 一般做法：在DOM中监听点击事件，获取升/降，在JS中根据属性存放不同的升/降数据，然后判断事件获取相应的数据，替换DOM中的内容

* 从上面这个方法中，不难发现，一旦属性过多，需要维护的数据将冗余，数据间的状态关系管理将变得复杂。而且，每次的排序操作都会导致DOM的重绘渲染，性能受到极大的影响。因此前端思辨，想借鉴流行的架构模式（MVC等），但需要操作的DOM还是存在，这时MVVC应运而生，将视图（View）和视图数据进行绑定，一旦数据发生改变将改变视图，而且视图改变不会导致整个DOM的重绘渲染，极大的简化了状态关系管理的逻辑以及提升了渲染性能。

* 虚拟DOM 本质上就是在 JS 和 DOM 之间做了一个缓存（渲染前的JS对象）。

  * 通过JS对象存储DOM元素

    ```javascript
        var element = {
          tagName: 'ul', // 节点标签名
          props: { // DOM的属性，用一个对象存储键值对
            id: 'list'
          },
          children: [ // 该节点的子节点
            {tagName: 'li', props: {class: 'item'}, children: ["Item 1"]},
            {tagName: 'li', props: {class: 'item'}, children: ["Item 2"]},
            {tagName: 'li', props: {class: 'item'}, children: ["Item 3"]},
          ]
        }

        作者：戴嘉华
        链接：https://www.zhihu.com/question/29504639/answer/73607810
        来源：知乎
        著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。
    ```

  * 监听事件，改变状态，从而重新渲染此JS对象，将渲染后的对象和渲染前的对象进行Diff对比，最后才操作DOM，只改变不同的地方

<h2 id="8">8. bus.js组件间的简单状态管理</h2>

* bus.js的原理剖析：vue文档中两行代码便演示了，这次我们放在单文件中理解

  ```javascript
  // bus.js 文件
  export default new Vue({
    data(){
      return{
        count: 1
      }
    },
    methods:{
      funcA(){
        console.log(this.count)
      }
    }
  })

  // hello.vue Hello单文件组件中
  <template></template>
  <script>
  import bus from './bus.js'
  export default{
    created(){
      bus.$on('handleHello',function(str){
        alert(str)
      })
    },
    methods:{
      getBusData(){
        console.log(bus.count) // 1
      }
    }
  }
  </script>
  // app.vue 文件中
  <template>
  	<h1 @click="emitHello">点击触发hello.vue中的事件</h1>
  	<hello></hello>
  </template>
  <script>
  import bus from './bus.js'
  import hello from './hello.vue'
  export default{
   components:{
     hello
   },
   methods:{
      emitHello(){
        bus.$emit('handleHello','app.vue触发Hello组件中的方法')
      }
    }
  }
  </script>

  // hello.vue中将handleHello方法挂载在bus实例上，
  // app.vue通过点击触发该事件
  // 至此就完成了不同组件之间共有状态，
  // 以及A组件触发B组件执行操作
  ```

