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
console.log(double) // double 还是4
```

删除 obj.count 属性，set 函数就不会执行，double 还是之前的数值。这也是为什么在 Vue 2 中，我们需要 $delete 一个专门的函数去删除数据。

#### Proxy

Proxy 的重要意义在于它解决了 Vue 2 响应式的缺陷。

```js
let getDouble = n=>n*2
let obj = {}
let count = 1
let double = getDouble(count)
let proxy = new Proxy(obj,{
    get : function (target,prop) {
        return target[prop]
    },
    set : function (target,prop,value) {
        target[prop] = value;
        if(prop==='count'){
            double = getDouble(value)
        }
    },
    deleteProperty(target,prop){
        delete target[prop]
        if(prop==='count'){
            double = NaN
        }
    }
})
console.log(obj.count,double) // undefined 2
proxy.count = 2
console.log(obj.count,double) // 2 4
delete proxy.count
console.log(obj.count,double) // undefined NaN
```

以上代码，通过 new Proxy 代理了 obj 这个对象，然后通过

 get、set 和 deleteProperty 函数代理了对象的读取、修改和删除操作，从而实现了响应式的功能。

为了进一步理解 Proxy ，可以把 doubel 相关的

代码都写在 set 和 deleteroperty 函数里。

```js
import {reactive,computed,watchEffect} from 'vue'

let obj = reactive({
    count:1
})
let double = computed(()=>obj.count*2)
obj.count = 2

watchEffect(()=>{
    console.log('数据被修改了',obj.count,double.value)
})
```

 以上代码， Vue 3 的 relative 函数可以把对象变成响应式数据，而 relativve 就是基于 Proxy 实现的。我们还可以通过 watchEffect，在 obj.count 修改后，执行数据的打印。

#### value setter

Vue 3 中还有另一个响应式实现的逻辑，就是利用对象的 get 和 set 函数来进行监听，**这种响应式的实现方式，只能拦截某一个 属性的修改，这也是 Vue 3 中 ref 这个 API 的实现。**这也是ref使用.value的原因！

```js
let getDouble = n => n * 2
let _value = 1
double = getDouble(_value)

let count = {
  get value() {
    return _value
  },
  set value(val) {
    _value = val
    double = getDouble(_value)

  }
}
console.log(count.value,double)
count.value = 2
console.log(count.value,double)
```

以上代码，我们拦截了 count 的 value 属性，并且拦截了 set 操作，也实现了类似的功能。

#### 三种原理的对比

![img](https://static001.geekbang.org/resource/image/b5/11/b5344de85923a2ba8bea60283b491711.png?wh=1336x650)

### 定制数据响应式

[详见项目](https://github.com/KurjaScript/geek-admain/tree/10809feafb80a8cb40c3c4a2091253b2010a5b56)封装 useStorage 函数

我们可以把日常开发中用到的数据，无论是浏览器的本地存储，还是网络数据，都封装成响应式数据，统一使用响应式数据开发的模式。这样，我们开发项目的时候，只需要修改对应的数据就可以了。

![img](https://static001.geekbang.org/resource/image/5a/0e/5a5yy5dc6f6b25f1c1ff8f3a434cd10e.jpg?wh=2316x1829)
