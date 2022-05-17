### 前后端开发模式的演变

在 jQuery 时代，对于大部分 Web 项目而言，前端都是不能控制路由的，而是需要依赖后端项目的路由系统。通常，前端项目也会部署在后端项目的模板里，整个项目执行的示意图如下：

![img](https://static001.geekbang.org/resource/image/26/2b/26ddd952f1f7d6dc3193af5be57e202b.jpg?wh=1569x462)

但是在这个时代，前端工程师并不需要了解路由的概念。对于每次的页面跳转，都由后端开发人员来负责重新渲染模板。

下图是目前前端开发中，用户访问页面的后代码的执行过程：

![img](https://static001.geekbang.org/resource/image/26/ec/2657d4eb129568d3c5b766e40eef60ec.jpg?wh=1920x435)

从上面示意图我们可以看到：用户访问路由后，无论是什么 UPL 地址，都直接渲染一个前端的入口文件 index.html，然后就会在 index.html 文件中加载 JS 和 CSS。之后，JavaScript 获取当前页面地址，以及当前路由匹配的组件，再去动态渲染当前页面。用户在页面上进行点击操作时，也不需要刷新页面，而是直接通过 JS 重新计算出匹配的路由渲染即可。

**这种所有路由都渲染一个前端入口文件的方式，是单页面应用程序（SPA，single page application）应用的雏形。**

通过 JavaScript 动态控制数据去提高用户体验的方式并不新奇，Ajax 让数据的获取不需要刷新页面，SPA 应用让路由跳转也不需要刷新页面。这种开发的模式在 jQuery 时代就出来了，浏览器路由的变化可以通过 pushState 来操作，这种纯前端开发应用的方式，以前称之为 Pjax （pushState+ Ajax）。之后，这种开发模式在 MVVM 框架的时代大放异彩，现在大部分使用 Vue/React/Angular 的应用都是这种架构。

SPA 应用相比于模板的开发方式，对前端更加友好，比如：前端对项目的控制权更大了、交互体验也更加丝滑，更重要的是，**前端项目终于可以独立出来单独部署了。**

### 前端路由的实现原理

现在，通过 URL 区分路由的机制上，有两种实现方式，一种是 hash 模式，通过 URL 中 # 后面内容做区分，我们称之为 hash-router；另外一种方式就是 history 模式，在这种方式下，路由看起来和正常的 URL 完全一致。

这两个不同的原理，在 vue-router 中对应两个函数，分别是 createWebHashHistory 和 createWebHistory。

![img](https://static001.geekbang.org/resource/image/d0/d3/d07894f8b9df7c1afed10b730f8276d3.jpg?wh=1546x561)

#### hash 模式

单页应用在页面交互、页面跳转上都是无刷新的，这极大地提高了用户访问网页的体验。

类似于服务端路由，前端路由实现起来其实也很简单，就是匹配不同的 URL 路径，进行解析，然后动态地渲染出区域 HTML 内容。**但是这样存在一个问题，就是 URL 每次变化的时候，都会造成页面的刷新。解决这一问题的思路便是在改变 URL 的情况下，保证页面的不刷新。**

在 2014 年之前，大家是通过 hash 来实现前端路由， URL hash 中的 # 就是类似于下面代码的这种 # ：

```
http://www.xxx.com/#/login
```

之后，在进行页面跳转的操作时，hash 值的变化并不会导致浏览器页面的刷新，只是会触发 hashchange 事件。在下面的代码中，通过对 hashchange 事件的监听，我们就可以在 fn 函数内部进行动态地页面切换。

```js
window.addEventListener('hashchange',fn)
```

#### history 模式

2014 年之后，因为 HTML5 标准发布，浏览器多了两个 **API：pushState** 和 **replaceState**。**通过这两个 API ，我们可以改变 URL 地址，并且浏览器不会向后端发送请求，我们就能用另外一种方式实现前端路由 **。**

在下面的代码中，我们监听了 popstate 事件，可以监听到通过 pushState 修改路由的变化。并且在 fn 函数中，我们实现了页面的更新操作。

```js
window.addEventListener('popstate', fn)
```

### 