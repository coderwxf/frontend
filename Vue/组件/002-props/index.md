在实际应用中，可以通过向组件传递数据来供组件展示和进行交互

默认情况下，传递给组件的数据会作为`穿透属性`，按照单向数据流的方式逐级向下传递给子组件

只有当这个穿透属性在组件中被显示声明，该穿透属性才会被转换为prop属性，并注入到组件实例中



## 使用

### 声明

### 数组形式

在使用 `<script setup>` 的单文件组件中，props 可以使用 `defineProps()` 宏来声明

`defineProps`是一个编译时宏，可以在不被显式引入的情况下，直接在事件中使用

`defineProps`这类编译时宏只能在`<script setup>`语法糖中被使用

```ts
<script setup>
// defineProps会返回一个包含所有props的对象
const props = defineProps(['foo'])

console.log(props.foo)
</script>
```



在没有使用 `<script setup>` 的组件中，prop 可以使用 [`props`](https://cn.vuejs.org/api/options-state.html#props) 选项来声明

```ts
export default {
  props: ['foo'],
  // props会作为setup函数的第一个参数被传入
  setup(props) {
    console.log(props.foo)
  }
}
```



### 对象形式

对象形式的 props 声明不仅可以一定程度上作为组件的文档

而且如果其他开发者在使用你的组件时传递了错误的类型，会在浏览器控制台中抛出警告

```ts
// 使用 <script setup>
defineProps({
  // key - 属性名
  // value - 属性预期类型对应的构造函数
  title: String,
  likes: Number
})
```

```ts
// 非 <script setup>
export default {
  props: {
    title: String,
    likes: Number
  }
}
```



如果你正在搭配 TypeScript 使用 `<script setup>`，也可以使用类型标注来声明 props

```ts
<script setup lang="ts">
defineProps<{
  title?: string
  likes?: number
}>()
</script>
```



### 使用

一个组件可以有任意多的 props，默认情况下，所有 prop 都接受任意类型的值

当一个 prop 被注册后，就可以向自定义属性那样被传递

```html
<BlogPost title="My journey with Vue" />
```



## 类型标注

 [类型标注](vaults/001-类型标注.md) 



## 传递 prop 的细节

 [传递 prop 的细节](vaults/002-传递 prop 的细节.md) 
