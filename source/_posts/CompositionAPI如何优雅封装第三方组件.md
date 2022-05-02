---
title: Composition API如何优雅封装第三方组件
date: 2022-04-11 01:03:00
tags: vue3
---

# 前言✨

对于第三方组件，如何在保持`第三方组件`原有功能（属性`props`、事件`events`、插槽`slots`、方法`methods`）的基础上，优雅地进行功能的扩展了？

以[`Element Plus`](https://element-plus.gitee.io/zh-CN/)的[`el-input`](https://element-plus.gitee.io/zh-CN/component/input.html#input-%E5%B1%9E%E6%80%A7)为例：

很有可能你以前是这样玩的，封装一个MyInput组件，把要使用的属性`props`、事件`events`和插槽`slots`、方法`methods`根据自己的需要再写一遍：

```js
// MyInput.vue
<template>
  <div class="my-input">
    <el-input v-model="inputVal" :clearable="clearable" @clear="clear">
    <template #prefix>
      <slot name="prefix"></slot>
    </template>
      <template #suffix>
      <slot name="suffix"></slot>
    </template>
    </el-input>
  </div>
</template>
<script setup>
import { computed } from 'vue'
const props = defineProps({
  modelValue: {
    type: String,
    default: ''
  },
  clearable: {
    type: Boolean,
    default: false
  }
})
const emits = defineEmits(['update:modelValue', 'clear'])

const inputVal = computed({
  get: () => props.modelValue,
  set: (val) => {
    emits('update:modelValue', val)
  }
})

const clear = () => {
  emits('clear')
}
</script>
```
可过一段时间后，需求变更，又要在MyInput组件上添加el-input组件的其它功能，可el-input组件总共有20个多属性，5个事件，4个插槽，那该怎么办呢，难道一个个传进去，这样不仅繁琐而且可读性差。

在Vue2中，我们可以这样处理，点击此处查看[`红尘炼心`大神的文章](https://juejin.cn/post/6943534547501858824#heading-4)。

此文诣在帮助大家做一个知识的迁移，探究如何使用`Vue3 CompositionAPI`优雅地封装第三方组件~

# 一、对于第三方组件的属性`props`、事件`events`
在`Vue2`中
- `$attrs`: 包含了父作用域中不作为 `prop` 被识别 (且获取) 的 attribute 绑定 (class 和 style 除外)。当一个组件没有声明任何`prop` 时，这里会包含所有父作用域的绑定 (class 和 style 除外)，并且可以通过 `v-bind="$attrs"` 传入内部组件
- `$listeners`:包含了父作用域中的 (不含 `.native` 修饰器的) `v-on` 事件监听器。它可以通过 `v-on="$listeners"` 传入内部组件

而在`Vue3`中
- `$attrs`:包含了父作用域中不作为组件 [props](https://v3.cn.vuejs.org/api/options-data.html#props) 或[自定义事件](https://v3.cn.vuejs.org/api/options-data.html#emits)的 attribute 绑定和事件（包括 `class` 和 `style`和`自定义事件`），同时可以通过 `v-bind="$attrs"` 传入内部组件。
- `$listeners` 对象在 Vue 3 中已被移除。事件监听器现在是 `$attrs` 的一部分。
- 在 `<script setup>`中辅助函数`useAttrs`可以获取到`$attrs`。
```js
//MyInput.vue 
<template>
  <div class="my-input">
    <el-input v-bind="attrs"></el-input>
  </div>
</template>
<script setup>
import { useAttrs } from 'vue'
const attrs = useAttrs()
</script>
```
当然，这样还不够。光这样写，我们绑定的属性（包括 `class` 和 `style`）同时会在根元素（上面的例子是`class="my-input"`的Dom节点）上起作用。要阻止这个默认行为，我们需要设置`inheritAttrs`为`false`。

下面我们来看看Vue3文档对`inheritAttrs`的解释
> 默认情况下父作用域的不被认作 props 的 attribute 绑定 (attribute bindings) 将会“回退”且作为普通的 HTML attribute 应用在子组件的根元素上。当撰写包裹一个目标元素或另一个组件的组件时，这可能不会总是符合预期行为。通过设置 `inheritAttrs` 到 `false`，这些默认行为将会被去掉。而通过实例 property `$attrs` 可以让这些 attribute 生效，且可以通过 `v-bind` 显性的绑定到非根元素上。

于是，我们对于第三方组件的属性`props`、事件`events`处理，可以写成如下代码：
```js
// MyInput.vue 
<template>
  <div class="my-input">
    <el-input v-bind="attrs"></el-input>
  </div>
</template>
<script>
export default {
  name: 'MyInput',
  inheritAttrs: false
}
</script>
<script setup>
import { useAttrs } from 'vue'
const attrs = useAttrs()
</script>
```

# 二、对于第三方组件的插槽`slots`
`Vue3`中
- `$slots`：我们可以通过其拿到父组件传入的插槽
- `Vue3`中移除了`$scopedSlots`，所有插槽都通过 `$slots` 作为函数暴露
- 在 `<script setup>`中辅助函数`useSlots`可以获取到`$slots`。

基于以上几点，**如果我们对于第三方组件的封装没有增加额外的插槽，且第三方组件的插槽处于同一个dom节点之中**，我们也有一种取巧的封装方式😎, 通过遍历`$slots`拿到插槽的name，动态添加子组件的插槽：
```js
//MyInput.vue 
<template>
  <div class="my-input">
    <el-input v-bind="attrs">
      <template v-for="k in Object.keys(slots)" #[k] :key="k">
        <slot :name="k"></slot>
      </template>
    </el-input>
  </div>
</template>
<script>
export default {
  name: 'MyInput',
  inheritAttrs: false
}
</script>
<script setup>
import { useAttrs, useSlots } from 'vue'
const attrs = useAttrs()
const slots = useSlots()
</script>
```
如果不满足以上条件的话，咱还得老老实实在子组件中手动添加需要的第三方组件的插槽~

# 三、对于第三方组件的方法`methods`

对于第三方组件的方法，我们通过`ref`来实现。首先在MyInput组件中的el-input组件上添加一个`ref="elInputRef"`属性，然后通过`defineExpose`把`elInputRef`暴露出去给父组件调用。

子组件：`MyInput.vue`

```js
// MyInput.vue 
<template>
  <div class="my-input">
    <el-input v-bind="attrs" ref="elInputRef">
      <template v-for="k in Object.keys(slots)" #[k] :key="k">
        <slot :name="k"></slot>
      </template>
    </el-input>
  </div>
</template>
<script>
export default {
  name: 'MyInput',
  inheritAttrs: false
}
</script>
<script setup>
import { useAttrs, useSlots } from 'vue'
const attrs = useAttrs()
const slots = useSlots()

const elInputRef = ref(null)
defineExpose({
  elInputRef  // <script setup>的组件里的属性默认是关闭的，需通过defineExpose暴露出去才能被调用
})
</script>
```
父页面：`Index.vue`的调用代码如下：

```js
// Index.vue 
<template>
  <my-input v-model='input' ref="myInput">
    <template #prefix>姓名</template>
  </my-input>
</template>
<script setup>
import MyInput from './components/MyInput.vue'
import { ref, onMounted } from 'vue'
const input = ref('')

const myInput = ref(null) // 组件实例
onMounted(()=> {
  myInput.value.elInputRef.focus() // 初始化时调用elInputRef实例的focus方法
})
</script>
```
🎉🎉🎉🎉🎉🎉


