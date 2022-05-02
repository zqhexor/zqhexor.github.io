---
title: 打造vue2的onePiece环境
date: 2022-04-24 13:27:03
tags: vue2
---
# 前言✨

或许你和我一样习惯了`Vue3`的写法，体验到使用`CompositionAPI`抽离`hooks`的酸爽以及使用`<script setup>`语法糖的精炼之后，瞬间就会觉得以前`Vue2`的`OptionsAPI`不香了，那我们如何在`Vue2`中使用`CompositionAPI`了？

嘻嘻(o゜▽゜)o☆~让我们来开启`Vue2`中的`One Piece`吧！🚀

# 1️⃣ 使用compositionAPI

首先我们先满足基本需求，使用`CompositionAPI`~

在命令行中输入以下命令，安装[@vue/composition-api](https://github.com/vuejs/composition-api)：

```sh
npm install @vue/composition-api
```
在入口文件`main.js`中，通过`Vue.use()`注册
```js
// main.js
import CompositionApi from '@vue/composition-api'

Vue.use(CompositionApi)
```
接下来，我们就可以使用`CompositionAPI`了

```js
<script>
import HelloWorld from '@/components/HelloWorld.vue'
import { ref } from '@vue/composition-api'

export default {
  components: {
    HelloWorld
  },
  props:{
    value: {
      type: String,
      default: ''
    }
  }
  setup(props， ctx) {
    const name = ref('小松菜奈')
    return {
      name
    }
  }
}
</script>

```

# 2️⃣ 使用script setup语法糖

> `<script setup>` 声明的顶层的绑定 (包括变量，函数声明，以及 import 引入的内容) 都能在模板中直接使用，即变量、函数无需返回，组件无需注册

既然`<script setup>`语法糖这么香，咱还等什么，立即行动~

在命令行中输入以下命令，安装[unplugin-vue2-script-setup](https://github.com/antfu/unplugin-vue2-script-setup):

```sh
npm i unplugin-vue2-script-setup -D
```

如果咱们使用的是`vue-cli`，则需在`vue.config.js`中加上如下配置：
```
// vue.config.js
const ScriptSetup = require('unplugin-vue2-script-setup/webpack').default

module.exports = {
  parallel: false,  // disable thread-loader, which is not compactible with this plugin
  configureWebpack: {
    plugins: [
      ScriptSetup({ /* options */ }),
    ],
  },
}
```
`vite`、`webpack`等更多配置，请查看[unplugin-vue2-script-setup](https://github.com/antfu/unplugin-vue2-script-setup) ~


使用了`<script setup>`语法糖后，代码就是如此简洁：
```js
<script setup>
import HelloWorld from '@/components/HelloWorld.vue'
import { ref } from '@vue/composition-api'
const props = defineProps({
  value: {
    type: String,
    default: ''
  }
})
const name = ref('小松菜奈')
</script>
```
> `defineProps`、`defineEmits`、`defineExpose`是只在 `<script setup>` 中才能使用的**编译器宏**，他们不需要导入且会随着 `<script setup>` 处理过程一同被编译掉。

因此，我们在使用这三个api时不需要导入，但是`eslint`却不知道。故我们需要在`eslintrc.js`中作如下配置，消除`eslint(no-undef)`提示：
```js
// .eslintrc.js
module.exports = {
   globals: {
    defineProps: 'readonly',
    defineEmits: 'readonly',
    defineExpose: 'readonly'
  }
}
```

- 可能你的eslint会出现如下提示:

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b80b8d6b638b410ba3f2243976e40069~tplv-k3u1fbpfcp-watermark.image?)

我们可以在`.eslintrc.js`中的`rules`作如下配置：

```js
// .eslintrc.js
  rules: {
    'no-unused-vars': [
      'error',
      // Allowed unused vars must match /^__/u  no-unused-vars
      { varsIgnorePattern: '^__', args: 'none' } 
    ]
  }
```
如果你是使用`ts`的话，参考下尤大的配置：[.eslintrc.js](https://github.com/vuejs/vue-next/blob/master/.eslintrc.js)
```js
// .eslintrc.js
  rules: {
    'no-unused-vars': [
      'error',
      // we are only using this rule to check for unused arguments since TS
      // catches unused variables but not args
      { varsIgnorePattern: '.*', args: 'none' }   
    ]
  }
```


- 在`Vue2`项目中使用`Volar`插件
```sh
npm i @vue/runtime-dom -D
```

```js
// jsconfig.json
{
  "vueCompilerOptions": {
    "experimentalCompatMode": 2
  }
}

```

# 3️⃣ 从compositionAPI中自动导入所需api（可选）
使用[`unplugin-auto-import`](https://github.com/antfu/unplugin-auto-import)，能帮助我们从`compositionAPI`中自动按需导入所需的API，如`ref`、`reactive`等。

这个插件不是必须的，但是能帮我们提高编码效率，使用后，我们的代码可以简化成：
```js
<script setup>
import HelloWorld from '@/components/HelloWorld.vue'
// import { ref } from '@vue/composition-api' // 无需导入，自动按需使用
const props = defineProps({
  value: {
    type: String,
    default: ''
  }
})
const name = ref('小松菜奈')
</script>
```

通过以下命令安装此插件
```sh
npm i -D unplugin-auto-import
```
在`vue.config.js`中加上如下配置：

```js
// vue.config.js

const AutoImport = require('unplugin-auto-import/webpack')

module.exports = {
    configureWebpack: {
      plugins: [
        AutoImport({
          imports: ['@vue/composition-api'], // auto import apis from '@vue/composition-api'
          // Generate corresponding .eslintrc-auto-import.json file
          eslintrc: {
            enabled: true, // Default `false`, true开启
            filepath: './.eslintrc-auto-import.json', 
            globalsPropValue: true // Default `true`
          }
        })
      ]
    }
  }

```
解释一下上面eslintrc的配置项：
   - `enabled`：可以是(true | false），是否开启自动生成自适应的全局变量配置文件
   - `filepath`： 配置自动生成的`eslint`全局变量的json文件路径
   - `globalsPropValue`： 生成的全局变量的可访问性，可以是(true | false | 'readonly' | 'readable' | 'writable' | 'writeable')中的一种，布尔值 `false` 和字符串值 `"readable"` 等价于 `"readonly"`。类似地，布尔值 `true` 和字符串值 `"writeable"` 等价于 `"writable"`。
   
   
根据上面的配置，在我们启动工程时，工程会在`src`目录下自动生成一份名叫`.eslintrc-auto-import.json`的全局变量配置文件，然后我们在`.eslintrc.js`配置文件中引入这个配置文件，即可消除`eslint`的对当前页未引入的全局变量的`eslint(no-undef)`提示。
```
// .eslintrc.js

module.exports = {
  /* ... */
  extends: [
    // ...
    './.eslintrc-auto-import.json',
  ],
}
```

当然这个插件[`unplugin-auto-import`](https://github.com/antfu/unplugin-auto-import)能帮助我们提高效率的方式远非如此，比如自动引入其他的类库，详见[unplugin-auto-import](https://github.com/antfu/unplugin-auto-import) ~

🎉🎉🎉🎉🎉

