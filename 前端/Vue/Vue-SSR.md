## 服务器端渲染vue模板或者组件

第一点：如何渲染一个vue模板或者组件

```javascript
// 1.创建一个Vue instance
const Vue = require('vue')
const app = new Vue({
  template: `<div>Hello World</div>`
})

// 2.创建一个renderer
const renderer = require('vue-server-renderer').createRenderer()

// 3.将Vue渲染为html实例
renderer.renderToString(app, (err, html) => {
  if (err) {
    throw err
  }
  console.log(html)
})

```

第二点：如何响应网络请求，根据请求去做SSR。

```javascript
const Vue = require('vue')
const server = require('express')()
const renderer = require('vue-server-renderer').createRenderer()

server.get('*', (req, res) => {
  const app = new Vue({
    data: {
      url: req.url
    },
    template: `<div>Hello World</div>`
  })
  renderer.renderToString(app, (err, html) => {
    if (err) {
      throw err
    }
    console.log(html) // <div data-server-rendered="true">访问的 URL 是： /</div>
  })
})
server.listen(8080)
```

通过express中间件响应网络请求，监听800端口，一旦这个端口有请求，就去渲染一个vue对象。

很明显，通过第二点，我们只能在服务端自己写模板，这不符合模块化的技术方案，所以，还需要作进一步的优化，通过fs系统来读取模板，然后进行渲染。

第三点：使用模板来实现大文件渲染

```javascript
const Vue = require('vue')
const server = require('express')()
const fs = require('fs')
// 读取渲染的模板
const renderer = require('vue-server-renderer').createRenderer({
  template: fs.readFileSync('./index.template.html', 'utf-8')
})

server.get('*', (req, res) => {
  const app = new Vue({
    data: {
      url: req.url
    },
    template: `<div>访问的 URL 是： {{ url }}</div>`
  })
  const context = { // 可以进行插值， {{ 转义 }} and {{{ 不转义 }}}
    title: 'Hello',
    meta: `
      <meta ...>
      <meta ...>
    `
  }
  renderer.renderToString(app, context, (err, html) => {
    if (err) {
      throw err
    }
    console.log(html)
  })
})

server.listen(8080)
```

index.tempalte.html

```html
<!DOCTYPE html>
<html>
<head>
  <title>{{ title }}</title>
  {{{ meta }}}
</head>
<body>
  <!--vue-ssr-outlet-->
</body>
</html>
```



## 通用代码

在上面中，我们已经实现在服务端渲染了，但是服务端和客户端由于运行的环境可能不同，所以需要写一些在服务端和客户端通用的代码。

在服务端，只有beforeCreate、created会在服务端渲染中有效，所以应该避免在这两个生命周期中声明全局作用的代码。比如：

在created中设置一个setInterval，在客户端可以在beforeDestroy、destroyed中将其销毁，但是SSR期间只能使用beforeCreate、created两个声明周期，那么这个定时器就会保存下来。



## 源码构建

### 避免状态单例

在客户端，我们会在新的上下文中对代码进行取值，但是Node服务器是一个长期运行的进程，代码在进入Node进程时，会被Node取值被留存在内存中，所以如果创建了一个单例对象的话，这个对象会在各个请求之间共享，造成状态污染。

为了避免这个情况，可以采用工厂模式，为每一次请求都创建一个新的应用程序对象实例。

```javascript
// app.js --负责生产一个应用对象实例
const Vue = require('vue')
module.exports = function createAppInstance (context) {
  return new Vue ({
    data: {
      url: context.url
    },
    template: `<div>访问的 URL 是： {{ url }}</div>`
  })  
}
```

```javascript
// server.js --负责响应请求，将服务端渲染好的html发送到客户端
const server = require('express')()
const fs = require('fs')
const createAppIntsance = require('./app')
const renderer = require('vue-server-renderer').createRenderer({
  template: fs.readFileSync('./index.template.html', 'utf-8')
})

server.get('*', (req, res) => {
  const context1 = {
    url: req.url
  }
  const app = createAppIntsance(context1)
  const context2 = {
    title: 'Hello',
  }
  renderer.renderToString(app, context2, (err, html) => {
    if (err) {
      throw err
    }
    res.end(html)
  })
})

server.listen(8080)
```

从上面的server.js中可以看出，目前我们能处理‘*’的请求，这允许我们将访问的URL传递到我们的Vue应用程序中，然后对客户端和服务端复用想听的路由。同样，就像Vue应用实例一样，每个请求都需要新的router实例避免状态污染。所以可以创建一个router工厂。

