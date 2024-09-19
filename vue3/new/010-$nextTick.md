`基本使用`

`this.$nextTick(callback, thisArg)`

+ callback 即使不是箭头函数，callback内部this也是vue实例
+ 存在第二个参数 可以修正callback中this的值



当在界面中修改状态时:  状态更新是同步的，DOM更新是异步的，以实现批处理

这和react是不一样的，react中状态和界面的更新都是异步的



1. 同步更新状态

```js
methods: {
  increment() {
    // 假设count的初始值是0
    this.count++

    // 状态的更改是同步的
    console.log(this.count) // => 1
  }
}
```

2. 将更新信息存入 更新队列中

3. 同步代码执行完毕，再依次执行更新队列中的数据

   「 从而保证，短期内无论界面中状态更新了几个，界面永远只会更新一次 => 批处理 」



```js
methods: {
  increment() {
    this.count++
    console.log(this.count)

    // 强制更新 => 本质其也会把界面更新操作存入更新队列中
    this.$forceUpdate()

    this.count++
    console.log(this.count)
  }
}
```

`this.$forceUpdate()`相当于主动告诉vue去更新视图

其通知视图更新的操作本质也是加入到了事件队列中

所以在上边的例子中，界面只会更新一次，并不会更新两次



```js
methods: {
  increment() {
    this.count++
    console.log(this.count)

    // 同步代码会执行完毕后，再进行异步更新
    // 所以一定要实现更新两次，可以将第二次更新内容放到异步操作中即可 「 例如 queueMicroTask、定时器等 」
    setTimeout(() => {
      // 此时界面会渲染两次
      this.count++
      console.log(this.count)
    })
  }
}
```



`nextTick`中的回调会被插入到`callbacks`队列中

+ 会在`updated`方法执行完毕后回调
+ 可以实现更新那个状态，执行特定逻辑
  + `updated`是任何状态更新后都需要执行的逻辑
  + `nextTick`是某个状态更新后需要执行的特定逻辑




`nextTick`执行流程

+ 执行同步代码

+ 同步代码执行完毕后，执行视图更新队列中的任务

+ 执行 updated 钩子

+ 执行callbacks队列中的方法执行



---

发布-订阅模式 「 不是观察者模式 」

