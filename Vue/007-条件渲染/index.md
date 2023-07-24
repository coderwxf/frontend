## v-if

`v-if` 指令用于条件性地渲染一块内容。

`v-if`是惰性的，`v-if`包裹的内容只会在指令的表达式返回真值时才被渲染

```html
<div v-if="type === 'A'">
  A
</div>
<div v-else-if="type === 'B'">
  B
</div>
<!-- 一个使用 v-else-if 的元素必须紧跟在一个 v-if 或一个 v-else-if 元素后面 -->
<div v-else-if="type === 'C'">
  C
</div>
<!-- 一个 v-else 元素必须跟在一个 v-if 或者 v-else-if 元素后面 -->
<div v-else>
  Not A/B/C
</div>
```



### `<template>` 上的 `v-if`

`v-if`必须依附于某个元素, 如果需要切换多个元素的时候，就需要结合`template`一起使用

 `<template>`是HTML原生提供了一个不可见的包装器元素，并不会存在于最终的渲染结果中

```html
<template v-if="ok">
  <h1>Title</h1>
  <p>Paragraph 1</p>
  <p>Paragraph 2</p>
</template>
```



## `v-show`

`v-show`的基本用法和`v-if`一模一样，都可以用来按条件显示一个元素

不同的是，`v-show` 仅切换了该元素上名为 `display` 的 CSS 属性，所以在整个渲染过程中，该元素将一直存在于DOM结构中

`v-show` 不支持在 `<template>` 元素上使用，也不能和 `v-else` 搭配使用

```html
<h1 v-show="ok">Hello!</h1>
```



## `v-if` VS `v-show`

`v-if `是 “真实的” 按条件渲染，因为它确保了在切换时，条件区块内的事件监听器和子组件都会被销毁与重建

`v-if` 也是**惰性**的，如果在初次渲染时条件值为 false，则不会做任何事。条件区块只有当条件首次变为 true 时才被渲染



相比之下，使用了`v-show`的元素无论初始条件如何，始终都会被渲染，只有 CSS `display` 属性会被切换



总的来说，`v-if` 有更高的切换开销，而 `v-show` 有更高的初始渲染开销

如果需要频繁切换，则使用 `v-show` 较好；如果在运行时绑定条件很少改变，则 `v-if` 会更合适



## `v-if` 和 `v-for`

同时使用 `v-if` 和 `v-for` 是**不推荐的**，因为这样二者的优先级不明显

当 `v-if` 和 `v-for` 同时存在于一个元素上的时候，`v-if` 会首先被执行

