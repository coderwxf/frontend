让我们开始使用TypeScript构建一个简单的Web应用程序。

安装TypeScript


对于npm用户：

> npm install -g typescript

==以上是安装TypeScript的命令。你可以在命令行中运行这个命令来全局安装TypeScript。==

==安装完成后，你就可以使用TypeScript编译器（tsc）来编译你的TypeScript代码==



构建您的第一个TypeScript文件
在您的编辑器中，将以下JavaScript代码输入greeter.ts文件中：

```typescript
function greeter(person) {
  return "Hello, " + person;
}
 
let user = "Jane User";
 
document.body.textContent = greeter(user);
```

尝试编译您的代码
我们使用了.ts扩展名，但这段代码实际上是JavaScript代码。您可以直接从现有的JavaScript应用程序中复制/粘贴这段代码。

在命令行中运行TypeScript编译器：

```
tsc greeter.ts
```

编译结果将是一个greeter.js文件，其中包含与您输入的JavaScript代码相同的内容。我们已经使用TypeScript在我们的JavaScript应用程序中运行起来了！

现在我们可以开始利用TypeScript提供的一些新工具了。在函数参数`person`上添加一个`: string`类型注解，如下所示：

```typescript
function greeter(person: string) {
  return "Hello, " + person;
}
 
let user = "Jane User";
 
document.body.textContent = greeter(user);
```



==类型注解在TypeScript中是一种轻量级的方式，用于记录函数或变量的预期约定==。在这种情况下，我们希望greeter函数以一个字符串参数进行调用。我们可以尝试更改greeter的调用方式，传递一个数组：

```typescript
function greeter(person: string) {
  return "Hello, " + person;
}

let user = [0, 1, 2];

document.body.textContent = greeter(user);
```

将代码重新编译，你将会看到一个错误：

```
error TS2345: Argument of type 'number[]' is not assignable to parameter of type 'string'.
```

类似地，尝试移除greeter调用中的所有参数。TypeScript会提示你使用了意外数量的参数来调用该函数。==在这两种情况下，TypeScript可以基于代码的结构和提供的类型注解进行静态分析。==

==请注意，尽管出现了错误，greeter.js文件仍然会被创建。即使代码中存在错误，你仍然可以使用TypeScript。但在这种情况下，TypeScript在警告你的代码可能无法按预期运行==。



接口（Interfaces）：
让我们进一步开发我们的示例。这里我们使用一个描述具有firstName和lastName字段的对象的接口。==在TypeScript中，如果两个类型的内部结构兼容，它们就是兼容的。这使得我们可以通过具有接口所需的形状来实现接口，而不需要显式的implements子句==。

```typescript
interface Person {
  firstName: string;
  lastName: string;
}

function greeter(person: Person) {
  return "Hello, " + person.firstName + " " + person.lastName;
}

let user = { firstName: "Jane", lastName: "User" };

document.body.textContent = greeter(user);
```



类（Classes）：
最后，让我们通过类一次性扩展示例。==TypeScript支持JavaScript中的新特性，如支持基于类的面向对象编程。==

这里，我们将创建一个名为Student的类，它具有一个构造函数和一些公共字段。请注意，类和接口可以很好地结合使用，让程序员决定正确的抽象级别。

还值得注意的是，构造函数参数上的public关键字是一个简写，它允许我们自动创建具有相应名称的属性。

```typescript
class Student {
  fullName: string;
  constructor(
    public firstName: string,
    public middleInitial: string,
    public lastName: string
  ) {
    this.fullName = firstName + " " + middleInitial + " " + lastName;
  }
}

interface Person {
  firstName: string;
  lastName: string;
}

function greeter(person: Person) {
  return "Hello, " + person.firstName + " " + person.lastName;
}

let user = new Student("Jane", "M.", "User");

document.body.textContent = greeter(user);
```

重新运行`tsc greeter.ts`，你会看到生成的JavaScript与之前的代码相同。在TypeScript中，类只是JavaScript中通常使用的基于原型的面向对象的简写形式。



现在在greeter.html中输入以下内容：

```html
<!DOCTYPE html>
<html>
  <head>
    <title>TypeScript Greeter</title>
  </head>
  <body>
    <script src="greeter.js"></script>
  </body>
</html>
```

在浏览器中打开greeter.html以运行你的第一个简单的TypeScript Web应用程序！

