### 指令

指令是用于在CSS中使用的自定义Tailwind专用规则，为Tailwind CSS项目提供了特殊的功能

指令一般使用`@`开头



#### @tailwind

使用@tailwind指令为项目提供Tailwind的基础样式、组件样式、实用程序样式和功能类前缀

```css
/* 
	@tailwind <bucket> --- 每一个都是一个样式桶
	样式桶中的样式可以是tailwind默认提供的，也可以是通过插件注入的
*/

/* 基础样式 --- 样式重置和默认样式 */
@tailwind base;

/* 组件样式 --- 按钮, 输入框, 表单, 表格的功能类 */
@tailwind components;

/* 单一属性类 --- 宽度、高度、颜色、边距、填充等 */
@tailwind utilities;

/* 
	功能类前缀
	+ 省略 --- 按需引入功能类前缀，并将其插入到css最后
	+ 显示指定 --- 全量引入功能类前缀，并将其插入到css最后
*/
@tailwind variants;
```



#### @layer

`@layer`指令可以将自定义样式添加到对应的样式桶中, 有效值为`base、components或utilities`

使用@layer添加的自定义功能类可以使用任何的功能类修饰符前缀，并且可以实现按需引用



在原生CSS中，后边的样式会覆盖之前的同名样式。在Tailwind中，将样式分为了是三个样式桶，并且对应的先后顺序为`base、components、utilities`

使用 `@layer` 添加的样式会被添加到对应的样式桶的末尾，以最大可能得减少样式和样式之间的冲突问题

```css
@tailwind base;
@tailwind components;
@tailwind utilities;

/* 
	在@layer中定义的样式在构建的时候会tree shaking
	如果你希望某些自定义样式无论是否使用都存在于构建后的样式文件中
	可以将样式直接编写在css中，不使用@layer
*/
h1 {
  @apply text-lime-300;
}

/* 如果是全局的基准样式 可以直接添加在html元素或body元素上，只有对某个元素单独设置基准样式的时候，才会使用@layer base */
@layer base {
  h1 {
    @apply text-2xl;
  }
  h2 {
    @apply text-xl;
  }
}

/* 
	如果需要覆盖第三方组件的样式, 而第三方组件并没有使用Tailwind编写样式的时候
	在@layer components中编写需要对应需要覆盖的样式 是最好的选择
*/
@layer components {
  .btn-blue {
    /* 添加自定义基础样式并希望引用主题中定义的任何值，可以使用 theme 函数或 @apply 指令 */
    @apply bg-blue-500 hover:bg-blue-700 text-white font-bold py-2 px-4 rounded;
  }
}

/* 单一的功能类是位于整个样式桶最后的, 以便于对components中的样式进行定制 */
@layer utilities {
  .filter-none {
    filter: none;
  }
  .filter-grayscale {
    filter: grayscale(100%);
  }
}
```



#### @apply

@apply 是一个 CSS 的语法规则，可以让你把已经存在的一些 CSS 样式直接应用到你自己写的 CSS 样式中

使用@apply可以很好的覆盖一些不是基于Tailwind编写的第三方库的样式



默认情况下，使用`@apply`添加的功能类不具有`!important`

如果需要具备`!important`，只需要在添加的样式的末尾添加`!important`即可

```css
/* Input */
.btn {
  @apply font-bold py-2 px-4 rounded !important;
}

/* Output */
.btn {
  font-weight: 700 !important;
  padding-top: .5rem !important;
  padding-bottom: .5rem !important;
  padding-right: 1rem !important;
  padding-left: 1rem !important;
  border-radius: .25rem !important;
}
```



像 Vue 和 Svelte 这样的框架在底层会独立处理每个 `<style>` 块，会针对每个块单独运行 PostCSS 插件链

这也就导致了如果多个组件通过使用`@layer`在同一个样式桶中添加对应样式的时候，Tailwind无法知道这些自定义样式在样式桶中的先后关系，容易出现样式冲突

所以不允许在组件化系统中通过`@layer`添加自定义样式，如果需要添加，要结合Tailwind的插件系统一起使用

```js
// tailwind.config.ts

// 引入Tailwind的插件函数
const plugin = require('tailwindcss/plugin')

module.exports = {
  // 这种方法可以让任何由 Tailwind 处理的文件都可以使用这个配置文件中定义的样式
  // 因为此时就相当于把多个自定义样式合并到了一个单独的文件中, 以方便Tailwind处理自定义样式的先后顺序
  // 但最好不要滥用 @apply 功能来做这样的事情，以获取更好的开发体验
  plugins: [
    // 完整的参数结构 --- { addBase, addComponents, addUtilities, theme }
    plugin(function ({ addComponents, theme }) {
      addComponents({
        '.card': {
          // @指令只能在CSS中使用，在JavaScript文件中，只能使用theme函数
          backgroundColor: theme('colors.white'),
          borderRadius: theme('borderRadius.lg'),
          padding: theme('spacing.6'),
          boxShadow: theme('boxShadow.xl'),
        }
      })
    })
  ]
}
```



#### @config

使用 @config 指令来自定义Tailwind对应的配置文件名和存储路径

```css
/* 
	@import有语法限制，必须位于css文件的最前边(注释不算) 
	所以对应的顺序为
		@import xxx;
		@config xxx;
		@tailwind xxx;
*/
@import "tailwindcss/base";
@import "tailwindcss/components";
@import "tailwindcss/utilities";

/* @config对应的路径是相对于当前css文件路径而言的 */
@config "./tailwind.prod.config.js";

@tailwind base;
@tailwind components;
@tailwind utilities;
```



### 函数

Tailwind 添加了一些会在构建阶段执行的自定义函数，这些函数会在构建的时候被执行，以确保在运行时对应的函数会被静态值替换



#### theme()

使用 theme() 函数使用点符号访问 Tailwind 配置值

```css
.content-area {
  /*
  	spacing-12 ---> spacing.12
  	spacing-2.5 ---> spacing[2.5]
    colors-blue-500 ---> colors.blue.500
    bg-sky-500/75 ---> bg.sky.500 / 75% ==> 75%表示透明度为0.75
  */
  height: calc(100vh - theme(spacing.12));
}
```



#### screen()

使用 Tailwind 的 screen 函数，可以使用 Tailwind 预设的响应式断点来编写媒体查询，而不必手动设置具体的断点值，从而使 CSS 代码更加简洁易读，同时也方便维护和更新

```css
/* input */
@media screen(sm) {
  /* ... */
}

/* output */
@media (min-width: 640px) {
  /* ... */
}
```

