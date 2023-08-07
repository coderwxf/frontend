# 基础知识

欢迎来到手册的第一页。如果这是你第一次接触TypeScript，你可能想从其中一个“入门指南”开始阅读。



==JavaScript中的每个值都具有一组可以通过运行不同操作来观察的行为==。这听起来有些抽象，但让我们以一个快速的例子来说明，假设我们对一个名为message的变量运行一些操作。

```ts
// 访问'message'的属性'toLowerCase'，并调用它
message.toLowerCase();
// 调用'message'
message();
```

如果我们来分解一下，第一行可运行的代码访问了一个名为toLowerCase的属性，然后调用它。第二行尝试直接调用message。

但是假设我们不知道message的值 - 这是非常常见的情况 - 我们无法可靠地说运行这些代码会得到什么结果。每个操作的行为完全取决于我们最初拥有的值。

message是否可调用？
它是否具有名为toLowerCase的属性？
如果是，toLowerCase是否可调用？
如果这两个值都可调用，它们会返回什么？
这些问题的答案通常是我们在编写JavaScript时要记住的事情，我们必须希望我们所有的细节都是正确的。

假设message是以下方式定义的。

```ts
const message = "Hello World!";
```

正如你可能猜到的那样，如果我们尝试运行message.toLowerCase()，我们将得到相同的字符串，只是转换为小写。

那么第二行代码呢？如果你熟悉JavaScript，你会知道这会导致一个异常错误：

```shell
TypeError: message is not a function
```

如果我们能够避免这样的错误就太好了。

==当我们运行我们的代码时，JavaScript运行时会通过确定值的类型来决定要执行的操作 - 它具有什么样的行为和能力。==这也是TypeError所暗示的一部分 - 它表示字符串"Hello World!"不能作为函数调用。

对于某些值，比如原始类型的字符串和数字，我们可以使用typeof运算符在运行时确定它们的类型。但对于其他一些东西，比如函数，没有相应的运行时机制来确定它们的类型。例如，考虑下面的函数：

```ts
function fn(x) {
  return x.flip();
}
```

通过阅读代码，我们可以观察到这个函数只能在给定具有可调用的flip属性的对象时才能正常工作，但JavaScript没有以我们可以在代码运行时检查的方式呈现这些信息。==在纯JavaScript中，唯一确定fn对特定值的操作是调用它并观察发生了什么。这种行为使得在代码运行之前很难预测代码会做什么，这意味着在编写代码时很难知道代码将要做什么。==

从这个角度来看，类型是描述哪些值可以传递给fn并且哪些值会导致错误的概念。==JavaScript只提供了真正的动态类型 - 运行代码以观察结果。==

==另一种选择是使用静态类型系统，在代码运行之前对代码的预期行为进行预测。==



## 静态类型检查

回想一下之前我们尝试将一个字符串作为函数调用时得到的TypeError。==大多数人不喜欢在运行代码时遇到任何类型的错误 - 这些都被视为错误！当我们编写新代码时，我们尽力避免引入新的错误。==

==如果我们添加了一小段代码，保存文件，重新运行代码，然后立即看到错误，我们可能能够快速定位问题；但情况并非总是如此。我们可能没有充分测试该功能，因此可能永远不会真正遇到可能抛出的错误！或者，如果我们足够幸运地遇到了错误，我们可能已经进行了大量的重构并添加了很多不同的代码，而我们被迫深入挖掘。==

理想情况下，我们可以有一个工具，==在我们的代码运行之前帮助我们找到这些错误。这就是静态类型检查器（如TypeScript）的作用。静态类型系统描述了当我们运行程序时值的形状和行为。像TypeScript这样的类型检查器使用这些信息，并告诉我们何时可能出现问题。==

```ts
const message = "hello!";
 
message();
// This expression is not callable.
//   Type 'String' has no call signatures.
```

在我们运行第一个代码之前，使用TypeScript运行最后一个示例会给我们一个错误消息。



## 非异常错误

到目前为止，我们一直讨论某些事情，比如运行时错误 - 即JavaScript运行时告诉我们某些东西是荒谬的情况。这些情况出现是因为ECMAScript规范对语言在遇到意外情况时应该如何行为有明确的指示。

