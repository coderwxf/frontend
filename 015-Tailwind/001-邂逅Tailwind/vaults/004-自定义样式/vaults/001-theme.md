tailwind.config.js文件的theme部分是定义项目的颜色调色板、字体比例尺、字体、断点、边框半径值等内容的地方

在默认情况下，Tailwind提供了 [丰富的预设主题](https://github.com/tailwindlabs/tailwindcss/blob/master/stubs/config.full.js) 



## screen

可以在`tailwind.config.js`文件的`theme.screens`部分中定义项目的断点, 具体配置可以查看  [screen常见配置](vaults/001-screen.md) 

```js
// tailwind.config.js
module.exports = {
  theme: {
    screens: {
      // key是修饰符前缀, value是该断点开始生效的最小宽度
      // 默认断点如下 --- 默认断点应该已经符合常见设备分辨率
      // 可以随意定义所需数量的屏幕，并以喜欢的方式为项目命名它们
      sm: '480px',
      md: '768px',
      lg: '1024px',
      xl: '1280px',
      2xl: '1536px'
    }
  }
}
```

