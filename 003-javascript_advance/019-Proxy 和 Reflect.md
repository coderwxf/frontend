## proxy

如果我们有一个需求，想要监听在一个对象上，对于其属性的各种操作，也就是在对对象属性进行操作的时候，自动执行一系列的预设操作

例如: 当我们修改对象的属性的时候，自动去更新界面上所有依赖（使用）该属性的dom，这种行为被称之为`响应式`

为了完成这个需求，我们第一想到的是`Object.defineProperty`方法

```js
const user = {
  name: 'Klaus',
  age: 23
}

// Object.keys方法的返回值是数组，所以可以直接使用forEach或for-of进行迭代
Object.keys(user).forEach(key => {
  let value = user[key]

  Object.defineProperty(user, key, {
    set(newV) {
      console.log(`属性：${key}设置了，值为 ${value}`)
      // 这里本质上就是一个闭包
      value = newV
    },

    get () {
      console.log(`获取属性${key}`)
      return value
    }
  })
})

// 这里设置和获取值，实际上是去获取和设置
// forEach构成的闭包中的那个value属性值
user.name = 'Alex'
console.log(user.name)

user.age = 18
console.log(user.age)
```



但是这样存在一个弊端，Object.defineProperty设计的初衷是为了添加属性描述符的，并不是用来监听一个完整的对象

所以如果我们想监听更加丰富的操作，比如新增属性、删除属性，in操作符，那么 Object.defineProperty是无能为力的



为此，在ES6中，新增了一个Proxy类，这个类从名字就可以看出来，是用于帮助我们创建一个代理对象

也就是说，如果我们希望监听一个对象的相关操作，那么我们可以先创建一个代理对象(Proxy对象)

之后对该对象的所有操作，都通过代理对象来完成，代理对象可以监听我们想要对原对象进行哪些操作



`使用方式:`

+ 首先，我们需要new Proxy对象，并且传入需要侦听的对象以及一个处理对象，可以称之为handler

  ```js
  const p = new Proxy(target, handler)
  ```

+ 其次，我们之后的操作都是直接对Proxy的操作，而不是原有的对象，因为我们需要在handler里面进行侦听

  + 我们所有的操作 都通过代理对象进行操作
  + 代理对象监听我们对对象的操作，进行二次处理和加工后，再作用于原对象



```js
const user = {
  name: 'Klaus',
  age: 23
}

// 第一个参数为 原始对象 也就是我们需要代理的那个对象
// 第二个参数为 侦听器对象
// 如果对应的操作 没有设置监听函数 那么就会使用默认行为 进行处理
// 如果设置了对应的操作，那么就会覆盖默认行为，直接执行对应的侦听器函数(捕获器函数)
// 即自己设置的捕获器函数是对原本行为的重写
const userProxy = new Proxy(user, {})

console.log(userProxy.name) // => Klaus
userProxy.name = 'Alex'
console.log(userProxy.name) // => Alex
```

```js
const user = {
  name: 'Klaus',
  age: 23
}

const userProxy = new Proxy(user, {
  // set方法有四个参数
  // 参数1 - 代理的原始对象
  // 参数2 - 属性名
  // 参数3 - 属性值
  // 参数4 - 代理对象本身 receiver === userProxy -> true
  set(target, key, value, receiver) {
    console.log(`属性${key}正在执行赋值操作`)
    // 操作源对象的时候需要使用target
    // 因为很有可能user和userProxy并不在同一个文件中
    target[key] = value
  },

  // get方法有三个参数
  // 参数1 - 代理的原始对象
  // 参数2 - 属性名
  // 参数3 - 代理对象本身
  get(target, key, receiver) {
    console.log(`属性${key}正在被读取`)
    return target[key]
  }
})

// 使用代理对象后，所有操作应该都通过代理对象进行操作，而不应该操作原始对象
console.log(userProxy.name) // => Klaus
userProxy.name = 'Alex'
console.log(userProxy.name) // => Alex
```



同时，proxy本身就是用来进行对象代理的，所以proxy相比defineProperty而言，可以监听的行为更多更全

