## 简介

欢迎来到Svelte参考文档！本文档旨在为那些已经对Svelte有一定了解并希望了解更多使用方法的人提供资源。

如果您还不熟悉Svelte，您可能更喜欢在查看本文档之前先参考交互式教程或示例。您可以使用REPL在线尝试Svelte。或者，如果您想要一个更全面的环境，可以在StackBlitz上尝试Svelte。



### 开始第一个程序

我们建议使用==**Svelte团队的官方应用程序框架SvelteKit**：==

```shell
npm create svelte@latest myapp
cd myapp
npm install
npm run dev
```

==**SvelteKit将处理调用Svelte编译器将您的.svelte文件转换为创建DOM的.js文件和样式它的.css文件**==。它还提供了构建Web应用程序所需的所有其他组件，例如开发服务器、路由、部署和SSR支持==。**SvelteKit使用Vite构建您的代码**==。



SvelteKit的替代方案

如果出于某种原因您不想使用SvelteKit，还可以使用Svelte与Vite（但不使用SvelteKit），方法是==**运行npm init vite并选择svelte选项。使用此选项，npm run build将在dist目录中生成HTML、JS和CSS文件**==。在大多数情况下，您可能还需要选择一个路由库。

此外，==**对于所有主要的Web捆绑器，都有插件来处理Svelte编译——这将输出.js和.css文件，您可以将其插入到HTML中**==——但大多数其他库不处理SSR。

==**这些构建工具的插件主要关注编译和打包阶段，而不涉及更高级的功能，例如路由和状态管理**==