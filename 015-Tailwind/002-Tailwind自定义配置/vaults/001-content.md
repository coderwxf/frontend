Tailwind CSS会使用正则表达式来扫描所有HTML、JavaScript组件和任何其他使用了功能类的模板文件来生成对应的CSS

为了让Tailwind生成所需要的所有CSS，项目中任何包含Tailwind类名的文件都需要在`content`中进行配置

```ts
module.exports = {
  content: [
    // 相对路径是基于项目根目录的，而不是tailwind.config.js所在目录
    // 在content中不需要配置css文件路径 --- 因为css需要被引入到别的文件中并进行解析，因此通常不需要单独进行解析
    './components/**/*.{html,js}',
    './pages/**/*.{html,js}',
    './index.html'
  ]
}
```



content中的路径匹配采用[glob模式](https://en.wikipedia.org/wiki/Glob_(programming)), Tailwind将使用[fast-glob](https://github.com/mrmlnc/fast-glob) 进行通用匹配

+ 使用`*`匹配除斜杠和隐藏文件之外的任何内容
+ 使用`**`匹配零个或多个目录
+ 使用`{}`之间的逗号分隔值匹配选项列表



## 动态类名

在编译的时候，Tailwind会以正则表达式的形式提取出所有的功能类并解析，所以对于Tailwind而言，它只会查找到完整的，不间断的功能类

```html
<!-- 此时Tailwind将无法正常提取出对应的功能类并进行解析 -->
<div class="text-{{ error ? 'red' : 'green' }}-600"></div>

<!-- 只有当功能类完整的时候，Tailwind才可以正常提取出对应的功能类并进行解析 -->
<div class="{{ error ? 'text-red-600' : 'text-green-600' }}"></div>
```



## 与第三方结合使用

### 样式覆盖

如果引入了没有使用tailwind编写样式的第三方库，在覆盖对应样式的时候，需要使用class类名来进行样式覆盖

```css
/* main.css */
@tailwind base;
@tailwind components;

.select2-dropdown {
  @apply rounded-b-lg shadow-md;
}
.select2-search {
  @apply border border-gray-300 rounded;
}
.select2-results__group {
  @apply text-lg font-bold text-gray-900;
}

/* ... */
@tailwind utilities;
```



```ts
// tailwind.config.js
module.exports = {
  content: [
    './components/**/*.{html,js}',
    './pages/**/*.{html,js}',
    // 如果引入了使用Tailwind编写的第三方库，需要将第三方库也在content中进行配置
    // 以便于tailwind可以正确解析所有的功能类 并执行优化操作
    './node_modules/@my-company/tailwind-components/**/*.js',
    
    // 如果结合monorepo一起使用，就需要结合require.resolve一起使用
    // require.resolve('@my-company/tailwind-components') 获取该模块的绝对路径
    // path.dirname --- 获取绝对路径对应的文件夹路径
    // path.join --- 将多个路径片段使用系统路径分隔符进行拼接 形成新路径
     path.join(path.dirname(require.resolve('@my-company/tailwind-components')), '**/*.js'),
    
    // 如果需要扫描原生html字符串，那么需要使用含有raw和extension关键字的对象
    { raw: '<div class="font-bold">', extension: 'html' },
  ],
  // ...
}
```



## 路径解析

默认情况下，tailwind的路径解析是基于当前工作目录进行解析的，也就是相对于tailwind命令执行所在的目录

如果需要始终相对于`tailwind.config.js`所在的目录，则需要配置`relative`的值为true  ---- 这将在下一个主版本中成为默认行为

```ts
// tailwind.config.js
module.exports = {
  content: {
    relative: true,
    files: [
      './pages/**/*.{html,js}',
      './components/**/*.{html,js}',
    ],
  },
  // ...
}
```