```javascript
import Vue from 'vue'
import Router from 'vue-router'

Vue.use(Router)

export function createRouter () {
  return new Router({
    mode: 'history',
    routes: [
      { path: '/', component: () => import('./components/Home.vue') },
      { path: '/item/:id', component: () => import('./components/Item.vue') }
    ]
  })
}
```

同时需要将app.js更新，加入router实例：

```javascript
import Vue from 'vue'
import App from './App.vue'
import { creatRrouter } from './router'

export function createAppInstance () {
  const router = creatRrouter()
  const app = new Vue({ // 由于有router引入组件之后，就不需要在写template了。
    router,
    render: h => h(App)
  })
  return { app, router }
}
```

当更新了app应用实例之后，就可以去看看entry-server.js端来实现服务器端的路由逻辑了：

```javascript
// entry-server.js
import { createApp } from './app'

export default context => {
  // 因为有可能会是异步路由钩子函数或组件，所以我们将返回一个 Promise，
  // 以便服务器能够等待所有的内容在渲染前，
  // 就已经准备就绪。
  return new Promise((resolve, reject) => {
    const { app, router } = createApp()

    // 设置服务器端 router 的位置
    router.push(context.url)

    // 等到 router 将可能的异步组件和钩子函数解析完
    router.onReady(() => {
      const matchedComponents = router.getMatchedComponents()
      // 匹配不到的路由，执行 reject 函数，并返回 404
      if (!matchedComponents.length) {
        return reject({ code: 404 })
      }

      // Promise 应该 resolve 应用程序实例，以便它可以渲染
      resolve(app)
    }, reject)
  })
}
}
```

当entry-server.js有webpack打包好之后，由服务端进行服务端渲染：

```javascript
// server.js
const createApp = require('/path/to/built-server-bundle.js')

server.get('*', (req, res) => {
  const context = { url: req.url }

  createApp(context).then(app => {
    renderer.renderToString(app, (err, html) => {
      if (err) {
        if (err.code === 404) {
          res.status(404).end('Page not found')
        } else {
          res.status(500).end('Internal Server Error')
        }
      } else {
        res.end(html)
      }
    })
  })
})
```



## 数据预取和状态

在分配好路由之后，还有些问题，一些组件依赖于一些异步数据，那么在开始渲染之前，需要预取和解析好这些数据。如果需要这些预取数据，可以采用官方状态管理库 --- Vuex：

```javascript
import Vue from 'vue'
import Vuex from 'vuex'
// './api' 表示一些通用api
import { fetchItem } from './api'
Vue.use(Vuex)

export default createStore () {
  return new Vuex({
    state: {
      items: {}
    },
    actions: {
      fetchItem({ commit }, id) {
        return fetchItem(id).then(item => {
          commit('setItem', { id, item })
        })
      }
    },
    mutations: {
      setItem (state, { id, item }) {
        Vue.set(state.items, id, item)
      }
    }
  })
}
```

同样和router和app一样，每个请求都需要一个Vuex实例，store.js需要引入到app中，app.js修改如下：

```javascript
// app.js
import Vue from 'vue'
import App from './App.vue'
import { createRouter } from './router'
import { createStore } from './store'
import { sync } from 'vuex-router-sync'

export function createApp () {
  // 创建 router 和 store 实例
  const router = createRouter()
  const store = createStore()

  // 同步路由状态(route state)到 store
  sync(store, router)

  // 创建应用程序实例，将 router 和 store 注入
  const app = new Vue({
    router,
    store,
    render: h => h(App)
  })

  // 暴露 app, router 和 store。
  return { app, router, store }
}
```

Vuex定义好之后，这些数据如何存取呢？这些数据会通过需要渲染的组件触发Vuex中的事件来存取预取数据：

```html
<!-- Item.vue -->
<template>
  <div>{{ item.title }}</div>
</template>

<script>
export default {
  asyncData ({ store, route }) {
    // 触发 action 后，会返回 Promise
    return store.dispatch('fetchItem', route.params.id)
  },
  computed: {
    // 从 store 的 state 对象中的获取 item。
    item () {
      return this.$store.state.items[this.$route.params.id]
    }
  }
}
</script>
```

在store.js搭建完成之后，就可以通过entry-server.js在服务端进行数据预取了。

