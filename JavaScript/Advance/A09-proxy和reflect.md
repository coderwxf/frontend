## proxy

拦截对对象的操作，并注入自定义逻辑，这种操作被称之为数据劫持



早期数据劫持方式使用`Object.defineProperty`

```js
const user = {
  name: 'Klaus',
  age: 23
}

Object.keys(user).forEach(key => {
  let value = user[key]

  Object.defineProperty(user, key, {
    set(newV) {
      console.log(`属性：${key}设置了，值为 ${value}`)
      // 这里形成了一个闭包
      value = newV
    },

    get () {
      console.log(`获取属性${key}`)
      return value
    }
  })
})

user.name = 'Alex'
console.log(user.name)

user.age = 18
console.log(user.age)
```



`Object.defineProperty` 主要用于在对象上定义新属性或修改现有属性，允许对属性的行为进行精细化控制。

虽然它可以通过定义 getter 和 setter 来拦截对属性的访问和修改（实现数据劫持），但这并不是其设计的主要目的。

使用 `Object.defineProperty` 进行数据劫持时，需要逐个属性进行处理，因为它只能对单个属性设置 getter 和 setter。

同时，`Object.defineProperty` 无法拦截诸如新增属性或删除属性等操作，这些操作超出了其能力范围。

因此，尽管 `Object.defineProperty` 可以实现一些数据劫持功能，但它在监控和控制对象属性方面存在一定的局限性。



在 ES6 中引入了 `Proxy` 类，用于创建代理对象，通过代理对象可以拦截和自定义对另一个对象的各种操作。

要监听某个对象的操作，可以创建一个 `Proxy` 对象，之后所有对目标对象的操作都通过代理对象进行处理，就可以实现对操作的拦截和定制化处理。

```js
const proxy = new Proxy(target, handler)
```

1. handler的类型是`Record<string, Function>`
2. 后续所有操作都操作代理对象「 代理对象再去操作源对象 」
3. 如果对应的操作 没有设置监听函数 那么就会使用默认行为 进行处理
4. 如果设置了对应的操作，那么就会覆盖默认行为，直接执行对应的侦听器函数(捕获器函数)

```js
const user = {
  name: 'Klaus',
  age: 23
}

const userProxy = new Proxy(user, {})

console.log(userProxy.name) // => Klaus
userProxy.name = 'Alex'
console.log(userProxy.name) // => Alex
```



![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/417521e6196a4fa4a0ae432aaff585f2~tplv-k3u1fbpfcp-zoom-1.image) 

```js
const user = {
  name: 'Klaus',
  age: 23
}

// handler中所有劫持方法:
// + 第一个参数都是target「 源对象 」
// + 其余参数均为正常操作需要使用的参数 「 可以查看对应handler的JSDOC 」
// + getter和setter劫持，最后还存在一个额外的参数receiver 「 代码对象 」
const userProxy = new Proxy(user, {
  set(target, key, value, receiver) {
    console.log(`属性${key}正在执行赋值操作`)

    // 通过target，而不是user，从而减少代理对象和源对象的耦合度
    target[key] = value
    
    // 返回布尔值表示新值是否设置成功
    return true
  },

  get(target, key, receiver) {
    console.log(`属性${key}正在被读取`)
    return target[key]
  },
  
  has(target, key) {
    console.log(`${key} 正在执行in判断`)
    return key in target
  },

  deleteProperty(target, key) {
    console.log(`${key} 正在执行delete操作`)
    return delete target[key]
  },
  
  construct(target, args) {
    console.log(`正在新建${target.name}的实例对象`)
    return new target(...args)
  },

  apply(target, thisArg, args) {
    console.log(`${target.name}正在执行apply方法`)
    return target.apply(thisArg, args)
  }
})

// 后续所有操作都通过代理对象，而不是源对象
// 1. 代理对象对行为进行了拦截，注入了额外的逻辑
// 2. 代理对象最终还是去操作源对象
// 直接操作源对象，将无法执行额外添加的劫持逻辑
console.log(userProxy.name) // => Klaus
userProxy.name = 'Alex'
console.log(userProxy.name) // => Alex
```



### 关注点分离

通过将不同的功能或责任分离到不同的模块或组件中，以提高系统的可维护性、可扩展性和可理解性

1. 模块化 --- 将不同的功能（如用户界面、业务逻辑、数据访问等）分离到不同的模块中。每个模块专注于完成某一特定任务，而不是混合多个功能
2. 低耦合 --- 通过减少模块之间的依赖关系，使得系统的各个部分可以独立修改或重用，而不会影响到其他部分
3. 单一职责原则 --- 
4. 每个模块或类应该只负责一种功能或任务。这种做法简化了代码的理解和维护



## Reflect

`Reflect`和`Math`一样是全局对象，直接使用，不需要通过new调用

`Reflect`提供了一系列`操作对象自身`的`静态方法`



随着JavaScript的发展，`Object`的职责越来越复杂

1. 所有类的父类，`Object.prototype`是原型链的顶端
2. `Object`可以生成对象，最基础的引用类型数据
3. 挂载了一系列操作对象的静态方法，如`Object.keys` 、`Object.defineProperty`等
4. 挂载了一系列用于元编程的静态方法，如`Object.preventExtensions`、`Object.seal`、`Object.freeze`等

这导致Object对象越来越庞大，Reflect对象的出现是为了将Object上那些元编程操作职能抽离出来

例如`Reflect.getPrototypeOf(target)`和`Object.getPrototypeOf(target)`具有相同的功能

Reflect在抽取的时候，同时对Object上的对应方法进行了相应的优化操作

1. 静默错误将不再静默，而是显示返回布尔值
2. 之前的一些操作符操作，如in、delete。都统一成了方法



![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5409b96e0936451a9c872e8a754b0481~tplv-k3u1fbpfcp-zoom-1.image) 

> Reflect可以进行的操作 和 Proxy可以进行的操作 是一一对应的

```js
const user = {
  name: 'Klaus',
  age: 23
}

Object.defineProperty(user, 'name', {
  configurable: false
})

// 早期的做法
// 1. delete是操作符, 不是方法调用
// 2. 对于无法删除的属性, 非严格模式下会静默失效, 严格模式下则直接报错
// delete user.name

// Reflect.deleteProperty就是用于替换delete操作符
// 1. 变成方法，和其它方法（如defineProperty）一样 更为的统一规范
// 2. 该方法通过返回布尔值来判断对应属性是否成功删除，而不会直接在严格模式抛出错误
if (Reflect.deleteProperty(user, 'name')) {
  console.log('删除成功')
} else {
  console.log('删除失败')
}
```





### 示例

#### 如何让`a===1 && a===2 && a===3`和`a==1 && a==2 && a==3`的判断结果为true

判等可能会触发`隐式类型转换`，所以可以使用 `valueOf` 来实现

```js
class A {
  constructor(value) {
    this.value = value;
  }

  valueOf() {
    return this.value ++
  }
}

const a = new A(1)
console.log(a == 1 && a == 2 && a == 3) // => true
```



而全等并不会进行类型转换，只能通过`defineProperty`或`Proxy`来进行监听

```js
let target = {
  a: 0
}

const proxy = new Proxy(target, {
  get(target, key, receiver) {
    if (key === 'a') {
      target[key]++
      return Reflect.get(target, key, receiver)
    }
  }
})

console.log(proxy.a===1 && proxy.a===2 && proxy.a===3) // => true
```







----

元编程