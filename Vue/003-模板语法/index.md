Vue 使用一种基于 HTML 的模板语法，使我们能够声明式地将组件实例的数据绑定到呈现的 DOM 上

所有的 Vue 模板都是语法层面合法的 HTML，可以被符合规范的浏览器和 HTML 解析器解析



在底层机制中，Vue 会将模板编译成高度优化的 JavaScript 代码

结合响应式系统，当应用状态变更时，Vue 能够智能地推导出需要重新渲染的最少组件个数，并应用最少的 DOM 操作



Vue推荐使用模板来描述页面结构，当我们也可以通过JSX来描述页面结构，甚至是以编写渲染函数的形式来描述页面结构

但请注意，这将不会享受到和模板同等级别的编译时优化



## 插值

可以使用双大括号语法(`mustache`语法)来设置元素的`textContent`

在编译后，双大括号中的内容会被替换为组件实例中对应的状态，并继续保持响应式

换句话来说，每当状态`msg`的值发生了更新，Vue会自动更新和该状态相关的所有模板

```html
<span>Message: {{ msg }}</span>
```



## 指令

`mustache`语法只能用来设置元素的`textContent`，如果需要设置元素的`innerHTML`或者将响应式状态设置为元素属性值，则需要使用指令

指令是Vue特有的，带有`v-`前缀的特殊属性，基本功能是在其表达式值发生改变的时候，响应式的更新DOM，并运用特殊行为

![完整的指令语法格式](https://vuejs.org/assets/directive.69c37117.png) 

### 指令值

大部分指令所期望的属性值是一个合法的JavaScript表达式 ( 少数除外 如`v-for`、`v-on` 和 `v-slot` )

指令值应该为字符串类型值或者`null`

如果值为`null`，表示显示移除该指令，其他非字符串的值会触发警告



### 指令参数

在指令名后通过一个冒号隔开的标识符就被称之为指令参数(Arguments) 

```vue
<!-- href就是指令参数 -->
<a v-bind:href="url"> ... </a>
```



#### 动态参数

指令参数也可以是一个合法的JavaScript表达式

如果指令参数是一个JavaScript表达式的时候，该参数就被称之为动态参数

动态参数一般被包裹在一对方括号内

```html
<a :[attributeName]="url"> ... </a>
```



Vue的模板应该是合法的HTML，可以直接被浏览器所解析

所以因为浏览器解析规则的限制，动态参数必须是合法的HTML属性名

换句话说，如果动态参数中需要使用逗号或者引号等字符，需要通过计算属性将其进行包裹，以确保动态参数是合法的HTML属性名



### 修饰符

修饰符(modifiers)是以点开头的特殊后缀，表明指令需要以一些特殊的方式被绑定

例如 `.prevent` 修饰符会告知 `v-on` 指令对触发的事件调用 `event.preventDefault()`



### 常见指令

#### `v-html`

`mustache`语法只能用于设置元素的`textContent`，如果需要设置元素的`innerHTML`，则需要使用`v-html`

```html
<p>Using text interpolation: {{ rawHtml }}</p>
<p>Using v-html directive: <span v-html="rawHtml"></span></p>
```

![v-html](https://s2.loli.net/2023/07/17/GwxI78NOtAlshW3.png)

`v-html`所做的事情其实很简单，就是将对应指令值设置为当前元素的`innerHTML`

在整个过程中，Vue并不会对指令值进行任何的转换

所以我们应该确保对应html格式字符串的来源是绝对可靠的，以避免造成 [XSS 漏洞](https://en.wikipedia.org/wiki/Cross-site_scripting)



#### `v-bind`

双大括号不能在 HTML 属性中使用。想要响应式地绑定一个属性，应该使用 [`v-bind` 指令](https://cn.vuejs.org/api/built-in-directives.html#v-bind)

如果绑定的值是 `null` 或者 `undefined`，那么就会显示的移除该属性

```html
<div v-bind:id="dynamicId"></div>
```



因为 `v-bind` 非常常用，所以可以被简写为`:`

```html
<div :id="dynamicId"></div>
```



##### 布尔类型属性

布尔类型属性是根据值的` true / false`来决定属性是否应该存在于对应元素上

```html
<button :disabled="isButtonDisabled">Button</button>
```

+ 当`isButtonDisabled`的值是`真值`或`空字符串`，会存在`disabled`属性
+ 其余情况，`disabled`属性会被自动移除



##### 动态绑定多个值

```html
<template>
	<div v-bind="objectOfAttrs"></div>
</template>

<script setup>
  const objectOfAttrs = {
    id: 'container',
    class: 'wrapper'
  }
</script>
```

如果使用不带参数的 `v-bind`，在解析时Vue会自动将对象中属性逐个绑定到对应元素上

```html
<!-- 最终被渲染为 -->
<div id="container" class="wrapper"></div>
```



### JavaScript表达式

在Vue中，任何可以支持数据绑定的地方都支持使用完整的JavaScript表达式

在数据绑定的时候**仅支持单一表达式**，也就是一段能够被求值的 JavaScript 代码

一个最为简单的判断方法是查看该表达式能否合法地被写在 `return` 关键字后面



在Vue中，每个组件都是独立可复用的，拥有自己独立的组件作用域

因此在数据绑定中的任何合法的表达式都会以当前组件作为作用域进行解析执行



模板中的表达式将被沙盒化，仅能够访问到[有限的全局对象列表](https://github.com/vuejs/core/blob/main/packages/shared/src/globalsAllowList.ts#L3) 

没有显式包含在列表中的全局对象将不能在模板内表达式中访问

有限的全局对象列表可以通过 [`app.config.globalProperties`](https://cn.vuejs.org/api/application.html#app-config-globalproperties) 进行自定义扩展



函数调用可以被认为是一种合法的JavaScript表达式，所以我们可以在绑定的表达式中使用一个组件暴露的方法

绑定在表达式中的方法在组件每次更新时都会被重新调用，因此**不**应该产生任何副作用

```html
<time :title="toTitleDate(date)" :datetime="date">
  {{ formatDate(date) }}
</time>
```



在 Vue 模板内，JavaScript 表达式可以被使用在如下场景上：

+ 在文本插值 (双大括号) 中
+ 在任何 Vue 指令的指令值中
+ 在动态指令参数中

