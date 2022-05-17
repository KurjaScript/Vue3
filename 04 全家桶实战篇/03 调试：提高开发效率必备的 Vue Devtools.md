### Chrome 调试工具

Chrome 的开发者工具 **Chrome DevTools**。

![img](https://static001.geekbang.org/resource/image/e2/25/e2d00f93dcbb2c960720b41c45479125.png?wh=1714x210)

- Elements 页面可以帮助我们调试页面的 HTML 和 CSS；
- Console 页面可以帮助我们调试 JavaScript；
- Source 页面可以帮助我们调试开发中的源码；
- Application 页面可以帮助我们调试本地存储和一些浏览器服务，比如 Cookie、Localstorage、通知等等。
- Network 页面在我们开发前后端交互接口的时候，可以让我们看到每个网络请求的状态和参数；
- Performance 页面则用来调试网页性能；
- Lighthouse 是 Google 官方开发的插件，用来获取网页性能报告。

以上都是 Chrome 的开发者工具中自带的选项，而调试窗口最后面的 Vue 页面就是需要额外安装的 Vue Devtools，也就是调试 Vue 必备的工具。

### Vue Devtools

Vue Devtools 可以算是一个 Elements 页面的 Vue 定制版本，调试页面左侧的显示内容并不是 HTML，而是 Vue 的组件的嵌套关系。

不多说了，得实际调试。[详细课程](https://time.geekbang.org/column/article/442479)

这里有一个小技巧：在 Component 页面下，选中一个组件后，调试窗口的右侧会出现 4 个小工具。

![img](https://static001.geekbang.org/resource/image/53/46/53b82ec4499c08b0b8403a8471070946.png?wh=1920x807)

最右边的那个工具可以让你直接在编辑器里打开这个代码。这样，调试组件的时候就不用根据路径再去 VS Code 里搜索代码文件了，这算是一个非常好用的小功能。

### 断点调试

如果代码逻辑比较复杂，过多的 Console 信息也会让我们难以调试。这种情况就需要使用断点调试的功能，Chrome 的调试窗口会识别代码中的 debugger 关键字，并中断代码的执行。

对于复杂代码逻辑的调试来说，使用断点调试，可以让整个代码执行过程清晰可见。debugger 也是高级程序员必备的断点调试法，你一定要掌握。

### 性能相关的调试

假设遇到交互略有卡顿的情况，这时可以在调试窗口中点击 Performance 页面中的录制按钮，然后重复卡顿的操作后，点击结束，就可以清晰看到你在页面进行交互时，浏览器中性能的变化。