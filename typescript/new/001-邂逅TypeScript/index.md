TypeScript（简称 TS）是微软公司开发的一种基于 JavaScript （简称 JS）语言的编程语言

TypeScript 可以看成是 JavaScript 的超集（superset），即它继承了后者的全部语法，所有 JavaScript 脚本都可以当作 TypeScript 脚本（但是可能会报错）

TypeScript = JavaScript + 静态类型系统



**类型是人为添加的一种编程约束和用法约束**，类型一般用来表示一组具有相同特征的值

如果两个值具有某种共同的特征，就可以说，它们属于同一种类型

举例来说，`123`和`456`这两个值，共同特征是都能进行数值运算，所以都属于“数值”（number）这个类型



一旦确定某个值的类型，就意味着，这个值具有该类型的所有特征，

凡是适用该类型的地方，都可以使用这个值；凡是不适用该类型的地方，使用这个值都会报错。



```ts
function addOne(n:number) {
  return n + 1;
}

addOne('hello') // 报错
```



类型系统的优点

1. 静态类型分析 在运行之前 就可以检查出类型错误
2. 函数定义里面加入类型 可以作为使用文档的一部分，具有提示作用，可以告诉开发者这个函数怎么用
3. 类型注解可以方便IDE帮助我们进行代码提示 和 自动补全



## 动态类型与静态类型

TypeScript 的主要功能是为 JavaScript 添加类型系统

JavaScript 语言本身就有一套自己的类型系统



在语法上，JavaScript 属于动态类型语言。

JavaScript 的类型系统非常弱，而且没有使用限制，运算符可以接受各种类型的值

对应的类型 只有在代码运行的时候 才可能知道 对应的类型

这对于提前发现代码错误，非常不利。



TypeScript 引入了一个更强大、更严格的类型系统，属于静态类型语言

TypeScript 有助于提高代码质量，保证代码安全，更适合用在大型的企业级项目

1. 有利于代码的静态分析

   ​	有了静态类型，不必运行代码，就可以确定变量的类型，从而推断代码有没有错误

   ​	可以在运行之前就发现错误 尽可能早的发现

   ​	捕获的报错 可能是语法错误 也可能是不合适的操作

2. 更好的 IDE 支持，做到语法提示和自动补全

   IDE（集成开发环境，比如 VSCode）一般都会利用类型信息，提供语法提示功能（编辑器自动提示函数用法、参数等）和自动补全功能（只键入一部分的变量名或函数名，编辑器补全后面的部分）

3. 提供了代码文档

   类型信息可以部分替代代码文档，解释应该如何使用这些代码，熟练的开发者往往只看类型，就能大致推断代码的作用



`引入了独立的编译步骤`

原生的 JavaScript 代码，可以直接在 JavaScript 引擎运行。添加类型系统以后，就多出了一个单独的编译步骤，检查类型是否正确，并将 TypeScript 代码转成 JavaScript 代码，这样才能运行。



## 类型声明

```ts
let foo:string; // string -- 类型注解
```

```ts
function toString(num:number):string {
  return String(num);
}
```



类型声明并不是必需的，如果没有，TypeScript 会自己推断类型。

```ts
let foo = 123;
```



TypeScript 的设计思想是，类型声明是可选的，你可以加，也可以不加。

即使不加类型声明，依然是有效的 TypeScript 代码，只是这时不能保证 TypeScript 会正确推断出类型

由于这个原因，所有 JavaScript 代码都是合法的 TypeScript 代码



这样设计还有一个好处，将以前的 JavaScript 项目改为 TypeScript 项目时，你可以逐步地为老代码添加类型，即使有些代码没有添加，也不会无法运行



## 编译

JavaScript 的运行环境（浏览器和 Node.js）不认识 TypeScript 代码。所以，TypeScript 项目要想运行，必须先转为 JavaScript 代码，这个代码转换的过程就叫做“编译”（compile）。

TypeScript 官方没有做运行环境，只提供编译器。编译时，会将类型声明和类型相关的代码全部删除，只留下能运行的 JavaScript 代码，并且不会改变 JavaScript 的运行结果

