`preset` 是一种将自定义配置抽象出来并打包成可复用的配置集合的方法

这个集合可以被多个 Tailwind 项目共享使用，从而实现在不同项目中保持一致的自定义样式的目的

预设是和`tailwind.config.js`一样的配置文件，他们可以添加的配置选项是完全相同的

```ts
// 这是预设示例 被保存在@acmecorp/tailwind-base
module.exports = {
  theme: {
    colors: {
      blue: {
        light: '#85d7ff',
        DEFAULT: '#1fb6ff',
        dark: '#009eeb',
      }
    },
  plugins: [
    require('@tailwindcss/typography'),
    require('@tailwindcss/aspect-ratio'),
  ],
}
```

```ts
module.exports = {
  presets: [
    // 使用预设
    require('@acmecorp/tailwind-base')
  ],
  // 针对项目的自定义配置
  theme: {
    extend: {
      minHeight: {
        48: '12rem',
      }
    }
  }
}
```



### 合并规则

预设中的配置和项目特定的配置都会被合并在一起，最终得到 Tailwind 的完整配置

默认情况下，项目特定配置会覆盖预设中的同名配置，但有些是例外

1. theme中的配置是同名覆盖，但是extend部分是合并而不是覆盖
2. presets数组是合并而不是覆盖
3. plugins是合并而不是覆盖
4. corePlugins 选项的行为取决于是将其配置为对象还是数组
   + 如果将corePlugins 配置为对象，则合并行为是合并而不是覆盖
   + 如果将corePlugins 配置为数组，则合并行为是覆盖而不是合并



### 禁用默认配置

使用空数组可以完全禁用 Tailwind 的默认配置，并且需要手动定义所有的样式配置

```ts
module.exports = {
  presets: [],
  // 自己的额外配置
}
```

