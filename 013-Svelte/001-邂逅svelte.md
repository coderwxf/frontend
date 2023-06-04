Svelte 类似 React 和 Vue，都是用来构建前端用界面的框架



## 特点

1. svelte将很多vue和react在运行时完成的工作提前到了构建时

   这就意味着运行项目的时候，不需要考虑框架运行所带来的额外性能成本，首次加载也无额外性能损耗

   也就是说Svelte 为了最优的性能及最简的代码输出，使用了在编译阶段直接生成最终代码的方式，而不是在运行阶段引入框架进行相应的操作，所以在这个方面，svelte更像一个编译器。

   

2. Svelte是渐进式框架

   Svelte 编写整个应用，也可以用来逐步重构现有代码，甚至可以只输出单个独立组件（无需强制附带框架自身）并将其用于任意框架中
   
   Svelte中的每一个组件都会被编译为创建DOM的JS文件和渲染组件的css文件， 我们可以在React,Vue,乃至任何地方引入这个类，使用new创建实例后就可以使用



## 组件

1. 一个 Svelte 应用由一个或多个*组件*（Components）组合而成
2. 每个组件对应一个 `.svelte` 文件, 也就是每一个`.svlete`文件就是一个SFC
3. 每个组件包含了组件自身的 HTML、CSS 和 JavaScript 代码
4. 每个组件有自己的组件作用域

```html
<!-- 这是最简单的组件 -->
<h1>Hello world!</h1>
```



## 动态绑定

`动态绑定数据`

```html
<script>
  // 这个变量是组件内部的局部变量
  let name = 'world'; 
</script> 

<!-- svelte使用大括号语法在绑定变量 -->
<h1>Hello {name}!</h1>

<!-- {}中支持所有的合法的js表达式 -->
<h1>Hello {name.toUpperCase()}!</h1>
```



`动态绑定属性`

```html
<script>
  let src = 'tutorial/image.gif';
  let action = 'dance'
</script>

<!-- {}语法同样可以用来绑定属性值 -->
<!--
	Svelte要求我们尽可能的完善我们的标签，如img标签必须加上alt(不加会提示warning)
	以便于服务屏幕阅读器用户 和 硬件或网络不佳的用户
	也就是所谓的可访问性（Accessibility，缩写 a11y）
-->
<img src={src} alt="A man dances">

<!-- 如果属性名和属性值同名的时候，svelte可以对这种属性进行简写 -->
<img {src} alt="A man dances">

<!-- {}中绑定属性可以直接使用在字符串中，会自动进行替换 -->
<img src={src} alt="A man {action}">
```



## 样式

```html
<style>
  /*
  	svelte 会为每一个组件添加一个svelte-xxx (xxx是唯一的hash值)
    h2设置的属性，实际会被渲染为h2.svelte-xxxx
  	所以组件中设置的样式都是组件内部的样式，不会溢出影响别的组件
  */
  h2 {
    color: blue;
  }
</style>

<h2>Hello World</h2>

<!-- 
	编译后的伪代码如下:
	html:
		<h2 class="svelte-136z7jr">Hello World</h2>

	css:
		/*
			136z7jr就是svelete为每一个组件形成的唯一hash值，可以认为就是组件的id值
		*/
		h2.svelte-136z7jr {
			color: blue;
		}
-->
```



## 子组件

```html
<!-- Child.svelte-->

<!-- 
	子组件中的p会变为红色，但是父组件中的p不会红色 
	=> 设置的样式的作用域默认是 组件作用域 
-->
<p>Child Component</p>

<style>
  p {
    color: red
  }
</style>
```



```html
<script>
  // svelte引入的组件不需要注册，就可以直接使用
  import Child from './Child.svlete'
</script>

<p>Father Component</p>

<!--
	svelte中的组件名由以下特点:
		1. 组件名的首字母要大写 --- 区分于一般的HTML标签
		2. 组件如果使用单标签的话 --- 必须以/闭合
-->
<Child />
```



## 渲染html

```html
<!-- 
	字符串以纯文本形式插入，这意味着像 < 和 > 这样的html字符没有特殊含义
	如果需要将这些标签需要以对应的html显示方式进行渲染的时候，可以使用@html
	这类似于vue的v-html，也就意味着，svelte在以html标签方式渲染这些内容的时候
	不会对数据进行任何处理，而是直接解析，所以必须手动转义来自不信任来源的HTML
	以减少遭受XSS攻击的风险
-->
<script>
	let string = `this string contains some <strong>HTML!!!</strong>`;
</script>

<!-- this string contains some <strong>HTML!!!</strong> -->
<p>{string}</p>

<!-- this string contains some HTML!!! -->
<p>{@html string}</p>
```



## 应用入口

```js
// index.js
import App from './App.svelte';

// Svelte 的组件经过编译后直接输出为原生的 JavaScript 对象，
// 可以轻松将这个对象在 React、Vue 或其他地方导入，然后通过 new 实例化，就可以使用
const app = new App({
  target: document.body, // 挂载点 - DOM节点
  // 父组件传入给子组件中的属性
  props: { 
    answer: 42
  }
});
```



## debugger

```html
<script>
	let user = {
		firstname: 'Ada',
		lastname: 'Lovelace'
	};
</script>

<input bind:value={user.firstname}>
<input bind:value={user.lastname}>

<!-- 
	调试方式1: 使用console.log
	{} 中 可以存放任何合法的JS表达式
	{} 中的代码 会在组件被编译的时候被解析
	所以可以使用这种方式进行调试
	但是因为console.log返回值是undefined
  所以会在界面上输出一个undefined
-->
{ console.log(user) }

<h1>Hello {user.firstname}!</h1>
```

```html
<script>
	let user = {
		firstname: 'Ada',
		lastname: 'Lovelace'
	};

	let demo = 'demo'
</script>

<input bind:value={user.firstname}>
<input bind:value={user.lastname}>

<!--
	调试方式2： 使用debug的方式来进行调试
	{@debug xxx}
	1. xxx 可以不跟任何内容 此时就等价于 debugger
  2. xxx 如果需要加上内容，必须是script中顶层作用域中定义的变量，因为这些顶层变量会被转化为组件对应的类的属性
	   此时就类似于 console.log(xxx) debugger
-->

<!-- success -->
{@debug user}

<!-- error -->
<!-- {@debug user.firstname} -->

<!-- success -->
{@debug demo}

<!-- success -->
{@debug}

<h1>Hello {user.firstname}!</h1>
```