```javascript
// entry-server.js
import { createApp } from './app'

export default context => {
  return new Promise((resolve, reject) => {
    const { app, router, store } = createApp()

    router.push(context.url)

    router.onReady(() => {
      const matchedComponents = router.getMatchedComponents()
      if (!matchedComponents.length) {
        return reject({ code: 404 })
      }

      // 对所有匹配的路由组件调用 `asyncData()`
      Promise.all(matchedComponents.map(Component => {
        if (Component.asyncData) {
          return Component.asyncData({
            store,
            route: router.currentRoute
          })
        }
      })).then(() => {
        // 在所有预取钩子(preFetch hook) resolve 后，
        // 我们的 store 现在已经填充入渲染应用程序所需的状态。
        // 当我们将状态附加到上下文，
        // 并且 `template` 选项用于 renderer 时，
        // 状态将自动序列化为 `window.__INITIAL_STATE__`，并注入 HTML。
        context.state = store.state

        resolve(app)
      }).catch(reject)
    }, reject)
  })
}
```



## Bundle Renderer

到目前为止，服务端渲染的主要工作都已经完成。接下来就是一些更加符合人性的设计，比如Bundle Renderer，它是用来解决每次webpack都要在打包完成，生成新的打包之后的entry-server.js之后，重启服务器。这就给开发带来了很多不方便的地方。Bundle Renderer可以在不重新启动的情况下使用新打包的entry-server.js。

创建和使用Bundle Renderer的方法：

```javascript
const { createBundleRenderer } = require('vue-server-renderer')

const renderer = createBundleRenderer(serverBundle, {
  runInNewContext: false, // 推荐
  template, // （可选）页面模板
  clientManifest // （可选）客户端构建 manifest
})

// 在服务器处理函数中……
server.get('*', (req, res) => {
  const context = { url: req.url }
  // 这里无需传入一个应用程序，因为在执行 bundle 时已经自动创建过。
  // 现在我们的服务器与应用程序已经解耦！
  renderer.renderToString(context, (err, html) => {
    // 处理异常……
    res.end(html)
  })
})
```



## 构建配置

在知道如何使用SSR之后，应该了解以下如何使用webpac去配置。配置分为服务端配置和客户端配置。

### 服务端配置

服务器配置，是用于生成传递给 `createBundleRenderer` 的 server bundle。它应该是这样的： 

```javascript
const merge = require('webpack-merge')
const nodeExternals = require('webpack-node-externals')
const baseConfig = require('./webpack.base.config.js')
const VueSSRServerPlugin = require('vue-server-renderer/server-plugin')

module.exports = merge(baseConfig, {
  // 将 entry 指向应用程序的 server entry 文件
  entry: '/path/to/entry-server.js',

  // 这允许 webpack 以 Node 适用方式(Node-appropriate fashion)处理动态导入(dynamic import)，
  // 并且还会在编译 Vue 组件时，
  // 告知 `vue-loader` 输送面向服务器代码(server-oriented code)。
  target: 'node',

  // 对 bundle renderer 提供 source map 支持
  devtool: 'source-map',

  // 此处告知 server bundle 使用 Node 风格导出模块(Node-style exports)
  output: {
    libraryTarget: 'commonjs2'
  },

  // https://webpack.js.org/configuration/externals/#function
  // https://github.com/liady/webpack-node-externals
  // 外置化应用程序依赖模块。可以使服务器构建速度更快，
  // 并生成较小的 bundle 文件。
  externals: nodeExternals({
    // 不要外置化 webpack 需要处理的依赖模块。
    // 你可以在这里添加更多的文件类型。例如，未处理 *.vue 原始文件、.sass文件等，
    // 你还应该将修改 `global`（例如 polyfill）的依赖模块列入白名单
    whitelist: /\.css$/
  }),

  // 这是将服务器的整个输出
  // 构建为单个 JSON 文件的插件。
  // 默认文件名为 `vue-ssr-server-bundle.json`
  plugins: [
    new VueSSRServerPlugin()
  ]
})
```

在生成 `vue-ssr-server-bundle.json` 之后，只需将文件路径传递给 `createBundleRenderer`：

```javascript
const { createBundleRenderer } = require('vue-server-renderer')
const renderer = createBundleRenderer('/path/to/vue-ssr-server-bundle.json', {
  // ……renderer 的其他选项
})
```

又或者，你还可以将 bundle 作为对象传递给 `createBundleRenderer`。这对开发过程中的热重载是很有用的



### 客户端配置

