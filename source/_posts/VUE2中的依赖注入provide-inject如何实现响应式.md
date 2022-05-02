---
title: VUE2中的依赖注入provide/inject如何实现响应式？
date: 2022-01-10 02:10:09
tags: vue2
---

# 前言
我们知道，使用`provide/inject`这一对API，可以**暴露出父组件的数据和方法供所有后代组件**使用。这个特性有两个部分：父组件有一个 `provide` 选项来提供数据，子组件有一个 `inject` 选项来开始使用这些数据。

现有父组件和子组件如下

`父组件`：
```
<template>
  <div class="parent">
    <div>父组件：{{ this.name }}</div>
    <button @click="changeIdol">改变偶像</button>
    <hr>
    <child></child>
  </div>
</template>

<script>
import Child from './Child'
export default {
  name: 'Parent',
  components: {
    Child
  },
  provide() {
    return {
      parentName: this.name
    }
  },
  data() {
    return {
      name: '小松菜奈'
    }
  },
  methods: {
    changeIdol() {
      this.name = '新垣结衣'
    }
  }
}
</script>
```
`子组件`：
```
<template>
  <div class="child">
    <!-- 模板中使用时，this不能省 -->
    <div>子组件：{{ this.parentName }}</div>
  </div>
</template>

<script>
  export default {
    name: 'Child',
    inject: ['parentName'],
    data() {
      return {
        
      }
    }
  }
</script>
```
如果父组件直接注入数据，不做任何处理，当我改变父组件的数据时，子组件的数据不变，可见响应式消失了。


![未标题-1.gif](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b097361db4c44548b59f3c0ba64956aa~tplv-k3u1fbpfcp-watermark.image?)

那如何让注入的数据保持响应式了？

# 解决方案
### 1.传入响应式数据✨
vue2虽然不能像vue3那样创造响应式数据ref、reactive，但vue2的`this`就是个响应式数据。如果我们直接在父元素中提供依赖`this`，那么后代组件就能获得响应式。

`父组件`：
```
provide() {
  return {
    parent: this
  }
}
```
`子组件`：
```
<template>
  <div class="child">
    <!-- 模板中使用时，this不能省 -->
    <div>子组件：{{ this.parent.name }}</div>
    <button @click="print">打印响应式数据</button>
  </div>
</template>

<script>
  export default {
    name: 'Child',
    inject: ['parent'],
    methods: {
      print() {
        console.log(this.parent.name) // 方法中使用
      }
    }
  }
</script>
```
### 2.以函数式的方式传入✨
父组件中以函数式的方式提供依赖，后代组件中采用函数的方式调用。

`父组件`：
```
provide() {
  return {
    parentName: () => this.name
  }
}
```
`子组件`：
```
<template>
  <div class="child">
    <!-- 模板中使用时，this不能省 -->
    <div>子组件：{{ this.parentName() }}</div>
    <button @click="print">打印响应式数据</button>
  </div>
</template>

<script>
  export default {
    name: 'Child',
    inject: ['parentName'],
    methods: {
      print() {
        console.log(this.parentName()) // 方法中使用时，以函数的方式调用
      }
    }
  }
</script>
```
--- 完结 --- 🎉🎉🎉
