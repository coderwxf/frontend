当`isolatedModules`为`true`时，每个 TypeScript 文件都被视为独立的模块。这意味着在一个文件中定义的内容不会自动成为其他文件的全局可见项。如果您想在一个文件中使用另一个文件中的内容，您需要通过模块导入和导出来实现。

相反，当`isolatedModules`的值为默认的`false`时，所有的 TypeScript 文件都被视为在同一个上下文中，它们共享相同的作用域。这意味着在一个文件中定义的内容可以在其他文件中直接访问，而无需显式的导入和导出。

isolatedModules在`tsconfig.json`中的默认值是false



JavaScript提供了一个运算符用于检查一个值是否是另一个值的“实例”。

具体来说，在JavaScript中，x instanceof Foo检查x的原型链是否包含Foo.prototype









## 类 

thisType







泛型 - 接受参数的类型
keyof类型运算符 - 使用keyof运算符创建新类型
typeof类型运算符 - 使用typeof运算符创建新类型
索引访问类型 - 使用Type['a']语法来访问类型的子集
条件类型 - 类型在类型系统中类似于if语句
映射类型 - 通过映射现有类型中的每个属性来创建类型
模板文字类型 - 通过模板文字字符串修改属性的映射类型



需要注意的是，泛型类的类型参数只适用于实例的一侧，而不包括静态成员。也就是说，在使用泛型类时，静态成员不能使用类的类型参数

```ts
function create<Type>(c: { new (): Type }): Type {
  return new c();
}
```





1. 这是md格式文本 以md格式进行解析并翻译为中文
2. 翻译要通俗易懂 方便小白理解
3. 以md格式进行返回 注意是md格式返回
