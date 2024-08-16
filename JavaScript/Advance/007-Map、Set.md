## weakRef

用于创建弱引用变量

```js
const user = { name: 'Klaus' }

const weak = new WeakRef(user) // => 此时weak对于user对象的引用就是弱引用


// 我们虽然可以通过弱引用去查找到对应的对象，但是我们不能保证对应的对象一定存在
// 所以在使用弱引用之前，需要先使用deref方法进行判断
// 如果对应的对象依旧存在，则可以继续使用
// 如果对于的对象已经被GC所销毁的时候，该方法就会返回undefined，告诉我们对应的对象已经不可使用了
if (weak.deref()) {
  // 注意这里需要使用weak.deref().name
  // 不可以使用let ref = weak.deref()
  // 因此此时ref是一个强引用，需要手动进行ref = null操作
  
  // 其实，如果打印这个weak.deref(), user是不会被销毁的
  // 因为此时控制台中打印的是一个可交互的对象，也就是其对于user对象也是一个强引用
  // 但是如果打印的是属性，那么就不是对user的强引用，user会被移除
  console.log(weak.deref().name) // => Klaus
}
```

