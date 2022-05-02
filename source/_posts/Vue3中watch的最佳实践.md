---
title: Vue3中watch的最佳实践
date: 2022-05-03 01:26:55
tags: vue3
---

# 前言🌟

本文诣在帮大家彻底梳理`Vue3`中`watch`的用法。

首先，介绍`Vue3`的`watch` API的基本使用，然后介绍了`watch`在侦听单个数据源、多个数据源的注意事项，最后通过对比得出watch侦听`引用类型数据源`的最佳实践。

# 一、API介绍

```js
watch(WatcherSource, Callback, [WatchOptions])

type WatcherSource<T> = Ref<T> | (() => T) 

interface WatchOptions extends WatchEffectOptions {
    deep?: boolean // 默认：false 
    immediate?: boolean // 默认：false 
    flush?: string // 默认：'pre'
}
```
参数说明：

WatcherSource: 用于指定要侦听的响应式数据源。侦听器数据源可以是返回值的 `getter` 函数，也可以直接是 `ref`。

Callback: 执行的回调函数，可**依次接收当前值newValue，先前值oldValue**作为入参。

WatchOptions：`deep`、`immediate`、`flush`可选。
 - 当需要对响应式对象进行深度监听时，设置`deep: true`。
 - 默认情况下watch是惰性的，当我们**设置`immediate: true`时，watch会在初始化时立即执行回调函数**。
- `flush` 选项可以更好地控制回调的时间。它可设置为 `pre`、`post` 或 `sync`。
  -  默认值是 `pre`，指定的回调应该在**渲染前**被调用。
  - `post` 值是可以用来将回调推迟到**渲染之后**的。如果回调需要通过 `$refs` 访问更新的 DOM 或子组件，那么则使用该值。
  - 如果 `flush` 被设置为 `sync`，一旦值发生了变化，回调将被**同步**调用（**少用，影响性能**）。
  
# 二、侦听单个数据源及停止侦听
```js
<script setup>
  import { watch, ref, reactive } from 'vue'
  // 侦听一个 getter
  const person = reactive({name: '小松菜奈'})
  watch(
    () => person.name,
    (value, oldValue) => {
      console.log(value, oldValue)
    }, {immediate:true}
  )
  person.name = '有村架纯'

  // 直接侦听ref
  const ageRef = ref(16)
  const stopAgeWatcher = watch(ageRef, (value, oldValue) => {
    console.log(value, oldValue)
    if (value > 18) {
      stopAgeWatcher() // 当ageRef大于18，停止侦听
    }
  })

  const changeAge = () => {
    ageRef.value += 1
  }
</script>
```

![未标题-3.gif](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/dec6f535d99c43a68dfa2e6ed2f70de4~tplv-k3u1fbpfcp-watermark.image?)

现象：配置了`immediate:true`的`watch`，在初始化时触发了一次`watch`的回调。我们连续点击增加年龄，当年龄的当前值大于18时，watch停止了侦听。

**结论**：侦听器数据源可以是返回值的 `getter` 函数，也可以直接是 `ref`。我们可以利用给`watch`函数取`名字`，然后通过执行`名字()`函数来停止侦听。

# 三、监听多个数据源

```js
<script setup>
  import {ref, watch, nextTick} from 'vue'

  const name = ref('小松菜奈')
  const age = ref(25)

  watch([name, age], ([name, age], [prevName, prevAge]) => {
    console.log('newName', name, 'oldName', prevName)
    console.log('newAge', age, 'oldAge', prevAge)
  })

  // 如果你在同一个函数里同时改变这些被侦听的来源，侦听器只会执行一次
  const change1 = () => {
    name.value = '有村架纯'
    age.value += 2
  }

  // 用 nextTick 等待侦听器在下一步改变之前运行,侦听器执行了两次
  const change2 = async () => {
    name.value = '新垣结衣'
    await nextTick()
    age.value += 2
  }
</script>
```

![未标题-4.gif](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b3d80d39224a4531af9e2c62dab4ea5c~tplv-k3u1fbpfcp-watermark.image?)

