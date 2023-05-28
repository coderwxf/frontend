Vue 单文件组件 (Single-File Component，缩写为 SFC)。SFC 是一种可复用的代码组织形式，它将从属于同一个组件的 HTML、CSS 和 JavaScript 封装在使用 `.vue` 后缀的文件中

Vue 的核心功能是**声明式渲染**：通过扩展于标准 HTML 的模板语法，我们可以根据 JavaScript 的状态来描述 HTML 应该是什么样子的。当状态改变时，HTML 会自动更新

能在改变时触发更新的状态被称作是**响应式**的。我们可以使用 Vue 的 `reactive()` API 来声明响应式状态。由 `reactive()` 创建的对象都是 JavaScript [Proxy](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Proxy)，其行为与普通对象一样

`reactive()` 只适用于对象 (包括数组和内置类型，如 `Map` 和 `Set`)。而另一个 API `ref()` 则可以接受任何值类型。`ref` 会返回一个包裹对象，并在 `.value` 属性下暴露内部值。

在组件的 `<script setup>` 块顶层中声明的响应式状态，可以直接在模板中使用。下面展示了我们如何使用双花括号语法，根据 `counter` 对象和 `message` ref 的值渲染动态文本：

```
<h1>{{ message }}</h1>
<p>count is: {{ counter.count }}</p>
```

注意我们在模板中访问的 `message` ref 时不需要使用 `.value`：它会被自动解包，让使用更简单。

在双花括号中的内容并不只限于标识符或路径——我们可以使用任何有效的 JavaScript 表达式。

```
<h1>{{ message.split('').reverse().join('') }}</h1>
```

---

在 Vue 中，mustache 语法 (即双大括号) 只能用于文本插值。为了给 attribute 绑定一个动态值，需要使用 `v-bind` 指令：

```
<!-- 
	v-bind -- 指令
	id -- 指令参数
	sync -- 修饰符
	dynamicId -- 指令的值 
-->
<div v-bind:id.sync="dynamicId"></div>
```

**指令**是由 `v-` 开头的一种特殊 attribute。它们是 Vue 模板语法的一部分。和文本插值类似，指令的值是可以访问组件状态的 JavaScript 表达式

由于 `v-bind` 使用地非常频繁，它有一个专门的简写语法：

```
<div :id="dynamicId"></div>
```

---

我们可以使用 `v-on` 指令监听 DOM 事件：

```
<button v-on:click="increment">{{ count }}</button>
```

因为其经常使用，`v-on` 也有一个简写语法：

```
<button @click="increment">{{ count }}</button>
```

事件处理函数也可以使用内置表达式，并且可以使用修饰符简化常见任务

```vue
<script setup>
import { ref } from 'vue'

const count = ref(0)
</script>

<template>
  <!--在函数的内置表达式中使用ref变量也是会自动解包的 -->
  <button @click="count++">count is: {{ count }}</button>
</template>
```

---

我们可以同时使用 `v-bind` 和 `v-on` 来在表单的输入元素上创建双向绑定：

```
<input :value="text" @input="onInput">
```

```
function onInput(e) {
  // v-on 处理函数会接收原生 DOM 事件
  // 作为其参数。
  text.value = e.target.value
}
```

为了简化双向绑定，Vue 提供了一个 `v-model` 指令，它实际上是上述操作的语法糖：

```
<input v-model="text">
```

`v-model` 会将被绑定的值与 `<input>` 的值自动同步

`v-model` 不仅支持文本输入框，也支持诸如多选框、单选框、下拉框之类的输入类型

---

我们可以使用 `v-if` 指令来有条件地渲染元素：

```
<h1 v-if="awesome">Vue is awesome!</h1>
```

