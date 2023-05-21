## 简介

Tailwind 是一个 CSS 框架，通过提供一系列的功能类，可以在 HTML 中可以直接使用这些类来实现样式的编写

与传统的 CSS 框架不同的是，Tailwind 的功能类可以添加伪类和响应式断点，并且可以使用 `@apply` 指令实现样式的复用

Tailwind 在编译时通过正则表达式提取所有位于 HTML 文件、JavaScript 组件和其他模板中的类名，生成相应的样式并将它们写入静态 CSS 文件，因此快速、灵活、可靠，并且没有运行时开销

Tailwind CSS 非常注重性能，通过仅生成项目实际使用的 CSS（Tree Shaking）来生成尽可能小的 CSS 文件，并结合缩小和网络压缩技术，通常可以产生小于 10kB 的 CSS 文件，即使对于大型项目也是如此

这意味着不必担心复杂的解决方案（如 code splitting），只需一次下载并缓存单个小型 CSS 文件，直到重新部署站点即可

Tailwind所处的功能只有将功能类按需转换为css，并不会处理添加浏览器前缀等css后续优化操作



## 核心概念

### 功能类优先

使用 Tailwind，可以通过直接在 HTML 中应用预定义的类来为元素添加样式

1. **不需要再花费时间来自定义类名**，尤其是为那些没有实际意义的容器添加样式的时候

   

2. 使用传统方法，每次添加新功能时，CSS的体积会越来越大

   有了实用程序，一切都是可重用的(CSS 模块化)，因此**CSS体积增长到一定程度就会停止增长**

   

3. 实用程序类可以避免在应用程序的不同组件和页面中出现样式不一致和样式冲突的问题

   因为所有样式都基于同一个设计系统，从而使整个应用程序中的元素和组件之间的**样式值保持一致，提高样式的可重用性和可维护性**

   这让开发者更加专注于应用程序的功能和业务逻辑，而不是样式的管理和调试

   

4. **使用预定义的功能类在 HTML 中工作，修改更加安全**

   传统的CSS是通过选择器来添加样式的，一个HTML元素可能被应用多个选择器，而这多个选择器之间可能出现样式冲突

   Tailwind使用功能类堆砌的方式为元素添加样式，这种行为类似于内联样式，大大减低了样式和样式之间的冲突问题

   

#### 修饰符前缀

可以通过Tailwind CSS中的修饰符前缀来为元素设置在特定状态下存在的样式，也就是说可以使用修饰符前缀来为元素有条件的添加对应的样式



##### 伪类

 [伪类.md](vaults/001-伪类.md) 



##### 伪元素

###### before和after

使用`before`和`after`修饰符来为`::before`和`::after`伪元素设置样式

推荐只有在需要确保伪元素的内容不会实际呈现在DOM中且不可被用户选择的情况下，才使用`before`和`after`伪元素, 其余情况下使用真实的HTML元素来进行设置

```html
<!--
	默认情况下会为before和after设置content:''
	除非需要指定content的值，否则可以不用设置content功能类，默认值是空字符串
	ps: content的默认值是空字符串这个行为是Tailwind的预设基础样式(preflight)
			如果关闭了preflight, 那么就需要显示指定content-['']
-->
<span class="after:content-['*'] after:ml-0.5"> Email</span>
```



###### placeholder

使用placeholder修饰符为任何输入框或文本框的占位文本设置样式

```html
<input class="placeholder:text-slate-400" placeholder="Search for anything..." type="text"/>
```



###### selection

使用 selection 修改器来设置文本被选择时候的样式

selection 修改器是可继承的，因此可以将其添加到树中的任何位置，并将其应用于所有子元素

```html
<div class="selection:bg-fuchsia-300 selection:text-fuchsia-900">
  <!-- ... -->
</div>
```



##### 数据属性

使用 `data-*` 前缀来根据自定义的`data-*`属性来有条件地应用样式

由于`data-*`属性不是标准属性，所以只存在自定义值

```html
<!-- 生效 -->
<div data-size="large" class="data-[size=large]:p-8">
  <!-- ... -->
</div>

<!-- 不生效 -->
<div data-size="medium" class="data-[size=large]:p-8">
  <!-- ... -->
</div>
```

可以在 `tailwind.config.js` 文件的` theme.data` 部分中配置常用的数据属性选择器的快捷方式

```js
// @filename: tailwind.config.js
module.exports = {
  theme: {
    data: {
      // 有data-ui属性 且值存在checked
      checked: 'ui~="checked"',
    },
  },
}
```