现象：以上，当我们在同一个函数里同时改变`name`和`age`两个侦听源，`watch`的回调函数只触发了**一次**；当我们在`name`和`age`的改变之间增加了一个`nextTick`，`watch`回调函数触发了两次。

**结论**：我们可以通过watch侦听**多个数据源**的变化。如果在同一个函数里同时改变这些被侦听的来源，侦听器只会执行一次。若要使侦听器执行多次，我们可以利用 `nextTick` ，等待侦听器在下一步改变之前运行。


# 四、侦听引用对象（数组Array或对象Object）

``` js
<template>
  <div>
    <div>ref定义数组：{{arrayRef}}</div>
    <div>reactive定义数组：{{arrayReactive}}</div>
  </div>
  <div>
    <button @click="changeArrayRef">改变ref定义数组第一项</button>
    <button @click="changeArrayReactive">改变reactive定义数组第一项</button>
  </div>
</template>

<script setup>
  import {ref, reactive, watch} from 'vue'

  const arrayRef = ref([1, 2, 3, 4])
  const arrayReactive = reactive([1, 2, 3, 4])

  // ref not deep, 不能深度侦听
  const arrayRefWatch = watch(arrayRef, (newValue, oldValue) => {
    console.log('newArrayRefWatch', newValue, 'oldArrayRefWatch', oldValue)
  })

  // ref deep， 深度侦听，新旧值一样
  const arrayRefDeepWatch = watch(arrayRef, (newValue, oldValue) => {
    console.log('newArrayRefDeepWatch', newValue, 'oldArrayRefDeepWatch', oldValue)
  }, {deep: true})

  // ref deep, getter形式 ， 新旧值不一样
  const arrayRefDeepGetterWatch = watch(() => [...arrayRef.value], (newValue, oldValue) => {
    console.log('newArrayRefDeepGetterWatch', newValue, 'oldArrayRefDeepGetterWatch', oldValue)
  })

  // reactive，默认深度监听，可以不设置deep:true, 新旧值一样
  const arrayReactiveWatch = watch(arrayReactive, (newValue, oldValue) => {
    console.log('newArrayReactiveWatch', newValue, 'oldArrayReactiveWatch', oldValue)
  })

  // reactive，getter形式 ， 新旧值不一样
  const arrayReactiveGetterWatch = watch(() => [...arrayReactive], (newValue, oldValue) => {
    console.log('newArrayReactiveFuncWatch', newValue, 'oldArrayReactiveFuncWatch', oldValue)
  })

  const changeArrayRef = () => {
    arrayRef.value[0] = 3
  }
  const changeArrayReactive = () => {
    arrayReactive[0] = 6
  }
</script>
```

![未标题-2.gif](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/983875649e7f4527bd520d801d14b586~tplv-k3u1fbpfcp-watermark.image?)

现象：当将引用对象采用`ref`形式定义时，如果不加上`deep:true`，`watch`是**侦听不到**值的变化的；而加上`deep:true`，`watch`可以侦听到数据的变化，但是当前值和先前值一样，即不能获取先前值。当将引用对象采用`reactive`形式定义时，不作任何处理，`watch`可以侦听到数据的变化，但是当前值和先前值一样。两种定义下，把`watch`的数据源写成getter函数的形式并进行**深拷贝**返回，可以在`watch`回调中同时获得当前值和先前值。

**结论**：
当我们使用`watch`侦听引用对象时
- 若使用`ref`定义的引用对象：
    - 只要获取当前值，watch第一个参数直接写成数据源，另外需要**加上**`deep:true`选项
    - 若要获取当前值和先前值，需要把数据源写成getter函数的形式，并且需对数据源进行深拷贝
- 若使用`reactive`定义的引用对象：
    - 只要获取当前值，watch第一个参数直接写成数据源，可以**不加**`deep：true`选项
    - 若要获取当前值和先前值，需要把数据源写成getter函数的形式，并且需对数据源进行深拷贝

🎉🎉🎉🎉🎉
