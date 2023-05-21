## 覆盖默认值

要完全替换默认断点，请直接在theme键下添加自定义屏幕配置

任何没有被覆盖的默认屏幕（例如xl）将被删除，并且不会作为屏幕修饰符可用

```js
// tailwind.config.js
module.exports = {
  theme: {
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



## 覆盖单个屏幕

要覆盖单个屏幕尺寸（如lg），在`theme.extend`下添加自定义屏幕值

这将用相同名称替换默认屏幕值，而不改变断点的顺序

```js
// tailwind.config.js
module.exports = {
  theme: {
    extend: {
      screens: {
        'lg': '992px',
        // => @media (min-width: 992px) { ... }
      },
    },
  },
}

```



## 添加最大的断点

添加更大的断点最简单的方法是使用extend选项， 此时自定义的最大断点将被默认添加到整个断点列表的最后

```js
// tailwind.config.js
module.exports = {
  theme: {
    extend: {
      screens: {
        '3xl': '1600px',
      },
    },
  }
}
```



## 添加最小的断点

如果想要添加额外的小断点，则不能使用extend，因为小断点将添加到断点列表的末尾，并且断点需要按照从小到大的顺序排序，以便与min-width断点系统一起按预期工作, 所以如果需要添加最小的断点，则需要重写整个断点列表

```js
// tailwind.config.js

// tailwindcss/defaultTheme --- Tailwind默认的theme配置
const defaultTheme = require('tailwindcss/defaultTheme')

module.exports = {
  theme: {
    screens: {
      'xs': '475px',
      // 直接使用tailwindcss/defaultTheme中导出的screens，从而避免了自己维护默认断点列表
      ...defaultTheme.screens,
    },
  },
  plugins: [],
}
```



## 使用自定义断点名

可以自定义自己喜欢的断点名，不必遵循Tailwind默认使用的`sm/md/lg/xl/2xl约定`

```js
// tailwind.config.js
module.exports = {
  theme: {
    screens: {
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

```html
<!-- 使用 -->
<div class="grid grid-cols-1 tablet:grid-cols-2 laptop:grid-cols-3 desktop:grid-cols-4">
  <!-- ... -->
</div>
```



## 高级配置

默认情况下，断点指定的是`min-width`

如果需要更多控制媒体查询，则还可以使用对象语法定义它们，该语法允许指定明确的`min-width和max-width值`



### max-width 断点

如果是希望指定`max-width`而不是`min-width`, 则可以将屏幕指定为具有max键的对象

```js
// tailwind.config.js
module.exports = {
  theme: {
    screens: {
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

上述的响应式断点会被编译为如下CSS规则 

```css
/* 
	对于CSS媒体查询规则 会从上往下进行解析，匹配第一个符合规则的媒体查询条件，后续的媒体查询条件将不再被执行
	所以如果使用max-width, 请确保对应的响应式断点是倒序排列的，以便按预期覆盖彼此
*/
@media (max-width: 1535px) {
  /* Styles for 2xl screens */
}

@media (max-width: 1279px) {
  /* Styles for xl screens */
}

@media (max-width: 1023px) {
  /* Styles for lg screens */
}

@media (max-width: 767px) {
  /* Styles for md screens */
}

@media (max-width: 639px) {
  /* Styles for sm screens */
}
```



### 固定范围断点

如果希望断点同时指定`min-width和max-width`，请一起使用`min和max键`：

```ts
// tailwind.config.js
module.exports = {
  theme: {
    screens: {
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



### 多重范围断点

有的时候在多个媒体查询条件下，所需要渲染的样式是一致的，此时就可以使用多重范围断点

```js
// tailwind.config.js
module.exports = {
  theme: {
    screens: {
      'sm': '500px',
      'md': [
        {'min': '668px', 'max': '767px'},
        {'min': '868px'}
      ],
      'lg': '1100px',
      'xl': '1400px',
    }
  }
}
```

上述的响应式断点会被编译为如下CSS规则

```css
@media (min-width: 500px) {
  /* Styles for sm screens */
}

@media (min-width: 668px) and (max-width: 767px) {
  /* Styles for md screens between 668px and 767px */
  /* 这里的样式和 @media (min-width: 868px) 中的样式是一样的 */
}

@media (min-width: 868px) {
  /* Styles for md screens above 868px */
  /* 这里的样式和 @media (min-width: 668px) and (max-width: 767px) 中的样式是一样的 */
}

@media (min-width: 1100px) {
  /* Styles for lg screens */
}

@media (min-width: 1400px) {
  /* Styles for xl screens */
}
```



### 自定义媒体查询

如果想要对生成的媒体查询拥有完全控制权，可以使用`raw`关键字

```js
// tailwind.config.js
module.exports = {
  theme: {
    extend: {
      screens: {
        // 在这个媒体查询对象中，因为使用了raw关键字，所以min关键字和max关键字将被忽略
        'tall': { 'raw': '(min-height: 800px)' },
        // => @media (min-height: 800px) { ... }
      }
    }
  }
}
```

