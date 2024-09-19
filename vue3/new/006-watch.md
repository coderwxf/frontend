## 基本语法

```js
watch: {
  // newV - 新值, oldV - 旧值
  // this.username 也是新值
  username(newV, oldV) {
		// watch中this是vue实例
    console.log('监听: ', this.username, newV, oldV);
  }
}
```

```js
watch: {
  // 侦听器每次只能箭头一个值的改变，并执行对应逻辑
  username: {
    handler(newV, oldV) {
      console.log('监听: ', this.username);
    },
    // 默认情况下，new Vue时，不执行侦听器
    // 只有状态改变，才会执行侦听器
    immediate: true, // 默认值false => 值为true，new Vue时立即执行一次侦听器
    deep: true // 默认值false => 值为true，为对象开启深度监听
  }
}
```

```js
watch: {
	// 直接监听属性 「 new Vue阶段调用时，newV是初始值，oldV是undefined 」
  'user.name'(newV, oldV) {
    console.log('监听: ', this.username, newV, oldV);
  }
}
```



## $watch

```js
// $watch 返回 unwatch 用于取消 对于obj的侦听
let unwatch = this.$watch(
  'obj', // 侦听对象
  function () { // handler函数
    console.log('监听: ', this.obj);
  },
  { // 侦听器配置对象
    immediate: true,
    deep: true
  }
);
```



## 额外写法

```js
watch: {
  name: {
    // 如果绑定的是字符串，会自动去vue实例上查找对应方法进行调用
    handler: 'handleNameChange',
    immediate: true
  }
},
methods: {
  handleNameChange(newV, oldV) {
    console.log(newV, oldV)
  }
}
```



```js
watch: {
  // 为状态绑定多个侦听处理函数
  msg: [
    {
      handler: function () {
        console.log('MSG监听1');
      },
      immediate: true
    },
    function () {
        console.log('MSG监听2');
    }
  ]
}
```



## 简单原理

在`new Vue`的时候，先执行 `initWatch`, 在此方法取出`Object.values(watch配置项)`并迭代

1. 如果是对象或函数，直接丢给`createWatcher`方法进行处理
2. 如果是数组，迭代数组中的每一项，丢给`createWatcher`方法进行处理
3. 如果是字符串，取出`methods`中对应的方法，丢给`createWatcher`方法进行处理



`createWatcher`调用的时候，参数只能是数组或字符串了

`createWatcher`会取出函数和配置对象，然后丢给`vm.$watch`方法

所以无论那种形式定义的侦听器，其最终的本质都是通过`$watch`方法进行监听



在`$watch`内部，会通过`Watcher类实例`对状态进行侦听，并在其变化的时候执行对应逻辑





## watch vs computed

### `computed`（计算属性）

- **功能**: 用于根据现有状态派生出一个新的值。

- 特点

  - 自动监听其依赖的状态。
  - 具有计算缓存功能，只有在依赖状态改变时才会重新计算。
  - 计算属性本质上是一个新的状态，因此会进行响应式处理。
  - 本质是派生出一个新状态值，所以新状态值应该是只读的。
  - 不应与 `data`、`methods` 等配置项重名。

  

### `watch`（侦听器）

- **功能**: 监听现有状态的改变并执行自定义逻辑。

- 特点

  - 用于执行副作用操作
    + 在vue中，修改状态驱动界面更新是主线，除此之外的都是支线，也就是副作用操作
    + 例如发送网络请求、操作 `localStorage` 等。
  - 不创建新的状态，只是对现有状态的变化进行响应。

  

### 共同点

- 两者都通过监听状态值的改变来进行处理，都是基于 `Watcher` 类实现的。



### 使用场景

- **`computed`**: 适用于需要根据状态派生新值的场景，尤其是当需要缓存计算结果时。
- **`watch`**: 适用于需要在状态改变时执行副作用的场景。