### 前端数据管理

假设有这样的场景：一些数据需要在组件间共享，应该如何实现？

解决这个问题的最常见的一种思路就是：专门定义一个全局变量，任何组件需要数据的时候都去这个全局变量中获取。一些通用的数据，比如用户登录信息，以及一个跨层级的组件通信都可以通过这个全局变量很好地实现。在下面的代码中我们使用 _store 这个全局变量存储数据。

```bash
window._store = {}
```

数据存储的结构图大致如下，任何组件内部都可以通过 window._store 获取数据并且修改。

![img](https://static001.geekbang.org/resource/image/4d/4b/4de32506d33f278704d2edd7b2d8914b.jpg?wh=1920x936)

这样会产生一个问题，window._store 并不是响应式的，如果在 Vue 项目中直接使用，那么就无法自动更新页面。所以我们需要用 ref 和 reactive 去把数据包裹成响应式数据，并且提供统一的操作方法，这其实就是数据管理框架 Vuex 的雏形了。

### Vuex 是什么

**Vuex 就相当于我们项目中的大管家，集中式存储管理应用的所有组件的状态。**

们项目结构中的 src/store 目录，就是专门留给 Vuex 的，在项目的目录下，我们执行下面这个命令，进行 Vuex 的安装工作。

```bash
npm install vuex@next
```

安装完成后，我们在 src/store 中先新建 index.js，在下面的代码中，我们使用 createStore 来创建一个数据存储，我们称之为 store。store 内部除了数据，还需要一个 mutation 配置去修改数据，你可以把这个 mutation 理解为数据更新的申请单，mutation 内部的函数会把 state 作为参数，我们直接操作 state.count 就可以完成数据的修改。

```js
import { createStore } from 'vuex'

const store = createStore({
  state () {
    return {
      count: 666
    }
  },
  mutations: {
    add (state) {
      state.count++
    }
  }
})
export default store
```

现在你会发现，我们的代码里，在 Vue 的组件系统之外，多了一个数据源，里面只有一个变量 count，并且有一个方法可以累加这个 count。然后，我们在 Vue 中注册这个数据源，在项目入口文件 src/main.js 中，使用 app.use(store) 进行注册，这样 Vue 和 Vuex 就连接上了。然后，我们使用 .use 就可以对路由进行注册，使用 .mount 就可以把 Vue 这个应用挂载到页面上，代码如下。

```js
const app = createApp(App)
app.use(store)
    .use(router)
    .mount('#app')
```

之后，我们在 src/components 文件夹下新建一个 Count.vue 组件，在下面的代码中，template 中的代码我们很熟悉了，就是一个 div 渲染了 count 变量，并且点击的时候触发 add 方法。在 script 中，我们使用 useStore 去获取数据源，初始化值和修改的函数有两个变化：count 不是使用 ref 直接定义，而是使用计算属性返回了 store.state.count，也就是刚才在 src/store/index.js 中定义的 count。add 函数是用来修改数据，这里我们不能直接去操作 store.state.count +=1，因为这个数据属于 Vuex 统一管理，所以我们要使用 store.commit(‘add’) 去触发 Vuex 中的 mutation 去修改数据。

```html
<template>
<div @click="add">
    {{count}}
</div>
</template>

<script setup>
import { computed } from 'vue'
import {useStore} from 'vuex'
let store = useStore()
let count = computed(()=>store.state.count)

function add(){
    store.commit('add')
}
</script>
```

[实际上我把随即颜色的功能也加进去了](https://github.com/KurjaScript/geek-admain/commit/bed78192880ee3110264eba768653937d5f02e68)

一个问题：什么时候的数据用 Vuex 管理，什么时候数据要放在组件内部使用 ref 管理呢？

答案就是，**对于一个数据，如果只是组件内部使用就是用 ref 管理；如果我们需要跨组件，跨页面共享的时候，我们就需要把数据从 Vue 的组件内部抽离出来，放在 Vuex 中去管理。**

比如项目中的登录用户名，页面的右上角需要显示，有些信息弹窗也需要显示。这样的数据就需要放在 Vuex 中统一管理，每当需要抽离这样的数据的时候，我们都需要思考这个数据的初始化和更新逻辑。

![img](https://static001.geekbang.org/resource/image/9f/b9/9fca00b12fb51d52bbb48277a3c4e2b9.jpg?wh=1920x1224)

### 手写一个迷你 Vuex

首先，我们需要创建一个变量 store 用来存储数据。下一步就是**把这个 store 的数据包转成响应式的数据**，并且提供给 Vue 组件使用。在 Vue 中有 provide/inject 这两个函数专门用来做数据共享，**provide 注册了数据后，所有的子组件都可以通过 inject 获取数据**。

完成刚才的数据转换之后，我们直接进入到 src/store 文件夹下，新建 gvuex.js。下面的代码中，我们使用一个 Store 类来管理数据，类的内部使用 _state 存储数据，使用 mutations 来存储数据修改的函数，注意这里的 state 已经使用 reactive 包裹成响应式数据了。

```js
import { inject, reactive } from 'vue'

const STORE_KEY = '__store__'
function useStore() {
  return inject(STORE_KEY)
}
function createStore(options) {
  return new Store(options)
}
class Store {
  constructor(options) {
    this._state = reactive({
      data: options.state()
    })
    this._mutations = options.mutations
  }
}
export { createStore, useStore }
```

上面的代码还暴露了 createStore 去创建 Store 的实例，并且可以在任意组件的 setup 函数内，使用 useStore 去获取 store 的实例。下一步我们回到 src/store/index.js 中，把 vuex 改成 ./gvuex。下面的代码中，我们使用 createStore 创建了一个 store 实例，并且实例内部使用 state 定义了 count 变量和修改 count 值的 add 函数。

```js
// import { createStore } from 'vuex'
import { createStore } from './gvuex'
const store = ...
export default store
```

最终我们使用 store 的方式，在项目入口文件 src/main.js 中使用 app.use(store) 注册。为了让 useStore 能正常工作，下面的代码中，我们需要给 store 新增一个 install 方法，这个方法会在 app.use 函数内部执行。我们通过 app.provide 函数注册 store 给全局的组件使用。

```js
class Store {
  // main.js入口处app.use(store)的时候，会执行这个函数
  install(app) {
    app.provide(STORE_KEY, this)
  }
}
```

下面的代码中，Store 类内部变量 _state 存储响应式数据，读取 state 的时候直接获取响应式数据 _state.data，并且提供了 commit 函数去执行用户配置好的 mutations。

```js
import { inject, reactive } from 'vue'
const STORE_KEY = '__store__'
function useStore() {
  return inject(STORE_KEY)
}
function createStore(options) {
  return new Store(options)
}
class Store {
  constructor(options) {
    this.$options = options
    this._state = reactive({
      data: options.state
    })
    this._mutations = options.mutations
  }
  get state() {
    return this._state.data
  }
  commit = (type, payload) => {
    const entry = this._mutations[type]
    entry && entry(this.state, payload)
  }
  install(app) {
    app.provide(STORE_KEY, this)
  }
}
export { createStore, useStore }
```

这样在组件内部，我们就可以使用这个迷你的 Vuex 去实现一个累加器了。下面的代码中，我们使用 useStore 获取 store 的实例，并且使用计算属性返回 count，在修改 count 的时候使用 store.commit(‘add’) 来修改 count 的值。

```js
import {useStore} from '../store/gvuex'
let store =useStore()
let count = computed(()=>store.state.count)
function add(){
    store.commit('add')
}
```

这样借助 vue 的插件机制和 reactive 响应式功能，我们只用 30 行代码，就实现了一个最迷你的数据管理工具，也就是一个迷你的 Vuex 实现。

### Vuex 实战

#### getters

Vuex 可以用 getters 配置，来实现 computed 的功能。[以累加器数字乘以 2 为例](https://github.com/KurjaScript/geek-admain/commit/ea648f65f116a833c8eb9f23f125aaffbf04743b)

#### action

在实际开发中，有很多数据是从网络请求获取到的。在 Vuex 中，mutation 的设计就是用来实现同步地修改数据。如果需要异步修改，我们需要一个新的配置 **action**。

现在模拟一个异步场景：[点击按钮之后的 1 秒，再去做数据修改](https://github.com/KurjaScript/geek-admain/commit/0c1cfcb455ddb282894386ef38621d24b0a5ebbc)。



在 action 中，开发者可以做任意的异步处理。这里使用 seTimeout 来模拟延时，然后在 action 内部调用 mutation。

**action 并不直接修改数据，而是通过 mutations 去修改。**actions 的调用方式是使用 store.dispatch。

Vuex 在整体上的逻辑，从宏观来说，Vue 的组件负责渲染页面，组件中用到跨页面的数据，就是用 state 来存储，但是 Vue 不能直接修改 state，而是通过 actions/mutations 去做数据的修改。

![img](https://static001.geekbang.org/resource/image/85/28/851478d3f2b0393474de6e5b3b355a28.png?wh=1280x866)

下面这个图也是 Vuex 官方的结构图，很好地拆解了 Vuex 在 Vue 全家桶中的定位。我们项目中也会用 Vuex 来管理所有跨组件的数据，并且我们也会在 Vuex 内部根据功能模块去做拆分，会把用户权限等不同模块的组件分开去管理。

![img](https://static001.geekbang.org/resource/image/23/7a/237557819e2148ac022305eaf86c0b7a.png?wh=701x551)

回到正在做的这个项目中，有大量的数据交互需求、用户的登录状态、登录的有效期、布局的设置，不同用户还会有不同的菜单权限等。

面对不同的交互需求，**总体来说，我们在决定一个数据是否用 Vuex 来管理的时候，核心就是要思考清楚，这个数据是否有共享给其他页面或者是其他组件的需要。**如果需要，就放置在 Vuex 中管理；**如果不需要，就应该放在组件内部使用 ref 或者 reactive 去管理。**