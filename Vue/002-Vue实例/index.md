我们可以通过`createApp()`来创建一个**`应用实例`**

`createApp()`方法的参数是一个`根组件选项对象`，返回值是当前应用对应的实例对象，也就是`应用实例`

```ts
import { createApp } from 'vue'

const app = createApp({
  setup() {
    // ...
  },
  template: `...`
})
```



在一个Vue应用中，可以创建多个应用实例。每一个应用实例都是相互独立的，拥有自己的全局配置和全局资源

```ts
const app1 = createApp({ /* ... */ })
app1.mount('#container-1')

const app2 = createApp({ /* ... */ })
app2.mount('#container-2')
```



## 组件选项对象

`组件选项对象(组件对象)`是一个包含了Vue各种选项的特殊JavaScript对象

`组件选项对象`可以包含使用 Options API 的`data`、`methods`、`template`等选项来编写Options API风格的组件,

也可以包含使用 Composition API 的 `setup`、`template` 等选项来编写Composition API风格的组件



如果导入一个以单文件组件格式编写的组件，在默认导出的时候，实际上导出的就是一个被转换后的组件选项对象

所以上述示例也可以按照如下形式进行编写

```ts
import { createApp } from 'vue'
import App from 'app.vue'

const app = createApp(App)
```



## 根组件

在实际应用中，组件常常被组织成层层嵌套的树状结构

最上层的那个组件被定义为**根组件**

每个应用都应该**有且只能有一个“根组件”**，其他组件都将作为其根组件的后代组件

![组件树](https://cn.vuejs.org/assets/components.7fbb3771.png) 

## 应用配置

`createApp()`方法返回的是应用实例，可以通过应用实例对整个应用进行`全局配置`和`资源注册`

 应用实例上**绝大多数方法都将返回应用实例自身，以方便进行链式调用**

```js
import { createApp } from 'vue'

const app = createApp({
  /* 根组件选项 */
})

// 链式调用
app
  .component(/.../)
  .use(/.../)
	.directive(/..../)
```

但应用实例上并不是所有的方法都返回应用实例自身

+ `app.config`是应用的全局配置对象，可以在该对象上挂载全局属性和方法

  ```ts
  // App.vue --- 根组件
  app.config.errorHandler = () => { console.log('error handler') } // 全局错误处理事件
  
  // Child.vue --- 子组件
  import { getCurrentInstance } from 'vue'
  
  const instance = getCurrentInstance() // 获取当前组件实例
  const app = instance.appContext // 通过当前组件实例获取根组件实例
  app.config.errorHandler() // => error handler
  ```

  

+ 调用`mount()`将应用挂载到挂载节点中

  + `挂载`真正渲染和显示组件的过程
  + 所以`mount()`返回的是`根组件实例`，而不是`应用实例`

  ```ts
  app.mount('#app')
  ```

  

## 挂载应用

应用实例必须在调用了 `mount()` 后才会渲染出来

`mount()`所接受的参数就是对应的容器元素，可以是以下的任意一种:

1. 一个实际的 DOM 元素
2. 一个 有效的 CSS 选择器字符串

应用根组件的内容将会被渲染在容器元素里面，而**容器元素自己并不会被视为应用的一部分**

`mount()`执行的是渲染行为，所以**`mount()`方法的返回值是根组件实例而非应用实例**

因此`mount()` 方法应该始终在整个应用配置和资源注册完成后才被调用



## 模板

### 定义模板

1. 如果组件存在`template`选项

   + 如果`template`选项的值是`html字符串`，那么就使用`template`选项的值作为对应模板

   + 如果`template`选项的值是`以#开头的id选择器`, 那么就会使用对应DOM元素中的内容作为模板

     `#开头的id选择器`推荐对应以下元素中的任意一种：

     1. `template`元素

        ```html
        <template id="tmp">
         </h2>{{ msg }}</h2>
        </template>
        ```

     2. `type`为`x-template`的`script`元素

        ```html
        <script type="x-template" id="tmp">
        	</h2>{{ msg }}</h2>
        </script>
        ```

2. 如果组件不存在`template`选项，那么就会自动以挂载节点的`innerHTML`作为组件模板

> 不管哪一种形式定义的模板，最终渲染的内容一旦会替换挂载节点下原本存在的内容



### 模板分类

| 分类             | 说明                                                         | 示例                                                         |
| ---------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| **`字符串模板`** | `template`的内容是一个字符串,会直接被Vue的解析器编译<br />可以使用Vue提供的所有功能和语法 | 1. SFC中的`template`选项<br />2. 组件选项对象中的`template`选项对应的值 |
| **`DOM模板`**    | `template`的内容先经过浏览器的解析,然后再给Vue解析器<br />可能受到浏览器语法的限制,不能使用Vue的一些特殊语法 | 1. 在`template`元素中定义的模板内容<br />2. 在`<script type="x-template">`中定义的模板内容 |



#### 语法限制

在绝大多数情况下，尤其是那些使用了构建工具的应用中，使用的模板基本都是`字符串模板`

只有在不依赖于构建工具的应用中，才会使用`DOM模板`



DOM模板在被Vue解析器解析之前，会先经过浏览器的解析

因为html的一些语法限制 (例如在html中，元素并不区分大小写),

所以在DOM模板中的组件名、事件、属性都只能使用`kebab-case`格式命名，



任何`camelCase`格式或`PascalCase`格式命名的变量 都会被转换为`camelcase`格式而导致无法被Vue解析器正常解析

为此Vue提供了将`kebab-case`格式转换为`camelCase`格式或`PascalCase`格式的能力

以确保一套模板既可以作为DOM模板进行解析，也可以作为字符串模板进行解析
