### 什么是响应式

![img](https://static001.geekbang.org/resource/image/5c/97/5c9a7aa3468f19b7edf067b7b252ea97.jpg?wh=1090x970)

### 响应式原理

Vue 中用过三种响应式解决方案，分别是  defineProperty、 Proxy 和 value setter.

#### defineProperty API

```js
let getDouble = n=>n*2
let obj = {}
let count = 1
let double = getDouble(count)

Object.defineProperty(obj,'count',{
    get(){
        return count
    },
    set(val){
        count = val
        double = getDouble(val)
    }
})
console.log(double)  // 打印2
obj.count = 2
console.log(double) // 打印4  有种自动变化的感觉
```

以上代码定义了一个对象 obj，使用 defineProperty 代理了 count 属性。 这样就对 obj 的 value 属性实现了拦截，读取 count 属性的时候执行 get 函数，修改 count 属性的时候执行 set 函数，并在 set 函数内部重新计算了 doubel。

defineProperty API 语法中也有缺陷，比如以下代码：

```js
delete obj.count
console.log(double) // doube还是4
```

删除 obj.count 属性，set 函数就不会执行，double 还是之前的数值。这也是为什么在 Vue 2 中，我们需要 $delete 一个专门的函数去删除数据。