```html
<div data-ui="checked active" class="data-checked:underline">
  <!-- ... -->
</div>
```



##### 自定义修饰符

```html
<ul role="list">
  {#each items as item}
    <!-- 
				使用[]来表示自定义选择器，下述自定义修饰符表示仅在元素是第三个子元素时选择该元素
				自定义修饰符可以和tailwind默认提供的修饰符一起结合使用
		-->
  	<li class="[&:nth-child(3)]:underline lg:[&:nth-child(3)]:hover:text-black">{item}</li>
  {/each}
</ul>
```

```html
<!-- 
	如果需要在选择器中使用空格 使用下划线代替
	以下表示的是 div p 添加 mt-4

	ps: 在转换的时候
      1. 下划线有效 空格无效 --- 使用下划线 --- bg-[url('/what_a_rush.png')]
      2. 下划线和空格同时有效 --- 使用空格 --- 可以使用转义字符保留下划线 --- before:content-['hello\_world']
         --- 对于JSX，是先React对JSX进行编译后，Tailwind在处理样式 --- 但是在JSX处理的时候就已经移除了下划线，需要使用String.raw来避免转换 
         --- String.raw`before:content-['hello\_world']` --- 或使用 before:content-['hello\\_world']
-->
<div class="[&_p]:mt-4">
	<!-- ... -->
</div>
```

```html
<!-- 
	自定义修饰符可以和@supports或@media结合使用
-->
<div class="flex [@supports(display:grid)]:grid">
  <!-- ... -->
</div>
```

```html
<!-- 
	[@media(any-hover:hover){&:hover}] 
	表示的含义如下：
	@media(any-hover:hover) {
		button:hover {
			// ...
		}	
	}
-->
<button type="button" class="[@media(any-hover:hover){&:hover}]:opacity-100">
  <!-- ... -->
</button>
```

```ts
// tailwind.config.js
let plugin = require('tailwindcss/plugin')

module.exports = {
  // 可以将经常使用的自定义修饰符前缀 添加到插件系统中
  plugins: [
    plugin(function ({ addVariant }) {
      // 添加 `third` 变体，例如 `third:pb-0`
      addVariant('third', '&:nth-child(3)')
    })
  ]
}
```



##### 解决歧义

在Tailwind中，很多功能类复用同一个命名空间，但是各自代表不同含义

```html
<!-- 其中text就是公用的命名空间 -->

<!-- 生成一个字体大小实用程序 -->
<div class="text-[22px]">...</div>

<!-- 生成一个颜色实用程序 -->
<div class="text-[#bada55]">...</div>
```

有时候，这真的是有歧义的

```html
<!-- 此时Tailwind就该变量具体代表的含义 -->
<div class="text-[var(--my-var)]">...</div>

<!-- 此时就需要在值之前添加 CSS 数据类型来“提示” Tailwind 底层的类型  -->

<!-- 生成一个字体大小实用程序 -->
<div class="text-[length:var(--my-var)]">...</div>

<!-- 生成一个颜色实用程序 -->
<div class="text-[color:var(--my-var)]">...</div>
```



#### 响应式断点 (Responsive breakpoints)

 [响应式断点.md](vaults/002-响应式断点.md) 



#### 黑夜模式

 [黑夜模式.md](vaults/003-黑夜模式.md) 



#### 样式复用

Tailwind优先推荐使用编辑器的`多光标功能`和`抽取组件`的方式来管理需要重复编写的功能类集合

但是在某些特殊情况下，为了复用功能类集合而单独抽取组件可能会使代码显得过于臃肿

可以使用 Tailwind 提供的 `@apply` 指令，将重复的实用程序模式提取到自定义 CSS 类中，以减少代码的冗余和重复

但是需要注意的是，无论任何情况下，都不要仅仅为了让代码看起来“更整洁”而滥用 `@apply`

当在项目中大多数地方使用 `@apply`，那么基本上等于是编写原生的 CSS样式，也就是说相当于放弃了 Tailwind 的可维护性优势

所以推荐仅在不使用react，vue之类的框架的情况下，需要抽取重复使用的样式的时候，使用`@apply`

```css
@tailwind base;
@tailwind components;
@tailwind utilities;

@layer components {
  /* <button class="btn-primary" /> */
  .btn-primary {
    @apply py-2 px-4 bg-blue-500;
  }
}
```



#### 自定义样式

 [自定义样式](vaults/004-自定义样式.md) 



#### Functions & Directives

 [Functions & Directives.md](vaults/005-Functions & Directives.md) 

