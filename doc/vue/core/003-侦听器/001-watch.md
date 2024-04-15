`watch(源, callback)`: 在状态变化时执行一些“副作用”



Vue 的副作用通常指的是那些影响或依赖于组件外部状态的操作

如果您在 `setup` 函数或任何其他组件逻辑中操作了组件外部的状态（例如，修改父组件的状态、全局状态或进行网络请求等），那么这些操作可以被认为是副作用



watch 源的类型:

1.  ref 
2. reactive
3. getter 函数
4. 多个数据源组成的数组

```js
const x = ref(0)
const y = ref(0)
const obj = reactive({ count: 0 })

// 单个 ref
watch(x, (newX) => {
  console.log(`x is ${newX}`)
})

// getter 函数
watch(
  () => x.value + y.value,
  (sum) => {
    console.log(`sum of x + y is: ${sum}`)
  }
)

// reactive对象 -- 提供一个 getter 函数
watch(
  () => obj.count,
  (count) => {
    console.log(`count is: ${count}`)
  }
)

// 多个来源组成的数组
watch([x, () => y.value], ([newX, newY]) => {
  console.log(`x is ${newX} and y is ${newY}`)
})
```



不要直接通过点运算符获取响应式对象的值，或直接解构响应式对象的属性

```js
const obj = reactive({ count: 0 })

obj.count // number 不再是响应式的了

const { count } = obj // 也不再是响应式的了
```



## 配置对象

`deep`

```js
const obj = reactive({ count: 0 })

// 在监听reactive对象时，会自动开启deep: true
// 其余情况下，deep皆为false
watch(obj, (newValue, oldValue) => {
  // 在嵌套的属性变更时触发
  // 注意：`newValue` 此处和 `oldValue` 是相等的, 因为它们是同一个对象！
})

obj.count++
```

```js
watch(
  () => state.someObject,
  (newValue, oldValue) => {
    // 注意：`newValue` 此处和 `oldValue` 是相等的
    // *除非* state.someObject 被整个替换了
  },
  { deep: true }
)
```



`immediate`

`watch` 默认是懒执行的：仅当数据源变化时，才会执行回调

我们可以通过传入 `immediate: true` 选项来强制侦听器的回调立即执行

```jsx
watch(
  source,
  (newValue, oldValue) => {
    // 立即执行，且当 `source` 改变时再次执行
  },
  { immediate: true }
)
```



`once`

如果希望回调只在源变化时触发一次，请使用 `once: true` 选项。

```js
watch(
  source,
  (newValue, oldValue) => {
    // 当 `source` 变化时，仅触发一次
  },
  { once: true }
)
```
