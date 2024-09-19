vue 构建视图方式

1. template
2. JSX 
   + 「 单大括号语法，大胡子语法 」
   + template构建视图 编程性较弱，JSX编程性较高，但编写难度大



template语法主要构成

1. 插值语法 「 mustache, 双大括号语法，小胡子语法 」
2. 指令 
   + `v-xxx.修饰符.修饰符:参数 = 值`
   + 本质就是供vue解析的自定义属性



无论是大胡子语法还是小胡子语法，其中插入的值都是JavaScript表达式



## 渲染

**基本数据类型**

1. `null/undefined` 渲染为空
2. 其余情况都可以理解为都转换为了字符串 ：`String([value])`

```js
String(10n) // => '10'
String(Symbol()) // => 'Symbol()'
```



**引用数据类型**

1. 普通对象/数组对象，是基于 JSON.stringify 变为JSON字符串进行渲染
2. 其余的对象，也可以理解为都是基于 String([value]) 变为字符串进行渲染

```js
JSON.stringify({name: 'Klaus'}) // => '{"name":"Klaus"}'
JSON.stringify([1, 2, 3]) // => '[1, 2, 3]'

// 没有参数，返回undefined 「不是字符串」
JSON.stringify() // => undefined 

JSON.stringify(/\d/) // => ’{}‘ // 转换失败，默认返回空对象
```



