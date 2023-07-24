数据绑定最常见的应用场景就是样式绑定，包括class属性和内联样式

但是使用`v-bind`进行动态字符串拼接时，非常麻烦且容易出错

因此，Vue 专门为 `class` 和 `style` 的 `v-bind` 用法提供了特殊的功能增强

除了`字符串形式`外，表达式的值也可以是`对象或数组`



## class

### 对象格式

```html
<!-- 
	key --- 样式名
	value --- 布尔值 
			--- 为true时 对应样式将被添加到样式列表中
			--- 为false时，对应样式将从样式列表中被移除
-->
<div :class="{ active: isActive }"></div>
```

`:class` 指令可以和一般的 `class` attribute 共存

```html
<div
  class="static"
  :class="{ active: isActive, 'text-danger': hasError }"
></div>
```

`:class`的值是一个`Record<string, boolean>`格式的对象，所以除了上述这种写成内联对象外

我们也可以将其抽离成一个响应式状态，或者计算属性或函数调用的返回值



### 数组格式

可以给 `:class` 绑定一个数组来渲染多个 CSS class

```ts
const activeClass = ref('active')
const errorClass = ref('text-danger')
```

```html
<div :class="[activeClass, errorClass]"></div>

<!-- 渲染结果 -->
<div class="active text-danger"></div>
```

数组中的每一项都会被视为一个独立的JavaScript表达式，并单独求值

```html
<div :class="[isActive ? activeClass : '', errorClass]"></div>

<div :class="[{ active: isActive }, errorClass]"></div>
```



### 在组件上使用

1. 对于单根组件，class属性会被添加到根元素上并与该元素上已有的 class 合并

2. 对于多根组件，calss属性需要通过`$attrs`手动指定那个根元素来接收该样式

   ```html
   <!-- 父组件 -->
   <Child class="baz foo" />
   
   <!-- 子组件 -->
   <p :class="$attrs.class">Hi!</p>
   <span>This is a child component</span>
   
   <!-- 会被渲染为 -->
   <p :class="baz foo">Hi!</p>
   <span>This is a child component</span>
   ```

   

## 内联样式

### 对象格式

```html
<template>
	<!-- 
		样式名推荐使用camelCase格式进行定义
		Vue也支持kebab-cased格式的属性名，在编译时会自动转换为camelCase格式
	-->
	<div :style="{ color: activeColor, fontSize: fontSize + 'px' }"></div>
</template>

<script setup>
	const activeColor = ref('red')
	const fontSize = ref(30)
</script>
```

同样也可以将样式抽取成一个对象，计算属性或者函数的返回值

```html
<template>
	<div :style="styleObject"></div>
</template>

<script setup>
	const styleObject = reactive({
    color: 'red',
    fontSize: '13px'
  })
</script>
```



### 数组格式

可以给 `:style` 绑定一个包含多个样式对象的数组。这些对象会被合并后渲染到同一元素上

```html
<div :style="[baseStyles, overridingStyles]"></div>
```



## 样式语法补充

### 自动前缀

Vue会在运行时检查样式属性是否支持在当前渲染浏览器中使用

如果浏览器不支持某个属性，那么Vue将自动尝试加上各个浏览器特殊前缀，以找到哪一个是被支持的



### 样式多值

可以对一个样式属性提供多个 (不同前缀的) 值, Vue会使用渲染浏览器所支持的最后一个值

```html
<div :style="{ display: ['-webkit-box', '-ms-flexbox', 'flex'] }"></div>
```

在这个示例中，在支持不需要特别前缀的浏览器中都会渲染为 `display: flex`
