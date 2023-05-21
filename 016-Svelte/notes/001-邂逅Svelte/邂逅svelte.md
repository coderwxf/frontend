Svelte是一个用于快速开发和构建响应式用户界面的Web框架

Svelte类似于Vue和React，具体响应式，渐进式， 组件化等相应特点

1. Svelte是一个渐进式的Web框架，可以根据需要逐步引入和使用，可以使用Svelte构建整个应用程序，也可以逐步将其添加到现有的代码库中

2. Svelte将组件编译为原生JavaScript，使得它们可以在任何支持JavaScript的运行环境中运行，而不需要依赖于特定的前端框架

3. Svelte也是基于组件化的应用程序，其中每个组件都包含了HTML(模板),  CSS(样式),  JavaScript(逻辑)的代码块，这些代码块一起工作，构成了一个可复用的代码块

   Svelte中的组件是以`.svelte`作为后缀名的SFC文件，与Vue和React类似，Svelte的组件化方法能够提高应用程序的可维护性和重用性，使得开发和维护Web应用程序变得更加容易

   

## 特点

### 编译时完成

Svelte 是一种全新的构建用户界面的方法， 传统的框架（如 React 和 Vue）大部分工作都在浏览器中完成，而 Svelte 则将这些工作放在了构建应用程序时的编译步骤中

也就是说，Svelte 通过编译器在构建应用程序时先将组件的代码编译为体积极小的纯净(不依赖于框架)的 JavaScript 代码，然后再将其运行在浏览器中

通过这种方式，Svelte 可以将很多运行时的工作转移到编译时，减少浏览器中的工作量，从而使得应用程序的加载和运行速度更快，同时也提高了开发效率

因此，可以将 Svelte 看作是一个`编译器，用于在编译时将 Svelte 代码编译为原生 JavaScript 代码`



### 没有 virtual dom

Vue 和 React 的做法是通过虚拟 DOM diff 算法来找到需要更新的最小元素，然后批量更新 DOM，以达到提高性能的目的

虚拟 DOM diff 算法需要进行大量的计算和比较，因此对于一些大型项目来说，才能体现出其优势



而 Svelte 的做法是在编译阶段就将组件中的代码转换成操作 DOM 的代码，当组件的状态发生变化时，Svelte 会根据新的状态重新计算出需要更新的 DOM 部分，并将这些更新操作编译成 JavaScript 代码，然后直接应用到实际的 DOM 树上

这种做法被称之为手术式更新(surgical updates), `手术式更新可以避免虚拟 DOM 的计算和比较过程，避免了虚拟 DOM 的消耗`，因此在一些中小型项目中可以提高性能



总的来说，Vue 和 React 通过虚拟 DOM diff 算法来最小化 DOM 操作次数，适用于大型项目；而 Svelte 通过直接操作 DOM 元素来更新视图，避免了虚拟 DOM 的计算和比较过程，适用于中小型项目。



虽然Svelte是直接操作DOM的，但用户并不需要手动操作DOM，而是通过编写Svelte组件来实现页面的交互和更新。Svelte框架的编译器会自动将组件转换为直接操作DOM的代码，这样可以减少手动操作DOM的工作量，同时提高了应用程序的性能和可维护性



### 响应式

Svelte在没有虚拟DOM的情况下实现了与React 和 Vue 相同的功能

所以Svelte框架也是响应式的，即可以在状态发生改变时自动更新组件的视图和副作用



## 安装

```shell
# 通过官方脚手架SvelteKit初始化svelte项目 --- 基于vite和vite-plugin-svelte
# 如果使用vscode可以安装官方插件Svelte for VS Code --- 以在编写组件时提供语法高亮显示和诊断消息
npm create svelte@latest svelte-demo
# 上述命令本质是 npm create vite@latest svelte-demo -- --template svelte 
# svelte可以换成svelte-ts 以自动集成TypeScript
```



### 入口文件

```js
import App from './App.svelte';

// 每一个svelte组件都会被编译为原生JavaScript类，通过new关键字创建实例
const app = new App({
  // 挂载点
	target: document.body,
  // 传递给App.svelte的props
	props: {
		answer: 42
	}
});
```



## 基础语法

[001-基础语法](001-基础语法.md)





我们将使用Svelte + Vite模板。您不必使用项目模板，但这意味着您必须做很少的设置工作。

npm create vite@latest my-svelte-project -- --template svelte

npm create svelte@latest svelte-demo

这将创建一个新目录my-svelte-project，从create-vite / template-svelte模板添加文件，并从npm安装了许多软件包

在文本编辑器中打开该目录并四处看看。应用程序的“源代码”位于src目录中，而应用程序可以加载的文件位于public中

