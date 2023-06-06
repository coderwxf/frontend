## 模板分类

在Vue的编写中，模板可以被分为字符串模板和DOM模板 



### 字符串模板

```vue
<template>
	<!-- 单文件组件 -->
  <div>
    <h1>单文件组件</h1>
  </div>    
</template>
```

```vue
<script>
new Vue({
  el: '#string-template',
  // 内联字符串模板
  template: '<h1>字符串模板</h1>' 
})
</script>
```

```vue
<script type="text/x-template" id="my-template">
 	<h1>字符串模板</h1>
</script>

<script>
  new Vue({
    el: '#app',
    // 使用template字符串模板
    template: '#my-template'
  })
</script>
```

###  

### DOM模板

```vue
<div id="dom-template">
  <template>
    <h1>DOM模板</h1>   
  </template>      
</div>

<script>
new Vue({
  el: '#dom-template'
})   
</script>
```



## 解析限制

对于字符串模板和单文件组件中的模板在被浏览器解析之前会被Vue先进行解析，所以他们并没有什么过多的语法限制

但是对于DOM模板，Vue 则必须从 DOM 中获取模板字符串，由于浏览器的原生 HTML 解析行为限制，有一些需要注意的事项



### 区分大小写

HTML 标签和属性名称是不分大小写的，所以浏览器会把任何大写的字符解释为小写

这意味着使用DOM模板时，无论是 PascalCase 形式的组件名称、camelCase 形式的 prop 名称还是 v-on 的事件名称，都需要转换为相应等价的 kebab-case (短横线连字符) 形式



### 标签闭合

 Vue 的模板解析器支持任意标签使用 `/>` 作为标签关闭的标志

```html
<!-- 这是一个闭合标签 (self-closing tag) -->
<MyComponent />
```



然而在 DOM 模板中，我们必须显式地写出关闭标签

这是由于 HTML 只允许[一小部分特殊的元素](https://html.spec.whatwg.org/multipage/syntax.html#void-elements)省略其关闭标签，最常见的就是 `<input>` 和 `<img>`

对于其他的元素来说，如果你省略了关闭标签，原生的 HTML 解析器会认为开启的标签永远没有结束

```html
<my-component></my-component>
```



### 元素位置限制

某些 HTML 元素对于放在其中的元素类型有限制，例如 `<ul>`，`<ol>`，`<table>` 和 `<select>`

相应的，某些元素仅在放置于特定元素中时才会显示，例如 `<li>`，`<tr>` 和 `<option>`



这将导致在使用带有此类限制元素的组件时出现问题

```html
<table>
 	<!-- 自定义的组件 <blog-post-row> 将作为无效的内容被忽略 -->
  <blog-post-row></blog-post-row>
</table>
```



我们可以使用特殊的 [`is` attribute](https://cn.vuejs.org/api/built-in-special-attributes.html#is) 作为一种解决方案：

```html
<table>
  <!-- 此时浏览器会将当前元素视为web component 所以这是合法的html元素 在解析的时候vue会使用vue组件来替换当前元素 -->
  <!-- 为了避免和原生的自定义内置元素相混淆. 当使用在原生HTML元素上时，is 的值必须加上前缀 vue: -->
  <tr is="vue:blog-post-row"></tr>
</table>
```



## 组件名格式

在单文件组件和内联字符串模板中，推荐使用PascalCase作为组件名， 但是PascalCase 的标签名在 DOM 模板中是不可用的，为了方便，Vue 支持将模板中使用 kebab-case 的标签解析为使用 PascalCase 注册的组件

这意味着一个以 `MyComponent` 为名注册的组件，在模板中可以通过 `<MyComponent>` 或 `<my-component>` 引用。这让我们能够使用同样的 JavaScript 组件注册代码来配合不同来源的模板。

```vue
<script>
  import HelloWorld from './cpns/HelloWorld'
</script>

<template>
	<!-- 以下两种形式的组件都是合法的 -->
	<HelloWorld />
	<hello-world>
</template>
```

