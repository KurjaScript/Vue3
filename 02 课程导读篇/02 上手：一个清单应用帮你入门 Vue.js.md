首先上一段 jQuery代码：

```html
<div>
    <h2 id="app"></h2>
    <input type="text" id="todo-input">
</div>
<script src="jquery.min.js"></script>
<script>
    // 找到输入框，监听输入
    $('#todo-input').on('input',function(){
        let val = $(this).val() // 获取值
        $('#app').html(val) // 找到标签，修改内容
    })
</script>
```

上述代码是 jQuery 时代开发逻辑的一个缩影。**jQuery 时代的开发逻辑，就是我们先要找到目标元素，然后再进行对应的修改。**

学习 Vue.js，就**不要在考虑页面元素的操作了，而是要思考数据是怎么变化的。**这就意味着，我们只需要操作数据，至于数据和页面的同步问题，Vue 会帮我们处理。

在 Vue 框架下，如果你想要在页面显示数据，就要先在 data 里声明数据；在输入框代码里，使用 v-model 来标记输入框和数据的同步；在 HTML 模板里，使用两个花括号标记，来显示数据，例如 {{title}}.对应的代码如下：

```vue
<div id="app">
  <h2>{{title}}</h2>
  <input type="text" v-model="title">
</div>

<script src="https://unpkg.com/vue@next"></script>
<script>
const App = {
  data() {
    return {
      title: "" // 定义一个数据
    }
  }
}
// 启动应用
Vue.createApp(App).mount('#app')
</script>
```

功能完善后的代码如下，具体的演变过程请参考Try : 2-2

```html
<div id="app">
  <h2>{{title}}</h2>
  <input type="text" v-model="title" @keydown.enter="addTodo">
  <button v-if="active < all" @click="clear">清理</button>
  <ul>
    <li v-for="todo in todos">
      <input type="checkbox" v-model="todo.done">
      <span :class="{done: todo.done}">{{todo.title}}</span>
    </li>
  </ul>
  <div>
    全选<input type="checkbox" v-model="allDone">
    {{active}} / {{all}}
  </div>
</div>

<script src="https://unpkg.com/vue@next"></script>
<script>
const App = {
  data() {
    return {
      title: "" ,// 定义一个数据
      todos:[
        {title: '吃饭', done: false},
        {title: '睡觉', done: true},
        {title: '学习', done: false}
      ]
    }
  },
  computed: {
    active(){
      return this.todos.filter(v => !v.done).length
    },
    all(){
      return this.todos.length
    },
    allDone: {
      // 在 get 函数里，对于 allDone 会返回什么值，只需要判断计算属性 active 是不是 0 就可以
      get: function() {
        return this.active === 0
      },
      // set 函数做到的就是，在修改 allDone，也就是前端页面切换全选框时，直接遍历 todos，把里面的 done 字段直接和 allDone 同步即可
      set: function(val) {
        this.todos.forEach(todo => {
          todo.done = val
        })
      }
    }
  },
  methods: {
    addTodo() {
      this.todos.push({
        title: this.title,
        done: false
      })
      this.title = ""
    },
    clear() {
      this.todos = this.todos.filter(v => !v.done)
    }
  }
}
// 启动应用
Vue.createApp(App).mount('#app')
</script>
<style scoped>
  .done {
    color: gray;
    text-decoration: line-through;
  }
</style>
```

