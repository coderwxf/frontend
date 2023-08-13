==**HTML本身没有表达*逻辑*的方法，比如条件和循环。Svelte则提供了这样的功能。**==

==**要根据条件渲染一些标记，我们可以将其包装在一个`if`块中。**==

让我们添加一些文本，当`count`大于`10`时显示：

```svelte
<button on:click={increment}>
	Clicked {count}
	{count === 1 ? 'time' : 'times'}
</button>

{#if count > 10}
	<p>{count} is greater than 10</p>
{/if}
```

---

就像在JavaScript中一样，if块可以有一个else块：

```svelte
{#if count > 10}
	<p>{count} is greater than 10</p>
{:else if count < 5}
	<p>{count} is less than 5</p>
{:else}
	<p>{count} is between 5 and 10</p>
{/if}
```

**==一个`#`字符始终表示块的打开标签。一个`/`字符始终表示块的闭合标签。一个`:`字符，如`{:else}`，表示块的继续标签==**。

---

当构建用户界面时，通常需要处理数据列表。此时可以使用`each`块：

```svelte
<div>
  <!-- colors 可以是任何类数组对象（即具有`length`属性) 和 任何的可迭代对象 -->
  <!-- svelte 和 vue不同，不能直接迭代原生对象 -->
	{#each colors as color, i}
		<button
			aria-current={selected === color}
			aria-label="{color}"
			style="background: {color}"
			on:click={() => selected = color}
		>
      {i + 1}
    </button>
	{/each}
</div>
```

---

**==默认情况下，当你修改 `each` 块的值时，它会在块的*末尾*添加和删除项目，并更新任何已更改的值==**。这可能并不是你想要的结果。

[通过示例更容易理解](https://learn.svelte.dev/tutorial/keyed-each-blocks)。多次点击"Remove first thing"按钮，并观察发生的情况：它不会删除第一个 `<Thing>` 组件，而是删除*最后一个* DOM 节点。然后它会更新剩余 DOM 节点中的 `name` 值，但不会更新 emoji，因为它在每个 `<Thing>` 创建时就已固定。

相反，我们希望仅删除第一个 `<Thing>` 组件及其 DOM 节点，而保持其他组件不受影响。

为了实现这一点**==，我们为 `each` 块指定一个唯一的标识符（或"key"）：==**

```svelte
{#each things as thing (thing.id)}
	<Thing name={thing.name}/>
{/each}
```

**==这里的 `(thing.id)` 是*键*，它告诉 Svelte 如何确定在组件更新时要更改哪个 DOM 节点。==**

**==你可以使用任何对象作为键, 例如可以使用 `(thing)` 而不是 `(thing.id)`。然而，使用字符串或数字通常更安全==**

==**svelte底层会将key作为map的key，所以这个key 可以使用对象**== 
==**但是存在对象属性值不一样 但是引用地址是一样的情况 ，所以在这种情况下，并不推荐使用对象作为遍历的key**==
==**而是推荐使用基础数据类型作为遍历的key，如number类型或字符串类型**==

---

以下是使用Svelte在标记中直接*await* Promise值的示例代码：

```svelte
{#await promise}
	<p>...等待中</p>
{:then number}
	<p>数字为 {number}</p>
{:catch error}
	<p style="color: red">{error.message}</p>
{/await}
```



**==如果您知道Promise不会被拒绝，可以省略`catch`块。如果您不希望在Promise解析之前显示任何内容，您也可以省略第一个块：==**

```svelte
{#await promise then number}
	<p>数字为 {number}</p>
{/await}
```



**==只有最近的`promise`被考虑，这意味着您不必担心竞态条件==**

**=="await" 指令在 Svelte 中有一个特殊的行为，即当等待的异步请求还没有完成时，如果发送了一个新的异步请求，那么原始的异步请求将被丢弃，并且将切换到新的异步请求。==**

这种行为被称为 "await-friendly"，它的目的是使得代码更加简洁，易于阅读和维护。它允许你编写更简洁的代码，不必担心处理多个异步请求的复杂情况。
