## JSX -> VDOM

1. 基于 `babel-preset-react-app` 把JSX编译为 `React.createElement(...)`
   + `babel-preset-react-app = @babel/preset-env + @babel/preset-react + @babel/preset-typescript`
   + 只要是元素节点(组件或html元素) 必然会被转换为 `React.createElement(...)`
2. 再把 createElement 方法执行，创建出virtualDOM
   + VDOM也有称之为：JSX元素、JSX对象、ReactChild对象...



### React.createElement(elem，props，... children)

1. `elem`
   + 组件名 --- 类名/函数名
   + html元素 --- 元素名 对应的字符串
2. props
   + 属性对象
   + 如果没有传入属性 则可以传入null
   + 如果该属性值是null的时候，该属性可以省略
3. 第三个及以后的参数
   + 当前元素对应的子元素
   + 如果没有子节点，第三个及以后参数可以不传

```js
virtualDOM = {
  $$typeof: Symbol(react.element), // 一个symbol值 标志该对象是一个VDOM
  ref: null,
  key: null,
  type: Symbol(react.fragment), // 节点类型
  // props中 存放了 所有 属性(props) 和 子节点信息(children)
  // 如果即没有props也没有children 则props为 {} --- props一定是一个对象
  props: { 
      className: "title",
      // children 是一个可选属性 - 没有children时可以不写
      // 如果有子节点
      // 1. 如果只有一个子节点，children是一个对象/字符串
      // 2. 如果有多个子节点，children是一个数组
      children: 'hello world'
  }
}
```



### 模拟实现React.createElement

```jsx
export function createEelement(elem, props = {}, ...children) {
  const virtualDom = {
    $$typeof: Symbol.for('react.element'),
    type: elem,
    ref: null,
    key: null,
    props: {
      ...(props ?? {} )
    },
  }

  if (children.length) {
    virtualDom.props.children = children.length === 1 ? children[0] : children
  }

  return virtualDom
}
```

