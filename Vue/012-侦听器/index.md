计算属性允许我们声明性地计算衍生值。然而在有些情况下，我们需要在状态变化时执行一些“副作用”：例如更改 DOM，或是根据异步操作的结果去修改另一处的状态。

在组合式 API 中，我们可以使用 [`watch` 函数](https://cn.vuejs.org/api/reactivity-core.html#watch)在每次响应式状态发生变化时触发回调函数：

```vue
<script setup>
import { ref, watch } from 'vue'

const question = ref('')
const answer = ref('Questions usually contain a question mark. ;-)')

// 可以直接侦听一个 ref
watch(question, (newQuestion, oldQuestion) => {
  // ...
  }
})
</script>

<template>
  <p>
    Ask a yes/no question:
    <input v-model="question" />
  </p>
  <p>{{ answer }}</p>
</template>
```



## 侦听数据源类型

`watch` 的第一个参数可以是不同形式的“数据源”

1. 一个 ref (包括计算属性)

```ts
import { watch, ref } from 'vue'

const count = ref(0)

function change() {
  count.value++
}

// 如果监听值是ref，在监听函数中获取的newValue和oldValue都是解包后的值
watch(count, v => console.log(v, typeof v)) // 1 number
```



2. 一个响应式对象

```ts
const obj = reactive({ count: 0 })

// 不能直接侦听响应式对象的属性值
// 因为 typeof obj.count是number 并不是响应式状态 不会触发响应式更新
watch(() => obj.count, (count) => {
  console.log(`count is: ${count}`)
})

// 如果需要监听响应式对象的属性值 可以借助getter函数
watch(() => obj.count, (count) => {
  console.log(`count is: ${count}`)
})
```





3. 一个 getter 函数

```ts
import { watch, ref } from 'vue'

const x = ref(0)
const y = ref(0)

function change() {
  x.value++
  y.value++
}

// 如果是getter函数 会将getter函数的返回值 传递给监听函数
watch(() => x.value + y.value, v => console.log(v, typeof v)) // 2 number
```



4. 多个数据源组成的数组

```js
import { watch, ref } from 'vue'

const x = ref(0)
const y = ref(0)

function change() {
  x.value++
  y.value++
}

// 如果传递多个值 那么多个数据源以数组的形式进行传递
// 在监听函数中 接收到的数据也是一个数组，并且对应位置是一一对应的
watch([x, () => y.value], ([x, y] )=> {
  console.log(x, typeof x) // 1 number
  console.log(y, typeof y) // 1 number
})
```



## 深层侦听器

直接给 `watch()` 传入一个响应式对象，会隐式地创建一个深层侦听器——该回调函数在所有嵌套的变更时都会被触发：

```ts
const obj = reactive({ count: 0 })

watch(obj, (newValue, oldValue) => {
  // 在嵌套的属性变更时触发
  // 注意：`newValue` 此处和 `oldValue` 是相等的
  // 因为它们是同一个对象！
})

obj.count++
```

相比之下，一个返回响应式对象的 getter 函数，只有在返回不同的对象时，才会触发回调：

```ts
watch(
  () => state.someObject,
  () => {
    // 仅当 state.someObject 被替换时触发
  }
)
```



你也可以给上面这个例子显式地加上 `deep` 选项，强制转成深层侦听器：

深度侦听需要遍历被侦听对象中的所有嵌套的属性，当用于大型数据结构时，开销很大。因此请只在必要时才使用它，并且要留意性能

```ts
watch(
  () => state.someObject,
  (newValue, oldValue) => {
    // 注意：`newValue` 此处和 `oldValue` 是相等的
    // **除非 state.someObject 被整个替换了**
  },
  { deep: true }
)
```



## 即时回调的侦听器

`watch` 默认是懒执行的：仅当数据源变化时，才会执行回调

但在某些场景中，我们希望在创建侦听器时，立即执行一遍回调

我们可以通过传入 `immediate: true` 选项来强制侦听器的回调立即执行：

```ts
watch(source, (newValue, oldValue) => {
  // 立即执行，且当 `source` 改变时再次执行
}, { immediate: true })
```



## `watchEffect()`

侦听器需要监听的变量和监听回调中使用的变量是同一个的情况非常常见

`todoId`被调用了两次，一次是作为源，另一次是在回调中，可以用 [`watchEffect` 函数](https://cn.vuejs.org/api/reactivity-core.html#watcheffect) 来简化上面的代码

```ts
const todoId = ref(1)
const data = ref(null)

watch(todoId, async () => {
  const response = await fetch(
    `https://jsonplaceholder.typicode.com/todos/${todoId.value}`
  )
  data.value = await response.json()
}, { immediate: true })
```

`watchEffect()` 会自动跟踪回调中的响应式依赖，上面的侦听器可以重写为：

```ts
watchEffect(async () => {
  const response = await fetch(
    `https://jsonplaceholder.typicode.com/todos/${todoId.value}`
  )
  data.value = await response.json()
})
```

`watchEffect()`的回调会立即执行，不需要指定 `immediate: true`

在执行期间，它会自动追踪 `todoId.value` 作为依赖（和计算属性类似）

每当 `todoId.value` 变化时，回调会再次执行

有了 `watchEffect()`，我们不再需要明确传递 `todoId` 作为源值



对于这种只有一个依赖项的例子来说，`watchEffect()` 的好处相对较小，但是对于有多个依赖项的侦听器来说，使用 `watchEffect()` 可以消除手动维护依赖列表的负担

此外，如果你需要侦听一个嵌套数据结构中的几个属性，`watchEffect()` 可能会比深度侦听器更有效，因为它将只跟踪回调中被使用到的属性，而不是递归地跟踪所有的属性



`watchEffect` 仅会在其**同步**执行期间，才追踪依赖。在使用异步回调时，只有在第一个 `await` 正常工作前访问到的属性才会被追踪。



### `watch` vs. `watchEffect`

`watch` 和 `watchEffect` 都能响应式地执行有副作用的回调。它们之间的主要区别是追踪响应式依赖的方式：

- `watch` 只追踪明确侦听的数据源。它不会追踪任何在回调中访问到的东西。另外，仅在数据源确实改变时才会触发回调。`watch` 会避免在发生副作用时追踪依赖，因此，我们能更加精确地控制回调函数的触发时机。
- `watchEffect`，则会在副作用发生期间追踪依赖。它会在同步执行过程中，自动追踪所有能访问到的响应式属性。这更方便，而且代码往往更简洁，但有时其响应性依赖关系会不那么明确。



## 回调的触发时机

当你更改了响应式状态，它可能会同时触发 Vue 组件更新和侦听器回调

默认情况下，用户创建的侦听器回调，都会在 Vue 组件更新**之前**被调用。这意味着你在侦听器回调中访问的 DOM 将是被 Vue 更新之前的状态。

如果想在侦听器回调中能访问被 Vue 更新**之后**的 DOM，你需要指明 `flush: 'post'` 选项：

```ts
watch(source, callback, {
  flush: 'post'
})

