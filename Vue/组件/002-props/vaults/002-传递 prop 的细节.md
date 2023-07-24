### 命令规范

如果一个 prop 的名字很长，应使用 camelCase 形式，因为它们是合法的 JavaScript 标识符，可以直接在模板的表达式中使用，也可以避免在作为属性 key 名时必须加上引号。

```ts
defineProps({
  userName: String
})
```



虽然理论上你也可以在向子组件传递 props 时使用 camelCase 形式, 但实际上为了和 HTML attribute 对齐

推荐使用`kebab-case `形式向组件传递prop

```vue
<MyComponent user-name="Klaus" />
```



### 动态prop

```vue
<!-- 字符串类型值 -->

<!-- 静态prop -->
<BlogPost title="Hello Vue" />

<!-- 动态prop -->
<BlogPost :title="post.title" />
```

```html
<!-- number类型值 -->

<!-- typeof like -> number -->
<BlogPost :like="42" />

<!-- typeof like -> string -->
<!-- 
	默认情况下, 传递的值都会被视为字符串
	如果需要传递其余类型值，需要结合使用v-bind
	因为此时传递的值会被视为JavaScript表达式并自动求值
-->
<BlogPost like="42" />
```

```html
<!-- 布尔类型值 -->

<!-- 仅写上 prop 但不传值，会隐式转换为 `true` -->
<BlogPost is-published />

<!-- 等价于 -->
<BlogPost :is-published="true" />
```



### 使用一个对象绑定多个 prop

如果你想要将一个对象的所有属性都当作 props 传入，可以使用[没有参数的 `v-bind`](https://cn.vuejs.org/guide/essentials/template-syntax.html#dynamically-binding-multiple-attributes)

```vue
<script>
	const post = {
    id: 1,
    title: 'My Journey with Vue'
  }
</script>

<template>
	<BlogPost v-bind="post" />
	
	<!-- 等价于 -->
	<BlogPost :id="post.id" :title="post.title" />
</template>
```



### 单向数据流

所有的 props 都应该遵循着**单向绑定**原则，props 因父组件的更新而变化，自然地将新的状态向下流往子组件，而不会逆向传递

也就是说

1. 状态是在哪个组件中被定义的，就只能在哪个组件中被修改
2. 父组件传递过来的prop应该被视为只读的

因此如果在子组件中修改了父组件传递过来的prop，Vue 会在控制台上抛出警告

这种更改的主要缺陷是它允许了子组件以某种不明显的方式影响父组件的状态，可能会使数据流在将来变得更难以理解

在大多数场景下，子组件应该[抛出一个事件](https://cn.vuejs.org/guide/components/events.html)来通知父组件需要更改prop对应的值



但如果prop对应的类型值是引用类型，虽然子组件无法更改 props 绑定，但仍然**可以**更改对象或数组内部的值，但是这么做是非常不被推荐的



### 校验

组件可以对传入的prop进行细致的校验，如果传入的值不满足类型要求，Vue 会在浏览器控制台中抛出警告来提醒使用者

```ts
// defineProps在编译时，会单独进行编译
// 这也就意味着在defineProps的参数中不可以访问在<script setup> 中定义的变量
defineProps({
  // 基础类型检查 -- 对于 `null` 和 `undefined` 会 自动跳过任何类型检查
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
    // 对象或数组的默认值必须从一个工厂函数返回。
    // 该函数接收组件所接收到的原始 prop 作为参数。
    default(rawProps) {
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
    // 不像对象或数组的默认，这不是一个工厂函数。这会是一个用来作为默认值的函数
    default() {
      return 'Default function'
    }
  }
})
```



#### 运行时类型检查

校验选项中的 `type` 可以是下列这些原生构造函数：

- `String`
- `Number`
- `Boolean`
- `Array`
- `Object`
- `Date`
- `Function`
- `Symbol`

也可以是`自定义的类或构造函数`, `type` 也可以是自定义的类或构造函数，Vue 将会通过 `instanceof` 来检查类型是否匹配

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





### 特点

1. 所有 prop 默认都是可选的，除非声明了 `required: true`
2. 未传递的可选 prop 将会有一个默认值 `undefined`
   + 布尔类型除外，布尔类型默认值将会是false
3. 当一个prop的值是undefined，那么就会使用默认值