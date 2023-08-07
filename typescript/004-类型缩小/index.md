想象一下我们有一个名为`padLeft`的函数。

```typescript
function padLeft(padding: number | string, input: string): string {
  throw new Error("Not implemented yet!");
}
```

如果`padding`是一个数字，它将把数字作为我们想要在`input`前面添加的空格数。如果`padding`是一个字符串，它应该只是在`input`前面添加`padding`。让我们尝试实现当`padLeft`的`padding`参数传递一个数字时的逻辑。

```typescript
function padLeft(padding: number | string, input: string) {
  // " ".repeat(3) --- 将空格重复显示3次
  return " ".repeat(padding) + input;
}

//Argument of type 'string | number' is not assignable to parameter of type 'number'.
//  Type 'string' is not assignable to type 'number'.
```

哎呀，我们在`padding`上得到了一个错误。TypeScript警告我们，我们将一个类型为`number | string`的值传递给了`repeat`函数，而`repeat`函数只接受一个数字，这是正确的。换句话说，我们没有明确检查`padding`是否为数字，也没有处理它是字符串的情况，所以让我们做这个处理。

```typescript
function padLeft(padding: number | string, input: string) {
  if (typeof padding === "number") {
    return " ".repeat(padding) + input;
  }
  return padding + input;
}
```

如果这段代码看起来像普通的JavaScript代码，那正是我们的目的。除了我们添加的类型注解之外，这段TypeScript代码看起来就像JavaScript。TypeScript的类型系统的目标是尽可能地使编写典型的JavaScript代码变得简单，而不需要过多地为了获得类型安全而努力。

尽管它看起来不起眼，但在这里实际上发生了很多事情。==就像TypeScript使用静态类型对运行时值进行分析一样，它还会在JavaScript的运行时控制流结构（如if/else、条件三元运算符、循环、真值检查等）上叠加类型分析，这些结构都会影响类型。==

==在我们的if检查中，TypeScript看到`typeof padding === "number"`，并将其理解为一种特殊形式的代码，称为类型保护==。==TypeScript跟踪我们程序可能采取的执行路径，分析给定位置的值的最具体可能类型。它查看这些特殊的检查（称为类型保护）和赋值，并且将类型缩小为比声明更具体的类型的过程称为缩小==。在许多编辑器中，我们可以观察到这些类型的变化，我们在示例中也会这样做。

```typescript
function padLeft(padding: number | string, input: string) {
  if (typeof padding === "number") {
    return " ".repeat(padding) + input;
		// (parameter) padding: number
  }
  return padding + input;        
	// (parameter) padding: string
}
```

TypeScript理解了几种不同的构造用于缩小类型。



## `typeof`和类型保护

正如我们所见，==JavaScript支持一个`typeof`运算符，它可以在运行时提供关于值的基本类型信息。TypeScript期望它返回一组特定的字符串：==

- "string"
- "number"
- "bigint"
- "boolean"
- "symbol"
- "undefined"
- "object"
- "function"

就像我们在`padLeft`中看到的那样，这个运算符在许多JavaScript库中经常出现，而TypeScript可以理解它并在不同的分支中缩小类型。

在TypeScript中，对`typeof`返回的值进行检查是一种类型保护。因为TypeScript对`typeof`在不同值上的操作方式进行了编码，它了解JavaScript中的一些怪癖。例如，请注意在上面的列表中，`typeof`不返回字符串"null"。看看下面的例子：

```typescript
function printAll(strs: string | string[] | null) {
  if (typeof strs === "object") {
    for (const s of strs) {
     // 'strs' is possibly 'null'.
      console.log(s);
    }
  } else if (typeof strs === "string") {
    console.log(strs);
  } else {
    // do nothing
  }
}
```

在`printAll`函数中，我们尝试检查`strs`是否为对象，以确定它是否为数组类型（现在可能是一个巧合的时机，强调一下，在JavaScript中，数组是对象类型）。但事实证明，==在JavaScript中，`typeof null`实际上是"object"==！这是历史上的一个不幸意外。

有足够经验的用户可能不会感到惊讶，但并不是每个人都在JavaScript中遇到过这个问题；幸运的是，TypeScript可以让我们知道`strs`只被缩小到了`string[] | null`，而不仅仅是`string[]`。

这可能是一个很好的过渡，引入我们将称之为“真值检查”的概念。



## 真值检测

"Truthiness"虽然可能不是你在字典中找到的词，但在JavaScript中却是常用的术语。

在JavaScript中，我们可以在条件语句、&&、||、if语句、布尔取反（!）等地方使用任何表达式。例如，if语句不要求其条件始终具有布尔类型。

