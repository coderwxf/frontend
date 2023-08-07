在引入编程社区20多年后，JavaScript现已成为有史以来最广泛的跨平台语言之一

1. JavaScript没有采用强制性的语法规则来约束代码的组织形式，开发人员可以自由选择如何组织和结构化代码

   这种自由度可以带来创造性和灵活性，但也可能导致不一致的代码风格和难以理解的代码结构

   在大型项目中缺乏统一的规范，将导致团队成员之间的合作困难，使得项目的维护和扩展变得复杂

2. JavaScript的设计之初仅仅只是为了给网页添加简单交互，因此在设计的时候，存在一些语法层面的“缺陷”

3. JavaScript是动态类型的解释型语言，所以对于JavaScript而言，只有运行代码才可能发现对应的错误



在这个过程中，最常见的错误是类型错误，也就是在期望使用某种类型值时实际使用了其他类型值

这可能是由于简单拼写错误、未理解库API接口、对运行时行为做出了错误假设或其他原因导致



因此 **TypeScript旨在成为JavaScript的静态类型检查器**

1. TypeScript将在代码运行之前对程序的各个部分进行类型校验，以确保类型安全
2. TypeScript 源于 JavaScript，并最终归于 JavaScript
   + TypeScript 仅仅只是 JavaScript的语法扩展，绝大多数的TypeScript扩展语法 都不会修改 JavaScript的运行时行为
   + TypeScript并不能被浏览器所直接解析，在实际运行前，TypeScript需要被编译为原生JavaScript
   + TypeScript 仅仅只是类型校验 并不会去修改JavaScript的运行时行为
     + TypeScript的类型错误 仅仅只是警告。即使TypeScript存在类型错误，依旧可以被编译为对应JavaScript
     + 任何合法可运行的JavaScript代码 都是合法可运行的TypeScript代码



在不运行代码的情况下检测错误被称为静态检查。根据操作的值的类型来确定什么是错误和什么不是错误被称为静态类型检查



---

---

恭喜你选择TypeScript作为你的首选语言，你已经做出了明智的决定！

你可能已经听说过TypeScript是JavaScript的一种“变种”或“扩展”。TypeScript（简称TS）和JavaScript（简称JS）之间的关系在现代编程语言中相当独特，因此了解这种关系将帮助你理解TypeScript是如何扩展JavaScript的。

什么是JavaScript？简要历史
JavaScript（也称为ECMAScript）最初是一种用于浏览器的简单脚本语言。当它被发明时，人们期望它用于嵌入网页中的简短代码片段，编写超过几十行代码将会有些不寻常。由于这个原因，早期的网页浏览器执行这样的代码速度相对较慢。然而随着时间的推移，JS变得越来越受欢迎，Web开发人员开始使用它来创建交互式体验。

为了应对JS的增加使用，Web浏览器开发者优化了执行引擎（动态编译）并扩展了JS的功能（添加API），这反过来又使得Web开发人员更多地使用它。现代网站上，你的浏览器经常运行跨越数十万行代码的应用程序。这是“Web”的长期和逐渐增长，从最初的简单静态页面网络发展成为一个支持各种丰富应用的平台。

此外，JS变得足够流行，可以在浏览器之外的环境中使用，比如使用Node.js实现JS服务器。JS的“随处运行”的特性使其成为跨平台开发的有吸引力的选择。现今有很多开发人员只使用JavaScript来编写他们的整个技术栈！

总结一下，==我们有了一种最初设计用于快速使用的语言，然后它逐渐发展成为一种用于编写数百万行代码的完整工具。每种语言都有其自己的怪癖 - 奇怪和令人惊讶的地方，而JavaScript的谦逊起源使其具有许多这样的特点。以下是一些例子：==

==JavaScript的相等运算符（`==`）会强制转换操作数，导致意外的行为：==

```javascript
if ("" == 0) {
  // 是相等的！但为什么？
}
```

```javascript
if (1 < x < 3) {
  // 对于*任何* x 值都是 true！
}
```

==JavaScript还允许访问不存在的属性：==

```javascript
const obj = { width: 10, height: 15 };
// 为什么会得到 NaN？拼写很难！
const area = obj.width * obj.heigth;
```