在模板中，vue定义了一个[白名单](https://github.com/vuejs/vue/blob/v2.6.10/src/core/instance/proxy.js#L9)

只有白名单中的全局属性和方法才可以在模板中使用

常见的有 JSON.parse/stringify、parseInt、Number、String等



## 指令

所有以 `v-` 开头的内容都是 Vue 提供的指令

指令其实就是给标签设置“自定义属性”

Vue在渲染视图的时候，会识别这些属性，根据不同的属性，实现不同的效果

```shell
v-xxx.修饰符:参数 = "值/状态"
```



### 设置内容「 非表单元素 」

+ `v-html`：等同于`innerHTML`
  + 可以识别HTML字符串，把字符串中出现的标签渲染为DOM结构
  + 注意需要避免XSS攻击

+ `v-text`：等同于`innerText`
  + 会把所有内容都当做普通文本渲染
  + 类似于小胡子语法，但是没有小胡子语法灵活

在同一个元素上，`v-html` 和 `v-text` 会覆盖掉 Mustache 插值的内容。




### 显示或隐藏

`v-show`和`v-if`用于控制元素的显示和隐藏 「 指令值都是布尔值 」

+ `v-show` => 基于`display='none'`来控制元素显示隐藏
+ `v-if「/v-else/v-else-if」`=> 基于控制元素的“销毁”和“渲染”，来控制元素显示隐藏
  + `v-if`是惰性的，如果初始状态为`false` => 什么都不渲染
  + `v-if/v-else/v-else-if`之间必须紧挨着，不能有其余元素存在




1. 频繁切换使用`v-show`, 不频繁切换使用`v-if`
2. 如果`v-if`和`v-show`出现于同一个元素上，`v-if` 优先级较高 「 不推荐 」
   + 如果`v-if`的值是false，则直接不渲染
   + 如果`v-if`的值是true，再根据`v-show`的值判断元素是否显示



### `v-for` 

用来创建循环元素

```shell
v-for = "(item, index) [in|of] 可迭代对象
```

1.  谁需要重复渲染，就给谁添加 `v-for`
2.  `in`和`of`的实际效果是一致的 「 in 或 of 的前后要有空格 」
3.  `(item, index)`的大括号是可以省略的 「 不推荐 」
4.  需要给循环的元素设置唯一不变的key值 「 值一般为string | number 」以用于优化DOM-DIFF算法
5.  `v-for`可以在迭代时 直接进行解构操作



迭代的值

+ 数组 「 `(item, index) in arr` 」
+ 数字 「 `(item, index) in <num>` 」 --- 假设`num`是`5`，则按照 `[1, 2, 3, 4, 5]`进行解析
+ 字符串 「 `(item, index) in 'Klaus'` 」 --- 作为字符数组进行解析
+ 对象 「 (value, key, index) in user 」
  + 本质迭代的是`Object.entries(user)` 「 特殊处理 」
  + 所以只能迭代自身所有可迭代的非Symbol类型值
+ 其余可迭代对象，会先`Array.from([value])`转换为数组后，再按照数组规则进行迭代



强烈不建议v-for和v-if作用在相同的元素上

+ Vue2中：v-for的优先级高于v-if，如果作用在相同元素上，很可能出现刚创建就被销毁「浪费性能」

  ```html
  <template>
    <div>
      <ul>
        <!-- Vue2: 先执行 v-for，再执行 v-if -->
        <li v-for="item in items" :key="item.id" v-if="item.isVisible">
          {{ item.name }}
        </li>
      </ul>
    </div>
  </template>
  
  <script>
  export default {
    data() {
      return {
        items: [
          { id: 1, name: 'Item 1', isVisible: true },
          // 这个项会被先渲染出来再销毁
          { id: 2, name: 'Item 2', isVisible: false },  
          { id: 3, name: 'Item 3', isVisible: true }
        ]
      };
    }
  };
  </script>
  ```

  

+ Vue3中：v-if的优先级高于v-for，如果作用在相同元素上，这样在v-if中是无法使用v-for中item/index

  ```html
  <template>
    <div>
      <ul>
        <!-- Vue3: 先执行 v-if，再执行 v-for -->
        <!-- 执行v-if的时候，v-for还没有被执行，所以不存在item.isVisible 「报错」 -->
        <li v-if="item.isVisible" v-for="item in items" :key="item.id">
          {{ item.name }}
        </li>
      </ul>
    </div>
  </template>
  
  <script>
  export default {
    data() {
      return {
        showItems: false,
        items: [
          { id: 1, name: 'Item 1' },
          { id: 2, name: 'Item 2' },
          { id: 3, name: 'Item 3' }
        ]
      };
    }
  };
  </script>
  ```

  

推荐使用`template标签`「 占位空标签 」, 将`v-for`和`v-if`分开 

在 Vue 2 中，`template` 标签不能直接设置 `key`。如

在 Vue 3 中，`template` 标签可以设置 `key`。Vue 会使用这个 `key` 来跟踪整个子元素块的变化

```html
<template v-for="item in items" :key="item.id">
  <div>{{ item.name }}</div>
  <span>{{ item.description }}</span>
</template>
```



### 事件绑定

#### methods

1. 在`new Vue`阶段

   + 使用`bind`方法，将函数中的this修改为vue实例对象，

     并将修正this后的新method挂载到组件实例上

     所以method不要写成箭头函数 

     

2. 修改methods，视图不需要更新，所以methods中的方法不需要进行响应式处理



#### v-on

`v-on`（简写`@`）用于实现事件绑定

`v-on`的值可以是函数本身，也可以是函数调用



```vue
<button @click="fun"> click me </button>
```

点击后，会执行fun方法，并传入事件对象



```html
<button @click="fn(10, 20, $event)"> click me </button>
```

点击后，内部会执行函数调用语句，并使用事件对象替换`$event`

> 这里传入的`fun`和`fn(10, 20, $event)`的本质其实是字符串



```html
<button @click="name = 'Klaus'"> click me </button>
```

如果只是简单逻辑，可以直接在事件处理函数中，直接编写业务逻辑



### 基本原理

```html
<button @click.stop="fun">click me</button>
```

会被解析为指令对象

```js
{
  name: 'on', /* 没有v-前缀 */
  arg: ['click'],
  value: fun,
  expression: 'fun',
  modifiers: {
		stop: true
	}
}
```

执行伪代码

```js
<按钮元素>.addEventListener(click, function(e) {
  // 一些额外逻辑 「 如对事件修饰符进行处理 」
  if (modifiers.stop) {
    e.stopPagination()
  }
  
  // 以独立函数调用形式调用绑定方法
  fun(e)
})
```



所以

```js
const vm = new Vue({
	methods: {
		foo() {} // foo内部this 就是vm
	}
})

vm.bar = function() {} // bar内部this 遵循独立函数调用
```



### 修饰符

1. 对应的修饰符「可以同时用多个」
2. 可以只有修饰符，没有绑事件处理函数 

```shell
@click.xxx.xxx
```



| 普通修饰符 | 描述                                                         |
| ---------- | ------------------------------------------------------------ |
| stop       | 阻止事件(冒泡)传播                                           |
| prevent    | 阻止默认行为                                                 |
| capture    | 捕获阶段触发<br />也就是说默认情况下，事件按照冒泡方式执行   |
| self       | 只有事件源是本身时才会触发<br />「 `e.target === e.currentTarget` 」 |
| once       | 事件绑定只处理一次，处理完毕后，移除事件绑定                 |
| passive    | 禁用`preventDefault` 「 提高高频事件的流畅性 」<br />如果事件处理函数依旧调用`preventDefault`, 则抛出警告⚠️ |



| 按键修饰符 | 描述                                                         |
| ---------- | ------------------------------------------------------------ |
| [keyword]  | 各种按键修饰符 <br />「 如enter/tab/delete/esc/space/up/right/down/left/page-down等 」 |
| [number]   | 按键码 「 如`@keyup.13='xxx'` 等价于 `@keyup.enter='xxx'` 」 |
| 自定义按键 | `Vue.config.keyCodes.kk = 13` 「 通过`Vue.config`进行全局配置 」<br />`@keyup.kk='xxx'` 也是按下 Enter 键才触发 |
| 系统按键   | `ctrl/alt/shift/meta`                                        |
| 组合按键   | `@keyup.alt.65` -> 同时按 Alt+a                              |
| 鼠标按键   | `left/right/middle`                                          |
| exact      | 实现精准匹配<br />`@click.ctrl.exact`表示只有ctrl键被按下时触发<br />`@click.ctrl`表示有ctrl键按下即可 「 可以是ctrl键和其它键一起按下 」 |



### 事件委托

在循环元素上添加事件绑定。vue并不会自动进行事件委托操作，需要手动进行事件提升

```html
<div id="app">
  <button
    v-for="user in users"
    :key="user"
    @click="printUser(user)"
  >
    {{ user }}
  </button>
</div>

<script>
  const vm = new Vue({
    data: {
      users: ['Klaus', 'Alex', 'Steven']
    },
    methods: {
      printUser(user) {
        console.log(user)
      }
    }
  });

  vm.$mount('#app');
</script>
```

上述代码修改为

```html
<div id="app">
  <div @click="printUser">
    <button
      v-for="user in users"
      :key="user"
      :data-user="user"
    >
      {{ user }}
    </button>
  </div>
</div>

<script>
  const vm = new Vue({
    data: {
      users: ['Klaus', 'Alex', 'Steven']
    },
    methods: {
      printUser(e) {
        const target = e.target
        if (target.tagName === 'BUTTON' && target.dataset.user) {
          console.log(target.dataset.user)
        }
      }
    }
  });

  vm.$mount('#app');
</script>
```



### 属性绑定

v-bind（简写:） 用于动态绑定元素的属性

v-bind 主要有两个作用：

1. 绑定非字符串类型值。
2. 将JavaScript表达式绑定到元素的属性上
3. 组件也可以使用`v-bind`，用于传递`props`

```vue
<!-- v-bind的值可以是任何合法JavaScript表达式 -->
<div :age="23"></div> <!-- 这里的age是number类型值 -->

<div age="23"></div>  <!-- 这里的age是string类型值 -->
```



### 设置内容 「 表单元素 」

`v-model`实现了数据的双向绑定。当表单元素的内容发生变化时，对应的状态也会自动更新。「 视图 驱动 状态 修改 」

```vue
<input type="text" :value="name" @input="e => name = e.target.value">
```

简化为了

```html
<input type="text" v-model="name">
```



#### 修饰符

| 修饰符 | 功能                                                         |
| ------ | ------------------------------------------------------------ |
| lazy   | 将监听事件从默认的 `input` 改为 `change`，从而实现类似**节流**的效果<br />即只在输入框失去焦点或内容真正变化后再更新绑定的值 |
| number | 默认情况下，表单内容是字符串形式<br />使用 `number` 修饰符后，会将输入的内容使用`parseFloat`方法进行转换<br />如果无法转换，静默失效 |
| trim   | 去除表单内容的首尾空格后，再赋值给对应的状态。               |

```html
<textarea v-model.trim="message"></textarea>
```

1. `v-model`和`trim`一起使用 是标配
2. 文本域绑定值不能使用小胡子语法，只能使用`v-model`



#### 绑定值

> 只要多个表单元素绑定的状态是同一个，则这多个表单元素就变成了一组
>
> 「  原生是根据表单元素的name属性来进行分组  」



 `radio`：无论存在多少个，`v-model` 绑定的值是选中项对应的 `value`

+ 如果状态值 等于 value => 选中，否则就是不选中状态

+ 原生 `input.radio` 是否选中 取决于 `checked`属性

> 原生情况下，单选radio被选中无法取消 
>
> 「 除非选了同组中另一个单选按钮 或 通过脚本移除 」
>
> 单选radio的value默认值是null
>
> 推荐单选按钮的值 就绑定为布尔值



1. 状态值和radio的value依次匹配 => 成功加上checked，否则移除checked
2. 监听radio的change事件 
   + 只有radio被选中，才会触发change事件 「 也会触发input事件，等同于change事件 」
   + 将绑定的value值赋值给状态



多选 `checkbox`：

+ 如果是单选 checkbox，`v-model` 绑定的值是布尔值
  + 如果设置或修改的值不是布尔值，会转换为布尔值
  + 所以单选checkbox只要设置`v-model`，不需要设置`value`
  + 其是根据状态值是否为true，来控制是否选中
  + 可以通过`true-value` 和 `false-value`来修改checkbox切换时候的值『 使其不再是布尔类型值 』
  
  ```html
  <input
    type="checkbox"
    v-model="toggle"
    true-value="yes"
    false-value="no" 
  />
  ```
  
  
+ 如果是一组 checkbox，`v-model` 绑定的是选中项的 `value` 构成的数组。 

> 1. 单个checkbox 自己就能选中 或 取消选中
> 2. `checkbox`实现原理 和 `radio`类似



对于 `select` 元素，单选时 `v-model` 是选中项的 `value`，多选时 `v-model` 是选中项的 `value` 构成的数组。

```html
<div id="app">
  <input type="radio" id="one" value="One" v-model="picked" />
  <label for="one">One</label>

  <input type="radio" id="two" value="Two" v-model="picked" />
  <label for="two">Two</label>

  <div>Picked: {{ picked }}</div>
</div>

<script>
  const vm = new Vue({
    data: {
      picked: 'One'
    }
  });

  vm.$mount('#app');
</script>
```



如果 `v-model` 表达式的初始值不匹配任何一个选择项，`<select>` 元素会渲染成一个“未选择”的状态

在 iOS 上，这将导致用户无法选择第一项，因为 iOS 在这种情况下不会触发一个 change 事件。

因此，建议始终提供一个空值的禁用选项

```vue
<select v-model="citys">
  <option value="" disabled>请选择</option>
  <option value="beijing">北京</option>
  <option value="shanghai">上海</option>
  <option value="guangzhou">广州</option>
  <option value="shenzhen">深圳</option>
  <option value="hangzhou">杭州</option>
</select>
```



### 渲染优化

#### v-pre

`v-pre` 用于指示 Vue 跳过这个元素及其后代元素的编译。

对于含有静态内容的区域，使用 `v-pre` 可以避免不必要的编译过程，直接输出内容，提升性能。具体来说，它会忽略数据绑定和指令的编译，并原样渲染元素的 HTML。

Vue 3 对静态内容的处理做了很多优化。它引入了**静态提升**和**静态节点缓存**，自动识别静态节点并跳过这些节点的重复编译。

因此，对于静态内容，Vue 3 的默认行为已经足够高效，通常不再需要手动使用 `v-pre` 来提升性能。



#### v-cloak

`v-cloak` 是 Vue 中用于防止模板在数据未完全渲染时出现的闪烁问题。

当 Vue 的实例尚未挂载到 DOM 上时，可以通过 `[v-cloak]` 选择器隐藏尚未渲染的元素，一旦 Vue 完成模板的解析和渲染，`v-cloak` 属性会自动移除。

使用 SFC 时, 导出的本质是组件对应的JavaScript文件 「 此时模板已经静态编译成了render函数 => render函数返回vdom，类似于React的render函数 」

所以:

1. 在运行前，模板就已经静态编译完全
2. 导出的是JS文件，挂载点下不存在模板内容

因此，通常不需要依赖 `v-cloak` 来防止模板渲染不完整的情况。

`v-cloak`仅用于通过CDN引入vue 「 不通过构建工具使用vue 」的情况下才需要使用



####  v-once

当使用 `v-once` 指令时，Vue 只会在组件首次渲染时对该元素及其子元素进行编译和渲染。

此后，无论状态如何变化，带有 `v-once` 的元素及其内容都不会重新渲染



#### v-memo



---

---

v-xxx.修饰符:参数 = "值/状态"

状态是函数时，传递参数









value: fun,
  expression: 'fun',



passive
