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

  // 判断是否是单一的JSX元素
  // 如果是 则返回该JSX元素
  // 如果不是 直接报错
  // console.log(React.Children.only(props.children))

  // 将props.children 转换为数组
  // 推荐在使用props.children的时候，都将其先转换为数组后再使用
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



`使用示例`

```jsx
// 直接解构 从props中取出children
function List({ children }) {
  // 将传入的每一个JSX元素都使用li进行包裹
  const items = React.Children.map(children, child => <li>{child}</li>);
  return <ul>{items}</ul>;
}

function App() {
  return (
    <List>
      <span>项目1</span>
      <span>项目2</span>
    </List>
  );
}
```



### Children 、 PropTypes

在React中，`PropTypes`和`React.Children` 本质就是对象

这两个对象中提供了一系列的工具函数帮助我们简化业务流程

为了区分普通对象，react将这类工具对象的首字母大写以进行区分