这个 `<h1>` 标签只会在 `awesome` 的值为[真值 (Truthy)](https://developer.mozilla.org/zh-CN/docs/Glossary/Truthy) 时渲染。若 `awesome` 更改为[假值 (Falsy)](https://developer.mozilla.org/zh-CN/docs/Glossary/Falsy)，它将被从 DOM 中移除。

我们也可以使用 `v-else` 和 `v-else-if` 来表示其他的条件分支：

```
<h1 v-if="awesome">Vue is awesome!</h1>
<h1 v-else>Oh no 😢</h1>
```

---

我们可以使用 `v-for` 指令来渲染一个基于源数组的列表：

```
<ul>
  <li v-for="todo in todos" :key="todo.id">
    {{ todo.text }}
  </li>
</ul>
```

这里的 `todo` 是一个局部变量，表示当前正在迭代的数组元素。它只能在 `v-for` 所绑定的元素上或是其内部访问，就像函数的作用域一样。

注意，我们还给每个 todo 对象设置了唯一的 `id`，并且将它作为[特殊的 `key` attribute](https://cn.vuejs.org/api/built-in-special-attributes.html#key) 绑定到每个 `<li>`。`key` 使得 Vue 能够精确的移动每个 `<li>`，以匹配对应的对象在数组中的位置。

在 Vue 中，`key` 是一个特殊的属性，用于帮助 Vue 识别不同的 DOM 元素。当 Vue 用 v-for 指令循环渲染相同类型的元素时，它会尝试尽可能高效地更新 DOM。在这种情况下，每个元素都应该有一个唯一的 `key` 属性，以便 Vue 可以识别它们之间的差异

在 Vue 中，如果没有为 `v-for` 指令中的元素设置 `key` 属性，Vue 会默认使用元素的类型和在数组中的索引位置来进行比较

更新列表有两种方式：

1. 在源数组上调用[变更方法](https://stackoverflow.com/questions/9009879/which-javascript-array-functions-are-mutating)：

   ```
   todos.value.push(newTodo)
   ```

2. 使用新的数组替代原数组：

   ```
   todos.value = todos.value.filter(/* ... */)
   ```

----

[`computed()`](https://cn.vuejs.org/guide/essentials/computed.html)。它可以让我们创建一个计算属性 ref，这个 ref 会动态地根据其他响应式数据源来计算其 `.value`

计算属性会自动跟踪其计算中所使用的到的其他响应式状态，并将它们收集为自己的依赖。计算结果会被缓存，并只有在其依赖发生改变时才会被自动更新。

----

有时我们也会不可避免地需要手动操作 DOM。

这时我们需要使用**模板引用**——也就是指向模板中一个 DOM 元素的 ref。我们需要通过[这个特殊的 `ref` attribute](https://cn.vuejs.org/api/built-in-special-attributes.html#ref) 来实现模板引用：

```
<p ref="p">hello</p>
```

要访问该引用，我们需要声明一个同名的 ref：

```
const p = ref(null)
```

注意这个 ref 使用 `null` 值来初始化。这是因为当 `<script setup>` 执行时，DOM 元素还不存在。模板引用 ref 只能在组件**挂载**后访问。

要在挂载之后执行代码，我们可以使用 `onMounted()` 函数：

```
import { onMounted } from 'vue'

onMounted(() => {
  // 此时组件已经挂载。
})
```

这被称为**生命周期钩子**——它允许我们注册一个在组件的特定生命周期调用的回调函数。还有一些其他的钩子如 `onUpdated` 和 `onUnmounted`。更多细节请查阅[生命周期图示](https://cn.vuejs.org/guide/essentials/lifecycle.html#lifecycle-diagram)。

---

有时我们需要响应性地执行一些“副作用”——例如，当一个数字改变时将其输出到控制台。我们可以通过侦听器来实现它：

```
import { ref, watch } from 'vue'

const count = ref(0)

watch(count, (newCount) => {
  // 没错，console.log() 是一个副作用
  console.log(`new count is: ${newCount}`)
})
```

`watch()` 可以直接侦听一个 ref，并且只要 `count` 的值改变就会触发回调。`watch()` 也可以侦听其他类型的数据源

---

在 Vue 中，副作用（Side Effect）通常指在组件生命周期钩子函数、指令、计算属性、观察者函数等中执行的操作，这些操作可能会对应用程序状态或 DOM 进行更改，从而产生副作用

如果在修改状态的同时引发其他操作，那么这些操作就可以被视为副作用。

---

真正的 Vue 应用往往是由嵌套组件创建的。

父组件可以在模板中渲染另一个组件作为子组件。要使用子组件，我们需要先导入它：

```
import ChildComp from './ChildComp.vue'
```

然后我们就可以在模板中使用组件，就像这样：

```
<ChildComp />
```

---

子组件可以通过 **props** 从父组件接受动态数据。首先，需要声明它所接受的 props：

```
<!-- ChildComp.vue -->
<script setup>
const props = defineProps({
  msg: String
})
</script>
```

注意 `defineProps()` 是一个编译时宏，并不需要导入。一旦声明，`msg` prop 就可以在子组件的模板中使用。它也可以通过 `defineProps()` 所返回的对象在 JavaScript 中访问。



父组件可以像声明 HTML attributes 一样传递 props。若要传递动态值，也可以使用 `v-bind` 语法：

```
<ChildComp :msg="greeting" />
```

---

除了接收 props，子组件还可以向父组件触发事件：

```
<script setup>
// 声明触发的事件
const emit = defineEmits(['response'])

// 带参数触发
emit('response', 'hello from child')
</script>
```



`emit()` 的第一个参数是事件的名称。其他所有参数都将传递给事件监听器。

父组件可以使用 `v-on` 监听子组件触发的事件——这里的处理函数接收了子组件触发事件时的额外参数并将它赋值给了本地状态：

```
<ChildComp @response="(msg) => childMsg = msg" />
```

---

除了通过 props 传递数据外，父组件还可以通过**插槽** (slots) 将模板片段传递给子组件：

```
<ChildComp>
  This is some slot content!
</ChildComp>
```



在子组件中，可以使用 `<slot>` 元素作为插槽出口 (slot outlet) 渲染父组件中的插槽内容 (slot content)：

```
<!-- 在子组件的模板中 -->
<slot/>
```



`<slot>` 插口中的内容将被当作“默认”内容：它会在父组件没有传递任何插槽内容时显示：

```
<slot>Fallback content</slot>
```



插槽出口（slot outlet）是指在子组件中定义的插槽（slot）所对应的出口，用于向子组件中插入父组件传入的内容。

当在父组件中使用子组件时，我们可以将要插入子组件中的内容放在父组件的模板中，并使用插槽语法将其传递给子组件。这些内容会被插入到子组件中对应的插槽出口中，从而替换子组件中定义的插槽的默认内容

---

丰富的、可渐进式集成的生态系统，可以根据应用规模在库和框架间切换自如。
