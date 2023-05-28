数据绑定的一个常见需求场景是操纵元素的 CSS class 列表和内联样式

因为 `class` 和 `style` 都是 attribute，我们可以和其他 attribute 一样使用 `v-bind` 将它们和动态的字符串绑定

但是，在处理比较复杂的绑定时，通过拼接生成字符串是麻烦且易出错的

因此，Vue 专门为 `class` 和 `style` 的 `v-bind` 用法提供了特殊的功能增强。除了字符串外，表达式的值也可以是对象或数组



## 绑定 HTML class

### 绑定对象

我们可以给 `:class`传递一个对象来动态切换 class

```html
<!-- active 是否存在取决于数据属性 isActive 的真假值 -->
<div :class="{ active: isActive }"></div>
```

可以在对象中写多个字段来操作多个 class。此外，`:class` 指令也可以和一般的 `class` attribute 共存

```html
<div
  class="static"
  :class="{ active: isActive, 'text-danger': hasError }"
></div>
```

绑定的对象并不一定需要写成内联字面量的形式，也可以直接绑定一个对象 或者是计算属性的返回值或函数的返回值，也就是说值是一个合法的`Record<string, boolean>`对象即可

```html
<template>
  <div :class="classObject"></div>
</template>
<script setup>
  const classObject = reactive({
    active: true,
    'text-danger': false
  })
</script>
```



### 绑定数组

我们可以给 `:class` 绑定一个数组来渲染多个 CSS class：

```html
<div :class="[activeClass, errorClass]"></div>
```

如果你也想在数组中有条件地渲染某个 class，你可以使用三元表达式

```html
<div :class="[isActive ? activeClass : '', errorClass]"></div>
```

然而，这可能在有多个依赖条件的 class 时会有些冗长。因此也可以在数组中嵌套对象

```html
<div :class="[{ active: isActive }, errorClass]"></div>
```



### 在组件上使用

对于只有一个根元素的组件，当你在组件上使用了 `class` 属性 时，这些 class 会被添加到根元素上，并与该元素上已有的 class 合并

```html
<!-- 父组件：组件调用 -->
<MyComponent class="baz boo" />

<!-- 子组件: MyComponent组件实现 -->
<p class="foo bar">Hi!</p>

<!-- 最终渲染结果 -->
<p class="foo bar baz boo">Hi!</p>
```



如果你的组件有多个根元素，你将需要指定哪个根元素来接收这个 class。你可以通过组件的 `$attrs` 属性来实现指定

```html
<!-- 父组件 -->
<MyComponent class="baz" />

<!-- 子组件 -->
<p :class="$attrs.class">Hi!</p>
<span>This is a child component</span>

<!-- 最终渲染结果 -->
<p class="baz">Hi!</p>
<span>This is a child component</span>
```



## 绑定内联样式

### 绑定对象

`:style` 支持绑定 JavaScript 对象值，对应的是 [HTML 元素的 `style` 属性](https://developer.mozilla.org/en-US/docs/Web/API/HTMLElement/style)：

```js
const activeColor = ref('red')
const fontSize = ref(30)
```

```html
<div :style="{ color: activeColor, fontSize: fontSize + 'px' }"></div>
```

尽管推荐使用 camelCase，但 `:style` 也支持 kebab-cased 形式的 CSS 属性 key (对应其 CSS 中的实际名称)

```html
<div :style="{ 'font-size': fontSize + 'px' }"></div>
```



直接绑定一个样式对象通常是一个好主意，这样可以使模板更加简洁：

同样的，如果样式对象需要更复杂的逻辑，也可以使用返回样式对象的计算属性

```js
const styleObject = reactive({
  color: 'red',
  fontSize: '13px'
})
```

```html
<div :style="styleObject"></div>
```



### 绑定数组

我们还可以给 `:style` 绑定一个包含多个样式对象的数组。这些对象会被合并后渲染到同一元素上

```html
<div :style="[baseStyles, overridingStyles]"></div>
```



### 自动前缀

当你在 `:style` 中使用了需要[浏览器特殊前缀](https://developer.mozilla.org/en-US/docs/Glossary/Vendor_Prefix)的 CSS 属性时，Vue 会自动为他们加上相应的前缀。Vue 是在运行时检查该属性是否支持在当前浏览器中使用。如果浏览器不支持某个属性，那么将尝试加上各个浏览器特殊前缀，以找到哪一个是被支持的



### 样式多值

你可以对一个样式属性提供多个 (不同前缀的) 值，如果有多个属性值都可以被使用，那么就会使用匹配到的最后一个属性值

相当于后边的属性值覆盖了之前的同名属性值

```html
<div :style="{ display: ['-webkit-box', '-ms-flexbox', 'flex'] }"></div>
```

