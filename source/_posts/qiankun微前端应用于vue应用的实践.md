---
title: qiankun微前端应用于vue应用的实践
date: 2021-09-01 16:28:00
tags: qiankun
---

# 前言✨
本文介绍了用qiankun微前端框架用来集成vue应用的基本方案以及集成过程中遇到的一些常见问题。

# 一、基本使用

## 主应用
1.先安装 `qiankun` ：

```
$ yarn add qiankun # 或者 npm i qiankun -S
```

2.注册微应用并启动：

```
import { registerMicroApps, start } from 'qiankun'

/**
 * step1 注册微应用
 */
registerMicroApps([
  {
    name: 'child-vue2', // 注册应用名
    entry: '//localhost:7012',// 注册服务
    container: '#sub-container', // 挂载的容器
    activeRule: '/vue2', // 路由匹配规则
  },
  {
    name: 'child-vue3',
    entry: '//localhost:7013',
    container: '#sub-container',
    activeRule: '/vue3',
  }
])

/**
 * Step2 设置默认进入的子应用（可选）
 */
setDefaultMountApp('/')

/**
 * Step3 启动qiankun应用
 */
start()
```
## 微应用
### Vue 微应用
微应用不需要额外安装任何其他依赖即可接入 qiankun 主应用。

以 `vue 2.x` 项目为例

1.  在 `src` 目录新增 `public-path.js`，并在`main.js`最顶部引入 `public-path.js`

    ```
    if (window.__POWERED_BY_QIANKUN__) {
      __webpack_public_path__ = window.__INJECTED_PUBLIC_PATH_BY_QIANKUN__;
    }
    ```
    
2. 微应用建议使用 `history` 模式的路由，需要设置路由 `base`，值和它的 `activeRule` 是一样的
```
const router = new VueRouter({
  mode: 'history',
  base: window.__POWERED_BY_QIANKUN__ ? '/vue2' : '/',
  routes
})
```


3.  在入口文件`main.js`中修改并导出`bootstrap`、`mount`、`unmount`三个生命周期函数，保证微应用单独运行且能在qiankun环境中运行。为了避免根 id `#app` 与其他的 DOM 冲突，需要限制查找范围。

    ```
    import './public-path'
    import Vue from 'vue'
    import App from './App.vue'
    import routes from './router'
    import store from './store'

    Vue.config.productionTip = false

    let instance = null
    function render(props = {}) {
      const { container } = props
      instance = new Vue({
        router,
        store,
        render: (h) => h(App)
      }).$mount(container ? container.querySelector('#app') : '#app')
    }

    // 独立运行微应用时
    if (!window.__POWERED_BY_QIANKUN__) {
      render()
    }

    export async function bootstrap() {
      console.log('[vue] vue app bootstraped')
    }
    
    // 在qiankun中运行时
    export async function mount(props) {
      render(props)
    }
    
    export async function unmount() {
      instance.$destroy()
      instance.$el.innerHTML = ''
      instance = null
    }
    ```

4.  修改 `webpack` 打包，允许开发环境跨域和 `umd` 打包（`vue.config.js`）：

    ```
    const { name } = require('./package');
    module.exports = {
      devServer: {
        headers: {
          'Access-Control-Allow-Origin': '*',
        },
      },
      configureWebpack: {
        output: {
          library: `${name}-[name]`,
          libraryTarget: 'umd', // 把微应用打包成 umd 库格式
          jsonpFunction: `webpackJsonp_${name}`,
        },
      },
    };
    ```

# 二、细节 
### 1.生命周期
通过此图我们可以了解主应用和子应用的钩子函数的加载顺序，方便我们对其进行控制。
![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4c2029c71cb2404281626d18c5d651fe~tplv-k3u1fbpfcp-watermark.image)
### 2.样式隔离
#### qiankun提供了两种隔离模式：
- 1.严格沙箱（不推荐）
```
start({sandbox: {strictStyleIsolation: true}})
```
开启此配置时，子应用会以Shadow DOM 的形式嵌入到主应用，实现真正的样式隔离，但是同时会带来新的问题：

Popover弹出框、Select下拉选项、Dropdown下拉菜单等会因找不到主应用的body会丢失，或跑到整个屏幕外。（目前还没有找到解决的办法，todo）

- 2.实验沙箱（不推荐）
```
start({sandbox: {experimentalStyleIsolation: true}})
```
开启此配置时，相当于实现了类似于vue中style标签中的scoped属性，qiankun会自动为子应用所有的样式增加后缀标签，如：`div[data-qiankun-microName]`，此配置会导致：

Popover弹出框、Select下拉选项、Dropdown下拉菜单等插入到主应用中，而主应用不能访问到scope域的样式，而导致样式丢失。

解决方案： 可以把子应用的弹窗样式copy一份至主应用的全局样式中。但是此方法带来的维护成本太大。

#### 不采用qiankun的沙箱隔离
既然qiankun带来的沙箱隔离带来这么多问题, 我们能不能自己来实现样式隔离了，下面提供三个思路：
- 1.采用规范的命名（主应用公共样式较少时推荐）

    在主应用的公共样式比较少的情况下可以使用规范唯一的命名来避免样式污染（如采用`BEM`规范来命名），把非公共样式尽量利用vue的scoped属性进行隔离。
    
- 2.采用css-module的方式隔离（主子隔离时，推荐）
（1）在主应用的`vue.config.js`里，加入如下配置，以.module.(css|less|sass|scss|styl) 结尾的css开启模块化
```
module.exports = {
  css: {
    extract: false, // 是否使用css分离插件 ExtractTextPlugin
    sourceMap: true,
    requireModuleExtension: true, // true则只有以.module.(css|less|sass|scss|styl) 结尾的开启模块化
    loaderOptions: {
      css: {
        modules: {
          localIdentName: '[path][name]-[hash]'
        }
      }
    }
  }
}
```

