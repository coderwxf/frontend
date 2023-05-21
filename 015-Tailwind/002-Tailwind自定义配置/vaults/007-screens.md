在 `tailwind.config.js` 文件的`theme.screens`部分定义项目的断点，其中key为断点名，value是断点开始的最小值

```ts
module.exports = {
  theme: {
    // 默认的断点形式是基于移动端优先的
    screens: {
      'sm': '640px',
      // => @media (min-width: 640px) { ... }

      'md': '768px',
      // => @media (min-width: 768px) { ... }

      'lg': '1024px',
      // => @media (min-width: 1024px) { ... }

      'xl': '1280px',
      // => @media (min-width: 1280px) { ... }

      '2xl': '1536px',
      // => @media (min-width: 1536px) { ... }
    }
  }
}
```



### 覆盖默认值

```ts
module.exports = {
  theme: {
    // theme.screens下定义的断点值 会覆盖默认值
    screens: {
      'sm': '576px',
      // => @media (min-width: 576px) { ... }

      'md': '960px',
      // => @media (min-width: 960px) { ... }

      'lg': '1440px',
      // => @media (min-width: 1440px) { ... }
    },
  }
}
```



### 覆盖单个值

```ts
module.exports = {
  theme: {
    extend: {
      screens: {
        // 在theme.extend.screens中添加存在于内置断点中的同名断点，会达到覆盖单个值的效果
        'lg': '992px',
        // 在theme.extend.screens中添加内置断点中不存在的断点，该断点会被插入到默认断点列表的最后，以实现添加更大断点的效果
        '3xl': '1600px'
      },
    },
  },
}
```



### 添加更小的断点

如果需要添加更小的断点，不可以使用`theme.extend.screens`，因为`theme.extend.screens`会将默认断点添加到原始断点列表的最后，而CSS在解析断点规则的时候是从上到下进行解析的，使用匹配规则的第一条规则

所以如果使用`theme.extend.screens`添加更小的断点，会先匹配到之前更大的断点，添加到最后的最小断点将永远无法被正确匹配

```ts
const defaultTheme = require('tailwindcss/defaultTheme')

module.exports = {
  theme: {
    screens: {
      // 结合默认提供的断点列表 可以很方便的添加更小的断点
      'xs': '475px',
      ...defaultTheme.screens,
    },
  },
  plugins: [],
}
```



### 使用自定义名称

你可以为自定义的断点命名任何你喜欢的名称，不必遵循 Tailwind 默认的 `sm/md/lg/xl/2xl` 命名规则。

```ts
module.exports = {
  theme: {
    screens: {
      // key 是 断点名
      // 值 是 断点开始的最小值
      'tablet': '640px',
      // => @media (min-width: 640px) { ... }

      'laptop': '1024px',
      // => @media (min-width: 1024px) { ... }

      'desktop': '1280px',
      // => @media (min-width: 1280px) { ... }
    },
  }
}
```



### 最大宽度断点

```ts
module.exports = {
  theme: {
    screens: {
      // 使用对象语法和max关键字 可以很方便的使用添加最大宽度断点
      // 最大断点必须逆序输出 才可以得到正确的结果
      '2xl': {'max': '1535px'},
      // => @media (max-width: 1535px) { ... }

      'xl': {'max': '1279px'},
      // => @media (max-width: 1279px) { ... }

      'lg': {'max': '1023px'},
      // => @media (max-width: 1023px) { ... }

      'md': {'max': '767px'},
      // => @media (max-width: 767px) { ... }

      'sm': {'max': '639px'},
      // => @media (max-width: 639px) { ... }
    }
  }
}
```



## 固定宽度断点

```ts
module.exports = {
  theme: {
    screens: {
      // 同时指定max和min 可以设置固定宽度断点
      'sm': {'min': '640px', 'max': '767px'},
      // => @media (min-width: 640px and max-width: 767px) { ... }

      'md': {'min': '768px', 'max': '1023px'},
      // => @media (min-width: 768px and max-width: 1023px) { ... }

      'lg': {'min': '1024px', 'max': '1279px'},
      // => @media (min-width: 1024px and max-width: 1279px) { ... }

      'xl': {'min': '1280px', 'max': '1535px'},
      // => @media (min-width: 1280px and max-width: 1535px) { ... }

      '2xl': {'min': '1536px'},
      // => @media (min-width: 1536px) { ... }
    },
  }
}
```



## 多范围断点

```ts
module.exports = {
  theme: {
    screens: {
      'sm': '500px',
      'md': [
        // 如果在两个宽度范围内应用相同的样式，可以配置多范围断点
        {'min': '668px', 'max': '767px'},
        {'min': '868px'}
      ],
      'lg': '1100px',
      'xl':'1400px',
    }
  }
}
```



## 设置原生断点

```ts
module.exports = {
  theme: {
    extend: {
      screens: {
        // 通过 raw关键字 可以设置原生的响应式断点
        'tall': { 'raw': '(min-height: 800px)' },
        // => @media (min-height: 800px) { ... }
      }
    }
  }
}
```

