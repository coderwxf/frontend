## Svelte 组件

==**组件是Svelte应用程序的构建块。它们编写在.svelte文件中，使用HTML的超集。**==

==**所有三个部分（脚本、样式和标记）都是可选的**。==

```html
<script>
  // 在此处编写逻辑
</script>

<!-- 在此处编写标记（零个或多个项） -->

<style>
  /* 在此处编写样式 */
</style>
```



`<script>`

==**一个`<script>块`包含的JavaScript代码在创建组件实例时运行。在顶层声明（或导入）的变量可以从组件的标记中访问**==。此外，还有四个附加规则：

1. ==**export关键字创建一个组件属性或prop**==。
    Svelte使用export关键字来将变量声明标记为属性或prop，这意味着组件的使用者可以访问它（有关更多信息，请参见有关属性和props的部分）。

  ==**使用`export`关键字可以将变量声明为组件的属性（props），并使其成为公开的属性，可以被外部访问和使用**==

==**获取到组件实例后可以直接访问和获取所有的props属性**==

```html
<script>
  export let foo;

  // 传入作为props的值
  // 立即可用
  console.log({ foo });
</script>
```

==**您可以为prop指定一个默认初始值。如果组件的使用者在实例化组件时没有指定prop（或者其初始值为undefined），则将使用该默认值**==。==**请注意，如果prop的值随后被更新，则任何值未指定的prop将被设置为undefined（而不是其初始值）**。==

==**在开发模式下（请参见编译器选项），如果未提供默认初始值且使用者没有指定值，则会打印警告。为了消除此警告，请确保指定一个默认初始值，即使它的值为undefined。**==

```html
<script>
  export let bar = '可选的默认初始值';
  export let baz = undefined;
</script>
```

==**如果您导出一个const、class或function，则从组件外部只读**==。但是，函数作为prop值是有效的，如下所示。

```html
<!-- App.svelte -->
<script>
  // 这些是只读的
  export const thisIs = '只读';

  /** @param {string} name */
  export function greet(name) {
    alert(`hello ${name}!`);
  }

  // 这是一个prop
  export let format = (n) => n.toFixed(2);
</script>
```

==**只读的props可以作为绑定到组件的元素的属性，使用bind:this语法来访问**。==

==**您可以将保留字用作prop名称。**==

```html
<!-- App.svelte -->
<script>
  /** @type {string} */
  let className;

  // 创建一个名为`class`的属性，
  // 尽管它是保留字
  export { className as class };
</script>
```



2. ==**赋值是“响应式”的固定链接**==

==**要改变组件状态并触发重新渲染，只需对一个在本地声明的变量进行赋值。**==

更新表达式（count += 1）和属性赋值（obj.x = y）具有相同的效果。

```html
<script>
  let count = 0;

  function handleClick() {
    // 当标记引用`count`时，调用此函数将触发更新
    count = count + 1;
  }
</script>
```

==**因为 Svelte 的响应性基于赋值，所以使用 `.push()` 和 `.splice()` 等数组方法不会自动触发更新。需要进行后续的赋值才能触发更新**==。有关详细信息，也可以在教程中找到。

```html
<script>
  let arr = [0, 1];

  function handleClick() {
    // 此方法调用不会触发更新
    arr.push(2);
    // 如果标记引用`arr`，则此赋值将触发更新
    arr = arr;
  }
</script>
```

==**Svelte 的 `<script>` 块仅在组件创建时运行，因此 `<script>` 块内的赋值不会在 prop 更新时自动重新运行**==。如果想要跟踪 prop 的变化，请参考以下部分中的下一个示例。

```html
<script>
  export let person;
  // 这将仅在组件创建时设置`name`，当`person`发生变化时不会更新
  let { name } = person;
</script>
```



3. ==**$：使用反应式标记语法将语句标记为反应式**==

