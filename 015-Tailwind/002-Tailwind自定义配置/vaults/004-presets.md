Tailwind 插件可以进一步扩展 Tailwind CSS 的功能，提供更多的样式规则，以便更轻松地实现特定的设计需求

```ts
// 导入插件函数
const plugin = require('tailwindcss/plugin')

module.exports = {
  plugins: [
    plugin(function({ addUtilities, addComponents, e, config }) {
      // 在这里添加您的自定义样式
    }),
  ]
}
```

plugin函数的参数是一个对象，该对象在解构后会存在如下方法

- addUtilities() --- 注册utility功能类
- matchUtilities()，用于注册新的动态实用程序样式
- addComponents() --- 注册 component功能类
- matchComponents()，用于注册新的动态组件样式
- addBase() --- 注册基础功能类
- addVariant() --- 添加功能类前缀
- matchVariant()，用于注册自定义动态变体
- theme() --- 获取配置中的值
- config()，用于查找用户的Tailwind配置中的值
- corePlugins()，用于检查是否启用了核心插件
- e()，用于手动转义字符串，以便在类名中使用