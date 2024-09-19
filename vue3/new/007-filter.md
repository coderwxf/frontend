过滤器用于进行一些格式化操作



过滤器不会被挂载到vue实例上

+ 所以过滤器可以和`data`、`methods`等中方法和属性重名

+ 不推荐，一般情况下，推荐过滤器以`xxxFilter`格式进行定义 
+ 没在实例上挂载，所以只能在视图中通过特殊语法进行使用
+ 在视图之外，filter就没有任何用武之地了



```html
<!-- 过滤器支持链式调用 -->
<div>{{ name | upperFilter | lowerFilter }}</div>
```

```js
// 局部过滤器
filters: {
  upperFilter: v => v.toUpperCase(),
  lowerFilter: v => v.toLowerCase()
}
```



```js
// 全局过滤器
Vue.filter(upperFilter, v => v.toUpperCase())
```



过滤器语法是`值 | 过滤器`，实际会被编译为 `过滤器(值)`

所以过滤器的本质是方法调用，所以每次视图更新，都会导致过滤器需要重新执行 

所以过滤器的性能不如计算属性，推荐优先使用计算属性 「 vue3已经移除了过滤器 」

并且对于方法调用而言，可是传递多个参数，而过滤器只能传递一个参数

「 一定要给过滤器传递多个参数，可以考虑将参数定义对象格式 」



其次，方法调用可以用于指令绑定的值中，而过滤器不可以

```html
<div v-html="name | nameFilter"></div> <!-- error -->
```

过滤器只能用于`mustache`语法 或 `v-bind给某个状态绑定状态`时使用

