有的时候，我们希望组件分享相同的视觉布局，但有不同的内容, 要实现这样的效果自然必须向组件中传递数据，这就会使用到 props

Props 是一种特殊的穿透属性，传递给组件的属性默认都是穿透属性, 只有在组件上声明注册了的属性，才会被**转换为**props



通过 defineProps 宏 在 props 列表中声明props

`defineProps` 是一个仅 `<script setup>` 中可用的编译宏命令，并不需要显式地导入，声明的 props 会自动暴露给模板

```html
<script setup>
defineProps(['title'])
</script>

<template>
  <h4>{{ title }}</h4>
</template>
```

`defineProps` 会返回一个对象，其中包含了可以传递给组件的所有 props

```ts
const props = defineProps(['title'])
console.log(props.title)
```



如果你没有使用 `<script setup>`，props 必须以 `props` 选项的方式声明，props 对象会作为 `setup()` 函数的第一个参数被传入

```ts
export default {
  props: ['title'],
  setup(props) {
    console.log(props.title)
  }
}
```



一个组件可以有任意多的 props，默认情况下，所有 prop 都接受任意类型的值。

当一个 prop 被注册后，可以像这样以自定义 attribute 的形式传递数据给它

```html
<BlogPost title="My journey with Vue" />
<BlogPost title="Blogging with Vue" />
<BlogPost title="Why Vue is so fun" />
```



## 类型标注

### 使用 `<script setup>`

当使用 `<script setup>` 时，`defineProps()` 宏函数支持从它的参数中推导类型

这被称之为“运行时声明”，因为传递给 `defineProps()` 的参数会作为运行时的 `props` 选项使用

```ts
const props = defineProps({
  foo: { type: String, required: true },
  bar: Number
})

props.foo // string
props.bar // number | undefined
```



然而，通过泛型参数来定义 props 的类型通常更直接

这被称之为“基于类型的声明”。编译器会尽可能地尝试根据类型参数推导出等价的运行时选项

```ts
<script setup lang="ts">
const props = defineProps<{
  foo: string
  bar?: number
}>()
</script>
```



但是当使用基于类型的声明时，我们失去了为 props 声明默认值的能力

这可以通过 `withDefaults` 编译器宏解决

```js
export interface Props {
  msg?: string
  labels?: string[]
}

// 这将被编译为等效的运行时 props `default` 选项
const props = withDefaults(defineProps<Props>(), {
  msg: 'hello',
  labels: () => ['one', 'two']
})
```

此外，`withDefaults` 帮助程序为默认值提供类型检查，并确保返回的 props 类型删除了已声明默认值的属性的可选标志



基于类型的声明转换为等价的运行时声明基于AST，而不是基于实际的类型分析

所以那些需要根据上下文进行实际类型推断的类型，如条件类型是不被支持的

我们可以使用条件类型来指定单个 prop 的类型，但不能用于整个 props 对象的类型



>  基于类型的声明或者运行时声明可以择一使用，但是不能同时使用



### 非 `<script setup>` 场景下

如果没有使用 `<script setup>`，那么为了开启 props 的类型推导，必须使用 `defineComponent()`

传入 `setup()` 的 props 对象类型是从 `props` 选项中推导而来

```js
import { defineComponent } from 'vue'

export default defineComponent({
  props: {
    message: String
  },
  setup(props) {
    props.message // <-- 类型：string
  }
})
```



### 复杂的 prop 类型

通过基于类型的声明，一个 prop 可以像使用其他任何类型一样使用一个复杂类型

```ts
interface Book {
  title: string
  author: string
  year: number
}

const props = defineProps<{
  book: Book
}>()
```

对于运行时声明，我们可以使用 `PropType` 工具类型

```ts
import type { PropType } from 'vue'

const props = defineProps({
  book: Object as PropType<Book>
})
```

其工作方式与直接指定 `props` 选项基本相同