==大多数编程语言在出现此类错误时会抛出错误，其中一些会在编译过程中(在任何代码运行之前）抛出错误。在编写小型程序时，这些怪癖很烦人但也还能应对；但是，在编写包含数百或数千行代码的应用程序时，这些不断出现的意外就成为一个严重的问题。==

TypeScript：静态类型检查器
我们之前说过，有些语言根本不允许运行有错误的程序。==在运行之前检测代码中的错误被称为静态检查。根据操作的值的类型来确定什么是错误和什么不是错误被称为静态类型检查==。

==TypeScript在执行之前会对程序进行错误检查，并且基于值的类型进行检查，因此它是一个静态类型检查器==。例如，上面的最后一个示例由于obj的类型而出现错误。这是TypeScript发现的错误

```ts
const obj = { width: 10, height: 15 };
const area = obj.width * obj.heigth;
// Property 'heigth' does not exist on type '{ width: number; height: number; }'. Did you mean 'height'?
```



==TypeScript（简称TS）是JavaScript的一个超集，因此JavaScript的语法在TypeScript中也是合法的。语法是指我们编写程序时使用的文本形式==。例如，以下代码由于缺少一个 ")" 而导致语法错误：

```ts
let a = (4
// ')' expected.
```

==TypeScript不会因为语法而将任何JavaScript代码视为错误。这意味着你可以将任何有效的JavaScript代码放入一个TypeScript文件中，而不必担心它的具体编写方式==。

==然而，TypeScript是一个带有类型的超集，这意味着它添加了关于不同类型值如何使用的规则==。前面关于"obj.heigth"的错误并不是语法错误：它是一种错误地使用某种类型值的错误。

以下是另一个示例，这是可以在浏览器中运行的JavaScript代码，并将会记录一个值：

```ts
console.log(4 / []);
```

这个在语法上合法的程序将会输出 Infinity。然而，TypeScript认为对一个数字除以一个数组是一个无意义的操作，并会发出错误提示：

```ts
console.log(4 / []);
// The right-hand side of an arithmetic operation must be of type 'any', 'number', 'bigint' or an enum type.
```

有时候你可能真的打算将一个数字除以一个数组，可能只是为了看看会发生什么，但大多数情况下，这是一个编程错误。==TypeScript的类型检查器被设计成允许正确的程序通过，同时尽可能捕捉到常见的错误==。（稍后，我们将学习如何使用设置来配置TypeScript对代码的严格检查程度。）

==如果你将一些代码从JavaScript文件移动到TypeScript文件中，根据代码的编写方式，你可能会看到类型错误。这些错误可能是代码中的合法问题，也可能是TypeScript过于保守==。在本指南中，我们将演示如何添加各种TypeScript语法来消除此类错误。

运行时行为
==TypeScript也是一种编程语言，它保留了JavaScript的运行时行为==。例如，在JavaScript中，除以零会产生Infinity而不是抛出运行时异常。==作为一项原则，TypeScript永远不会改变JavaScript代码的运行时行为==。

这意味着，==如果你将代码从JavaScript移动到TypeScript，它保证以相同的方式运行，即使TypeScript认为代码存在类型错误==。

==保持与JavaScript相同的运行时行为是TypeScript的基本承诺，因为这意味着你可以轻松地在两种语言之间切换，而不必担心细微差别可能导致你的程序停止工作==。

擦除类型
粗略地说，==一旦TypeScript的编译器完成代码检查，它会擦除类型以生成最终的“编译”代码。这意味着一旦你的代码被编译，生成的纯JS代码将不包含任何类型信息==。

==这也意味着TypeScript永远不会根据推断的类型改变程序的行为。最重要的是，虽然你可能在编译过程中看到类型错误，但类型系统本身对程序运行时的工作方式没有影响==。

==最后，TypeScript不提供任何额外的运行时库。你的程序将使用与JavaScript程序相同的标准库（或外部库），因此没有额外的特定于TypeScript的框架需要学习。==



学习JavaScript和TypeScript

经常有人问：“我应该学习JavaScript还是TypeScript呢？”

答案是你不能学习TypeScript而不学习JavaScript！TypeScript与JavaScript共享语法和运行时行为，所以你学习JavaScript的任何知识都会同时帮助你学习TypeScript。

对于程序员来说，有很多资源可以学习JavaScript；如果你正在编写TypeScript，你不应该忽视这些资源。例如，StackOverflow上打上javascript标签的问题大约是typescript问题的20倍，但所有的javascript问题也适用于TypeScript。

如果你在搜索类似“如何在TypeScript中对列表进行排序”的内容时，记住：TypeScript是JavaScript的运行时，具有编译时的类型检查器。在TypeScript中对列表进行排序的方式与在JavaScript中相同。如果你找到了直接使用TypeScript的资源，那也很好，但不要局限于认为你需要针对如何完成运行时任务的日常问题寻找特定于TypeScript的答案。



下一步

以下是对日常使用的TypeScript语法和工具的简要概述。接下来，你可以：

学习一些JavaScript的基础知识，我们推荐以下资源之一：

- 微软的JavaScript资源
- Mozilla Web Docs上的JavaScript指南

继续学习《TypeScript for JavaScript Programmers》

从头到尾阅读完整的手册（Handbook）

探索Playground中的示例代码