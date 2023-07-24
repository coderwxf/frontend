在Vue中，子组件通过**自定义事件系统**向父组件传递数据

1. 子组件在模板中通过`$emit`方法 向父组件抛出一个名为`say`的事件，并传递参数

   ```html
    <button @click="$emit('say', 'Hello Vue')">click me</button>
   ```

   

2. 父组件通过`v-on`指令选择性的监听子组件触发的事件

   ```html
   <Child @say="msg => console.log(msg)"/>
   ```



## 特点

1. 像组件与 `prop` 一样，事件的名字也提供了**自动的格式转换**

   也就是说`kebab-case`格式的事件名和`camelCase`格式的事件名是同一个事件的两个不同名称而已

   和`prop`一样，在模板中也推荐使用`kebab-case`格式来定义事件名



2. 和原生 DOM 事件不一样，组件触发的事件**没有冒泡机制**。只能监听直接子组件触发的事件

   平级组件或是跨越多层嵌套的组件间通信，应使用一个外部的事件总线，或是使用[全局状态管理方案](https://cn.vuejs.org/guide/scaling-up/state-management.html)。



## 声明

和`prop`不一样，对事件的声明在原则上是可选的，但是推荐在子组件中使用[`defineEmits()`](https://cn.vuejs.org/api/sfc-script-setup.html#defineprops-defineemits)对需要触发的事件进行声明

1. 可以作为文档的一部分，便于后期维护

2. 事件本质也是一个对象，因此一个没有被显示声明的事件也是一个穿透属性，会被逐级传递下去

   对事件进行声明可以很好的将事件从穿透属性中剔除出来，方便后期维护，并可以有效的避免一些边界错误

   如孙级组件触发了一个和穿透属性同名的事件所导致的bug



**`使用<script setup>`**

组件可以显式地通过 [`defineEmits()`](https://cn.vuejs.org/api/sfc-script-setup.html#defineprops-defineemits) 宏来声明它要触发的事件

`defineEmits()` 宏**不能**在子函数中使用，须直接放置在 `<script setup>` 的顶级作用域下

在组件的 `<script setup>` 中不能使用`$emit`，为此`defineEmits`方法返回了一个相同功能的函数供我们使用

```ts
const emit = defineEmits(['inFocus', 'submit'])

function buttonClick() {
  emit('submit')
}
```



`不使用<script setup>`

如果显式地使用了 `setup` 函数而不是 `<script setup>`, 那么事件需要通过 [`emits`](https://cn.vuejs.org/api/options-state.html#emits) 选项来进行声明

```ts
export default {
  emits: ['inFocus', 'submit'],
  // emit 函数会被暴露在 setup() 的上下文对象ctx中 -- 也就是setup的第二个参数
  // 因为ctx本质是一个普通的JavaScript对象，所以可以直接通过解构的形式取出emit 函数
  setup(props, { emit }) {
    emit('submit')
  }
}
```



### 定义格式

和`prop`一样，声明事件既可以使用`数组形式`，也可以使用`对象形式`



`数组形式`

```ts
defineEmits(['inFocus', 'submit'])
```



`对象形式`

通过对象语法，我们可以对事件的参数进行校验

```ts
const emit = defineEmits({
  // key - 事件名, value -- 校验函数
  submit(param1, param2) { // 触发事件时候传入的参数 会依次作为校验函数的参数被传入
    // 检验函数需要返回一个布尔类型的值
    // 如果返回true 表示参数校验通过，反之就是校验失败
    return true
  }
})
```



如果使用的是 `TypeScript 搭配 <script setup>`, 则可以使用**编译时声明**来完成相同的功能

```vue
<script setup lang="ts">
const emit = defineEmits<{
  (e: 'change', id: number): void
  (e: 'update', value: string): void
}>()
</script>
```











在Vue中，子组件通过自定义的事件系统向父组件传递数据

```vue
<!-- 子组件 --- 使用 $emit 方法触发自定义事件 -->
<button @click="$emit('someEvent')">click me</button>

<!-- 父组件可以通过 v-on 来监听事件 -->
<MyComponent @some-event.self="callback" />
```

注意: 

 1. 和原生 DOM 事件不一样，组件触发的事件**没有冒泡机制**，只能监听直接子组件触发的事件

    平级组件或是跨越多层嵌套的组件间通信，应使用一个外部的事件总线，或是使用一个[全局状态管理](https://cn.vuejs.org/guide/scaling-up/state-management.html)

 2. 事件名也提供了自动的格式转换

    触发了一个以 camelCase 格式命名的事件，可以使用kebab-case 格式的事件名进行监听

 3. 在模板中，推荐使用 kebab-case 形式来编写事件名



## 事件参数

有时候，在触发事件的时候，需要传递对应的参数

```vue
<!-- 子组件 -->

<!-- $emit(事件名, 参数1, 参数2, ...) -->
<button @click="$emit('increaseBy', 1, 2)">
  Increase by 1
</button>
```

```vue
<!-- 父组件 -->

<!-- 
	触发事件的时候，传入了几个参数
	在触发的事件回调中，就可以接受几个参数
-->
<MyButton @increase-by="(m, n) => count += (m + n)" />
<MyButton @increase-by="increaseCount" />
```



## 声明触发的事件

组件可以显式地通过 [`defineEmits()`](https://cn.vuejs.org/api/sfc-script-setup.html#defineprops-defineemits) 宏来声明它要触发的事件

在`<script setup>`中不可以使用`$emit`, 为此defineEmits方法返回了一个相同功能的函数

`defineEmits()` 宏**不能**在子函数中使用，必须直接放置在 `<script setup>` 的顶级作用域下

```js
const emit = defineEmits(['inFocus', 'submit'])

function buttonClick() {
  emit('submit')
}
```



如果你显式地使用了 `setup` 函数而不是 `<script setup>`

则事件需要通过 [`emits`](https://cn.vuejs.org/api/options-state.html#emits) 选项来定义，`emit` 函数会被暴露在 `setup()` 的上下文对象上

```ts
export default {
  emits: ['inFocus', 'submit'],
  // 声明的事件会被挂载到setup的第二个参数 ctx上，并可以在使用的时候直接解构
  setup(props, { emit }) {
    ctx.emit('submit')
  }
}
```



尽管事件声明是可选的，但还是推荐你完整地声明所有要触发的事件，以此在代码中作为文档记录组件的用法

同时，事件声明能让 Vue 更好地将事件和[透传 attribute](https://cn.vuejs.org/guide/components/attrs.html#v-on-listener-inheritance) 作出区分