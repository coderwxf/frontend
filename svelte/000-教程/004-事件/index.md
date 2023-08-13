可以使用`on:`指令在元素上监听任何DOM事件

```html
<script>
	let m = { x: 0, y: 0 };

	function handleMove(event) {
		m.x = event.clientX;
		m.y = event.clientY;
	}
</script>

<div on:pointermove={handleMove}>
	The pointer is at {m.x} x {m.y}
</div>

<style>
	div {
		position: fixed;
		left: 0;
		top: 0;
		width: 100%;
		height: 100%;
		padding: 1rem;
	}
</style>
```

```html
<script>
	let m = { x: 0, y: 0 };
</script>

<div
	on:pointermove={(e) => {
		m = { x: e.clientX, y: e.clientY };
	}}
>
	The mouse position is {m.x} x {m.y}
</div>
```

在某些框架中，出于性能原因，可能会建议避免在循环内部使用内联事件处理程序。然而，对于Svelte框架来说，这个建议并不适用——无论你选择哪种形式，编译器都会自动处理，保证正确的操作。

在编译时，Svelte会将相应的事件定义提取出来，确保只定义一个事件处理函数，即使在循环中也是如此。这样可以避免性能问题，并确保正确的事件处理

---

**==DOM事件处理程序可以具有*修饰符*，以改变其行为==**。例如，带有`once`修饰符的处理程序只会运行一次：

```svelte
<button on:click|once={() => alert('clicked')}>
	Click me
</button>
```

修饰符的完整列表：

- **==`preventDefault` — 在运行处理程序之前调用`event.preventDefault()`==**。在客户端表单处理中很有用，例如。

- ==**`stopPropagation` — 调用`event.stopPropagation()`，阻止事件传播到下一个元素。**==

- ==**`passive` — 提高触摸/滚轮事件的滚动性能（Svelte会在安全的情况下自动添加它）。**==

- ==**`nonpassive` — 显式设置`passive: false`。**==

- ==**`capture` — 在[*捕获*](https://developer.mozilla.org/en-US/docs/Learn/JavaScript/Building_blocks/Events#event_capture)阶段而不是[*冒泡*](https://developer.mozilla.org/en-US/docs/Learn/JavaScript/Building_blocks/Events#event_bubbling)阶段触发处理程序。**==

- ==**`once` — 在第一次运行后移除处理程序。**==

- ==**`self` — 仅在`event.target`是元素本身时触发处理程序。**==

- ==**`trusted` — 仅在`event.isTrusted`为`true`时触发处理程序，这意味着事件是由用户操作触发的，而不是因为某些JavaScript调用了`element.dispatchEvent(...)`。**==

  ==**event.isTrusted`属性是一个只读属性，不可修改**==

==**您可以链式使用修饰符，例如`on:click|once|capture={...}`**。==

---

**==组件也可以派发事件。为了这样做，它们必须创建一个事件派发器==**。更新`Inner.svelte`文件：

```svelte
<script>
	import { createEventDispatcher } from 'svelte';

	const dispatch = createEventDispatcher();

	function sayHello() {
		dispatch('message', {
			text: 'Hello!'
		});
	}
</script>
```

> **==`createEventDispatcher`必须在组件第一次实例化时调用，不能在后面的`setTimeout`回调函数中调用。这将把`dispatch`与组件实例关联起来。==**

然后，在`App.svelte`文件中添加一个`on:message`事件处理程序：

```svelte
<Inner on:message={handleMessage} />
```

> 您也可以尝试将事件名称更改为其他内容。例如，在`Inner.svelte`中将`dispatch('message', {...})`更改为`dispatch('greet', {...})`，并在`App.svelte`中将属性名称从`on:message`更改为`on:greet`。

---

**==与 DOM 事件不同，组件事件不会“冒泡”。如果你想监听某个深度嵌套组件上的事件，中间组件必须将事件“转发”。==**

在这个例子中，我们有与[前一章节](https://learn.svelte.dev/tutorial/component-events)相同的 `App.svelte` 和 `Inner.svelte`，但现在有一个包含 `<Inner/>` 的 `Outer.svelte` 组件。

一种解决方法是在 `Outer.svelte` 中添加 `createEventDispatcher`，监听 `message` 事件并为其创建处理程序：

```svelte
<script>
	import Inner from './Inner.svelte';
	import { createEventDispatcher } from 'svelte';

	const dispatch = createEventDispatcher();

	function forward(event) {
		dispatch('message', event.detail);
	}
</script>

<Inner on:message={forward}/>
```

但是这是很多冗余代码，**==因此 Svelte 提供了一个等效的简写方式——一个没有值的 `on:message` 事件指令表示“转发所有的 `message` 事件”。==**

```svelte
<script>
	import Inner from './Inner.svelte';
</script>

<Inner on:message/>
```

---

事件转发对于 DOM 事件也适用。

我们想要在 `<BigRedButton>` 上得到点击事件的通知，为了实现这个目的，我们只需要在 `BigRedButton.svelte` 文件中将 `<button>` 元素的 `click` 事件进行转发：

```svelte
<button on:click>
	Push
</button>
```

**==在 Svelte 中，如果一个组件上的事件没有处理函数，它会自动向上抛出一个同名的事件。这意味着如果你在一个组件上监听了一个事件，但没有为该事件指定处理函数，该事件将会被自动转发到组件的父组件上。这种机制使得事件可以在组件层级中进行传递，直到某个组件处理该事件或者事件到达根组件==**