## ref

在组合式 API 中, 使用`ref()`来声明响应式状态

```ts
import { ref } from 'vue'

const count = ref(0)
```



Vue提供的响应式系统是一个**基于依赖追踪的响应式系统**

+ 当一个组件被首次渲染时, Vue 会将每一个使用ref对象的地方都作为其依赖，这个过程被称之为**收集依赖**
+ 当一个 ref 对象被修改时, Vue会重新渲染ref对象的所有依赖，这个过程也被称之为**触发副作用更新**



然而在标准 JavaScript 中, 并没有提供用于检测普通变量的访问或改变的原生API，但是我们可以拦截一个对象属性的获取和设置操作

为此Vue将`ref对象`设计成了一个含有`value`属性的对象，我们需要通过获取或修改ref对象的value属性来触发Vue的响应式更新

```js
// 伪代码
const myRef = {
  _value: 0,
  
  get value() {
    // 收集依赖
    track() 
    return this._value
  },
  
  set value(newValue) {
    this._value = newValue
    // 触发更新
    trigger()
  }
}
```

```html
<!-- 基本使用 -->
<script setup>
import { ref } from 'vue'

// 在JavaScript中获取和设置ref对象需要通过value属性
const count = ref(0)

function increment() {
  count.value++
}
</script>

<template>
  <!-- 在模板中使用ref对象会自动解包，不需要通过value属性 -->
  <button @click="increment">
    {{ count }}
  </button>
</template>
```



### 类型标注

默认情况下，Vue会根据初始值推导其类型

```js
import { ref } from 'vue'

// 推导出的类型：Ref<number>
const year = ref(2020)
```



如果需要给出具体的类型，以覆盖默认的推导行为, 可以使用以下方式中的任意一种

+ 可以通过使用 `Ref`类型

  ```ts
  import { ref } from 'vue'
  import type { Ref } from 'vue'
  
  // 推导出的类型：Ref<string | numbe>
  const year: Ref<string | number> = ref('2020')
  ```

+ 通过`ref()`的泛型参数

  ```ts
  import { ref } from 'vue'
  import type { Ref } from 'vue'
  
  // 推导出的类型：Ref<string | numbe>
  const year = ref<string | number>('2020')
  ```



如果在使用`ref()`的时候，没有指定默认值

+ 给出具体类型，返回值的类型就是 **具体类型和`undefined`一起构成的联合类型**
+ 没有给出具体类型，返回值的类型就是 `any`



### 深层响应式

`ref()`的参数可以是任何类型值

如果参数中存在嵌套的引用类型，则这个嵌套的引用类型会先通过`reactive()`转换为响应式对象后再被挂载

因此`ref()`返回的是一个具有深层响应式的`ref对象`

```ts
import { isReactive, isRef, ref } from 'vue'

const user = ref({
  name: 'Klaus',
  friend: {
    name: 'Alex'
  }
})

function change() {
  // 深层嵌套对象的修改是可以被检测到的
  user.value.friend.name = 'Steven'

  // isRef() 用于检测参数是不是一个ref对象
  console.log(isRef(user.value.friend)) // false

  // isReactive() 用于检测参数是不是一个reactive对象
  console.log(isReactive(user.value.friend)) // true
}
```



如果传入的深层嵌套对象不会发生改变，或者有意想要通过减少对于大对象的检测来优化性能，可以通过`shallowRef()`来创建一个浅层的ref对象

对于浅层ref对象而言，只有对顶层属性的修改才会触发响应式，也就是说只有对`value属性值`进行修改，才会触发浅层ref对象的响应式行为

```ts
import { shallowRef } from 'vue'

 const user = shallowRef({
  name: 'Klaus',
  friend: {
    name: 'Alex'
  }
 })

 // 对于嵌套对象属性的修改不会触发响应式
 function changeFriend() {
  user.value.friend.name = 'John'
 }

 // 只有对value属性的修改，才会触发响应式
 function change() {
  user.value = {
    name: 'Klaus',
    friend: {
      name: 'Steven'
    }
  }
 }
```



### 自动解包

