props 是一种特别的属性，用于声明那些父组件需要传递给子组件的数据

父组件通过自定义属性的方式将props传递给子组件

```html
<BlogPost title="My journey with Vue" />
```

子组件可以通过`defineProps`或`prop选项`来选择性的接收所有父组件传递给子组件的数据

如果父组件传递给子组件的数据并没有被子组件接收，那么这些数据将变成`穿透属性`



在使用 `<script setup>` 的单文件组件中，props 可以使用 `defineProps()` 来声明

`defineProps` 是一个仅 `<script setup>` 中可用的编译宏命令，并不需要显式地导入，声明的 props 会自动暴露给模板

`defineProps` 会返回一个对象，其中包含了可以传递给组件的所有 props

```vue
<script setup>
<!-- 数组形式 --- 仅接收props -->
const props = defineProps(['foo'])
console.log(props.foo)
</script>
```

```js
// 对象形式 --- 可以在接收props的时候进行类型校验
defineProps({
  // key 是 prop 的名称，值是该prop预期类型的构造函数
  title: String,
  likes: Number
})
```



如果你没有使用 `<script setup>`，props 必须以 `props` 选项的方式声明，props 对象会作为 `setup()` 函数的第一个参数被传入

```ts
export default {
  // 数组形式
  props: ['foo'],
  setup(props) {
    // setup() 接收 props 作为第一个参数
    console.log(props.foo)
  }
}
```

```js
// 对象形式
export default {
  props: {
    title: String,
    likes: Number
  }
}
```



## 类型标注

### 使用 `<script setup>`

当使用 `<script setup>` 时，`defineProps()` 宏函数支持从它的参数中推导类型

这被称之为“运行时声明”，因为传递给 `defineProps()` 的参数会作为运行时的 `props` 选项使用

```ts
const props = defineProps({
  // typeof foo -> string
  foo: { type: String, required: true },
  // typeof bar -> number | undefined
  bar: Number
})
```



然而，通过泛型参数来定义 props 的类型通常更直接

这被称之为“基于类型的声明”。编译器会尽可能地尝试根据类型参数推导出等价的运行时选项

```ts
const props = defineProps<{
  foo: string
  bar?: number
}>()
```

基于类型的声明或者运行时声明可以择一使用，但是不能同时使用



#### 语法限制

在defineProps的泛型中可以使用基本数据类型，当前文件中定义的类型和从其他文件导入的类型，但是由于类型到运行时转换仍然基于 AST，一些需要实际类型分析的复杂类型，例如条件类型

可以使用条件类型来指定单个 prop 的类型，但不能用于整个 props 对象的类型

```ts
interface PropsType<Type extends 'string' | 'number'> {
  type: Type;
  data: Array<string | number>;
}

defineProps<{
  type: PropType<'string' | 'number'>;
  // 给单个prop使用条件类型是可以被vue解析的
  value: Type extends 'string' ? string : number;
  data: Array<string | number>;
}>();
```

```ts
interface PropsType {
  type: 'string' | 'number';
  data: Array<string | number>;
  value: PropsType['type'] extends 'string' ? string : number;
}

// 传入含有条件类型的对象，vue在解析的时候会抛出错误
defineProps<PropsType>
```



#### 默认值

当使用基于类型的声明时，我们失去了为 props 声明默认值的能力。这可以通过 `withDefaults` 编译器宏解决

这将被编译为等效的运行时 props `default` 选项.

此外，`withDefaults` 帮助程序为默认值提供类型检查，并确保返回的 props 类型删除了已声明默认值的属性的可选标志

```ts
export interface Props {
  msg?: string
  labels?: string[]
}

const props = withDefaults(defineProps<Props>(), {
  msg: 'hello',
  labels: () => ['one', 'two']
})
```



### 不使用`<script setup>`

如果没有使用 `<script setup>`，那么为了开启 props 的类型推导，必须使用 `defineComponent()`

