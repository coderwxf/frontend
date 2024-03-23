## VDOM -> DOM

react通过`render`方法将VDOM转换为DOM

`react16/17`

```jsx
ReactDOM.render(<></>, document.getElementById('root'))
```

`react18`

```jsx
const root = ReactDOM.createRoot(document.getElementById('root'))

root.render(<></>)
```



### 循环问题

1. 一般来讲，内置属性都是不可枚举的 例如 `prototype`、`length` --- 在控制台输出是淡紫色

   一般来讲，自定义属性都是可枚举的 --- 在控制台输出是深紫色

   可以通过`Object.defineProperty`方法来修改属性描述符

   

2. 常见的迭代方法

| 迭代方式                                           | 迭代可枚举 | 迭代原型 | 迭代symbol | 迭代不可枚举 |
| -------------------------------------------------- | ---------- | -------- | ---------- | ------------ |
| for - in                                           | true       | true     | false      | false        |
| Object.keys(val)                                   | true       | false    | false      | false        |
| Object.getOwnPropertyNames(val)                    | true       | false    | false      | true         |
| Object.keys(Object.getOwnPropertyDescriptors(val)) | true       | false    | false      | true         |
| Reflect.ownKeys(val)                               | true       | false    | true       | true         |
| Object.getOwnPropertySymbols(val)                  | false      | false    | true       | false        |

+ 对于即能迭代自身属性，也能迭代原型上属性的 迭代方法 性能相对较差

  因为要去迭代原型链，即便原型链上没有任何可枚举属性，也要去确认一遍

+ `Reflect.ownKeys = Object.getOwnPropertyNames + Object.getOwnPropertySymbols`



### 设置属性

#### 直接通过属性设置

`直接设置`

```js
const el = document.getElementById('root')

el.name = 'root el'
console.log(el.name) // 'root el'
```

![image-20240315180645689](https://s2.loli.net/2024/03/15/UPXwg5taQjfpnG9.png) 

直接设置 是设置到位于堆内存的DOM对象上，并不会直接体现到实际的DOM元素上



`通过内置属性设置`

```js
const el = document.getElementById('root')

el.style.color = 'red'
console.log(el.style.color) // 'red'
```

![image-20240315180908012](https://s2.loli.net/2024/03/15/xYXqo7Zyp3FaVi5.png) 

通过内置属性操作，可以同时设置到 内存中的DOM元素上，也可以体现到实际的DOM元素上



对于直接通过属性设置属性，其CRUD操作和普通属性一致

```js
// 添加
el.name = 'title'

// 修改
el.name = 'root'

// 获取
console.log(el.name)

// 删除
delete el.name
```



#### setAttribute

```js
const el = document.getElementById('root')

el.setAttribute('key', 'root')
console.log(el.key) // undefined
console.log(el.getAttribute('key')) // root
```

![image-20240315181457967](https://s2.loli.net/2024/03/15/fdshkbRgwNCQGXl.png) 

`setAttribute` 设置属性 会存在真实DOM中，不存在于内存DOM中

`setAttribute`设置的属性，只能通过`getAttribute`来获取, 只能通过`removeAttribute`来移除



#### 两者比较

```js
const el = document.getElementById('root')

el.name = 'user'
el.style.color = 'red'
el.setAttribute('key', 'root')

console.log(el.getAttribute('name')) // => null
console.log(el.getAttribute('key')) // => root
console.log(el.getAttribute('style')) // => color: red;

console.log(el.name) // => user
console.log(el.style) // 一个CSS样式对象
console.log(el.key) // => undefined
```

1. 如果没有获取到对应属性值
   + `getAttribute` --- 返回 `null`
   + 直接通过属性 --- 返回 `undefined`

2. `getAttribute` 获取的是那些 真实DOM 上存在的属性

   直接通过属性 获取的是那些 内存DOM上 存在的属性



### push vs concat

```js
let arr1 = [1, 2, 3]
let arr2 = [4, 5, 6]

// push 不是一个纯函数
console.log(arr1.push(arr2)) // => 4
console.log(arr1) // => [1, 2, 3, [4, 5, 6]] 

arr1 = [1, 2, 3]
arr2 = [4, 5, 6]

// concat 是纯函数
console.log(arr1.concat(arr2)) // => [1, 2, 3, 4, 5, 6]
```



### substr vs subString vs slice

| 方法                                                         | 说明                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| ''.substring(startIndex, endIndex)                           | 1. 包头不包尾<br />2. 传入负值，负值视为0<br />3. starIndex < endIndex, 交换两者位置后，在执行字符串截取<br />4. endIndex 不传，默认表示截取到 字符串最后边 |
| ''.substr(startIndex, length)                                | 1. startIndex 支持负值<br />2. length为负值，直接返回 空字符串<br />3. length 不传，默认表示截取到 字符串最后边<br />4. 不是MDN规范，不推荐使用 |
| ''.slice(beginIndex [, endIndex])<br />或<br />[].slice(beginIndex [, endIndex]) | 1. 包头不包尾<br />2. beginIndex 和 endIndex 支持负值<br />3. endIndex 不传或大于beginIndex，则表示截取到字符串结尾 |



### VDOM -> DOM 具体实现

```js
// 模拟实现React.render方法
// + 仅处理元素标签形式，不处理组件
export function render(vDOM, container) {
  const el = document.createElement(vDOM.type)

  Reflect.ownKeys(vDOM.props).forEach(key => {
    // 事件处理
    if (key.startsWith('on')) {
      // 判断事件是否是原生事件
      if (key.toLowerCase() in window) {
        // 事件绑定
        el.addEventListener(key.toLowerCase().substring(2), vDOM.props[key])
      }
      return
    }

    // 特殊key处理
    if (key === 'className') {
      // 原生DOM 通过className设置class
      el.className = vDOM.props[key]
      return
    }

    if (key === 'htmlFor') {
      el.setAttribute('for', vDOM.props[key])
      return
    }

    if (key === 'style') {
      // 对于font-size等带有-的属性，设置的时候使用的是fontSize
      // 所以React在设置样式的时候，要求使用小驼峰命名法
      Reflect.ownKeys(vDOM.props[key]).forEach((styleKey) => {
        el.style[styleKey] = vDOM.props[key][styleKey]
      })
      return
    }

    // 插入子节点
    if (key === 'children') {
      let children = vDOM.props.children

      // 统一转换为数组
      if (!(vDOM.props.children instanceof Array)) {
        children = [vDOM.props.children]
      }

      children.forEach(child => {
        // 子节点可能是一个字符串，也可能是一个vDOM
        if (typeof child === 'string') {
          // el.appendChild(document.createTextNode(child)) 等价于 el.textContent += child
          el.textContent += child
        } else {
          render(child, el)
        }
      })
      return
    }

    // 常规属性设置
    el.setAttribute(key, vDOM.props[key])
  })

  // 插入到容器
  container.appendChild(el)
}
```

`注意点`

```js
// el.appendChild(document.createTextNode(child)) 等价于 el.textContent += child
el.textContent += child
```

使用`+=`的原因是，可能出现以下JSX结构

```jsx
<div>
	{'Hello'}
  {'World'}
</div>
```

这种情况下，存在两个文本节点，所以需要使用`+=`

以避免 `后续设置的文本内容` 覆盖 `之前设置的文本内容`

