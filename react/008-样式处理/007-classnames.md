[classnames](https://www.npmjs.com/package/classnames)是一个可以方便我们快速编写class样式类的第三方库

vue默认集成了classnames，所以理论上，如果我们在react中安装了classnames，就可以像vue那样编写class样式表



## 对象写法

`classNames`参数是一个样式类列表，可以接收的类型值如下:

1. 任意普通类型值 

2.  `Recored<string, boolean>`  

   + 第一个参数，表示样式类名，可以使用计算属性名

   + 如果第二个参数不是boolean类型值，会优先转换为布尔类型值

最终留下来的样式是

1. truthy值
2. `Recored<string, boolean>`中，第二个参数为true的样式

```jsx
classNames(null, false, 'bar', undefined, 0, 1, {[`foo-${bar}`]: true }, { baz: null }, '')// => 'bar foo-bar 1'
```



## 数组写法

样式类列表中也可以传入一个数组，会自动对数组进行展开后进行识别

```jsx
const arr = ['b', { c: true, d: false }];
classNames('a', arr); // => 'a b c'
```