（2）然后公共样式以模块化的方式引入，然后再绑定在页面相应的class上，下面是一个模块化css引入并使用的示例：

```
<template>
  <div class="home">
    <div :class="module.good">good</div>
  </div>
</template>

<script>
import module from './../assets/reset.module.less'
export default {
  data() {
    return {
      module // 需要在data中变成响应式，然后再绑定到class上
    }
  }
}
</script>
```
- 3.安装配置postcss-plugin-namespace插件来实现隔离（如果多个子应用同时出现在一个页面，推荐）
（1） 添加依赖
```
npm i postcss-plugin-namespace -D
```
（2） 配置postcss
在`postcss.config.js`中添加如下配置
```
module.exports = {
  plugins: [
    require('postcss-plugin-namespace')('.main-project', { // 相当于给工程的css加一个作用域前缀
      ignore:  ['*']
    })
  ]
}
```
上述配置相当于给工程的所有样式添加了一个作用域前缀`.main-project`，我们只需要在挂载的根节点加上这个class，就可以实现应用之间的隔离。

### 3.history模式下`publicPath`不能使用相对路径，需使用绝对路径
主应用需配置
```
publicPath:'/'
```
### 4.应用间的通信
qiankun提供了`initGlobalState` `onGlobalStateChange` `setGlobalState`用于通信。

主应用我们可以直接导入
```
import { initGlobalState } from 'qiankun'
// 初始化全局变量
const { onGlobalStateChange, setGlobalState } = initGlobalState({
  user: 'qiankun'
})

// 把监测变量改变和改变状态挂载到Vue上
Vue.prototype.$onGlobalStateChange = onGlobalStateChange
Vue.prototype.$setGlobalState = setGlobalState

// 监测变量改变
onGlobalStateChange((value, prev) => console.log('[onGlobalStateChange - master]:', value, prev),true)

// 改变全局变量
setGlobalState({
  user: 'master'
})
```

子应用我们需在生命周期里把`onGlobalStateChange` `setGlobalState`挂载到子应用的vue对象上：

```
export async function mount(props) {
    Vue.prototype.$onGlobalStateChange = props.onGlobalStateChange
    Vue.prototype.$setGlobalState = props.setGlobalState
}
```

### 5.如何在主应用的多级嵌套路由页面下加载微应用
主应用注册这个路由时给 `path` 加一个 `*`，使其在未匹配到主应用的某个页面时，也能加载主应用的外框，确保子应用要挂载的容器存在。
```
import empty from "./../empty"

export default {
  path: '/app',
  name: 'AppLayout',
  component: AppLayout,
  children: [{
    path: "manage",
    name: 'AppManage',
    component: Manage,
  },{
    path: "*",
    name: 'empty',
    component: empty //空的div标签
  }]
}
```
另外在注册微应用时，`activeRule` 需要包含主应用的这个路由 `path`
```
registerMicroApps([
  {
    name: 'website', // 注册应用名
    entry: '//localhost:8888',// 注册服务
    container: '#con', // 挂载的容器
    activeRule: '/app/website', // 路由匹配规则
  }
])
```

子应用的路由注册时`base`也需要加上这个`path`
```
function render(props = {}) {
  const {container} = props
  router = new VueRouter({
    mode: 'history',
    base: container ? /app/website : '/',
    routes
  })
  instance = new Vue({
    router,
    render: h => h(App)
  }).$mount(container ? container.querySelector('#app') : '#app')
}
```
### 6.微应用打包之后 css 中的字体文件和图片加载 404
原因是 `qiankun` 将外链样式改成了内联样式，但是字体文件和背景图片的加载路径是相对路径。而 `css` 文件一旦打包完成，就无法通过动态修改 `publicPath` 来修正其中的字体文件和背景图片的路径。

借助 `webpack` 的 `url-loader` 将字体文件和图片打包成 `base64`（适用于字体文件和图片体积小的项目）（**推荐**）

`vue-cli3` 项目写法：

```
module.exports = {
  chainWebpack: (config) => {
    config.module.rule('fonts').use('url-loader').loader('url-loader').options({}).end();
    config.module.rule('images').use('url-loader').loader('url-loader').options({}).end();
  },
};
```

### 7.发布时，微应用的nginx要配置支持跨域
只需要在微应用Nginx的配置文件中配置以下参数：

```
location / {  
    add_header Access-Control-Allow-Origin *;
    add_header Access-Control-Allow-Methods 'GET, POST, OPTIONS';
    add_header Access-Control-Allow-Headers 'DNT,X-Mx-ReqToken,Keep-Alive,User-Agent,X-Requested-With,If-Modified-Since,Cache-Control,Content-Type,Authorization';

    if ($request_method = 'OPTIONS') {
        return 204;
    }
} 
```
主应用的nginx需要配置代理转发：
```
location ~ ^/api/ {
    proxy_pass http://8.130.24.19:8105; // 代理地址
 proxy_set_header   Host             $host;
 proxy_set_header   X-Real-IP        $remote_addr;
 proxy_set_header   X-Forwarded-For  $proxy_add_x_forwarded_for;
 proxy_set_header Cookie $http_cookie;
 proxy_cookie_path / /df/;
 rewrite ^/api/(.*)$ /$1 break;
}
```
更多细节方案持续更新中~🚀🚀🚀

官网：https://qiankun.umijs.org/zh
