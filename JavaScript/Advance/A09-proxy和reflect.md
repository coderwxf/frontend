## Proxy

需求: 对对象的各种操作进行拦截，并添加自定义处理逻辑 「 本质是名为代理模式的设计模式 」

最为常见的使用场景是实现**响应式**，即在修改值的同时，去更新所有使用到该值的UI界面

实现方法 :

1. `Object.defineProperty`
2. `Proxy`

```js
const user = {
  name: 'Klaus',
  age: 23
}

Object.keys(user).forEach(key => {
  let _value = user[key]

  Object.defineProperty(user, key, {
    // 通过闭包实现对属性值的拦截
    set(newV) {
      console.log(`属性：${key}设置了，值为 ${value}`)
      _value = newV
    },

    get () {
      console.log(`获取属性${key}`)
      return _value
    }
  })
})

user.name = 'Alex'
console.log(user.name)

user.age = 18
console.log(user.age)
```

`Object.defineProperty()`设计初衷是为了更粒度化的控制属性值, `getter/setter`并不是其主要功能

所以只能监听属性值的设置和修改操作，而其余操作 「 如新增属性，删除属性，in操作等 」都是无法监听的



`Proxy`类，生成一个代理对象，专门用于对源对象进行代理 「 需要操作代理对象，而不是源对象 」

```js
const p = new Proxy(target, handler)
```

`handler`是侦听器对象，如果侦听器对象中有我们对某个方法的侦听处理逻辑，则执行自定义逻辑，否则则执行默认处理逻辑

```js
const user = {
  name: 'Klaus',
  age: 23
}

// 没有侦听逻辑，走默认处理逻辑
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
  // 参数1 - 源对象
  // 参数2 - 属性名
  // 参数3 - 属性值
  // 参数4 - 代理对象 「 receiver === userProxy -> true 」
  set(target, key, value, receiver) {
    console.log(`属性${key}正在执行赋值操作`)
    // 操作源对象的时候需要使用target
    // 因为很有可能user和userProxy并不在同一个文件中
    target[key] = value
    // 返回布尔值表示新值是否设置成功
    return true
  },

  // get方法有三个参数
  // 参数1 - 源对象
  // 参数2 - 属性名
  // 参数3 - 代理对象
  get(target, key, receiver) {
    console.log(`属性${key}正在被读取`)
    // 要使用target，而不是user 「 代理对象处理逻辑和源对象解耦 」
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

// 使用代理对象后，所有操作应该都通过代理对象进行操作，而不应该操作原始对象 「 否则无法执行自定义逻辑 」
console.log(userProxy.name) // => Klaus
userProxy.name = 'Alex'
console.log(userProxy.name) // => Alex
```

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/417521e6196a4fa4a0ae432aaff585f2~tplv-k3u1fbpfcp-zoom-1.image#?w=2013\&h=1070\&s=280328\&e=png\&b=ffffff) 



## Reflect

Reflect是一个和Math类似的全局对象，提供了一组静态方法，用于操作自身 「 被称之为反射 」

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5409b96e0936451a9c872e8a754b0481~tplv-k3u1fbpfcp-zoom-1.image#?w=2358\&h=1080\&s=429542\&e=png\&b=fefefe)

随着JavaScript发展，Object职能越来越多

1. 顶层对象
2. 用于创建对象字幕量
3. 提供了一系列静态方法可以在运行时查看或修改对象自身
   + 但设计不规范，有些是方法，有些是操作符
   + 有些在非严格模式下不报错，但是到了严格模式反而报错了

Reflect将Object中那些操作自身的静态方法进行抽离，并进行了优化

```js
const user = {
  name: 'Klaus',
  age: 23
}

Object.defineProperty(user, 'name', {
  configurable: false
})

// Reflect.deleteProperty用于替换delete操作符
// 1. 使属性删除操作变成方法，和其它方法（如defineProperty）一样 更为的统一规范
// 2. 该方法会返回boolean类型值，表示是否删除成功
//    我们可以直接进行判断，而不会出现静默错误，或单独只在严格模式下报错的情况
if (Reflect.deleteProperty(user, 'name')) {
  console.log('删除成功')
} else {
  console.log('删除失败')
}
```



```js
const user = {
  name: 'Klaus',
  age: 23
}

const proxy = new Proxy(user, {
  set(target, key, value) {
    console.log('setter')
    // 1. Reflect操作源对象可以更好的实现代理对象和源对象的分离
    // 2. Reflect操作源对象自身更规范和统一
    // 所以 Proxy一般会和Reflect 一起结合使用
    Reflect.set(target, key, value)
    return true
  },

  get(target, key) {
    console.log('getter')
    return Reflect.get(target, key)
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



## receiver

```js
const user = {
  _name: 'Klaus',
  get name() {
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
    // 如果target中key是getter/setter方法时
    // getter/setter方法中的this值是源对象不是代理对象
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
  // receiver的作用就是修改getter/setter中的this指向
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

## 