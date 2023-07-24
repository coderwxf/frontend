每个组件都会经历诞生到销毁的整个过程，这个过程被称之为生命周期

Vue预先提供了一系列名为生命周期钩子的函数，这些函数会在组件的一些关键时刻被回调，以便于让开发者有机会在特定阶段运行自己的代码

```ts
import { onMounted } from 'vue'

onMounted(() => {
  console.log('the component is now mounted.')
})
```



Vue 的生命周期钩子(`onMounted`、`onUpdated` 等)需要在组件初始化时被同步注册, 也就是说需要在组件初始化的过程中直接调用这些钩子函数进行注册。

如果在组件初始化完毕后再去异步注册这些钩子, 就无法确保组件实例在钩子函数被调用时一定存在,可能会导致挂载失败或其他错误。



但这并不意味着对 `onMounted` 的调用必须放在 `setup()` 或 `<script setup>` 内的词法上下文(其实就是词法作用域)中

`onMounted()` 也可以在一个外部函数中调用，只要调用栈是同步的，且最终起源自 `setup()` 就可以

```ts
setTimeout(() => {
  onMounted(() => {
    // 异步注册时当前组件实例已丢失
    // 这将不会正常工作
  })
}, 100)
```



![组件生命周期图示](https://cn.vuejs.org/assets/lifecycle.16e4c08e.png) 

----

---

- 词法作用域(Lexical Scope):一个变量的作用域在它定义的位置决定,与调用的位置无关。JavaScript采用词法作用域。
- 动态作用域(Dynamic Scope):一个变量的作用域取决于调用它的函数,而不是定义的位置。调用位置决定变量作用域。