```typescript
function getUsersOnlineMessage(numUsersOnline: number) {
  if (numUsersOnline) {
    return `There are ${numUsersOnline} online now!`;
  }
  return "Nobody's here. :(";
}
```

==在JavaScript中，像if这样的结构首先将其条件“强制转换”为布尔值，然后根据结果是true还是false选择相应的分支==。像以下值：

- 0
- NaN
- ""（空字符串）
- 0n（大整数版本的零）
- null
- undefined

都被强制转换为false，而其他值则被强制转换为true。你可以通过将它们传入Boolean函数或使用较短的双重布尔取反来将值强制转换为布尔值。（==后者的优点是TypeScript会推断出一个狭义的字面量布尔类型true，而对于前者，TypeScript会推断为布尔类型。==）

```typescript
// 这两个结果都是 'true'
Boolean("hello"); // 类型: boolean，值: true
!!"world"; // 类型: true，值: true
```

在JavaScript中，利用这种行为特性非常流行，特别是用于防范null或undefined等值。例如，让我们尝试在printAll函数中使用它。

```typescript
function printAll(strs: string | string[] | null) {
  if (strs && typeof strs === "object") {
    for (const s of strs) {
      console.log(s);
    }
  } else if (typeof strs === "string") {
    console.log(strs);
  }
}
```

你会注意到，通过检查strs是否为truthy值，我们已经消除了之前的错误。这至少可以防止我们在运行代码时出现以下可怕的错误：

```shell
TypeError: null is not iterable
```

然而，请记住，对于原始值的truthiness检查往往容易出错。例如，考虑另一种尝试编写printAll的方式：

```typescript
function printAll(strs: string | string[] | null) {
  // !!!!!!!!!!!!!!!!
  //  不要这样做！
  //   继续阅读
  // !!!!!!!!!!!!!!!!
  if (strs) {
    if (typeof strs === "object") {
      for (const s of strs) {
        console.log(s);
      }
    } else if (typeof strs === "string") {
      console.log(strs);
    }
  }
}
```

我们在函数的整个主体中进行了一个truthy检查，但这有一个微妙的缺点：我们可能无法正确处理空字符串的情况。

TypeScript在这里对我们没有任何伤害，但如果你对JavaScript不太熟悉，这种行为值得注意。==TypeScript经常可以帮助你及早捕获错误，但如果你选择对一个值不做任何处理，它的作用就有限。如果你希望，你可以使用一个代码检查工具（linter）来确保你处理这类情况。==

最后，关于通过truthiness进行类型缩小的一点说明是，使用`!`进行布尔取反会从取反的分支中过滤掉值。

```typescript
function multiplyAll(
  values: number[] | undefined,
  factor: number
): number[] | undefined {
  if (!values) {
    return values;
  } else {
    return values.map((x) => x * factor);
  }
}
```



## 相等性缩小

==在TypeScript中，还可以使用switch语句和相等性检查（例如`===、!==、==和!=`）来缩小类型范围==。例如：

```typescript
function example(x: string | number, y: string | boolean) {
  if (x === y) {
    // 现在我们可以在'x'或'y'上调用任何'string'方法。
    x.toUpperCase();
    y.toLowerCase();
  } else {
    console.log(x);
    console.log(y);
  }
}
```

在上面的示例中，当我们检查x和y是否相等时，TypeScript知道它们的类型也必须相等。由于字符串是x和y都可能采用的唯一公共类型，TypeScript知道x和y在第一个分支中必须是字符串。

==检查特定字面值（而不是变量）也有效==。在我们关于真值缩小的部分中，我们编写了一个名为printAll的函数，它容易出错，因为它意外地没有正确处理空字符串。相反，我们可以进行特定的检查来排除null，并且TypeScript仍然能够正确从strs的类型中移除null。

```typescript
function printAll(strs: string | string[] | null) {
  if (strs !== null) {
    if (typeof strs === "object") {
      for (const s of strs) {
        console.log(s);
      }
    } else if (typeof strs === "string") {
      console.log(strs);
    }
  }
}
```

==JavaScript中使用`==`和`!=`进行宽松相等性检查时，也能够正确进行类型缩小。如果你不熟悉，检查`something == nulll实际上不仅检查它是否具体为null，还检查它是否可能为undefined==。同样，`something == undefined`也是如此，它检查一个值是否为null或undefined。==

```typescript
interface Container {
  value: number | null | undefined;
}
 
function multiplyValue(container: Container, factor: number) {
  // 从类型中移除'null'和'undefined'。
  if (container.value != null) {
    console.log(container.value);
 
    // 现在我们可以安全地将'container.value'乘以'factor'。
    container.value *= factor;
  }
}
```



## in操作符