![image.png](https://s2.loli.net/2022/06/29/dT86aVpRvlCMDoP.png) 

```js
const user = {
  name: 'Klaus',
  age: 23
}

const proxy = new Proxy(user, {
  has(target, key) {
    console.log(`${key} 正在执行in判断`)
    return key in target
  },

  deleteProperty(target, key) {
    console.log(`${key} 正在执行delete操作`)
    return delete target[key]
  }
})

console.log('name' in proxy)
delete proxy.age
```

```js
function Foo(num1, num2) {
  console.log(this, num1, num2)
}

const proxy = new Proxy(Foo, {
  construct(target, args) {
    console.log(`正在新建${target.name}的实例对象`)
    return new target(...args)
  },

  apply(target, thisArg, args) {
    console.log(`${target.name}正在执行apply方法`)
    return target.apply(thisArg, args)
  }
})

proxy.apply({ name: 'Klaus' }, [23, 32])
const foo = new proxy(10, 20)
```



## reflect

[Reflect](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Reflect/Comparing_Reflect_and_Object_methods)也是ES6新增的一个API，它是一个`对象`，字面的意思是反射

也就意味着Reflect是一个和Math，Date类似的对象，不需要通过new关键字直接可以使用

Reflect 是 JavaScript 中的一个内置对象，它提供了一组静态类方法，用于操作对象自身

它与全局对象 Object 有着相似的功能，比如`Reflect.getPrototypeOf(target)`类似于` Object.getPrototypeOf()`

但是它更加严格、更具语义性



Reflect出现的原因是因为在早期的ECMA规范中没有考虑到这种对 **对象本身** 的操作如何设计会更加规范，所以将这些API放到了Object上面

而随着时间的推移，在Object上定义的那些操作对象本身的方法，也就是那些类方法越来越多，就导致Object对象变得越来越庞大

另外还包含一些类似于 in、delete操作符，让JS看起来是会有一些奇怪

所以在ES6中新增了Reflect，让我们这些操作都集中到了Reflect对象上



Reflect 反映了对象自身的一些操作和修改，包括但不限于设置和读取对象属性、调用对象方法、创建对象实例等

Reflect 提供了一组 API，可以帮助我们更灵活地控制和管理对象的行为，而不必依赖于固定的语法

简而言之，Reflect是指对象上的一些操作，修改，调用对象自身的方法

![image.png](https://s2.loli.net/2022/06/30/Pu5StascXoMhCK9.png) 



```js
const user = {
  name: 'Klaus',
  age: 23
}

Object.defineProperty(user, 'name', {
  configurable: false
})

// 早期的做法
// 1. 单从这句delete语句中而言，我们不能知道name属性是否被正常删除
// 2. 对于对象的操作有些是方法，而有些是操作符，不够统一
// 3. 对于对象的操作, 非严格模式下 可能是静默错误，而在严格模式下，可能直接报错
// delete user.name

// 为此，ES6将这里对于对象自身进行操作的(主要是那些类方法)，单独抽取到了Reflect对象上
// Reflect.deleteProperty就是用于替换delete操作符
// 1. 使属性删除操作变成方法，和其它方法（如defineProperty）一样 更为的统一规范
// 2. 该方法会返回boolean类型值，表示是否删除成功
//    我们可以直接进行判断，而不会出现静默错误，或单独只在严格模式下报错的情况
if (Reflect.deleteProperty(user, 'name')) {
  console.log('删除成功')
} else {
  console.log('删除失败')
}
```



### 基本使用

```js
const user = {
  name: 'Klaus',
  age: 23
}

const proxy = new Proxy(user, {
  set(target, key, value) {
    console.log('setter')
    // target === user -> true
    // 所以使用下述方法 本质依旧是直接在操作源对象
    // target[key] = value
    
    // 但是使用Proxy的目的就是为了不直接操作源对象
    // 所以为了使原始对象和代理对象进行分离
    // 同时 为了 更好的符合规范, 可以使用Reflect来替换上述方法的操作
    // 使用Reflect来在语言层面去操作源对象，而不是直接使用JS API在语言层面去操作原对象（优点一）
    // 同时Reflect上的API相比较原本的那些操作对象的API而言，更为的规范和安全(优点二)
    // 所以Reflect 经常 和 Proxy 一起结合使用
    Reflect.set(target, key, value)
  },

  get(target, key) {
    console.log('getter')
    // return target[key]
    return Reflect.get(target, key)
  }
})

proxy.name = 'Alex'
console.log(proxy.name)
```



### receiver

如果我们的源对象(obj)有setter、getter的访问器属性，那么可以通过receiver来改变里面的this

```js
const user = {
  _name: 'Klaus',
  get name() {
    // 默认情况下，代理操作原始对象的时候
    // 访问器中的get和set方法中的this是原始对象
    // 此时对于_name的设置 无法被监听到
    console.log(this === user)  // => true
    return this._name
  },
  set name(v) {
    console.log(this === user)  // => true
    this._name = v
  }
}

const proxy = new Proxy(user,  {
  get(target, key) {
    console.log('getter')
    // 此时操作target对象的this就是源对象，并不是代理对象
    return Reflect.get(target, key)
  },
  set(target, key, value) {
    console.log('setters')
    Reflect.set(target, key, value)
  }
})

proxy.name = 'Alex'
console.log(proxy.name)
```

```js
const user = {
  _name: 'Klaus',
  get name() {
    return this._name
  },
  set name(v) {
    this._name = v
  }
}

const proxy = new Proxy(user,  {
  // receiver参数的作用是用于修改访问器中的this
  // 以便于操作私有属性(在本例中就是_name)的操作，也可以被监听到
  get(target, key, rceiver) {
    // receiver就是当前proxy对象本身
    console.log(reveiver === proxy) // => true
    console.log('getter')
    return Reflect.get(target, key, receiver)
  },
  set(target, key, value, receiver) {
    console.log('setters')
    Reflect.set(target, key, value, receiver)
  }
})

proxy.name = 'Alex'
console.log(proxy.name)
```



### Reflect.construct

```shell
# 下述代码在功能上等价于
# new target(...argumentsList)
Reflect.construct(target, argumentsList)

# 下述代码在功能上等价于 
# new target.apply(newTarget, argumentsList)
Reflect.construct(target, argumentsList, newTarget)
```

```js
function Person(name, age) {
  this.name = name
  this.age = age
}

// 借用构造函数继承属性
function Student(name, age) {
 // 如果存在 Reflect.construct
 if (Reflect && Reflect.construct) {
   // Reflect.construct(需要调用构造函数, 参数列表构成的数组，实际需要的类型对应的构造函数)
   return Reflect.construct(Person, [name, age], Student)
  // 上述代码的含义是 调用Person的构造函数，传入name和age，但是创建出的实例类型是Student的实例
  // 等价于 Person.apply(this, [name, age])
 } else {
  // Person.apply(Student, [name, age])
  return Person.apply(this, [name, age])
 }
}

const stu = new Student('Klaus', 23)
console.log(stu)
```

