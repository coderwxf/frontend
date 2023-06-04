## 模拟apply方法实现

```js
// 挂载到Function.prototype上，以便于所有的函数实例对象都可以访问该方法
Function.prototype.myApply = function(thisArg, args) {
  // this === 调用函数
  // thisArg === 实际绑定的this

  // Object方法
  // 如果参数是非引用类型 返回对应的包装类类型（如果参数没有对应的包装类类型，直接返回空对象）
  // 如果参数是引用类型 直接返回对应的引用类型
  
  // 如果传入的是null或undefined直接使用globalThis，否则尽可能将this转换为对象类型
  const thisArgument = [null, undefined].includes(thisArg) ? globalThis : Object(thisArg)

  // 通过隐式绑定的方式 修正this指向
  Object.defineProperty(thisArgument, '__fn__', {
    configurable: true,
    value: this
  })

  thisArgument.__fn__(...args)
  delete thisArgument.__fn__
}

// Test Code
function foo(name, age) {
  console.log(this, name, age)
}

const thisArg  = {
  name: 'Klaus'
}

foo.myApply(thisArg, ['Klaus', 23])
```



## 模拟call方法实现

```js
// call方法的实现和apply方法的实现是基本一致的
// 唯一的区别在于传入的参数类型不同
Function.prototype.myCall = function(thisArg, ...args) {
  const thisArgument = [null, undefined].includes(thisArg) ? globalThis : Object(thisArg)

  Object.defineProperty(thisArgument, '__fn__', {
    configurable: true,
    value: this
  })

  thisArgument.__fn__(...args)
  delete thisArgument.__fn__
}

// Test Code
function foo(name, age) {
  console.log(this, name, age)
}

const thisArg  = {
  name: 'Klaus'
}

foo.myCall(thisArg, 'Klaus', 23)
```



我们可以看到call和apply方法的时候中，除了传入的参数类型不同之外，其余的代码实现逻辑都是一致的

所以我们可以将call方法和apply方法的代码实现进行抽取

```js
// 将_execFn绑定到Function.prototype上的目的是为了，提高_execFn和myApply及myCall方法之间的内聚度
Function.prototype._execFn = function(thisArg, args) {
  const thisArgument = [null, undefined].includes(thisArg) ? globalThis : Object(thisArg)

  Object.defineProperty(thisArgument, '__fn__', {
    configurable: true,
    value: this
  })

  thisArgument.__fn__(...args)
  delete thisArgument.__fn__
}

Function.prototype.myApply = function(thisArg, args) {
  this._execFn(thisArg, args)
}

Function.prototype.myCall = function(thisArg, ...args) {
  this._execFn(thisArg, args)
}

// Test Code
function foo(name, age) {
  console.log(this, name, age)
}

const thisArg  = {
  name: 'Klaus'
}

foo.myApply(thisArg, ['Klaus', 23])
foo.myCall(thisArg, 'Klaus', 23)
```



## 模拟bind方法实现

```js
Function.prototype.myBind = function(thisArg, ...args) {
  // 使用箭头函数，以便于返回的函数内部可以使用正确的this值
  // 此处使用了函数柯里化
  return (...arguments) => {
    const _this = [null, undefined].includes(thisArg) ? globalThis : Object(thisArg)

    Object.defineProperty(_this, 'fn', {
      configurable: true,
      value: this
    })

    _this.fn(...args, ...arguments)

    // bind函数中不需要delete掉fn函数
    // 因为bind返回的新方法可能会被多次进行调用
    // delete _this.fn
  }
}

// Test Code
function fun(name, age) {
  console.log(this, name, age)
}

const foo = fun.bind({name: 'Klaus'}, 'Klaus')
foo(23)
```

