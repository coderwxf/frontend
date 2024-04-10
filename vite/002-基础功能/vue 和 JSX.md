## Vue

Vite 为 Vue 提供第一优先级支持，安装官方插件就可自动识别SFC文件，并支持HMR



## JSX

`.jsx` 和 `.tsx` 文件同样开箱即用

JSX 的转译同样是通过 esbuild

如果想在vue中使用JSX，需要安装额外插件，以支持vue中JSX的特殊写法



Vite 提供了一套用于模块热替换（HMR）的 API，这允许开发者在不刷新页面的情况下，即时地替换、添加或删除模块

由于这套 API 是基于原生的 ECMAScript 模块（ESM）实现的，所以它可以提供非常快速和高效的 HMR 功能

对于常用的前端框架如 Vue、React，Vite 对HMR功能提供了内置支持，是开箱即用的