watchEffect(callback, {
  flush: 'post'
})
```

后置刷新的 `watchEffect()` 有个更方便的别名 `watchPostEffect()`：

```js
import { watchPostEffect } from 'vue'

watchPostEffect(() => {
  /* 在 Vue 更新后执行 */
})
```



## 停止侦听器

在 `setup()` 或 `<script setup>` 中用同步语句创建的侦听器，会自动绑定到宿主组件实例上，并且会在宿主组件卸载时自动停止。因此，在大多数情况下，你无需关心怎么停止一个侦听器。

一个关键点是，侦听器必须用**同步**语句创建：如果用异步回调创建一个侦听器，那么它不会绑定到当前组件上，你必须手动停止它，以防内存泄漏

+ 同步的监听器会绑定到组件实例上并在组件被卸载的时候自动销毁
+ 非同步的监听器是游离的状态，就是普通的函数存在于内存中，所以需要通过去返回的函数找到该绑定并手动销毁

```ts
<script setup>
import { watchEffect } from 'vue'

// 它会自动停止
watchEffect(() => {})

// ...这个则不会！
setTimeout(() => {
  watchEffect(() => {})
}, 100)
</script>
```

要手动停止一个侦听器，请调用 `watch` 或 `watchEffect` 返回的函数：

```js
const unwatch = watchEffect(() => {})

// ...当该侦听器不再需要时
unwatch()
```

注意，需要异步创建侦听器的情况很少，请尽可能选择同步创建。如果需要等待一些异步数据，你可以使用条件式的侦听逻辑：

```ts
// 需要异步请求得到的数据
const data = ref(null)

watchEffect(() => {
  if (data.value) {
    // 数据加载后执行某些操作...
  }
})
```