例如，规范规定尝试调用不可调用的内容应该抛出错误。也许这听起来像是“显而易见的行为”，但你可以想象，在对象上访问一个不存在的属性也应该抛出错误。然而，JavaScript给我们不同的行为，并返回值undefined：

```ts
const user = {
  name: "Daniel",
  age: 26,
};
user.location; // returns undefined
```



==最终，静态类型系统必须在其系统中决定哪些代码应被标记为错误，即使它是“有效”的JavaScript，不会立即抛出错误==。在TypeScript中，以下代码会产生关于location未定义的错误：

```ts
const user = {
  name: "Daniel",
  age: 26,
};
 
user.location;
// Property 'location' does not exist on type '{ name: string; age: number; }'.
```



虽然有时这意味着在表达能力上进行权衡，但其目的是捕捉程序中的合法错误。TypeScript可以捕捉许多合法的错误。

例如：拼写错误，

```ts
const announcement = "Hello World!";

// 你能多快发现这些拼写错误？
announcement.toLocaleLowercase();
announcement.toLocalLowerCase();

// 我们可能本意是这样写的...
announcement.toLocaleLowerCase();
```

未调用的函数

```ts
function flipCoin() {
  // 本意是使用 Math.random()
  return Math.random < 0.5;
  // Operator '<' cannot be applied to types '() => number' and 'number'.
}
```

或者基本逻辑错误

```ts
const value = Math.random() < 0.5 ? "a" : "b";
if (value !== "a") {
  // ...
} else if (value === "b") {
// This comparison appears to be unintentional because the types '"a"' and '"b"' have no overlap.
  // Oops, unreachable
}
```



## TypeScript的工具化类型

TypeScript在我们编写代码时可以捕获错误。这非常棒，但TypeScript也可以防止我们在一开始就犯下这些错误。

类型检查器具有检查诸如我们是否在变量和其他属性上访问正确属性的信息。一旦它具备了这些信息，它还可以开始建议您可能想要使用的属性。

==这意味着TypeScript也可以用于编辑代码，并且核心类型检查器可以在您输入编辑器时提供错误消息和代码补全功能==。这就是人们在谈论TypeScript的工具化时常常提到的一部分。

TypeScript非常注重工具化，这不仅仅局限于在输入时提供代码补全和错误提示。支持TypeScript的编辑器可以提供"快速修复"功能，自动修复错误，提供重构功能以便轻松重新组织代码，并提供有用的导航功能，可以跳转到变量的定义处，或查找对给定变量的所有引用。所有这些功能都是基于类型检查器构建的，并且完全跨平台，因此很可能您喜欢的编辑器已经支持TypeScript。



## tsc，TypeScript编译器

我们已经讨论了类型检查，但我们还没有使用我们的类型检查器。让我们来认识一下我们的新朋友==tsc，即TypeScript编译器==。首先，我们需要通过npm安装它。

```shell
npm install -g typescript
```

这将全局安装TypeScript编译器tsc。如果您喜欢，也可以使用npx或类似的工具从本地的node_modules包运行tsc。

现在让我们进入一个空文件夹，并尝试编写我们的第一个TypeScript程序：hello.ts：

```typescript
// 向世界打招呼。
console.log("Hello world!");
```

尝试一下。请注意，这里没有花哨的东西；这个“hello world”程序看起来与您在JavaScript中编写的“hello world”程序完全相同。现在，让我们通过运行由typescript包为我们安装的tsc命令对其进行类型检查。

```
tsc hello.ts
```

塔达！

等等，“塔达”是什么意思？我们运行了tsc，但什么也没发生！嗯，没有类型错误，所以我们的控制台没有输出，因为没有任何需要报告的内容。

但是再次检查-我们得到了一些文件输出。如果我们查看当前目录，我们会在hello.ts旁边看到一个hello.js文件。那是==tsc将我们的hello.ts文件编译或转换为普通JavaScript文件的输出==。如果我们检查其内容，我们会看到TypeScript在处理.ts文件后输出了什么：

```javascript
// 向世界打招呼。
console.log("Hello world!");
```

在这种情况下，TypeScript需要转换的内容非常少，所以它与我们编写的代码完全相同。编译器尝试生成干净易读的代码，这些代码看起来像人类编写的代码。虽然这并不总是那么容易，但TypeScript会一致缩进，注意我们的代码跨越多行时的情况，并尽量保留注释。

