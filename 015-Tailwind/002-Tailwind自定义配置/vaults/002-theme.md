`tailwind.config.js` 文件的` theme` 部分是可以定义功能类的预设值，Tailwind默认提供了完全可以自定义的[默认配置](https://github.com/tailwindlabs/tailwindcss/blob/master/stubs/config.full.js)

```js
// tailwind.config.js
module.exports = {
  // theme下的配置会覆盖默认主题中对应配置，对于未设置的选项将依旧使用默认配置
  theme: {
    // 自定义响应式断点
    screens: {
      sm: '480px',
      md: '768px',
      lg: '976px',
      xl: '1440px'
    },
    // 自定义调色板
    colors: {
      'blue': '#1fb6ff',
      'purple': '#7e5bef',
      'pink': '#ff49db'
    },
    // 自定义间距和尺寸值
    spacing: {
      px: '1px',
      0: '0',
      0.5: '0.125rem',
      1: '0.25rem',
      1.5: '0.375rem',
    }
    // 其余部分都是用来定义core plugin中对应的值
    // key --- 功能类的后缀 value --- 功能类的值
     borderRadius: {
      'none': '0',
      'sm': '.125rem',
      DEFAULT: '.25rem',
      'lg': '.5rem',
      'full': '9999px',
    },
    fontFamily: {
      sans: ['Graphik', 'sans-serif'],
      serif: ['Merriweather', 'serif'],
    },
    // extend下的配置用于在默认主题的基础上进行扩展
    extend: {
      spacing: {
        '128': '32rem',
        '144': '36rem',
      },
      borderRadius: {
        '4xl': '2rem',
      }
    }
  }
}
```



## 引用其他值

```ts
// tailwind.config.js
module.exports = {
  theme: {
    spacing: {
      '2.5': '2.5rem',
      '3.5': '3.5rem',
    },
    // 值可以是一个函数 参数是一个含有theme函数的对象
    // theme函数可以递归调用，只要拥有结束条件皆可 --- 一般情况下，结尾的那个值是静态值 即常量值
    // theme函数会在配置合并后，再进行执行，也就是说即可以引用默认配置对应的值，也可以使用自定义配置对应的值
    backgroundSize: ({ theme }) => ({
      auto: 'auto',
      cover: 'cover',
      contain: 'contain',
      // 将spacing对应的值结构到backgroundSize功能类值中
      // 所以此时就具有了 bg-2.5 和 bg-3.5
      ...theme('spacing')
    }),
    
    // 函数值只能在theme的顶层配置中进行使用
    fill: ({ theme }) => ({
      gray: theme('colors.gray')
    })
  }
}
```



## 引用默认主题

```ts
// 引入默认主题
const defaultTheme = require('tailwindcss/defaultTheme')

module.exports = {
  theme: {
    extend: {
      fontFamily: {
        sans: [
          'Lato',
          // 使用默认主题
          ...defaultTheme.fontFamily.sans,
        ]
      }
    }
  }
}
```



## 禁用核心插件

在 Tailwind CSS 中，有一些默认的核心插件（Core Plugins），它们会提供一些常用的 CSS 类。但是，在某些情况下，你可能不需要使用这些插件中的某些类，或者你可能需要完全禁用这些插件

```ts

module.exports = {
  corePlugins: {
    // 这将禁用opacity功能类
    // 虽然值给个空对象，实现的效果也是一样的，但是因为大多数核心插件的值可能不是对象，此时就只能使用false来禁用
    // 例如float属性  float属性值只能是left，right，center 其值是不可以自定义的，所以不存在在theme关键字中通过对象语法扩展其属性值
    // 如果需要禁用float属性 只能将其值设置为false
    // 所以为了统一 推荐一律使用false来禁用核心插件
    opacity: false,
  }
}
```

