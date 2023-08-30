当`isolatedModules`为`true`时，每个 TypeScript 文件都被视为独立的模块。这意味着在一个文件中定义的内容不会自动成为其他文件的全局可见项。如果您想在一个文件中使用另一个文件中的内容，您需要通过模块导入和导出来实现。

相反，当`isolatedModules`的值为默认的`false`时，所有的 TypeScript 文件都被视为在同一个上下文中，它们共享相同的作用域。这意味着在一个文件中定义的内容可以在其他文件中直接访问，而无需显式的导入和导出。

isolatedModules在`tsconfig.json`中的默认值是false





小程序 迭代 字符串





## 类 

thisType