Tailwind 是一个用于构建定制用户界面的框架，所以Tailwind可以非常方便的进行定制和扩展

默认情况下，Tailwind 会在项目根目录中查找一个可选的 `tailwind.config.js` 文件，可以在其中定义任何自定义[配置](https://github.com/tailwindlabs/tailwindcss/blob/master/stubs/config.full.js)

```ts
// tailwind.config.js
// 配置的任何部分都是可选的，任何没有自定义的部分都将使用默认配置
module.exports = {
  // content用于指定那些文件需要被tailwind处理
  content: ['./src/**/*.{html,js}'],
  theme: {
    // 对功能类的预设值进行修改
    colors: {
      'blue': '#1fb6ff',
      'purple': '#7e5bef',
      'pink': '#ff49db',
      'orange': '#ff7849',
      'green': '#13ce66',
      'yellow': '#ffc82c',
      'gray-dark': '#273444',
      'gray': '#8492a6',
      'gray-light': '#d3dce6',
    },
    fontFamily: {
      sans: ['Graphik', 'sans-serif'],
      serif: ['Merriweather', 'serif'],
    },
    // 对内置的功能类进行自定义扩展
    extend: {
      spacing: {
        '8xl': '96rem',
        '9xl': '128rem',
      },
      borderRadius: {
        '4xl': '2rem',
      }
    }
  },
}
```



## 创建配置文件

### 基本使用

```shell
# 该命令将在项目根目录中创建一个最小的tailwind.config.js文件
npx tailwindcss init
```



### 自定义配置文件名

```shell
# 自定义配置文件名
npx tailwindcss init tailwindcss-config.js
```

如果自定义了配置文件名，需要在postcss中显示指定对应配置文件名

```shell
module.exports = {
  plugins: {
    tailwindcss: { config: './tailwindcss-config.js' },
  },
}
```

或者，可以使用 `@config` 指令指定自定义配置路径

```css
@config "./tailwindcss-config.js";

@tailwind base;
@tailwind components;
@tailwind utilities;
```



### 自动生成postcss配置文件

```shell
# 在生成最基本的tailwind.config.js的同时，自动生成最基本的postcss.config.js
npx tailwindcss init -p
```



### 完整配置

对于大多数用户，建议尽可能保持配置文件的简洁性，并仅指定想要自定义的内容

但如果在某些情况下，希望使用[完整的配置文件](https://github.com/tailwindlabs/tailwindcss/blob/master/stubs/config.full.js)，可以加上`--full`参数

```shell
npx tailwindcss init --full
```



## 配置

### content

所有使用了Tailwind功能类的文件都需要经过Tailwind进行编译后才可以被浏览器识别

而content属性就是配置那些文件使用了Tailwind的功能类

在  [Content 配置](vaults/001-content.md) 中了解有关配置内容源的更多信息



### theme

theme是自定义所有功能类的值和扩展功能类的可选值的地方

```ts
module.exports = {
  // ...
  theme: {
    colors: {
      'blue': '#1fb6ff',
      'purple': '#7e5bef',
      'pink': '#ff49db',
      'orange': '#ff7849',
      'green': '#13ce66',
      'yellow': '#ffc82c',
      'gray-dark': '#273444',
      'gray': '#8492a6',
      'gray-light': '#d3dce6',
    },
    fontFamily: {
      sans: ['Graphik', 'sans-serif'],
      serif: ['Merriweather', 'serif'],
    },
    extend: {
      spacing: {
        '8xl': '96rem',
        '9xl': '128rem',
      },
      borderRadius: {
        '4xl': '2rem',
      }
    }
  }
}
```

在  [主题配置指南](vaults/002-theme.md)  中了解有关默认主题和如何自定义主题的更多信息



### color

可以在`tailwind.config.js`的`theme.colors`中配置[自定义颜色](./vaults/006-colors.md)



### screen

在 `tailwind.config.js` 文件的 `theme.screens` 部分定义[项目的断点](./vaults/007-screens.md)

可以根据需要自行决定断点的数量和断点的名称




### plugin

plugin用于扩展功能类，添加自定义功能类

```ts
module.exports = {
  // ...
  plugins: [
    require('@tailwindcss/forms'),
    require('@tailwindcss/aspect-ratio'),
    require('@tailwindcss/typography'),
    require('tailwindcss-children'),
  ],
}
```

在  [插件编写指南](vaults/003-plugin.md)  中了解有关编写自己的插件的更多信息



### presets

Presets 部分允许指定自己的自定义基础配置，而不是使用 Tailwind 的默认基础配置

```ts
module.exports = {
  // ...
  presets: [
    require('@acmecorp/base-tailwind-config')
  ],

  // Project-specific customizations
  theme: {
    //...
  },
}
```

在  [预设文档](vaults/004-presets.md)  中了解有关预设的更多信息



### prefix

当在现有 CSS 代码库中使用 Tailwind 时，可能会出现与 Tailwind 类名重复的情况，这可能会导致样式冲突或被覆盖，从而影响网站的外观和布局

在这种情况下，可以使用 prefix 选项来添加一个自定义前缀，以避免与现有类名冲突

该前缀会添加到 Tailwind 生成的所有类名上，并且只对 Tailwind 类名生效，不会影响自己定义的类名

这样，就可以确保 Tailwind 类名与现有类名不会发生冲突，同时保持代码库的可读性和一致性

```ts
module.exports = {
  // 设置前缀后，再使用功能类的时候，每个功能类都必须加上前缀
  // 例如 .tw-text-left, .tw-text-center, .tw-text-right
  prefix: 'tw-',
}
```

```html
<!-- 添加prefix后，依旧可以和修饰符前缀一起结合使用 -->
<div class="tw-text-lg md:tw-text-xl tw-bg-red-500 hover:tw-bg-blue-500">
  <!-- -->
</div>
```



负值的连字符修饰符应在前缀之前添加，因此如果将 tw- 配置为前缀，则 `-mt-8` 将变为 `-tw-mt-8`

```html
<div class="-tw-mt-8">
  <!-- -->
</div>
```



prefix只会被添加到内置的功能类上，如果是自定义扩展的功能类将不会默认添加prefix，需要再扩展的时候显示添加

```css
@layer utilities {
  .tw-bg-brand-gradient { /* ... */ }
}
```



### important

`important`选项可以控制Tailwind的工具类是否应该被标记为`!important`

```ts
// tailwind.config.js

module.exports = {
  // 此时 Tailwind的所有工具类(无论是内置的还是自定义扩展的)都将被生成为!important
  important: true,
}
```



#### 选择器策略

为所有的功能类添加`!important`很有可能会和第三方库中的一些内联样式冲突

功能类的样式会强行覆盖第三方库的内联样式，从而无法达到我们所需要的效果

此时可以使用选择器策略

```ts
// tailwind.config.js

module.exports = {
  // 此时功能类编译后会变成 #app .font-bold --- 而不是使用!important
  // 选择器可以任意，不一定是id选择器，也可以是类选择器，或其他类型的选择器
  // 选择器需要存在于项目的根节点上，以确保选择器策略可以作用于所有的Tailwind功能类
  important: '#app',
}
```



#### 重要修饰符

可以通过在任何功能类前缀之后、任何前缀(prefix)之前的工具类名称的开头`!`来为该功能类添加`!important`

通过这种方式可以为某一个单独的功能类设置`!important` --- 这是最推荐的选项

```html
<div class="sm:hover:!tw-font-bold" />
```



### separator

`separator`选项用于自定义功能类和选择器之间的连字符，默认是`:`

如果将`separator`选项设置为下划线（`_`），则`.text-center:hover`将变成`.text-center_hover`

更改`separator`选项不会影响Tailwind的功能或性能，它只会影响生成的类名的格式

这种方式适合于Pug等这种模板引擎，因为在Pug中不允许在类名中使用特殊字符



### core plugins

Tailwind CSS 通过模块化的方式提供各种功能，每个功能都是通过一个或多个插件实现的。默认情况下，Tailwind 会启用所有的插件，以便能够生成所有的 CSS 类

但是，在某些情况下，可能只需要部分插件提供的功能，这时候可以使用 `corePlugins` 选项来禁用不需要的插件，以减少生成的 CSS 体积

在使用构建工具进行构建的时候，会执行`tree shaking`操作，这也就意味着，会自动按需加载使用的功能类，此时core plugins是完全没有任何意义的

但是在不使用构建工具的情况下，如果需要完成类似tree shaking的功能，就需要手动配置所需要使用的插件，以减少生产的css文件体积



如果corePlugins的值是一个数组, 那么就说明这个数组是白名单, 在这个数组中的所有配置都是需要被使用的插件

```ts
module.exports = {
  corePlugins: [
    'margin',
    'padding',
    'backgroundColor',
    // ...
  ]
}
```



## preflight

Preflight 是一个针对 Tailwind 项目的基础样式集，Preflight基于[modern-normalize](https://github.com/sindresorhus/modern-normalize)进行二次开发

Preflight 旨在提供一致的基础样式，并消除跨浏览器不一致性，所以可以将Preflight看成是对Tailwind项目进行样式初始化操作的一组[基础样式集](https://unpkg.com/tailwindcss@%5E3/src/css/preflight.css)



在Tailwind中，preflight主要进行了以下样式重置，如果需要有条件的引用浏览器原生默认样式，可以使用Tailwind官方提供的插件[@tailwindcss/typography](https://tailwindcss.com/docs/typography-plugin)

1. 所有元素的`默认margin`都被清空，重置为了`0`

2. 所有标题样式被清空

3. 列表样式被清空

4. 可替换元素都是块级元素  --- 设置了display的值为block

5. 图片，视频等宽度为父容器的宽度，高度自适应 --- 可以使用工具类`max-w-none`取消这个默认行为

6. 默认边框都被清空

   ```css
   *,
   ::before,
   ::after {
     /*  
     	默认边框被重置为宽度为0的实现边框，颜色为默认颜色
     	所以只需要简单添加border类就可以非常方便的实现宽度为1px的边框样式
     */
     border-width: 0;
     border-style: solid;
     border-color: theme('borderColor.DEFAULT', currentColor);
   }
   ```

   
