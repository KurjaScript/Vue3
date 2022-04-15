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

### 使用自动化升级工具进行 Vue 的升级

对于复杂项目，我们需要借助几个自动化工具来帮我们过渡。

首先是在 Vue 3 的项目里，有一个 @vue/compat 的库，这是一个 Vue 3 的构建版本，提供了兼容 Vue 2 的行为。这个版本默认运行在 Vue 2 下，它的大部分 API 和 Vue 2 保持了一致。当使用那些在 Vue 3 中发生变化或者废弃的特性时，这个版本会提出警告，从而避免兼容性问题的发生，帮助你很好地迁移项目。并且通过升级的提示信息，@vue/compat 还可以很好地帮助你学习版本之间的差异。

在下面的代码中，首先我们把项目依赖的 Vue 版本换成 Vue 3，并且引入了 @vue/compat 。

```js
"dependencies": {
-  "vue": "^2.6.12",
+  "vue": "^3.2.19",
+  "@vue/compat": "^3.2.19"
   ...
},
"devDependencies": {
-  "vue-template-compiler": "^2.6.12"
+  "@vue/compiler-sfc": "^3.2.19"
}
```

然后给 vue 设置别名 @vue/compat，也就是以 compat 作为入口，代码如下：

```js
// vue.config.js
module.exports = {
  chainWebpack: config => {
    config.resolve.alias.set('vue', '@vue/compat')
    ......
  }
}
```

这时你就会在控制台看到很多警告，以及很多优化的建议。我们参照建议，挨个去做优化就可以了。

在 @vue/compat 提供了很多建议后，我们自己还是要慢慢做修改。但从另一个角度看，“偷懒”是优秀程序员的标志，社区就有能够做自动化替换的工具，比较好用的就是“阿里妈妈”出品的 gogocode，[官方文档](https://gogocode.io/zh/docs/specification/vue2-to-vue3)，就不在这里赘述了。

**自动化替换工具的原理很简单，和 Vue 的 Compiler 优化的原理是一样的，也就是利用编译原理做代码替换。**如下图所示，我们利用 babel 分析左边 Vue 2 的源码，解析成 AST，然后根据 Vue 3 的写法对 AST 进行转换，最后生成新的 Vue 3 代码。

![img](https://static001.geekbang.org/resource/image/e3/e0/e371fee0a7e75942151724yy58fbfee0.jpg?wh=1920x1040)

对于替换过程的中间编译成的 AST，你可以理解为用 JavaScript 的对象去描述这段代码，这和虚拟 DOM 的理念有一些相似，我们基于这个对象去做优化，最终映射生成新的 Vue 3 代码。关于 AST 的细节，在课程后面的 Vue 3 生态源码篇中，我会带你手写一个迷你版的 Vue 3 Compiler，那时你会对 AST 和它背后的编译原理有一个更深的认识。