TypeScript 是由微软开发的编程语言，是 JavaScript 的语法超集，主要通过引入静态类型系统来增强 JavaScript 的功能。

TypeScript 在编译时对代码进行类型检查，这有助于开发者在编码阶段及早发现潜在的错误，而不是等到运行时才发现问题。



## 类型

**类型**指的是一组具有相同特性的值的集合。例如，数字类型包括 `123` 和 `456`

这些值具有共同的特征，可以进行相同的操作并使用相同的 API。



### 静态类型

JavaScript 是一种动态类型语言，具体表现为：

1. **变量类型可以动态改变**：同一个变量可以在不同时间表示不同的类型值。
2. **对象结构可以动态修改**：对象的属性可以随时添加或删除。

虽然动态类型提供了极大的灵活性，但也带来了不可预测的风险：JavaScript 引擎在执行代码前，无法确定变量的具体类型。许多类型错误只能在运行时被发现。

TypeScript 的**静态类型系统**通过在编译阶段确定变量的类型，显著降低了此类风险。借助 IDE 和插件支持，TypeScript 能够在编码时捕获大部分类型错误，帮助开发者尽早发现问题

```ts
// 变量名: 类型注解 = 初始值
let age: number = 25; // 变量 age 被定义为 number 类型
```



TypeScript 可以自动推断变量和函数的类型，即使没有显式声明类型。

```ts
let foo = 123; // TypeScript 推断 foo 为 number 类型
```



尽管 TypeScript 可以通过类型推导自动识别大多数变量的类型，但在复杂场景中，类型推导可能不准确。

此时开发者可以手动添加类型注解或使用类型断言来明确声明变量类型。

```ts
let num = [1, 2]; // 推断类型为 number[]

let tuple: [number, number] = [1, 2]; // 显式指定元组类型
```



如果类型推导失败，默认类型为 `any`

这就确保了所有 JavaScript 代码都是合法的 TypeScript 代码，可以逐步为现有项目添加类型支持

```ts
function greet(name) { // 此时name的类型就是any
  console.log("Hello, " + name);
}
```

但`any` 类型表示可以是任何类型，本质就是关闭静态类型检测，如果使用过多容易导致类型安全问题

可以通过开启 `noImplicitAny` 选项让TypeScript 在无法推断出具体类型时报错，而不是使用`any`



#### 静态类型的优点

1. **静态分析**：可以在不运行代码的情况下发现潜在的错误。
2. **错误检查**：可以更早地发现拼写错误、类型错误等问题。
3. **IDE 支持**：通过智能提示和自动补全提高开发效率。
4. **代码文档**：类型信息充当了部分文档的角色。
5. **代码重构**：有助于降低重构的风险，特别是在大型项目中。



#### 静态类型的缺点

1. **灵活性降低**：静态类型系统减少了 JavaScript 动态类型的灵活性。
2. **工作量增加**：需要为变量和函数编写类型声明。
3. **学习成本**：需要掌握更为复杂的类型系统。
4. **编译步骤**：浏览器无法直接识别TypeScript，需要将 TypeScript 代码编译成 JavaScript
   + 常见的编译器包括 `tsc`（官方）、`babel` 和 `esbuild`等。
5. **兼容性问题**：某些模块可能是JavaScript编写的，需要自己为其添加类型声明



## 编译

JavaScript 的运行时环境（如浏览器和 Node.js）无法直接识别 TypeScript

因此需要将 TypeScript 转换为 JavaScript，这个过程称为“编译”。



TypeScript 提供了一个编译器 `tsc`，用于将 TypeScript 编译为 JavaScript。除此之外还有`babel` 和 `esbuild`等

```shell
# 安装 TypeScript
npm install -g typescript

# 查看版本
tsc --version

# 编译 TypeScript 文件
tsc example.ts

# 实时编译 TypeScript 文件
tsc example.ts --watch

# TypeScript 编译器会解析项目中的依赖关系，自动编译所有需要的文件，无需手动编译每个文件
tsc index.ts # index.ts 是入口文件
```



### 类型擦除

在编译过程中，类型声明和类型相关的代码会被删除，只保留可执行的 JavaScript 代码。这种行为被称为**类型擦除**。大多数 TypeScript 的语法扩展都会在编译时被移除，不影响 JavaScript 的运行时。

```ts
// TypeScript 代码
let message: string = "Hello, TypeScript";

// 编译后的 JavaScript 代码
let message = "Hello, TypeScript";
```

不过，并非所有 TypeScript 的语法扩展都会被类型擦除。

例如，枚举、命名空间等会在编译后以对象形式注入编译后的JavaScript中。



### 编译配置

执行编译时，参数过多不利于编写和维护，可以将参数放到配置文件 `tsconfig.json` 中。

当执行 `tsc` 命令时，会自动读取该文件并编译当前目录及子目录中所有 `.ts` 文件。

所以`tsconfig.json` 一般位于项目根目录

```shell
# 初始化 tsconfig.json
tsc --init
```



### 编译错误处理

- **默认行为**：即使有错误，`tsc` 也会生成 JavaScript 文件。
- **参数控制**：
  - `--noEmitOnError`：报错时不生成编译产物。
  - `--noEmit`：仅检查类型，不生成 JavaScript 文件。

```shell
tsc --noEmitOnError example.ts
```



## tsc

- tsc** 是 TypeScript 的官方命令行编译器。
- 它的主要功能是检查 TypeScript 代码并将其编译为 JavaScript。
- 默认情况下，**tsc** 会自动使用当前目录下的 `tsconfig.json` 配置文件。
- 命令行参数可以覆盖 `tsconfig.json` 中的响应设置。



**使用 tsconfig.json 配置：**

```bash
tsc
```

直接运行 `tsc` 会根据 `tsconfig.json` 的配置进行编译。



**编译单个文件：**

```bash
tsc index.ts
```



**编译目录下的所有文件：**

```bash
tsc src/*.ts
```



**指定编译配置文件：**

```shell
tsc --project tsconfig.production.json
```



TSC的绝大部分参数都可以配置在`tsconfig.json`的`compilerOptions`中

但有一些只能通过命令行方式传入

1. **--watch**：进入观察模式，文件修改时自动重新编译。
2. **--experimentalDecorators**:
   + 装饰器是实验性功能，TS默认关闭了对其的类型检测 「 使用直接报错 」
   + 可以通过该配置项来开启TS对装饰器的类型检测功能
3. **--skipLibCheck**：跳过 `.d.ts` 文件的类型检查
   + 在大项目中，使用 `--skipLibCheck` 可以跳过对所有 `.d.ts` 文件的类型兼容性检查
   + 包括第三方库和自己编写的声明文件，从而提升编译性能。
   + 关闭后语法检查仍然会进行，所以语法错误仍会被捕捉到

