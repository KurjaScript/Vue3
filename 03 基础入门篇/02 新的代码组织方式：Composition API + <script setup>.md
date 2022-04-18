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