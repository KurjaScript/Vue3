### 升级到 Vue3 的理由

**而 Vue 3 的 Composition API 带来的代码组织方式更利于封装代码，维护起来也不会上下横跳。**

Vite 则带来了更丝滑的调试体验，一步步跟着专栏完成你的第一个 Vue 3 项目，你会感受到 Vue 3 的魅力。

### Vue3 不兼容的那些写法

首先，我们来看一下 Vue 2 和 Vue 3 在项目在启动上的不同之处。

在 Vue 2 中，我们使用 `new Vue()` 来新建应用，有一些全局的配置我们会直接挂在 Vue 上，比如我们通过 `Vue.use` 来使用插件，通过 `Vue.component` 来注册全局组件，如下面代码所示：

```vue

Vue.component('el-counter', {
  data(){
    return {count: 1}
  },
  template: '<button @click="count++">Clicked {{ count }} times.</button>'
})

let VueRouter = require('vue-router')
Vue.use(VueRouter)
```

在上面的代码里，我们注册了一个 el-counter 组件，这个组件是全局可用的，它直接渲染一个按钮，并且在点击按钮的时候，按钮内的数字会累加。

然后我们需要注册路由插件，这也是 Vue 2 中我们使用 vue-router 的方式。这种形式虽然很直接，但是由于全局的 Vue 只有一个，所以当我们在一个页面的多个应用中独立使用 Vue 就会非常困难。

看下面这段代码，我们在 Vue 上先注册了一个组件 el-counter，然后创建了两个 Vue 的实例。这两个实例都自动都拥有了 el-couter 这个组件，但这样做很容易造成混淆。

```vue
Vue.component('el-counter',...)

new Vue({el:'#app1'})
new Vue({el:'#app2'})
```

为了解决这个问题，Vue 3 引入一个新的 API ，`createApp`，来解决这个问题，也就是新增了 App 的概念。全局的组件、插件都独立地注册在这个 App 内部，很好的解决了上面提到的两个实例容易造成混淆的问题。下面的代码是使用 createApp 的简单示例：

```vue
const { createApp } = Vue
const app = createApp({})
app.component(...)
app.use(...)
app.mount('#app1')

const app2 = createApp({})
app2.mount('#app2')
```

createApp 还移除了很多我们常见的写法，比如在 createApp 中，就不再支持 `filter、$on、$off、$set、$delete` 等 API。不过这都不用担心，后面我会告诉你怎么去实现类似这些 API 的功能。

在 Vue 3 中，v-model 的用法也有更改。在后面讲到组件化，也就是我们需要深度使用 v-model 的时候，我会再细讲。 其实 Vue 3 还有很多小细节的更新，比如 **slot 和 slot-scope 两者实现了合并**，而 directive 注册指令的 API 等也有变化。你现在记不住这些也不要紧，我们会在后面的实战项目里逐渐掌握这些内容。

### Vue3 生态现状

Vue-cli4 已经提供内置选项，你当然可以选择它支持的 Vue 2。如果你对 Vite 不放心的话，Vue-cli4 也全面支持 Vue 3，这还是很贴心的。

vue-router 是复杂项目必不可少的路由库，它也包含一些写法上的变化，比如从 new Router 变成 createRouter；使用方式上，也全面拥抱 Composition API 风格，提供了 useRouter 和 useRoute 等方法。