```ts
import { defineComponent } from 'vue'

export default defineComponent({
  props: {
    message: String
  },
  // 传入 `setup()` 的 props 对象类型是从 `props` 选项中推导而来
  setup(props) {
    props.message // <-- 类型：string
  }
})
```



### 复杂的 prop 类型

对于基于类型的声明，一个 prop 可以像使用其他任何类型一样使用一个复杂类型：

```vue
<script setup lang="ts">
interface Book {
  title: string
  author: string
  year: number
}

const props = defineProps<{
  book: Book
}>()
</script>
```

对于运行时声明可以使用 `PropType` 工具类型

```ts
import { defineComponent } from 'vue'
import type { PropType } from 'vue'

export default defineComponent({
  props: {
    // Object as Book 是错误的
    // 因为prop的类型是PropType --- PropType是一个标准化的prop对象
    book: Object as PropType<Book>
  }
})
```



## 语法细节

### 命名格式

如果一个 prop 的名字很长，应使用 camelCase 形式

```vue
defineProps({
  greetingMessage: String
})
```



为了和 HTML attribute 对齐, 在将属性传递给子组件的时候，通常会将其写为 kebab-case 形式

```html
<MyComponent greeting-message="hello" />
```

对于组件名我们推荐使用 [PascalCase](https://cn.vuejs.org/guide/components/registration.html#component-name-casing)，因为这提高了模板的可读性，能帮助我们区分 Vue 组件和原生 HTML 元素。然而对于传递 props 来说，使用 camelCase 并没有太多优势，因此我们推荐更贴近 HTML 的书写风格



### 传递不同的值类型

```html
<!-- 需要使用v-bind来绑定js表达式 以确保likes传入的值是number类型的42，而不是string类型的42 -->
<BlogPost :likes="42" />

<!-- 只有属性名没有属性值的属性 会被认为是值为true的属性 -->
<BlogPost is-published />
```



#### 对象props

```vue
<script setup>
  const post = {
    id: 1,
    title: 'My Journey with Vue'
  }
</script>

<template>
	<!-- 以下prop传递方式是等价的 -->
	<BlogPost v-bind="post" />
	<BlogPost :="post" />
	<BlogPost :id="post.id" :title="post.title" />
</template>
```



## 单向数据流

所有的 props 都遵循着**单向绑定**原则，props 因父组件的更新而变化，自然地将新的状态向下流往子组件，而不会逆向传递，这避免了子组件意外修改父组件的状态的情况，不然应用的数据流将很容易变得混乱而难以理解，**所以永远不要在子组件中修改某一个prop**



## 校验

Vue 可以对传入组件的props进行类型校验，要声明对 props 的校验，你可以向 `defineProps()` 宏提供一个带有 props 校验选项的对象

`defineProps()` 宏中的参数**不可以访问 `<script setup>` 中定义的变量**，因为在编译时整个表达式都会被移到外部的函数中

+ 所有 prop 默认都是可选的，除非声明了 `required: true`
+ 除 `Boolean` 外的未传递的可选 prop 将会有一个默认值 `undefined`
+ `Boolean` 类型的未传递 prop 将被转换为 `false`。这可以通过为它设置 `default` 来更改
+ 如果一个 prop 的值被解析为 `null` 或 `undefined`，那么 Vue 将跳过对这个 prop 的类型检测
+ 如果声明了 `default` 值，那么在 prop 的值被解析为 `undefined` 时，无论 prop 是未被传递还是显式指明的 `undefined`，都会使用 `default` 值

```ts
defineProps({
  // 基础类型检查
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
    // 对象或数组的默认值, 必须从一个工厂函数返回
    default() {
      return { message: 'hello' }
    }
  },
  // 自定义类型校验函数
  propF: {
    validator(value) {
      return ['success', 'warning', 'danger'].includes(value)
    }
  },
  // 函数类型的默认值
  propG: {
    type: Function,
    // 不像对象或数组的默认值，这不是一个工厂函数。这会是一个用来作为默认值的函数
    default() {
      return 'Default function'
    }
  }
})
```



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

// Vue 会判断author的实际值是否是Person的实例
defineProps({
  author: Person
})
```

