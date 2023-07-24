在前端处理表单时，我们常常需要将表单输入框的内容同步给 JavaScript 中相应的变量

```html
<input
  :value="text"
  @input="event => text = event.target.value">
```

手动连接值绑定和更改事件监听器可能会很麻烦, `v-model` 指令帮我们简化了这一步骤：

```html
<input v-model="text">
```



另外，`v-model` 还可以用于各种不同类型的输入，`<textarea>`、`<select>` 元素

它会根据所使用的元素自动使用对应的 DOM 属性和事件组合：

- 文本类型的 `<input>` 和 `<textarea>` 元素会绑定 `value` property 并侦听 `input` 事件；
- `<input type="checkbox">` 和 `<input type="radio">` 会绑定 `checked` property 并侦听 `change` 事件；
- `<select>` 会绑定 `value` property 并侦听 `change` 事件



`v-model` 会忽略任何表单元素上初始的 `value`、`checked` 或 `selected

它将始终将当前绑定的 JavaScript 状态视为数据的正确来源



## 基本用法

### 文本

```html
<input v-model="message" placeholder="edit me" />
<textarea v-model="message" placeholder="add multiple lines"></textarea>
```



对于需要使用 [IME](https://en.wikipedia.org/wiki/Input_method) 的语言 (中文，日文和韩文等)，你会发现 `v-model` 不会在 IME 输入还在拼字阶段时触发更新。如果你的确想在拼字阶段也触发更新，请直接使用自己的 `input` 事件监听器和 `value` 绑定而不要使用 `v-model`。



注意在 `<textarea>` 中是不支持插值表达式的。请使用 `v-model` 来替代：

```html
<!-- 错误 -->
<textarea>{{ text }}</textarea>

<!-- 正确 -->
<textarea v-model="text"></textarea>
```



### 复选框

单一的复选框，绑定布尔类型值：

```html
<input type="checkbox" id="checkbox" v-model="checked" />
```

我们也可以将多个复选框绑定到同一个数组或[集合](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Set)的值：

```html
<template>
  <div>Checked names: {{ checkedNames }}</div>

  <input type="checkbox" id="jack" value="Jack" v-model="checkedNames">
  <label for="jack">Jack</label>

  <input type="checkbox" id="john" value="John" v-model="checkedNames">
  <label for="john">John</label>

  <input type="checkbox" id="mike" value="Mike" v-model="checkedNames">
  <label for="mike">Mike</label>
</template>

<script setup>
	const checkedNames = ref([])
</script>
```



### 单选按钮

```html
<div>Picked: {{ picked }}</div>

<input type="radio" id="one" value="One" v-model="picked" />
<label for="one">One</label>

<input type="radio" id="two" value="Two" v-model="picked" />
<label for="two">Two</label>
```



### 选择器

单个选择器的示例如下：

```html
<div>Selected: {{ selected }}</div>

<select v-model="selected">
  <option disabled value="">Please select one</option>
  <option>A</option>
  <option>B</option>
  <option>C</option>
</select>
```

如果 `v-model` 表达式的初始值不匹配任何一个选择项，`<select>` 元素会渲染成一个“未选择”的状态。在 iOS 上，这将导致用户无法选择第一项，因为 iOS 在这种情况下不会触发一个 change 事件。因此，我们建议提供一个空值的禁用选项



多选 (值绑定到一个数组)：

```html
<div>Selected: {{ selected }}</div>

<select v-model="selected" multiple>
  <option>A</option>
  <option>B</option>
  <option>C</option>
</select>
```



## 值绑定

对于单选按钮，复选框和选择器选项，`v-model` 绑定的值通常是静态的字符串 (或者对复选框是布尔值)

```html
<!-- `picked` 在被选择时是字符串 "a" -->
<input type="radio" v-model="picked" value="a" />

<!-- `toggle` 只会为 true 或 false -->
<input type="checkbox" v-model="toggle" />

<!-- `selected` 在第一项被选中时为字符串 "abc" -->
<select v-model="selected">
  <option value="abc">ABC</option>
</select>
```



但我们可以通过 `v-bind` 来绑定动态值，甚至是非字符串值

```html
<!--
	默认情况下 checkbox的值 将会是 true/false
	但是可以通过true-value 和 false-value 来修改默认值

	true-value 和 false-value 是 Vue 特有属性，仅支持和 v-model 配套使用
-->
<input
  type="checkbox"
  v-model="toggle"
  true-value="yes"
  false-value="no" />
```

true-value 和 false-value 属性通常用于 `<input type="checkbox">` 和` <input type="radio">` 元素

且绑定的值是布尔类型值的时候，用于修改默认值

例如多选框虽然是checkbox 但是绑定的值是数组，所以不可以使用true-value 和 false-value

但是单选框或者单个checkbox 其绑定的值是布尔值，所以可以使用true-value 和 false-value



`true-value` 和 `false-value` attributes 不会影响 `value` attribute，因为浏览器在表单提交时，并不会包含未选择的复选框。为了保证这两个值 (例如：“yes”和“no”) 的其中之一被表单提交，请使用单选按钮作为替代。



```html
<select v-model="selected">
  <!-- 内联对象字面量 -->
  <option :value="{ number: 123 }">123</option>
