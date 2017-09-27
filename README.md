### Vue源码精读

1. [Vue.extend](#extend)



#### <a name="extend">1. Vue.extend</a>
----

  基础 Vue 构造器，可以创建一个“子类”。

  * `extend`函数（213行）：将所有属性混合至目标对象

  * `initExtend`函数（4355行）：构建Vue.extend函数并调用extend函数

  * `initGlobalAPI`函数（4593行）：构建Vue全局API，如extend/nextTick/set/delete/mixin等，并调用initExtend函数



