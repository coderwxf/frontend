插槽 是一种 特殊的props

如果父组件向子组件传递

1. 传递 数据值 使用 props
2. 传递 html结构 使用 插槽



所以插槽的作用和props 是 一致的 `都是为了增加 组件的复用性 或 可扩展性`

> 复用性 就是 同一个东西 可以 同时满足 多种情况下的使用需求



Vue默认已经实现了对应的插槽机制

但是React中默认并没有实现对应的插槽机制，需要我们自行实现



## 基本使用

`调用`

```jsx
<App>
	<span>child1</span>
	<span>child2</span>
</App>

<App>
	<span>child1</span>
</App>

<App />
```

`定义`

```jsx
export default function App(props) {
  return (
   <div>
    {/*
    	1. 子节点 转换为对应的 VDOM
    	2. 转换后的VDOM 会作为 props.children 传递进来
    */}
    {props.children}
   </div>
  )
}
```



## 多个插槽

`调用`

```jsx
<>
 <App>
  <div>header</div>
  <div>footer</div>
 </App>

 <App>
  <div>header</div>
 </App>

 <App />
</>
```



`定义`

```jsx
import React from 'react'

export default function App(props) {
  // 注意 这里的 Children 首字母是大写的
  const children = React.Children.toArray(props.children)

  return (
   <>
    {/* 第一个JSX元素作为header */}
    <header>{children[0]}</header>
    <div>content</div>
 		{/* 第二个JSX元素作为footer */}
    <footer>{children[1]}</footer>
   </>
  )
}
```



### React.children

在props.children 存在三种情况

1. undefined
2. JSX元素
3. 数组

由于 `props.children` 可以是不同类型，React 提供了 `React.Children` 工具函数集

这个工具集包含了若干方法，允许以统一的方式操作 `children`，无论它们的数据类型是什么

```jsx
import React from 'react'

export default function App(props) {
  // 获取 props.children 的长度
  console.log(React.Children.count(props.children))

  // 对props.children 执行遍历
  React.Children.forEach(props.children, (child, index) => {
    console.log(child, index)
  })

  // 对props.children执行map操作
  React.Children.map(props.children, (child, index) => {
    console.log(child, index)
    return child
  })

  // 将props.children 转换为 ReactElement
  // 1. 如果props.children 是数组或undefined 转换失败 抛出错误
  // 2. 如果props.children 是单一JSX元素 转换成功 直接返回该JSX元素
  // console.log(React.Children.only(props.children))

  // 将props.children 转换为数组
  // 推荐在使用props.children的时候，都将其先转换为数组后再使用, 以避免在使用的时候出现类型错误
  const children = React.Children.toArray(props.children)

  return (
   <>
    <header>{children[0]}</header>
    <div>content</div>
    <footer>{children[1]}</footer>
   </>
  )
}
```



### Children 、 PropTypes

在React中，`PropTypes`和`React.Children` 其实就是含有一系列属性和方法的普通对象，并不是类

只不过，`PropTypes`和`React.Children`中，提供的是一系列工具函数

所以React为了和普通对象进行区分，将这些工具函数对象的命名使用了大驼峰写法



## 具名插槽

`具名插槽`就是给插槽取名字，相应的没有名字的插槽就是`默认插槽`

具名插槽的目的是 `在传入多个插槽的时候，我们不需要考虑传入插槽的顺序`

`调用`

```jsx
<App>
  {/*
    1. 通过slot设置插槽名
    2. 这个slot是我自定义的属性，不是React内置的，可以随意命名
  */}
  <div slot='footer'>footer</div>
  {/* 没有名称的插槽就是 默认插槽 */}
  <div>content</div>
  <div slot='header'>header</div>
</App>
```

`定义`

```jsx
import React from 'react'

export default function App(props) {
  const children = React.Children.toArray(props.children)

  // 在子组件中获取的插槽 本质是VDOM，不是JSX
  const headerSlot = children.filter(child => child.props.slot === 'header')
  const defaultSlot = children.filter(child => !child.props.slot)
  const footerSlot = children.filter(child => child.props.slot === 'footer')

  return (
    <>
      {headerSlot}
      {defaultSlot}
      {footerSlot}
    </>
  )
}
```

