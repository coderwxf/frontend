在模板中，直接调用方法会在每次渲染时重新执行，即使该方法与本次更新无关。

对于依赖于其他状态派生出来的值，最好使用**计算属性**

计算属性只有才**首次使用** 或 **依赖状态** 发生改变时才会被计算



在 Vue 中，通过 `computed` 定义计算属性。

在 React 中，使用 `useMemo` 钩子实现类似的计算缓存机制。



计算属性定义方法看似是方法，其实是一个getter函数

1. 计算属性会在new Vue阶段被挂载到vue实例 「 不能与 `data` 或 `methods` 中的属性或方法同名 」
2. 自动监听 计算属性中使用的响应式状态 「 在状态发生更新时，自动重新计算新值 」`

```js
computed: {
  sum() {
    return this.x + this.y;
  }
}
```



计算属性完整写法如下

```js
computed: {
  fullName: {
    get() {
      return `${this.firstName}-${this.lastName}`;
    },
    set(val) {
     	[this.firstName, this.lastName] = val.split('-')
    }
}
```

1. 计算属性本质是一个响应式状态，不是方法

2. 计算属性`getter/setter`方法内部this值是vue实例

3. 计算属性虽然可以设置`setter`, 但其本质是插入额外执行逻辑

   计算属性应该永远是**只读**的，其具体值应该根据依赖值派生出来



### 计算缓存

![image.png](https://s2.loli.net/2024/09/15/Xwkv2MLFf8JhKoT.png) 



### 惰性计算

计算属性在`new Vue`阶段 仅仅只是被挂载到了实例上，

计算属性除非被使用，否则其`getter/setter` 将永远不会被执行



### 计算属性 vs 方法

**计算属性**

1. 会被getter 和 setter 劫持
2. 存在缓存，适合参与界面渲染



**方法**

1. 执行业务逻辑
2. 没有缓存，不适合参与界面渲染