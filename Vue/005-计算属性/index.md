虽然可以在模板中编写表达式，但是过多的表达式会让模板变得臃肿，难以维护

为此Vue提供了**计算属性用来描述依赖于响应式状态的复杂逻辑**

```ts
const publishedBooksMessage = computed(() => {
  return author.books.length > 0 ? 'Yes' : 'No'
})
```

计算属性本质上是一种特殊的ref，核心功能是基于依赖的响应式状态派生出一个新值

计算属性会自动追踪组成自己所依赖的响应式依赖，并在响应式依赖发生改变时，触发响应式更新

1. **计算属性应该被视为只读的**，如果需要修改计算属性，应直接修改计算属性所依赖的响应式状态
2. **计算属性中不应该存在任何副作用操作**, 如何需要执行副作用操作，可以通过侦听器



在这里Vue会自动识别出`publishedBooksMessage`的值依赖于`author.books`

所以当`author.books`发生改变的时候，所有依赖于 `publishedBooksMessage` 的绑定都会同时更新



### 类型标注

在默认情况下，Vue可以自动根据`computed`的getter方法的返回值推导出计算属性的类型

```ts
import { ref, computed } from 'vue'

const count = ref(0)

// 推导得到的类型：ComputedRef<number>
const double = computed(() => count.value * 2)
```

当然我们也可以通过泛型参数来显示指定`computed`的类型, 以覆盖默认的类型推导行为

```ts
const double = computed<number>(() => {
  // ...
})
```



### 计算属性和方法

在表达式中调用一个函数也会获得和计算属性相同的结果

```html
<p>{{ calculateBooksMessage() }}</p>
```

```ts
function calculateBooksMessage() {
  return author.books.length > 0 ? 'Yes' : 'No'
}
```



但是计算属性相比函数而言，有一个巨大的优势： **计算属性值会基于其响应式依赖被缓存**

一个计算属性仅会在其响应式依赖发生更新时才会被重新计算，而函数会在组件发送更新时，会被多次重复调用

```ts
// 在初始渲染的时候，为了收集依赖 now 会被执行一次
// 但是因为 Date.now() 并不是响应式状态，所以该计算属性永远都不会再次被执行
const now = computed(() => Date.now())
```



### 可写计算属性

一般情况下，计算属性是通过一些响应式状态派生出一个新值

所以只需要给计算机属性传递`getter`方法即可

```js
const publishedBooksMessage = computed(() => {
  return author.books.length > 0 ? 'Yes' : 'No'
})
```



我们也可以通过同时提供 getter 和 setter 来创建一个可读可写的计算属性

```ts
import { ref, computed } from 'vue'

const firstName = ref('John')
const lastName = ref('Doe')

const fullName = computed({
  // getter
  get() {
    return firstName.value + ' ' + lastName.value
  },
  
  // setter
  set(newValue) {
    [firstName.value, lastName.value] = newValue.split(' ')
  }
})
```