</select>
```



## 修饰符

### `.lazy`

默认情况下，`v-model` 会在每次 `input` 事件后更新数据 ([IME 拼字阶段的状态](https://cn.vuejs.org/guide/essentials/forms.html#vmodel-ime-tip)例外)。你可以添加 `lazy` 修饰符来改为在每次 `change` 事件后更新数据：

```html
<!-- 在 "change" 事件后同步更新而不是 "input" -->
<input v-model.lazy="msg" />
```



### `.number`

如果你想让用户输入自动转换为数字，你可以在 `v-model` 后添加 `.number` 修饰符来管理输入：

```html
<input v-model.number="age" />
```

如果该值无法被 `parseFloat()` 处理，那么将返回原始值。

`number` 修饰符会在输入框有 `type="number"` 时自动启用。



### `.trim`

如果你想要默认自动去除用户输入内容中两端的空格，你可以在 `v-model` 后添加 `.trim` 修饰符：

```html
<input v-model.trim="msg" />
```



## 组件上的 `v-model`

HTML 的内置表单输入类型并不总能满足所有需求。幸运的是，我们可以使用 Vue 构建具有自定义行为的可复用输入组件，并且这些输入组件也支持 `v-model`

`v-model` 可以在组件上使用以实现双向绑定。



 `v-model` 在原生元素上的用法：

```html
<input v-model="searchText" />
```

```html
<input
  :value="searchText"
  @input="searchText = $event.target.value"
/>
```



当使用在一个组件上时，`v-model` 会被展开为如下的形式

```html
<CustomInput v-model="searchText" />
```

```html
<CustomInput
  :modelValue="searchText"
  @update:modelValue="newValue => searchText = newValue"
/>
```

`<CustomInput>` 组件内部需要做两件事：

1. 将内部原生 `<input>` 元素的 `value` attribute 绑定到 `modelValue` prop
2. 当原生的 `input` 事件触发时，触发一个携带了新值的 `update:modelValue` 自定义事件

```html
<!-- CustomInput.vue -->
<script setup>
  defineProps(['modelValue'])
  defineEmits(['update:modelValue'])
</script>

<template>
  <input
    :value="modelValue"
    @input="$emit('update:modelValue', $event.target.value)"
  />
</template>
```

另一种在组件内实现 `v-model` 的方式是使用一个可写的，同时具有 getter 和 setter 的 `computed` 属性。`get` 方法需返回 `modelValue` prop，而 `set` 方法需触发相应的事件：

```html
<!-- CustomInput.vue -->
<script setup>
import { computed } from 'vue'

const props = defineProps(['modelValue'])
const emit = defineEmits(['update:modelValue'])

const value = computed({
  get() {
    return props.modelValue
  },
  set(value) {
    emit('update:modelValue', value)
  }
})
</script>

<template>
  <input v-model="value" />
</template>
```



### `v-model` 的参数

默认情况下，`v-model` 在组件上都是使用 `modelValue` 作为 prop，并以 `update:modelValue` 作为对应的事件

我们可以通过给 `v-model` 指定一个参数来更改这些名字

```html
<MyComponent v-model:title="bookTitle" />
```

```html
<!-- MyComponent.vue -->
<script setup>
defineProps(['title'])
defineEmits(['update:title'])
</script>

<template>
  <input
    type="text"
    :value="title"
    @input="$emit('update:title', $event.target.value)"
  />
</template>
```

我们可以在单个组件实例上创建多个 `v-model` 双向绑定。

组件上的每一个 `v-model` 都会同步不同的 prop，而无需额外的选项

```html
<UserName
  v-model:first-name="first"
  v-model:last-name="last"
/>

<script setup>
defineProps({
  firstName: String,
  lastName: String
})

defineEmits(['update:firstName', 'update:lastName'])
</script>

<template>
  <input
    type="text"
    :value="firstName"
    @input="$emit('update:firstName', $event.target.value)"
  />
  <input
    type="text"
    :value="lastName"
    @input="$emit('update:lastName', $event.target.value)"
  />
</template>
```



### 处理 `v-model` 修饰符

在某些场景下，你可能想要一个自定义组件的 `v-model` 支持自定义的修饰符

组件的 `v-model` 上所添加的修饰符，可以通过 `modelModifiers` prop 在组件内访问到

该prop的默认值是一个空对象

```html
<MyComponent v-model.capitalize="myText" />
```

```html
<script setup>
const props = defineProps({
  modelValue: String,
  modelModifiers: { default: () => ({}) }
})

defineEmits(['update:modelValue'])

console.log(props.modelModifiers) // { capitalize: true }
</script>
```

注意这里组件的 `modelModifiers` prop 包含了 `capitalize` 且其值为 `true`，因为它在模板中的 `v-model` 绑定 `v-model.capitalize="myText"` 上被使用了。

有了这个 prop，我们就可以检查 `modelModifiers` 对象的键，并编写一个处理函数来改变抛出的值

```html
<script setup>
const props = defineProps({
  modelValue: String,
  modelModifiers: { default: () => ({}) }
})

const emit = defineEmits(['update:modelValue'])

function emitValue(e) {
  let value = e.target.value
  if (props.modelModifiers.capitalize) {
    value = value.charAt(0).toUpperCase() + value.slice(1)
  }
  emit('update:modelValue', value)
}
</script>

<template>
  <input type="text" :value="modelValue" @input="emitValue" />
</template>
```



对于又有参数又有修饰符的 `v-model` 绑定，生成的 prop 名将是 `arg + "Modifiers"`

```html
<template>
  <UserName
    v-model:first-name.capitalize="first"
    v-model:last-name.uppercase="last"
  />
</template>
<script setup>
  const props = defineProps({
    firstName: String,
    lastName: String,
    firstNameModifiers: { default: () => ({}) },
    lastNameModifiers: { default: () => ({}) }
  })
  defineEmits(['update:firstName', 'update:lastName'])

  console.log(props.firstNameModifiers) // { capitalize: true }
  console.log(props.lastNameModifiers) // { uppercase: true}
</script>
```



