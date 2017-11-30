### Vue源码精读

![vue流程图](./vue流程图.png)

1. [Vue.extend](#extend)
2. [Vue生命周期顺序](#life-cycle)
3. [vm.$options](#options)

#### <a name="extend">1. Vue.extend</a>
----

  基础 Vue 构造器，可以创建一个“子类”。

  * `extend`函数（213行）：将所有属性混合至目标对象

  * `initExtend`函数（4355行）：构建Vue.extend函数并调用extend函数

  * `initGlobalAPI`函数（4593行）：构建Vue全局API，如extend/nextTick/set/delete/mixin等，并调用initExtend函数


#### <a name="life-cycle">2. Vue生命周期顺序</a>
----

  包含`keep-alive`动态组建被调用的情况，进入组件时的调用顺序：

  ```javascript
  // App.vue
  <template>
    <div id="app">
      <keep-alive><router-view></router-view></keep-alive>
    </div>
  </template>
  // Hello.vue
  <template></template>
  <script>
    export default {
      name: 'hello',
      data () {
        return {
          msg: 'Welcome to Your Vue.js App'
        }
      },
      created: function () {
        alert('实例已经创建')
      },
      beforeMount: function () {
        alert('实例挂载之前')
      },
      mounted: function () {
        alert('实例挂载到根元素 #app上')
      },
      beforeUpdate: function () {
        alert('数据更新之前')
      },
      updated: function () {
        alert('数据更新之后')
      },
      activated: function () {
        alert('keep-alive 组件激活时调用')
      },
      deactivated: function () {
        alert('keep-alive 组件停用时调用')
      },
      beforeDestroy: function () {
        alert('实例销毁之前调用')
      },
      destroyed: function () {
        alert('实例销毁之后调用')
      }
    }
    </script>
  ```
  * beforeCreated
  * created
  * beforeMount
  * Mounted
  * activated 此钩子函数是在mounted之后调用的


#### <a name="options">3. vm.$options</a>
----

  用于当前 Vue 实例的初始化选项。需要在选项中包含自定义属性时会有用处：

  ```javascript
  new Vue({
    customOption: 'foo',
    created: function () {
      console.log(this.$options.customOption) // => 'foo'
    }
  })
  ```