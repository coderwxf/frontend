可以在`tailwind.config.js`的`theme.colors`中配置自定义颜色

```ts
module.exports = {
  theme: {
    // theme.extend.colors中的配置是对tailwindcss/colors中的颜色扩展
    extend: {
      colors: {
        // 为现有颜色进行扩展
        blue: {
        	950: '#17275c',
        },
        // 添加全新的自定义颜色
        brown: {
          50: '#fdf8f6',
          100: '#f2e8e5',
          200: '#eaddd7',
          300: '#e0cec7',
          400: '#d2bab0',
          500: '#bfa094',
          600: '#a18072',
          700: '#977669',
          800: '#846358',
          900: '#43302b',
        },
      }
    },
    // theme.colors 中的配置会覆盖tailwindcss中的默认颜色配置
    colors: {
      transparent: 'transparent',
      current: 'currentColor',
      'white': '#ffffff',
      'purple': '#3f3cbb',
      'midnight': '#121063',
      'metal': '#565584',
      'silver': '#ecebff',
      'bubble-gum': '#ff77e9',
      'bermuda': '#78dcca',
      // 当调色板包含同一颜色的多个不同阴影时, 使用对象语法会更方便
      'tahiti': {
        100: '#cffafe',
        200: '#a5f3fc',
        300: '#67e8f9',
        400: '#22d3ee',  // => 对应功能类为 bg-tahiti-400 其余同理
        DEFAULT: '#06b6d4', // => DEFAULT是一个特殊的默认值 对应功能类为 bg-tahiti -- 不需要数值后缀
        600: '#0891b2',
        700: '#0e7490',
        800: '#155e75',
        900: '#164e63',
      },
    }
  }
}
```

```ts
// 引入tailwindcss的默认颜色配置
const colors = require('tailwindcss/colors')

module.exports = {
  theme: {
    // 如果想要禁用任何默认颜色，最好的方法是覆盖默认颜色调色板，只包含任何所需要的颜色值
    colors: {
      transparent: 'transparent',
      current: 'currentColor',
      black: colors.black,
      white: colors.white,
      // 同时可以很方便的为颜色值起别名
     	gray: colors.slate,
      green: colors.emerald,
      indigo: colors.indigo,
      yellow: colors.yellow,
    },
  },
}
```



#### 自定义颜色名

默认情况下，Tailwind使用文字颜色名称和数字比例，但是在Tailwind中自定义颜色名是十分容易的

```ts
const colors = require('tailwindcss/colors')

module.exports = {
  theme: {
    colors: {
      primary: colors.indigo,
      secondary: '#ecc94b',
      neutral: colors.gray
    }
  }
}
```



#### 使用CSS变量

```css
@tailwind base;
@tailwind components;
@tailwind utilities;

@layer base {
  :root {
    /* 
      在Tailwind中定义css变量的时候，只需要定义RGB信道即可
      不需要使用rbg函数包裹，以便于css变量可以正常和alpha信道一起使用
    */

    /* 对于 rgb(255 115 179 / <alpha-value>) */
     --color-primary: 255 115 179;

     /* 对于 hsl(198deg 93% 60% / <alpha-value>) */
    --color-primary: 198deg 93% 60%;

    /* 对于 rgba(255, 115, 179, <alpha-value>) */
    --color-primary: 255, 115, 179;
  }
}
```

```ts
module.exports = {
  theme: {
    // 通过结合css变量和alpha修饰符来定义含有透明度值的颜色功能类
    colors: {
      // 使用现代的 `rgb`
      primary: 'rgb(var(--color-primary) / <alpha-value>)',
      secondary: 'rgb(var(--color-secondary) / <alpha-value>)',

      // 使用现代的 `hsl`
      primary: 'hsl(var(--color-primary) / <alpha-value>)',
      secondary: 'hsl(var(--color-secondary) / <alpha-value>)',

      // 使用旧的 `rgba`
      primary: 'rgba(var(--color-primary), <alpha-value>)',
      secondary: 'rgba(var(--color-secondary), <alpha-value>)',
    }
  }
}
```

