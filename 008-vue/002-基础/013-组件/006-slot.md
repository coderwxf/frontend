在某些场景中，我们可能想要为子组件传递一些模板片段，让子组件在它们的组件中渲染这些片段，这可以通过 Vue 的自定义 `<slot>` 元素来实现

Vue 组件的插槽机制是受[原生 Web Component 元素](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/slot)的启发而诞生, 通过插槽组件变得更有灵活性和可复用性

```html
<!-- 父组件 -->
<FancyButton>
  Click me! <!-- 插槽内容 -->
</FancyButton>

<!-- 子组件 -->
<button class="fancy-btn">
  <slot /> <!-- 插槽出口 --- 本质上就是一个临时占位符 -->
</button>

<!-- 最终渲染 -->
<button class="fancy-btn">Click me!</button>
```

 `<slot>` 元素是一个**插槽出口** (slot outlet)，标示了父元素提供的**插槽内容** (slot content) 将在哪里被渲染

![插槽图示](https://cn.vuejs.org/assets/slots.dbdaf1e8.png)  

上述内容类似于JavaScript的函数调用

```ts
// 父元素传入插槽内容
FancyButton('Click me!')

// FancyButton 在自己的模板中渲染插槽内容
function FancyButton(slotContent) {
  return `<button class="fancy-btn">
      ${slotContent}
    </button>`
}
```



插槽内容可以是任意合法的模板内容，不局限于文本, 我们可以传入多个元素，甚至是组件

```html
<FancyButton>
  <span style="color:red">Click me!</span>
  <AwesomeIcon name="plus" />
</FancyButton>
```



## 渲染作用域

插槽内容可以访问到父组件的数据作用域，因为插槽内容本身是在父组件模板中定义且在父组件作用域中被解析

所以插槽内容**无法访问**子组件的数据，父组件模板中的表达式只能访问父组件的作用域；子组件模板中的表达式只能访问子组件的作用域



## 默认值

在外部没有提供任何内容的情况下，可以为插槽指定默认内容

```vue
<button type="submit">
  <slot>
    Submit <!-- 默认内容 -->
  </slot>
</button>
```



## 具名插槽

插槽的默认名称是`default`。所以只有一个插槽的时候，无需给插槽命名

但是当存在多个插槽的时候，就需要使用`name`属性给组件一个具体的名称使其成为具名插槽 (named slots)

```vue
<div class="container">
  <header>
    <slot name="header"></slot>
  </header>
  <main>
    <slot></slot>
  </main>
  <footer>
    <slot name="footer"></slot>
  </footer>
</div>
```

在父组件中，我们需要使用`v-slot` 指令将多个插槽内容传入到各自目标插槽的出口

```vue
<BaseLayout>
  <template v-slot:header>
    <!-- header 插槽的内容放这里 -->
  </template>
</BaseLayout>
```

`v-slot` 的简写是`#`, 因此 `<template v-slot:header>` 可以简写为 `<template #header>`

```vue
<BaseLayout>
  <template #header>
    <!-- header 插槽的内容放这里 -->
  </template>
</BaseLayout>
```

![具名插槽图示](https://cn.vuejs.org/assets/named-slots.ebb7b207.png) 

当一个组件同时接收默认插槽和具名插槽时，所有位于顶级的非 `<template>` 节点都被隐式地视为默认插槽的内容

```vue
<BaseLayout>
  <template #header>
    <h1>Here might be a page title</h1>
  </template>

  <!-- 隐式的默认插槽 -->
  <p>A paragraph for the main content.</p>
  <p>And another one.</p>

  <template #footer>
    <p>Here's some contact info</p>
  </template>
</BaseLayout>
```

使用 JavaScript 函数来类比具名插槽

```ts
// 传入不同的内容给不同名字的插槽
BaseLayout({
  header: `...`,
  default: `...`,
  footer: `...`
})

// <BaseLayout> 渲染插槽内容到对应位置
function BaseLayout(slots) {
  return `<div class="container">
      <header>${slots.header}</header>
      <main>${slots.default}</main>
      <footer>${slots.footer}</footer>
    </div>`
}
```



## 动态插槽名

动态插槽名是值在 `v-slot` 上使用[动态指令参数](https://cn.vuejs.org/guide/essentials/template-syntax.html#dynamic-arguments)，所以动态插槽名和动态指令参数具有相同的[语法限制](https://cn.vuejs.org/guide/essentials/template-syntax.html#directives)

```vue
<base-layout>
  <template v-slot:[dynamicSlotName]>
    ...
  </template>

  <!-- 缩写为 -->
  <template #[dynamicSlotName]>
    ...
  </template>
</base-layout>
```



## 作用域插槽

在正常情况下插槽的内容无法访问到子组件的状态，然而在某些场景下插槽的内容可能想要同时使用父组件域内和子组件域内的数据, 此时可以像对组件传递 props 那样，向一个插槽的出口上传递属性

```html
<div>
  <slot :text="greetingMessage" :count="1"></slot>
</div>
```



### 默认插槽

默认插槽通过父组件标签上的 `v-slot` 指令来接收子组件传递给父组件的插槽props对象

```vue
<MyComponent v-slot="{ text, count }">
  <!-- 可以在 v-slot 中使用解构 -->
  {{ text }} {{ count }}
</MyComponent>
```

![scoped slots diagram](https://cn.vuejs.org/assets/scoped-slots.1c6d5876.svg) 

使用 JavaScript 函数来类比默认插槽的作用域插槽

```ts
MyComponent({
  // 类比默认插槽，将其想成一个函数
  default: (slotProps) => {
    return `${slotProps.text} ${slotProps.count}`
  }
})

function MyComponent(slots) {
  const greetingMessage = 'hello'
  return `<div>${
    // 在插槽函数调用时传入 props
    slots.default({ text: greetingMessage, count: 1 })
  }</div>`
}
```



### 具名作用域插槽

具名作用域插槽的工作方式也是类似的

```html
<MyComponent>
  <template #header="headerProps">
    {{ headerProps }}
  </template>

  <template #default="defaultProps">
    {{ defaultProps }}
  </template>

  <template #footer="footerProps">
    {{ footerProps }}
  </template>
</MyComponent>
```

```vue
<!-- 插槽上的 name 是一个 Vue 特别保留的属性，不会作为 props 传递给插槽 -->
<slot name="header" message="hello"></slot>
```



如果你同时使用了具名插槽与默认插槽，则需要为默认插槽使用显式的 `<template>` 标签。尝试直接为组件添加 `v-slot` 指令将导致编译错误

```vue
<template>
  <MyComponent>
    <!-- 使用显式的默认插槽 -->
    <template #default="{ message }">
      <p>{{ message }}</p>
    </template>

    <template #footer>
      <p>Here's some contact info</p>
    </template>
  </MyComponent>
</template>
```



## 无渲染组件

无渲染组件指的是描述需要渲染的界面,但实际渲染通过子组件来进行的组件

```vue
<!-- Mouse.vue -->
<MouseTracker v-slot="{ x, y }">
  <p>Mouse is at: {{ x }}, {{ y }}</p>
</MouseTracker>
```