TypeScript 的类型检查只是编译时的类型检查，而不是运行时的类型检查。一旦代码编译为 JavaScript，运行时就不再检查类型了。



## 类型和值

“类型”是针对“值”的，可以视为是后者的一个元属性。

每一个值在 TypeScript 里面都是有类型的。

比如，`3`是一个值，它的类型是`number`。



TypeScript 项目里面，其实存在两种代码，一种是底层的“值代码”，另一种是上层的“类型代码”

TypeScript 代码只涉及类型，不涉及值。所有跟“值”相关的处理，都由 JavaScript 完成。

它们是可以分离的，TypeScript 的编译过程，实际上就是把“类型代码”全部拿掉，只保留“值代码”。



## tsc 编译器

TypeScript 官方提供的编译器叫做 tsc，可以将 TypeScript 脚本编译成 JavaScript 脚本。本机想要编译 TypeScript 代码，必须安装 tsc

根据约定，TypeScript 脚本文件使用`.ts`后缀名，JavaScript 脚本文件使用`.js`后缀名。tsc 的作用就是把`.ts`脚本转变成`.js`脚本

```shell 
npm install -g typescript

tsc -v # 或者 tsc --version
```



`tsc`命令后面，加上 TypeScript 脚本文件，就可以将其编译成 JavaScript 脚本。

```shell
tsc file1.ts file2.ts file3.ts
```

上面命令会在当前目录生成三个 JavaScript 脚本文件`file1.js`、`file2.js`、`file3.js`。



为了保证编译结果能在各种 JavaScript 引擎运行，tsc 默认会将 TypeScript 代码编译成很低版本的 JavaScript，即3.0版本（以`es3`表示）。这通常不是我们想要的结果。

这时可以使用`--target`参数，指定编译后的 JavaScript 版本。建议使用`es2015`，或者更新版本。

```shell
$ tsc --target es2015 app.ts
```



编译过程中，如果没有报错，`tsc`命令不会有任何显示。所以，如果你没有看到任何提示，就表示编译成功了。

如果编译报错，`tsc`命令就会显示报错信息，但是这种情况下，依然会编译生成 JavaScript 脚本。



编译器的作用只是给出编译错误，至于怎么处理这些错误，那就是开发者自己的判断了。开发者更了解自己的代码，所以不管怎样，编译产物都会生成，让开发者决定下一步怎么处理。

如果希望一旦报错就停止编译，不生成编译产物，可以使用`--noEmitOnError`参数。

```shell
$ tsc --noEmitOnError app.ts
```



tsc 还有一个`--noEmit`参数，只检查类型是否正确，不生成 JavaScript 文件。

```shell
$ tsc --noEmit app.ts
```



## tsconfig.json 

TypeScript 允许将`tsc`的编译参数，写在配置文件`tsconfig.json`。

只要当前目录有这个文件，`tsc`就会自动读取，所以运行时可以不写参数。



```shell
$ tsc file1.ts file2.ts --outFile dist/app.js
```

上面这个命令写成`tsconfig.json`，就是下面这样。

```json
{
  // 需要编译的ts文件数组
  "files": ["file1.ts", "file2.ts"],
  // 编译选项
  "compilerOptions": {
    // 所有ts文件编译后 输出到一个文件中
    "outFile": "dist/app.js"
  }
}
```

有了这个配置文件，编译时直接调用`tsc`命令就可以了。

```shell
$ tsc
```



## ts-node 模块 

[ts-node](https://github.com/TypeStrong/ts-node) 是一个非官方的 npm 模块，可以在node环境中直接运行 TypeScript 代码。

1. ts -> js
2. 运行js

```shell
$ npm install -g ts-node

$ ts-node script.ts
```



如果执行 ts-node 命令不带有任何参数，它会提供一个 TypeScript 的命令行 REPL 运行环境，

要退出这个 REPL 环境，可以按下 Ctrl + d，或者输入`.exit`