**==任何顶层语句（即不在块或函数内部的语句）都可以通过在其前面加上 $: JS 标签语法来使其成为反应式。反应式语句在其他脚本代码运行之后、组件标记呈现之前运行，只要它们所依赖的值发生了变化。==**

```html
<script>
  export let title;
  export let person;

  // 当 `title` 属性发生变化时，更新 `document.title`
  $: document.title = title;

  $: {
    console.log(`可以组合多个语句`);
    console.log(`当前标题是 ${title}`);
  }

  // 当 'person' 发生变化时，更新 `name`
  $: ({ name } = person);

  // 不要这样做，它会在上一行之前运行
  let name2 = name;
</script>
```

==**只有直接出现在 $: 块内部的值才会成为反应式语句的依赖项**==。例如，在下面的代码中，total 只会在 x 变化时更新，而不会在 y 变化时更新。

```html
<script>
  let x = 0;
  let y = 0;

  /** @param {number} value */
  function yPlusAValue(value) {
    return value + y;
  }

  $: total = yPlusAValue(x);
</script>

Total: {total}
<button on:click={() => x++}> Increment X </button>

<button on:click={() => y++}> Increment Y </button>
```

==**需要注意的是，反应式块的顺序是在编译时通过简单的静态分析确定的，编译器只会查看在块内部分配和使用的变量，而不会查看任何被调用函数内的变量**==。这意味着在以下示例中，当 x 发生变化时，yDependent 不会被更新：

```html
<script>
  let x = 0;
  let y = 0;

  /** @param {number} value */
  function setY(value) {
    y = value;
  }

  $: yDependent = y;
  $: setY(x);
</script>
```

==**将 `$: yDependent = y` 放在` $: setY(x) `的下面会导致 yDependent 在 x 更新时被更新。**==

==**如果一个语句完全由对未声明变量的赋值组成，Svelte 会自动为您注入一个 let 声明。**==

```html
<script>
  /** @type {number} */
  export let num;

  // 我们不需要声明 `squared` 和 `cubed`
  // - Svelte 会代劳
  $: squared = num * num;
  $: cubed = squared * num;
</script>
```



4. ==**使用 $ 前缀访问存储的值**==

存储（store）是一种对象，通过简单的存储合约（store contract）允许以反应式方式访问值。svelte/store 模块包含了满足该合约的最小存储实现。

任何时候当您有一个对存储的引用时，您可以在组件内部使用 $ 字符作为前缀来访问其值。这会导致 Svelte 声明带有前缀的变量，并在组件初始化时订阅存储，适当的时候取消订阅。

对带有 $ 前缀的变量的赋值要求该变量是一个可写存储，并将导致调用存储的 .set 方法。

请注意，存储必须在组件的顶层声明，而不是在 if 块或函数内部声明。

不代表存储值的局部变量不能以 $ 前缀开头。

```html
<script>
  import { writable } from 'svelte/store';

  const count = writable(0);
  console.log($count); // 输出 0

  count.set(1);
  console.log($count); // 输出 1

  $count = 2;
  console.log($count); // 输出 2
</script>
```

存储合约

存储必须包含一个 .subscribe 方法，该方法必须接受一个订阅函数作为参数。调用 .subscribe 时，该订阅函数必须立即且同步地使用存储的当前值调用。存储的所有活动订阅函数在存储的值发生变化时必须同步地再次调用。

.subscribe 方法必须返回一个取消订阅函数。调用取消订阅函数必须停止相应的订阅，并且存储的订阅函数不得再次被调用。

存储可以可选地包含一个 .set 方法，该方法必须接受存储的新值作为参数，并且同步地调用存储的所有活动订阅函数。这样的存储称为可写存储。

为了与 RxJS Observables 进行互操作，.subscribe 方法也可以返回一个带有 .unsubscribe 方法的对象，而不是直接返回取消订阅函数。但请注意，除非 .subscribe 同步地调用订阅函数（这不被 Observable 规范要求），否则 Svelte 将认为存储的值为 undefined，直到订阅函数被调用。