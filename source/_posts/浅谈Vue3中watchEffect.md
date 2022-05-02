---
title: 浅谈Vue3中watchEffect
date: 2022-05-02 18:50:38
tags: vue3
---

## 前言
`watchEffect`，它立即执行传入的一个函数，同时响应式追踪其依赖，并在其依赖变更时重新运行该函数。

换句话说：`watchEffect`相当于将`watch` 的依赖源和回调函数合并，当任何你有用到的响应式依赖更新时，该回调函数便会重新执行。不同于 `watch`，`watchEffect` 的回调函数会被立即执行（即 `{ immediate: true }`）

此文主要讲述怎样利用`清除副作用`使我们的代码更加优雅~

## watchEffect的副作用

什么是副作用（`side effect`），简单的说副作用就是执行某种操作，如对外部可变数据或变量的修改，外部接口的调用等。`watchEffect`的回调函数就是一个副作用函数，因为我们使用`watchEffect`就是侦听到依赖的变化后执行某些操作。

当执行副作用函数时，它势必会对系统带来一些影响，如在副作用函数里执行了一个定时器`setInterval`，因此我们必须处理副作用。
`Vue3`的`watchEffect`侦听副作用传入的函数可以接收一个 `onInvalidate` 函数作为入参，用来注册清理失效时的回调。当以下情况发生时，这个失效回调会被触发：
-   副作用即将重新执行时（即依赖的值改变）
-   侦听器被停止 (通过显示调用返回值停止侦听，或组件被卸载时隐式调用了停止侦听)

```js
import { watchEffect, ref } from 'vue'

const count = ref(0)
watchEffect((onInvalidate) => {
  console.log(count.value)
  onInvalidate(() => {
    console.log('执行了onInvalidate')
  })
})

setTimeout(()=> {
  count.value++
}, 1000)
```

上述代码打印的顺序为： `0` -> `执行了onInvalidate，最后执行` -> `1`

分析：初始化时先打印`count`的值`0`， 然后由于定时器把`count`的值更新为`1`, 此时副作用即将重新执行，因此`onInvalidate`的回调函数会被触发，打印`执行了onInvalidate`，然后执行了副作用函数，打印`count`的值`1`。

```js
import { watchEffect, ref } from 'vue'

const count = ref(0)
const stop = watchEffect((onInvalidate) => {
  console.log(count.value)
  onInvalidate(() => {
    console.log('执行了onInvalidate')
  })
})

setTimeout(()=> {
  stop()
}, 1000)
```
上述代码：当我们显示执行`stop`函数停止侦听，此时也会触发`onInvalidate`的回调函数。同样，`watchEffect`**所在的组件被卸载时**会隐式调用`stop`函数停止侦听，故也能触发`onInvalidate`的回调函数。

## watchEffect的应用
利用`watchEffect`的非惰性执行，以及传入的`onInvalidate` 函数，我们可以做什么事情了？

**场景一**：平时我们定义一个定时器，或者监听某个事件，我们需要在`mounted`生命周期钩子函数内定义或者注册，然后组件销毁之前在`beforeUnmount`钩子函数里清除定时器或取消监听。这样做我们的逻辑被分散在两个生命周期，不利于维护和阅读。

如果我利用`watchEffect`，创造和销毁逻辑放在了一起，此时代码更加优雅易读~

```js
// 定时器注册和销毁
watchEffect((onInvalidate) => {
  const timer = setInterval(()=> {
    // ...
  }, 1000)
  onInvalidate(() => clearInterval(timer))
})

const handleClick = () => {
 // ...
}
// dom的监听和取消监听
onMounted(()=>{
  watchEffect((onInvalidate) => {
    document.querySelector('.btn').addEventListener('click', handleClick, false)
    onInvalidate(() => document.querySelector('.btn').removeEventListener('click', handleClick))
  })
})
```

**场景二**：利用watchEffect作一个防抖节流（如取消请求）

```js
const id = ref(13)
watchEffect(onInvalidate => {
   // 异步请求
  const token = performAsyncOperation(id.value)
  // 如果id频繁改变，会触发失效函数，取消之前的接口请求
  onInvalidate(() => {
    // id has changed or watcher is stopped.
    // invalidate previously pending async operation
    token.cancel()
  })
})
```
......

当然`watchEffect`还能做很多事情，比如打开一个修改的`modal`弹窗，如果检测到`id`变化，我们可以在`onInvalidate`函数内，重置初始参数...这里只是一个抛砖引玉的作用，望大家多多发掘~

🚀🚀🚀🚀🚀

相关阅读：[Vue3中watch的最佳实践](https://zqhexor.github.io/2021/07/04/Vue3中watch的最佳实践/)
