我们可以使用 `v-for` 指令基于一个数组来渲染一个列表

`v-for` 指令的值需要使用 `item in items` 或 `item of items` 这种特殊语法格式

其中 `items` 是源数据数组，而 `item` 是迭代项的**别名**

```html
<template>
  <!-- 
		item --- 数组值 -- 可直接解构
		index --- 索引值
	-->
	<li v-for="({ message }, index) in items">
    {{ message }} --- {{ index }}
  </li>
</template>

<script setup>
	const items = ref([{ message: 'Foo' }, { message: 'Bar' }])
</script>
```



`v-for` 变量的作用域和下面的 JavaScript 代码很类似，所以在 `v-for` 块中可以完整地访问父作用域内的属性和变量

```ts
const parentMessage = 'Parent'
const items = [
  /* ... */
]

// v-for变量本质上就是函数形参 所以可以直接执行解构操作
items.forEach(({ message }, index) => {
  console.log(parentMessage, message, index)
})
```



## 数据源

在`item in items`中，`items`是需要被迭代的数据源

在Vue中，可以使用以下任意一种类型作为`v-for`的数据源:

1.  可迭代对象
2. 普通对象
3. 正整数



### 可迭代对象

```html
<template>
  <!-- 
		item --- 数组值 -- 可直接解构
		index --- 索引值
	-->
	<li v-for="({ message }, index) in items">
    {{ message }} --- {{ index }}
  </li>
</template>

<script setup>
	const items = ref([{ message: 'Foo' }, { message: 'Bar' }])
</script>
```



### 普通对象

可以使用 `v-for` 来遍历一个对象的所有属性，属性遍历的顺序和`Object.keys(该对象)` 的返回值的顺序一致

```html
<template>
  <!-- 
		value --- 属性值
		key --- 属性名
		index --- 索引值
	-->
<li v-for="(value, key, index) in myObject">
  {{ index }}. {{ key }}: {{ value }}
</li>
</template>

<script setup>
	const items = ref([{ message: 'Foo' }, { message: 'Bar' }])
</script>
```



### 整数值

`v-for` 可以直接接受一个整数值。在这种用例中，会将该模板基于 `1...n` 的取值范围重复多次

>注意此处 `n` 的初值是从 `1` 开始而非 `0`。

```html
<span v-for="n in 10">{{ n }}</span>
```



## `<template>` 上的 `v-for`

与模板上的 `v-if` 类似，你也可以在 `<template>` 标签上使用 `v-for` 来渲染一个包含多个元素的块
```html
<ul>
  <template v-for="item in items">
    <li>{{ item.msg }}</li>
    <li class="divider" role="presentation"></li>
  </template>
</ul>
```



## `v-for` 与 `v-if`

当`v-if` 和`v-for` 同时存在于一个节点上时，`v-if` 比 `v-for` 的优先级更高

这意味着 `v-if` 的条件将无法访问到 `v-for` 作用域内定义的变量别名：

```html
<!--
 这会抛出一个错误，因为属性 todo 此时
 没有在该实例上定义
-->
<li v-for="todo in todos" v-if="!todo.isComplete">
  {{ todo.name }}
</li>
```

在外新包装一层 `<template>` 再在其上使用 `v-for` 可以解决这个问题 (这也更加明显易读)：

```html
<template v-for="todo in todos">
  <li v-if="!todo.isComplete">
    {{ todo.name }}
  </li>
</template>
```



## key

Vue 默认按照“就地更新”的策略来更新通过 `v-for` 渲染的元素列表

当数据项的顺序改变时，Vue 不会随之移动 DOM 元素的顺序，而是就地更新每个元素，确保它们在原本指定的索引位置上渲染。

默认模式是高效的，但**只适用于列表渲染输出的结果不依赖子组件状态或者临时 DOM 状态 (例如表单输入值) 的情况**。

为了给 Vue 一个提示，以便它可以跟踪每个节点的标识，从而重用和重新排序现有的元素，你需要为每个元素对应的块提供一个唯一的 `key`属性

```html
<!-- 这里的key是一个特殊属性 并不是一个props属性 -->
<template v-for="todo in todos" :key="todo.name">
  <li>{{ todo.name }}</li>
</template>
```

[推荐](https://cn.vuejs.org/style-guide/rules-essential.html#use-keyed-v-for)在任何可行的时候为 `v-for` 提供一个 `key`属性，除非所迭代的 DOM 内容非常简单 (例如：不包含组件或有状态的 DOM 元素)

`key` 绑定的值期望是一个基础类型的值，例如字符串或 number 类型。不要用对象作为 `v-for` 的 key



## 数组变化侦测

### 变更方法

Vue 能够侦听响应式数组的变更方法，并在它们被调用时触发相关的更新

这些变更方法包括：

- `push()`
- `pop()`
- `shift()`
- `unshift()`
- `splice()`
- `sort()`
- `reverse()`

### 替换一个数组

变更方法，顾名思义，就是会对调用它们的原数组进行变更。相对地，也有一些不可变 (immutable) 方法，例如 `filter()`，`concat()` 和 `slice()`，这些都不会更改原数组，而总是**返回一个新数组**

当遇到的是非变更方法时，我们需要将旧的数组替换为新的：

```ts
// `items` 是一个数组的 ref
items.value = items.value.filter((item) => item.message.match(/Foo/))
```

你可能认为这将导致 Vue 丢弃现有的 DOM 并重新渲染整个列表——幸运的是，情况并非如此。Vue 实现了一些巧妙的方法来最大化对 DOM 元素的重用，因此用另一个包含部分重叠对象的数组来做替换，仍会是一种非常高效的操作。



## 展示过滤或排序后的结果

有时，我们希望显示数组经过过滤或排序后的内容，而不实际变更或重置原始数据。在这种情况下，你可以创建返回已过滤或已排序数组的计算属性。

```ts
const numbers = ref([1, 2, 3, 4, 5])

const evenNumbers = computed(() => {
  return numbers.value.filter((n) => n % 2 === 0)
})
```

在计算属性中使用 `reverse()` 和 `sort()` 的时候务必小心！这两个方法将变更原始数组，计算函数中不应该这么做。请在调用这些方法之前创建一个原数组的副本：

```diff
- return numbers.reverse()
+ return [...numbers].reverse()
```