通过以下任意一种方式使用或设置`ref对象`时，不需要借助`value`属性

1. **作为深层响应式reactive对象的属性时**: `reacitve()`会自动对其参数对象中的ref属性进行解包操作

   ```js
   const count = ref(0)
   const newCount = ref(2)
   
   const state = reactive({
     count 
   })
   
   // 不需要使用value属性
   console.log(state.count) // 0
   state.count = 1 
   
   // 自动解包行为并不会丢失ref对象原本的响应式链接
   console.log(count.value) // 1
   
   // 重新赋值会丢失ref对象原本的响应式链接
   state.count = newCount
   console.log(state.count) // 2
   console.log(count.value) // 1
   ```

   

   需要注意的是，**这种自动解包行为仅会在ref属性的值是对象类型的时候才会发生**

   如果ref属性的值是数组或`Set`、`Map`这类的集合类型时，并不会触发器自动解包行为

   ```ts
   const books = reactive([ref('Vue 3 Guide')])
   // 这里需要 .value
   console.log(books[0].value)
   
   const map = reactive(new Map([['count', ref(0)]]))
   // 这里需要 .value
   console.log(map.get('count').value)
   ```

2. **当ref对象是模板的顶级属性时，ref对象会自动执行解包操作**

   ```ts
   <!-- count是顶层属性 可以自动解包 -->
   {{ count + 1 }}
   
   <!-- object是顶层属性 而object.id不是顶层属性 -->
   {{ object.id + 1 }} <!-- 最终会被渲染为 [object Object]1 -->
   ```

3. **如果ref对象是插值语法的最终求值结果，会自动执行解包操作**

   ```ts
   <!-- 虽然 object.id 并不是顶级属性 但是其是插值语法的最终求值结果 所以会自动进行解包操作 -->
   {{ object.id }} <!-- 等价于 {{ object.id.value }} -->
   ```

   

## reactive

除了可以通过`ref()`将一个值转换为响应式状态,  我们也可以通过`reactive()`将一个值转换为响应式状态

和`ref()`不同的是，`reactive()`是直接将参数本身转换为响应式状态,

因此**`reactive()`的参数必须是类似于对象、数组这类的引用类型值**

```ts
import { reactive } from 'vue'

const state = reactive({ count: 0 })
```



`reactive()`返回的是传入的参数对应的代理对象，其使用方法和普通对象一模一样

唯一不同的是 Vue 可以拦截所有对于响应式对象属性的访问和修改, 以触发响应式行为 (依赖收集和副作用更新)

```html
<button @click="state.count++">
  {{ state.count }} 
</button>
```



`reactive()`返回的也是一个深层次的响应式状态

类似于`shallowRef()`，我们也可以通过`shallowReactive()`创建一个浅层的reactive对象



### 类型标注

默认情况下，`reactive()`可以自动根据参数进行类型推导

```ts
import { reactive } from 'vue'

// 推导得到的类型：{ title: string }
const book = reactive({ title: 'Vue 3 指引' })
```



我们也可以通过接口来指定一个更复杂的类型，以覆盖默认的类型推导行为

```ts
import { reactive } from 'vue'

interface Book {
  title: string
  year?: number
}

// 推导得到的类型：Book
const book: Book = reactive({ title: 'Vue 3 指引' })
```



`reactive对象`中如果存在ref属性，则在实际使用的时候，会自动对其ref属性执行解包操作

换句话来说，传入的泛型参数实际上指定是参数的类型，而返回值是参数类型解构后的类型，他们并不一定是同一种类型

如果需要使用泛型参数指定`reactive()`的返回值类型, ，需要借助辅助类型`UnwrapNestedRefs`

```ts
import { ref, reactive } from 'vue'
import type { Ref, UnwrapNestedRefs } from 'vue'

type User = {
  name: string;
  friend: Ref<{
    name: string
  }>
}

const user: UnwrapNestedRefs<User> = reactive<User>({
  name: 'Klaus',
  friend: ref({
    name: 'Alex'
  })
})
```



>  **为了方便起见，推荐在手动指定`reactive对象`类型时，推荐使用类型接口，而不要使用泛型参数**