那么如果我们引入了类型检查错误会怎么样呢？让我们重写hello.ts：

```typescript
// 这是一个工业级通用的问候函数：
function greet(person, date) {
  console.log(`Hello ${person}, 今天是 ${date}！`);
}

greet("Brendan");
```

尝试一下。如果我们再次运行`tsc hello.ts`，请注意我们在命令行上收到了一个错误！

```
Expected 2 arguments, but got 1.
```

TypeScript告诉我们我们忘记向greet函数传递一个参数，这是正确的。到目前为止，我们只编写了标准的JavaScript代码，但类型检查仍然能够找到我们代码中的问题。感谢TypeScript！



## 带有错误的输出

您可能没有注意到上一个示例中的一个细节，那就是我们的hello.js文件再次发生了变化。如果我们打开该文件，我们会看到其内容基本上与我们的输入文件相同。这可能有点令人惊讶，因为tsc报告了我们代码的错误，但这是基于TypeScript的一个核心价值观：==大部分时间里，您会比TypeScript更了解自己的代码。==

重申之前的内容，类型检查限制了您可以运行的程序类型，因此在类型检查器找到可接受的内容之间存在权衡。大多数情况下这没问题，但也有一些情况下这些检查会妨碍开发。例如，想象一下将JavaScript代码迁移到TypeScript并引入类型检查错误的情况。最终，您会着手为类型检查器清理代码，但最初的JavaScript代码已经能够正常工作！为什么将其转换为TypeScript后就无法运行了呢？

==因此，TypeScript不会妨碍您的工作。当然，随着时间的推移，您可能希望对错误更加防范，使TypeScript更加严格。在这种情况下，您可以使用noEmitOnError编译器选项==。尝试更改您的hello.ts文件，并带上该标志运行tsc：

```shell
tsc --noEmitOnError hello.ts
```

您会注意到hello.js文件永远不会被更新。



## 显式类型

到目前为止，我们还没有告诉TypeScript person和date是什么类型。让我们编辑代码，告诉TypeScript person是一个字符串类型，date应该是一个Date对象。我们还将在date上使用toDateString()方法。

```typescript
function greet(person: string, date: Date) {
  console.log(`Hello ${person}, 今天是 ${date.toDateString()}！`);
}
```

我们所做的是==在person和date上添加类型注解，以描述greet函数可以接受的值的类型。您可以将这个函数签名理解为“greet接受一个类型为string的person参数和一个类型为Date的date参数”==。

有了这个类型注解，TypeScript可以告诉我们其他可能错误地调用greet函数的情况。例如：

```typescript
function greet(person: string, date: Date) {
  console.log(`Hello ${person}, 今天是 ${date.toDateString()}！`);
}

greet("Maddison", Date());
```

TypeScript报告了一个错误，指出我们的第二个参数存在问题，但为什么呢？

也许令人惊讶的是，在JavaScript中调用Date()函数会返回一个字符串。另一方面，使用new Date()构造一个Date对象才能得到我们期望的结果。

无论如何，我们可以快速修复这个错误：

```typescript
function greet(person: string, date: Date) {
  console.log(`Hello ${person}, 今天是 ${date.toDateString()}！`);
}

greet("Maddison", new Date());
```

==请记住，我们并不总是需要编写显式的类型注解。在许多情况下，即使我们省略了类型注解，TypeScript也可以推断出变量的类型。==

```typescript
let msg = "hello there!";
```

即使我们没有告诉TypeScript msg的类型是字符串，它也能够推断出来。这是一种特性，最好在类型系统可以推断出相同类型的情况下不添加注解。

注意：前面代码示例中的消息气泡是您的编辑器在将鼠标悬停在单词上时显示的内容。



## 擦除类型

让我们来看看当我们使用tsc编译上面的greet函数以生成JavaScript代码时会发生什么：

```javascript
"use strict";
function greet(person, date) {
    console.log("Hello ".concat(person, ", today is ").concat(date.toDateString(), "!"));
}
greet("Maddison", new Date());
```

请注意以下两点：

