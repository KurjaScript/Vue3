![img](https://static001.geekbang.org/resource/image/6f/0a/6fd86f3d33a0200d64c7423bc88e890a.png?wh=1222x432)

### Composition API 和 `<script setup>`

`<script set>` 会自动把引入的组件注册到当前组件

在后续的课程中，会详细介绍基于组件去搭建应用的方法，**可以实现业务对逻辑的复用。**这样做的好处是，如果有其他页面需要用到这个功能，可以直接复用过去。

### 计算属性

在 Composition API 的语法中，计算属性和生命周期等功能，都可以脱离 Vue 的组件机制单独使用。

<template>
  <div>
  	<input type="text" v-model="title" @keydown.enter="addTodo" />
    <button v-if="active < all" @click="clear">清理</button>
    <ul v-if="todos.length">
      <li v-for="todo in todos">
      	<input type="checkbox" v-model="todo.done" />
        <span :class="{ done: todo.done }">{{ todo.title }}</span>
      </li>
    </ul>
    <div v-else>暂无数据</div>
    <div>
      全选<input type="checkbox" v-model="allDone" />
      <span>{{active}} / {{all}}</span>
    </div>
  </div>
</template>

<script setup>
  import { ref, computed } from "vue";
  let title = ref("")
  let todos = ref([{title:'学习Vue', done: false}])
  function addTodo() {
    ...
  };
  function clear() {
    todos.value = todos.value.filter(v => !v.done);
  };
  let active = computed(() => {
    return todos.value.filter(v => !v.done).length
  });
  let all = computed(() => {
    return todos.value.length
  });
  let allDone = computed({
    get: function() {
      return active.value === 0;
    },
    set: function(value) {
      todos.value.forEach(todo => {
        todo.done = value
      })
    }
  })
</script>

在这段代码中，computed 被单独引入使用，而不是组件的一个配置项。

### Composition API 拆分代码

使用 Composition API 的逻辑来拆分代码，把一个功能相关的数据都维护在一起。

但是，把所有功能代码都写在一起的话，也会带来一些问题：随着功能越来越复杂，script 内部的代码也会越来越多。因此需要对代码进一步拆分，把独立的模块封装成一个函数，真正做到**按需拆分**。

新建一个函数[关于](http://localhost:3000/#/about)

```js
function useTodos() {
  let title = ref("");
  let todos = ref([{ title: "学习Vue", done: false }]);
  function addTodo() {
    todos.value.push({
      title: title.value,
      done: false,
    });
    title.value = "";
  }
  function clear() {
    todos.value = todos.value.filter((v) => !v.done);
  }
  let active = computed(() => {
    return todos.value.filter((v) => !v.done).length;
  });
  let all = computed(() => todos.value.length);
  let allDone = computed({
    get: function () {
      return active.value === 0;
    },
    set: function (value) {
      todos.value.forEach((todo) => {
        todo.done = value;
      });
    },
  });
  return { title, todos, addTodo, clear, active, all, allDone };
}
```



**因为 ref 和 computed 等功能都可以从 Vue 中全局引入，所以就可以把组建任意颗粒化地拆分和组合。**这样大大地提高了代码的可维护性和复用性。

### style 样式的特性

如果在 scoped 内部，你还想写全局的样式，可以用:global 来标记。

可以通过 v-bind 函数，直接在 CSS 中使用 JavaScript 中的变量。

```vue
<template>
  <div>
    <h1 @click="add">{{ count }}</h1>
  </div>
</template>

<script setup>
import { ref } from "vue";
let count = ref(1)
let color = ref('red')
function add() {
  count.value++
  color.value = Math.random()>0.5? "blue":"red"
}
</script>

<style scoped>
h1 {
  color:v-bind(color);
}
</style>>
```

### 总结

- 在 Composition API 的语法中，所有的功能都是通过全局引入的方式使用的，并且通过 `<script setup> `的功能，我们定义的变量、函数和引入的组件，都不需要额外的生命周期，就可以直接在模板中使用。
- Composition API 可以使开发者任意拆分组件的功能，抽离出独立的工具函数，大大提高了代码的可维护性。
- 通过在 style 标签里标记 scoped 可以让样式只在当前的组件内部生效，还可以通过 v-bind 函数来使用 JavaScript 中的变量去渲染样式，如果这个变量是响应式数据，就可以很方便地实现样式的切换。
