## setup

`setup`是Vue 3中新增的用于编写组合式API选项

通过`setup`可以取代了以往的大多数Option API选项,如data、methods、computed等

`setup`是一个钩子函数，在组件创建时被调用，并用于进行初始化设置

`setup`函数返回一个对象, 在该对象中显示定义的属性和方法可以直接在组件模板中被使用

与传统的Options API不同，`setup`选项中的代码是在组件实例创建之前执行的，因此无法访问组件实例本身



## script setup

在 `setup()` 函数中手动暴露大量的状态和方法非常繁琐，为此Vue提供了`<script setup>` 语法糖来大幅度地简化代码编写流程

`<script setup>` 只能在单文件组件(SFC) 中被使用，因为`<script setup>` 需要在编译的时候被转换为`setup()`写法

换句话来说`<script setup>`必须要结合构建工具才可以使用



### 顶层的绑定会被暴露给模板

当使用 `<script setup>` 的时候

任何在 `<script setup>` 声明的顶层的声明 (例如变量、方法、import导入等)都能在模板中直接使用

模板在编译的时候会被转换为位于同一个作用域的渲染函数，因此自然可以访问顶层的声明

```js
import { ref } from 'vue'

const count = ref(0)

function increment() {
  count.value++
}
```

本质等价于

```js
module.export = {
  setup() {
    import { ref } from 'vue'
    
    const count = ref(0)

    function increment() {
      count.value++
    }

    // 在setup()中 只有在返回的对象中显示定义的属性和方法才可以在模板中被直接使用
    return {
      count,
      increment
    }
  }
}
```





### [ref](https://cn.vuejs.org/guide/essentials/reactivity-fundamentals.html#script-setup)

在 `setup()` 函数中声明并返回ref对象，以便于在模板中使用

```js
import { ref } from 'vue'

export default {
  // `setup` 是一个特殊的钩子，专门用于组合式 API。
  setup() {
    const count = ref(0)

    function increment() {
      // 在 JavaScript 中需要 .value
      count.value++
    }

    // 状态和方法需要被显示导出 才可以在模板中被使用
    return {
      count,
      increment
    }
  }
}
```

在 `setup()` 函数中手动暴露大量的状态和方法非常繁琐

幸运的是，我们可以通过使用[单文件组件 (SFC)](https://cn.vuejs.org/guide/scaling-up/sfc.html) 来避免这种情况。我们可以使用 `<script setup>` 来大幅度地简化代码

```vue
<script setup>
import { ref } from 'vue'

const count = ref(0)

function increment() {
  count.value++
}
</script>

<template>
  <button @click="increment">
    {{ count }}
  </button>
</template>
```

`<script setup>` 中的顶层的导入、声明的变量和函数可在同一组件的模板中直接使用。你可以理解为模板是在同一作用域内声明的一个 JavaScript 函数——它自然可以访问与它一起声明的所有内容。


----

词法上下文