```ts
import { defineComponent } from 'vue'
import type { PropType } from 'vue'

export default defineComponent({
  props: {
    book: Object as PropType<Book>
  }
})
```



## Props 声明

一个组件需要显式声明它所接受的 props，这样 Vue 才能知道外部传入的哪些是 props，哪些是透传 attribute 

在使用 `<script setup>` 的单文件组件中，props 可以使用 `defineProps()` 宏来声明：

```html
<script setup>
const props = defineProps(['foo'])

console.log(props.foo)
</script>
```

在没有使用 `<script setup>` 的组件中，prop 可以使用 [`props`](https://cn.vuejs.org/api/options-state.html#props) 选项来声明：

```js
export default {
  props: ['foo'],
  setup(props) {
    // setup() 接收 props 作为第一个参数
    console.log(props.foo)
  }
}
```

注意传递给 `defineProps()` 的参数和提供给 `props` 选项的值是相同的，两种声明方式背后其实使用的都是 prop 选项。



除了使用字符串数组来声明 prop 外，还可以使用对象的形式：

```js
// 使用 <script setup>
defineProps({
  title: String,
  likes: Number
})
```

```js
// 非 <script setup>
export default {
  props: {
    title: String,
    likes: Number
  }
}
```

对于以对象形式声明中的每个属性，key 是 prop 的名称，而值则是该 prop 预期类型的构造函数

对象形式的 props 声明不仅可以一定程度上作为组件的文档，而且如果其他开发者在使用你的组件时传递了错误的类型，也会在浏览器控制台中抛出警告



## 传递 prop 的细节

如果一个 prop 的名字很长，应使用 camelCase 形式，因为它们是合法的 JavaScript 标识符，可以直接在模板的表达式中使用，也可以避免在作为属性 key 名时必须加上引号

虽然理论上你也可以在向子组件传递 props 时使用 camelCase 形式 (使用 [DOM 模板](https://cn.vuejs.org/guide/essentials/component-basics.html#dom-template-parsing-caveats)时例外)，但实际上为了和 HTML attribute 对齐，我们通常会将其写为 kebab-case 形式：

对于组件名我们推荐使用 [PascalCase](https://cn.vuejs.org/guide/components/registration.html#component-name-casing)，因为这提高了模板的可读性，能帮助我们区分 Vue 组件和原生 HTML 元素。然而对于传递 props 来说，使用 camelCase 并没有太多优势，因此我们推荐更贴近 HTML 的书写风格。



### 传递不同的值类型

**任何**类型的值都可以作为 props 的值被传递

```html
<!-- 虽然 `42` 是个常量，我们还是需要使用 v-bind -->
<!-- 因为这是一个 JavaScript 表达式而不是一个字符串 -->
<BlogPost :likes="42" />

<!-- 仅写上 prop 但不传值，会隐式转换为 `true` -->
<BlogPost is-published />

<!-- 
	如果你想要将一个对象的所有属性都当作 props 传入
	你可以使用没有参数的 v-bind，即只使用 v-bind 而非 :prop-name 
-->
<BlogPost v-bind="post" />

<!-- 这实际上等价于 -->
<BlogPost :id="post.id" :title="post.title" />
```



## 单向数据流

所有的 props 都遵循着**单向绑定**原则，props 因父组件的更新而变化，自然地将新的状态向下流往子组件，而不会逆向传递

这避免了子组件意外修改父组件的状态的情况，不然应用的数据流将很容易变得混乱而难以理解

每次父组件更新后，所有的子组件中的 props 都会被更新到最新值

这意味着props应该都被认为是只读的，**不应该**在子组件中去更改一个 prop

若你这么做了，Vue 会在控制台上向你抛出警告(开发环境)



当对象或数组作为 props 被传入时，虽然子组件无法更改 props 绑定，但仍然**可以**更改对象或数组内部的值。这是因为 JavaScript 的对象和数组是按引用传递，而对 Vue 来说，禁止这样的改动，虽然可能生效，但有很大的性能损耗，比较得不偿失

这种更改的主要缺陷是它允许了子组件以某种不明显的方式影响父组件的状态，可能会使数据流在将来变得更难以理解

在大多数场景下，子组件应该[抛出一个事件](https://cn.vuejs.org/guide/components/events.html)来通知父组件做出改变



## Prop 校验

Vue 组件可以更细致地声明对传入的 props 的校验要求

比如我们上面已经看到过的类型声明，如果传入的值不满足类型要求，Vue 会在浏览器控制台中抛出警告来提醒使用者。这在开发给其他开发者使用的组件时非常有用。

要声明对 props 的校验，你可以向 `defineProps()` 宏提供一个带有 props 校验选项的对象

```ts
defineProps({
  // 基础类型检查
  // （给出 `null` 和 `undefined` 值则会跳过任何类型检查）
  propA: Number,
  // 多种可能的类型
  propB: [String, Number],
  // 必传，且为 String 类型
  propC: {
    type: String,
    required: true
  },
  // Number 类型的默认值
  propD: {
    type: Number,
    default: 100
  },
  // 对象类型的默认值
  propE: {
    type: Object,
    // 对象或数组的默认值
    // 必须从一个工厂函数返回。
    // 该函数接收组件所接收到的原始 prop 作为参数。
    default(rawProps) {
      return { message: 'hello' }
    }
  },
  // 自定义类型校验函数
  propF: {
    validator(value) {
      // The value must match one of these strings
      return ['success', 'warning', 'danger'].includes(value)
    }
  },
  // 函数类型的默认值
  propG: {
    type: Function,
    // 不像对象或数组的默认，这不是一个工厂函数。这会是一个用来作为默认值的函数
    default() {
      return 'Default function'
    }
  }
})
```

一些补充细节：

+ `defineProps()` 宏在编译时，整个表达式都会被移到外部的函数中进行解析

  所以props的默认值不可以是` <script setup> `中定义的其他变量

- 所有 prop 默认都是可选的，除非声明了 `required: true`

- 除 `Boolean` 外的未传递的可选 prop 将会有一个默认值 `undefined`

- `Boolean` 类型的未传递 prop 将被转换为 `false`

  这可以通过为它设置 `default` 来更改——例如：设置为 `default: undefined` 将与非布尔类型的 prop 的行为保持一致

- 如果声明了 `default` 值，那么在 prop 的值被解析为 `undefined` 时，无论 prop 是未被传递还是显式指明的 `undefined`，都会改为 `default` 值



### 运行时类型检查

校验选项中的 `type` 可以是下列这些原生构造函数：

- `String`
- `Number`
- `Boolean`
- `Array`
- `Object`
- `Date`
- `Function`
- `Symbol`

另外，`type` 也可以是自定义的类或构造函数，Vue 将会通过 `instanceof` 来检查类型是否匹配

```ts
class Person {
  constructor(firstName, lastName) {
    this.firstName = firstName
    this.lastName = lastName
  }
}

defineProps({
  author: Person
})
```



## Boolean 类型转换

为了更贴近原生 boolean attributes 的行为，声明为 `Boolean` 类型的 props 有特别的类型转换规则

```html
<!-- 等同于传入 :disabled="true" -->
<MyComponent disabled />

<!-- 等同于传入 :disabled="false" -->
<MyComponent />
```

当一个 prop 被声明为允许多种类型时，`Boolean` 的转换规则也将被应用

然而，当同时允许 `String` 和 `Boolean` 时，有一种边缘情况

​		——只有当 `Boolean` 出现在 `String` 之前时，`Boolean` 转换规则才适用：

````ts
// disabled 将被转换为 true
defineProps({
  disabled: [Boolean, String]
})

// disabled 将被转换为 true
defineProps({
  disabled: [Number, Boolean]
})
  
// disabled 将被解析为空字符串 (disabled="")
defineProps({
  disabled: [String, Boolean]
})
````

