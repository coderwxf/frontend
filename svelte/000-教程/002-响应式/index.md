Svelte的核心是一个强大的响应系统，用于将DOM与应用程序状态保持同步，例如响应事件。

Svelte可以自动更新组件状态变化时的DOM

请注意，响应式声明和语句将在其他脚本代码之后运行，并在组件标记呈现之前运行。

```html
<script>
	let count = 0;

	function increment() {
		count += 1;
	}
</script>

<button>
	Clicked {count}
	{count === 1 ? 'time' : 'times'}
</button>
```



```javascript
let count = 0;

// $: 开头的叫做 响应式声明或者响应式语句块
// 通过响应式声明 或 响应式语句块 定义 计算属性 --- 具有缓冲功能 --- 每当引用的值发生更改时重新运行此代码
$: doubled = count * 2;
```

在Svelte中，如果在`$`块中将值分配给一个以前未声明的变量，则Svelte会自动在该变量上使用`let`关键字进行声明，然后将其赋值给指定的值

---

在Svelte中，我们不仅可以声明响应式值，还可以以响应式的方式运行任意语句。例如，我们可以在`count`变化时记录`count`的值：

```javascript
let count = 0;

$: console.log(`the count is ${count}`);
```

我们还可以使用代码块来组合多个语句：

```javascript
$: {
	console.log(`the count is ${count}`);
	console.log(`this will also be logged whenever count changes`);
}
```

甚至可以在`if`块前加上`$:`，使其以响应式的方式运行：

```javascript
$: if (count >= 10) {
	alert('count is dangerously high!');
	count = 0;
}
```

这些特性让我们能够以响应式的方式运行任意代码，从而能够更加灵活地控制应用程序的行为。

---

**==在Svelte中，响应式是通过赋值操作来触发的，所以使用`push`和`splice`等数组方法不会自动引起更新==**。

例如，单击“添加一个数字”按钮不会做任何事情，即使我们在`addNumber`中调用了`numbers.push(...)`。

**==一种解决方法是添加一个本来多余的赋值操作==**：

```javascript
function addNumber() {
	numbers.push(numbers.length + 1);
	numbers = numbers;
}
```

但是，==**有一种更常见的解决方法：**==

```javascript
function addNumber() {
	numbers = [...numbers, numbers.length + 1];
}
```

你可以使用类似的模式来替换`pop`、`shift`、`unshift`和`splice`等方法。

**==对于数组和对象的*属性*赋值，例如`obj.foo += 1`或`array[i] = x`，与对值本身的赋值操作相同。==**

```javascript
function addNumber() {
	numbers[numbers.length] = numbers.length + 1;
}
```

**==一个简单的经验法则是：更新的变量名必须出现在赋值操作的左侧==**。

例如，这个例子...

```javascript
const foo = obj.foo;
foo.bar = 'baz';
```

...不会触发`obj.foo.bar`的响应式更新，除非你接下来添加`obj = obj`。

---

在任何实际应用程序中，你都需要将数据从一个组件传递到其子组件中。为此，我们需要声明*属性*，通常缩写为“props”。在Svelte中，我们可以使用`export`关键字来声明组件的属性。

```svelte
<script>
  // 指定props 并设置默认值
	export let answer = 'this is no answer';
</script>
```

---



在这个练习中，我们忘记了指定`PackageInfo.svelte`期望的`version`属性，导致它显示为“version undefined”。

我们*可以*通过添加`version`属性来修复它：

```svelte
<script>
	import PackageInfo from './PackageInfo.svelte';

	const pkg = {
		name: 'svelte',
		speed: 'blazing',
		version: 4,
		website: 'https://svelte.dev'
	};
</script>

<PackageInfo {...pkg} />
```

会被编译为

```svelte
<PackageInfo
	name={pkg.name}
	speed={pkg.speed}
	version={pkg.version}
	website={pkg.website}
/>
```

`$$props` 对象是 Svelte 组件中的一个特殊对象，它包含了组件接收的所有穿透属性（context）和 props 属性

使用 `$$props` 可以在组件内部访问和操作这些属性，但是 Svelte 在编译时很少对 `$$props` 进行优化操作

因此，一般情况下，不推荐频繁地使用 `$$props`。