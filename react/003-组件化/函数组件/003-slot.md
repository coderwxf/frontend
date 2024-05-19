插槽 是一种 特殊的props

如果父组件向子组件传递

1. 传递 数据值 使用 props
2. 传递 html结构 使用 插槽

所以插槽的作用和props 是 一致的 `都是为了增加 组件的复用性 或 可扩展性`

Vue默认已经实现了对应的插槽机制，但是React中默认并没有实现对应的插槽机制，需要我们自行实现



## 基本使用

`调用`

```html
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

react编译是深度优先遍历，如果遇到子节点，会优先编译子节点，子组件渲染完毕后，才会继续父组件的渲染流程

所以传入的`props.children`是编译后的vdom

```jsx
export default function App(props) {
  return (
   <div>
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



## 具名插槽

具名插槽就是给插槽一个名字

对应着没有名字的插槽就是默认插槽

目的是 `在传入多个插槽的时候，我们不需要考虑传入插槽的顺序`



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



具名插槽还可以`通过props进行实现`

`调用`

```jsx
<Layout
  header={<h1>Header Section</h1>}
  content={<p>This is the main content of the page.</p>}
  footer={<footer>© 2024 Company Name</footer>}
/>
```

`定义`

```jsx
import React from 'react';

function Layout({ header, content, footer }) {
  return (
    <div>
      <div className="header">{header}</div>
      <div className="content">{content}</div>
      <div className="footer">{footer}</div>
    </div>
  );
}

export default Layout;
```