### 单例代理

`reactive()`返回的是原始对象的代理对象

只有代理对象是响应式的，更改原始对象不会触发更新，所以**无论何时应该都只声明并使用原始对象的代理对象**



为了保证访问代理的统一性，代理对象被设计成了**单例模式**

```js
// 在同一个对象上调用 reactive() 会返回相同的代理
console.log(reactive(raw) === proxy) // true

// 在一个代理上调用 reactive() 会返回它自己
console.log(reactive(proxy) === proxy) // true
```

```ts
const proxy = reactive({})

const raw = {}
// 依赖于深层响应式，如果给响应式对象的属性赋值为一个原生对象
// 这个原生对象会自动被转换为响应式对象
proxy.nested = raw

console.log(proxy.nested === raw) // false
```



### 局限性

1. **参数必须是引用类型**

   `reactive()`是直接将对象类型(对象、数组和像 `Map` 和 `Set` 这样的**集合类型**)转换为代理对象

   如果参数类型是基本数据类型, `reactive()`会直接报错

2. **不能替换整个对象**

    Vue 的响应式跟踪是通过对 **对象属性的访问和修改** 来工作的

   ```ts
   let state = reactive({ count: 0 })
   
   // state的引用被重置了
   // 所以上面的引用({ count: 0 })不再被追踪 --- (失去了响应式连接!)
   state = reactive({ count: 1 })
   ```

3. **不支持解构**

   如果单独使用响应式对象的属性并且这个属性值是原始数据类型的时候，将会丢失响应式链接

   ```ts
   const state = reactive({ count: 0 })
   
   // 解构 等价于 `count = state.count` -- 丢失响应式链接
   let { count } = state 
   // 不会影响原始的 state
   count++
   
   // 对于函数而言 获取到的是 `count = state.count` 是一个number类型的实参
   // 我们必须传入整个对象才能保持响应式
   callSomeFunction(state.count)
   ```

> 因为`reactive()`的种种限制，推荐使用`ref()`来指定响应式状态



## DOM更新时机

当更改响应式状态的时候，会自动更新需要更新的DOM，但这些更新并不是立即执行的，而是会缓存下来，直到下一个`tick`在一起更新

这样就可以确保在一个tick周期中无论对状态进行了怎么样的修改，DOM都只会会执行一次更新

> `tick周期 `指代的是 在任务队列中，从开始执行第一个微任务到最后一个微任务执行完毕的这个时间周期
>
> 换句话来说就是所有的微任务执行完毕，可以准备开始执行对应的宏任务



然而有的时候，某些操作必须要等到DOM更新完毕后再执行，为此Vue提供了`nextTick`

```ts
import { nextTick } from 'vue'

async function increment() {
  count.value++

	// 当DOM更新完毕后，会立即执行nextTick中的回调
 	nextTick(() => {
		// 需要DOM更新完成后执行的操作
  }) 
}
```



```ts
import { nextTick } from 'vue'

async function increment() {
  count.value++
  
  // nextTick的返回值是一个Promise
  // 当DOM更新完毕, Promise的状态会被转换为resolved
  await nextTick()
  // 因此 需要DOM更新完成后执行的操作可以被编写在这里
}
```



----

---



当一个组件首次渲染时,Vue 会**追踪**渲染过程中使用的所有 ref。之后,当一个 ref 被改变时,它会**触发**重新渲染那些正在追踪它的组件。

在标准的 JavaScript 中,没有办法检测到普通变量的



refs 的另一个好处是与普通变量不同,你可以将 refs 传递到函数中,同时保留对最新值和响应式连接的访问。这在重构复杂逻辑到可重用代码中特别有用。



非原始值会通过 `reactive()` 转成响应式代理,这会在下面讨论。

也可以通过**浅层 refs** 来选择不进行深层响应式。对于浅层 refs,只有 `.value` 的访问会被追踪其响应式。浅层 refs 可以通过避免观测大对象的开销来优化性能,或者在内部状态由外部库管理的情况下使用。

延伸阅读:

- **减少大不可变结构的响应式开销**
- **与外部状态系统集成**