1. ==我们的person和date参数不再具有类型注解==。
2. 我们的“模板字符串” - 使用反引号（`）括起来的字符串 - 被转换为使用字符串拼接的普通字符串。
3. ==默认开启了严格模式==

关于第二点，稍后我们会详细解释，但现在让我们专注于第一点。==类型注解并不属于JavaScript（或者严谨一点说，不属于ECMAScript），所以没有浏览器或其他运行时可以直接运行未经修改的TypeScript代码。这就是为什么TypeScript需要一个编译器 - 它需要一种方法来剥离或转换任何特定于TypeScript的代码，以便您可以运行它。大部分特定于TypeScript的代码都会被擦除，同样地，这里我们的类型注解也完全被擦除掉了==。

==记住：类型注解不会改变程序的运行行为。==



## 降级转换

除此之外，与上面的代码不同的是，我们的模板字符串被重写为：

```javascript
"Hello ".concat(person, ", today is ").concat(date.toDateString(), "!");
```

为什么会发生这种情况呢？

模板字符串是ECMAScript的一个特性，它属于ECMAScript 2015（也称为ECMAScript 6、ES2015、ES6等等）。TypeScript具有将代码从较新版本的ECMAScript重写为较旧版本（例如ECMAScript 3或ECMAScript 5，也称为ES3和ES5）的能力。==从较新或“更高”的ECMAScript版本向较旧或“更低”的版本迁移的过程有时被称为降级。==

==默认情况下，TypeScript的目标是ES3，这是一个非常古老的ECMAScript版本。通过使用`--target`选项，我们可以选择更近期一些的目标版本。使用`--target es2015`运行命令将TypeScript的目标设置为ECMAScript 2015，这意味着代码应该能够在支持ECMAScript 2015的任何地方运行==。因此，运行`tsc --target es2015 hello.ts`会得到以下输出：

```javascript
function greet(person, date) {
  console.log(`Hello ${person}, today is ${date.toDateString()}!`);
}
greet("Maddison", new Date());
```

虽然默认目标是ES3，但大多数当前的浏览器都支持ES2015。因此，大多数开发者可以安全地将目标指定为ES2015或更高版本，除非需要与某些古老的浏览器兼容。



## 严格模式

==不同的用户在使用类型检查器时寻求的功能可能不同。有些人希望获得一种更宽松的选择体验，可以帮助验证程序的部分内容，并且仍然具有良好的工具支持。这就是TypeScript的默认体验，其中类型是可选的，推断采用最宽松的类型，并且不会检查潜在的null/undefined值。就像tsc在出现错误时仍然会生成输出一样，这些默认设置旨在不干扰您。如果您正在迁移现有的JavaScript代码，这可能是一个可取的第一步。==

==相反，许多用户更喜欢让TypeScript尽可能多地进行验证，这就是语言提供严格性设置的原因。这些严格性设置将静态类型检查从开关（要么检查您的代码，要么不检查）转变为更接近调节旋钮的方式。您将此旋钮调得越高，TypeScript将为您进行的检查就越多。这可能需要一些额外的工作，但总体而言，它会在长期内回报，并提供更全面的检查和更准确的工具支持。在可能的情况下，新的代码库应该始终打开这些严格性检查。==

==TypeScript有几个类型检查严格性标志，可以打开或关闭==，除非另有说明，否则我们的所有示例代码都将使用打开的严格性设置。==在CLI中，strict标志或tsconfig.json中的"strict": true同时打开所有严格性设置，但我们也可以单独关闭它们。其中最重要的两个标志是noImplicitAny和strictNullChecks。==



### noImplicitAny

回想一下，==在某些情况下，TypeScript不会尝试为我们推断类型，而是退而求其次，使用最宽松的类型any==。这并不是最糟糕的情况 - 毕竟，退而求其次使用any只是普通的JavaScript体验。

然而，过多地使用any往往会削弱使用TypeScript的初衷。程序类型化程度越高，您将获得越多的验证和工具支持，这意味着在编码过程中会遇到更少的错误。==打开noImplicitAny标志将对那些隐式推断类型为any的变量发出错误提示。==



### strictNullChecks

==默认情况下，null和undefined等值可以赋值给任何其他类型==。这样做可以使编写某些代码更容易，但是忘记处理null和undefined是世界上无数错误的原因之一 - 有人认为这是一个价值数十亿美元的错误！==strictNullChecks标志使处理null和undefined更加明确，让我们不必担心是否忘记处理null和undefined。==